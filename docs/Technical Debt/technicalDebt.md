# Technical Debt

## General Strategies when solving technical debt

1. No large components - split up components into smaller pieces in separate files
2. Keep JSX Clean - it needs to have only JSX. So no functions inside JSX, complex .map calls. Always move out in seperate functions and call the function in the JSX instead.
3. Remove unnecessary code. We have a lot of this - unused state, state where only the setState is being used, functions that do some extra unnecessary checks (extra if else statements just in case - no need for this)
4. DO NOT USE ANY AS A TYPE
5. Remove ALL comments. Comments are code smells. If you need to explain your code with comments - then your code is bad and you need to re-think it.
6. Use ENUMS, CONST whenever you can. 
7. Don't use console.log, console.warn, console.error - only for debugging, otherwise remove it.
8. In Pages - try to move as much as you can to SSR. 
9. NextJS API Routes - use only for fetching data from DB, not for updating - this is a security risk. If you need more complex operations, use Firebase Functions.
10. Follow the projects coding rules in general

## Refactor: 

* FindColleges Page
    * FindCollege [schoolId] page
* Email Assistant Modal
    * useEmailFlow Hook
    * Nested Item 2.2
* Saved Colleges Page
* Remove unused components
* Pricing Section Refactor