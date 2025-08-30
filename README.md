# 🚀 AWS EC2 Launch Monitoring & Alerting System

## 📌 Project Overview
This project is a real-time monitoring and alerting system built on AWS.  
It tracks EC2 instance launches by IAM users and sends an email notification with details such as:
- Who launched the instance  
- When it was launched  
- Instance ID  

The solution integrates **IAM, CloudTrail, EventBridge, Lambda, and SNS**.

---

## 🏗️ Architecture
1. IAM User launches an EC2 instance.  
2. CloudTrail records the `RunInstances` API call.  
3. EventBridge Rule detects the event pattern.  
4. Lambda Function is triggered with event details.  
5. SNS Topic publishes the alert to subscribed email(s).  

---

## ⚙️ Services Used
- IAM – for user/group setup  
- CloudTrail – to capture API calls  
- EventBridge – to filter EC2 launch events  
- Lambda – to process events and send alerts  
- SNS – to deliver notifications via email  

---

## 🔧 Step-by-Step Setup

### 1️⃣ IAM Setup
- Create a group `EC2-Launchers` with AmazonEC2FullAccess.  
- Create an IAM User and add it to the group.  
- Enable AWS Management Console access for the user.  

### 2️⃣ Enable CloudTrail
- Create a new trail `ec2-audit-trail`.  
- Store logs in a new S3 bucket.  

### 3️⃣ Create EventBridge Rule
- Rule name: `ec2-launch-detect`.  
- Event pattern: Match `RunInstances` API call from CloudTrail.  
- Target: Lambda function (to be created).  

### 4️⃣ Create Lambda Function
```python
import json
import boto3

sns_client = boto3.client('sns')

SNS_TOPIC_ARN = "Replace with your ARN"

def lambda_handler(event, context):
    try:
        print("Event: ", json.dumps(event))

        user = event['detail']['userIdentity']['userName']
        event_time = event['detail']['eventTime']
        instance_id = event['detail']['responseElements']['instancesSet']['items'][0]['instanceId']

        message = (
            f"EC2 Instance Launched!\n\n"
            f"User: {user}\n"
            f"Time: {event_time}\n"
            f"Instance ID: {instance_id}"
        )

        sns_client.publish(
            TopicArn=SNS_TOPIC_ARN,
            Message=message,
            Subject="EC2 Launch Alert"
        )

        return {"statusCode": 200, "body": json.dumps("Alert sent successfully!")}

    except Exception as e:
        print("Error: ", str(e))
        return {"statusCode": 500, "body": json.dumps("Error: " + str(e))}

---
