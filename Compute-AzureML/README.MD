#  Working with compute targets in Azure Machine Learning 

## Introduction

- As a data scientist, you may train machine learning models on your local device. For large-scale projects, a single local device won't be enough to efficiently train machine learning models. When you use cloud compute for machine learning workloads, you'll be ready to scale your work when needed.

- In Azure Machine Learning, you can use various types of managed cloud computes. By using any of the compute options in the Azure Machine Learning workspace, you'll save time on managing compute.

- Whether you're working in notebooks during experimentation, or need to run scripts for production, Azure Machine Learning compute will help you run your workloads.

- Most commonly, you'll work with either a compute instance or a compute cluster in Azure Machine Learning.

## Choose the appropriate compute target

- In Azure Machine Learning, compute targets are physical or virtual computers on which jobs are run.

## Understand the available types of compute

- Azure Machine Learning supports multiple types of compute for experimentation, training, and deployment. By having multiple types of compute, you can select the most appropriate type of compute target for your particular needs.

    - Compute instance: Behaves similarly to a virtual machine and is primarily used to run notebooks. It's ideal for experimentation.

    - Compute clusters: Multi-node clusters of virtual machines that automatically scale up or down to meet demand. A cost-effective way to run scripts that need to process large volumes of data. Clusters also allow you to use parallel processing to distribute the workload and reduce the time it takes to run a script.

    - Kubernetes clusters: Cluster based on Kubernetes technology, giving you more control over how the compute is configured and managed. You can attach your self-managed Azure Kubernetes (AKS) cluster for cloud compute, or an Arc Kubernetes cluster for on-premises workloads.

    - Attached compute: If you already use an Azure-based compute environment for data science, such as an Azure  virtual machine or an Azure Databricks cluster, you can attach it to your Azure Machine Learning workspace and use it as a compute target for certain types of workload.

- *In Azure Machine Learning, another type of compute exists for inference. Inference compute is more lightweight than compute targets designed for training, and can only be used to deploy trained models as endpoints.*

## Choose a compute target for experimentation

- Imagine you're a data scientist and you've been asked to develop a new machine learning model. You'll likely have a small subset of the training data with which you can experiment.

- During experimentation and development, you may prefer working in a Jupyter notebook. A notebook experience - benefits most from a compute that is continuously running.

- When working with notebooks, you'll therefore want to use a compute instance.

## Choose a compute target for production

- After experimentation, you may train your models by running Python scripts to prepare for production. Scripts will be easier to automate and schedule for when you want to retrain your model continuously over time.

- When moving to production, you want the compute target to be ready to handle large volumes of data. The more data you use, the better the machine learning model is likely to be.

- When training models with scripts, you'll prefer a compute cluster. A compute cluster will automatically scale up when the script(s) need to be executed, and scale down when the script has finished executing.

## Choose a compute target for scalability

- Training machine learning models can take a long time. To train multiple models that you want to evaluate afterwards, you may spend a long time waiting for all models to complete before you can compare their evaluation metrics.
 
- Azure Machine Learning offers features like Automated Machine Learning and hyperparameter tuning with sweep jobs to iterate over various configurations of a model. When working with such features, you'll want the models to be trained in parallel to more quickly decide which model performs best and is suited for further development or even production.

- When you want to train multiple models simultaneously to speed up processing time, you'll want to use a compute cluster. A cluster offers multiple nodes, which each can perform a separate task in parallel.

# Creating a Compute Instance

- Creating a Compute instance using a Python SDK

```python

from azure.ai.ml.entities import ComputeInstance

ci_basic_name = "basic-ci-12345"
ci_basic = ComputeInstance(
    name=ci_basic_name, 
    size="STANDARD_DS3_v2"
)
ml_client.begin_create_or_update(ci_basic).result()

```

- Alternatively, you can also create a compute instance with a script. The benefit of using a script is that you can ensure any necessary packages, tools, or software are automatically installed and clone any repositories that you need to work with. When you need to create compute instances for multiple users, using a script will allow you to create a consistent development environment for everyone.

- As a data scientist, you'll attach a compute instance to notebooks in order to run cells within the notebook. To be allowed to work with the compute instance, it needs to be assigned to you as a user.

- A compute instance can only be assigned to one user, as the compute instance can't handle parallel workloads. When you create a new compute instance, you can assign it to someone else if you have the appropriate permissions.

- When you're actively working on code in a notebook, you want your compute instance to be running. When you're not executing any code, you want your compute instance to be stopped to save on costs.

- When a compute instance is assigned to you, you can start and stop a compute instance whenever you need. You can also add a schedule to the compute instance to start or stop at set times.

- By scheduling your compute instance to stop at the end of every day, you'll avoid unnecessary costs if you forget to stop a compute instance.

# Create a compute cluster

- After experimentation and development, you want your code to be production-ready. To run code in production environments, it's recommended to save code as scripts. When you run a script, you'll want to use a compute target that is scalable. Within Azure Machine Learning, compute clusters are ideal for running scripts.

- Create a compute cluster with the Python SDK

```python

from azure.ai.ml.entities import AmlCompute

cluster_basic = AmlCompute(
    name="cpu-cluster",
    type="amlcompute",
    size="STANDARD_DS3_v2",
    location="westus",
    min_instances=0,
    max_instances=2,
    idle_time_before_scale_down=120,
    tier="low_priority",
)
ml_client.begin_create_or_update(cluster_basic).result()

```

- When you create a compute cluster, there are three main parameters you need to consider:

    size: Specifies the virtual machine type of each node within the compute cluster. Based on the sizes for virtual machines in Azure. Next to size, you can also specify whether you want to use CPUs or GPUs.

    max_instances: Specifies the maximum number of nodes your compute cluster can scale out to. The number of   parallel workloads your compute cluster can handle is analogous to the number of nodes your cluster can scale to.

    tier: Specifies whether your virtual machines will be low priority or dedicated. Setting to low priority can lower costs as you're not guaranteed availability.

## Using a compute cluster

- There are three main scenarios in which you'll want to use a compute cluster:

    - Running a pipeline job you built in the Designer.
    - Running an Automated Machine Learning job.
    - Running a script as a job.

- In each of these scenarios, a compute cluster is ideal as a compute cluster will automatically scale up when a job is submitted, and automatically shut down when a job is completed.

- A compute cluster will also allow you to train multiple models in parallel, which is a common practice when using Automated Machine Learning.

- You can run a Designer pipeline job and an Automated Machine Learning job through the Azure Machine Learning studio. When you submit the job through the studio, you can set the compute target to the compute cluster you created.

- When you prefer a code-first approach, you can set the compute target to your compute cluster by using the Python SDK.

```python

from azure.ai.ml import command

# configure job
job = command(
    code="./src",
    command="python diabetes-training.py",
    environment="AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest",
    compute="cpu-cluster",
    display_name="train-with-cluster",
    experiment_name="diabetes-training"
    )

# submit job
returned_job = ml_client.create_or_update(job)
aml_url = returned_job.studio_url
print("Monitor your job at", aml_url)

```

- After submitting a job that uses a compute cluster, the compute cluster will scale out to one or more nodes. Resizing will take a few minutes, and your job will start running once the necessary nodes are provisioned. When a job's status is preparing, the compute cluster is being prepared. When the status is running, the compute cluster is ready, and the job is running.