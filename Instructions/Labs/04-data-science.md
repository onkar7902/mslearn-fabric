# Use notebooks to train a model in Microsoft Fabric

In this lab, we will use Microsoft Fabric to create a notebook and train a machine-learning model to predict customer churn. We will use Scikit-Learn to train the model and MLflow to track its performance. Customer churn is a critical business problem that many companies face, and predicting which customers are likely to churn can help companies retain their customers and increase revenue. By completing this lab, you will gain hands-on experience in machine learning and model tracking, and learn how to use Microsoft Fabric to create a notebook for your projects.

This lab will take approximately **45** minutes to complete.

## Create a lakehouse and upload files

Using the same workspace, it's time to switch to the *Data science* experience in the portal.

1. Navigate back to your lakehouse, and in the **... (1)** menu for the **Files** node in the **Explorer** pane, select **Upload (2)** and **Upload files (3)**. 

   ![](./Images/Pg6-S1.png)

2. Navigate to **C:\LabFiles\Files\churn.csv (1)**, select the **churn.csv (2)** file and click on **Open (3)** to upload it to the lakehouse.   

   ![](./Images/Pg6-S2.png)

3. After the files have been uploaded, expand **Files** and verify that the CSV file has been uploaded.

   ![](./Images/Pg6-S2.1.png)

## Create a notebook

To train a model, you can create a *notebook*. Notebooks provide an interactive environment in which you can write and run code (in multiple languages) as *experiments*.

1. At the bottom left of the Power BI portal, switch to the **Data science** experience.

1. In the **Data science** home page, create a new **Notebook**.

    After a few seconds, a new notebook containing a single *cell* will open. Notebooks are made up of one or more cells that can contain *code* or *markdown* (formatted text).

1. Select the first cell (which is currently a *code* cell), and then in the dynamic toolbar at its top-right, use the **M&#8595;** button to convert the cell to a *markdown* cell.

    When the cell changes to a markdown cell, the text it contains is rendered.

1. Use the **&#128393;** (Edit) button to switch the cell to editing mode, then delete the content and enter the following text:

    ```text
   # Train a machine learning model and track with MLflow

   Use the code in this notebook to train and track models.
    ```    

## Load data into a dataframe

Now you're ready to run code to prepare data and train a model. To work with data, you'll use *dataframes*. Dataframes in Spark are similar to Pandas dataframes in Python, and provide a common structure for working with data in rows and columns.

1. In the **Add lakehouse** pane, select **Add** to add a lakehouse.

   ![](./Images/Pg6-Edit-S4.png)

1. Select **Existing lakehouse (1)** and select **Add (2)**.

   ![](./Images/Pg6-AddLakehouse.png)

1. Select the lakehouse you created in a previous section.

1. Expand the **Files (1)** folder so that the CSV file is listed next to the notebook editor.

1. In the **...** menu for **churn.csv (2)**, select **Load data (3)** > **Pandas (4)**.

    ![](./Images/Pg6-LoadData-S5.png)

1.  A new code cell containing the following code should be added to the notebook:

    ```Python
    import pandas as pd
    # Load data into pandas DataFrame from "/lakehouse/default/" + "Files/churn.csv"
    df = pd.read_csv("/lakehouse/default/" + "Files/churn.csv")
    display(df)
    ```
    
    > **Tip**: You can hide the pane containing the files on the left by using its **<<** icon. Doing so will help you focus on the notebook.

1. Use the **&#9655; Run cell** button on the left of the cell to run it.

    > **Note**: Since this is the first time you've run any Spark code in this session, the Spark pool must be started. This means that the first run in the session can take a minute or so to complete. Subsequent runs will be quicker.

1. When the cell command has been completed, review the output below the cell, which should look similar to this:

    |Index|CustomerID|years_with_company|total_day_calls|total_eve_calls|total_night_calls|total_intl_calls|average_call_minutes|total_customer_service_calls|age|churn|
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    |1|1000038|0|117|88|32|607|43.90625678|0.810828179|34|0|
    |2|1000183|1|164|102|22|40|49.82223317|0.294453889|35|0|
    |3|1000326|3|116|43|45|207|29.83377967|1.344657937|57|1|
    |4|1000340|0|92|24|11|37|31.61998183|0.124931779|34|0|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    The output shows the rows and columns of customer data from the churn.csv file.

## Train a machine learning model

Now that you've loaded the data, you can use it to train a machine-learning model and predict customer churn. You'll train a model using the Scikit-Learn library and track the model with MLflow. 

