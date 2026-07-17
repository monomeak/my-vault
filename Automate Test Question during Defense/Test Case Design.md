>[!tip]
Why each test case need to be solid?

--- 
 >[!answer]
 Yes, each test case need to be solid but it should not always be too large. A solid test cases means it has a clear objective, clear steps, proper test data, and clear expected result. For example, login, create filing, checkout, payment, download transaction, and search can be tested as one complete end-to-end flow. This is useful to verify that the whole business journey works correctly. 
 However, I would not put everything into one big test case only, because if test failed, it becomes difficult to identify which part caused the issue. The Failure could come from login, filing creation, checkout payment, download, or search...

>[!summary]
I would separate them into smaller test cases for each feature such as login test, create filing test, checkout test, payment test, download transaction test, and search test. Then I would also create one complete end-to-end test case to verify the full user journey. This approach makes the test cases easier to maintain, easier to debug, and better for regression testing, while still ensuring the complete business flow is covered.

---

>[!note]
Small test cases = easier to find bugs
Full flow test = confirms the real user journey works
Separated by feature, then add one complete E2E flow for critical business scenarios.


