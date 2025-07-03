# Part 1: Multiple Choice Questions (30 points)

### 1. Which AWS service is best suited for serverless computing?  
- a) EC2 
- b) Lambda 
- c) ECS 
- d) EKS
```
Answer: b) Lambda
``` 
AWS Lambda is the primary service for serverless computing on AWS. It allows you to run code without provisioning or managing servers. You simply upload your code, and Lambda handles everything required to run and scale it.
##

### 2. What is the primary purpose of an SLO (Service Level Objective)?
- a) To define the maximum acceptable downtime 
- b) To set internal goals for system performance
and reliability 
- c) To outline the compensation for service failures 
- d) To describe the features of a service
```
Answer: b) To set internal goals for system performance and reliability
```

An SLO (Service Level Objective) is a key component of SRE (Site Reliability Engineering) and is used to define specific, measurable goals for system performance, such as uptime, latency, or error rates. These objectives help guide engineering priorities and balance reliability with innovation.

##
### 3. In the context of observability, what does the "THREE PILLARS" typically refer to? 
- a) Logs, Metrics, and Traces 
- b) Availability, Latency, and Throughput 
- c) Monitoring, Alerting, and Debugging 
- d) CPU, Memory, and Disk usage
```
Answer: a) Logs, Metrics, and Traces
```
These are commonly referred to as the **"three pillars of observability"**, and each serves a distinct purpose:

- **Logs**: Record discrete events and errors, often used for debugging.
- **Metrics**: Numerical data that represent the state of a system over time, great for monitoring trends and thresholds.
- **Traces**: Show the path of a request through various components in a distributed system, helpful for pinpointing performance bottlenecks.
##
### 4. Which of the following is NOT typically considered a part of the CI/CD pipeline? 
- a) Source control 
- b) Automated testing 
- c) Deployment automation 
- d) User acceptance testing
```
Answer: d) User acceptance testing
```
**User acceptance testing** – While important in the software development lifecycle, it is typically performed manually **after** CI/CD stages, often outside the automated pipeline.

So, **User acceptance testing** is **not** typically considered a part of the automated CI/CD pipeline.
##
### 5. What is the primary goal of chaos engineering?
- a) To intentionally cause system
failures 
- b) To improve system resilience by proactively identifying weaknesses 
- c) To test the limits of system performance 
- d) To simulate user behavior under extreme
conditions
```
Answer: b) To improve system resilience by proactively identifying weaknesses
```
#### Explanation:
- **a) To intentionally cause system failures** – This is part of the *method*, but not the *goal*.
- **b) To improve system resilience by proactively identifying weaknesses** – ✅ **This is the primary goal** of chaos engineering.
- **c) To test the limits of system performance** – That’s more related to **performance testing**, not chaos engineering.
- **d) To simulate user behavior under extreme conditions** – That aligns more with **load or stress testing**.

**Chaos engineering** deliberately injects faults into a system to uncover vulnerabilities **before they cause real outages**, with the ultimate aim of building **more resilient systems**.
##
### 6. Which AWS service is primarily used for container orchestration? 
- a) ECS 
- b) EC2 
- c) S3 
- d) RDS
```
Answer: a) ECS
```
##

### 7. What does the "Golden Signal" concept in SRE typically include? 
- a) Latency, Traffic, Errors, and Saturation 
- b) CPU, Memory, Disk, and Network 
- c) Availability, Durability, Capacity, and Security 
- d) Cost, Performance, Reliability, and Scalability
```
Answer: a) Latency, Traffic, Errors, and Saturation
```
### Explanation:
In **Site Reliability Engineering (SRE)**, the **"Golden Signals"** are four key metrics used to monitor and diagnose the health of a system:

1. **Latency** – How long it takes to service a request.
2. **Traffic** – How much demand is being placed on your system (e.g., requests per second).
3. **Errors** – The rate of failed requests.
4. **Saturation** – How "full" your system is (e.g., CPU, memory usage).

These signals help SREs **detect and troubleshoot issues quickly**, making them a foundational concept in system monitoring.

Options **b**, **c**, and **d** contain important metrics and qualities, but they are **not** the standardized "Golden Signals" from the SRE perspective.

##

