# Sync Resend Emails Flow

```mermaid
flowchart TD
    Start([Webhook Received]) --> ValidateRequest{Validate Request Body}
    ValidateRequest -->|Missing emailData/resendEmailId| Error1[Return 400 Error]
    ValidateRequest -->|Valid| ParseEmail[Parse Resend Email]
    
    ParseEmail --> ExtractHeaders[Extract Headers:<br/>- Message-ID<br/>- In-Reply-To<br/>- References]
    ExtractHeaders --> ExtractContent[Extract Content:<br/>- From/To emails<br/>- Subject<br/>- Body text/HTML]
    ExtractContent --> CleanBody[Clean Body:<br/>extractLatestMessage<br/>Remove quoted content]
    CleanBody --> ExtractFirstDiv[Extract First Div from HTML]
    ExtractFirstDiv --> ParsedEmail{Parsed Successfully?}
    
    ParsedEmail -->|No| Error2[Return 400 Error]
    ParsedEmail -->|Yes| FindUser[Find User by Recipient Email<br/>Search users.resendEmail]
    
    FindUser --> UserFound{User Found?}
    UserFound -->|No| Error3[Return 404 Error]
    UserFound -->|Yes| CheckInReplyTo{In-Reply-To<br/>Header Exists?}
    
    CheckInReplyTo -->|Yes| Strategy1[Strategy 1: Find by In-Reply-To]
    Strategy1 --> SearchEmailMessageId[Search emails.emailMessageId<br/>with brackets]
    SearchEmailMessageId --> Found1{Found?}
    Found1 -->|Yes| SetConversationId1[Set conversationId]
    Found1 -->|No| SearchEmailMessageId2[Search emails.emailMessageId<br/>without brackets]
    SearchEmailMessageId2 --> Found2{Found?}
    Found2 -->|Yes| SetConversationId1
    Found2 -->|No| SearchConversationId[Search conversations.doc<br/>by conversationId]
    SearchConversationId --> Found3{Found?}
    Found3 -->|Yes| SetConversationId1
    Found3 -->|No| SearchResendMessageId[Search emails.resendMessageId]
    SearchResendMessageId --> Found4{Found?}
    Found4 -->|Yes| SetConversationId1
    Found4 -->|No| SearchMessageId[Search emails.messageId]
    SearchMessageId --> Found5{Found?}
    Found5 -->|Yes| SetConversationId1
    Found5 -->|No| CheckReferences{References<br/>Header Exists?}
    
    CheckInReplyTo -->|No| CheckReferences
    SetConversationId1 --> CheckConversationId{conversationId<br/>Found?}
    
    CheckReferences -->|Yes| Strategy2[Strategy 2: Find by References]
    Strategy2 --> ParseReferences[Parse References Header<br/>JSON array or space/comma separated]
    ParseReferences --> LoopReferences[Loop through each Reference ID]
    LoopReferences --> SearchRefEmailMessageId[Search emails.emailMessageId<br/>for each reference]
    SearchRefEmailMessageId --> FoundRef{Found?}
    FoundRef -->|Yes| SetConversationId2[Set conversationId]
    FoundRef -->|No| MoreReferences{More References?}
    MoreReferences -->|Yes| LoopReferences
    MoreReferences -->|No| CheckSubject{Subject<br/>Exists?}
    
    CheckConversationId -->|Yes| UpdateConversation
    CheckConversationId -->|No| CheckReferences
    SetConversationId2 --> CheckConversationId
    
    CheckSubject -->|Yes| Strategy3[Strategy 3: Find by Subject + Participants]
    Strategy3 --> NormalizeSubject[Normalize Subject<br/>Remove Re: prefix]
    NormalizeSubject --> GetUserConversations[Get all conversations<br/>for userId]
    GetUserConversations --> LoopConversations[Loop through conversations]
    LoopConversations --> MatchSubject{Subject Matches?}
    MatchSubject -->|Yes| CheckParticipants{Participants<br/>Overlap?}
    MatchSubject -->|No| MoreConversations{More Conversations?}
    CheckParticipants -->|Yes| SetConversationId3[Set conversationId]
    CheckParticipants -->|No| MoreConversations
    MoreConversations -->|Yes| LoopConversations
    MoreConversations -->|No| CreateNewConversation
    
    CheckSubject -->|No| CreateNewConversation[Create New Conversation<br/>conversationId = resendEmailId]
    
    CreateNewConversation --> SetNewConversation[Set conversation document:<br/>- participants<br/>- subject<br/>- unreadCount: 1<br/>- labels: INBOX]
    
    UpdateConversation[Update Existing Conversation] --> MergeParticipants[Merge participants]
    MergeParticipants --> UpdateFields[Update:<br/>- lastMessage<br/>- lastMessagePreview<br/>- unreadCount++<br/>- participants]
    
    SetNewConversation --> StoreEmail
    UpdateFields --> StoreEmail[Store Email Document]
    
    StoreEmail --> SetEmailFields[Set email fields:<br/>- userId<br/>- resendMessageId<br/>- conversationId<br/>- emailMessageId<br/>- from/to/subject/body<br/>- cleanedBody<br/>- attachments<br/>- labels: INBOX]
    
    SetEmailFields --> Success[Return 200 Success<br/>with emailId and conversationId]
    
    Error1 --> End([End])
    Error2 --> End
    Error3 --> End
    Success --> End
    
    style Start fill:#e1f5ff
    style Success fill:#d4edda
    style Error1 fill:#f8d7da
    style Error2 fill:#f8d7da
    style Error3 fill:#f8d7da
    style Strategy1 fill:#fff3cd
    style Strategy2 fill:#fff3cd
    style Strategy3 fill:#fff3cd
    style CreateNewConversation fill:#d1ecf1
    style UpdateConversation fill:#d1ecf1
```

