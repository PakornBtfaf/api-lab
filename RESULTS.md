# API lab results

## Part 0

- Step 0.1, the number of pizza names: 18
- Step 0.3, status code of the request without a token: 401
- Step 0.4, the name of your pizza: A Quiet Americana
- Step 0.12, results of the three runs without a session (optional): failed, failed, failed
- Step 0.16, blocks of "how many of the three tools are excluded": none (0), one (1), two (2), all three (3)

## Defect report

### Defect 1

- Title: Excluding all three tools returns 500 with an empty body
- Request: `POST /api/pizza` with header `Authorization: token <token>` and body `{"excludedTools": ["Knife", "Pizza cutter", "Scissors"]}`
- Expected result: status 400 and a JSON error message
- Actual result: status 500 and an empty body
- Severity: Medium. The server fails on valid input and the app gets no message to show the customer.

### Defect 2

- Title: Made-up token is accepted and returns ratings of other users
- Request: `GET /api/ratings` with header `Authorization: token 0000000000000000`
- Expected result: status 401 and `{"error": "authentication failed"}`
- Actual result: status 200 and a list of ratings that belong to other users
- Severity: High. Authentication does not check that a token was issued by a login.

## Part A

- Step A4, status codes when the other user reads, changes and deletes: read 404, update 400, delete 400

## Part B

1. Two kinds of failure that tests against a real server can find and a unit test with stubs cannot: (a) an integration or configuration fault, for example the load balancer sending a request to a server that does not know the new user (Step 0.12); (b) a difference between what a stub promises and what the real system does, for example a 500 with an empty body from the real server (Step 0.16). One reason to write unit tests with stubs as well: they run in milliseconds, need no network or shared server, give the same result every time, and point to the exact function that is broken.

2. The public token is shared by everyone, so another student could change or delete my ratings while my test runs, and my result would depend on other people. A new user with a unique name gives every test its own private data, so the tests are independent and repeatable and I never touch data that is not mine. In Project 2 I will make each test create its own data on the shop or hotel (for example its own cart or booking with a unique name), use only that data, and delete it afterwards, so my tests never change the data of other groups.

3. A session keeps the cookies that the server sends back, here AWSALB. The load balancer reads this cookie and sends every request to the same server, and each server keeps its own users. If every request is sent on its own, the login can reach a server that never saw the new user and fail with 401. In my three runs without a session the login failed, failed, failed, so that test is flaky: its result depends on which server answers, not on my code.

4. The block "all three tools excluded" revealed the defect, because no tool is left to cut the pizza and the server answers 500. A 500 with an empty body tells the app only that the server failed, with no reason, so the app cannot show a useful message and may treat it as a crash or retry in vain. A 400 with a message says that the input was the problem and what to change. The test stays as xfail because the defect still exists and must stay visible, and the test documents the expected behaviour. With strict=True the run fails as soon as QuickPizza fixes the defect, which tells me to remove the mark.

5. Base choice coverage needs 1 + 2 + 3 + 1 + 2 = 9 tests: one base test, plus (3 - 1) = 2 for C1, (4 - 1) = 3 for C2, (2 - 1) = 1 for C3 and (3 - 1) = 2 for C4. All combinations coverage needs 3 x 4 x 2 x 3 = 72 tests.

6. The other user got 404 when reading, 400 when changing and 400 when deleting the rating. I would expect 403 Forbidden (the user is logged in but not allowed) or 404 (hide that the rating exists) for all three. The rating did not change and nothing was exposed, so these three results are not a security hole. But the codes are inconsistent, and 400 means a bad request, which does not describe a permission problem, so it is a minor defect in the API design. However, see question 7: the server also accepts a made-up token, so authentication is weak overall.

7. The request with the made-up token 0000000000000000 returned status 200 and a JSON body with many ratings (ids 1 to 10 and 129650 and up, mostly 2 stars on pizza 1). Yes, this is a defect. The token was never issued by a login, yet the server accepted it and treated it as valid. It also returned ratings that belong to other users, not an empty list as a new user would get. On a real system a stranger could invent a token of the right length and read the data of other users, and if the server accepts such tokens for writing too, the stranger could create, change or delete that data. The server must check that a token was really issued by a login and answer 401 otherwise. I only read with this token and did not create or change anything.

8. An API test checks better the business rules, status codes, error messages and the data in the response, and it runs faster and breaks less often than a test through the web pages. Only a test through the web pages can check what the user sees and does, such as layout, buttons, the text on the screen, and whether the page and the API work together in the browser.