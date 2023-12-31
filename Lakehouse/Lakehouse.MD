# Create a Lakehouse

## Activate a Microsoft Fabric trial

1. After signing up for a Microsoft Fabric account, navigate to the Microsoft Fabric portal at `https://app.fabric.microsoft.com.`

2. Select the Account Manager icon (the user image at the top right)

3. In the account manager menu, select Start trial to start a Microsoft Fabric trial.

4. After successfully upgrading to Microsoft Fabric, navigate to the home page by selecting Fabric Home Page.

## Create a Workspace

1. Before working with data in Fabric, create a workspace with the Fabric trial enabled.

2. On the Microsoft Fabric home page, select Synapse Data Engineering.

3. In the menu bar on the left, select Workspaces (the icon looks similar to 🗇).

4. Create a new workspace with a name of your choice, selecting a licensing mode that includes Fabric capacity (Trial, Premium, or Fabric).

5. When your new workspace opens, it should be empty.

## Create a notebook

- To run code, you can create a notebook. Notebooks provide an interactive environment in which you can write and run code (in multiple languages).

1. In the Synapse Data Science home page, create a new Notebook.

- After a few seconds, a new notebook containing a single cell will open. Notebooks are made up of one or more cells that can contain code or markdown (formatted text).

2. Select the first cell (which is currently a code cell), and then in the dynamic tool bar at its top-right, use the M↓ button to convert the cell to a markdown cell.

- When the cell changes to a markdown cell, the text it contains is rendered.

3. Use the (Edit) button to switch the cell to editing mode, then delete the content.

## Get the Data

- After loading the data, you’ll convert the data to a Pandas dataframe: a common structure for working with data in rows and columns.

*check the diabetes.ipynb for code implementation in notebook*

- (Run only first two cells)

- There are two tabs at the top of the rendered table: Table and Chart. Select Chart.

- Select the View options at the right top of the chart to change the visualization.

- Change the chart to the following settings:
  - Chart Type: Box plot
  - Key: Leave empty
  - Values: Y

- Select Apply to render the new visualization and explore the output.

## Prepare the Data

1. The data is loaded as a Spark dataframe. To launch the Data Wrangler, you need to convert the data to a Pandas dataframe. Run the following code in your noteboo

2. Select Data in the notebook ribbon, and then select Launch Data Wrangler dropdown.

3. Select the df dataset. When Data Wrangler launches, it generates a descriptive overview of the dataframe in the Summary panel.

- Currently, the label column is Y, which is a continuous variable. To train a machine learning model that predicts Y, you need to train a regression model. The (predicted) values of Y may be difficult to interpret. Instead, we could explore training a classification model which predicts whether someone is low risk or high risk for developing diabetes. To be able to train a classification model, you need to create a binary label column based on the values from Y.

4. Select the Y column in the Data Wrangler. Note that there is a decrease in frequency for the 220-240 bin. The 75th percentile 211.5 roughly aligns with transition of the two regions in the histogram. Let’s use this value as the threshold for low and high risk.

5. Navigate to the Operations panel, expand Formulas, and then select Create column from formula.

6. Create a new column with the following settings:
 - Column name: Risk
 - Column formula: (df['Y'] > 211.5).astype(int)

7. Review the new column Risk that is added to the preview. Verify that the count of rows with value 1 should be roughly 25% of all rows (as it’s the 75th percentile of Y).

8. Select Apply.

9. Select Add code to notebook.

10. Run the cell with the code that is generated by Data Wrangler.

## Explore Your Experiments

- Microsoft Fabric will keep track of all your experiments and allows you to visually explore them.

1. Navigate to your workspace from the hub menu bar on the left.

2. Select the diabetes-regression experiment to open it.

3. Review the Run metrics to explore accurate your regression model is.

4. Navigate back to the home page and select the diabetes-classification experiment to open it.

5. Review the Run metrics to explore the accuracy of the classification model. Note that the type of metrics are different as you trained a different type of model.

## Save the model

- After comparing machine learning models that you’ve trained across experiments, you can choose the best performing model. To use the best performing model, save the model and use it to generate predictions.

1. Select Save in the Save as model box.

2. Select Create a new model in the newly opened pop-up window.

3. Select the model folder.

4. Name the model model-diabetes, and select Save.

5. Select View model in the notification that appears at the top right of your screen when the model is created.You can also refresh the window. The saved model is linked under Model versions.

- Note that the model, the experiment, and the experiment run are linked, allowing you to review how the model is trained.

## Save the notebook and end the Spark session

- Now that you’ve finished training and evaluating the models, you can save the notebook with a meaningful name and end the Spark session.

1. In the notebook menu bar, use the Settings icon to view the notebook settings.

2. Set the Name of the notebook to Train and compare models, and then close the settings pane.

3. On the notebook menu, select Stop session to end the Spark session.

## Clean up resources

1. In the bar on the left, select the icon for your workspace to view all of the items it contains.

2. In the … menu on the toolbar, select Workspace settings.

3. In the Other section, select Remove this workspace 