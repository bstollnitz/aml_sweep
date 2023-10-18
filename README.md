
# Hyperparameter Tuning in Azure Machine Learning

This project shows how to train a Fashion MNIST model with, by doing hyperparameter tuning using an Azure ML sweep job, and how to deploy it using an online managed endpoint. It uses MLflow for tracking and model representation.

## Blog Post

To learn more about the code in this repo, check out the accompanying blog post: [Hyperparameter Tuning with Azure ML Sweep Job](https://bea.stollnitz.com/blog/aml-sweep/)

## Setup

To replicate this project, follow the steps below:

1. **Azure Subscription**: You'll need an [Azure subscription](https://azure.microsoft.com/en-us/free) to proceed.

2. **Resource Group**: Create a [resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal).

3. **Machine Learning Workspace**: Establish a new machine learning workspace as outlined in the ["Create the Workspace" section](https://docs.microsoft.com/en-us/azure/machine-learning/quickstart-create-resources) of the Azure ML documentation. Note that you'll be creating a "machine learning workspace" Azure resource, not a "workspace" Azure resource.

4. **Azure CLI**: Install the Azure CLI by following the instructions in the [official documentation](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

5. **Azure ML Extension**: Extend the Azure CLI with the ML extension by following the steps outlined in the ["Installation" section](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-configure-cli) of the Azure ML documentation.

6. **Conda Environment**: Set up and activate the Conda environment by executing the commands below:

   ```bash
   conda env create -f environment.yml
   conda activate aml_sweep
   ```

7. **Visual Studio Code (VS Code)**:
   - In VS Code, navigate to the Command Palette using "Ctrl + Shift + P," search for "Python: Select Interpreter," and choose the environment corresponding to the project's name.
   - Log in to Azure using the Azure CLI: `az login --use-device-code`.
   - Set your default subscription: `az account set -s "<YOUR_SUBSCRIPTION_NAME_OR_ID>"`.
   - Set your default resource group and workspace: `az configure --defaults group="<YOUR_RESOURCE_GROUP>" workspace="<YOUR_WORKSPACE>"`.

8. **Azure Machine Learning Studio**: Open the [Azure Machine Learning studio](https://ml.azure.com/) to manage your machine learning resources.

9. **Azure Machine Learning Extension for VS Code**: Install the [Azure Machine Learning extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai) and sign in to your Azure account through the extension.

## Local Training and Inference

1. **Training**:
   - In VS Code's left navigation, select "Run and Debug," then choose the "Train locally" run configuration and press F5.
   - Analyze logged metrics using MLflow UI: `mlflow ui`.

2. **Local Prediction**:
   - Perform local predictions using the trained MLflow model with CSV or JSON files:
   
     ```bash
     cd aml_sweep
     mlflow models predict --model-uri "model" --input-path "test_data/images.csv" --content-type csv --env-manager local
     mlflow models predict --model-uri "model" --input-path "test_data/images.json" --content-type json --env-manager local
     ```

## Cloud-Based Training and Deployment

1. **Create Compute Cluster**:
   - Create a compute cluster using the provided YAML configuration:
   
     ```bash
     az ml compute create -f cloud/cluster-cpu.yml
     ```

2. **Create Dataset**:
   - Create a dataset using the provided YAML configuration:
   
     ```bash
     az ml data create -f cloud/data.yml
     ```

3. **Run Training Job**:
   - Run the training job using the provided YAML configuration and capture the run ID:
   
     ```bash
     run_id=$(az ml job create -f cloud/sweep-job.yml --query name -o tsv)
     ```

4. **Azure ML Model Creation**:
   - Create an Azure ML model from the output of the training job:
   
     ```bash
     az ml model create --name model-sweep --version 1 --path "azureml://jobs/$run_id/outputs/model_dir" --type mlflow_model
     ```

5. **Create and Deploy Endpoint**:
   - Create an endpoint using the provided YAML configuration:
   
     ```bash
     az ml online-endpoint create -f cloud/endpoint.yml
     az ml online-deployment create -f cloud/deployment.yml --all-traffic
     ```

6. **Invoke Endpoint**:
   - Invoke the endpoint for predictions:
   
     ```bash
     az ml online-endpoint invoke --name endpoint-sweep --request-file test_data/images_azureml.json
     ```

7. **Cleanup**:
   - Delete the endpoint to prevent charges:
   
     ```bash
     az ml online-endpoint delete --name endpoint-sweep -y
     ```

## Additional Resources

For further information, refer to the following resources:

- [Sweep Job YAML Schema](https://docs.microsoft.com/en-us/azure/machine-learning/reference-yaml-job-sweep?WT.mc_id=aiml-44167-bstollnitz)
- [Hyperparameter Tuning in Azure ML](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-tune-hyperparameters?WT.mc_id=aiml-44167-bstollnitz)
  
