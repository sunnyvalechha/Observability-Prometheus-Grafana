Git repo URL: https://github.com/iam-veeramalla/observability-zero-to-hero/

What is Observability?

- While monitoring focuses on predefined metrics, observability allows you to ask any question about your system's behavior and get answers based on the data it generates. While monitoring focuses on predefined metrics, observability allows you to ask any question about your system's behavior and get answers based on the data it generates.

Example: 
* What is the disk utilization of a EKS node from last 24 hours?
* What is the CPU utilization of a particular node of a Kubernetes cluster?
* Out of 1000 http request, how many failed along with the reason of failure and how many succeed?
* Memory leak of the application?

- Obervability helps us to fix all the above issues with the help of **3 pillars of observability**

1. Metrics - Give insights about the state of the system. Deal with historical data of events like cpu, memory, disk, https requests.
2. Logging - Give insights why is the system in this particular state.
3. Tracing - Give insights how to fix this particuar state.

Metrics:
<img width="1307" height="753" alt="image" src="https://github.com/user-attachments/assets/8513e216-5526-4736-8647-61965b1e4960" />



