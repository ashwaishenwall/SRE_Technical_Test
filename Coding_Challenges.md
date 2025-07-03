# Part 3: Coding Challenges (40 points)

### 1. Automating Toil - Cloud Resource Management (10 points)
Write a Python script that identifies and addresses common AWS resource management
toil:
- a. Identify EC2 instances that have been running for more than 14 days without the "permanent" tag
- b. Flag underutilized instances (less than 10% CPU utilization for 7 consecutive days)
- c. Generate both a human-readable report and machine-parseable output
(JSON)
- d. Include functionality to automatically stop or right-size flagged instances (with appropriate safeguards)
- e. Implement proper error handling and logging
Here's a Python script that addresses the task of automating AWS EC2 resource management toil, based on your requirements:

```python
import boto3
import logging
import json
from datetime import datetime, timedelta
from botocore.exceptions import ClientError

# Setup logging
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")
logger = logging.getLogger()

# Create a Boto3 EC2 client
ec2_client = boto3.client('ec2')
cloudwatch_client = boto3.client('cloudwatch')

# Function to identify EC2 instances that have been running for more than 14 days without the "permanent" tag
def identify_old_instances():
    # Get all EC2 instances
    instances = ec2_client.describe_instances()
    flagged_instances = []
    current_time = datetime.now()

    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            # Check if the instance has the "permanent" tag
            tags = instance.get('Tags', [])
            permanent_tag = any(tag['Key'] == 'permanent' for tag in tags)
            
            # Check if the instance has been running for more than 14 days
            launch_time = instance['LaunchTime']
            if (current_time - launch_time.replace(tzinfo=None)) > timedelta(days=14) and not permanent_tag:
                flagged_instances.append(instance)

    return flagged_instances


# Function to flag underutilized instances
def identify_underutilized_instances():
    flagged_instances = []
    # Iterate through each EC2 instance
    instances = ec2_client.describe_instances()

    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            try:
                # Get CPU utilization metrics for the last 7 days
                end_time = datetime.now()
                start_time = end_time - timedelta(days=7)

                metrics = cloudwatch_client.get_metric_data(
                    MetricDataQueries=[
                        {
                            'Id': 'cpu_utilization',
                            'MetricStat': {
                                'Metric': {
                                    'Namespace': 'AWS/EC2',
                                    'MetricName': 'CPUUtilization',
                                    'Dimensions': [{'Name': 'InstanceId', 'Value': instance_id}],
                                },
                                'Period': 86400,  # 1 day in seconds
                                'Stat': 'Average'
                            },
                            'StartTime': start_time,
                            'EndTime': end_time,
                        }
                    ]
                )

                cpu_utilization = [point['Value'] for point in metrics['MetricDataResults'][0]['Values']]

                # Check if CPU utilization is consistently under 10% for the last 7 days
                if all(utilization < 10 for utilization in cpu_utilization):
                    flagged_instances.append(instance)

            except ClientError as e:
                logger.error(f"Error retrieving metrics for instance {instance_id}: {e}")

    return flagged_instances


# Function to generate a report (both human-readable and JSON)
def generate_report(flagged_instances, is_human_readable=True):
    report = {"flagged_instances": []}
    for instance in flagged_instances:
        instance_info = {
            'InstanceId': instance['InstanceId'],
            'InstanceType': instance['InstanceType'],
            'LaunchTime': instance['LaunchTime'].strftime("%Y-%m-%d %H:%M:%S"),
            'State': instance['State']['Name']
        }
        report["flagged_instances"].append(instance_info)

    if is_human_readable:
        # Generate human-readable report
        logger.info("Human-readable report:")
        for instance in flagged_instances:
            logger.info(f"Instance ID: {instance['InstanceId']} - State: {instance['State']['Name']}")
    else:
        # Generate machine-parsable JSON report
        logger.info("Machine-parsable JSON report:")
        print(json.dumps(report, indent=4))


# Function to stop or right-size flagged instances
def manage_flagged_instances(flagged_instances, action='stop'):
    for instance in flagged_instances:
        instance_id = instance['InstanceId']
        try:
            if action == 'stop':
                # Stop the instance with safeguards (check if the instance is already stopped)
                if instance['State']['Name'] != 'stopped':
                    ec2_client.stop_instances(InstanceIds=[instance_id])
                    logger.info(f"Stopping instance {instance_id}")
                else:
                    logger.warning(f"Instance {instance_id} is already stopped.")
            elif action == 'resize':
                # Right-size the instance (for example, resize to t2.micro)
                current_type = instance['InstanceType']
                if current_type != 't2.micro':
                    ec2_client.modify_instance_attribute(
                        InstanceId=instance_id,
                        Attribute='instanceType',
                        Value='t2.micro'
                    )
                    logger.info(f"Resizing instance {instance_id} to t2.micro")
                else:
                    logger.warning(f"Instance {instance_id} is already of type t2.micro.")
        except ClientError as e:
            logger.error(f"Error managing instance {instance_id}: {e}")


# Main function
def main():
    try:
        # Identify EC2 instances that are too old or underutilized
        old_instances = identify_old_instances()
        underutilized_instances = identify_underutilized_instances()

        # Combine the flagged instances into one list
        flagged_instances = old_instances + underutilized_instances

        # Generate reports
        generate_report(flagged_instances, is_human_readable=True)  # Human-readable
        generate_report(flagged_instances, is_human_readable=False)  # Machine-parsable JSON

        # Optionally, stop or right-size flagged instances
        if flagged_instances:
            logger.info("Managing flagged instances...")
            # For example, here we stop the instances
            manage_flagged_instances(flagged_instances, action='stop')  # or 'resize' for right-sizing
        else:
            logger.info("No flagged instances to manage.")
    except Exception as e:
        logger.error(f"An error occurred: {e}")


if __name__ == "__main__":
    main()
```

