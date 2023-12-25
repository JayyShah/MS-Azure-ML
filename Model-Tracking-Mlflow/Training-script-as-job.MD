## Convert a NoteBook to Script

- Scripts are ideal for testing and automation in your production environment. To create a production-ready script, you'll need to:

    - Remove nonessential code.
    - Refactor your code into functions.
    - Test your script in the terminal.

## Remove All Non-Essential Codes

- The main benefit of using notebooks is being able to quickly explore your data. For example, you can use print() and df.describe() statements to explore your data and variables. When you create a script that will be used for automation, you want to avoid including code written for exploratory purposes.

- The first thing you therefore need to do to convert your code to production code is to remove the nonessential code. Especially when you'll run the code regularly, you want to avoid executing anything nonessential to reduce cost and compute time.

## Refactor your code into Functions

- When using code in business processes, you want the code to be easy to read so that anyone can maintain it. One common approach to make code easier to read and test is to use functions.

- For Example

```python

# read and visualize the data
print("Reading data...")
df = pd.read_csv('diabetes.csv')
df.head()

# split data
print("Splitting data...")
X, y = df[['Pregnancies','PlasmaGlucose','DiastolicBloodPressure','TricepsThickness','SerumInsulin','BMI','DiabetesPedigree','Age']].values, df['Diabetic'].values

from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)

```

- As functions also allow you to test parts of your code, you might prefer to create multiple smaller functions, rather than one large function. If you want to test a part of your code, you can choose to only test a small part and avoid running more code than necessary.

```python

def main(csv_file):
    # read data
    df = get_data(csv_file)

    # split data
    X_train, X_test, y_train, y_test = split_data(df)

# function that reads the data
def get_data(path):
    df = pd.read_csv(path)
    
    return df

# function that splits the data
def split_data(df):
    X, y = df[['Pregnancies','PlasmaGlucose','DiastolicBloodPressure','TricepsThickness',
    'SerumInsulin','BMI','DiabetesPedigree','Age']].values, df['Diabetic'].values

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)

    return X_train, X_test, y_train, y_test

```

## Test your scripts

- Before using scripts in production environments, for example by integrating them with automation pipelines, you'll want to test whether the scripts work as expected.

- One simple way to test your script, is to run the script in a terminal. Within the Azure Machine Learning workspace, you can quickly run a script in the terminal of the compute instance.

- When you open a script in the Notebooks page of the Azure Machine Learning studio, you can choose to save and run the script in the terminal.

## Running a Script as Command Job

- To run a script as a command job, you'll need to configure and submit the job.

- To configure a command job with the Python SDK (v2), you'll use the command function. To run a script, you'll need to specify values for the following parameters:

    - code: The folder that includes the script to run.
    - command: Specifies which file to run.
    - environment: The necessary packages to be installed on the compute before running the command.
    - compute: The compute to use to run the command.
    - display_name: The name of the individual job.
    - experiment_name: The name of the experiment the job belongs to.

```python

from azure.ai.ml import command

# configure job
job = command(
    code="./src",
    command="python train.py",
    environment="AzureML-sklearn-0.24-ubuntu18.04-py37-cpu@latest",
    compute="aml-cluster",
    display_name="train-model",
    experiment_name="train-classification-model"
    )

```

## Using Parameters in Command Job

- To use parameters in a script, you must use a library such as argparse to read arguments passed to the script and assign them to variables.

```python

# import libraries
import argparse
import pandas as pd
from sklearn.linear_model import LogisticRegression

def main(args):
    # read data
    df = get_data(args.training_data)

# function that reads the data
def get_data(path):
    df = pd.read_csv(path)
    
    return df

def parse_args():
    # setup arg parser
    parser = argparse.ArgumentParser()

    # add arguments
    parser.add_argument("--training_data", dest='training_data',
                        type=str)

    # parse args
    args = parser.parse_args()

    # return args
    return args

# run script
if __name__ == "__main__":

    # parse args
    args = parse_args()

    # run main function
    main(args)

```

- Any parameters you expect should be defined in the script. In the script, you can specify what type of value you expect for each parameter and whether you want to set a default value.

- After submitting a command job, you can review the input and output parameters you specified.