# Introduction

- As a data scientist, you want to write code that works in any development environment. Whether you're using local or cloud compute, the code should successfully execute to train a machine learning model for example.

- To run code, you need to ensure necessary packages, libraries, and dependencies are installed on the compute you use to run the code. In Azure Machine Learning, environments list and store the necessary packages that you can reuse across compute targets.

## Environment in Azure ML

- When you create an Azure Machine Learning workspace, curated environments are automatically created and made available to you. Alternatively, you can create and manage your own custom environments and register them in the workspace. Creating and registering custom environments makes it possible to define consistent, reusable runtime contexts for your experiments - regardless of where the experiment script is run.

- Azure Machine Learning builds environment definitions into Docker images and conda environments. When you use an environment, Azure Machine Learning builds the environment on the Azure Container registry associated with the workspace.

- *When you create an Azure Machine Learning workspace, you can choose whether to use an existing Azure Container registry, or whether to let the workspace create a new registry for you when needed.*

## Exploring and Using Curated Environments

- Curated environments are prebuilt environments for the most common machine learning workloads, available in your workspace by default.

- Curated environments use the prefix AzureML- and are designed to provide for scripts that use popular machine learning frameworks and tooling.

- For example, there are curated environments for when you want to run a script that trains a regression, clustering, or classification model with Scikit-Learn.

## Custom Environments

- When you need to create your own environment in Azure Machine Learning to list all necessary packages, libraries, and dependencies to run your scripts, you can create custom environments.

- You can define an environment from a Docker image, a Docker build context, and a conda specification with Docker image.

- The easiest approach is likely to be to create an environment from a Docker image. Docker images can be hosted in a public registry like Docker Hub or privately stored in an Azure Container registry.

```python

from azure.ai.ml.entities import Environment

env_docker_image = Environment(
    image="pytorch/pytorch:latest",
    name="public-docker-image-example",
    description="Environment created from a public Docker image.",
)
ml_client.environments.create_or_update(env_docker_image)

```

## Create Custom Env with conda specific file

```yml

name: basic-env-cpu
channels:
  - conda-forge
dependencies:
  - python=3.7
  - scikit-learn
  - pandas
  - numpy
  - matplotlib

```

- Create an environment from a base Docker image and a conda specification file

```python

from azure.ai.ml.entities import Environment

env_docker_conda = Environment(
    image="mcr.microsoft.com/azureml/openmpi3.1.2-ubuntu18.04",
    conda_file="./conda-env.yml",
    name="docker-image-plus-conda-example",
    description="Environment created from a Docker image plus Conda environment.",
)
ml_client.environments.create_or_update(env_docker_conda)

```