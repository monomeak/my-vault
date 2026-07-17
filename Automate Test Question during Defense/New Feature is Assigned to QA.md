# QA Flow: Testing a New Feature

> [!question] Jury Question  
> If a new feature is implemented and assigned to you for testing, how would you start? Describe the complete flow from test case design to reporting.

---

## Short Flow

-  Understand requirement
    
-  Identify test scenarios
    
-  Design test cases
    
-  Prepare test data
    
-  Execute the test
    
-  Report bugs or improvements
    
-  Automate important cases
    
-  Run regression testing
    
-  Prepare test report
    

---

## Complete Answer

> [!info] 1. Understand the Requirement  
> When a new feature is implemented and assigned to me for testing, I would start by understanding the requirement first.
> 
> I would review:
> 
> - User story
>     
> - Acceptance criteria
>     
> - UI design
>     
> - API behavior
>     
> - Business rules related to the feature
>     

---

> [!tip] 2. Identify Test Scenarios  
> After that, I would identify the test scenarios.
> 
> I would consider:
> 
> - Main successful flow
>     
> - Validation rules
>     
> - Negative cases
>     
> - Edge cases
>     
> - Permission cases
>     
> - Regression impact on existing features
>     

---

> [!example] 3. Design Test Cases  
> Then, I would design test cases based on those scenarios.
> 
> Each test case should include:
> 
> - Test steps
>     
> - Test data
>     
> - Expected result
>     
> - Actual result
>     
> 
> I would also prepare the necessary test data before execution.

---

> [!todo] 4. Execute the Test  
> Next, I would execute the test cases manually first to understand the behavior and confirm that the feature works correctly.

---

> [!bug] 5. Report Issues  
> If I find any issue, I would report the bug with:
> 
> - Clear steps to reproduce
>     
> - Actual result
>     
> - Expected result
>     
> - Screenshot
>     
> - Environment information
>     
> - Other supporting evidence
>     

---

> [!success] 6. Automate Important Cases  
> Once the feature is stable, I would select important and repeatable scenarios for automation, especially critical flows and regression cases.
> 
> I would implement automated tests using the test framework, run them locally, and then integrate them into the CI pipeline.

---

> [!check] 7. Run Regression Testing  
> After automation or fixing issues, I would run regression testing to make sure the new feature does not break existing features.

---

> [!summary] 8. Prepare Test Report  
> Finally, I would prepare a test report summarizing:
> 
> - Test coverage
>     
> - Passed and failed cases
>     
> - Detected bugs
>     
> - Automation results
>     
> - Remaining risks
>     
> 
> This helps the team and stakeholders understand the quality status of the new feature before release.

---

## Simple Speaking Version

When a new feature is assigned to me for testing, I start by understanding the requirement, including the user story, acceptance criteria, UI design, API behavior, and business rules.

Then, I identify test scenarios such as the main flow, validation cases, negative cases, edge cases, permission cases, and regression impact.

After that, I design test cases with test steps, test data, expected results, and actual results. I also prepare the required test data before execution.

Next, I execute the test manually first to confirm the feature behavior. If I find any issue, I report it clearly with steps to reproduce, actual result, expected result, screenshots, and environment information.

Once the feature is stable, I automate the important and repeatable cases, especially critical flows and regression cases. Then, I run regression testing and prepare a final test report for the team and stakeholders.

---

## SETFO Example

>[!example] Example Answer  
For example, if a new filing feature is added in SETFO, I would design test cases for valid filing, invalid filing, save as draft, checkout, payment, receipt generation, and regression impact on the filing list and public search.