## Helper Functions Flow

### parseResendEmail()

```mermaid
flowchart TD
    Start([Start: parseResendEmail]) --> GetHeaders[Extract Headers from resendEmail]
    GetHeaders --> GetMessageId[Extract Message-ID Header<br/>from headers object]
    GetMessageId --> FormatMessageId{Message-ID<br/>has brackets?}
    FormatMessageId -->|No| AddBrackets[Add angle brackets<br/>format: &lt;message-id&gt;]
    FormatMessageId -->|Yes| ExtractFrom[Extract From Email<br/>extractEmail function]
    AddBrackets --> ExtractFrom
    ExtractFrom --> ExtractTo[Extract To Emails<br/>Map through to array]
    ExtractTo --> GetSubject[Get Subject]
    GetSubject --> GetBody[Get Body Text]
    GetBody --> GetHtml[Get HTML Content]
    GetHtml --> CallExtractLatest[Call extractLatestMessage<br/>on body text]
    CallExtractLatest --> CallExtractDiv[Call extractFirstDiv<br/>on HTML content]
    CallExtractDiv --> ParseDate[Parse created_at<br/>to Date object]
    ParseDate --> MapAttachments[Map Attachments Array<br/>Convert to EmailAttachment format]
    MapAttachments --> BuildParsedEmail[Build ParsedEmail Object:<br/>- id: resendEmailId<br/>- messageId: emailMessageId<br/>- from/to/subject/body<br/>- cleanedBody<br/>- bodyFirstDiv<br/>- date/attachments]
    BuildParsedEmail --> ReturnParsed[Return ParsedEmail Object]
    ReturnParsed --> End([End])
    
    style Start fill:#e1f5ff
    style End fill:#d4edda
    style CallExtractLatest fill:#fff3cd
    style CallExtractDiv fill:#fff3cd
```

### extractLatestMessage()

