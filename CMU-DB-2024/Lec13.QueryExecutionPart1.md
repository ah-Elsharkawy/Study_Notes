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
> - Each operator processes a batch of tuples at a time (e.g., 1024 tuples).
> - The operator processes the batch in a loop, calling Next() on its children to retrieve their tuples.
> - This model is efficient for modern hardware, as it reduces the overhead of function calls and allows for better CPU cache utilization.

> [!note] Processing Directions
>
> - Top-to-Bottom (Pull-based)
>   The parent operator (root) initiates the data flow by requesting tuples from its child operators, which in turn request data from their children, propagating down to the leaf nodes (data sources).
> - Bottom-to-Top (Push-based)
>   The leaf nodes (data sources) initiate the data flow by pushing tuples up to their parent operators, which then process and push the data further up the tree to the root operator.

> [!note] Access Methods
>
> - **Sequential Scan**: Reads all tuples from a table one by one, To speed this up, systems use data skipping like Zone Maps—pre-computed aggregates (MIN, MAX, etc.) per page—to skip pages that definitely don't match the query.
> - **Index Scan**: Uses an index to quickly locate tuples that match a specific condition, avoiding the need to scan the entire table.
> - **Multi-Index Scan**: Combines multiple indexes to satisfy a query condition, useful when no single index covers all the required attributes.

> [!note] Modification Queries
>
> Operators for INSERT, UPDATE, and DELETE are responsible for modifying both the table and its indexes while checking constraints.

> [!bug] The Halloween Problem
>
> a data integrity anomaly that occurs when an update operation changes a tuple's physical location or indexed value in a way that causes the scanning operator to encounter and process the same tuple more than once.

> [!note] How the Anomaly Occurs
>
> This problem typically happens when a query uses an index to find records and then updates the very attribute that the index is based on.
> 1. **The Scan**: A scan operator (like an index scan) identifies a tuple that matches the WHERE clause.
> 1. **The Update**: The modification operator changes a value in that tuple.
> 1. **The Relocation**: If the modified attribute is used for sorting the index, the DBMS must move the tuple to a new position in the index to maintain order.
> 1. **The Re-Discovery**: If the new position is further along in the scan's path and the tuple still matches the original search criteria, the scan operator will find the same tuple again and update it a second time.

> [!example] Example Scenario (The Salary Infinite Loop)
>
> Imagine a query intended to give a $100 bonus to everyone earning less than $1,100: `UPDATE people SET salary = salary + 100 WHERE salary < 1100`;
> - Initial State: The employee "Andy" earns $999.
> - Step 1: An index scan on salary finds Andy ($999).
> - Step 2: The system removes Andy from the index, increases his salary to $1,099, and re-inserts him into the index.
> - Step 3: Because the index is sorted by salary, Andy's record "moves" to a later position in the scan.
> - Step 4: The scan continues, finds Andy at $1,099 (which is still less than 1,100),andupdateshimagainto∗∗1,199**.,
> In extreme cases, if the update doesn't push the value outside the WHERE clause range, this can result in an infinite loop or a record being updated far more times than intended.

> [!note] The Origin
>
> The problem was discovered by IBM researchers working on System R (one of the first relational databases) on Halloween Day in 1976., They found the issue on a Friday and, rather than fixing it immediately, decided to name it after the holiday and deal with it the following week.

> [!note] The Solution (Tracking RIDs)
>
> To prevent this, the DBMS must ensure that each logical record is modified only once per query, regardless of how many times it changes physical locations.,
> - **Logical Tracking**: The system maintains a set of Record IDs (RIDs) that have already been touched by the current query.,
> - **Validation**: Before applying an update, the modification operator checks if the current RID has already been modified.
> - **Alternative Methods**: Some systems solve this by completing the entire scan first and storing the RIDs in a temporary buffer before starting any updates, though this is less scalable for massive datasets.