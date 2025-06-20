**CloudOps Suite: Disk Monitoring & Alerting on AWS (Free Tier)**

This project demonstrates how to set up real-time disk usage monitoring and alerting using free-tier AWS services

**Project Summary**

Set up a CloudWatch monitoring and alerting system on an EC2 instance using:

* Amazon EC2 (Free Tier)
* Amazon CloudWatch Agent
* Custom Metrics (disk usage)
* CloudWatch Alarms
* SNS Email Notifications

**Use Case**  
This setup is ideal for detecting low-disk-space conditions on critical servers and automatically alerting admins/devs via email. It's a foundational DevOps/CloudOps skill.

**Tools & Services Used**
* Tool	
* EC2 (Amazon Linux 2)
* CloudWatch Agent	
* CloudWatch Metrics 
* CloudWatch Alarm	
* SNS

**Architecture**

![CloudOps toolkit drawio](https://github.com/user-attachments/assets/3ad586b9-7c3c-4e23-a859-e42518de8cca)


**Setup Guide**  
**1. Launch EC2 Instance**
* Type: Amazon Linux 2 (t2.micro)
* IAM Role: Attach with CloudWatchAgentServerPolicy

![Instance ](https://github.com/user-attachments/assets/a562c07a-11c6-44d2-931f-163a73401313)

**2. Install & Configure CloudWatch Agent**  
Enter the command in your instance  

sudo yum install amazon-cloudwatch-agent -y     

**Create config file:**  
sudo tee /opt/aws/amazon-cloudwatch-agent/bin/config.json > /dev/null <<'EOF'  
{  
  "agent": {  
    "metrics_collection_interval": 60,  
    "run_as_user": "cwagent"  
  },  
  "metrics": {  
    "append_dimensions": {  
      "InstanceId": "${aws:InstanceId}"  
    },  
    "metrics_collected": {  
      "disk": {  
        "measurement": ["disk_used_percent"],  
        "metrics_collection_interval": 60,  
        "resources": ["/"]  
      }  
    }  
  }  
}  
EOF  

**Start the agent:**

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s

**3. View Metrics in CloudWatch**
* Go to CloudWatch → Metrics → All Metrics → CWAgent
* Check disk_used_percent under your instance’s dimensions

![Cloudwatch](https://github.com/user-attachments/assets/8b1f4784-a555-4ad8-9d8b-89c0df1dce4d)

**4. Create CloudWatch Alarm**
* Metric: disk_used_percent
* Condition: Greater than or equal to 70 for 1 out of 1 datapoints
* Action: Send notification to SNS topic cloudops-alerts

![Alarm](https://github.com/user-attachments/assets/5c1a6ac5-ec9d-4f1d-a5c7-6443646f78ff)

**5. SNS Email Notification**
* Go to SNS → Topics → Create topic
* Subscribe your email
* Confirm subscription in your inbox
* Link this topic to the alarm

**6. Test the Alarm**     
Fill disk intentionally:  
sudo dd if=/dev/zero of=/tmp/bigfile bs=100M count=20  

* Wait 1–2 mins
* Alarm state becomes ALARM
* Email alert is received ✅

Clean up:  
sudo rm -f /tmp/bigfile  


**What I Learned**
* Setting up AWS CloudWatch Agent for custom metrics
* Building real-time monitoring pipelines
* Configuring CloudWatch Alarms and SNS Topics
* Testing alarm triggers in a real-world scenario