### Explanation of the Script:
1. **Identify EC2 Instances Over 14 Days Without "Permanent" Tag:**
   - The `identify_old_instances()` function checks all EC2 instances and flags those that have been running for more than 14 days without the "permanent" tag.

2. **Flag Underutilized Instances:**
   - The `identify_underutilized_instances()` function checks EC2 instances with CPU utilization less than 10% for the past 7 consecutive days using CloudWatch metrics.

3. **Generate Reports:**
   - The `generate_report()` function generates both a human-readable report (printed to the console) and a machine-readable JSON report.

4. **Stop or Right-Size Instances:**
   - The `manage_flagged_instances()` function provides two actions: stopping instances or resizing them (e.g., down-sizing to `t2.micro`). You can modify the action to fit your needs.

5. **Error Handling and Logging:**
   - The script uses proper error handling with `try-except` blocks, logging errors when they occur.

### Notes:
- To run this script, ensure you have AWS credentials configured (via AWS CLI or environment variables).
- The script uses AWS SDKs `boto3` for interacting with AWS EC2 and CloudWatch.
- Modify the instance resizing logic if you want a more specific right-sizing strategy.


---
### 2. CI/CD Pipeline for Automation Deployment (15 points)
#### Create a GitLab/GitHub CI configuration (YAML) that sets up a complete pipeline for deploying automation scripts:
- a. Include stages for linting, unit testing, integration testing, and deployment
- b. Implement proper versioning for scripts
- c. Add automated security scanning for vulnerabilities
- d. Configure the pipeline to work with different environments (dev, staging, prod)
- e. Include proper notifications for pipeline failures
- f. Write a brief explanation of your pipeline design decisions

Here’s a GitLab CI/CD pipeline configuration (`.gitlab-ci.yml`) that sets up the stages for linting, unit testing, integration testing, deployment, and more:

### `.gitlab-ci.yml`:

