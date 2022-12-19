# TITANIC CLASSIFICATION EXAMPLE

- This example supports 3 dataset sources i.e. **Local, AWS S3 and SQL**. 
- By default this example uses local data source.
  - Local data source is created in DKube storage space
- The notebooks in this example can be run inside or outside Dkube.

## Example Flow
- Create DKube resources. This includes Dataset and Model repo resources.
- (Optional) Train a model for titanic example using Tensorflow and deploy the model for inference.
- Generate data for analysis by Modelmonitor
  - Predict data: Inference inputs/outputs
  - Label data:  Dataset with Groundtruth values for the inferences made above
  **In production, the predict data would be logged by the deployed model and Label data would be generated by experts in the domain manually.**
- Create a Modelmonitor. 
  - There is no requirement to deploy a model for this example
- (Optional) Retrain the model and update Modelmonitor
- Cleanup resources after the example is complete


## Prerequisites
- For Aws-S3 **(S3 bucket is required)**
  - Create an AWS S3 bucket with the name mm-workflow. 
  - You need access and secret keys to access the bucket.
- For SQL **(SQL database is required)**. 
  - You need the following to access the SQL Database
    - username
    - password
    - hostaddress (server address)
    - portnumber
    - databasename


## Section 1: Create Dkube Resources

### Launch IDE (Inside Dkube)

#### Note: Follow the instructions if you are running Notebook IDE inside DKube. In case you are Notebook IDE outside DKube then clone the repo and checkout to monitoring branch and follow from step 4 of this section.

1. Add Code. Create Code Repo in Dkube with the following information
  - Name: monitoring-examples
  - Source: Git
  - URL: https://github.com/oneconvergence/dkube-examples.git
  - Branch : monitoring-v3
2. Create an IDE (JupyterLab)
   - Use Tensorflow framework with version 2.0.0
   - **If your data is in local**, move to step 3 directly.
   - **If your data is in aws-s3:**
     - Add the following environment variables with your secret values in configuration tab 
       - AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
   - **If your data is in SQL:**DBHOSTNAME,
     - Add the following environment variables with your secret values in configuration tab
       - DBUSERNAME
       - DBPASSWORD
       - DBHOSTNAME(should be specified in following format ip:port or domain:port)
       - DATABASENAME    
3. Click Submit.
4. From **workspace/monitoring-examples/titanic_datasources** open resources.ipynb and fill the following details in the first cell. 
     - **MODELMONITOR_NAME** = {your model monitor name}
     - **DATASET_SOURCE** = { one of your choice in ['local' or 'aws-s3' or 'sql'] }
     - **INPUT_TRAIN_TYPE** = {'training'}
     - **DKUBEUSERNAME** = {your dkube username}
     - The following will be derived from the environment automatically. Otherwise, please fill in 
       - **TOKEN** = {your dkube authentication token}
       - **DKUBE_URL** = {your dkube url}
       - If the data source is **aws-s3**, fill the below details also:
         - **ACCESS_KEY** = {your s3 access key}
         - **SECRET_KEY** = {your s3 secret key}
       - If the data source is **sql**, fill the below details also:
         - **DBHOSTNAME** = {sql server hostname}
         - **DATABASENAME** = {sql database name} 
         - **DBUSERNAME** = {your username}
         - **DBPASSWORD** = {your password}
         - **DB_PROVIDER** = {provider} (Two values are supported mysql and mssql, default value is mysql)
     - Modelmonitor run frequency in minutes. The same run interval is used for both Drift & Performance monitoring
         - **RUN_FREQUENCY** = {integer value. units are minutes}
5. Run all the cells. This will create all the dkube resources required for this example automatically.
6. Once all the cells complete the run you will see the following resources will get created,
   1. `titanic-data` dataset.
   2. `titanic-training-data` dataset.
   3. `titanic-model` model.
   4. `titanic-mm-{DATA_SOURCE}-predict` and `titanic-mm-{DATA_SOURCE}-groundtruth` datasets.

## Section 2: Titanic Model Training (Optional)

#### Note: This uses DKube Runs, Kubeflow Pipelines and KfServing. If your DKube configuration doesn't support this, please skip this step and go to Modelmonitoring Section. This is the case with minimal DKube.