```mermaid
flowchart TD
    Start([Start: extractLatestMessage]) --> CheckEmpty{emailText<br/>exists?}
    CheckEmpty -->|No| ReturnEmpty[Return Empty String]
    CheckEmpty -->|Yes| DefinePatterns[Define Reply Patterns:<br/>1. On ... at ... wrote<br/>2. From: ...<br/>3. Original Message<br/>4. Separator lines<br/>5. Quoted On ... wrote]
    DefinePatterns --> InitIndex[Initialize earliestReplyIndex<br/>to emailText.length]
    InitIndex --> LoopPatterns[Loop through each pattern]
    LoopPatterns --> MatchPattern[Match pattern in emailText]
    MatchPattern --> FoundMatch{Pattern<br/>Found?}
    FoundMatch -->|Yes| CheckIndex{Match index &lt;<br/>earliestReplyIndex?}
    FoundMatch -->|No| NextPattern{More<br/>patterns?}
    CheckIndex -->|Yes| UpdateIndex[Update earliestReplyIndex<br/>to match.index]
    CheckIndex -->|No| NextPattern
    UpdateIndex --> NextPattern
    NextPattern -->|Yes| LoopPatterns
    NextPattern -->|No| ExtractContent[Extract substring<br/>from 0 to earliestReplyIndex]
    ExtractContent --> RemoveTrailingQuotes[Remove trailing quote characters<br/>regex pattern]
    RemoveTrailingQuotes --> CleanWhitespace[Replace multiple newlines<br/>max 2 consecutive newlines]
    CleanWhitespace --> Trim[Trim whitespace]
    Trim --> ReturnCleaned[Return cleaned message]
    ReturnCleaned --> End([End])
    ReturnEmpty --> End
    
    style Start fill:#e1f5ff
    style End fill:#d4edda
    style ExtractContent fill:#d1ecf1
    style ReturnCleaned fill:#d4edda
```

### findThreadByInReplyToHeader()

```mermaid
flowchart TD
    Start([Start: findThreadByInReplyToHeader]) --> CleanId[Clean In-Reply-To ID<br/>Remove &lt;&gt; brackets and trim]
    CleanId --> CheckEmpty{cleanInReplyTo<br/>exists?}
    CheckEmpty -->|No| ReturnNull1[Return null]
    CheckEmpty -->|Yes| Search1[Search emails collection<br/>where emailMessageId == &lt;cleanInReplyTo&gt;]
    Search1 --> Found1{Found<br/>email?}
    Found1 -->|Yes| GetConvId1[Get conversationId<br/>from matching email]
    Found1 -->|No| Search2[Search emails collection<br/>where emailMessageId == cleanInReplyTo<br/>without brackets]
    GetConvId1 --> ReturnConvId1[Return conversationId]
    Search2 --> Found2{Found<br/>email?}
    Found2 -->|Yes| GetConvId2[Get conversationId<br/>from matching email]
    Found2 -->|No| Search3[Search conversations collection<br/>doc by cleanInReplyTo ID]
    GetConvId2 --> ReturnConvId2[Return conversationId]
    Search3 --> Found3{Conversation<br/>exists?}
    Found3 -->|Yes| ReturnConvId3[Return cleanInReplyTo<br/>as conversationId]
    Found3 -->|No| Search4[Search emails collection<br/>where resendMessageId == cleanInReplyTo]
    Search4 --> Found4{Found<br/>email?}
    Found4 -->|Yes| GetConvId4[Get conversationId<br/>from matching email]
    Found4 -->|No| Search5[Search emails collection<br/>where messageId == cleanInReplyTo]
    GetConvId4 --> ReturnConvId4[Return conversationId]
    Search5 --> Found5{Found<br/>email?}
    Found5 -->|Yes| GetConvId5[Get conversationId<br/>from matching email]
    Found5 -->|No| ReturnNull2[Return null]
    GetConvId5 --> ReturnConvId5[Return conversationId]
    
    ReturnConvId1 --> End([End])
    ReturnConvId2 --> End
    ReturnConvId3 --> End
    ReturnConvId4 --> End
    ReturnConvId5 --> End
    ReturnNull1 --> End
    ReturnNull2 --> End
    
    style Start fill:#e1f5ff
    style End fill:#d4edda
    style ReturnNull1 fill:#f8d7da
    style ReturnNull2 fill:#f8d7da
    style ReturnConvId1 fill:#d4edda
    style ReturnConvId2 fill:#d4edda
    style ReturnConvId3 fill:#d4edda
    style ReturnConvId4 fill:#d4edda
    style ReturnConvId5 fill:#d4edda
```

