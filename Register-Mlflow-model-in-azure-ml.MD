# Log models with MLflow

*Code Implementation can be found in registering-mlflow-in-AzureML.ipynb in the Notebooks Directory.*

- To train a machine learning model, you may choose to use an open source framework that best suits your needs. After training, you want to deploy your model. MLflow is an open source platform that streamlines machine learning deployment, regardless of the type of model you trained and the framework you used.

- MLflow is integrated with Azure Machine Learning. The integration with Azure Machine Learning allows you to easily deploy models that you train and track with Mlflow. For example, when you have an MLflow model, you can opt for the no-code deployment in Azure Machine Learning.

## Why Mlflow?

- When you train a machine learning model with Azure Machine Learning, you can use MLflow to register your model. MLflow standardizes the packaging of models, which means that an MLflow model can easily be imported or exported across different workflows.

- For example, imagine training a model in an Azure Machine Learning workspace used for development. If you want to export the model to another workspace used for production, you can use an MLflow model to easily do so.

- When you train and log a model, you store all relevant artifacts in a directory. When you register the model, an MLmodel file is created in that directory. The MLmodel file contains the model's metadata, which allows for model traceability.

- You can register models with MLflow by enabling autologging, or by using custom logging.

*MLflow allows you to log a model as an artifact, or as a model. When you log a model as an artifact, the model is treated as a file. When you log a model as a model, you're adding information to the registered model that enables you to use the model directly in pipelines or deployments.*

## Use autologging to log a model

- When you train a model, you can include mlflow.autolog() to enable autologging. MLflow's autologging automatically logs parameters, metrics, artifacts, and the model you train. The model is logged when the .fit() method is called. The framework you use to train your model is identified and included as the flavor of your model.

- Optionally, you can specify which flavor you want your model to be identified as by using mlflow.<flavor>.autolog(). Some common flavors that you can use with autologging are:

    - Keras: `(mlflow.keras.autolog)`
    - Scikit-learn: `(mlflow.sklearn.autolog())`
    - LightGBM: `(mlflow.lightgbm.autolog)`
    - XGBoost: `(mlflow.xgboost.autolog)`
    - TensorFlow: `(mlflow.tensorflow.autolog)`
    - PyTorch: `(mlflow.pytorch.autolog)`
    - ONNX: `(mlflow.onnx.autolog)`

- When you use autologging, an output folder is created which includes all necessary model artifacts, including the MLmodel file that references these files and includes the model's metadata.

## Manually log a model

- When you want to have more control over how the model is logged, you can use autolog (for your parameters, metrics, and other artifacts), and set log_models=False. MLflow doesn't automatically log the model, and you can add it manually.

- As logging the model allows you to easily deploy the model, you may want to customize the model's expected inputs and outputs. The schemas of the expected inputs and outputs are defined as the signature in the MLmodel file. If you deploy your model and the inputs don't match the defined schema in the signature, you may encounter errors.

- Therefore, you may want to customize the signature to alter the deployed model's behavior.

## Customize the signature

- The model signature defines the schema of the model's inputs and outputs. The signature is stored in JSON format in the MLmodel file, together with other metadata of the model.

- The model signature can be inferred from datasets or created manually by hand.

- To log a model with a signature that is inferred from your training dataset and model predictions, you can use infer_signature().

```bash
import pandas as pd
from sklearn import datasets
from sklearn.ensemble import RandomForestClassifier
import mlflow
import mlflow.sklearn
from mlflow.models.signature import infer_signature

iris = datasets.load_iris()
iris_train = pd.DataFrame(iris.data, columns=iris.feature_names)
clf = RandomForestClassifier(max_depth=7, random_state=0)
clf.fit(iris_train, iris.target)

# Infer the signature from the training dataset and model's predictions
signature = infer_signature(iris_train, clf.predict(iris_train))

# Log the scikit-learn model with the custom signature
mlflow.sklearn.log_model(clf, "iris_rf", signature=signature)
```

- Alternatively, you can create the signature manually

```bash
from mlflow.models.signature import ModelSignature
from mlflow.types.schema import Schema, ColSpec

# Define the schema for the input data
input_schema = Schema([
  ColSpec("double", "sepal length (cm)"),
  ColSpec("double", "sepal width (cm)"),
  ColSpec("double", "petal length (cm)"),
  ColSpec("double", "petal width (cm)"),
])

# Define the schema for the output data
output_schema = Schema([ColSpec("long")])

# Create the signature object
signature = ModelSignature(inputs=input_schema, outputs=output_schema)
```

# Understand the MLflow model format

- MLflow uses the MLModel format to store all relevant model assets in a folder or directory. One essential file in the directory is the MLmodel file. The MLmodel file is the single source of truth about how the model should be loaded and used.

## Explore the MLmodel file format

- The MLmodel file may include:

    - artifact_path: During the training job, the model is logged to this path.
    - flavor: The machine learning library with which the model was created.
    - model_uuid: The unique identifier of the registered model.
    - run_id: The unique identifier of job run during which the model was created.
    - signature: Specifies the schema of the model's inputs and outputs:
        - inputs: Valid input to the model. For example, a subset of the training dataset.
        - outputs: Valid model output. For example, model predictions for the input datas.

- Example of MLmodel file:

```yml
artifact_path: classifier
flavors:
  fastai:
    data: model.fastai
    fastai_version: 2.4.1
  python_function:
    data: model.fastai
    env: conda.yaml
    loader_module: mlflow.fastai
    python_version: 3.8.12
model_uuid: e694c68eba484299976b06ab9058f636
run_id: e13da8ac-b1e6-45d4-a9b2-6a0a5cfac537
signature:
  inputs: '[{"type": "tensor",
             "tensor-spec": 
                 {"dtype": "uint8", "shape": [-1, 300, 300, 3]}
           }]'
  outputs: '[{"type": "tensor", 
              "tensor-spec": 
                 {"dtype": "float32", "shape": [-1,2]}
            }]'
```