1. From **workspace/monitoring-examples/titanic_datasources** open **train.ipynb** to build the pipeline.
2. The pipeline includes preprocessing, training and serving stages. Run all cells
     - **preprocessing**: the preprocessing stage generates the dataset (either training-data or retraining-data) depending on user choice.
     - **training**: the training stage takes the generated dataset as input, train a model and outputs the model.
     - **serving**: The serving stage takes the generated model and serve it with a predict endpoint for inference. 
3. Verify that the pipeline has created the following resources
     - Datasets: 'titanic-training-data' with version v2.
     - Model: 'titanic-model' with version v2

### Inference
1. Navigate to the model (titanic-model) and click on test inference.
2. Give the test inference name, say titanic.
3. The serving image is ocdr/tensorflowserver:2.0.0.
4. Check transformer option, and type the transformer script as titanic/transformer.py
5. Choose CPU, and submit.
6. Go to `https://<URL>:32222/inference`
   - Copy the model serving URL from the test inference tab.  
   - Copy the auth token from developer settings  
   - Select model type sk-stock  
   - Copy the contents of https://raw.githubusercontent.com/oneconvergence/dkube-examples/tensorflow/titanic/titanic_sample.csv and save then as CSV, and upload.  
   - Click predict.


## Section 3: Data Generation
1. Open [data_generation.ipynb](https://github.com/oneconvergence/dkube-examples/tree/monitoring/titanic_datasources/data_generation.ipynb) notebook for generating predict and groundtruth datasets.
2. In 1st cell, Update Frequency according to what you set in Modelmonitor. For eg: for 5 minutes, specify it as `5m`.
3. Then Run All Cells. It will start Pushing the data. It uses the data definitions specified in resources.ipynb file.

## Section 4: Modelmonitoring
DKube provides Python SDK for creating a modelmonitor programmatically. You could also choose to create a modelmonitor from the DKube UI. Follow one of the following workflows as per your need.

- **Create Modelmonitor using SDK**
1. From **workspace/monitoring-examples/titanic_datasources** open [modelmonitor.ipynb](https://github.com/oneconvergence/dkube-examples/tree/monitoring/titanic_datasources/modelmonitor.ipynb) and run all the cells. New model monitor will be created.
2. Predict and Groundtruth datasets will be generated by Data Generation step and will be utilised by modelmonitor.
3. After the completion of the notebook, you will see the model monitor `titanic-mm-{DATA_SOURCE}` in active state.

- **Create Modelmonitor using UI**
  - Follow [README.ui.md](https://github.com/oneconvergence/dkube-examples/blob/monitoring/titanic_datasources/README.ui.md) for the next steps.


## Section 5: Retrain (Optional)
1. Open [resources.ipynb](https://github.com/oneconvergence/dkube-examples/tree/monitoring/titanic_datasources/resources.ipynb) and set INPUT_TRAIN_TYPE = 'retraining' in the 1st cell and run all the cells.
2. Open train.ipynb and run all the cells.
3. This creates a new version of dataset and a new version of model
   - New dataset version will be created for 'titanic-training-data' dataset
   - New model version will be created for 'titanic-model' model
   - Follow step 4 if you want to retrain via sdk or step 5 if you want to retrain using UI.
4. **SDK**:
   - From **workspace/monitoring-examples/titanic_datasources** open modelmonitor.ipynb and run the Retraining cell. It will update the dataset and model version in the existing model monitor.
5. **UI**:
   - Edit modelmonitor (UI)
   - Specify the new model version on basic page
   - Specify new dataset version on Training data page
   - Save & Submit
   - Click Next to go to the schema page and Accept the regenerated schema.
   - Wait for a few (30) sec
   - Start the modelmonitor

## Section 6: SMTP Settings (Optional)
Configure your SMTP server settings on Operator screen. This is optional. If SMTP server is not configured, no email alerts will be generated.

## Section 7: Cleanup
1. After your experiment is complete, 
   - Open [resources.ipynb](https://github.com/oneconvergence/dkube-examples/tree/monitoring/titanic_datasources/resources.ipynb) and set CLEANUP=True in last Cleanup cell and run.
   - Open [modelmonitor.ipynb](https://github.com/oneconvergence/dkube-examples/tree/monitoring/titanic_datasources/modelmonitor.ipynb) and set CLEANUP=True in last Cleanup cell and run.
