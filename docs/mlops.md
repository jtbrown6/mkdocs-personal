# ML on Kubernetes with KubeFlow

## Building ML Apps Overview and Challenges

Machine Learning Enabled Applications do not have linear workflow. This means that the stages of research and development as well as production are iterative for the life cycle.


1. **Research**: the initial machine learning models are created and datasets are explored

2. **Development**: this is where initial models are refined, tested and tured into more production ready solutions

3. **Production**: models are deployed into real-world applications to process live data

4. **Post-Production**: where monitoring performance and retraining with new data is important which returns back to Research stage.

Using CI/CD helps manage complexity by providing structure into merging, testing and delivering code changes to production systems that can be done in an automated and reliable way. This helps ensure that applications remain stable and can rapidly adapt to new requirements or data. 


## Introducing Containers

Since containers do not require the entire OS, only the immediate components it needs to operate, this can improve the speed of development as developers know that the application will run the same way on different platforms. This allows developers to focus on their apps and operations can focus on instrastructure. This allows for integrating new code into an application so it can grow and mature through its lifecycle.


### Container-less Environment Challenges

Although MLOps and these capabilities can be solved without containers, there are many challenges developers and infrastructure teams will face. 

There is the option of `Virtual Machines` as a Partial Solution because the application can run with all dependancies in a self-contained environment BUT due the resource usage and overhead of managing full OS's which can lead to `under-utilization` of resources and `longer startup` times.

| Challenges | Details |
| ----------| -------- |
| Dev Environment Inconsistency &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | As Developers build code on their local machines, this leads to inconsistent setups and configurations  |
| Dependency Management | As the application grows, new code might require new libraries at specific versions which forces all servers (testing and prod), local dev machines etc to be installed for the code to actually run properly |
| Deployment Overheads | Need to deploy a new application? This would require coodination with the system admin or ops to manually setup the server environment to match the dev's environment. Each change could potentially lead to downtime. <br /> <br /> Containers are deployed as simple as pulling and building the container image on the host that has the Docker Runtime installed.  |
| Configuration Drift | This is common to "get things to work" where adhoc changes in a production environment to resolve issues or update components. This makes it harder to replicate issues or test new features due to the drift.  |
| Scaling Challenges  | This tends to require cloning production environments and setting this back up on new servers which is time consuming and can be error proned. You then need to load-balance and redistribute traffic to these new servers and new challenges might arise which could require manual configuration. <br /> <br /> Containers can be quickly started and stopped and even replicated.   |

## Kubernetes Value with AI/ML Workloads

Tools like `KubeFlow` help simply the process of training and deploying ML models at scale to standardize the idea of `MLOps`. 

1. **Scalability**: ML pipelines can accomodate large-scale processing and training without intefereing with other project elements

2. **Efficiency**: K8s will handle resource allocation by scheduling workloads on nodes based on availability and capacity. This insures that resources are being utilized with intention

3. **Portability**: allows for developing one ML model and deploy across multiple environments making it agnostic for any platform

4. **Fault Tolerance**: has self-healing capabilities to ensure K8's keeps ML pipelines running in the event of failures

### What does MLOps look like without Kubeflow?

Without Kubeflow or other Kubernetes-based platforms and tools, managing the machine learning lifecycle would be more complex and could look something like this:

1. **Development Stage**:
    - Developers write ML code on their local machines, which may have different environments and dependencies.
    - Sharing code across a team is challenging, and reproducing results is difficult due to environment discrepancies.
    - Scaling experiments from a local machine to a larger cluster is a manual and error-prone process.

2. **Testing Stage**:
    - Rigorous testing of models would require setting up multiple environments to simulate different operating systems or configurations.
    - There's no automated way to ensure that the code changes will work across all these environments, leading to potential failures after deployment.

3. **Deployment and Production Stage**:
    - Deploying an ML model into production involves transferring from a development environment to a server environment, which might not match.
    - Scaling the model to handle more data or more requests would typically require manual replication of the model setup on new servers.
    - Load balancing, rolling updates, and version control of ML models in production are done manually.

4. **Portability and Scalability**:
    - Moving an ML application from one cloud provider to another, or from cloud to on-premises, would require significant effort due to the tight coupling with the underlying infrastructure.
    - Scaling up and down to handle variable workloads is not automated and requires manual intervention, which could lead to inefficiencies and high operational costs.

5. **Infrastructure Management**:
    - Managing the underlying infrastructure for different ML tools and frameworks is done manually by IT and operations teams.
    - Each change in the infrastructure might affect the running ML applications, so careful coordination is required between teams.

6. **Operational Overhead**:
    - Each team must manage their own updates and security patches for all different environments, creating additional operational overhead.
    - Without a standardized application packaging system, collaboration and troubleshooting become more difficult.

In summary, without a platform like Kubeflow, the ML lifecycle would be characterized by fragmented development practices, inefficient resource utilization, operational challenges in maintaining consistency across environments, and difficulty in managing the orchestration of ML workflows at scale.

**Challenges Kubeflow Solves for MLOps**:

- **Standardization**: Kubeflow simplifies the operationalization of ML workflows by providing a standard, unified platform that works across various environments.
  
- **Automation**: With Kubernetes, many of the manual deployment and scaling processes are automated. Kubeflow builds on this to automate ML pipelines, from data preprocessing to model training and serving.
  
- **Scalability**: Kubeflow leverages Kubernetes' ability to scale resources efficiently and automatically according to the demands of the workload.

- **Portability**: Kubeflow makes it easier to move ML workloads between environments—cloud, on-premises, or hybrid—without needing significant changes.

- **Reproducibility**: Kubeflow helps ensure that ML experiments are reproducible, with a consistent environment from development to production.

- **Resource Utilization**: Kubernetes optimizes the use of underlying hardware, and Kubeflow manages the ML-specific resource requirements.

By explaining these challenges and showcasing how Kubeflow helps overcome them, you can make a compelling case for adopting Kubeflow and Kubernetes in managing ML workloads. The efficiency, agility, and cost benefits provided by these technologies are strong motivators for organizations looking to streamline their MLOps practices.