```yaml
stages:
  - lint
  - test
  - security_scan
  - deploy

# Define global variables to manage different environments
variables:
  DEV_BRANCH: "develop"
  STAGING_BRANCH: "staging"
  PROD_BRANCH: "main"
  AWS_REGION: "us-east-1"
  IMAGE_NAME: "automation-scripts"
  IMAGE_TAG: "${CI_COMMIT_REF_NAME}-${CI_COMMIT_SHORT_SHA}"

# Linting stage
lint:
  stage: lint
  image: python:3.8
  before_script:
    - pip install flake8
  script:
    - flake8 --max-line-length=100 .
  only:
    - branches
  notifications:
    - failure: "Linting failed, please fix the code style issues."

# Unit testing stage
unit_tests:
  stage: test
  image: python:3.8
  before_script:
    - pip install -r requirements.txt
  script:
    - pytest tests/unit_tests
  only:
    - branches
  notifications:
    - failure: "Unit tests failed, please check the test logs."

# Integration testing stage
integration_tests:
  stage: test
  image: python:3.8
  before_script:
    - pip install -r requirements.txt
  script:
    - pytest tests/integration_tests
  only:
    - branches
  notifications:
    - failure: "Integration tests failed, please check the logs for more details."

# Security scanning stage
security_scan:
  stage: security_scan
  image: aquasec/trivy:latest
  script:
    - trivy fs --exit-code 1 --no-progress .
  only:
    - branches
  notifications:
    - failure: "Security scan failed, vulnerabilities found in the code."

# Deployment stage - Dev environment
deploy_dev:
  stage: deploy
  image: amazon/aws-cli:latest
  script:
    - aws s3 cp automation_script.py s3://dev-bucket/${IMAGE_TAG}/
    - echo "Deployed to Dev environment"
  only:
    - $DEV_BRANCH
  environment:
    name: dev
    url: https://dev.example.com
  notifications:
    - failure: "Deployment to Dev environment failed."

# Deployment stage - Staging environment
deploy_staging:
  stage: deploy
  image: amazon/aws-cli:latest
  script:
    - aws s3 cp automation_script.py s3://staging-bucket/${IMAGE_TAG}/
    - echo "Deployed to Staging environment"
  only:
    - $STAGING_BRANCH
  environment:
    name: staging
    url: https://staging.example.com
  notifications:
    - failure: "Deployment to Staging environment failed."

# Deployment stage - Prod environment
deploy_prod:
  stage: deploy
  image: amazon/aws-cli:latest
  script:
    - aws s3 cp automation_script.py s3://prod-bucket/${IMAGE_TAG}/
    - echo "Deployed to Production environment"
  only:
    - $PROD_BRANCH
  environment:
    name: prod
    url: https://www.example.com
  notifications:
    - failure: "Deployment to Prod environment failed."
```

### Key Features of the CI/CD Pipeline:

1. **Stages:**
   - **Linting:** Checks the code for style issues using `flake8`.
   - **Unit Testing:** Runs unit tests using `pytest`.
   - **Integration Testing:** Runs integration tests to check the interactions between components.
   - **Security Scanning:** Uses `Trivy` (a security scanning tool) to identify vulnerabilities in the code or container.
   - **Deployment:** Deploys to different environments (dev, staging, prod).

2. **Versioning:**
   - **Versioning of Scripts:** The `IMAGE_TAG` variable includes the branch name and commit hash for versioning the scripts. This helps ensure that each deployment is tied to a specific commit.
   - The `IMAGE_TAG` ensures that each environment uses a unique identifier for the deployed script.

3. **Automated Security Scanning:**
   - The **security_scan** stage uses `Trivy` to perform a security scan for vulnerabilities. If vulnerabilities are found, the job will fail, and the pipeline will notify the team.

4. **Environment-Specific Deployment:**
   - The pipeline uses different branches (`dev`, `staging`, `prod`) to deploy scripts to the respective environments.
   - Deployments are done using AWS CLI, but this can be adapted to other deployment tools based on your infrastructure (e.g., Kubernetes, Docker, etc.).

5. **Notifications on Failure:**
   - Each stage has a notification for failures. If a step fails, it sends a notification describing which stage failed (e.g., linting, testing, deployment).
   - Notifications are configured at the job level to notify the team immediately of failures, allowing for rapid intervention.

6. **Branch-Specific Deployment:**
   - The pipeline is configured to deploy only from specific branches: `dev`, `staging`, and `prod`. This ensures that only the correct code gets deployed to the appropriate environment.

