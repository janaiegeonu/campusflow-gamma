# AI learning log — complete with real experiments

> The assignment requires at least two meaningful learning cycles per engineer and at least one team cycle in which an AI suggestion was changed or rejected after evidence review. These are templates, not completed evidence. Only record questions you actually asked and experiments you actually ran.

## Engineer A — learning cycle 1

- Date / topic: `2026-10-15 — Strict integer validation in Python`
- Initial concept gap: `I wanted to understand why checking isinstance(value, int) is not
sufficient when a field must contain an integer but must reject
Boolean values.`

- Exact question asked AI: `Explain why isinstance(True, int) returns True in Python, and explain how I can validate a positive integer without accepting
Boolean values.`

- Summary of guidance received: `Python's bool type is a subclass of int, so True can pass an
isinstance(value, int) check. A strict type check can reject Boolean
values when the application requires an actual integer.
`
- Tiny unrelated experiment and code/output:

    *CODE
        ```
            values = [3, 2.5, True]

        for value in values:
            print(value, type(value) is int)

    *OUTPUT

            3 True
        2.5 False
        True False
        ```
 
- Independent verification (test, docs, observed output): `I ran the experiment and observed that type(value) is int accepts
the integer but rejects the float and Boolean. This confirmed that
the check distinguishes actual integers from Boolean values.`

- Final engineering decision and why: `I used strict integer validation for affected_users because the
CampusFlow requirements explicitly reject floats, strings, and
Boolean values. I also check that the integer is greater than zero.`

- Test added/updated to guard the learning: `test_reject_boolean_affected_users`

- Can I explain it without AI? Example explanation: `Python treats bool as a subclass of int. Therefore, True can pass
isinstance(True, int). Since affected_users must be a positive
integer and must reject Boolean values, I use strict type validation
and check that the number is greater than zero.`

## Engineer A — learning cycle 2

- Date / topic: 15 October 2026 — Safe JSON loading and validate-before-replace.

- Initial concept gap:
  I understood how to read JSON files using Python, but I wanted to understand how to prevent partially invalid data from replacing a valid ticket collection already held in memory. I also wanted to understand the difference between successfully parsing JSON and validating the application data.

- Exact question asked AI:
  "Using a simple book catalogue example, explain how to load JSON safely when one record may be invalid. Show me how to validate the complete collection before replacing the existing in-memory data. Provide a small Python experiment using only the standard library, and explain how to test that existing data remains unchanged when loading fails. Do not implement my CampusFlow project."

- Summary of guidance received:
  AI explained that successfully decoding JSON does not guarantee that the resulting data satisfies the application's business rules. The application should decode the file into temporary data, validate the complete collection, and only replace the active collection after every record passes validation. If validation fails, the existing collection should remain unchanged.

- Tiny unrelated experiment and code/output:

  I used a small book catalogue to demonstrate why validation should happen before replacing existing data.

  ```python
  current_books = [
      {"id": "B001", "title": "Python Basics"}
  ]

  incoming_books = [
      {"id": "B002", "title": "Go Basics"},
      {"id": "B003", "title": ""}
  ]

  def validate_books(books):
      for book in books:
          if not book["title"].strip():
              raise ValueError("Book title cannot be empty.")

  try:
      validate_books(incoming_books)
      current_books = incoming_books
  except ValueError as error:
      print(f"Load rejected: {error}")

  print("Current book IDs:", [
      book["id"] for book in current_books
  ])
  ```

  Expected output:

  ```text
  Load rejected: Book title cannot be empty.
  Current book IDs: ['B001']
  ```

- Independent verification (test, docs, observed output):
  After running the experiment, I verified that the invalid book collection was rejected and the original B001 record remained in memory. This demonstrated the importance of validating all incoming records before replacing the existing collection.

- Final engineering decision and why:
  I applied the validate-before-replace principle to CampusFlow storage. Ticket data must be decoded and fully validated before it can be returned for use by the application. Invalid records, duplicate ticket IDs, or inconsistent data must cause the load to fail rather than allowing invalid data into the application.

- Test added/updated to guard the learning:
  Storage tests for rejecting invalid JSON data, rejecting duplicate ticket IDs, and verifying that a failed load does not replace the existing valid ticket collection. I will confirm the exact test names against the tests in my repository.

- Can I explain it without AI? Example explanation:
  JSON decoding checks whether the file can be read as JSON, but it does not guarantee that the records satisfy CampusFlow's rules. I validate the complete collection before accepting it. If any record is invalid, the load fails before the new collection replaces the existing valid data.


## Engineer B — learning cycle 1


- Date / topic: 15 October 2026 — Case-insensitive search and combined filtering.
Initial concept gap:
  I wanted to understand how to implement case-insensitive title searching in Python and how to combine multiple optional filters without returning tickets that satisfy only some of the supplied conditions. I also wanted to ensure that searching would not modify the original ticket collection.


- Exact question asked AI:
  "Explain the difference between lower() and casefold() for case-insensitive string comparison in Python. Then use an unrelated collection of sample records to demonstrate how to combine several filters using logical AND. Show a small experiment that proves only records matching every supplied filter are returned, and explain how to verify that the original collection remains unchanged. Do not implement my CampusFlow search module."


- Summary of guidance received:
  AI explained that casefold() supports case-insensitive string comparisons, including certain Unicode characters that lower() does not handle identically. It also explained that combined search filters must use logical AND when every supplied condition is required. A separate result list should be created so that searching does not modify the original records.


