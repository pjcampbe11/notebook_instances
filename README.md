# Amazon SageMaker Notebook Instance Setup and Configuration Guide

This guide provides step-by-step instructions to set up an Amazon SageMaker notebook instance and configure it to run a Jupyter notebook targeting a SageMaker endpoint. Additionally, it covers the permissions granted to running SageMaker notebook instances and access to the Jupyter server.

## Step 1: Create an Amazon SageMaker Notebook Instance

1. **Open the SageMaker Console**
   - Navigate to the [Amazon SageMaker console](https://console.aws.amazon.com/sagemaker/).

2. **Create a Notebook Instance**
   - In the SageMaker console, choose `Notebook instances` from the left navigation pane, then click `Create notebook instance`.
   - Provide a **Notebook instance name**.
   - Choose **Notebook instance type** (e.g., `ml.t2.medium` for small workloads).
   - For **Platform identifier**, choose the appropriate platform type (e.g., `notebook-al2-v1` for Amazon Linux 2 with JupyterLab 1).
   - For **IAM role**, choose `Create a new role`. This role should have permissions to access S3 and SageMaker.
   - Leave other options at their default values and click `Create notebook instance`.

3. **Wait for the Notebook Instance to be Created**
   - This process might take a few minutes. The status will change to `InService` once it’s ready.

## Step 2: Open the Jupyter Notebook

1. **Open JupyterLab**
   - In the SageMaker console, under `Notebook instances`, find your instance and choose `Open JupyterLab`.

## Step 3: Upload Your Notebook

1. **Upload the `.ipynb` File**
   - In JupyterLab, click the upload button (an arrow pointing upwards) and select the `pytorch_sentiment_analysis.ipynb` file to upload.

## Step 4: Configure the Notebook to Target a SageMaker Endpoint

1. **Set Up the Environment**
   - Ensure that your notebook has the required dependencies. You might need to install additional packages:
     ```python
     !pip install sagemaker boto3
     ```

2. **Load and Configure the Notebook**
   - Open the `pytorch_sentiment_analysis.ipynb` notebook in JupyterLab.
   - Modify the notebook to include the following steps for interacting with SageMaker.

3. **Connect to SageMaker and S3**
   - Add the necessary code to set up the SageMaker session and roles:
     ```python
     import sagemaker
     from sagemaker import get_execution_role

     role = get_execution_role()
     session = sagemaker.Session()
     bucket = session.default_bucket()
     ```

4. **Upload Data to S3**
   - If your notebook requires data, upload it to S3:
     ```python
     import boto3

     s3 = boto3.resource('s3')
     data = open('path_to_your_data', 'rb')
     s3.Bucket(bucket).put_object(Key='data/your_data_file', Body=data)
     ```

5. **Define the Model and Endpoint Configuration**
   - If you have a pre-trained model to deploy, set up the model and endpoint configuration:
     ```python
     from sagemaker.pytorch import PyTorchModel

     model = PyTorchModel(model_data='s3://{}/path_to_your_model'.format(bucket),
                          role=role,
                          framework_version='1.5.0',
                          entry_point='inference.py')  # Ensure you have an inference script

     predictor = model.deploy(initial_instance_count=1, instance_type='ml.m4.xlarge')
     ```

6. **Invoke the Endpoint**
   - Add code to invoke the endpoint and make predictions:
     ```python
     import numpy as np

     predictor = sagemaker.predictor.Predictor(endpoint_name='your_endpoint_name',
                                               serializer=sagemaker.serializers.JSONSerializer(),
                                               deserializer=sagemaker.deserializers.JSONDeserializer())

     test_data = np.array([your_test_data])
     predictions = predictor.predict(test_data)
     print(predictions)
     ```

## Step 5: Save and Run the Notebook

1. **Save the Notebook**
   - Ensure you save your notebook after making the necessary changes.

2. **Run the Notebook**
   - Execute the cells in your notebook to ensure everything is configured correctly and the predictions are made using the SageMaker endpoint.

## Step 6: Clean Up Resources

1. **Terminate the Endpoint**
   - Once done, ensure you delete the endpoint to avoid unnecessary costs:
     ```python
     predictor.delete_endpoint()
     ```

2. **Stop the Notebook Instance**
   - In the SageMaker console, stop the notebook instance when not in use to save costs.

## Permissions for Running Amazon SageMaker Notebook Instances

Amazon SageMaker notebook instances are fully managed Amazon EC2 instances running the Jupyter Notebook application. These instances are pre-configured with several tools and libraries for machine learning tasks, such as the Amazon SageMaker Python SDK, AWS SDK for Python (Boto3), AWS Command Line Interface (AWS CLI), and various data science and machine learning libraries.

### Default Permissions

1. **IAM Role**: When creating a SageMaker notebook instance, you need to assign an IAM role to it. If you create a new role, the AWS managed policy `AmazonSageMakerFullAccess` is attached to it. This role provides permissions for the notebook instance to interact with SageMaker and Amazon S3.
   - **Permissions included**:
     - Full access to SageMaker resources (e.g., creating training jobs, deploying models, etc.)
     - Access to S3 buckets that have "sagemaker" in their name
     - Permissions to add tags to resources

2. **Root Access**: By default, you can choose to enable or disable root access for all notebook instance users. Root access allows users to have administrator privileges on the instance and can access and edit all files.

3. **Network Configuration**:
   - You can configure the notebook instance to be part of a Virtual Private Cloud (VPC) for additional security.
   - Optionally, enable direct internet access for the notebook instance.

4. **Lifecycle Configuration (LCC) Scripts**:
   - These scripts allow you to run custom shell scripts when creating or starting the notebook instance, providing flexibility to install additional packages or configure the environment.

### Access to the Jupyter Server Running the Notebook

1. **Access Methods**:
   - **Console**: Access the notebook instance via the SageMaker console by choosing either `Open Jupyter` or `Open JupyterLab`.
   - **API**: Use the `CreatePresignedNotebookInstanceUrl` API to get a URL that opens the notebook instance.

2. **Directory Structure**:
   - The primary working directory is `/home/ec2-user/SageMaker`. Files and data saved here persist between sessions.
   - The `/tmp` directory provides temporary storage, which is cleared when the instance is stopped or restarted.
   - NVMe instance store volumes, if supported, are automatically attached but do not provide persistent storage and must be reconfigured every time the instance is launched.

3. **Example Notebooks**:
   - SageMaker provides example notebooks demonstrating various machine learning tasks. These examples can be accessed and used as a starting point for your own projects.

### Key Points to Remember

- **Security and Permissions**:
  - Ensure the IAM role has appropriate permissions to access necessary AWS resources.
  - Regularly review and update the IAM policies attached to the notebook instance role to follow the principle of least privilege.

- **Network Configuration**:
  - Use VPCs for better security and control over the network traffic to and from the notebook instance.
  - Be cautious when enabling direct internet access to minimize security risks.

- **Maintenance**:
  - Stop and restart the notebook instance periodically to apply updates and patches provided by SageMaker.
  - Use lifecycle configuration scripts to automate the setup and configuration of the notebook environment.

This setup ensures that your SageMaker notebook instance is secure, properly configured, and ready for machine learning tasks, while also providing you with the necessary access and control over the Jupyter server running your notebooks.
"""