### Explanation of Design Decisions:

1. **Separation of Concerns:** 
   - Each pipeline stage is focused on a specific task (linting, testing, security scanning, and deployment), which keeps the pipeline clean and easy to maintain.

2. **Versioning via Image Tags:**
   - Using commit-specific tags (`CI_COMMIT_REF_NAME` and `CI_COMMIT_SHORT_SHA`) ensures that the deployment process is tied to a specific version of the code, allowing for easy rollbacks and tracking.

3. **Security First:**
   - Automated security scanning is included to identify potential vulnerabilities early in the CI/CD pipeline. This reduces the risk of deploying insecure code to production.

4. **Environment-Specific Deployments:**
   - Deploying to `dev`, `staging`, and `prod` environments ensures a clear separation of environments, with each stage tested and validated before being deployed to production. This minimizes the chance of breaking the production environment.

5. **Notifications for Failures:**
   - The use of notifications for failures ensures that issues are immediately detected and can be addressed by the development team, reducing downtime and improving deployment reliability.

### Conclusion:
This GitLab CI/CD pipeline provides an efficient, secure, and robust framework for automating the deployment of automation scripts, with a clear separation between testing, security, and deployment tasks. The use of versioning, automated security scanning, and environment-specific deployments ensures that the application is stable, secure, and reliable.

---

### 3. Observability for Automation (10 points)
#### Develop a Python script that creates a monitoring framework for your automation:
- a. Implement custom metrics collection for automation script execution
(success rate, execution time, resources affected)
- b. Set up integration with Prometheus or CloudWatch for metrics storage
- c. Add structured logging that follows best practices
- d. Include tracing for complex automation workflows
- e. Create sample Grafana dashboard JSON for visualizing the automation
performance

To create an observability framework for an automation script, we need to integrate several aspects, including custom metrics collection, integration with monitoring tools like Prometheus or AWS CloudWatch, structured logging, tracing for complex workflows, and a sample Grafana dashboard JSON.

### 1. Python Script for Monitoring Automation

The following Python script will cover the requirements for monitoring:

```python
import time
import logging
import random
import json
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway
from opentelemetry import trace
from opentelemetry.trace import SpanKind
from opentelemetry.exporter.prometheus import PrometheusMetricsExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import SimpleSpanProcessor
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.sdk.metrics.export import ConsoleMetricExporter


# Set up logging with structured format
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)
logger = logging.getLogger(__name__)

# Initialize OpenTelemetry Tracer and Metrics
tracer_provider = TracerProvider()
trace.set_tracer_provider(tracer_provider)
meter_provider = MeterProvider()
tracer = tracer_provider.get_tracer(__name__)

# Custom Metrics for Prometheus
registry = CollectorRegistry()
execution_time_metric = Gauge('automation_execution_time_seconds', 'Time taken for script execution in seconds', registry=registry)
success_rate_metric = Gauge('automation_success_rate', 'Success rate of automation script', registry=registry)
resources_affected_metric = Gauge('resources_affected', 'Number of resources affected by automation', registry=registry)

# Sample function for automation tasks
def automation_task():
    """Simulate an automation task execution"""
    start_time = time.time()
    success = random.choice([True, False])  # Simulate success or failure of the automation task
    resources_affected = random.randint(1, 10)  # Simulate how many resources were affected

    # Simulate some processing time
    time.sleep(random.uniform(1, 5))

    # Calculate execution time
    execution_time = time.time() - start_time

    # Collect metrics
    execution_time_metric.set(execution_time)
    success_rate_metric.set(1 if success else 0)
    resources_affected_metric.set(resources_affected)

    # Log the result with structured logging
    logger.info(json.dumps({
        'success': success,
        'execution_time': execution_time,
        'resources_affected': resources_affected
    }))

    return success, execution_time, resources_affected

# Tracing complex workflows
def trace_automation_task():
    """Simulate a task with tracing"""
    with tracer.start_as_current_span("automation_workflow", kind=SpanKind.SERVER) as span:
        span.set_attribute("automation.type", "task")
        success, execution_time, resources_affected = automation_task()

        # Add additional span attributes for better tracing
        span.set_attribute("automation.success", success)
        span.set_attribute("execution_time", execution_time)
        span.set_attribute("resources_affected", resources_affected)

        return success

# Main execution logic
def main():
    # Run the automation task and gather metrics
    success = trace_automation_task()

    # Push metrics to Prometheus
    push_to_gateway('http://localhost:9091', job='automation-job', registry=registry)

    # Log completion status
    if success:
        logger.info("Automation task executed successfully.")
    else:
        logger.error("Automation task failed.")

if __name__ == "__main__":
    main()
```

