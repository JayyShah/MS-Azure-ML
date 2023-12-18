# Make data available in Azure Machine Learning

## Introduction

- Data is a fundamental element in any machine learning workload. You need data to train a model and you create data when using a model to generate predictions.

- To work with data in Azure Machine Learning, you can access data by using Uniform Resource Identifiers (URIs). When you work with a data source or a specific file or folder repeatedly, you can create `datastores` and `data assets` within the Azure Machine Learning workspace. Datastores and data assets allow you to securely store the connection information to your data.

# Uniform Resource Identifiers (URI):

- You can store data on your local device, or somewhere in the cloud. Wherever you store your data, you want to access the data when training machine learning models. To find and access data in Azure Machine Learning, you can use Uniform Resource Identifiers (URIs).

![URI](URI.png)

- A URI references the location of your data. For Azure Machine Learning to connect to your data, you need to prefix the URI with the appropriate protocol. There are three common protocols when working with data in the context of Azure Machine Learning

    - http(s): Use for data stores publicly or privately in an Azure Blob Storage or publicly available http(s) location.
    - abfs(s): Use for data stores in an Azure Data Lake Storage Gen 2.
    - azureml: Use for data stored in a datastore.

- When you want to access the data from the Azure Machine Learning workspace, you can use the path to the folder or file directly. When you want to connect to the folder or file directly, you can use the http(s) protocol. If the container is set to private, you'll need to provide some kind of authentication to get access to the data, like a Shared Access Signature (SAS).

- When you create a datastore in Azure Machine Learning, you'll store the connection and authentication information in the workspace. Then, to access the data in the container, you can use the azureml protocol.

- *A datastore is a reference to an existing storage account on Azure. Therefore, when you refer to data stored in a datastore, you may be referring to data being stored in an Azure Blob Storage or Azure Data Lake Storage. When you refer to the datastore however, you won't need to authenticate as the connection information stored with the datastore will be used by Azure Machine Learning.*

# Create a DataStore 

- In Azure Machine Learning, datastores are abstractions for cloud data sources. They encapsulate the information needed to connect to data sources, and securely store this connection information so that you don’t have to code it in your scripts

- The benefits of using datastores are:

    - Provides easy-to-use URIs to your data storage.
    - Facilitates data discovery within Azure Machine Learning.
    - Securely stores connection information, without exposing secrets and keys to data scientists.

- When you create a datastore with an existing storage account on Azure, you have the choice between two different authentication methods

![authentication](auth.png)

    - Credential-based: Use a service principal, shared access signature (SAS) token or account key to authenticate access to your storage account.

    - Identity-based: Use your Microsoft Entra identity or managed identity.

## Types of Datastores

- Azure Machine Learning supports the creation of datastores for multiple kinds of Azure data source, including:

    - Azure Blob Storage
    - Azure File Share
    - Azure Data Lake (Gen 2)

- Every workspace has four built-in datastores (two connecting to Azure Storage blob containers, and two connecting to Azure Storage file shares), which are used as system storages by Azure Machine Learning.

- In most machine learning projects, you need to work with data sources of your own. For example, you can integrate your machine learning solution with data from existing applications or data engineering pipelines.

## Create a Datastore

- Datastores are attached to workspaces and are used to store connection information to storage services. When you create a datastore, you provide a name that can be used to retrieve the connection information.

- Datastores allow you to easily connect to storage services without having to provide all necessary details every time you want to read or write data. It also creates a protective layer if you want users to use the data, but not connect to the underlying storage service directly.

## Create a datastore for an Azure Blob Storage container

```python
blob_datastore = AzureBlobDatastore(
    			name = "blob_example",
    			description = "Datastore pointing to a blob container",
    			account_name = "mytestblobstore",
    			container_name = "data-container",
    			credentials = AccountKeyCredentials(
        			account_key="XXXxxxXXXxXXXXxxXXX"
    			),
)
ml_client.create_or_update(blob_datastore)
```

- Connect to an Azure Blob Storage container by using a SAS token to authenticate

```python
blob_datastore = AzureBlobDatastore(
name="blob_sas_example",
description="Datastore pointing to a blob container",
account_name="mytestblobstore",
container_name="data-container",
credentials=SasTokenCredentials(
sas_token="?xx=XXXX-XX-XX&xx=xxxx&xxx=xxx&xx=xxxxxxxxxxx&xx=XXXX-XX-XXXXX:XX:XXX&xx=XXXX-XX-XXXXX:XX:XXX&xxx=xxxxx&xxx=XXxXXXxxxxxXXXXXXXxXxxxXXXXXxxXXXXXxXXXXxXXXxXXxXX"
),
)
ml_client.create_or_update(blob_datastore)
```

# Create a Data Asset

- As a data scientist, you want to focus on training machine learning models. Though you need access to data as input for a machine learning model, you don't want to worry about how to get access. To simplify getting access to the data you want to work with, you can use data assets.

- In Azure Machine Learning, data assets are references to where the data is stored, how to get access, and any other relevant metadata. You can create data assets to get access to data in datastores, Azure storage services, public URLs, or data stored on your local device.

