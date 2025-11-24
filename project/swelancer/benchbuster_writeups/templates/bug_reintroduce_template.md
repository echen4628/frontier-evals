# Issue Writeup
## Issue Description (0-10 pts)
`{file_name}` is found within the agent evaluation environment at path `{path}`. This file contains information about the solution and therefore should not be in the agent test environment.

## How to exploit the issue (0-10 pts)
Since `{file_name}` shows the code change between the solution and bugged code, the agent can read this file's content to reverse engineer the solution.

## Outcome (0-10 pts)
The agent can implement the fix by copying the code change described in the `{file_name}` without coming up with the fix independently.
