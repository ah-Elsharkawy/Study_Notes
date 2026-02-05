# Design Data Intensive Applications

## Chapter 1: Reliable, Scalable, and Maintainable Applications
![[Ch1_Overview.png]]

> [!TIP] **Important**
> 
> Read the highlighted parts of the book, It has more info.

Modern applications are increasingly data-intensive, meaning their primary challenges involve the quantity, complexity, or speed of change of data, rather than raw CPU power. These applications are typically built using standard functional blocks, including:

- **Databases**: For storing data to be found again later.
- **Caches**: For remembering the results of expensive operations to speed up reads.
- **Search Indexes**: For allowing users to filter or search data by keyword.
- **Stream Processing**: For sending asynchronous messages to other processes.
- **Batch Processing**: For periodically crunching large amounts of accumulated data.

## Thinking About Data Systems

The traditional boundaries between databases, caches, and message queues are becoming blurred as new tools are optimized for diverse use cases; for example, Redis can be used as both a datastore and a message queue. Furthermore, many applications now have requirements so demanding that a single tool can no longer meet all needs, requiring the work to be broken down into tasks performed across multiple tools stitched together by application code.
When combining tools in this way—such as keeping a cache or search index in sync with a primary database—the developer essentially becomes a data system designer. This design process must address tricky questions regarding data correctness, performance during component degradation, and scaling for increased load. To navigate these challenges, the focus is placed on three fundamental concerns:

1. **Reliability**: The system should work correctly even in the face of adversity.
2. **Scalability**: The system should have reasonable ways of dealing with growth in data or traffic volume.
3. **Maintainability**: Many different people should be able to work on the system productively over time.

## 1.1 Reliability

Reliability is roughly defined as `continuing to work correctly, even when things go wrong`. This includes performing the function expected by the user, tolerating user mistakes, maintaining performance under expected load, and preventing unauthorized access.

### Faults vs. Failures

• **Faults**: One component of the system deviating from its specification.
• **Failures**: When the system as a whole stops providing the required service to the user.
• **Fault-Tolerant (Resilient)**: Systems that anticipate and cope with specific types of faults to prevent them from causing a total failure.

Interestingly, it can be beneficial to deliberately trigger faults (such as randomly killing processes) to ensure that the fault-tolerance machinery is continually exercised and tested, as seen in the Netflix Chaos Monkey.

### Causes of Reliability Issues

1. **Hardware Faults**: Hard disks crash, RAM becomes faulty, and power grids have blackouts. For large clusters of machines, hardware redundancy (RAID, dual power supplies, generators) is often insufficient, leading to a move toward software fault-tolerance techniques that tolerate the loss of entire machines.
2. **Software Errors**: These are systematic errors that are often correlated across nodes and harder to anticipate than random hardware faults. Examples include software bugs causing crashes on specific inputs, runaway processes exhausting resources, or cascading failures where a small fault triggers further faults.
3. **Human Errors**: Configuration errors by operators are a leading cause of outages. To combat this, systems should be designed to minimize opportunities for error through good abstractions, provide sandbox environments for safe experimentation, and allow for quick recovery through easy rollbacks.

### The Importance of Reliability

Reliability is not just for critical infrastructure; even mundane business or personal applications have a responsibility to their users to prevent data corruption or loss of revenue and reputation. While developers may sometimes sacrifice reliability to reduce costs during early prototyping, they should be conscious of exactly when they are cutting these corners.

## 1.2 Scalability

Scalability is the ability of a system to cope with increased load. It is not a single property but a set of questions regarding how to maintain performance when load parameters increase.

- **Describing Load**: Load is described via **load parameters**, which are specific to the application. For example, Twitter’s scalability is defined by its "fan-out" load—the number of followers per user—which determines how many writes occur when a tweet is posted.
- **Describing Performance**: In service-oriented systems, performance is usually measured by **response time**, specifically using **percentiles** (p50,p95,p99). Percentiles are superior to arithmetic means because they accurately represent the "tail latency" experienced by users.
- **Coping with Load**: The author discusses the dichotomy between **scaling up** (vertical scaling to more powerful machines) and **scaling out** (horizontal scaling across multiple machines). While distributed systems are becoming the future default, they introduce significant complexity for stateful data systems.

> [!question] response time vs latency vs service time
>
> - **Response Time**: The time between a client sending a request and receiving a response.
> - **Latency**: The duration that a request is waiting to be handled—during which it is latent, awaiting service.
> - **Service Time**: The time it takes to handle a request.

> [!question] How can we measeure the performance by response time?
>
> - **Percentiles**: The median (50th percentile) is the value below which 50% of the values fall. The 95th percentile is the value below which 95% of the values fall.


## 1.3 Maintainability

Since the majority of software cost occurs during its ongoing maintenance, systems should be designed to minimize pain for engineering and operations teams.

• **Operability**: This means making it easy for operations teams to keep the system running smoothly by providing visibility into health and supporting automation.

• **Simplicity**: The goal is to remove **accidental complexity**—complexity that is not inherent in the problem but arises from the implementation.

• **Evolvability**: Also known as extensibility or plasticity, this is the ease with which a system can be modified to meet unanticipated changes in requirements.