1. Use the **+ Code** icon below the cell output to add a new code cell to the notebook, and enter the following code in it:

    ```python
   from sklearn.model_selection import train_test_split

   print("Splitting data...")
   X, y = df[['years_with_company','total_day_calls','total_eve_calls','total_night_calls','total_intl_calls','average_call_minutes','total_customer_service_calls','age']].values, df['churn'].values
   
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. Run the code cell you added, and note you're omitting 'CustomerID' from the dataset, and splitting the data into a training and test dataset.
1. Add another new code cell to the notebook, enter the following code in it, and run it:
    
    ```python
   import mlflow
   experiment_name = "experiment-churn"
   mlflow.set_experiment(experiment_name)
    ```
    
    The code creates an MLflow experiment named `experiment-churn`. Your models will be tracked in this experiment.

1. Add another new code cell to the notebook, enter the following code in it, and run it:

    ```python
   from sklearn.linear_model import LogisticRegression
   
   with mlflow.start_run():
       mlflow.autolog()

       model = LogisticRegression(C=1/0.1, solver="liblinear").fit(X_train, y_train)

       mlflow.log_param("estimator", "LogisticRegression")
    ```
    
    The code trains a classification model using Logistic Regression. Parameters, metrics, and artifacts are automatically logged with MLflow. Additionally, you're logging a parameter called `estimator`, with the value `LogisticRegression`.

1. Add another new code cell to the notebook, enter the following code in it, and run it:

    ```python
   from sklearn.tree import DecisionTreeClassifier
   
   with mlflow.start_run():
       mlflow.autolog()

       model = DecisionTreeClassifier().fit(X_train, y_train)
   
       mlflow.log_param("estimator", "DecisionTreeClassifier")
    ```

    The code trains a classification model using a Decision Tree Classifier. Parameters, metrics, and artifacts, are automatically logged with MLflow. Additionally, you're logging a parameter called `estimator`, with the value `DecisionTreeClassifier`.

## Use MLflow to search and view your experiments

When you've trained and tracked models with MLflow, you can use the MLflow library to retrieve your experiments and their details.

1. To list all experiments, use the following code:

    ```python
   import mlflow
   experiments = mlflow.search_experiments()
   for exp in experiments:
       print(exp.name)
    ```

1. To retrieve a specific experiment, you can get it by its name:

    ```python
   experiment_name = "experiment-churn"
   exp = mlflow.get_experiment_by_name(experiment_name)
   print(exp)
    ```

1. Using an experiment name, you can retrieve all jobs of that experiment:

    ```python
   mlflow.search_runs(exp.experiment_id)
    ```

1. To more easily compare job runs and outputs, you can configure the search to order the results. For example, the following cell orders the results by `start_time`, and only shows a maximum of `2` results: 

    ```python
   mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    ```

1. Finally, you can plot the evaluation metrics of multiple models next to each other to easily compare models:

    ```python
   import matplotlib.pyplot as plt
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_accuracy_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_accuracy_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("Accuracy")
   ax.set_title("Accuracy by Estimator")
   for i, v in enumerate(df_results["metrics.training_accuracy_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    The output should resemble the following image:

    ![Screenshot of the plotted evaluation metrics.](./Images/plotted-metrics.png)

## Explore your experiments

Microsoft Fabric will keep track of all your experiments and allows you to visually explore them.

1. Navigate to your **Workspace (1)**, select **Data Science (2)**  you will see the **experiment-churn (3)** Experiment created.

   ![](./Images/Pg6-ExpChurn-S1.png)

1. Select the `experiment-churn` experiment to open it.

    > **Tip:**
    > If you don't see any logged experiment runs, refresh the page.

1. Select the **View (1)** tab.

1. Select **Run list (2)**. 

1. Select the **two latest runs (3)** by checking each box.
    As a result, your two last runs will be compared to each other in the **Metric comparison** pane. By default, the metrics are plotted by run name. 

1. Select the **&#128393;** **(Edit) (4)** button of the graph visualizing the accuracy for each run. 

   ![](./Images/Pg6-ExpChurn-S6.png)

1. Change the **visualization type** to `bar`. 

1. Change the **X-axis** to `estimator`. 

1. Select **Replace** and explore the new graph.

By plotting the accuracy per logged estimator, you can review which algorithm resulted in a better model.

## Save the model

After comparing machine learning models that you've trained across experiment runs, you can choose the best-performing model. To use the best-performing model, save the model and use it to generate predictions.

1. In the experiment overview, ensure the **View** tab is selected.

1. Select **Run details**.

1. Select the run with the highest accuracy. 

1. Scroll right to see the Save as model option. Select **Save** in the **Save run as an ML model** box.

   ![Empty data pipeline.](./Images/36.png)

1. Select **Create a new model** in the newly opened pop-up window.

1. Name the model `model-churn`, and select **Save**. 

1. Select **View model** in the notification that appears at the top right of your screen when the model is created. You can also refresh the window. The saved model is linked under **Registered version**. 

Note that the model, the experiment, and the experiment run are linked, allowing you to review how the model is trained. 

## Save the notebook and end the Spark session

Now that you've finished training and evaluating the models, you can save the notebook with a meaningful name and end the Spark session.

1. In the notebook menu bar, use the ⚙️ **Settings** icon to view the notebook settings.

2. Set the **Name** of the notebook to **Train and compare models notebook**, and then close the settings pane.

3. On the notebook menu, select **Stop session** to end the Spark session.

---

In this exercise, you have created a notebook and trained a machine-learning model. You used Scikit-Learn to train the model and MLflow to track its performance.

---
Continue to page 7 ...
