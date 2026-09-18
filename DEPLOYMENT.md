# Deployment Evidence

Fill this in as you go. Paste real output, not descriptions of output. A TA reads this
file with you at recitation.

## 1. Deployed URL and instance id

<!-- The ServiceUrl and InstanceId outputs. Paste both here every time
describe-stacks prints them, for the healthy deploy and for scenario 2. Both
change on every recreate, and you will need them for curls and sessions. -->

### First Print: 
InstanceId - i-0b8cce31ad91417ed                                   
ServiceUrl - http://ec2-34-207-78-8.compute-1.amazonaws.com:8080  

### Second Print:
InstanceId - i-0ede858abca6fae3b                                    
ServiceUrl - http://ec2-54-92-206-27.compute-1.amazonaws.com:8080  

### Third Print: 
InstanceId - i-05e82442660441002
ServiceUrl - http://ec2-34-229-133-55.compute-1.amazonaws.com:8080

## 2. External health check

Run the check from your own machine, not from the instance. Paste the command and the
response.

```
curl http://ec2-34-207-78-8.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}                     
```

## 3. What the template created

Three or four sentences, your own words. What compute, what network access, and what
glue made the service start.

<!-- Your answer here. -->

The template created a `t3.micro` EC2 instance running Amazon Linux 2023. It created a security group that allows inbound TCP traffic on the service port (8080) from the internet, along with SSH access on port 22. The instance's user-data script installed Docker, enabled it at boot, and ran the `lab04-service` container with the configured image, port mapping, and `PORT` environment variable. CloudFormation's `Fn::Base64` and `!Sub` functions provide the startup script with the template parameter values.

## 4. Scenario 2 diagnosis

**The failing curl** (command and output):

```
curl http://ec2-54-92-206-27.compute-1.amazonaws.com:8080/api/health
curl: (7) Failed to connect to ec2-54-92-206-27.compute-1.amazonaws.com port 8080 after 33 ms: Couldn't connect to server
```

**The log line that told you what was wrong:**

```
lab04-service listening on 9090
```

**What was wrong, and the fix you applied:**

<!-- One or two sentences. Say what you changed and where you changed it. -->
The security group and Docker host mapping expose port 8080, but the PortOverride parameter caused the application inside the container to listen on port 9090. I fixed it by redeploying with PortOverride set to empty, so the service listens on port 8080

**The healthy curl after the fix:**

```
curl http://ec2-34-229-133-55.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}%
```

## 5. Teardown proof

Paste the delete output, or describe the console evidence that the resources are gone.

```
aws cloudformation describe-stacks --stack-name lab04-service
aws: [ERROR]: An error occurred (ValidationError) when calling the DescribeStacks operation: Stack with id lab04-service does not exist
```

Tools/Models Used: ChatGPT Codex gpt-5.6-luna medium effort