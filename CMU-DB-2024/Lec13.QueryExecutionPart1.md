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
>   The parent operator (root) initiates the data flow by requesting tuples from its child operators, which in turn request data from their children, propagating down to the leaf nodes (data sources), it's the most common model used in DBMS.
> - Bottom-to-Top (Push-based)
>   The leaf nodes (data sources) initiate the data flow by pushing tuples up to their parent operators, which then process and push the data further up the tree to the root operator, it's less common but can be more efficient for certain workloads, especially in streaming systems as we control where the data is held in memory either cpu registers or cache.

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

> [!tip] Expression Evaluation
>
> is the process by which a DBMS determines whether a tuple satisfies a specific predicate (like a WHERE clause) or calculates values for projections (like SELECT ID + 1).
> Expressions are evaluated using an expression tree, where each node represents an operation (e.g., addition, comparison) or a value (e.g., column reference, constant).

> [!bug] The Problem: Expression Trees (branching costs)
>
> When a SQL query is parsed, the DBMS logically represents the predicates as an expression tree.
> - **Structure**:  Each node in the tree represents an expression type, such as comparison operators (=, <, >), arithmetic (+, -), constant values, or attribute references (table columns).
> - **Evaluation Method**: The system evaluates these by performing a depth-first traversal. For every single tuple, the engine must look up the table schema, fetch the column values, and walk through the tree nodes to compute the result.
> - **Performance Issue**: This "interpretive" approach is very slow for modern CPUs. Following pointers down a tree creates high `branching costs` and indirection, preventing the CPU from executing instructions sequentially, which is its fastest mode of operation.

> [!note] The Solution: Just-In-Time (JIT) Compilation
>
> To solve tree traversal overhead, modern systems use JIT Compilation to convert the expression tree into machine code at runtime.
> - **Code Generation**: The DBMS generates optimized machine code using frameworks like LLVM, which can produce highly efficient code tailored to the specific query.
> - **Execution**: This machine code is then executed directly by the CPU, eliminating the overhead of tree traversal and interpretation.
> - **PostgreSQL Example**: PostgreSQL can JIT-compile WHERE clauses. In a query with 50 million tuples, JIT might take 200ms to compile the code but save 600ms in execution time, resulting in a net win.
> - **Trade-offs**: JIT is not always better. For simple queries that only access one or two tuples (OLTP), the time spent compiling the code might be longer than the time it takes to just run the query using the standard tree traversal.
> - **Vectorization Hybrid**: Some systems, like Snowflake, avoid full JIT and instead use pre-compiled macros. They process a batch (vector) of tuples at once, which spreads the overhead of a function call across 1,024 tuples instead of just one.

> [!note] Optimization Techniques
>
> Whether using JIT or standard traversal, the DBMS uses several logic optimizations to minimize work:
> - **Constant Folding**: Pre-computing expressions that involve only constants at compile time rather than runtime, The DBMS identifies parts of an expression that consist only of constant values and computes them once before execution begins.
> - - **Example**:  For WHERE UPPER(name) = UPPER('wutang'), the system pre-computes UPPER('wutang') into 'WUTANG'. This prevents the CPU from running the "uppercase" function millions of times for the same constant string.
> - **Common Sub-expression Elimination**: Identifying and reusing the results of expressions that are computed multiple times within a query, The system looks for identical sub-expressions that appear multiple times in the same tree and ensures they are only calculated once per tuple.
> - - **Example**: In WHERE (a + b) > 10 AND (a + b) < 20, the expression (a + b) is computed once and reused for both comparisons, reducing redundant calculations.

> [!note] Summary Table: Expression Evaluation
>
|Feature|Standard Tree Traversal|JIT Compilation|
|---|---|---|
|**Model**|Interpretive (depth-first)|Compiled (Machine Code)|
|**Speed**|Slow due to branching/pointers|Fast (direct CPU instructions)|
|**Workload**|Simple/Short Queries (OLTP)|Complex/Large Scans (OLAP)|
|**Flexibility**|High (handles any tree)|High, but adds compile-time lag|