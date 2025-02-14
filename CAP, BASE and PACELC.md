---
tags:
  - Distributed-Systems
  - TODO
---

# The CAP Theorem

Three properties related to the CAP theorem:

- **Consistency**: Every read request returns the latest write or an error
- **Availability**: Every request receives a non-error response, even if it may not contain the most up-to-date data
- **Partition tolerance**: The system still works if some nodes can't communicate due to network issues. Only network failures are considered in the theorem, not node failures

The CAP theorem says you can only choose two of these properties:

- **CP**: The system ensures up-to-date reads even during network issues, but some requests may fail
- **AP**: The system stays available during network issues but may return outdated data, breaking consistency
- **CA**: The system ensures both consistency and availability. In practice, partition tolerance is essential because network instability is inevitable in distributed systems. Therefore, **CA systems are not viable**

# The PACELC Theorem

The PACELC theorem expands the CAP theorem with two key questions:

- In the case of a network partition (**P**), should we favor **Availability** (**A**) or **Consistency** (**C**)?
- But else, in the absence of partition (**E**), should we favor **Latency** (**L**) or **Consistency** (**C**)?

Systems can be **PA** or **PC**, and they can be **EL** or **EC**

Latency refers to the maximum time a request should take. The CAP theorem doesn't account for latency, so a system could be considered available even if it takes too long to respond. PACELC addresses this by adding latency to the framework, making it more complete by considering both network partitions and response times

# The BASE Theorem

#TODO

# References

- [The CAP Theorem - by Teiva Harsanyi - The Coder Cafe](https://www.thecoder.cafe/p/cap)
- [The PACELC Theorem - by Teiva Harsanyi - The Coder Cafe](https://www.thecoder.cafe/p/pacelc)
