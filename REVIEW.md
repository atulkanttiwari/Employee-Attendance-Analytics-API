# REVIEW

| # | Where (function) | What is wrong                                                | How you'd notice it (test or symptom)            | How you fixed it                                                               |
|---|------------------|--------------------------------------------------------------|--------------------------------------------------|--------------------------------------------------------------------------------|
| 1 | `list_employees` | Pagination skip skips page 1 completely; total is wrong.     | Requesting page 1 skips the first 20 employees.  | Changed skip math to `(page - 1) * page_size` and passed `q` to total.         |
| 2 | `create_employee`| Susceptible to race conditions (check-then-insert pattern).  | Concurrent requests with same emp_code succeed.  | Added startup event to create unique MongoDB indexes & caught DuplicateKeyError|
| 3 | `list_attendance`| Loads entire collection into memory; leaks "_id" field.      | Slow performance at 100k records; schema error.  | Used MongoDB `.sort()`, `.skip()`, `.limit()`, and excluded `_id`.             |
| 4 | `punch_in`       | Race condition; 500 error if missing emp; naive datetimes.   | Crashes on bad emp_code; concurrent inserts fail.| Caught DuplicateKeyError; added 404 check; enforced UTC & R1 logic.            |