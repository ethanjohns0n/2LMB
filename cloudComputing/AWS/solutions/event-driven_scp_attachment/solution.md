---
doc_type: solution
cloud_provider: AWS
cloud_location: Global
cloud_product:
  - "AWS Organizations"
  - "AWS IAM"
  - "AWS CloudTrail"
  - "AWS S3"
  - "AWS EventBridge"
  - "AWS Lambda"
technical_stack:
  - "Python 3.9"
  - "Node 22"
owner: "ethanjohns0n@outlook.com"
original: True
last_updated: "2025-02-06"
---

![Event-driven scp attachment architecture diagram](assets/event-driven_scp_attachment_architecture_diagram.png)

CloudTrail logs management events from the AWS Organization to an S3 bucket. Then, when specific events like AcceptHandshake occur, CloudTrail sends them to EventBridge. EventBridge has a rule that triggers a Lambda function when that event is detected. The Lambda function then attaches the necessary SCPs to the new account in the organization.

Architecture Diagram Code
```
from diagrams import Diagram, Cluster, Edge
from diagrams.aws.storage import S3
from diagrams.aws.management import Organizations, Cloudtrail, CloudwatchEventEventBased, OrganizationsAccount
from diagrams.aws.compute import Lambda

with Diagram("Event-Driven SCP Attachment Architecture Diagram", show=False, direction="LR"):
    # Source Section
    with Cluster("AWS Organization\n(Source)"):
        org = Organizations("Organization Unit\n(SCP Policies)")
        new_account = OrganizationsAccount("New Account\n(Create/Join)")
        new_account - Edge(style="dashed", label="Membership\nRequest") - org
    
    # Collection Section
    with Cluster("Event Collection"):
        cloudtrail = Cloudtrail("CloudTrail\n(Management Events)")
        s3 = S3("Log Storage\n(S3 Bucket)")
        cloudtrail - Edge(style="dashed") - s3
    
    # Processing Section
    with Cluster("Event Processing\n(Automation)"):
        event_bridge = CloudwatchEventEventBased("EventBridge\n(AcceptHandshake Rule)")
        lambda_func = Lambda("SCP Attachment\nLambda Function")
        event_bridge >> Edge(color="green4", label="Trigger") >> lambda_func
    
    # Flow Connections
    org >> Edge(label="1. Record\nOrg Events") >> cloudtrail
    cloudtrail >> Edge(label="2. Forward\nAcceptHandshake") >> event_bridge
    lambda_func >> Edge(color="red", label="3. Apply SCP\nPolicies") >> org
    
    # Layout alignment helpers (invisible edges)
    new_account - Edge(style="invis") - cloudtrail
    s3 - Edge(style="invis") - event_bridge

```