- The most important things to set are the flavor and the signature.

## Choose the flavor

- A flavor is the machine learning library with which the model was created.

- For example, to create an image classification model to detect breast cancer you're using fastai. Fastai is a flavor in MLflow that tells how you how a model should be persisted and loaded. Because each model flavor indicates how they want to persist and load models, the MLModel format doesn't enforce a single serialization mechanism that all the models need to support. Such a decision allows each flavor to use the methods that provide the best performance or best support according to their best practices - without compromising compatibility with the MLModel standard.

- Python function flavor is the default model interface for models created from an MLflow run. Any MLflow python model can be loaded as a python_function model, which allows for workflows like deployment to work with any python model regardless of which framework was used to produce the model. This interoperability is immensely powerful as it reduces the time to operationalize in multiple environments.

- An Example of Python flavor function:

```yml
artifact_path: pipeline
flavors:
  python_function:
    env:
      conda: conda.yaml
      virtualenv: python_env.yaml
    loader_module: mlflow.sklearn
    model_path: model.pkl
    predict_fn: predict
    python_version: 3.8.5
  sklearn:
    code: null
    pickled_model: model.pkl
    serialization_format: cloudpickle
    sklearn_version: 1.2.0
mlflow_version: 2.1.0
model_uuid: b8f9fe56972e48f2b8c958a3afb9c85d
run_id: 596d2e7a-c7ed-4596-a4d2-a30755c0bfa5
signature:
  inputs: '[{"name": "age", "type": "long"}, {"name": "sex", "type": "long"}, {"name":
    "cp", "type": "long"}, {"name": "trestbps", "type": "long"}, {"name": "chol",
    "type": "long"}, {"name": "fbs", "type": "long"}, {"name": "restecg", "type":
    "long"}, {"name": "thalach", "type": "long"}, {"name": "exang", "type": "long"},
    {"name": "oldpeak", "type": "double"}, {"name": "slope", "type": "long"}, {"name":
    "ca", "type": "long"}, {"name": "thal", "type": "string"}]'
  outputs: '[{"name": "target", "type": "long"}]'
```

## Configure the signature

- Apart from flavors, the MLmodel file also contains signatures that serve as data contracts between the model and the server running your model.

- There are two types of signatures:

    - Column-based: used for tabular data with a pandas.Dataframe as inputs.
    - Tensor-based: used for n-dimensional arrays or tensors (often used for unstructured data like text or images), with numpy.ndarray as inputs.

- As the MLmodel file is created when you register the model, the signature also is created when you register the model. When you enable MLflow's autologging, the signature is inferred in the best effort way. If you want the signature to be different, you need to manually log the model.

- The signature's inputs and outputs are important when deploying your model. When you use Azure Machine Learning's no-code deployment for MLflow models, the inputs and outputs set in the signature will be enforced. In other words, when you send data to a deployed MLflow model, the expected inputs and outputs need to match the schema as defined in the signature.

# Register an MLFlow Model

- In Azure Machine Learning, models are trained in jobs. When you want to find the model's artifacts, you can find it in the job's outputs. To more easily manage your models, you can also store a model in the Azure Machine Learning model registry.

- The model registry makes it easy to organize and keep track of your trained models. When you register a model, you store and version your model in the workspace.

- Registered models are identified by name and version. Each time you register a model with the same name as an existing one, the registry increments the version. You can also add more metadata tags to more easily search for a specific model.

- *You can also register models trained outside Azure Machine Learning by providing the local path to the model's artifacts*

- There are three types of models you can register:

    - MLflow: Model trained and tracked with MLflow. Recommended for standard use cases.
    - Custom: Model type with a custom standard not currently supported by Azure Machine Learning.
    - Triton: Model type for deep learning workloads. Commonly used for TensorFlow and PyTorch model deployments.

- Azure Machine Learning integrates well with MLflow, which is why it's a best practice to log and register an MLflow model. Working with MLflow models makes model management and deployment in Azure Machine Learning easier. During deployment, for example, the environment and scoring script are created for you when using an MLflow model.

- To register an MLflow model, you can use the studio, the Azure CLI, or the Python SDK.

- To train the model, you can submit a training script as a command job

```python
from azure.ai.ml import command

# configure job

job = command(
    code="./src",
    command="python train-model-signature.py --training_data diabetes.csv",
    environment="AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest",
    compute="aml-cluster",
    display_name="diabetes-train-signature",
    experiment_name="diabetes-training"
    )

# submit job
returned_job = ml_client.create_or_update(job)
aml_url = returned_job.studio_url
print("Monitor your job at", aml_url)
```

- Once the job is completed and the model is trained, use the job name to find the job run and register the model from its outputs.

```python
from azure.ai.ml.entities import Model
from azure.ai.ml.constants import AssetTypes

job_name = returned_job.name

run_model = Model(
    path=f"azureml://jobs/{job_name}/outputs/artifacts/paths/model/",
    name="mlflow-diabetes",
    description="Model created from run.",
    type=AssetTypes.MLFLOW_MODEL,
)
# Uncomment after adding required details above
ml_client.models.create_or_update(run_model)
```

- All registered models are listed in the Models page of the Azure Machine Learning studio. The registered model includes the model's output directory. When you log and register an MLflow model, you can find the MLmodel file in the artifacts of the registered model.