# Achieving Operational Excellence Using Automated Playbook and Runbook

This lab is provided as part of **[AWS Innovate For Every Application Edition](TBA)**

Click [here](TBA) to explore the full list of hands-on labs.

ℹ️ You will run this lab in your own AWS account. Please follow directions at the end of the lab to remove resources to avoid future costs.

## Introduction

This lab was derived directly from one of Operataional Excellence Labs named [Automating operations with Playbooks and Runbooks](https://wellarchitectedlabs.com/operational-excellence/200_labs/) in AWS Well-Architected Lab. 

Manually running your [runbooks](https://wa.aws.amazon.com/wat.concept.runbook.en.html) and [playbooks](https://wa.aws.amazon.com/wat.concept.playbook.en.html) for operational activities has a number of drawbacks:

* Activities are prone to errors & difficult to trace. 
* Manual activities do not allow your operational practice to scale in line with your business requirements. 

In contrast, implementing automation in these activities has the following benefits:

* Improved reliability by preventing the introduction of errors through manual processes.
* Increased scalability by allowing non linear resource investment to operate your workload.
* Increased traceability on your operation through log collection of the automation activity.
* Improved incident response by reducing idle time and automatically triggering activity based on known events.

<details>
<summary> Click here if you would like to know what runbook and playbook are </summary>


At a glance, both **runbooks** and **playbooks** appear to be similar documents that technical users, can use to perform operational activities. However, there an essential difference between them:

* A [playbook](https://wa.aws.amazon.com/wellarchitected/2020-07-02T19-33-23/wat.concept.playbook.en.html) documents contain processes that guides you through activities to investigate an issue. For example, gathering applicable information, identifying potential sources of failure, isolating faults, or determining the root cause of issues. Playbooks can follow multiple paths and yield more than one outcome.

* A [runbook](https://wa.aws.amazon.com/wat.concept.runbook.en.html) contains procedures necessary to achieve a specific outcome. For example, creating a user, rolling back configuration, or scaling resource to resolve the issue identified.

</details>

This hands-on lab will guide you through the steps to automate your operational activities using runbooks and playbooks built with AWS tools.

We will show how you can build automated runbooks and playbooks to investigate and remediate application issues using the following AWS services:

* [Systems Manager Automation](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html)
* [Simple Notification Service](https://aws.amazon.com/sns/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)
* [Amazon CloudWatch synthetic monitoring](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html)

## Prerequisites:

* An [AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) that you are able to use for testing. The account should not be used for production purposes.  
* An [IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html) in your AWS account with full access to [CloudFormation,](https://aws.amazon.com/cloudformation/) [Amazon ECS,](https://aws.amazon.com/ecs/)[Amazon RDS,](https://aws.amazon.com/rds/) [Amazon Virtual Private Cloud (VPC),](https://aws.amazon.com/vpc/) [AWS Identity and Access Management (IAM),](https://aws.amazon.com/iam/) [AWS Cloud9](https://aws.amazon.com/cloud9/)

## Costs

NOTE: You will be billed for any applicable AWS resources used if you complete this lab that are not covered in the [AWS Free Tier](https://aws.amazon.com/free/).

This lab walks you through creating a CI/CD workflow for serveress applications. 
## Content 

- [Step 1. Deploy the sample application environment](https://github.com/aws-samples/build-and-operate-a-secure-and-successful-cloud-operations-model#step-1-Deploy-the-sample-application-environment)
- [Step 2. Simulate an Application Issue](https://github.com/aws-samples/build-and-operate-a-secure-and-successful-cloud-operations-model#step-2-Simulate-an-Application-Issue)
- [Step 3. Build and Run an Investigative Playbook](https://github.com/aws-samples/build-and-operate-a-secure-and-successful-cloud-operations-model#step-3-Build-and-Run-an-Investigative-Playbook)
- [Step 4. Build and Run Remediation Runbook](https://github.com/aws-samples/build-and-operate-a-secure-and-successful-cloud-operations-model#step-4-Build-and-Run-Remediation-Runbook)
- [Teardown](https://github.com/aws-samples/build-and-operate-a-secure-and-successful-cloud-operations-model#Teardown)
- [Summary](https://github.com/aws-samples/build-and-operate-a-secure-and-successful-cloud-operations-model#Summary)

### Step 1. Deploy the sample application environment
In this section, you will prepare a sample application. The application is an API hosted inside a docker container, using [Amazon Elastic Compute Service (ECS).](https://aws.amazon.com/ecs/). The container is accessed via an [Application Load Balancer.](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) 

The API is a private microservice within your [Amazon Virtual Private Cloud (VPC)](https://aws.amazon.com/vpc/). Communication to the API can only be done privately through routes within the VPC subnet. In our lab example, the business owner has agreed to run the API over HTTP protocol to simplify the implementation. 

The API has two actions available which encrypt and decrypt information. This is triggered by doing a REST POST call to the */encrypt* / */decrypt* methods as appropriate.

* The *encrypt* action will allow you to pass a secret message along with a 'Name' key as the identifier and it will return a 'Secret Key Id' that you can use later to decrypt your message.
* The *decrypt* action allows you to then decrypt the secret message passing along the 'Name' key and 'Secret Key Id' you obtained before to get your secret message.

Both actions will make a write and read call to the application database hosted in [Amazon Relation Database Service (RDS)](https://aws.amazon.com/rds/), where the encrypted messages are stored. 

The following step-by-step instructions will provision the application that you will use with your  **runbooks**  and  **playbooks** . 

Explore the contents of the CloudFormation script to learn more about the environment and application.

You will use this sample application as a sandbox to simulate an application performance issue, start your  **runbooks**  and  **playbooks**  to autonomously investigate and remediate.

#### Actions items in this section:

1. You will prepare the [Cloud9](https://aws.amazon.com/cloud9/) workspace launched with a new VPC.
2. You will run the application build script from the Cloud9 console to build the sample application as shown in the diagram below.

![Section1 App Arch](/Images/section2-base-application.png)


### 1.0 Prepare Cloud9 workspace.

In this first step you will provision a [CloudFormation](https://aws.amazon.com/cloudformation/) stack that builds a Cloud9 workspace along with the VPC for the sample application. This Cloud9 workspace will be used to run the provisioning script of the sample application. You can choose the to deploy stack in one of the regions below. 

1. Click on the link below to deploy the stack. This will take you to the CloudFormation console in your account. Use `walab-ops-base-resources` as the stack name, and take the default values for all options.

    * **us-west-2** : [here](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/create/review?stackName=walab-ops-base-resources&templateURL=https://aws-well-architected-labs-singapore.s3.ap-southeast-1.amazonaws.com/Operations/200_Automating_operations_with_playbooks_and_runbooks/base_resources.yml)
    * **ap-southeast-2** : [here](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-2#/stacks/create/review?stackName=walab-ops-base-resources&templateURL=https://aws-well-architected-labs-singapore.s3.ap-southeast-1.amazonaws.com/Operations/200_Automating_operations_with_playbooks_and_runbooks/base_resources.yml)
    * **ap-southeast-1** : [here](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/stacks/create/review?stackName=walab-ops-base-resources&templateURL=https://aws-well-architected-labs-singapore.s3.ap-southeast-1.amazonaws.com/Operations/200_Automating_operations_with_playbooks_and_runbooks/base_resources.yml)

2. Once the template is deployed, wait until the CloudFormation Stack reaches the **CREATE_COMPLETE** state.

![Section1 ](/Images/section2-base-resources-create-complete.png)


### 1.1 Run the build application script.

Next, run the build script to build and deploy you application environment from the Cloud9 workspace as follows:

  1. From the main console, access the **Cloud9** service. 
  2. Click **Your environments** section on the left menu, and locate an environment named `WellArchitectedOps-walab-ops-base-resources` as below, then click **Open IDE**.

      ![Section 2 Cloud9 IDE Welcome Screen](/Images/section2-environment-open-ide.png)

  3. Your environment will bootstrap the lab repository. You should see a terminal output showing the following output: 
  
      ![Section 2](/Images/section2-base-bootstrap.png)

      When the bootstrap script finishes you will see a folder called `aws-well-architected-labs`. 

  4. In the IDE terminal console, change directory to the working folder where the build script is located:

      ```
      cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/scripts/
      ```

  5. Copy and paste the command below, replacing `sysops@domain.com` and `owner@domain.com` with the email address you would like the application to notify you with. Replace the `sysops@domain.com` value with email representing system operators team and `owner@domain.com` with email address representing business owner.


      ```
      bash build_application.sh walab-ops-base-resources sysops@domain.com owner@domain.com
      ```


>  The `build_application.sh` script will build and deploy your sample application, along with the architecture that hosts it.
  The application architecture will have capabilities to notify systems operators and owners, leveraging [Amazon Simple Notification Service](https://aws.amazon.com/sns/).
  You can use the same email address for `sysops@domain.com` and `owner@domain.com` if you need to, but ensure that you have both values specified.

  If you have deployed Amazon ECS before in your account, you may encounter InvalidInput error with message "AWSServiceRoleForECS has been taken" while running the build_application.sh script. You can safely ignore this message, as the script will continue despite the error.

  6. The above command runs the build and provisioning of the application stack. The script should take about 20 mins to finish.

        ![Section 2 Cloud9 IDE Welcome Screen](/Images/section2-base-app-build.png)

>  The `build_application.sh` will deploy the application docker image and push it to [Amazon ECR](https://aws.amazon.com/ecr/). This is used by [Amazon ECS.](https://aws.amazon.com/ecs/) Once the build script completes, another CloudFormation stack containing the application resources (ECS, RDS, ALB, and others) will be deployed.

  7. In the CloudFormation console, you should see a new stack being deployed called `walab-ops-sample-application`. Wait until the stack reaches **CREATE_COMPLETE** state and proceed to the next step.
  
      ![Section 2 CreateComplete](/Images/section2-base-app-create-complete.png)

### 1.2. Confirm the application status.

Once the application is successfully deployed, go to your [CloudFormation console](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-2) and locate the stack named `walab-ops-sample-application`.

  1. Confirm that the stack is in a **'CREATE_COMPLETE'** state. 
  2. Record the following output details as it will be required later:
  3. Take note of the DNS value specified under **OutputApplicationEndpoint**  of the Outputs.

      The screenshot below shows the output from the CloudFormation stack:

      ![Section2 DNS Output](/Images/section2-dns-outputs.png)

  4. Check for an email sent to the system operator and owner addresses you've specified in the build_application.sh script. This email should also be visible in the CloudFormation parameter under in the **SystemOpsNotificationEmail** and **SystemOwnerNotificationEmail**.

  5. Click `confirm subscription` on the email links to subscribe.

      ![Section2 DNS Output](/Images/section2-email-confirm.png)

>  There will be 2 emails sent to your address, please ensure to subscribe to **both** of them.

### 1.3. Test the application.

In this section, you will be testing the encrypt API action from the deployed application. 

The application will take a JSON payload with `Name` as the identifier and `Text` key as the value of the secret message.

The application will encrypt the value under `Text` key with a designated KMS key and store the encrypted text in the RDS database with `Name` as the primary key.

> **Note:** For simplicity purposes the sample application will re-use the same KMS keys for each record generated.

<details>
<summary> Click here to test </summary>

1. In the **Cloud9** terminal, run the command below, replacing the `ApplicationEndpoint` with the **OutputApplicationEndpoint** from previous step. This command will run [curl](https://curl.se/) to send a POST request with the secret message payload `{"Name":"Bob","Text":"Run your operations as code"}` to the API.

    ```
    ALBEndpoint="ApplicationEndpoint"
    ```

    ```
    curl --header "Content-Type: application/json" --request POST --data '{"Name":"Bob","Text":"Run your operations as code"}' $ALBEndpoint/encrypt
    ```

2. Once you run this command, you should see output as follows:

    ```
    {"Message":"Data encrypted and stored, keep your key save","Key":"EncryptKey"}
    ```

3. Take note of the encrypt key value under **Key** .

4. Run the command below, pasting the encrypt key you took note of previously under the **Key** section to test the decrypt API.


    ```
    curl --header "Content-Type: application/json" --request GET --data '{"Name":"Bob","Key":"EncryptKey"}' $ALBEndpoint/decrypt

    ```

5. Once you run the command you should see the following output:

    ```
    {"Text":"Run your operations as code"}
    ```
</details>

## Congratulations! 

You have now completed the first section of the Lab.

You should have a sample application API which we will use for the remainder of the lab.

### Step 2. Simulate an Application Issue
Understanding the health of your workload is an essential component of Operational Excellence. Defining metrics and thresholds, together with appropriate alerts will ensure that issues can be acknowledged and remediated within an appropriate timeframe.

In this section of the lab, you will simulate a performance issue within the API. Using Amazon CloudWatch synthetic, your API will utilize a canary monitor, which continuously checks API response time to detect an issue. 

In this example, should the API take longer than 6 seconds to respond, an alert will be created, triggering a notification email.

#### Actions items in this section:

1. You will run a script that will send a large amount of traffic to the API.
2. You will observe and confirm the issue through AWS monitoring tools. 

The following resources had been deployed to perform these actions.

![Section3 Base Architecture](/Images/section3-testing-canary-alarm-architecture.png)

### 2.0 Sending traffic to the application

In this section, you will send multiple concurrent requests to the application, simulating a large surge of incoming traffic. This will overwhelm the API, which will gradually increase the response time of the application. This results in the canary monitoring exceeding the set threshold, triggering the CloudWatch Alarm to send notification.

Follow below steps to continue:

1. From the **Cloud9** terminal, run the command shown below to change directory to the working script folder:

    ```
    cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/scripts/
    ```

2. Confirm that you have the `test.json` in the folder and it contains the following text:

    ```
    {"Name":"Test User","Text":"This Message is a Test!"}
    ```

3. Go to CloudFormation console and take note of the **OutputApplicationEndpoint** value under Output tab of `walab-ops-sample-application` stack. This is the DNS endpoint of the Application Load Balancer.


    ![Section3 Succces Screenshot](/Images/section3-stackoutput.png)


4. Make sure you have test the application previously. If so, execute the command below:

    ```
    bash simulate_request.sh $ALBEndpoint
    ```

    This script uses the [Apache Benchmark](https://httpd.apache.org/docs/2.4/programs/ab.html) to send 60,000,000 requests, 3000 concurrent request at a time. 
    
    When you run the command you will see the output gradually change from a consistently successful 200 response to include 504 time-out responses. 
    
    The requests generated by the script are overwhelming the application API and result in occasional timeouts by your load balancer. 
    
    Keep the command running in the background as you proceed through the lab.

    ![Section3 Succces Screenshot](/Images/section3-success-traffic-requests.png)

    ![Section3 Failure Screenshot](/Images/section3-failure-traffic-requests.png)


### 2.1 Observing the alarm being triggered.

1. After approximately 6 minutes, you will see an alarm which is triggered as a response to the generated activity. This will trigger an email indicating that the CloudWatch alarm has been triggered.

    ![Section3 Email](/Images/section3-email.png)
 
2. Check and confirm the alarm by going to the CloudWatch console.

3. Click on the Alarms section on the left menu.

4. Click on the Alarms called `mysecretword-canary-duration-alarm`, which should be in an alarm state.

    ![Section3 Failure Screenshot](/Images/section3-alarm.png)

5. Click on the alarm to display the CloudWatch metrics that the alarm data is based from.

6. The alarm is based on the `Duration` metric data emitted by the `mysecretword-canary` CloudWatch synthetic canary monitor. The Duration metric measures how long it takes for the canary requests to receive a response from the application. 

7. The alarm is triggered whenever the value of the `Duration` metric is above 6 seconds within a 1 minute duration. The latest threshold will be 5000 for 3 datapoints within 6 minutes.

    ![Section3 Failure Screenshot](/Images/section3-alarm-detail.png)

8. On the left menu click on **Application monitoring and **Synthetics Canaries** and locate the canary monitor named `mysecretword-canary`.

    ![Section3 Canary](/Images/section3-canary.png)

9. Click on the canary and the select the **Configuration** tab.

10. From here you will see the canary configuration and a snippet of the canary script.

11. In the canary script section, scroll down to the section that contains `let requestOptionStep1` as shown in the screenshot below. This is the configuration that controls the destination of the request (hostname, path and payload body).

    ![Section3 Canary](/Images/section3-canary-detail.png)

12. Click on the **Monitoring** tab.

13. From here you will see the visualization of the metrics that the canary monitor generates.

14. Locate the 'Duration' metric that is being used to trigger the CloudWatch alarm.

15. You will see the average duration value of the canary request representing the time to complete. A value above 6000ms signifies that the request has taken more than 6 seconds to receive a response from the application, indicating a performance issue in the API.

    ![Section3 Canary](/Images/section3-canary-monitor.png)

You have now completed the second section of the lab.

You should still have the `simulate_request.sh` running in the background, simulating a large influx of traffic to your API. This causes the application to respond slowly and time-out periodically. The CloudWatch Alarm will be triggering and performance issue notifications sent to your System Operator to prompt them into action.

> This concludes **Section 2** of this lab. Click 'Next step' to continue to the next section of the lab where we will build an automated **playbook** to assist investigation of the issue. 

### Step 3. Build and Run an Investigative Playbook
The efficiency of issue resolution within an Operations team is directly linked to their tenure and experience. Where an Operator has prior knowledge of a particular issue, they will have a headstart in being able to reach resolution in terms of understanding logs and metrics which were used in previous situations. Whilst this constitutes value to an Operations group, it also represents a single point of failure and a scalability challenge.

This is where [playbooks](https://wa.aws.amazon.com/wat.concept.playbook.en.html) become important. Playbooks are a documented set of predefined steps, which are run to identify an issue. The result of each step can be used to either call more steps to run, or alternatively to trigger manual intervention.

Automating **playbook** activities wherever possible, is critical to reducing the time to respond to an incident.

The AWS Cloud offers multiple services you can use to build an automated playbook, one which is AWS Systems Manager.

AWS Systems Manager offers an automation document capability (known within Systems Manager as [runbooks](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-documents.html
)), which allows for the creation of a series of executable steps to orchestrate your investigation and remediation. AWS Systems Manager Automation Documents allow a user to run custom scripts, call AWS service APIs, or even run remote commands on cloud or on-premise compute instances.

In this section, you will focus on creating an automated **playbook** in assisting your investigation, as a Systems Operator.

#### Actions items in this section:

1. You will build a **playbook** to gather information about the workload and query the relevant metrics and logs.
2. You will run the automation document to investigate your issue. 

### 3.0 Prepare Automation Document IAM Role

The Systems Manager Automation Document you are building will require assumed permissions to run the investigation and remediation steps. You will need to create the IAM role that will assume the permissions to perform the **playbook** activities. To simplify the deployment process, a CloudFormation template has been provided that you can deploy via the console or AWS CLI. Please choose one of the two following deployment steps:

<details>
<summary> Click here for CloudFormation Console deployment step </summary>

  1. Download the template [here.](/Code/templates/automation_role.yml "Resources template")
  2. Follow this [guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) for information on how to deploy the CloudFormation template.
  3. Use `waopslab-automation-role` as the **Stack Name**, as this is referenced by other stacks later in the lab.

</details>

<details>
<summary> Click here for CloudFormation CLI deployment step (Preferred way) </summary>

**Note:** To deploy from the command line, ensure that you have installed and configured AWS CLI with the appropriate credentials.

1. From the **Cloud9** terminal change to the appropriate folder as shown:

  ```
  cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/templates
  ```

2. Then run the command listed below:

  ```
  aws cloudformation create-stack --stack-name waopslab-automation-role \
                                  --capabilities CAPABILITY_NAMED_IAM \
                                  --template-body file://automation_role.yml 
  ```

3. Confirm that the stack has installed correctly. You can do this by running the **describe-stacks** command:

  ```
  aws cloudformation describe-stacks --stack-name waopslab-automation-role
  ```

Locate the **StackStatus** and confirm it is set to **CREATE_COMPLETE** 
</details>

1. Once you have deployed the CloudFormation stack above, go to the IAM Console.

2. On the side menu, click on **Roles** and locate the IAM role named **AutomationRole**.

3. Take note of the ARN of the role, as we will need it later in the lab.

  
![Section3 ](/Images/section3-automationrole.png)

### 3.1 Building the "Gather-Resources" Playbook.

In preparation for the investigation, you need to know all services and resources associated to the issue. When the email notification is sent, information in the email does not contain any resources information. To gather this necessary information, we will build a **playbook** to acquire all related resources using our CloudWatch alarm ARN as a reference. 

Codifying your **playbook** with AWS Systems Manager allows for maximum code reusability. This will reduce overhead in re-writing codes that has identical objectives.   

![Section4 ](/Images/section4-architecture-graphics1.png)


> **Note:** Follow these step to build and run playbook. Select a guide to deploy using either the AWS console, the AWS CLI or via a CloudFormation template deployment. 

<details>
<summary> Click here for CloudFormation Console deployment step </summary>

Download the template [here.](/Code/templates/playbook_gather_resources.yml "Resources template")


If you decide to deploy the stack from the console, ensure that you follow below requirements & step:

  1. Follow this [guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) for information on how to deploy the CloudFormation template.
  2. Use `waopslab-playbook-gather-resources` as the **Stack Name**, as this is referenced by other stacks later in the lab.

</details>

<details>
<summary> Click here for CloudFormation CLI deployment step (Preferred way) </summary>

**Note:** To deploy from the command line, ensure that you have installed and configured AWS CLI with the appropriate credentials.

1. From the **Cloud9** terminal, run the command to get into the working script folder

  ```
  cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/templates
  ```

2. Then run the below commands, replacing the 'AutomationRoleArn' with the Arn of **AutomationRole** you took note in previous step 3.0.

  ```
  aws cloudformation create-stack --stack-name waopslab-playbook-gather-resources \
                                  --parameters ParameterKey=PlaybookIAMRole,ParameterValue=AutomationRoleArn \
                                  --template-body file://playbook_gather_resources.yml 
  ```

  Example:

  
  ```
  aws cloudformation create-stack --stack-name waopslab-playbook-gather-resources \
                                  --parameters ParameterKey=PlaybookIAMRole,ParameterValue=arn:aws:iam::000000000000:role/AutomationRole \
                                  --template-body file://playbook_gather_resources.yml 
  ```

  **Note:** Please adjust your command-line if you are using profiles within your aws command line as required.

3. Confirm that the stack has installed correctly. You can do this by running the **describe-stacks** command below, locate the **StackStatus** and confirm it is set to **CREATE_COMPLETE**. 

  ```
  aws cloudformation describe-stacks --stack-name waopslab-playbook-gather-resources
  ```

</details>


<details>
<summary> Click here for Console step-by-step </summary>

  1. Go to the AWS Systems Manager console. Click **Documents** under **Shared Resources** on the left menu. Then click **Create Automation** as show in the screen shot below:

  ![Section4 ](/Images/section4-create-automation.png)

  2. Enter `Playbook-Gather-Resources` in the **Name** field and copy the notes shown below into the **Document description** field. 

```
# What does this **playbook** do?

Query the CloudWatch Synthetics Canary and look for all resources related to the application based on it's Application Tag. This **playbook** takes an input of the CloudWatch Alarm ARN triggered by the canary

Note : Application resources must be deployed using CloudFormation and properly tagged accordingly.

## Actions taken in this playbook.
1. Describe CloudWatch Alarm ARN and identify the Canary resource.
2. Describe the Canary resource to gather the value of 'Application' tag
3. Gather CloudFormation Stack with the same value of 'Application' tag.
4. List all resources in CloudFormation Stack.
5. Parse list of resources into String Output.
```

  3. In the **Assume role** field, enter the IAM role ARN we created in the previous section **3.0 Prepare Automation Document IAM Role**.

  ![Section4 ](/Images/section4-create-automation-playbook-role.png)


  4. Expand the **Input Parameters** section and enter `AlarmARN` as the **Parameter name**. Set the type as `String` and **Required** as `Yes`. This will define a Parameter within our playbook, so that the value of the CloudWatch Alarm ARN can be passed into the playbook to run the action.

  ![Section4 ](/Images/section4-create-automation-parameter-input.png)

  5. Under **Step 1** section specify `Gather_Resources_For_Alarm` **Step name**, select `aws::executeScript` as the **Action type**. 

  6. Under **Inputs** set `Python3.6` as the **Runtime** and specify `script_handler` as the **Handler**.
  7. Paste in below python codes into the **Script** section.

  ![Section4 ](/Images/section4-create-automation-addstep.png)

  ```
    import json
    import re
    from datetime import datetime
    import boto3
    import os

    def arn_deconstruct(arn):
    arnlist = arn.split(":")
    service=arnlist[2]
    region=arnlist[3]
    accountid=arnlist[4]
    servicetype=arnlist[5]
    name=arnlist[6]
    return {
      "Service": service,
      "Region": region,
      "AccountId": accountid,
      "Type": servicetype,
      "Name": name
    }

    def locate_alarm_source(alarm):
    cwclient = boto3.client('cloudwatch', region_name = alarm['Region'] )
    alarm_source = {}
    alarm_detail = cwclient.describe_alarms(AlarmNames=[alarm['Name']])  

    if len(alarm_detail['MetricAlarms']) > 0:
      metric_alarm = alarm_detail['MetricAlarms'][0]
      namespace = metric_alarm['Namespace']
      
      # Condition if NameSpace is CloudWatch Syntetics
      if namespace == 'CloudWatchSynthetics':
        if 'Dimensions' in metric_alarm:
          dimensions = metric_alarm['Dimensions']
          for i in dimensions:
            if i['Name'] == 'CanaryName':
              source_name = i['Value']
              alarm_source['Type'] = namespace
              alarm_source['Name'] = source_name
              alarm_source['Region'] = alarm['Region']
              alarm_source['AccountId'] = alarm['AccountId']

      result = alarm_source
      return result

    def locate_canary_endpoint(canaryname,region):
    result = None
    synclient = boto3.client('synthetics', region_name = region )
    res = synclient.get_canary(Name=canaryname)
    canary = res['Canary']
    if 'Tags' in canary:
      if 'TargetEndpoint' in canary['Tags']:
        target_endpoint = canary['Tags']['TargetEndpoint']
        result = target_endpoint
    return result


    def locate_app_tag_value(resource):
    result = None
    if resource['Type'] == 'CloudWatchSynthetics':
      synclient = boto3.client('synthetics', region_name = resource['Region'] )
      res = synclient.get_canary(Name=resource['Name'])
      canary = res['Canary']
      if 'Tags' in canary:
        if 'Application' in canary['Tags']:
          apptag_val = canary['Tags']['Application']
          result = apptag_val
    return result

    def locate_app_resources_by_tag(tag,region):
    result = None

    # Search CloufFormation Stacks for tag
    cfnclient = boto3.client('cloudformation', region_name = region )
    list = cfnclient.list_stacks(StackStatusFilter=['CREATE_COMPLETE','ROLLBACK_COMPLETE','UPDATE_COMPLETE','UPDATE_ROLLBACK_COMPLETE','IMPORT_COMPLETE','IMPORT_ROLLBACK_COMPLETE']  )
    for stack in list['StackSummaries']:
      app_resources_list = []
      stack_name = stack['StackName']
      stack_details = cfnclient.describe_stacks(StackName=stack_name)
      stack_info = stack_details['Stacks'][0]
      if 'Tags' in stack_info:
        for t in stack_info['Tags']:
          if t['Key'] == 'Application' and t['Value'] == tag:
            app_stack_name = stack_info['StackName']
            app_resources = cfnclient.describe_stack_resources(StackName=app_stack_name)
            for resource in app_resources['StackResources']:
              app_resources_list.append(
                { 
                  'PhysicalResourceId' : resource['PhysicalResourceId'],
                  'Type': resource['ResourceType']
                }
              )
            result =  app_resources_list

    return result
    def script_handler(event, context):
    result = {}
    arn = event['CloudWatchAlarmARN']
    alarm = arn_deconstruct(arn)
    # Locate tag from CloudWatch Alarm

    alarm_source = locate_alarm_source(alarm) # Identify Alarm Source
    tag_value = locate_app_tag_value(alarm_source) #Identify tag from source

    if alarm_source['Type'] == 'CloudWatchSynthetics':
      endpoint = locate_canary_endpoint(alarm_source['Name'],alarm_source['Region'])
      result['CanaryEndpoint'] = endpoint
      
    # Locate cloudformation with tag
    resources = locate_app_resources_by_tag(tag_value,alarm['Region'])
    result['ApplicationStackResources'] = json.dumps(resources) 

    return result
  ```

  8. Under **Additional inputs** specify the input value to the step, passing in the parameter we created previously. To do this, specify below values:
  
      * `InputPayload` as the **Input name** 
      * `CloudWatchAlarmARN: '{{AlarmARN}}'` as the **Input Value**.
  
  9. Under **Outputs** specify below values:
  
      * `Resources` as **Name**
      * `$.Payload.ApplicationStackResources` as **Selector** 
      * `String` as **Type**
    
  10. Once your settings match the screenshot below, click on **Create Automation**

  ![Section4 ](/Images/section4-create-automation-additionals.png)

</details>


Once the automation document is created, you can now give it a test.

  1. You can then find the newly created document under the **Owned by me** tab of the **Document** section in Systems Manager Console.

  ![Section3 ](/Images/section3-playbook-gather-resource-tab.png)

  2. Click on the **playbook** called `Playbook-Gather-Resources` and click on **Execute Automation** to run your playbook.
  3. Paste in the CloudWatch Alarm ARN ( You can find this ARN in the email notification in section **2.1 Observing the alarm being triggered** ) and click on **Execute** to test the playbook.

  ![Section3 ](/Images/section3-alarm-email.png)

  4. Once the **playbook** run is completed successfully, click on the **Step Id** to see the final message and output of the step. You should be able to see this output listing all the resources of the application

  ![Section3 ](/Images/section3-gather-resources-stepid.png)

  5. **Copy** the Resources list output from the section as highlighted in the screenshot below. This list consist of the all the resources defined in the CloudFormation stack related to our application. These information includes the Elastic Load Balancer, ECS and RDS resource id that we can now use to further our investigation of the underlying issue.  

  ![Section3 ](/Images/section4-create-automation-playbook-run-output.png)

  6. You can **Paste** the output into a temporary location like notepad for now. You will need this value for our next step. 

### 3.2 Building the "Investigate-Application-Resources" Playbook.

In the previous step, you have created a **playbook** that finds all related AWS resources in the application.
In this step you will create a **playbook** that will interrogate resources, capture recent metrics and logs, to look for insights and better understand the root cause of the issue.

In practice, there can be various possibilities of actions that the **playbook** can take to investigate, depending on the scenario presented by the issue. The purpose of this Lab is to showcase how you can use **playbook** to aid investigation, rather than advise on a specific action path. 

Therefore, in this lab we will assume an example scenario. The **playbook** will look at metrics and logs of the ELB, ECS and RDS services in the resource list. The **playbook** will then highlight the metrics and logs that is considered outside of normal operational threshold. 


![Section3 ](/Images/section4-architecture-graphics2.png)

Please follow the below instructions to build this playbook:

> **Note:** We will deploy this **playbook** via CloudFormation template to simplify deployment. Please follow the steps below to deploy the CloudFormation template via CLI / or Console. 


<details>
<summary> Click here for CloudFormation Console deployment step </summary>

Download the template [here.](/Code/templates/playbook_investigate_application_resources.yml "Resources template")


If you decide to deploy the stack from the console, ensure that you follow below requirements & step:

  1. Please follow this [guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) for information on how to deploy the CloudFormation template.
  2. Use `waopslab-playbook-investigate-resources` as the **Stack Name**, as this is referenced by other stacks later in the lab.


</details>

<details>
<summary> Click here for CloudFormation CLI deployment step (Preferred way)</summary>


1. From the Cloud9 terminal, change to the required folder as shown:

  ```
  cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/templates
  ```

2. Run the command below, replacing the 'AutomationRoleArn' with the Arn of **AutomationRole** you took note in previous step **3.0 Prepare Automation Document IAM Role**.

  ```
  aws cloudformation create-stack --stack-name waopslab-playbook-investigate-resources \
                                  --parameters ParameterKey=PlaybookIAMRole,ParameterValue=AutomationRoleArn \
                                  --template-body file://playbook_investigate_application_resources.yml 
  ```
  Example:

  ```
  aws cloudformation create-stack --stack-name waopslab-playbook-investigate-resources \
                                  --parameters ParameterKey=PlaybookIAMRole,ParameterValue=arn:aws:iam::000000000000:role/xxxx-playbook-role \
                                  --template-body file://playbook_investigate_application_resources.yml 
  ```

3. Confirm that the stack has installed correctly. You can do this by running the **describe-stacks** command as follows:

  ```
  aws cloudformation describe-stacks --stack-name waopslab-playbook-investigate-resources
  ```

4. Locate the **StackStatus** and confirm it is set to **CREATE_COMPLETE** 

</details>

When the document is created, you can go ahead and run a quick test.

You can find the newly created document under the **Owned by me** tab of the Document resource in the Systems Manager console.

  1. Click on the **playbook** called `Playbook-Investigate-Application-Resources` and click on **Execute Automation** to run our playbook.
  
  2. Paste in the resources list you took note from the output of the previous **playbook** ( refer to section **3.1 Building the "Gather-Resources" Playbook** ) under **Resources** and click on **Execute**

      ![Section3 ](/Images/section3-investigate-resourcelist.png)

  3. Under **Executed Steps** you should be able to see each of the step the **playbook**. If you view the content of the document you will be able to see the code and find out what each step does. 

      ![Section3 ](/Images/section3-steps-explain.png)

      For simplicity, we have created a list of output and description for each step. Expand the list below to view.

        <details>
        <summary> Output list </summary>


        | Step Name              | Description                                                                                                                                | Output list                          |
        |------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
        |  Gather_ELB_Statistics |  Go through the resource list and locate the ELB. Query data from the ELB CloudWatch metrics, looking at metrics from the last 60 minutes. |  TargetResponseTime (Average)        |  
        |||  HTTPCode_Target_2XX_Count (Sum) | 
        |||  HTTPCode_Target_3XX_Count (Sum) |  
        |||  HTTPCode_Target_4XX_Count (Sum) |  
        |||  HTTPCode_Target_5XX_Count (Sum) |  
        |||  TargetConnectionErrorCount (Sum) |  
        |||  UnHealthyHostCount (Average) |  
        |||  ActiveConnectionCount (Sum) |  
        |||  HTTPCode_ELB_3XX_Count (Sum) |  
        |||  HTTPCode_ELB_4XX_Count (Sum) |  
        |||  HTTPCode_ELB_5XX_Count (Sum) |  
        |||  HTTPCode_ELB_500_Count (Sum) |  
        |||  HTTPCode_ELB_502_Count (Sum) |  
        |||  HTTPCode_ELB_503_Count (Sum) |  
        |||  HTTPCode_ELB_504_Count (Sum) | 
        |  Gather_RDS_Statistics |  Go through resource list and locate the RDS resource. Query data from the RDS CloudWatch metrics, looking at metrics from the last 60 minutes. |  BinLogDiskUsage (Sum) |  
        |||  BinLogDiskUsage (Sum) |
        |||  BurstBalance (Average) |
        |||  CPUUtilization (Average) |
        |||  CPUCreditUsage (Sum) |
        |||  CPUCreditBalance (Maximum) |
        |||  DatabaseConnections (Sum) |
        |||  DiskQueueDepth (Maximum) |
        |||  FailedSQLServerAgentJobsCount (Average) |
        |||  FreeableMemory (Maximum) |
        |||  MaximumUsedTransactionIDs (Maximum) |
        |||  NetworkReceiveThroughput (Average) |
        |||  OldestReplicationSlotLag (Average) |
        |||  ReadIOPS (Average) |
        |||  ReadLatency (Average) |
        |||  ReadThroughput (Maximum) |
        |||  ReplicaLag (Average) |
        |||  ReplicationSlotDiskUsage (Maximum) |
        |||  SwapUsage (Maximum) |
        |||  TransactionLogsDiskUsage (Maximum) |
        |||  TransactionLogsGeneration (Average) |
        |||  ReplicationSlotDiskUsage (Maximum) |                                                   
        |||  WriteIOPS (Average) |
        |||  WriteLatency (Average) |
        |||  WriteThroughput (Average) |
        |  Gather_ECS_Statistics |  Go through the resource list and locate the ECS resource. Query data from the ECS CloudWatch metrics, looking at metrics from the last 6 minutes. |  CPUUtilization (Maximum) |
        |||  MemoryUtilization (Maximum) |
        |  Gather_ECS_Error_Logs |  Go through the resource list and locate the ECS Service. Search in CloudWatch logs for any Error occurrence. ||
        |  Gather_ECS_Config |  Go through the resource list and locate the ECS resource. Describe the ECS service configuration. ||
        |  Gather_RDS_Config |  Go through the resource list and locate the RDS resource. Describe RDS Instance Config & Parameters. ||
        |  Inspect_Playbook_Results |  Go through the output of above steps, inspect results and check if it is above the threshold. | TargetResponseTime = 5 (ELB)  |  
        |||TargetConnectionErrorCount= 0 (ELB)
        |||UnHealthyHostCount = 0 (ELB)
        |||ELB5XXCount = 0 (ELB)
        |||ELB500Count = 0 (ELB)
        |||ELB502Count = 0 (ELB)
        |||ELB503Count = 0 (ELB)
        |||ELB504Count = 0 (ELB)
        |||Target4XXCount = 0 (ELB)
        |||Target5XXCount = 0 (ELB)
        |||CPUUtilization = 80 (ECS)
        </details>

  4. Wait until all steps are completed successfully.



### 3.3 Building the "Investigate-Application-From-Alarm" Playbook.

So far we have 2 separate playbooks. The first playbook gathers the list of resources associated with the application. The second playbook queries the relevant resources and investigates the appropriate logs and metrics.

In this step we will automate our **playbooks** further by creating a parent **playbook** that orchestrates the 2 Investigative **playbooks**. We will add another step to send notification to our Developers and System Owners.

![Section4 ](/Images/section4-architecture-graphics3.png)

Follow the instructions below to build the parent Playbook.

> **Note:** Select a step-by-step guide below to build the parent playbook using either the AWS console a CloudFormation template.

<details>
<summary> Click here for CloudFormation Console deployment step </summary>

Download the template [here.](/Code/templates/playbook_investigate_application.yml "Resources template")


If you decide to deploy the stack from the console, follow these steps:

  1. Please follow this [guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) for information on how to deploy the CloudFormation template.
  2. Use `waopslab-playbook-investigate-application` as the **Stack Name**, as this is referenced by other stacks later in the lab.
  3. In the parameter input screen, under **PlaybookIAMRole** enter ARN of **playbook** IAM role (defined in previous step), under **NotificationEmail** enter your designated email for **playbook** notification

</details>

<details>
<summary> Click here for CloudFormation CLI deployment step (Preferred way) </summary>


1. From the Cloud9 terminal, change to the required folder as shown:

  ```
  cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/templates
  ```

2. Then run below command :

  ```
  aws cloudformation create-stack --stack-name waopslab-playbook-investigate-application \
                                  --parameters ParameterKey=PlaybookIAMRole,ParameterValue=AutomationRoleArn \
                                  --template-body file://playbook_investigate_application.yml 
  ```
  Example:

  ```
  aws cloudformation create-stack --stack-name waopslab-playbook-investigate-application \
                                  --parameters ParameterKey=PlaybookIAMRole,ParameterValue=arn:aws:iam::000000000000:role/xxxx-playbook-role \
                                  --template-body file://playbook_investigate_application.yml 
  ```

**Note:** Please adjust your command-line if you are using profiles within your aws command line as required.

Confirm that the stack has installed correctly. You can do this by running the **describe-stacks** command as follows:
```
aws cloudformation describe-stacks --stack-name waopslab-playbook-investigate-application 
```

Locate the **StackStatus** and confirm it is set to **CREATE_COMPLETE** 

</details>

<details>
<summary> Click here for Console step-by-step guide </summary>

  1. From the AWS Systems Manager console, click on **documents** as shown below. Once you are there, click on **Create Automation**

  ![Section4 ](/Images/section4-create-automation.png)

  2. Next, enter in `Playbook-Investigate-Application-From-Alarm` in the **Name** and paste in the notes shown below into the **Description** box. This provides a description of the **playbook**. Systems Manager supports putting in notes as markdown, so feel free to format as required. 


  ```
  # What is does this **playbook** do?

  This **playbook** will run **Playbook-Gather-Resources** to gather Application resources monitored by Canary.

  Then subsequently run **Playbook-Investigate-Application-Resources** to Investigate the resources for issues. 

  Outputs of the investigation will be sent to SNS Topic Subscriber
    
  ```

  3. Under **Assume role** field, enter in the ARN of the IAM role we created in the previous step.
  
  4. Under **Input Parameters** field, enter `AlarmARN` as the **Parameter name**. Set the type as `String` and **Required** as `Yes`. This will define a Parameter into our playbook, which allows the value of the CloudWatch Alarm to be passed to the main step that will run the action.
  
  5. Add another parameter by clicking on the **Add a parameter** link. Enter `SNSTopicARN` as the **Parameter name**. Set the type as `String` and **Required** as `Yes`. This will  define another Parameter into our playbook, so that we can send notification to the Owner and Developer.

  ![Section4 ](/Images/section4-create-automation-parameter-input-2.png)


  6. Click **Add Step** and  create the first step of `aws:executeAutomation` Action type with StepName `PlaybookGatherAppResourcesCanaryCloudWatchAlarm`

  7. Specify `Playbook-Gather-Resources` as the **Document name** under Inputs and under **Additional inputs** specify `RuntimeParameters` with `{"AlarmARN":'{{AlarmARN}}'}` as it's value (refer to screenshot below). This step we will be run the `Gather-Resources` **playbook** which we created previously. 

  ![Section4 ](/Images/section4-create-automation-parameter-input-2-step1.png)

  8. Once this step is defined, add another step by clicking on **Add Step** at the bottom of the section.
  
  9. For this second step, specify the **Step name** as `PlaybookInvestigateAppResourcesELBECSRDS` and an action type of `aws:executeAutomation`.
  
  10. Specify `Playbook-Investigate-Application-Resources` as the **Document name** and `RuntimeParameters` as `Resources: '{{PlaybookGatherAppResourcesCanaryCloudWatchAlarm.Output}}'` This will take the output of the first step and pass to the second **playbook** to run the investigation of associated resources.

  ![Section4 ](/Images/section4-create-automation-parameter-input-2-step2.png)

  11. For the last step, take the output investigation from the second step and send that to the SNS topic where our owner, developers and admin are subscribed.

  12. Specify the **Step name** as `AWSPublishSNSNotification` and the action type as `aws:executeAutomation`. 
  13. Specify `AWS-PublishSNSNotification` as the **Document name** and `RuntimeParameters` as shown below. This will take the output of the second step which contains summary data of the investigation and AWS-PublishSNSNotification which will send an email to the SNS we specified in the parameters.


  ```
  TopicArn: '{{SNSTopicARN}}'
  Message: '{{ PlaybookInvestigateAppResourcesELBECSRDS.Output }}'
  ```

  ![Section4 ](/Images/section4-create-automation-parameter-input-2-step3.png)

  14. Our **playbook** will run investigative tasks and send the result to an SNS topic where our Systems administrator / engineer will subscribe to. To do this we will need to create an SNS topic that our **playbook** will send notification to. Please follow the instructions specified in this [link](https://docs.aws.amazon.com/sns/latest/dg/sns-create-topic.html) and create a Standard SNS topic and name it `PlaybookNotificationSNSTopic`

  15. Once you've created the topic, go ahead and subscribe your an email using this instruction [here](https://docs.aws.amazon.com/sns/latest/dg/sns-email-notifications.html)

</details>

### 3.4 Executing investigation Playbook.

You can now run the **playbook** to discover the result of the investigation. 

  1. Go to the **Output** section of the deployed CloudFormation stack `walab-ops-sample-application` and take note of below output values.

  2. Go to the Systems Manager Automation document we just created in the previous step, `Playbook-Investigate-Application-From-Alarm`.
  
  3. And then run the **playbook** passing the ARN as the **AlarmARN** input value, along with the **SNSTopicArn**.
     
     * You can get the **AlarmARN** from the email that you received from CloudWatch Alarm as described in step **3.1 Building the "Gather-Resources" Playbook.** in this lab.
     * To get the value for **SNSTopicArn**, go to the CloudFormation console output of `walab-ops-sample-application` stack and copy, paste the value of **OutputSystemEventTopicArn** 

  ![Section3](/Images/section4-create-automation-playbook-test-run-playbook.png)


  4. When the **playbook** completed, an email will be send to you, which contains a summary of the investigation completed by the playbook as shown.

  ![Section3 ](/Images/section4-create-automation-playbook-test-run-playbook-email-summary.png)

  5. Copy and paste the message section and use a json linter tool such as [jsonlint.com](http://jsonlint.com) to give better structure for visibility. The result from the **playbook** investigation might vary slightly, but the overall findings should be similar to the below screenshot.

  ![Section3 ](/Images/section4-create-automation-playbook-test-run-playbook-summary.png)

  6. From the report being generated you should see a large number of **ELB504Count error** and a high **TargetResponseTime** from the Load balancer. This explains the delay we are seeing from our canary alarm. 
  
      If you then look at the ECS summary, you will notice that there is only 1 ECS **TaskRunningCount**, with a relatively high **CPUUtilization** average. The script calculates the average of maximum value on the ECS service in the last 6 minutes window. If you do not see CPUUtilization value in the json, you can confirm this by going to the ECS service console and click on the **Metrics** tab.

      ![Section3 ](/Images/section3-create-automation-playbook-test-run-playbook-cpu.png)

      Therefore, it is likely that the immediate cause of the latency is resource constrained at the application API level running in ECS. Ideally, if we can increase the number of tasks in the ECS service, the application should be able to release some of the CPU Utilization constraints. 
      
      With all of these information provided by our **playbook** findings, we should be able to determine what is the next course of action to attempt remediation to the issue. 

This concludes **Section 3** of this lab, click on the link below to move on to the next section to build the remediation runbook.

### Step 4. Build and Run Remediation Runbook

In contrast to playbooks, **runbooks** are procedures that accomplish specific tasks to achieve an outcome. In the previous section, you have identified an issue with CPU utilization, which occurs because there is only 1 ECS task running in the cluster. This could be remediated through the use of auto-scaling. 

However, implementing this requires preparation and planning. When an incident occurs, operations teams should have a defined escalation path for the issue. Depending on the criticality of the system they should also be equipped to do what is necessary to ensure system availability is protected while the escalation occurs.

In this section, you will build an automated **runbook** to remediate the CPU utilization issue by increasing the number of tasks in the ECS cluster. Your automated **runbook**, will notify the owner of the workload and give them the option to be able to intercept the scale-up action should they choose not to proceed. 

#### Actions items in this section:

1. You will build a **runbook** to scale up the ECS cluster, with the approval mechanism.
2. You will execute the **runbook** and observe the recovery of your application. 

### 4.0 Building the "Approval-Gate" Runbooks.

In this section you will build a reusable **runbook**, which provides the owner with the ability to deny or approve remediation actions within a defined waiting period. If the wait time is exceeded and a decision has has not been made, the runbook will automatically approve the action as shown. 

  ![Section5 ](/Images/section5-create-automation-graphics1.png)

We will achieve this through the use of a Systems Manager Automation document, which we will build using the following steps:

1. The `Approval-Gate` **runbook** executes a separate document called the `Approve-Timer`. 

2. The `Approve-Timer` **runbook** will then wait for a preconfigured amount of time and send an approve signal to the `Approval-Gate` **runbook**.

3. Meanwhile, the `Approval-Gate` **runbook** then sends an approval request to the workload owner via a designated SNS topic.

  * If the owner choose to approve, the `Approval-Gate` **runbook** will continue to the next step. 
  * If the owner declines the approval, the **runbook** will fail, blocking further steps.
  * However, if the owner does not response within the preconfigured wait time, the `Approve-Timer` **runbook** will automatically approve the request.

Follow the instructions below to build the runbook:

> **Note:** Select a step-by-step guide below to build the runbook using either the AWS console or CloudFormation template.

<details>
<summary> Click here for Console step by step </summary>

1. Go to the AWS Systems Manager console. Click **Documents** under **Shared Resources** on the left menu. Then click **Create Automation** as show in the screen shot below:

      ![Section5 ](/Images/section5-create-automation.png)

2. Enter `Approval-Timer` in the **Name** field and copy the notes shown below into the **Document description** field. 

      ```
        # What does this automation do?

        Automatically trigger 'Approval' Signal to an execution, after a timer lapse

        ## Steps 

        1. Sleep for X time specified on the parameter input
        2. Automatically signal 'Approval' to the Execution specified in parameter input
      ```

3. In the **Assume role** field, enter the IAM role ARN we created in the previous section **3.0 Prepare Automation Document IAM Role**.

4. Expand the **Input Parameters** section and enter `Timer` as the **Parameter name**. Set the type as `String` and **Required** as `Yes`. 

5. Then add another parameter this time called `AutomationExecutionId`, of type `String` and set **Required** to `Yes`. Once you are done, your configuration should look like the screenshot below.

      ![Section4 ](/Images/section4-approve-timer-input-param.png)

6. Under **Step 1** section specify `SleepTimer` as  **Step name**, select `aws::sleep` as the **Action type**. 

7. Expand the **Inputs** section of the step, and specify `{{Timer}}` as the **Duration**

      ![Section4 ](/Images/section4-approve-timer-step1.png)


8. Click on **Add step** and specify `ApproveExecution` as **Step name**, select `aws::executeAwsApi` as the **Action type**.

9. Expand the **Inputs** section of the step, and specify `ssm` in the **Service** field and `SendAutomationSignal` in the API field.

10. Under **Additional inputs** specify below values.
  
      * `Approve` as the **SignalType** 
      * `{{AutomationExecutionId}}` as the **AutomationExecutionId**.

    Once you are done, your configuration should look like the screenshot below.

      ![Section5 ](/Images/section5-create-automation-step2.png)

      ![Section5 ](/Images/section5-create-automation-step2-input.png)

6 . Click on **Create automation** once you are done.

Next, you will create the `Approval-Gate` **runbook** responsible for running the `Approval-Timer` **runbook** asynchronously. Follow below steps to complete the configuration:

1. From the AWS Systems Manager console, select **Documents** under **Shared Resources** on the left menu. Then click **Create Automation** as show in the screen shot below:

      ![Section5 ](/Images/section5-create-automation.png)

2. Next, enter `Approval-Gate` in the **Name** field and add the notes shown below to the **Document description** field. 

      ```
        # What does this automation do?

        Place a gate before your desired step to create approval mechanism.
        Automation will trigger an asynchronously timer that will automatically approve once the time has lapsed.
        Automation will then send approval / deny request to the designated SNS Topic.
        When deny is triggered by approver, the step will fail and block the following step from executing.

        Note: Please ensure to have onFailure set to abort in your automation document.

        ## Steps 

        1. Trigger an asynchronously timer that will automatically approve once the time has lapsed.
        2. Send approval / deny request to the designated SNS Topic.

      ```

3. In the **Assume role** field, enter the IAM role ARN we created in the previous section **3.0 Prepare Automation Document IAM Role**.

4. Expand the **Input Parameters** section and enter the following:
   
    * `Timer` as the **Parameter name**, set the type as `String` and **Required** as `Yes`. 
    * `NotificationMessage` as the **Parameter name**, set the type as type `String` and **Required** is `Yes`.
    * `NotificationTopicArn` as the **Parameter name**, set the type as type `String` and **Required** is `Yes`.
    * `ApproverRoleArn` as the **Parameter name**, set the type as type `String` and **Required** is `Yes`.

5. Expand **Step 1** create a step named `executeAutoApproveTimer` and action type `aws:executeScript`. 

6. Expand **Inputs**, then set the **Runtime** as `Python3.6` and paste in below code into the script section. Note that code snippet will execute the `Approval-Timer` **runbook** you created asyncronously.

    ```
    import boto3
    def script_handler(event, context):
      client = boto3.client('ssm')
      response = client.start_automation_execution(
          DocumentName='Approval-Timer',
          Parameters={
              'Timer': [ event['Timer'] ],
              'AutomationExecutionId' : [ event['AutomationExecutionId'] ]
          }
      )
      return None
    ```

6. Expand **Additional Inputs**, then select `InputPayload` under **Input Name**, and add the text shown below to  **Input Value**:

    ```
    AutomationExecutionId: '{{automation:EXECUTION_ID}}'
    Timer: '{{Timer}}'
    ```
    Once you have completed this step, your **Step 1** configuration should look like below screenshot.

    ![Section4 ](/Images/section4-create-approval-gate-step1.png)

7. Click **Add step** to create **Step 2**

8. Create a step named `ApproveOrDeny` and action type `aws:approve`. 

9. Expand **Inputs** and specify below values under **Approvers**, replacing the `AutomationRoleArn` with the Arn of **AutomationRole** you took note of in section **3.0 Prepare Automation Document IAM Role**.

    ```
    [ '{{ApproverRoleArn}}', 'AutomationRoleArn' ]
    ```

    Example:

    ```
    [ '{{ApproverRoleArn}}', 'arn:aws:iam::xxxxx:role/AutomationRole' ]
    ```


10. Expand **Additional Inputs** and specify the following values:

    * `NotificationArn` as the **Input name**, and `{{NotificationTopicArn}}` as the **Input value** 
    * `Message` as the **Input name**, and `{{NotificationMessage}}` as the **Input value** 
    * `MinRequiredApprovals` as the **Input name**, and `1` as the **Input value** 

12. Expand **Common properties** and change the following properties to below values (keep the remaining as it is):

    * `Continue` for **On failure**
    * `false` for **Is critical**

    Once you have completed this step, your **Step 2** configuration should look like below screenshot.

      ![Section4 ](/Images/section4-create-approval-gate-step2.png)



13. Click **Add step** to create **Step 3**

14. Create a step named `getApprovalStatus` and action type `aws:executeAwsApi`

15. Expand **Inputs** and specify `ssm` in the **Service** field, and `DescribeAutomationStepExecutions` in the **API** field.

16. Expand **Additional Inputs** and specify below values:

    * `AutomationExecutionId` as the **Input Name**, and `{{automation:EXECUTION_ID}}` as the **Input value**
    * `Filters` as the **Input Name**, and copy below values as the **Input value**

      ```
        - Key: StepName
          Values:
            - requestApproval
      ```
17. Expand **Outputs** and specify below values:

    * `approvalStatusVariable` as the **Name**
    * `$.StepExecutions[0].Outputs.ApprovalStatus[0]` as the **Selector**
    * `String` as the **Type**

    Once you have completed this step, your **Step 3** configuration should look like below screenshot.

      ![Section4 ](/Images/section4-create-approval-gate-step3.png)


18. Click on **Create automation** to complete the configuation.

</details>

<details>
<summary> Click here for CloudFormation deployment steps </summary>

Download the template [here.](/Code/templates/runbook_approval_gate.yml "Resources template")

If you decide to deploy the stack from the console, ensure that you follow below requirements & step:

  1. Please follow this [guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) for information on how to deploy the CloudFormation template.
  2. Use `waopslab-runbook-approval-gate` as the **Stack Name**, as this is referenced by other stacks later in the lab.

</details>

<details>
<summary> Click here for CloudFormation CLI deployment step (Preferred way) </summary>

1. From the Cloud9 terminal, change to the templates folder as shown:

    ```
    cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/templates
    ```


2. Run the below commands, replacing the `AutomationRoleArn` with the Arn of **AutomationRole** you took note of in section **3.0 Prepare Automation Document IAM Role**.

    ```
    aws cloudformation create-stack --stack-name waopslab-runbook-approval-gate \
                                    --parameters ParameterKey=PlaybookIAMRole,ParameterValue=AutomationRoleArn \
                                    --template-body file://runbook_approval_gate.yml 
    ```
    
    With your AutomationRole Arn in place your command will look similar to the following example:

    ```
    aws cloudformation create-stack --stack-name waopslab-runbook-approval-gate \
                                    --parameters ParameterKey=PlaybookIAMRole,ParameterValue=arn:aws:iam::000000000000:role/xxxx-runbook-role \
                                    --template-body file://runbook_approval_gate.yml 
    ```
    
3. Confirm that the stack has installed correctly. You can do this by running the **describe-stacks** command below, locate the **StackStatus** and confirm it is set to **CREATE_COMPLETE**. 

```
aws cloudformation describe-stacks --stack-name waopslab-runbook-approval-gate
```

</details>

### 4.1 Building the "ECS-Scale-Up" runbook.

  ![Section5 ](/Images/section5-create-automation-graphics2.png)

Next, you are going to build the ECS-Scale-Up **runbook** which will complete the following:

1. Run the `Approval-Gate` **runbook** which you created previously. 
2. Wait for the `Approval-Gate` **runbook** to complete.
3. Once the `Approval-Gate` **runbook** completes successfully, the runbook will increase the number of ECS tasks in the cluster.

Please follow below steps to build the runbook.

> **Note:** Select a step-by-step guide below to build the runbook using either the AWS console or CloudFormation template.

<details>
<summary> Click here for Console step by step </summary>

1. Go to the AWS Systems Manager console. Click **Documents** under **Shared Resources** on the left menu. Then click **Create Automation** as show in the screen shot below.

      ![Section5 ](/Images/section5-create-automation.png)

2. Next, enter `Runbook-ECS-Scale-Up` in the **Name** field and add the notes shown below to the **Document description** field:

      ```
        # What does this automation do?

        Scale up a given ECS service task desired count to certain number, with approval process.
        The automation will trigger Approval-Gate runbook, before executing.

        ## Steps 

        1. Trigger Approval-Gate
        2. Scale ECS Service by number of service
      ```

3. In the **Assume role** field, enter the IAM role ARN we created in the previous section **3.0 Prepare Automation Document IAM Role**.

4. Expand the **Input Parameters** section and enter the following.
   
    * `ECSDesiredCount` as the **Parameter name**, set the type as `Integer` and **Required** as `Yes`. 
    * `ECSClusterName` as the **Parameter name**, set the type as `String` and **Required** is `Yes`.
    * `ECSServiceName`, as the **Parameter name**, set the type as `String` and **Required** is `Yes`.
    * `NotificationTopicArn`, as the **Parameter name**, set the type as `String` and **Required** is `Yes`.
    * `NotificationMessage`, as the **Parameter name**, set the type as `String` and **Required** is `Yes`.    
    * `ApproverRoleArn`, as the **Parameter name**, set the type as `String` and **Required** is `Yes`.
    * `Timer`, as the **Parameter name**, set the type as `String` and **Required** is `Yes`.


5. Expand **Step 1** create a step named `executeApprovalGate` and action type `aws:executeAutomation`. 

6. Expand **Inputs**, then set the  **Document name** as `Approval-Gate`.

7. Expand **Additional inputs** and select  `RuntimeParameters` as the **Input Name**

8. Paste in below as the **Input Value**

  ```
  {
  "Timer":'{{Timer}}',
  "NotificationMessage":'{{NotificationMessage}}',
  "NotificationTopicArn":'{{NotificationTopicArn}}',
  "ApproverRoleArn":'{{ApproverRoleArn}}'
  }
  ```

9. Click **Add Step** to create the second step.

10. Specify `updateECSServiceDesiredCount` as **Step Name** and select `aws:executeAwsApi` as Action type. 

11. Expand **Inputs** and configure the following values:

    * `ecs` as **Service**  
    * `UpdateService` as **Api**
    
12. Expand **Additional inputs** and configure the following values:

    * `forceNewDeployment` as the **Input Name** and `true` as **Input Value**
    * `desiredCount`as the **Input Name** and `{{ECSDesiredCount}}` as **Input Value**
    * `service` as the **Input Name** and `{{ECSServiceName}}` as **Input Value**
    * `cluster` as the **Input Name** and `{{ECSClusterName}}` as **Input Value**

13 . Click on **Create automation** once complete


</details>

<details>
<summary> Click here for CloudFormation Console deployment step </summary>

Download the template [here.](/Code/templates/runbook_scale_ecs_service.yml "Resources template")

If you decide to deploy the stack from the console, ensure that you complete the following steps:

  1. Please follow this [guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) for information on how to deploy the CloudFormation template.
  2. Use `waopslab-runbook-scale-ecs-service` as the **Stack Name**, as this is referenced by other stacks later in the lab.

</details>

<details>
<summary> Click here for CloudFormation CLI deployment step (Preferred way) </summary>

1. From the Cloud9 terminal, run the command to get into the working script folder.

    ```
    cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/templates
    ```

2. Then run below commands, replacing the 'AutomationRoleArn' with the Arn of **AutomationRole** you took note in previous step **3.0 Prepare Automation Document IAM Role**.
  
    ```
    aws cloudformation create-stack --stack-name waopslab-runbook-scale-ecs-service \
                                    --parameters ParameterKey=PlaybookIAMRole,ParameterValue=AutomationRoleArn \
                                    --template-body file://runbook_scale_ecs_service.yml 
    ```
    Example:

    ```
    aws cloudformation create-stack --stack-name waopslab-runbook-scale-ecs-service \
                                    --parameters ParameterKey=PlaybookIAMRole,ParameterValue=arn:aws:iam::000000000000:role/AutomationRole \
                                    --template-body file://runbook_scale_ecs_service.yml 
    ```

3. Confirm that the stack has installed correctly. You can do this by running the **describe-stacks** command below, locate the **StackStatus** and confirm it is set to **CREATE_COMPLETE**. 


```
aws cloudformation describe-stacks --stack-name waopslab-runbook-scale-ecs-service
```

</details>

### 4.2 Executing remediation Runbook.

Now, lets run the **runbook** you created above to remediate the issue.

  1. Go to the AWS CloudFormation console.
  
  2. Click on the stack named `walab-ops-sample-application`. 
  
  3. Click on the **Output** tab, and take note following output values. You will need these values to execute the runbook. 
  
      * OutputECSCluster
      * OutputECSService
      * OutputSystemOwnersTopicArn

  ![ Section4 ](/Images/section4-output.png)

  4. If you are currently using an IAM user or role to log into your AWS Console, take note of the ARN. 
     You will need this ARN when executing the **runbook** to restrict access to approve or deny request capability.

     To find your current IAM user ARN, go to the IAM console and click **Users** on the left side menu, then click on your **User** name. 
     For IAM role, go to the IAM console and click **Roles** on the left side menu, then click on the **Role** name, you are using. 
     
     You will see something similar to the example below. Take note of the ARN value,and proceed to the next step.

  ![ Section4 ](/Images/section4-iam.png)

  5. Go to the Systems Manager Automation console, click on **Document** under **Shared Resources**, locate and click an automation document called `Runbook-ECS-Scale-Up`. 
  
  8. Then click **Execute automation**.
  
  7. Fill in the **Input parameters** with values below. 

      ![ Section4 ](/Images/section4-scale-up.png)

      * For **ECSServiceName**, place the value of **OutputECSService** you took note on step 3.
      * For **ECSClusterName**, Place the value of **OutputECSCluster** you took note on step 3. 
      * For **ApproverArn**, place the ARN value you took note on step 4.
      * For **ECSDesiredCount**, place in `100` to increase the task number to 100. 
      * For **NotificationMessage**, place in any message that can help the approver make an informed decision when approving or denying the requested action. 
      
        For example:
        ```
        Hello, your mysecretword app is experiencing performance degradation. To maintain quality customer experience we will manually scale up the supporting cluster. This action will be approximately 10 minutes after this message is generated unless you do not consent and deny the action within the period.
        ```  

      * For **NotificationTopicArn**, place the value of **OutputSystemOwnersTopicArn** you took note on step 3.
      * For **Timer**, you can specify `PT5M` or specify a value defined in ISO 8601 duration format.
  
  5. Click **Execute** to run the **runbook**.

  6. Once the **runbook** is running, you will receive an email with instructions approve or deny, on the email address subscribed to the owners SNS topic ARN. 
     Follow the link in the email using the User of the ApproverArn you placed in the Input parameters. The link will take you to the SSM Console where you can approve or deny the request.
  

      ![ Section4 ](/Images/section4-approveordeny.png)

      If you approve, or ignore the email, the request will be automatically be approved after the Timer set in the runbook expires.
      If you deny, the **runbook** will fail and no action will be taken.

  7. Once the **runbook** completes, you can see that the ECS task count increased to the value specified. 

  8. Go to ECS console and click on **Clusters** and select `mysecretword-cluster`. 
  
  9. Click on the  `mysecretword-service` **Service**, and you will see the number of running tasks increasing to 100 and the average CPUUtilization decrease.

      ![ Section4 ](/Images/section4-scale-up2.png)

      ![ Section4 ](/Images/section4-scale-up3.png)

  9. Subsequently, you will see the API response time returns to normal and the CloudWatch Alarm returns to an OK state. 

      ![ Section4 ](/Images/section4-normal.png)

     You can check both using your CloudWatch Console, following the steps you ran in section **2.1 Observing the alarm being triggered**.


#### Congratulations ! 
You have now completed the **Automating operations with Playbooks and Runbooks** lab, click on the link below to cleanup the lab resources.


## Teardown
In this section you will delete all resources related to the lab environment.

1. Run the following command to navigate to the script folder.

```
cd ~/environment/aws-well-architected-labs/static/Operations/200_Automating_operations_with_playbooks_and_runbooks/Code/scripts/
```

2. Run the teardown_resources.sh script to delete all resources related to the lab.
```
bash teardown_resources.sh
```
## Summary
In this lab you learnt:
- Build and run automated playbooks to support your investigations
- Build and run automated runbooks to remediate specific faults
- Enabling traceability of operations activities in your environment

## Survey
Let us know what you thought of this session and how we can improve the presentation experience for you in the future by completing this [event session poll](https://amazonmr.au1.qualtrics.com/jfe/form/SV_1U4cxprfqLngWGy?Session=HOL10). Participants who complete the surveys from AWS Innovate Online Conference will receive a gift code for USD25 in AWS credits1, 2 & 3. AWS credits will be sent via email by September 29, 2023.

Note: Only registrants of AWS Innovate Online Conference who complete the surveys will receive a gift code for USD25 in AWS credits via email.

1. AWS Promotional Credits Terms and conditions apply: https://aws.amazon.com/awscredits/
2. Limited to 1 x USD25 AWS credits per participant.
3. Participants will be required to provide their business email addresses to receive the gift code for AWS credits.