### findThreadByReferences()

```mermaid
flowchart TD
    Start([Start: findThreadByReferences]) --> CheckEmpty{references array<br/>exists and has items?}
    CheckEmpty -->|No| ReturnNull[Return null]
    CheckEmpty -->|Yes| LoopStart[Start Loop:<br/>For each reference in array]
    LoopStart --> CleanRef[Clean Reference ID<br/>Remove &lt;&gt; brackets and trim]
    CleanRef --> CheckRefEmpty{cleanRef<br/>exists?}
    CheckRefEmpty -->|No| NextRef{More<br/>references?}
    CheckRefEmpty -->|Yes| Search1[Search emails collection<br/>where emailMessageId == &lt;cleanRef&gt;]
    Search1 --> Found1{Found<br/>email?}
    Found1 -->|Yes| GetConvId1[Get conversationId<br/>from matching email]
    Found1 -->|No| Search2[Search emails collection<br/>where emailMessageId == cleanRef<br/>without brackets]
    GetConvId1 --> CheckConvId1{conversationId<br/>exists?}
    CheckConvId1 -->|Yes| ReturnConvId[Return conversationId]
    CheckConvId1 -->|No| NextRef
    Search2 --> Found2{Found<br/>email?}
    Found2 -->|Yes| GetConvId2[Get conversationId<br/>from matching email]
    Found2 -->|No| NextRef
    GetConvId2 --> CheckConvId2{conversationId<br/>exists?}
    CheckConvId2 -->|Yes| ReturnConvId
    CheckConvId2 -->|No| NextRef
    NextRef -->|Yes| LoopStart
    NextRef -->|No| ReturnNull
    
    ReturnConvId --> End([End])
    ReturnNull --> End
    
    style Start fill:#e1f5ff
    style End fill:#d4edda
    style ReturnNull fill:#f8d7da
    style ReturnConvId fill:#d4edda
    style LoopStart fill:#fff3cd
```

### findThreadBySubjectAndParticipants()

```mermaid
flowchart TD
    Start([Start: findThreadBySubjectAndParticipants]) --> NormalizeSubject[Normalize Subject<br/>Remove Re:/RE:/re: prefix<br/>and trim]
    NormalizeSubject --> CheckSubject{normalizedSubject<br/>exists?}
    CheckSubject -->|No| ReturnNull[Return null]
    CheckSubject -->|Yes| GetConversations[Get all conversations<br/>where userId == userId]
    GetConversations --> LoopStart[Start Loop:<br/>For each conversation]
    LoopStart --> GetConvSubject[Get conversation subject]
    GetConvSubject --> NormalizeConvSubject[Normalize conversation subject<br/>Remove Re:/RE:/re: prefix<br/>and trim]
    NormalizeConvSubject --> CompareSubjects{normalizedSubject ==<br/>normalizedConvSubject?<br/>case insensitive}
    CompareSubjects -->|No| CheckMore{More<br/>conversations?}
    CompareSubjects -->|Yes| GetParticipants[Get participants array<br/>from conversation]
    GetParticipants --> CheckParticipantMatch{from email in participants<br/>OR<br/>any to email in participants?}
    CheckParticipantMatch -->|No| CheckMore
    CheckParticipantMatch -->|Yes| LogMatch[Log: Found thread by<br/>subject and participants]
    LogMatch --> ReturnConvId[Return conversationId]
    CheckMore -->|Yes| LoopStart
    CheckMore -->|No| ReturnNull
    
    ReturnConvId --> End([End])
    ReturnNull --> End
    
    style Start fill:#e1f5ff
    style End fill:#d4edda
    style ReturnNull fill:#f8d7da
    style ReturnConvId fill:#d4edda
    style LoopStart fill:#fff3cd
    style CompareSubjects fill:#d1ecf1
    style CheckParticipantMatch fill:#d1ecf1
```