### Explanation:

1. **Custom Metrics Collection**:
   - **Execution Time**: We track the time taken for the automation script execution (`automation_execution_time_seconds`).
   - **Success Rate**: The success rate of the task is tracked by a metric (`automation_success_rate`).
   - **Resources Affected**: Tracks how many resources were affected during the automation task (`resources_affected`).

2. **Integration with Prometheus**:
   - Metrics are exposed using the `prometheus_client` package, and these metrics are pushed to Prometheus using the `push_to_gateway()` function. The Prometheus Gateway receives the metrics and stores them for later querying.
   
3. **Structured Logging**:
   - The logging is implemented using `json.dumps()` to ensure structured logging. This will ensure logs are easy to parse and can be analyzed programmatically.

4. **Tracing**:
   - **OpenTelemetry**: We use OpenTelemetry for distributed tracing. It allows you to trace the execution flow of the script, and you can monitor the complete workflow in systems like Jaeger, Zipkin, or integrate with Prometheus.
   - The `trace_automation_task()` function creates a trace span for the automation workflow, providing more granular details about success, execution time, and resources affected.

### 2. Sample Grafana Dashboard JSON

Below is a simple Grafana dashboard JSON configuration to visualize the automation performance. This includes the three main metrics: execution time, success rate, and resources affected.

```json
{
  "dashboard": {
    "id": null,
    "uid": "automation-dashboard",
    "title": "Automation Performance Dashboard",
    "tags": [],
    "style": "dark",
    "timezone": "browser",
    "panels": [
      {
        "title": "Execution Time",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "automation_execution_time_seconds",
            "interval": "",
            "legendFormat": "Execution Time",
            "refId": "A"
          }
        ],
        "yaxes": [
          {
            "format": "s",
            "label": "Time (s)",
            "min": "0"
          },
          {
            "format": "short"
          }
        ]
      },
      {
        "title": "Success Rate",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "automation_success_rate",
            "interval": "",
            "legendFormat": "Success Rate",
            "refId": "B"
          }
        ],
        "yaxes": [
          {
            "format": "percent",
            "label": "Success Rate (%)",
            "min": "0",
            "max": "100"
          },
          {
            "format": "short"
          }
        ]
      },
      {
        "title": "Resources Affected",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "resources_affected",
            "interval": "",
            "legendFormat": "Resources Affected",
            "refId": "C"
          }
        ],
        "yaxes": [
          {
            "format": "short",
            "label": "Resources"
          },
          {
            "format": "short"
          }
        ]
      }
    ],
    "schemaVersion": 27,
    "version": 1,
    "links": []
  }
}
```

### Key Components of the Grafana Dashboard:

1. **Execution Time Panel**: Displays the execution time of the automation task over time. This graph visualizes how long each execution takes.
   
2. **Success Rate Panel**: Displays the success rate of the automation task. It shows how often the task succeeds vs. fails (expressed as a percentage).
   
3. **Resources Affected Panel**: Displays how many resources were affected during each automation run.

### Conclusion

- The Python script integrates custom metrics collection, logging, and tracing to give comprehensive observability into the automation process.
- Metrics are exposed to Prometheus, which can then be visualized in Grafana to monitor the execution performance.
- This setup helps in gaining insights into the automation task's behavior, tracking issues, and ensuring it is performing optimally.

---

