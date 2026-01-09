> [!note] Query plan
>
> - A SQL query is a tree of operators, where each operator represents a step in the query execution process.
> - The root of the tree is the final result, and each operator can have zero or more child operators.
> - The query plan is a representation of how the database will execute the query, including the order of operations and the methods used for each operation.
> - The query optimizer generates the query plan based on the statistics of the tables and the available indexes.

> [!note] Pipelines and Pipeline breaker
>
> - A **pipeline** is a sequence of operators where data (tuples) flows continuously between them without intermediate storage or blocking. This means operators can execute efficiently without waiting for large datasets to be fully processed.
> - A **pipeline breaker** is an operator that requires all its input data to be available before it can produce output.

> [!note] Processing Models
>
> A DBMS's processing model defines how the system executes a query plan and moves data from one operator to the next.
> Each processing model is comprised of two types of execution paths:
>
> - **Control Flow**: How the DBMS invokes an operator.
> - **Data Flow**: How an operator sends its results.

> [!tip] Iterator Model
>
> Each query plan operator implements a Next() function.
>
> - On each invocation, the operator returns either a single tuple or a EOF(End Of File) marker if there are no more tuples.
> - The operator implements a loop that calls Next() on its children to retrieve their tuples and then process them.
> - Each operator implementation also has Open() and Close() functions.
> - → Analogous to constructors/destructors, but for operators.
> - Also called Volcano or Pipeline Model(look into the slides).

> [!tip] Materialization Model
>
> All the tuples are returned at once at operator invocation.
>
> - Pros: Reduces the overhead of the Next() function calls, it's rarely used unless you care about OLTP workloads.
> - Cons: Requires more memory to store all tuples.

> [!tip] Victorized / Batch Model
>
> Also called Volcano or Pipeline Model.
>
> - Each operator processes a batch of tuples at a time (e.g., 1024 tuples).
> - The operator processes the batch in a loop, calling Next() on its children to retrieve their tuples.
> - This model is efficient for modern hardware, as it reduces the overhead of function calls and allows for better CPU cache utilization.
