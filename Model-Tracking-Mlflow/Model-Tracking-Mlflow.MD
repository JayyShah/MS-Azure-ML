# Tracking Metrics with MLFlow

- When you train a model with a script, you can include MLflow in the scripts to track any parameters, metrics, and artifacts. When you run the script as a job in Azure Machine Learning, you're able to review all input parameters and outputs for each run.

## Include MLflow in the environment

- To use MLflow during training job, the mlflow and azureml-mlflow pip packages need to be installed on the compute executing the script. Therefore, you need to include these two packages in the environment. You can create an environment by referring to a YAML file that describes the Conda environment. As part of the Conda environment, you can include these two packages.

```yml

name: mlflow-env
channels:
  - conda-forge
dependencies:
  - python=3.8
  - pip
  - pip:
    - numpy
    - pandas
    - scikit-learn
    - matplotlib
    - mlflow
    - azureml-mlflow

```

## Enable Autologging

- When working with one of the common libraries for machine learning, you can enable autologging in MLflow. Autologging logs parameters, metrics, and model artifacts without anyone needing to specify what needs to be logged.

- Autologging is supported for the following libraries:

    - Scikit-learn
    - TensorFlow and Keras
    - XGBoost
    - LightGBM
    - Spark
    - Fastai
    - Pytorch

## Log Metrics with MLFlow

- In your training script, you can decide whatever custom metric you want to log with MLflow.

- Depending on the type of value you want to log, use the MLflow command to store the metric with the experiment run:

    - mlflow.log_param(): Log single key-value parameter. Use this function for an input parameter you want to log.
    - mlflow.log_metric(): Log single key-value metric. Value must be a number. Use this function for any output you want to store with the run.
    - mlflow.log_artifact(): Log a file. Use this function for any plot you want to log, save as image file first.

## View Metrics and Evaluate models

- After you've trained and tracked models with MLflow in Azure Machine Learning, you can explore the metrics and evaluate your models.

    - Review metrics in the Azure Machine Learning studio.
    - Retrieve runs and metrics with MLflow.
    
- *Azure Machine Learning uses the concept of jobs when you run a script. Multiple job runs in Azure Machine Learning can be grouped within one experiment. MLflow uses a similar syntax where each script is a run, which is part of an experiment.*

## View Metrics in Azure ML studio

- When your job is completed, you can review the logged parameters, metrics, and artifacts in the Azure Machine Learning studio.

- When you review job runs in the Azure Machine Learning studio, you'll explore a job run's metrics, which is part of an experiment.

- To view the metrics through an intuitive user interface, you can:

    - Open the Studio by navigating to https://ml.azure.com.
    - Find your experiment run and open it to view its details.
    - In the Details tab, all logged parameters are shown under Params.
    - Select the Metrics tab and select the metric you want to explore.
    - Any plots that are logged as artifacts can be found under Images.
    - The model assets that can be used to register and deploy the model are stored in the models folder under Outputs + logs.