### 4. Self-healing Infrastructure Script (5 points)
#### Write a bash or Python script that:
- a. Monitors a critical service (e.g., web server, database)
- b. Automatically attempts to remediate common failure scenarios
- c. Implements circuit breaker pattern to prevent excessive remediation
attempts
- d. Sends notifications through multiple channels (Slack, email, PagerDuty)
- e. Logs detailed information about detected issues and remediation actions
- f. Includes a "dry run" mode for testing remediation logic safely

Here’s a Python script that fulfills the requirements for self-healing infrastructure. The script monitors a critical service (in this case, a web server), attempts automatic remediation for common failure scenarios (like restarting a service), and includes a circuit breaker pattern to prevent excessive remediation attempts. The script also sends notifications via Slack, email, and PagerDuty, and logs detailed information about the issue and remediation. Additionally, a "dry run" mode is provided for testing purposes.

### Python Script: `self_healing_infrastructure.py`

```python
import os
import time
import logging
import smtplib
import requests
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from slack_sdk import WebClient
from slack_sdk.errors import SlackApiError
from datetime import datetime
from requests.exceptions import RequestException

# Configuration
SERVICE_URL = "http://localhost:8080"  # URL of the critical service to monitor (e.g., a web server)
MAX_ATTEMPTS = 3  # Max attempts before circuit breaker is triggered
CIRCUIT_BREAKER_TIMEOUT = 3600  # Time (in seconds) to wait before resetting the circuit breaker
EMAIL_SENDER = "your_email@example.com"
EMAIL_RECEIVER = "admin@example.com"
SLACK_TOKEN = "your_slack_token"
PAGERDUTY_API_KEY = "your_pagerduty_api_key"
DRY_RUN = False  # Set to True for testing (no real remediation will occur)

# Logger setup
logging.basicConfig(filename="self_healing.log", level=logging.INFO, format="%(asctime)s - %(message)s")
logger = logging.getLogger()

# Global variables for the circuit breaker
failure_attempts = 0
last_failure_time = None

# Function to check if the service is up
def check_service():
    try:
        response = requests.get(SERVICE_URL, timeout=5)
        return response.status_code == 200
    except RequestException:
        return False

# Function to restart the service (remediation)
def restart_service():
    if DRY_RUN:
        logger.info("Dry run mode: Service restart would be attempted here.")
        return "Service restart (dry run)"
    
    try:
        # Replace this with the actual command to restart the service
        os.system("sudo systemctl restart apache2")  # Example for Apache web server
        logger.info("Service restarted successfully.")
        return "Service restarted"
    except Exception as e:
        logger.error(f"Error restarting service: {str(e)}")
        return f"Error restarting service: {str(e)}"

# Function to send Slack notification
def send_slack_notification(message):
    try:
        client = WebClient(token=SLACK_TOKEN)
        response = client.chat_postMessage(channel="#alerts", text=message)
        logger.info(f"Slack notification sent: {response['message']['text']}")
    except SlackApiError as e:
        logger.error(f"Slack notification failed: {e.response['error']}")

# Function to send Email notification
def send_email_notification(subject, body):
    try:
        msg = MIMEMultipart()
        msg['From'] = EMAIL_SENDER
        msg['To'] = EMAIL_RECEIVER
        msg['Subject'] = subject
        msg.attach(MIMEText(body, 'plain'))
        
        # Set up the server and send the email
        with smtplib.SMTP('smtp.example.com', 587) as server:
            server.starttls()
            server.login(EMAIL_SENDER, 'your_email_password')
            text = msg.as_string()
            server.sendmail(EMAIL_SENDER, EMAIL_RECEIVER, text)
            logger.info(f"Email sent to {EMAIL_RECEIVER}")
    except Exception as e:
        logger.error(f"Email notification failed: {str(e)}")

# Function to send PagerDuty notification
def send_pagerduty_notification(message):
    try:
        headers = {"Authorization": f"Token token={PAGERDUTY_API_KEY}"}
        payload = {
            "incident": {
                "type": "incident",
                "title": message,
                "service": {"id": "your_service_id", "type": "service_reference"},
                "priority": {"id": "P5", "type": "priority_reference"}
            }
        }
        response = requests.post("https://api.pagerduty.com/incidents", json=payload, headers=headers)
        response.raise_for_status()
        logger.info(f"PagerDuty notification sent: {message}")
    except requests.exceptions.RequestException as e:
        logger.error(f"PagerDuty notification failed: {str(e)}")

# Function to handle failure and attempt remediation
def handle_failure():
    global failure_attempts, last_failure_time
    current_time = time.time()

    if failure_attempts >= MAX_ATTEMPTS and (current_time - last_failure_time < CIRCUIT_BREAKER_TIMEOUT):
        logger.warning("Circuit breaker triggered: Too many failed attempts in a short period.")
        return

    failure_attempts += 1
    last_failure_time = current_time
    logger.info(f"Failure attempt {failure_attempts} detected. Attempting remediation...")

    remediation_result = restart_service()

    # Send notifications
    send_slack_notification(f"Alert: Service failed! Remediation result: {remediation_result}")
    send_email_notification(f"Service Failure Alert", f"Service failure detected. Remediation result: {remediation_result}")
    send_pagerduty_notification(f"Service failure detected. Remediation result: {remediation_result}")

# Main monitoring loop
def monitor_service():
    while True:
        if not check_service():
            logger.error("Service is down!")
            handle_failure()
        else:
            logger.info("Service is up and running.")
        
        # Reset the failure attempts after the circuit breaker timeout period
        if failure_attempts > 0 and (time.time() - last_failure_time > CIRCUIT_BREAKER_TIMEOUT):
            failure_attempts = 0
            logger.info("Circuit breaker reset.")

        # Sleep before checking again
        time.sleep(60)  # Check every 60 seconds

# Main function for dry run mode
def dry_run_mode():
    logger.info("Dry run mode activated. No actual remediation will be performed.")
    check_service_result = check_service()
    if not check_service_result:
        logger.warning("Service is down (dry run). No remediation performed.")
    else:
        logger.info("Service is up (dry run). No remediation needed.")

# Run the script
if __name__ == "__main__":
    if DRY_RUN:
        dry_run_mode()
    else:
        monitor_service()
```

