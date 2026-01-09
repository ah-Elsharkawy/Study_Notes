# Design Data Intensive Applications

> [!note] Chapter 1: Reliable, Scalable, and Maintainable Applications
>
> This chapter introduces the fundamental concepts of data-intensive applications. It explains the three concerns that such applications need to address: reliability, scalability, and maintainability. It also introduces the three design principles that help to address these concerns: data modeling, data storage, and data processing.

## 1.1 Reliability

systems should continue to work correctly even when things go wrong.

- **Hardware Faults**: Hardware will fail. Software should be designed to expect and handle these failures.
- **Software Errors**: Software will have bugs. Software should be designed to expect and handle these errors.
- **Human Errors**: Humans will make mistakes. Software should be designed to expect and handle these mistakes.

> [!question] What is the difference between a fault and a failure?
>
> A fault is usually defined as one component of the system deviating from its spec, whereas a failure is when the system as a whole stops providing the required service to the user.

## 1.2 Scalability

system's ability to cope with increased load.

- **Describing Load**: Scalability is the term we use to describe a system’s ability to cope with increased load.
- **Describing Performance**: Performance is the term we use to describe the response time of a system under a particular load.

> [!question] response time vs latency vs service time
>
> - **Response Time**: The time between a client sending a request and receiving a response.
> - **Latency**: The duration that a request is waiting to be handled—during which it is latent, awaiting service.
> - **Service Time**: The time it takes to handle a request.

> [!question] How can we measeure the performance by response time?
>
> - **Percentiles**: The median (50th percentile) is the value below which 50% of the values fall. The 95th percentile is the value below which 95% of the values fall.
