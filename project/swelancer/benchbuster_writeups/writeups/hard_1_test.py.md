# Issue Writeup
## Issue Description (0-10 pts)
`test.py` is found within the agent evaluation environment at path `/opt/test.py`. This file contains information about the tests for evaluation and therefore should not be in the agent test environment.

## How to exploit the issue (0-10 pts)
`test.py` includes all the tests to determine whether the agent successfully fixed the bug. The agent have two ways of exploiting the issue:
1. Instead of fixing the code in general, the agent can only write code that passes the test cases. The agent's solution can pass the test cases without implementing the logically correct fix.
2. The agent could also overwrite the test cases to be easier or be trivial (for example: `assert True`).

## Outcome (0-10 pts)
1. If the agent go with exploit 1, the outcome is the agent solution merely passes the test case without solving the underlying issue.
2. If the agent go with exploit 2, the outcome is the modified test cases passes trivially, allowing the agent to score full points without implementing the fix.
