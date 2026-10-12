# DECISIONS.md  (about 150-200 words in total - plain language, your own words)

1. **Indexes.** Which indexes did you create, and which query or guarantee does each one serve? Which one did you consider and reject?
2. **Punch-in race.** Two identical punch-in requests arrive at the same instant. Walk through exactly what happens in your code.
3. **Ties.** Two employees are tied on late minutes at the cutoff of `limit`. What does your leaderboard return, and why?
4. **Headcount.** How does your department summary make sure employees with zero logs are counted?
5. **One thing you would change** if this had to serve 100x the data.

1. Indexes. Which indexes did you create, and which query or guarantee does each one serve? Which one did you consider and reject?

I created a unique index on emp_code in the employees collection to guarantee no duplicate employees are created. I also created a compound unique index on emp_code and date in the attendance_logs collection. This serves the punch-in endpoint by physically preventing duplicate records for the same employee on the same day. I considered adding an index on the history array, but rejected it because we only ever append to it, we never query inside it.

2. Punch-in race. Two identical punch-ins arrive at the same instant. Walk through exactly what happens in your code.

When both requests hit the database simultaneously, the MongoDB engine processes one insert first. When the second request attempts its insert, the compound unique index on (emp_code, date) triggers a database-level DuplicateKeyError. My FastAPI code catches this specific exception in a try/except block and safely returns a 409 Conflict to the second request, guaranteeing exactly one 201 Created response.

3. Ties. Two employees are tied on late minutes at the cutoff of limit. What does your leaderboard return, and why?

The leaderboard returns all tied employees, exceeding the limit count if necessary. I used MongoDB's $setWindowFields with the $rank function to calculate standard competition ranks first. Because the $match filter (rank <= limit) is applied sequentially after the true rank is calculated, all employees sharing the cutoff rank are correctly returned rather than arbitrarily truncating the array.

4. Headcount. How does your department summary make sure employees with zero logs are counted?

To guarantee employees with no logs are counted, I started the aggregation pipeline on the employees collection rather than the logs. I filtered for employees who joined on or before the end of the month, then used a $lookup stage (functioning like a SQL LEFT JOIN) to pull in their records from the attendance_logs collection. Using $unwind with preserveNullAndEmptyArrays: True ensures that employees without any attendance logs remain in the data stream so their headcount of 1 is accurately summed.

5. One thing you would change if this had to serve 100x the data.

At 100x scale, performing $lookup operations across collections in the analytics endpoints would become a severe bottleneck. I would introduce a pre-aggregated monthly rollup collection or utilize MongoDB Time Series collections for the raw attendance logs to drastically reduce read times and pipeline complexity.