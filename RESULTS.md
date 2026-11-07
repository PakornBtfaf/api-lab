# API lab results

## Part 0

- Step 0.1, the number of pizza names:18
- Step 0.3, status code of the request without a token:401
- Step 0.4, the name of your pizza: A Quiet Americana
- Step 0.12, results of the three runs without a session (optional):failed, failed, failed
- Step 0.16, blocks of "how many of the three tools are excluded":none (0), one (1), two (2), all three (3)

## Defect report

- Title: Excluding all three tools returns 500 with an empty body
- Request: POST /api/pizza with header "Authorization: token <token>" and body {"excludedTools": ["Knife", "Pizza cutter", "Scissors"]}
- Expected result: status 400 and a JSON error message
- Actual result: status 500 and an empty body
- Severity: Medium. The server fails on valid input and the app gets no message to show the customer.

## Part A

- Step A4, status codes when the other user reads, changes and deletes:

## Part B

1.
2.
3.
4.
5.
6.
7.
8.