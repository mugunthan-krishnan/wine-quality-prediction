**Wine Quality Prediction Application**

**UCID: mk2246**

**Name: Mugunthan Krishnan**

GitHub Repo - https://github.com/mugunthankrishnan/cs643/tree/main/Programming%20Assignment%202

Docker Hub - https://hub.docker.com/r/mugunthankrishnan/mlapplication_pa2

**Environment Set-up:**

1. EMR Cluster Setup:

    General Configuration: Cluster Name - Enter a cluster name. Example: "ml-cluster"
    
    Software Configuration:
    
    ![image](https://user-images.githubusercontent.com/123665839/235271305-eb5aed8a-d9f6-43d4-89bb-0373be712db0.png)
    
    Cluster Nodes and Instances:
    
    Choose 1 master and 3 slaves. The slaves are only 3 because AWS Academy has a limit of only 9 EC2 instances running in parallel.
    
    ![image](https://user-images.githubusercontent.com/123665839/235271345-8e85f04e-0108-4cbe-ab17-c88914cd49c8.png)

    Disable Auto-termination.
    
    Security Options:
    
    Create a new EC2 key pair for the EMR Cluster and include that here.
    
    Permissions:
    
    EMR_Role - EMR_DefaultROle
    
    EMR Instance Profile - EMR_EC2_DefaultRole
    
2. EC2 Configuration:
    
    1. In the EC2 instances created for the cluster - choose the Master EC2 instance and an inbound rule for the security group for SSH the custom ip address for the device.
    
    2. Login to the EC2 instance via Powershell using the Public DNS via SSH command and the EC2 Key pair created in the previous step.
    
    3. Configure the AWS Credentials and AWS Session Token.
    
    4. In the shell, enter sudo yum update.
    
    5. Install Flintrock: pip3 install flintrock.
    
    6. Add flintrock installation directory to the PATH Variable.
    
    7. Configure Flintrock using - 'flintrock configure'
    
    8. Change the following fields in the ~/.config/flintrock/config.yaml. The EC2 key pair must be added to the EC2 instance and read-only permission must be set to it using - chmod 400 <key.pem>

    providers:
        
        key-name: MLSparkKey

        identity-file: /home/hadoop/MLSparkKey.pem
        
        isntance-type: m5.xlarge
        
        ami: <the ami of the logged in ec2 instance from the ec2 console>

    launch:
        
        num-slaves: 4
        
        install-hdfs: True
        
        install-spark: True
    
    9. flintrock launch ml-cluster - This will launch 5 instances of 1 master and 4 slaves.
    
    10. Copy Python file to Flintrock using the command - flintrock copy-file ml-cluster /home/hadoop/wineQualityParallelTraining.py /home/ec2-user/wineQualityParallelTraining.py
    
    11. flintrock login ml-cluster
    
    12. Configure AWS Credentials.
    
    13. Install the following libraries in the flintrock in the ec2 instances:

        pyspark => pip install pyspark

        boto3 => pip install boto3

        numpy => pip install numpy
    
    14. Run the spark application using the below command
        
        spark-submit --deploy-mode client --master spark://<master_pubic_ipv4_dns>:7077 wineQualityParallelTraining.py
        
        <master_pubic_ipv4_dns> => ec2-54-83-99-140.compute-1.amazonaws.com => This can be obtained from running the command flintrock describe


**Steps to run the Training Model in 4 slave instances:**

1. Follow the EMR setup and EC2 setup steps from step 1 to 14.

Results: After the training of the model is completed, the model is stored in the S3 bucket as tar.gz file for future usage.

The spark application displays the F1-Score for the Validation Dataset from S3 bucket model and the locally trained model.


**Steps to run the Single Machine Prediction Application in 1 instance:**

Without HDFS and Flintrock:

1. Follow the EMR setup and EC2 setup steps from step 1 to 4.

2. Add the wineQualityPrediction.py to the EC2 instance and the testdataset csv file.

3. Install the following libraries in the flintrock in the ec2 instances:

    pyspark => pip install pyspark

    boto3 => pip install boto3

    numpy => pip install numpy

4. Run the application using - **python wineQualityPrediction.py <testdataset.csv>** (Pass the testdataset filename as a command line argument)

5. Results: The training model is retrieved from Amazon S3 bucket and the model is applied over the test dataset. The F-1 score for the Random Forest Classification model is returned along with the Prediction and Quality columns after the model is applied.

**Docker Configuration**

1. Create the Dockerfile and requirements.txt file with the libraries and modules to be installed.

2. Create a docker repository in the Docker profile named - mlapplication_pa2.

3. Run the following commands to create and publish a docker image to the docker hub.
    
    Create a docker image:
    
    **docker build -t mlapplication .**
    
    To tag the docker image created:
    
    **docker tag mlapplication mugunthankrishnan/mlapplication_pa2:v1**
    
    To push the docker image to docker hub:
    
    **docker push mugunthankrishnan/mlapplication_pa2:v1**

    Docker Execution:
    
    To pull the docker image:

    docker pull mugunthankrishnan/mlapplication_pa2

    To run the docker image:

    docker run -p 8080:80 mugunthankrishnan/mlapplication_pa2

    UI Images from flask application:
    
    ![image](https://user-images.githubusercontent.com/123665839/235308969-feef93b7-0111-423d-87c0-89a62514701e.png)

