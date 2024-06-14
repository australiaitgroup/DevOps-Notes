# High Availability with Kubernetes

## Table of Contents

- [High Availability](#high-availability)
- [The Basic Elements of High Availability](#the-basic-elements-of-high-availability)
- [Technical Components for High Availability](#technical-components-for-high-availability)
  - [Frontend Related](#frontend-related)
  - [Backend Related](#backend-related)
- [High Availability Checklist](#high-availability-checklist)
- [SRE Fundamentals: SLIs, SLAs, and SLOs](#sre-fundamentals-slis-slas-and-slos)
- [Auto Scaling Basics](#auto-scaling-basics)
  - [Kubernetes Autoscaling](#kubernetes-autoscaling)
- [Chaos Engineering](#chaos-engineering)
  - [Planning Your First Chaos Experiments](#planning-your-first-chaos-experiments)
- [Hands-on Exercises](#hands-on-exercises)
- [Reading Materials](#reading-materials)

## High Availability

High availability is a quality of computing infrastructure that allows it to continue functioning, even when some of its components fail. This is important for mission-critical systems that cannot tolerate interruption in service, and any downtime can cause damage or result in financial loss.

Highly available systems guarantee a certain percentage of uptime—for example, a system that has 99.9% uptime will be down only 0.1% of the time—0.365 days or 8.76 hours per year. The number of “nines” is commonly used to indicate the degree of high availability. For example, “five nines” indicates a system that is up 99.999% of the time.

## The Basic Elements of High Availability

The following three elements are essential to a highly available system:

- **Redundancy**: Ensuring that any elements critical to system operations have an additional, redundant component that can take over in case of failure. Redundancy is essential to avoid single points of failure.
- **Monitoring**: Collecting data from a running system and detecting when a component fails or stops responding.
- **Failover**: A mechanism that can switch automatically from the currently active component to a redundant component if monitoring shows a failure of the active component.

## Technical Components for High Availability

### Frontend Related

- **DNS**: Domain Name Server maps URLs to IP addresses, crucial for external customers to access your service. In AWS, you can configure multiple DNS resources in case of emergency.
  - [AWS DNS Failover Configuration](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover-configuring.html)
- **CDN**: Content Delivery Network holds frontend static files and helps deliver your content quickly to customers. This service typically has good high availability guarantees.
  - [AWS CDN Failover](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/high_availability_origin_failover.html)

### Backend Related

- **Load Balancing**: A load balancer manages traffic, routing it between multiple systems that can serve that traffic. The load balancer can detect when a target system fails and redirect traffic to another available system, thus implementing monitoring and failover.
  - [AWS Load Balancer for Blue/Green Deployment](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-create-loadbalancer-bluegreen.html)
- **Clustering**: A cluster contains several nodes that serve a similar purpose. Users typically access and view the entire cluster as one unit. Each node in the cluster can potentially failover to another node if failure occurs. Replication within the cluster creates redundancy between nodes.
  - Considerations in AWS:
    - Multiple EC2 instances + autoscaling policy
    - Multiple caches
    - Persistent queues
    - Multi-AZ (Availability Zone)
    - Multi-region
- **Data Backup and Recovery**: A system that automatically backs up data to a secondary location and recovers it back to the source. This setup can be used to ensure redundancy and failover.
  - [AWS RDS Multi-AZ](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)

### Open Question to Discuss

Although there are many things you can do to improve your product, you may not be able to achieve everything in a short term or with limited resources. So, which one should we improve first?

## High Availability Checklist

### 1. Define Availability Requirements

- Identify the cloud workloads that require high availability and their usage patterns.
- Define your availability metrics, such as:
  - Percentage of Uptime
  - Mean Time to Recovery (MTTR)
  - Mean Time Between Failures (MTBF)
  - Recovery Time Objective (RTO)
  - Recovery Point Objective (RPO)

### 2. Plan Your High Availability Architecture

- **Failure Mode Analysis (FMA)**: Identify potential failure types, their implications, and recovery strategies. Determine the required redundancy for each component to avoid single points of failure and use load balancing to distribute requests between redundant components.
- **Consider Costs**: Each redundant layer effectively doubles your cloud costs. Ensure you have licenses and infrastructure to support additional redundant instances.
- **Consider Resiliency**: Ensure that systems can fail gracefully and restore operations without disruption. Use isolation for critical resources, compensating transactions, and asynchronous operations.
- **Replicate Data**: Ensure application data replication supports your redundancy strategy and RTO and RPO. Failing to replicate fresh data to the redundant component before failure makes failover or recovery impossible.
- **Document Everything**: Document steps for failover to a redundant component and recovery or “failback” to the original component. Instructions should be short and clear for emergency use.

### 3. Perform End-to-End Testing

- Use fault injection testing to simulate realistic failure conditions and measure recovery time. Test both failover and failback.
- Additional tests:
  - **Identify Failures Under Load**: Perform load testing until a system fails and observe how failure mechanisms behave.
  - **Run Disaster Recovery Exercises**: Conduct planned or unplanned experiments to test the disaster recovery plan.
  - **Test Health Probes/Checks**: Ensure that health probes/checks correctly identify component failure.
  - **Test Monitoring Systems**: Periodically verify that monitoring data is accurate to detect failures promptly.

### 4. Deploy Applications Consistently

- Any change can result in failure. Automated, consistent deployment processes minimize errors and failures and help in recovery.
- Consider availability in your release process by designing for rolling updates to minimize service disruption. Use blue-green releases or similar strategies.
- Plan for rollback with an automated process to restore systems to a previous working version.

### 5. Monitor Application Health

- **Use Probes and Check Functions**: Detect failures in time using health probes/checks and check functions for fresh service availability data.
- **Watch Degrading Health Metrics**: Degrading metrics can indicate impending failure. Create an early warning system by identifying key health indicators and alerting operators at problematic thresholds.
- **Leverage Logging and Auditing**: Use extensive logging and auditing capabilities to monitor service limits and act before reaching overage.

[Azure High Availability: Basic Concepts and a Checklist](https://cloud.netapp.com/blog/azure-high-availability-basic-concepts-and-a-checklist)

## SRE Fundamentals: SLIs, SLAs, and SLOs

### 1. Service-Level Objective (SLO)

- **Background**: The goal of SRE is to ensure the service is up and running 24/7. Availability defines whether a system can fulfill its intended function at a point in time. Historical availability measurement describes the probability that your system will perform as expected in the future.
- **Quantify Availability**: Set a numerical target for availability, known as SLO.

### 2. Service-Level Agreement (SLA)

- An SLA involves a promise to someone using your service that its availability SLO will meet a certain level over a specific period. If it fails, a penalty is paid, such as a partial refund or additional subscription time. The concept is that going out of SLO hurts the service team, pushing them to stay within SLO.

  Examples:

  - [AWS SLA](https://aws.amazon.com/compute/sla/)
  - [Google Cloud SLA](https://cloud.google.com/terms/sla/)
  - [Jira SLA](https://www.atlassian.com/legal/sla/service-credits)

### 3. Service-Level Indicator (SLI)

- Precise measurements on key metrics help decide the service availability percentage. Falling below or exceeding SLOs too much indicates a problem.

### 4. Setting SLI, SLO, and SLA

- Identify system boundaries within the platform.
- Identify customer-facing capabilities at each system boundary.
- Articulate a plain-language definition of what it means for each capability to be available.
- Define one or more SLIs for that definition and start measuring to get a baseline.
- Define an SLO for each capability and track performance.
- Iterate and refine the system and SLOs over time.
- Leave SLA setting to legal or management teams.

### Managing Risk

[Embracing Risk](https://landing.google.com/sre/sre-book/chapters/embracing-risk/)

- **Redundant Resources Cost**: Includes the cost of redundant equipment for offline routine or unforeseen maintenance and parity code blocks for data durability.
- **Opportunity Cost**: Includes the cost of engineering resources diverted from new features to risk reduction.

In SRE, manage service reliability by managing risk. Align service risk with the business's risk tolerance. Explicitly align the risk taken by a given service with what the business is willing to bear, aiming for a balance between reliability and cost.

### Measuring Service Risk

- **Time-Based Availability**:

  ```markdown
  availability = uptime / (uptime + downtime)
  ```

  Calculate acceptable downtime to reach a specific availability target.

- **Aggregate Availability**:

  ```markdown
  availability = successful requests / total requests
  ```

  Define availability in terms of request success rate for both serving and non-serving systems.

### Error Budget and Cost

- Key factors include the incremental increase in revenue and the cost of achieving an additional nine of availability. For example:

  ```markdown
  Proposed improvement in availability target: 99.9% → 99.99%
  Proposed increase in availability: 0.09%
  Service revenue: $1M
  Value of improved availability: $1M \* 0.0009 = $900
  ```

## Auto Scaling Basics

### What is Auto Scaling?

Auto Scaling ensures you have the correct number of resources to handle application load.

### Kubernetes Autoscaling

#### Three Autoscaling Methods for Kubernetes

1. **Horizontal Pod Autoscaler**: Automatically increases or decreases the number of running pods as application usage changes. Defines target metrics (e.g., CPU, memory utilization, number of queued messages) and desired max/min replicas.
2. **Vertical Pod Autoscaler**: Adds more resources (CPU or memory) to an existing machine. Useful for services where horizontal scaling is not possible or ideal.
3. **Cluster Autoscaler**: Increases or decreases the number of nodes in the cluster as pod load changes.

### Solution

- **Orchestration Solutions**:

  - Kubernetes
  - Swarm
  - AWS Auto Scaling

- **Best Practices**:
  - [Best Practices for Container Orchestration](https://techbeacon.com/enterprise-it/5-best-practices-container-orchestration-it-production)

## Chaos Engineering

### Preventive Medicine

Chaos Engineering is a disciplined approach to identifying failures before they become outages. By testing system responses under stress, you can identify and fix failures preemptively.

### Brief History

- **2010**: Netflix's Eng Tools team created Chaos Monkey to ensure resilience during the move to AWS cloud infrastructure.
- **2011**: Simian Army added more failure injection modes for broader testing.
- **2012**: Netflix shared Chaos Monkey source code on GitHub.
- **2014**: Netflix introduced the Chaos Engineer role and announced Failure Injection Testing (FIT) for more controlled failure injection.

### Principles of Chaos Engineering

1. Form a hypothesis about system behavior during failures.
2. Design the smallest possible experiment to test the hypothesis.
3. Measure the impact of the failure at each step to understand the system's real-world behavior.

### Companies Practicing Chaos Engineering

Large tech companies like Twilio, Netflix, LinkedIn, Facebook, Google, Microsoft, Amazon, and more. Traditional industries like banking and finance also use Chaos Engineering to reduce incident counts.

### Planning Your First Chaos Experiments

#### Steps

1. **What Could Go Wrong?**: Identify potential weaknesses and expected outcomes.
2. **Creating a Hypothesis**: Discuss expected outcomes as a team.
3. **Measuring the Impact**: Measure system availability and durability. Monitor key performance metrics and system resources.
4. **Have a Rollback Plan**: Prepare a backup plan in case the experiment goes wrong.
5. **Fix Issues**: Verify system resilience or identify problems to fix.
6. **Have Fun**: Proactively testing system failure modes reduces operational burden and increases availability.

## Hands-on Exercises

[Link to Hands-on Exercises](https://github.com/JiangRenDevOps/DevOpsNotes/tree/master/WK9_High_Availability_Scaling_K8S/hands_on)

## Reading Materials

[Link to Reading Materials](https://github.com/JiangRenDevOps/DevOpsNotes/tree/master/WK9_High_Availability_Scaling_K8S/reading_materials)