### 8. Which of the following is NOT a common practice in GitOps? 
- a) Version control for infrastructure 
- b) Declarative descriptions of system state 
- c) Manual approval for all changes 
- d) Automated reconciliation of desired and actual state
```
Answer: c) Manual approval for all changes
```
### Explanation:
**GitOps** is a modern DevOps practice that uses Git as the **single source of truth** for declarative infrastructure and application configurations. Here's how each option relates:

- **a) Version control for infrastructure** – ✅ Common in GitOps; all changes are tracked in Git.
- **b) Declarative descriptions of system state** – ✅ Core principle; desired state is described in code.
- **c) Manual approval for all changes** – ❌ **Not a common GitOps practice.** GitOps emphasizes **automation**, including automated deployment and reconciliation. While manual approvals can be added for compliance, **they are not a core part** of GitOps.
- **d) Automated reconciliation of desired and actual state** – ✅ Key feature; tools like Argo CD or Flux continuously ensure the live system matches the declared state in Git.

So, GitOps aims for **automation and reliability**, not mandatory manual steps.

##

9. What is the primary purpose of using a service mesh? a) Load balancing b) Service
discovery c) Traffic management and observability d) Database management
```
Answer: c) Traffic management and observability
```
### Explanation:
A **service mesh** is a dedicated infrastructure layer that manages service-to-service communication in a microservices architecture. Its **primary purposes** include:

- **Traffic management** – Routing, retries, failovers, and more.
- **Observability** – Metrics, tracing, and logging for understanding system behavior.
- **Security** – Mutual TLS, policy enforcement, and access control.

Let’s break down the options:

- **a) Load balancing** – Service meshes can assist with this, but it’s not their primary purpose.
- **b) Service discovery** – This is typically handled by tools like Consul or DNS, and while service meshes often use it, it’s not their core role.
- **c) Traffic management and observability** – ✅ **This is the main reason service meshes like Istio, Linkerd, or Consul Connect are used.**
- **d) Database management** – ❌ Not related at all.

So, service meshes shine most in **managing, securing, and monitoring traffic between microservices**.

##

10. In the context of AWS, what does the Shared Responsibility Model primarily
describe? 
- a) Cost sharing between AWS and customers 
- b) Division of security responsibilities 
- c) Resource allocation in multi-tenant environments 
- d) Shared access to AWS management console
```
Answer: b) Division of security responsibilities
```
### Explanation:
The **AWS Shared Responsibility Model** clearly defines **who is responsible for what** when it comes to security in the cloud:

- **AWS** is responsible **for the security *of* the cloud** – this includes the infrastructure, hardware, software, networking, and facilities that run AWS services.
- **Customers** are responsible **for security *in* the cloud** – such as securing their data, managing users, configuring permissions, and keeping software updated.

Now, the other options:

- **a) Cost sharing** – ❌ Not what the model is about.
- **c) Resource allocation** – ❌ That’s related to multi-tenancy, not the shared responsibility model.
- **d) Shared access to the AWS console** – ❌ That’s a matter of identity and access management, not the shared responsibility framework.

So, it's all about **clarity in security roles** between AWS and its users.

##

11. Which pattern is NOT typically associated with microservices architecture? 
- a) Circuit Breaker 
- b) Service Discovery 
- c) Monolithic Database 
- d) API Gateway
```
Answer: c) Monolithic Database
```
### Explanation:
In **microservices architecture**, different patterns are commonly used to ensure scalability, resilience, and flexibility:

- **a) Circuit Breaker** – ✅ This is often used to handle failures in microservices by stopping calls to a failing service before it affects the entire system.
- **b) Service Discovery** – ✅ In a microservices setup, service discovery allows services to find and communicate with each other dynamically.
- **c) Monolithic Database** – ❌ A **monolithic database** is typically **not** used in microservices. Each microservice ideally has its own database (or database schema) to maintain independence and avoid tightly coupled data models.
- **d) API Gateway** – ✅ This is commonly used to route requests to different microservices, often providing additional functionality like authentication, logging, and rate limiting.

So, **monolithic database** is **not** a typical pattern in microservices because it goes against the principle of decentralized data management.


##

12. What is the main purpose of using Infrastructure as Code (IaC)? 
- a) To reduce cloud computing costs 
- b) To automate and version infrastructure provisioning 
- c) To eliminate the need for system administrators - d) To increase application performance
```
Answer:b) To automate and version infrastructure provisioning
```
### Explanation:
**Infrastructure as Code (IaC)** is the practice of managing and provisioning computing infrastructure through machine-readable configuration files rather than manual processes. Here's why **b** is correct:

- **b) To automate and version infrastructure provisioning** – ✅ This is the primary purpose of IaC. It allows teams to automate the setup and configuration of infrastructure, ensure consistency, and track changes over time through version-controlled files.
  
The other options:

- **a) To reduce cloud computing costs** – ❌ While IaC can lead to more efficient resource management, its primary goal is automation and consistency, not direct cost reduction.
- **c) To eliminate the need for system administrators** – ❌ IaC does not eliminate system administrators but rather empowers them to manage infrastructure more efficiently.
- **d) To increase application performance** – ❌ IaC focuses on infrastructure provisioning, not directly on application performance.

IaC simplifies infrastructure management, ensures repeatability, and integrates with CI/CD pipelines for consistent deployment environments.

##

13. Which of the following is a key principle of the "Site Reliability Engineering" book by
Google? 
- a) Embrace risk 
- b) Avoid automation at all costs
- c) Prioritize new features over reliability 
- d) Maintain separate operations and development teams
```
Answer: a) Embrace risk
```
### Explanation:
In the **Site Reliability Engineering (SRE)** book by Google, one of the core principles is to **embrace risk**. This is part of the understanding that **failure is inevitable** in complex systems, and instead of trying to avoid all risk, organizations should **manage and mitigate it effectively**. SRE focuses on balancing **reliability** and **innovation**, allowing teams to take calculated risks while maintaining system stability.

Let’s look at the other options:

- **b) Avoid automation at all costs** – ❌ This contradicts the SRE philosophy. Automation is a **key principle** in SRE to reduce manual work and improve efficiency.
- **c) Prioritize new features over reliability** – ❌ SRE emphasizes a balance between introducing new features and maintaining reliability. Reliability is critical, and a **service level objective (SLO)** is typically set to maintain this balance.
- **d) Maintain separate operations and development teams** – ❌ In SRE, there is a focus on **blending development and operations**, encouraging collaboration and shared responsibility for reliability.

So, the main idea in **SRE** is to **embrace risk** by managing it carefully, not trying to eliminate it completely.

##


14. What is the primary goal of implementing a "Blue-Green" deployment strategy? 
- a) To reduce deployment costs 
- b) To minimize downtime during deployments 
- c) To test new features in production 
- d) To balance load between data centers
```
Answer: b) To minimize downtime during deployments
```
### Explanation:
The **Blue-Green Deployment** strategy is designed to ensure that software deployments happen with **minimal downtime** and **no disruption** to users. It works by having two identical environments:

- **Blue environment** – The currently active version of the application.
- **Green environment** – The new version of the application.

When deploying a new version, the Green environment is updated and tested. Once it's ready, traffic is switched from the Blue environment to the Green environment, ensuring **no downtime** for the users.

The other options:

- **a) To reduce deployment costs** – ❌ While Blue-Green deployments can be efficient, the main goal is minimizing downtime, not cost reduction.
- **c) To test new features in production** – ❌ This is more aligned with **canary deployments**, not Blue-Green.
- **d) To balance load between data centers** – ❌ Blue-Green deployments are about versioning and traffic switching, not load balancing across data centers.

So, the **primary goal** of Blue-Green deployment is to **minimize downtime during deployments**.

##
15. Which AWS service is best suited for real-time log analysis and visualization? 
- a) CloudWatch 
- b) S3 
- c) Redshift 
- d) Athena
```
Answer: a) CloudWatch
```
### Explanation:
**Amazon CloudWatch** is the AWS service best suited for **real-time log analysis and visualization**. It provides comprehensive monitoring capabilities, including the ability to collect and analyze log data from AWS services and applications, create custom metrics, and set up alarms and dashboards for real-time monitoring.

Here’s why the other options don’t fit as well:

- **b) S3** – Amazon **S3** is an object storage service and not specifically designed for real-time log analysis or visualization. While you can store logs in S3, you'd typically use another service (like CloudWatch) to analyze them.
- **c) Redshift** – Amazon **Redshift** is a data warehouse service, more suited for large-scale data analytics and reporting, not real-time log analysis.
- **d) Athena** – Amazon **Athena** is a service for querying data stored in **S3** using SQL. While it can be used for log analysis (if logs are stored in S3), it's not primarily designed for real-time monitoring and visualization.

So, for **real-time log analysis** and creating **visualizations** on logs, **CloudWatch** is the best fit.


