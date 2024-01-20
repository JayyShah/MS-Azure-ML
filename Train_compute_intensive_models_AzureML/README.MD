# Introduction

- Organizations can efficiently preprocess large amounts of data in Azure Machine Learning.

- Imagine you work as a data scientist for an air carrier company. The analytics team gives you access to a large volume of flight data that is stored in one folder. They want you to analyze this data to extract insights. Together with the team you decide to do everything within Azure Machine Learning to keep the data within one environment for preprocessing, model training, and deployment. As a first step, you wonder how to load this data into Azure Machine Learning to preprocess the data.

# Choose the appropriate AI approach for your data

- Machine learning and deep learning are two approaches to analyze your data. Using GPU-optimized virtual machines in Azure Machine Learning can speed up training when using both machine learning or deep learning algorithms.

- The company you work for as a data scientist has given you a large amount of data. You want to analyze the data to make any insights available to the analytics team.You've heard about machine learning and deep learning, but are unsure which approach to use in this situation.

- Machine learning is the practice of using algorithms to analyze data. By training a model on a large dataset, we can derive predictions or decisions from the data.

- Some examples of what machine learning can be used for:

    - Classification: predict a categorical value.
    - Regression: predict a numerical value.
    - Time-series forecasting: predict future numerical values based on time-stamped data

- Deep learning is an advanced form of machine learning that tries to mimic the way the human brain learns.

- The human brain works by examples. We experience what input corresponds to what output. For example, we see round and red shapes in an image, and learn that the shapes correspond to an apple. By experiencing similar inputs and outputs, we improve over time and eventually are able to correctly identify an apple.

- A deep learning algorithm works in a similar way. The list of examples is the dataset a model uses during training time. The model takes the input and tries to guess what the output should be during an epoch. At the end of an epoch, the model finds out if the answer was correct and takes that learning to improve over time.

- Some examples of what deep learning can be used for:

    - Computer vision: classify images or detect objects in images.
    - Voice recognition: transcribe speech to text and identify the individual speaker.
    - Real-time language translation: translate text in real time for automatic subtitles.

- Whether working with machine learning or deep learning algorithms, it's important to have a good quality dataset to improve your model's performance. Next to quality, data scientists also strive for large datasets as more data can result in a more representative model.

- Both the amount of data you use and the algorithm you choose requires enough compute power. Within Azure Machine Learning, you can easily choose the compute that suits your workload best.

# CPU OR GPU

- Azure Machine Learning supports the use of both CPU, and GPU virtual machines when creating compute instances or clusters. You can run a script with either compute type, but when should you use which?

- For standard workloads, a CPU virtual machine type is likely to be sufficient. For example, if you have a small dataset and you want to manipulate the data with the pandas library, use CPU.

- *Note:Some frameworks and libraries only work with either CPU or GPU compute. Be aware of the restrictions of the framework or library you want to work with to avoid paying for unused resources. For example, scikit-learn is only compatible with CPU compute.*

- Alternatively, GPU-optimized compute is more powerful, which results in faster processing. The performance GPU offers however, comes at a larger cost. If you want to train a deep learning model with the TensorFlow or PyTorch library, for example, you’ll need GPU compute to do it efficiently.

- Whether you should use GPU instead of CPU compute depends on the data you work with and the processing you want to do. In general, GPUs can be more cost-efficient when:

    - You use a large dataset for training, slowing down model training.

    - You train deep learning models.

- Whenever CPUs appear to increase the processing time significantly, GPUs can minimize costs by running the workload in a fraction of the time.

# Efficient Data Storage Options 

- As many machine-learning models benefit from large amounts of data, store your large dataset efficiently to reduce processing time.

- Recall that you received a large dataset from the analytics team. You know you have to store it efficiently to optimize processing time. Whether it is for preparing and exploring the flight data, or to train a machine or deep learning model on the data.

## Choose an Azure Data Lake Storage Gen2

