# Audit and Control Hands On

In this lab you will build a scenario where an EC2 instance failure will generate a CloudWatch Log. You will then trigger 
a Lambda function in response to the failure that will send an email to the account owner using an SNS Service.



## Deliverables 

## Prerequisites

1. **MAKE SURE YOU DO ALL YOUR WORK ON `us-east-1`**

## Instructions

### Create an EC2 instance

1. Open the EC2 console and create an EC2 instance named `ec2-stop`
    1. The EC2 instance needs to have an SSH key
    1. The EC2 instance needs to be in a public subnet
    1. After creating the EC2 instance copy the `instance-id` you will need it later


### Create and Configure an SNS Topic 

1. Open Amazon SNS console, on the left nagivation pane select "Topics" and then click on "Create Topic."

    <img src="img/open_sns.png" alt="drawing" height="200"/>

        
1. When creating the SNS topic:
    1. Select `Standard`
    1. Enter the name of the topic as `EC2-stop`
    1. Click on Create Topic


        <img src="img/create_sns.png" alt="drawing" height="500"/>

1. After creating the Topic, go the topic you have already created, and click on `Create Subscription`
1. Configure the subscription:
    1. Set the Protocol to `Email`
    1. For Endpoint enter your email.
    1. Click on `Create Subscription`

    <img src="img/click_create_subscription.png" alt="drawing" height="400"/>

1. Check your email and you will find a confirmationi email. Confirm your subscription **Make sure to check your spam/junk**

    <img src="img/confirm_email.png" alt="drawing" height="200"/>


### Create an IAM Policy and Attach it to a Role

1. Open IAM console, in navigator pane select policies and click on create policy.

    <img src="img/create_policy.png" alt="drawing" height="200"/>

1. Configure your IAM Policy:
    1. For service, choose `SNS`
    1. For Actions, select all
    1. For reseources, select `All resources`
    1. Click on `Add additional Permissions`
        1. For service choose `CloudWatch Logs` **It has to be CloudWatch Logs not just CloudWatch
        1. For Actions, select all
        1. For resources, select `All resources`

    1. Click on Next
    1. Click on Next again
    1. Enter a policy name `EC2-stop`, click on `Create Policy`

1. Create a Role.
    1. After creating an IAM policy, go the navigator pane, select Roles and click on `Create Role`

        <img src="img/create_role.png" alt="drawing" height="200"/>

    1. Select `Lambda` as AWS Service and click Next

        <img src="img/select_lambda.png" alt="drawing" height="500"/>

    1. Search the policy `EC2=stop` that you created and click on Next

        <img src="img/attach_policy_to_role.png" alt="drawing" height="200"/>

    1. Enter a name for the role `EC2-stop`

        <img src="img/name_role.png" alt="drawing" height="500"/>


### Create a rule for receiving events in CloudWatch Logs when a specific EC2 instance fails

1. Open Amazon EventBridge console. On the navigation pane click on Rules and `Create Rule`
    1. Give the rule a name and click Next
    1. For event source, select `AWS events of EventBridge partner events.`
    1. Scrolle down to `Creation Method`
        1. For Event Source, choose `AWS services`
        1. For AWS service, choose `EC2`
        1. For Event Type, choose `EC2 Instance State-change Notification`
            1. Select `Specific state(s)` and choose `stopped`
            1. Select `Specific instance Id(s)` and paste the instance-id from the EC2 instance that you created

            <img src="img/creation_method.png" alt="drawing" height="500"/>

### Create a Lambda Function
1. Open AWS Lambda Console and click `Create Function`

    <img src="img/create_lambda.png" alt="drawing" height="200"/>

1. Enter function name as `EC2-stop`
1. For Runtime select `Python 3.9`
1. For Execution Role select `Use an existing role`
    1. Choose the IAM role that you created.
1. Click on Create Function

    <img src="img/create_lambda_function.png" alt="drawing" height="500"/>

1. Paste the following code into the function **Make sure to delete all other code before pasting this code and make sure to replace YOUR ACCOUNT NUMBER with your details. The code will only work if you name your topic `EC2-Stop` otherwise you will have to replace that with the name of your SNS topic**

    ```
    import boto3

    client = boto3.client('sns')
    def lambda_handler(event, context):

        topic_arn = 'arn:aws:sns:us-east-1:YOUR_ACCOUNT_NUMBER:EC2-Stop' 
        message = 'your server is down' 
        client.publish(TopicArn=topic_arn, Message=message)
    ```
1. Click on Deploy
1. Click on Add Trigger

    <img src="img/click_deploy.png" alt="drawing" height="500"/>

1. Choose CloudWatch Logs as trigger
1. Select name of CloudWatch log group that you have already created
1. Enter filter name `EC2-stop` and click on Add

    <img src="img/configure_trigger.png" alt="drawing" height="500"/>

### Test
1. Open EC2 Console
1. Select EC2 instance that you created in the beginning of this lab, click on the instance status and stop the instance
1. Wait until the instance has stoppped
1. 