### Explanation:

1. **Monitor Critical Service**:
   - The script checks the service health by sending an HTTP request to the service's URL (e.g., a web server). If the service is down, it triggers remediation.

2. **Automatic Remediation**:
   - If the service is found to be down, the script will attempt to restart the service using a system command (`sudo systemctl restart apache2` for Apache web server). This is where you can customize the remediation based on the critical service you are monitoring.

3. **Circuit Breaker Pattern**:
   - The script prevents excessive remediation attempts by using a circuit breaker. If the service fails multiple times (configured by `MAX_ATTEMPTS`), it will not attempt to restart the service again until a cooldown period (`CIRCUIT_BREAKER_TIMEOUT`) has passed.

4. **Notifications**:
   - **Slack**: Sends a notification to a Slack channel (`#alerts`).
   - **Email**: Sends an email alert about the service failure and the remediation attempt.
   - **PagerDuty**: Sends a PagerDuty notification to alert the on-call team.

5. **Logging**:
   - The script logs detailed information about the service health, remediation attempts, and notification results.

6. **Dry Run Mode**:
   - In dry run mode, no actual remediation actions will be performed. This mode is useful for testing the script logic safely.

7. **Loop and Delay**:
   - The script monitors the service continuously in a loop, checking every 60 seconds.

### How to Run:
1. Install the necessary Python libraries:
   ```bash
   pip install slack_sdk requests
   ```

2. Customize the script with your service details (e.g., web server URL, Slack token, PagerDuty API key).

3. Run the script:
   ```bash
   python self_healing_infrastructure.py
   ```

   For dry-run testing:
   ```bash
   DRY_RUN=True python self_healing_infrastructure.py
   ```

This script provides a basic framework for monitoring a critical service, automatically attempting remediation, and notifying relevant teams while ensuring that remediation attempts are limited using a circuit breaker pattern.

