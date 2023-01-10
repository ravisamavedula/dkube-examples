# Create Monitor for Insurance Prediction Example Automatically

 This section explains how to use a JupyterLab .ipynb file to create a Monitor for the insurance prediction example.

 - This example uses the DKube built-in MinIO server and uses prediction datasets as "CloudEvents"
 - This example assumes that the serving cluster and the model monitoring cluster are the same

## Example Flow
 - Create the DKube resources
 - Train a model for the Insurance example using TensorFlow and deploy the model for inference
 - Create a Model Monitor
   - Although a deployed Model is not required for a Monitor within DKube, this example uses a deployed model
 - Generate data for analysis by the Monitor
   - Predict data: Inference inputs/outputs
   - Label data:  Dataset with Groundtruth values for the inferences made above
 - Cleanup the resources after the example is complete

 > **_Note:_** The labels are generated in the example for purposes of illustration.  In an actual Production environment, the label data would be generated manually by experts in the domain.

## Set up Resources

 You can choose the names for your resources in most cases.  It is recommended that you choose names that are unique to your workflow even if you are organizing them by Project.  This will ensure that there is a system-wide organization for the names, and that you can easily filter based on your own work.  A sensible approach might be to have it be something like **\<example-name\>-\<your-initials\>-\<resource-type\>**.  But this is simply a recommendation.  The specific names will be up to you.

### Create Code & Model Repos

 > **_Note:_** This step may have been completed in an earlier section of this example.  If so, skip the steps here and use the Code & Model repo names that you previously created.  If you need to create a new Code and/or Model repo, follow the instructions at:
 - [Create Code Repo](../readme.md#create-project)
 - [Create Model Repo](../readme.md#create-model-repo)

### Create & Launch JupyterLab IDE

 > **_Note:_** This step may have been completed in an earlier section of this example.  If so, skip the steps here and use the JupyterLab IDE that you previously created.  If you need to create a new IDE, follow the instructions at:
 - [Create JupyterLab IDE](../readme.md#create-jupyterlab-ide) <br><br>
 - Once the IDE is in the "Running" state, select the JupyterLab icon on the far right of the IDE line
   - This will create a JupyterLab tab

### Execute File to Create Resources

 - Navigate to folder <code>/workspace/**\<your-code-repo\>**/insurance/monitoring</code>
 - Open `resources.ipynb`
 - In case of running the example other than the serving setup, In the 1st cell, set RUNNING_IN_SAME to False and Fill in the external IP address for the field `SERVING_DKUBE_URL` in the form "https://\<External IP Address\>:32222/"
   - Ensure that there is a final `/` in the url field
   - Leave the other fields in their current selection
 - From the top menu item `Run`, select `Run All Cells`
 - This will create the DKube resources required for this example to run automatically, including the required Datasets <br><br>
 - The following Datasets will be created
   - `insurance-data`, with a pub_url source
   - A Dataset that includes the username and ends in "-s3", with an "s3 | remote" source

## Train & Deploy Insurance Model
 
 In order to Monitor a Model in this example, it needs to be trained and deployed.
 > **_Note:_** This section requires DKube Runs, Kubeflow Pipelines, and KServe.  It requires a full DKube installation.

 > **_Note:_** This step may have been completed in an earlier section of the example.  If so, you can skip this section and use the deployed Model for the Monitor.  If you need to train and deploy the Model, follow the pipeline instructions at:
 - [Train and Deploy Model](../readme.md#create-kubeflow-pipeline)

 - The Pipeline will create a new Deployment.  It will be at the top of the Deployment list.

## Create Monitor Automatically

 In this example, the Monitor is created programmatically through the DKube SDK. 
 
 > **_Note:_** The script in this section will fail if there is already a Monitor with the automatically-generated name.  This can happen if the script is run more than once.  Delete the Monitor name before you run this script a 2nd time.

 - From the JupyterLab tab, navigate to folder <code>/workspace/**\<your-code-repo\>**/insurance/monitoring</code>
 - Open `modelmonitor.iypnb`
 - **Ensure that the last cell at the bottom of the file has "CLEANUP = False".**  This may have been set to "True" from a previous execution.
 - Run all of the cells
 - This will create a new Model Monitor and put it into the `Active` state
   - The Monitor name will be the same as the Deployment name <br><br>
 - Navigate to the `Deployments` menu on the left
 - Select the `Monitors` tab at the top
 - You will see the new Monitor at the top of the list

## Generate Monitor Data

 In order for the Monitor to operate, predictions and groundtruth Datasets must be generated. 
 
 - Open `data_generation.ipynb`
   - This will create the predictions with the Deployment endpoint and generate the groundtruth Datasets for this example
   - **Ensure that the last cell at the bottom of the file has "CLEANUP = False".**  This may have been set to "True" from a previous execution.
   - In the 1st cell, specify the number of Dataset samples to run before stopping the data generation.  You can leave it at the default, or modify it.  The larger the number of samples, the more data will be generated for the Monitor graphs.
<!---
   - The 3rd cell controls how often the script will run.  The default is 5 min.  If you want to change the frequency, change the variable to another number.
     - An example would be **FREQUENCY = "2m"** to run the script every 2 minutes
--->
   - Leave the other fields at their current selection
   - `Run All Cells`
   - The script will start to push the data

<!--- Not sure if we need to do this

## Section 5: SMTP Settings (Optional)
Configure your SMTP server settings on Operator screen. This is optional. If SMTP server is not configured, no email alerts will be generated.
--->

## View the Monitor

 After the data has been generated for a few data points, it can be viewed within DKube.
 
 - Navigate to the `Deployments` menu on the left
 - Select the `Monitors` tab on the top
 - Your new Monitor will be at the top of the list
 - The details of how to view and understand the Monitor are described at [DKube Monitor Dashboard](https://dkube.io/monitor/monitor3_x/Monitor_Workflow.html#monitor-dashboard)

## Cleanup
 After the experiment is complete, the following cleanup should be performed in order to delete the Datasets and stop the Monitor:
 
 - Within `modelmonitor.ipynb`, set the variable "CLEANUP = True" in the last cell
   - Run the "Cleanup" cell
 - Within `resources.ipynb`, set the variable "CLEANUP = True" in the last cell
   - Run the "Cleanup" cell