- Together with the analytics team, you have decided to do all processing and model training in Azure Machine Learning. To easily and securely access the data from the Azure Machine Learning workspace, you want to store the data in Azure.

- Although there are several options to store data in Azure, the best solution when working with Azure Machine Learning is to store the data in an Azure Data Lake Storage Gen2, no matter the size of the data.

- Compared to an Azure Blob Storage, the Azure Data Lake Gen2 provides a hierarchical namespace to store your files.

- With the hierarchical namespace, you can use a nested folder structure to optimize listing operations. Next to better scalability and performance, structuring your files like this will also allow for fine-granular access.

- The reason why it's more efficient to use a data lake instead of a flat object storage, is because it's best to avoid putting all your files in one folder.

- If all files are stored in one folder, regardless of which storage solution you choose, reading the files will be demanding for your compute.

- The flight data you received, are a large collection of CSV files that show the flight information for each month. Based on these recommendations, you choose to migrate the data to an Azure Data Lake Storage and create a nested folder structure based on date. Doing so will allow you to easily select for which time period you want to load in the flight data.

- And finally, when storing your files you should avoid having many small files. Reading a 1000 small files is much slower than reading one file with 1000x the size.

- After migrating the data and allowing Azure Machine Learning to connect to the Azure Data Lake, you want to use the flight data as input when running a job.

- When working with data in Azure Machine Learning, you can either download or mount the data to the Compute Cluster assigned to run the job:

    - Download the data if you estimate the dataset will fit onto the virtual machine's disk.
    - Mount the data if you expect the dataset to be too large to be downloaded on to the disk.

# Optimize Data loading and PreProcessing in Azure Machine Learning

- Data scientists can load and process data more quickly using RAPIDS with GPU compute.

- Recall the large volume of flight data you received as a data scientist for the air carrier company. One important performance metric of this company is whether a flight is delayed for more than 15 minutes. You want to train a classification model so you can predict when a future flight will be delayed. Before you can train the model, you need to load and process the data.

## RAPIDS

- RAPIDS is a machine-learning framework created by NVIDIA. You can use the RAPIDS libraries for:

    - Data loading and preprocessing
    - Machine learning
    - Graph analytics
    - Visualization

## cuDF

- A typical approach for data scientists is to use the pandas library to process tabular data. When working with large volumes of data however, it can take a long time to load and manipulate your data with pandas on CPUs. Next to a decline in performance, you may run into memory issues. To improve performance and avoid memory issues on your compute, use GPUs to process large data more quickly. As pandas only works with CPUs, you'll have to use the cuDF library to work with GPUs.

- cuDF is essentially pandas on GPU. Similar to pandas, cuDF is a Python DataFrame library for loading and processing data.

```python

# Loading a File into a Dataframe with Pandas
import pandas as pd

flight_data = pd.read_csv(csvfile)


# Loading a File into a Dataframe with cuDF

import cudf

flight_data = cudf.read_csv(csvfile)


```

-  The data includes where and when an airplane took off, and where and when it landed. The values for the origin and destination of a flight are the codes of the airports they visited.

- You find an open dataset with information about the airports, like the full names of the airport, the codes, and the location on the map. To add the airport data to your flight data, you want to merge these two datasets.

```python

data = cudf.merge(flight_data, airports, left_on='Dest', right_on='iata_code', how='left')
data = cudf.merge(flight_data, airports, left_on='Origin', right_on='iata_code', how='left')

# TO make columns more readable you can rename as

data = data.rename(columns= { 'latitude_deg_y' : 'origin_lat', 'longitude_deg_y': 'origin_long',
                              'latitude_deg_x' : 'dest_lat', 'longitude_deg_x': 'dest_long',
                              'Description' : 'Airline'})

```

- Machine learning models can be affected by missing data. An easy way to avoid this, especially when working with large datasets, is to remove the rows in which any of the columns have missing values.