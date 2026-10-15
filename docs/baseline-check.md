# Baseline check — complete against the audited commit

> This file is intentionally a template. Do not record assumed results. Run the application and tests at the actual audited commit, then fill in what happened.

## Repository and commit

- Repository URL: `https://github.com/janaiegeonu/campusflow-gamma`
- Audited commit SHA: `9ed4f59b13b4bb30174928678f1cad7cc1cf7926`
- Date of baseline run: `Oct 14, 2026`
- Python version: `python3 --version`

## Commands and actual results

- Application command: `python3 main.py`
- Application result: `will display a list of operations in the terminal featuring the service helpdesk options`
- Existing test command: `python3 -m unittest discover -s tests -v`

## Three input scenarios actually tested

1. Input/scenario: Create a ticket with Network category, high urgency, and 15 affected users.
   - Expected behaviour: The ticket should be created with critical priority.
   - Actual behaviour: Code inspection indicates that the priority calculation should return critical.
   - Bug confirmed? No — execution is still required to verify the behaviour.

2. Input/scenario: Attempt to create a ticket with 0 affected users.
   - Expected behaviour: The application should reject the value and request valid input.
   - Actual behaviour: The CLI checks that the number is positive and prompts again if it is not.
   - Bug confirmed? No — code inspection indicates the validation is present, but runtime behaviour has not been verified.

3. Input/scenario: View the unresolved ticket queue containing open and in-progress tickets.
   - Expected behaviour: The original Open Work Queue should display only open tickets, ordered by priority and numeric ticket ID.
   - Actual behaviour: The existing queue function includes both open and in-progress tickets.
   - Bug confirmed? No — this is a likely requirement mismatch identified through code inspection; reproduce it by running the original application before confirming it.

