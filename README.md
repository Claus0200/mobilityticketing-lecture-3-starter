# MobilityTicketing: Lecture 3 starter

Submitted commit: [8c6b32c](https://github.com/Claus0200/mobilityticketing-lecture-3-starter/commit/8c6b32c1df9aa788c09147f4089f772e022f436f)
Setup and reset instructions: [link](setup.md)

## Where to find the work
Lecture 1: model, workload map and queries: [link](https://github.com/Claus0200/mobilityticketing-lecture-1-starter)
Lecture 2: constraints and tests: [link](https://github.com/Claus0200/mobilityticketing-lecture-2-starter)
Lecture 3: reporting experiment and comparison: [link](https://github.com/Claus0200/mobilityticketing-lecture-3-starter)
Lecture 4: migration stages and verification: [link](https://github.com/Claus0200/mobilityticketing-lecture-4-starter)

## Two decisions worth discussing

### SQL function for the revenue report

I chose to use a SQL function to calculate the captured revenue for an operator on a specific day. The alternative was to use a materialized view with precomputed results.

The reason for choosing the SQL function is that the current MobilityTicketing report needs the latest data when the report is requested. The function calculates the result directly from the authoritative payment and ticket data, so there is no need to refresh a stored result. It is also relatively simple to maintain for this reporting requirement.

The relevant [evidence](database/postgres/migrations/020_reporting_function.sql) is captured_revenue_for_day function and the comparison of the different reporting approaches in Lecture 3

### Trigger maintained summary

I chose not to use a trigger-maintained summary table for the current report. The alternative was to maintain daily revenue totals automatically through database triggers whenever payments change.

The reason for this choice is that a trigger would make payment writes responsible for keeping the reporting summary correct. This add another dependency and requires handling cases such as status change Failed, Captured, Refunded. For the current MobilityTracking report, calculating the result from the authoritative data is much simpler.

The relevant [evidence](database/postgres/evidence/responsibility_matrix.md) is responsibility matrix, where you can see the benefits of SQL function.


## One limitation or open question
One limitation is that the reporting experiment was tested with the current MobilityTicketing dataset and workload, so we have not established at what data size or reporting frequency a SQL function would become too expensive.

The [comparison](database/postgres/evidence/responsibility_matrix.md) shows that the SQL function is suitable for the current case, while a materialized view or stored summary could become more useful if the report needed to be generated very frequently or over a much larger dataset.

The next step would be to test the report with a larger dataset and measure the query time before deciding whether a precomputed approach is necessary.