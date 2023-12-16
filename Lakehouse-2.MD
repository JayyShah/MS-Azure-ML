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

## Create a Lakehouse

- Now that you have a workspace, it's time to create a data lakehouse for your data files

1. In the Synapse Data Engineering home page, create a new Lakehouse with a name of your choice.After a minute or so, a new lakehouse will be created

2. View the new lakehouse, and note that the Lakehouse explorer pane on the left enables you to browse tables and files in the lakehouse:

 - The Tables folder contains tables that you can query using SQL semantics. Tables in a Microsoft Fabric lakehouse are based on the open source Delta Lake file format, commonly used in Apache Spark.

 - The Files folder contains data files in the OneLake storage for the lakehouse that aren't associated with managed delta tables. You can also create shortcuts in this folder to reference data that is stored externally.

- Currently, there are no tables or files in the lakehouse.

## Upload a file

- Fabric provides multiple ways to load data into the lakehouse, including built-in support for pipelines that copy data external sources and data flows (Gen 2) that you can define using visual tools based on Power Query. However one of the simplest ways to ingest small amounts of data is to upload files or folders from your local computer (or lab VM if applicable).

1. In the menu for the Files folder in the Lakehouse explorer pane, select New subfolder, and create a subfolder named data.

2. In the menu for the new data folder, select Upload and Upload file, and then upload the data (csv) file from your local computer (or lab VM if applicable).

3. After the file has been uploaded, select the Files/data folder and verify that the sales.csv file has been uploaded

4. Select the file to see a preview of its contents.

## Explore shortcuts

- In many scenarios, the data you need to work with in your lakehouse may be stored in some other location. While there are many ways to ingest data into the OneLake storage for your lakehouse, another option is to instead create a shortcut. Shortcuts enable you to include externally sourced data in your analytics solution without the overhead and risk of data inconsistency associated with copying it.

  1. In the ... menu for the Files folder, select New shortcut.

  2. View the available data source types for shortcuts. Then close the New shortcut dialog box without creating a shortcut.

## Load file data into a table

- The data you uploaded is in a file, which data analysts and engineers can work with directly by using Apache Spark code. However, in many scenarios you may want to load the data from the file into a table so that you can query it using SQL.

- On the Home page, select the Files/Data folder so you can see the sales.csv file it contains.

- In the menu for the file, select Load to Tables.

- In Load to table dialog box, set the table name to sales and confirm the load operation. Then wait for the table to be created and loaded.

- In the Lakehouse explorer pane, select the table that has been created to view the data.

- In the menu for the table, select View files to see the underlying files for this table

- Files for a delta table are stored in Parquet format, and include a subfolder named _delta_log in which details of transactions applied to the table are logged.

## Use SQL to query tables

- When you create a lakehouse and define tables in it, a SQL endpoint is automatically created through which the tables can be queried using SQL SELECT statements.

1. At the top-right of the Lakehouse page, switch from Lakehouse to SQL endpoint. Then wait a short time until the SQL query endpoint for your lakehouse opens in a visual interface from which you can query its tables.

2. Use the New SQL query button to open a new query editor, and enter the following SQL query:

```bash

SELECT Item, SUM(Quantity * UnitPrice) AS Revenue
FROM sales
GROUP BY Item
ORDER BY Revenue DESC;

```

3. Use the Run button to run the query and view the results, which should show the total revenue for each product.

## Create a Visual Query

- While many data professionals are familiar with SQL, data analysts with Power BI experience can apply their Power Query skills to create visual queries.

1. On the toolbar, select New visual query.

2. Drag the sales table to the new visual query editor pane that opens to create a Power Query

3. In the Manage columns menu, select Choose columns. 

- When you're done, the results pane under the visual query shows the result.

## Create a Report

The tables in your lakehouse are automatically added to a default dataset that defines a data model for reporting with Power BI.

1. At the bottom of the SQL Endpoint page, select the Model tab. The data model schema for the dataset is shown.

2. In the menu ribbon, select the Reporting tab. Then select New report. A new browser tab opens in which you can design your report.

3. In the Data pane on the right, expand the table. Then select your required fields:

- A table visualization is added to the report.

4. Hide the Data and Filters panes to create more space. Then ensure the table visualization is selected and in the Visualizations pane, change the visualization to a Clustered bar chart.

5. On the File menu, select Save. Then save the report as *name* Report in the workspace you created previously.

6. Close the browser tab containing the report to return to the SQL endpoint for your lakehouse. Then, in the hub menu bar on the left, select your workspace to verify that it contains the following items:

 1. Your lakehouse.
 2. The SQL endpoint for your lakehouse.
 3. A default dataset for the tables in your lakehouse.
 4. The *name* Report report.

## Clean up resources

1. In the bar on the left, select the icon for your workspace to view all of the items it contains.

2. In the … menu on the toolbar, select Workspace settings.

3. In the Other section, select Remove this workspace 