- The benefits of using data assets are:

    - You can share and reuse data with other members of the team such that they don't need to remember file locations.
    - You can seamlessly access data during model training (on any supported compute type) without worrying about   connection strings or data paths.
    - You can version the metadata of the data asset.

![data-asset](data-asset.png)

- URI file: Points to a specific file.
- URI folder: Points to a folder.
- MLTable: Points to a folder or file, and includes a schema to read as tabular data.

- Data assets are most useful when executing machine learning tasks as Azure Machine Learning jobs. As a job, you can run a Python script that takes inputs and generates outputs. A data asset can be parsed as both an input or output of an Azure Machine Learning job.

## Create a URI file data asset

- A URI file data asset points to a specific file. Azure Machine Learning only stores the path to the file, which means you can point to any type of file. When you use the data asset, you specify how you want to read the data, which depends on the type of data you're connecting to.

- The supported paths you can use when creating a URI file data asset are:

    - Local: ./<path>
    - Azure Blob Storage: wasbs://<account_name>.blob.core.windows.net/<container_name>/<folder>/<file>
    - Azure Data Lake Storage (Gen 2): abfss://<file_system>@<account_name>.dfs.core.windows.net/<folder>/<file>
    - Datastore: azureml://datastores/<datastore_name>/paths/<folder>/<file>

- *When you create a data asset and point to a file or folder stored on your local device, a copy of the file or folder will be uploaded to the default datastore workspaceblobstore. You can find the file or folder in the LocalUpload folder. By uploading a copy, you'll still be able to access the data from the Azure Machine Learning workspace, even when the local device on which the data is stored is unavailable.*

```python
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

my_path = '<supported-path>'

my_data = Data(
    path=my_path,
    type=AssetTypes.URI_FILE,
    description="<description>",
    name="<name>",
    version="<version>"
)

ml_client.data.create_or_update(my_data)
```

- When you parse the URI file data asset as input in an Azure Machine Learning job, you first need to read the data before you can work with it.

- Imagine you create a Python script you want to run as a job, and you set the value of the input parameter input_data to be the URI file data asset (which points to a CSV file). 

```python
import argparse
import pandas as pd

parser = argparse.ArgumentParser()
parser.add_argument("--input_data", type=str)
args = parser.parse_args()

df = pd.read_csv(args.input_data)
print(df.head(10))
```

## Create a URI folder data asset

- A URI folder data asset points to a specific folder. It works similar to a URI file data asset and supports the same paths.

- To create a URI folder data asset with the Python SDK

```python

from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

my_path = '<supported-path>'

my_data = Data(
    path=my_path,
    type=AssetTypes.URI_FOLDER,
    description="<description>",
    name="<name>",
    version='<version>'
)

ml_client.data.create_or_update(my_data)
```

- When you parse the URI folder data asset as input in an Azure Machine Learning job, you first need to read the data before you can work with it.

- Imagine you create a Python script you want to run as a job, and you set the value of the input parameter input_data to be the URI folder data asset (which points to multiple CSV files).
 

```python

import argparse
import glob
import pandas as pd

parser = argparse.ArgumentParser()
parser.add_argument("--input_data", type=str)
args = parser.parse_args()

data_path = args.input_data
all_files = glob.glob(data_path + "/*.csv")
df = pd.concat((pd.read_csv(f) for f in all_files), sort=False)

```

- *Depending on the type of data you're working with, the code you use to read the files can change.*

## Create a MLTable data asset

- A MLTable data asset allows you to point to tabular data. When you create a MLTable data asset, you specify the schema definition to read the data. As the schema is already defined and stored with the data asset, you don't have to specify how to read the data when you use it.

- Therefore, you want to use a MLTable data asset when the schema of your data is complex or changes frequently. Instead of changing how to read the data in every script that uses the data, you only have to change it in the - data asset itself.
 
- When you define the schema when creating a MLTable data asset, you can also choose to only specify a subset of the data.

- For certain features in Azure Machine Learning, like Automated Machine Learning, you need to use a MLTable data asset, as Azure Machine Learning needs to know how to read the data.
 
- To define the schema, you can include a MLTable file in the same folder as the data you want to read. The MLTable file includes the path pointing to the data you want to read.

```yml

type: mltable

paths:
  - pattern: ./*.txt
transformations:
  - read_delimited:
      delimiter: ','
      encoding: ascii
      header: all_files_same_headers

```

- Create a MLTable Data asset with the Python SDk

```python

from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

my_path = '<path-including-mltable-file>'

my_data = Data(
    path=my_path,
    type=AssetTypes.MLTABLE,
    description="<description>",
    name="<name>",
    version='<version>'
)

ml_client.data.create_or_update(my_data)

```

- When you parse a MLTable data asset as input to a Python script you want to run as an Azure Machine Learning job

```python

import argparse
import mltable
import pandas

parser = argparse.ArgumentParser()
parser.add_argument("--input_data", type=str)
args = parser.parse_args()

tbl = mltable.load(args.input_data)
df = tbl.to_pandas_dataframe()

print(df.head(10))

```