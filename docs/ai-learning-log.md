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