- Tiny unrelated experiment and code/output:
  I used sample support tasks to demonstrate case-insensitive substring matching and combined filters.


 python
  tasks = [
      {
          "id": "T001",
          "title": "Campus WiFi failure",
          "category": "Network",
          "status": "open",
      },
      {
          "id": "T002",
          "title": "WiFi login issue",
          "category": "Network",
          "status": "resolved",
      },
      {
          "id": "T003",
          "title": "Laptop WiFi application",
          "category": "Hardware",
          "status": "open",
      },
  ]


  query = "WIFI"


  matches = [
      task for task in tasks
      if query.casefold() in task["title"].casefold()
      and task["category"] == "Network"
      and task["status"] == "open"
  ]


  print([task["id"] for task in matches])
 


  Expected output:


 text
  ['T001']
 


- Independent verification (test, docs, observed output):
  After running the experiment, I verified that only T001 matched. T002 was excluded because it was resolved, while T003 was excluded because its category was Hardware. This demonstrated that all supplied filters must match.


- Final engineering decision and why:
  I used case-insensitive substring matching for title searches and combined the optional category, status, and priority filters using AND logic. I kept search read-only by constructing a separate result list rather than modifying the original ticket collection.


- Test added/updated to guard the learning:
  Tests for case-insensitive title matching, combined filters using AND, and preserving the original ticket collection during search. Proposed test names include test_title_search_is_case_insensitive and test_combined_filters_use_and; I will confirm the actual names in my repository.


- Can I explain it without AI? Example explanation:
  Case-insensitive search allows a user to find a title without having to match its capitalization. When multiple filters are supplied, each one restricts the result. A ticket must satisfy every supplied condition to be included. Search should return matching records without changing the original ticket data.

## Engineer B — learning cycle 2


- Date / topic: 15 October 2026 — Safe percentage calculations and empty datasets.
Initial concept gap:
  I understood basic counting and arithmetic in Python, but I wanted to understand how to calculate a completion percentage safely when a collection contains no records. I also wanted to understand why reports must reflect the current state of each record instead of counting records based on their previous states.


- Exact question asked AI:
  "Explain how to calculate a completion percentage safely in Python when a collection may be empty. Use an unrelated task-tracking example to demonstrate the calculation, including the empty case. Then explain how counting records by their current status prevents double-counting when a completed task is reopened. Suggest tests that verify these behaviours without implementing my CampusFlow reports module."


- Summary of guidance received:
  AI explained that dividing by the total number of records causes a ZeroDivisionError when the total is zero. The calculation should explicitly return zero percent for an empty collection. It also explained that status reports should count each record according to its current status so that reopened tasks are no longer counted as completed.


- Tiny unrelated experiment and code/output:
  I used a small task-tracking example to demonstrate safe percentage calculation.


 python
  tasks = []


  total_tasks = len(tasks)


  completed_tasks = sum(
      1 for task in tasks
      if task["status"] == "completed"
  )


  if total_tasks == 0:
      completion_percentage = 0
  else:
      completion_percentage = (
          completed_tasks / total_tasks
      ) * 100


  print(f"Total tasks: {total_tasks}")
  print(f"Completed tasks: {completed_tasks}")
  print(f"Completion percentage: {completion_percentage:.1f}%")
 


  Expected output:


 text
  Total tasks: 0
  Completed tasks: 0
  Completion percentage: 0.0%
 


- Independent verification (test, docs, observed output):
  After running the experiment, I verified that the empty collection produced a completion percentage of 0.0% without raising a ZeroDivisionError. I also verified that the calculation counts records by their current status rather than their previous status.


- Final engineering decision and why:
  I implemented report calculations that handle empty datasets explicitly and calculate resolution percentage from the current ticket collection. Reopened tickets count according to their current status, and unresolved tickets include both open and in-progress tickets. This keeps the report consistent with the application's workflow rules.


- Test added/updated to guard the learning:
  Tests for empty reports returning zero percent, correct resolution percentage calculations, and report counts after tickets are resolved and reopened. Proposed test names include test_empty_report_returns_zero_percent and test_reopened_ticket_counted_by_current_status; I will confirm the actual names in my repository.


- Can I explain it without AI? Example explanation:
  A percentage calculation must handle an empty collection because dividing by zero raises an exception. I explicitly return zero percent when there are no records. For non-empty collections, I calculate the percentage using the number of records currently in the completed state divided by the total number of records. Reopened records are counted according to their current status.


## Team example where evidence changed or rejected an AI suggestion

- Exact suggestion and question context: `Used isinstance(value, int) to validate affected_users.`
- Evidence collected independently: `We run an experiment and observe that isinstance(True, int) returns True`
- Why the suggestion was changed/rejected/accepted with modification: `The assessment explicitly requires Boolean values to be rejected, so that validation check alone is insufficient.`
- Final code/test decision: `Used strict integer validation and add a regression test for Boolean values`

### Engineer A — Personal reflection

- What did I believe before?
  I initially thought that raising an exception was enough to
  prevent an invalid operation from changing the ticket.

- What did AI explain?
  An exception does not automatically undo mutations performed
  before the exception is raised.

- Which claim did I test?
  I tested whether a rejected assignment left the ticket unchanged.

- What did the test show?
  The state comparison confirmed whether the ticket was preserved.

- What changed in my code?
  I arranged the assignment operation to validate the request
  before changing the ticket.

- Which test protects this learning against regression?
  test_rejected_assignment_preserves_ticket