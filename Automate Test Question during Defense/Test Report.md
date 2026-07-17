# Project / Feature Information
- Feature Name:  New Filing Feature 
- Project: User Portal
- Module: Filing
- Environment:  UAT / Staging 
- Tester: **Saren Sokmeak**
- Date:  08 July 2026
- Test Type: Functional Testing, Regression Testing, End-to-End Testing.

# Test Objective

>[!objective]
The objective of the test is to verify / validate that the new feature works according to the requirement, acceptance criteria, and business rules. The testing also ensures that the new features does not break existing functionality.


# Scope of the Test

>[!summary]
>The following areas were tested:
>- Valid filing creation
>- Invalid filing validation
>- Save filing as draft
>- Checkout process
>- Payment with prepaid balance
>- Receipt generation
>- Filing list update
>- Public search impact
>- Regression impact on related features

# Test Summary
| Total Test Cases | Passed | Failed | Blocked | Skipped |
| ---------------- | ------ | ------ | ------- | ------- |
| 10               | 8      | 1      | 1       | 0       |
# Test Case Result

| Test Case ID | Test Scenario | Status | Remark |
|---|---|---|---|
| TC001 | Create valid filing as draft | Passed | Filing was saved successfully |
| TC002 | Create filing with invalid data | Passed | Validation message displayed correctly |
| TC003 | Update draft filing | Passed | Draft data was updated successfully |
| TC004 | Checkout draft filing | Passed | Checkout page displayed correctly |
| TC005 | Pay with prepaid balance | Passed | Payment completed successfully |
| TC006 | Generate receipt after payment | Passed | Receipt was generated correctly |
| TC007 | Download transaction report | Failed | Download button did not respond |
| TC008 | Search paid filing by notice number | Passed | Correct filing was found |
| TC009 | Regression on filing list | Passed | Existing filing list worked correctly |
| TC010 | Certified request after filing | Blocked | Blocked due to unavailable test data |
# Defect Summary

| Bug ID | Description                                                | Severity | Status |
| ------ | ---------------------------------------------------------- | -------- | ------ |
| BUG001 | Download transaction button does not respond after payment | Medium   | Open   |
# Automation Result

>[!important]
>Important and repeatable scenarios were selected for automation. The automated test cases were executed using Playwright and integrated with Jenkins CI. 

| Automation Area | Result |
|---|---|
| Filing creation | Passed |
| Checkout and payment | Passed |
| Receipt validation | Passed |
| Public search | Passed |
# Risk / Limitation
>[!risk]
>Some scenarios could not be fully tested because of limited teste data and unavailable email service. The areas should be tested again when the required data or service become available.

# Final Conclusion

>[!summary]
>Based on the test execution result, most of the tested scenarios passed successfully. One defect was found in the transaction download feature, and one case was block due to unavailable test data.
>The  feature can be considered mostly stable, but the failed and blocked cased should be fixed and retested before production release.
