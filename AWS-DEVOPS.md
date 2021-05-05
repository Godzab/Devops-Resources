# AWS DEVOPS PROFESSIONAL

DOMAIN ONE

CICD
- You can do 5 releases a day
	- Code-Deploy
	- Jenkins
	- Spinnaker
- CD means ability to deploy often through automation
- May involve a manual step

CODE  -> BUILD -> TEST -> DEPLOY -> PROVISION


CODE COMMIT
- Github for AWS
- Private repository
- No size limit
- Very secure and compliant
- Integrated with CI tools as Jenkins.

CODE COMMIT - SECURING BRANCHES
- You never push to master or develop
- You can limit pushes to certain branches.
- You can do this in iAM policy.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "codecommit:GitPush",
        "codecommit:DeleteBranch",
        "codecommit:PutFile",
        "codecommit:MergeBranchesByFastForward",
        "codecommit:MergeBranchesBySquash",
        "codecommit:MergeBranchesByThreeWay",
        "codecommit:MergePullRequestByFastForward",
        "codecommit:MergePullRequestBySquash",
        "codecommit:MergePullRequestByThreeWay"
      ],
      "Resource": "arn:aws:codecommit:*:*:*",
      "Condition": {
        "StringEqualsIfExists": {
          "codecommit:References": [
            "refs/heads/master",
            "refs/heads/develop"
          ]
        },
        "Null": {
          "codecommit:References": false
        }
      }
    }
  ]
}
```

NOTIFICATIONS AND TRIGGERS
- Under a repo you can set notifications
- Triggers can also be set that can be assigned to an SNS topic or a lambda function for automation.
- Notification make use of cloudwatch to implement the notifications.



CODE BUILD
- Fully managed build service
- Continuos scalling
- Alternative to tools like jenkins
- You pay for usage, time it takes to complete build.
- Leverages docker under the hood.
- Can provide own docker images
- Secure, uses KMS, IAM for build permissions, VPC for network sec, cloud trail for API calls logging.
- Instructions are in a file called buildspec.yaml
- Can also send notification to SNS through build triggers
- We have a source provider for the build.

buildspec.yaml
```
version: 0.2

phases: 
    install:
        runtime-versions:
            nodejs: 10
        commands:
            - echo "installing something"
    pre_build:
        commands: 
            - echo "we are in the pre build phase"
    build:
        commands:
            - echo "we are in the build block"
            - echo "we will run some tests"
            - grep -Fq "Congratulations" index.html
    post_build:
        commands:
            - echo "we are in the post build phase"
```

```
version: 0.2


run-as: Linux-user-name


env:
  shell: shell-tag
  variables:
    key: "value"
    key: "value"
  parameter-store:
    key: "value"
    key: "value"
  exported-variables:
    - variable
    - variable
  secrets-manager:
    key: secret-id:json-key:version-stage:version-id
  git-credential-helper: no | yes


proxy:
  upload-artifacts: no | yes
  logs: no | yes


phases:
  install:
    run-as: Linux-user-name
    runtime-versions:
      runtime: version
      runtime: version
    commands:
      - command
      - command
    finally:
      - command
      - command
  pre_build:
    run-as: Linux-user-name
    commands:
      - command
      - command
    finally:
      - command
      - command
  build:
    run-as: Linux-user-name
    commands:
      - command
      - command
    finally:
      - command
      - command
  post_build:
    run-as: Linux-user-name
    commands:
      - command
      - command
    finally:
      - command
      - command
reports:
  report-group-name-or-arn:
    files:
      - location
      - location
    base-directory: location
    discard-paths: no | yes
    file-format: JunitXml | NunitXml | CucumberJson | VisualStudioTrx | TestNGXml
artifacts:
  files:
    - location
    - location
  name: artifact-name
  discard-paths: no | yes
  base-directory: location
  secondary-artifacts:
    artifactIdentifier:
      files:
        - location
        - location
      name: secondary-artifact-name
      discard-paths: no | yes
      base-directory: location
    artifactIdentifier:
      files:
        - location
        - location
      discard-paths: no | yes
      base-directory: location
cache:
  paths:
    - path
    - path

```


Example
```
version: 0.2


env:
  variables:
    JAVA_HOME: "/usr/lib/jvm/java-8-openjdk-amd64"
  parameter-store:
    LOGIN_PASSWORD: /CodeBuild/dockerLoginPassword


phases:
  install:
    commands:
      - echo Entered the install phase...
      - apt-get update -y
      - apt-get install -y maven
    finally:
      - echo This always runs even if the update or install command fails 
  pre_build:
    commands:
      - echo Entered the pre_build phase...
      - docker login –u User –p $LOGIN_PASSWORD
    finally:
      - echo This always runs even if the login command fails 
  build:
    commands:
      - echo Entered the build phase...
      - echo Build started on `date`
      - mvn install
    finally:
      - echo This always runs even if the install command fails
  post_build:
    commands:
      - echo Entered the post_build phase...
      - echo Build completed on `date`


reports:
  arn:aws:codebuild:your-region:your-aws-account-id:report-group/report-group-name-1:
    files:
      - "**/*"
    base-directory: 'target/tests/reports'
    discard-paths: no
  reportGroupCucumberJson:
    files:
      - 'cucumber/target/cucumber-tests.xml'
    discard-paths: yes
    file-format: CucumberJson # default is JunitXml
artifacts:
  files:
    - target/messageUtil-1.0.jar
  discard-paths: yes
  secondary-artifacts:
    artifact1:
      files:
        - target/artifact-1.0.jar
      discard-paths: yes
    artifact2:
      files:
        - target/artifact-2.0.jar
      discard-paths: yes
cache:
  paths:
    - '/root/.m2/**/*'
```

- Artifacts are all the files that are going to be kept after our build is done.
- Cache can be specified also to speed up the process.


CODEBUILD ENVIRONMENT VARIABLES AND PARAMETER STORE
- You can inject variables in spec file or on console
- We don’t add in code



CODE BUILD IS USED TO BUILD ITEMS. THE OUTPUT IS AN ARTIFACT WHICH WILL BE UPLOADED SOMEWHERE AND CONSUMED BY OTHER SERVICES.

```
artifacts:
  files:
    - target/messageUtil-1.0.jar          //FILE REQUIRED BY BUILD JOB AS RESULT
```


- You can schedule build events through cloudwatch events.


CODE DEPLOY
- Automatically deploy application versions
- Alternative to (Ansible, terraform, chef, puppet)
- It is a managed service
- Each EC2 machine or on-premise machine must be running the code deploy agent
- Agent continuously Polls AWS code deploy for work to do
- Code deploy send appspec.yaml file
- Application is pulled from source code
- EC2 will run the deployment instructions
- Agent will report success or failure of deployment
- EC2 instances can be grouped into deployment groups.
- Flexibility with deployment
- Can use artifacts and pipelines
- Blue/Green only works when you have EC2 instances and not on-premise.
- Support for lambda and ECS also.
- Does NOT provision resources



DEPLOYMENT GROUPS - CONFIGURATIONS

- These are like target groups for deployments.
- There are strategies same as kubernetes
	- One at a time (rolling updates)
	- Half at a time (half instances down)
	- All at once
- You can create your own strategy in % of healthy instances at a time
- BLUE/GREEN creates new instances for deployments and destroys previous instances. (Load balancer needs to be enabled.)


APPSPEC.YAML DEEP-DIVE
```
version: 0.0
os: linux
files:
  - source: /index.html
    destination: /var/www/html/
hooks:
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root


  AfterInstall:
    - location: scripts/after_install.sh
      timeout: 300
      runas: root


  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root


  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root


  ValidateService:
    - location: scripts/validate_service.sh
      timeout: 300
  
```

- You do not have access to the install, allowTestTraffic, allowTraffic, end hook.
- Hooks:
	-   - BeforeInstall: "BeforeInstallHookFunctionName"
	-   - AfterInstall: "AfterInstallHookFunctionName"
	-   - AfterAllowTestTraffic: "AfterAllowTestTrafficHookFunctionName"
	-   - BeforeAllowTraffic: "BeforeAllowTrafficHookFunctionName"
	-   - AfterAllowTraffic: "AfterAllowTrafficHookFunctionName"

- Traffic hooks are for when you have a LB for traffic draining.
- There are environmental variables available for deployments
	- APPLICATION_NAME
	- DEPLOYMENT_ID
	- DEPLOYMENT_GROUP_NAME
	- DEPLOYMENT_GROUP_ID
	- LIFECYCLE_EVENT
- You can have if statements in the scripts of our appspec.yaml file to target specific environments.

1. CREATE AN APPLICATION FOR THE DEPLOYMENT
2. CREATE A DEPLOYMENT GROUP



CODE DEPLOY INTEGRATIONS
- For logs you need to install the cloudwatch logs agent also on the instances.
- You can set triggers for deployment groups.
- They are attached to specific events and they target an SNS topic.

ROLLBACKS
- You can have manual rollbacks or automatic rollbacks
	- Roll back when a deployment fails
	- Roll back when alarm thresholds are met - create alarm and rollback based on the alarm trigger.

CODE DEPLOY FOR ON-PREM INSTANCES
- You can use STS with IAM role
- You can also use IAM user

CODE DEPLOY FOR LAMBDA

DEPLOYMENT STARTEGIES
- Canary
	- Versions will co-exists for a short while
	- Traffic is shifted in 2 increments, the hesitant iteration and and the full commit.
- Linear
	- Traffic is shifted in equal increments
	- E.g every 1 minute the version 2 application will receive 10% more traffic until full change-over.
	- Incremental and useful to monitor metrics.
	- Multiple increments here
- A lambda function does not need S3 for deployments, its done automatically for you.
- There are only 2 hooks 
```
BeforeAllowTraffic
AfterAllowTraffic
```


CODE PIPELINE
- This is a continuous delivery tool
- Offers a visual workflow
- Source can be any remote repository
- Build can come from code build/ Jenkins/ etc
- Load Testing using 3rd party tools
- Deploy using code deploy / elastic beanstalk / cloudformation
- Made of stages
	- Each stage can have sequential or parallel actions
	- Manual approval can be defined at any stage.
- The key ingredient of code pipeline is artifacts
- They are passed to S3 for the next stage

A pipeline is triggered per branch so you need to create 1 pipeline per desired branch.
You can deploy in a different region if chosen as such.

```
// Cloudwatch rule that triggers code pipeline

{
  "source": [
    "aws.codecommit"
  ],
  "detail-type": [
    "CodeCommit Repository State Change"
  ],
  "resources": [
    "arn:aws:codecommit:us-east-1:650524209925:perion_leads_core"
  ],
  "detail": {
    "event": [
      "referenceCreated",
      "referenceUpdated"
    ],
    "referenceType": [
      "branch"
    ],
    "referenceName": [
      "master"
    ]
  }
}
```
ADDING CODE-BUILD
- Making sure source code is tested before we push to the environment.
- You can add multiple stages and if they are in the same line they are called parallel stages.
- For sequential you just add another action group on the stage.

Services communicate with each other through artifacts
S3 is the backbone of codepipeline

- There are 2 types of artifacts
	- Code build artifacts 
	- Code pipeline artifacts

MANUAL APPROVAL STEPS
- Create an action group which is sequential

Code pipeline uses cloudwtch events to trigger pipeline events when code commit updates.

- The runOrder parameter is used to determine whether or not a pipline stage is sequential or parallel.

CUSTOM ACTION JOBS - LAMBDA
- Allow to extend pipeline to do whatever you want.
- Can add 3rd party packages and do things like post to slack channel.
- The lambda function needs to have a role that can allow access to code pipline actions. Will indicate success or failure.

YOU CAN CREATE A CODE PIPELINE USING CLOUD-FORMATION


codestar
- Integrated environment to quickly build develop and deploy applications.
- Easier way to get started with deployments

Jenkins
- Opensource cicd tool
- Has master/slave setup
- Uses a jenkinsfile
- Can replace any or all code pipeline stages.


AWS CLOUDFORMATION

- Cloudformation is a declarative way of outlining your aws infrastructure.
- It creates resources in the right order and exact configuration that you specify.
- Benefits
	- No manual creation of resources
	- Code for infrastucture can be version controlled
	- Changes to infrastructure can be reviewed through code.

Its friend each stack has an identifier so you can identify costs per stack.
You can estimate costs of resources using cloudformation template
Can automate the creation and deletion of environments to when needed e.g dev - 0800am - 1700pm

It is declarative programming
Seperation of concerns, create stacks for many apps.

HOW IT WORKS
- Templates are uploaded on S3 behind the scenes
- Stacks are identified by name
- Deleting stacks deletes everything within it and created by it.
- Can use 2 ways
	- Manual - template designer
	- Automatic - YAML

TEMPLATE COMPONENTS
- RESOURCES - aws resources (mandatory)
- Parameters - dynamic inputs for you templates
- Mappings - static variables for your templates
- Outputs - exports from template
- conditions
- metadata

TEMPLATE HELPERS
- References - linking stuff
- functions - transform items

You can cater for unsupported resources using lambda resources.

MAPPINGS
- Are fixed variables in your cloudformation templates
- They are handy to differentiate between environments.
- All the values are hardcoded within a template
- Used when value does not need to be user specific
- Retrieving mappings you use the function Fn::FindInMap
```
!FindInMap [MapName, TopLevelKey, SecondaryLevelKey]
```

OUTPUTS
- This section is optional but we can declare optional outputs that we can import into other stacks.
- We can also view the outputs
- We can have a network CF template that can export stuff like VPC ID
- Enables cross stack collaboration
- You cannot delete a stack with outputs still being referenced elsewhere.

```
Outputs:
  StackSSHSecurityGroup:
    Description: This is an output
    Value: !Ref ServerSecurityGroup
    Export:
      Name: SSHSecurityGroup
```
- For imports we use the Fn::ImportValue function.
- You cannot delete the underlaying stack until all the references are deleted too.
```
MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    AvailabilityZone: us-east-1a
    ImageId: ami-009d6802948d06e52
    InstanceType: t2.micro
    SecurityGroups:
      - !Ref SSHSecurityGroup
      - !ImportValue SSHSecurityGroup             # VERY IMPORTANT
```



CONDITIONS
- Used to control the creation of resources or outputs based on a condition.
- Environmental selections (dev/prod)
```
Conditions:
  CreateProductionResources: !Equals[!Ref EnvType, prod]
```
- They use logical operators
	- Fn::And
	- Fn::Equals
	- Fn::Or
	- Fn::If
	- Fn::Not

INTRINSIC FUNCTIONS
- These are the must know functions required for maneuvering CF templates
- These are:
	- Ref
		- Used to reference parameter
		- Used to reference resources (security group)
		- Returns the physical Id of the referenced item
		- Short hand is !Ref
	- GetAtt
		- Gets other attributes excluded by the Ref function.
	- FindInMap
		- Used to find attributes from a defined mapping
		- !FindInMap [MapName, TopLevelKey, SecondaryLevelKey]
	- ImportValue
		- Import values that are exported from other templates
	- Join
		- We can join values with a delimiter.
		- !Join [delimiter, [comma-delimited list of values]]
		- !Join[ ‘ : ’ ,  [ a , b , c] ] => a : b : c
	- Sub
		- Short hand for substitute
		- Very handy to fully customize your templates
		- String must contain ${variable} and will substitute them
	- Conditions (And, If, Or, Not,Equals)

CLOUDFORMATION USERDATA
- We can include user-data in the cloudformation template.
- The entire script should be passed through the function Fn::Base64
- The log will be located in /var/log/cloud-init-output.log



CFN-INIT
- AWS::CloudFormation::Init must be in the metadata of a resource.
- With the cfn-init script it helps make complex EC2 configurations readable.
- Logs go to /var/log/cfn-init.log
```
MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    AvailabilityZone: us-east-1a
    ImageId: ami-009d6802948d06e52
    InstanceType: t2.micro
    KeyName: !Ref SSHKey
    SecurityGroups:
      - !Ref SSHSecurityGroup
    # we install our web server with user data
    UserData: 
      Fn::Base64:
        !Sub |
          #!/bin/bash -xe
          # Get the latest CloudFormation package
          yum update -y aws-cfn-bootstrap
          # Start cfn-init
          /opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyInstance --region ${AWS::Region} || error_exit 'Failed to run cfn-init'
  Metadata:
    Comment: Install a simple Apache HTTP page
    AWS::CloudFormation::Init:
      config:
        packages:
          yum:
            httpd: []
        files:
          "/var/www/html/index.html":
            content: |
              <h1>Hello World from EC2 instance!</h1>
              <p>This was created using cfn-init</p>
            mode: '000644'
        commands:
          hello:
            command: "echo 'hello world'"
        services:
          sysvinit:
            httpd:
              enabled: 'true'
              ensureRunning: 'true'
```

- Almost like configuration management through ansible.

CFN-SIGNAL AND WAIT CONDITIONS
- Determines cfn-init success or failure.
- Run this script to control what happens when cfn-init passes or fails.
- For this we need to define a WaitCondition
	- This blocks a template until it receives a signal from cnf-signal
	- We attach a CreationPolicy (Also works on EC2 and ASG)
	- Dictates how many signals and how long were are willing to wait. 
```
UserData: 
  Fn::Base64:
    !Sub |
      #!/bin/bash -xe
      # Get the latest CloudFormation package
      yum update -y aws-cfn-bootstrap
      # Start cfn-init
      /opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyInstance --region ${AWS::Region}
      # Start cfn-signal to the wait condition
      /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource SampleWaitCondition --region ${AWS::Region}       #Signal is sent here IMPORTANT   
---
SampleWaitCondition:                                                                                                     #Wait Condition for the signal for 2minutes
  CreationPolicy:
    ResourceSignal:
      Timeout: PT2M
      Count: 1                                 #For the amount of instances expected to send the signal. 1 if only 1 instance is being created
  Type: AWS::CloudFormation::WaitCondition
```

CFN-SIGNAL FAILURES TROUBLESHOOTING
- AMI does not have Cloudformation helper scripts installed. (You can download to your instance)
- Verify that the cfn-init and con-signal were successfully run on the instance.
- View logs from cloud-init.log and cnf-init.log
- Verify instance internet connectivity.

CLOUDFORMATION ROLLBACKS
- Stack Creation Fails:
	- Everything gets deleted for any errors on the createStackApi
	- OnFailure=Rollback (Default setting)
	- To troubleshoot we can set to disable rollback
	- OnFailure=DO_NOTHING
- Stack Update Fails (UpdateStack Api)
	- The stack automatically rolls back to the previously known working state
- We can disable rollback on failure so we can debug what happened to cause the failure.


NESTED STACKS
- These are stacks in other stacks
- They allow you to isolate repeated patterns / common components in separate stacks and call them from other stacks.
- E.g reuse loadbalancer configuration, reuse security group.
- To update a nested stack always update the parent (root) stack.
- They are considered as best practice

CHANGESETS
- When you update a stack you need to know what changes are going to be implemented before that happens.
- They won’t say if the update will be successful or not.


CLOUDFORMATION DELETION POLICIES
- Controls the behavior whenever a cloudformation template is deleted.
- DeletionPolicy=Retain:
	- Keeps resources after the stack is deleted.
	- Works for any resources and nestled stacks
- DeletionPolicy=Snapshot:
	- Applies to Databases and storage resources.
	- Instances get deleted but a snapshot is created before that happens.
	- This ensures that we keep data after deletion
- DeletionPolicy=Delete:
	- This is the default.
	- To delete an S3 bucket you first need to make it empty
- Policies are on Resource level.

CLOUDFORMATION TERMINATION PROTECTION
- To prevent accidental deletes of cloudformation templates.


——————————————————————————————————    

CLOUDFORMATION ADVANCED

PARAMETERS FROM SSM
- Injection of parameters into cloudformation
```
Parameters:
  InstanceType:
    Type: 'AWS::SSM::Parameter::Value<String>’           #VERY IMPORTANT
    Default: /EC2/InstanceType


  ImageId: 
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>’ #VERY IMPORTANT
    Default: /EC2/AMI_ID
```
- When you update a parameter you have to trigger a stack update again and the changes will be put in effect.


CLOUDFORMATION PUBLIC PARAMETERS FROM SSM
- Amazon has public parameters that we can reference in our stacks.
- Using the below picks the latest AMIs in the respective categories.
```
Parameters:
  LatestLinuxAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    # obtain list with
    # aws ssm get-parameters-by-path --path /aws/service/ami-amazon-linux-latest  --query 'Parameters[].Name'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'


  # this works for Windows too
  LatestWindowsAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    # obtain list with
    # aws ssm get-parameters-by-path --path "/aws/service/ami-windows-latest" --region us-east-1
    Default: '/aws/service/ami-windows-latest/Windows_Server-2016-English-Core-Base'
```

CLOUDFROMATION DEPENDS-ON
- As with docker-compose, this is a way to create a sequence in which resources will be created so that dependancies are met.
- Creates a relation between resources.
- Same as on docker-compose services.
```
Resources:
  Ec2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [AWSRegionArch2AMI, !Ref 'AWS::Region', HVM64]
    DependsOn: MyDB                  #IMPORTANT LINE


  MyDB:
    Type: AWS::RDS::DBInstance
    Properties:
      AllocatedStorage: '5'
      DBInstanceClass: db.t2.micro
```


CLOUDFORMATION DEPLOYING LAMBDA FUNCTIONS
- We can create inline lambda functions in cloudformation.
- Code is defined inline, lambda inline
- Restrictions would be - 
	- No dependancies are allowed
	- Limited to 4000 characters
- You can use S3 versioning to update code in the lambda function.


CLOUDFORMATION CUSTOM RESOURCES (LAMBDA)
- These are lambda scripts that bridge the creation of fringe resources.
- These are used to address any of the following use-cases:
	- New resource not yet covered by cloudformation
	- Used for on-premise resources
	- Emptying an S3 bucket before deletion
	- Fetch AMI id (Old school)
- These can be hooked to create, update and delete.
- The resource can do whatever you want and are really powerful.
```
AWSTemplateFormatVersion: '2010-09-09'


Resources:
  myBucketResource:
    Type: AWS::S3::Bucket


  LambdaUsedToCleanUp:
    Type: Custom::cleanupbucket
    Properties:
      ServiceToken: !ImportValue EmptyS3BucketLambda. #CUSTOM RESOURCE HERE
      BucketName: !Ref myBucketResource
```


CLOUDFORMATION DRIFT DETECTION
- The detection of the changes between the template and the environment
- This may be because of manual intervention
- The are no reverts, it just shows the changes that have happened to stack.

CLOUDFORMATION STATUS CODES - DEEP DIVE
- These are all the statuses that a stack goes through during executions

CLOUDFORMATION INSUFFICIENT CAPABILITIES EXCEPTION
- Requires confirmation when creating a stack.

CLOUDFORMATION CFN-HUP
- This ensures that we can do configuration updates without taking down resources
- Metadata is updated and listened to by the cfn-hup (similar to nohup) at certain intervals.

CLOUDFORMATION STACK POLICIES
- Looks like an IAM policy
- Restricts actions on certain resources 

Demo Stack Policy Document
```
{
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "Update:*",
            "Principal": "*",
            "Resource": "*"
        },
        {
            "Effect": "Deny",
            "Action": "Update:*",
            "Principal": "*",
            "Resource": "LogicalResourceId/CriticalSecurityGroup"
        },
        {
            "Effect" : "Deny",
            "Action" : "Update:*",
            "Principal": "*",
            "Resource" : "*",
            "Condition" : {
              "StringEquals" : {
                "ResourceType" : ["AWS::RDS::DBInstance"]
              }
            }
        }
    ]
}
```



ELASTIC BEANSTALK
- This is developer centric application deployment.
- Install awsebcli first
- Begin a project by running eb init
- To create an environment on the cloud run eb create <env-name>
- To open environment run eb open an a browser will pop up
- To view status or health we can use eb health and eb status

EB - SAVED CONFIGURATIONS
- There is the concept of saved configurations 
- We can also use cloud formation to backup EB configurations
- To backup the current configuration we use
```
eb config save dev-env --cfg  initial-configuration
```
- It backs up all none default adjustments.
- We can also update environmental properties
```
eb setenv ENABLE_COOL_NEW_FEATURE=true
```
- To update a configuration we use the following 
```
eb config put production-configuration
```
- This pushes updates to the environment
- We would subsequently update the environment through
```
eb config dev-env --cfg production-configuration
```

EBEXTENSIONS FOR CONFIGS
- These are extensions run to further help configurations
- Saved configurations have precedence over ebextensions
- You can also add any cloudfomation resources in the extensions file.
```
Resources:
  DynamoDBTable:
    Type: AWS::DynamoDB::Table
    Properties:
      KeySchema:
         HashKeyElement:
           AttributeName: id
           AttributeType: S
      # create a table with the least available rd and wr throughput
      ProvisionedThroughput:
         ReadCapacityUnits: 1
         WriteCapacityUnits: 1


  NotificationTopic:
    Type: AWS::SNS::Topic
```
- Resources that are declared .ebextensions will be deleted when the stack is deleted.
- This means we need to created persistence resources outside of the EB resources. (Manually create and link in ENV.)

EBEXTENSIONS FOR COMMANDS AND CONTAINER COMMANDS
- Commands run before application and web server are set up and application version file is extracted.
- Container commands are used to modify the application source code
- They are run after the web server and application are set up but before the application version is extracted.

```
commands:
  create_hello_world_file:
    command: touch hello-world.txt
    cwd: /home/ec2-user

application version archive has been extracted, but before the application version is deployed.
container_commands:
  modify_index_html:
    command: 'echo " - modified content" >> index.html'

  database_migration:
    command: 'echo "do a database migration"'
    leader_only: true #runs only on a single instance
```


GOOD FEATURES TO KNOW
- Environments can be cloned
- There are lifecycle policies for application versions
- You can rebuild an environment (Take note of stateful applications and DBs)
- Platform updates can be configured

ELASTIC BEANSTALK DEPLOYMENT MODES
- Single Instance Deployment
	- Great for development
- High availability with LB 
	- Great for production
	- Has ASGs possibly in multi-AZ
	- 

All At Once
- Fastest but instances will slightly be unavailable to serve traffic
- Application has downtime
- Great for quick iterations in development environment
- No additional costs
- 
Rolling 
- Update a few instances at a time
- Bucket size is the capacity permitted during update e.g 50%
- May be long deployment if many instances are available
- Application will be running below capacity
- No additional cost
- 
Rolling with additional batches
- Spins up new instances for the new batch and terminates old
- Good for prod
- Application runs at capacity
- Longer deployment.
- Small additonal cost
Immutable
- New instances in new ASG are prepared and swapped out after all updates are completed on the new ASG
- Zero downtime
- Double capacity, high cost
- Quick rollback in case of failures
- Longest deployment
Blue / Green
- 2 environments are used at the same time
- Route 53 can be used to redirect a fraction of the traffic to the new version
- Weighted policy used.
- Allows for more testing
- Swap URLs can babe used when satisfied with the staging.

We can swap URLs for environments if needed.


WORKER ENVIRONMENTS
- For custom jobs that don’t need web servers
- Processes long running workloads
- Creates queues for the workloads. 2 worker queue and dead letter queue
- We use cron.yaml for cron jobs

ELASTIC BEANSTALK MULTI-DOCKER
- We use a file named Dockerrun.aws.json to set up complex docker for elastic beanstalk


——————————————————————————————   

AWS LAMBDA
- Provides customization required by other services to work
- A lambda function needs to reside in a VPC
- Needs a security group.

SOURCES AND USE CASES
- API-Gateway - For serverless APIs
- Application load balancer - Does not give additional functionality like security, just creates an interface to interact with the lambda funtion
- Cloudwatch events - React to any event in the the cloud.
- Cloudwatch logs - Through subscriptions - note of patterns
- Code-commit
- DynamoDB - For streams and reaction to streams
- Kinesis - Streams can feed into lambda
- S3 Events
- SNS - Asysncronous invokation 
- SQS - process queues

SECURITY AND ENVIRONMENTAL VARIABLES
- 

AWS LAMBDA VERSIONS, CANARY ROUTING
- When you work on lambda you work on version $LATEST
- Versions get incremental numbers
- Each version has a unique ARN
- Every version is Immutable
- Each version can be accessed at any point.
- We use aliases which are used as pointers to versions
- Dev/prod/staging can point to any version
- Aliases are mutable
- We can use aliases to do blue/green deployments


AWS SAM (SERVERLESS APPLICATION MODEL)
- Makes it easy to create and manage serverless applications
- Sam when testing locally uses docker
```
Sam init
Sam build
sam local invoke "HelloWorldFunction" -e events/event.json
Sam start local-api
```

- We can subsequently package and deploy our application easily
- This is achieved through a package.yaml file which is a valid cloudformation file.
```
sam package --output-template packaged.yaml --s3-bucket aws-devops-course-stephane --region eu-west-1 --profile aws-devops

sam deploy --template-file packaged.yaml --capabilities CAPABILITY_IAM --stack-name aws-sam-getting-started --region eu-west-1 --profile aws-devops
```

AWS STEP FUNCTIONS
- These are used when you have a complex workflow
- Makes it intuitive
- Can coordinate between. A lot of different services.
- Can also be used to sequence batch jobs
- Handle workflows.
- Can also coordinate manual approvals on workflows
- They can run for up to a year.
- We can retrace all event history in the step functions and state machines.


AWS API-GATEWAY
- We can create a rest API or a websocket
- We can import an API from swagger or OpenApi 3
- You can link to several services
	- Lambda function
	- Http
	- Mock 
	- AWS Service
	- VPC Link
- A good feature of API-Gateway is the use of mapping templates to change the format of a request before being forwarded to the destination.
- This does transformations for the requests and responses.
- Changes are required on the API gateway also for the application changes to be live
- Changes are deployed to stages

API-GATEWAY WITH LAMBDA
- You can point an api endpoint to a an alias in lambda
- You can also point to a lambda version
- When defining a lambda function for api gateway you can also point to a specific version


A good feature of API-Gateway is the use of mapping templates to change the format of a request before being forwarded to the destination.
- You can use a language called VTC to modify content before it goes to lambda funtion
- It can also modify response.

API-GATEWAY DEEPLOYMENT STAGES
- Making changes in Api-gateway does not mean they are emmediatel effective
- You need to make a deployment for them
- Changes are deployed to stages
- Each stage can get its own configuration parameters
- Stages can be rolled back
- Stages can co-exist
- Stage variables are like ENV variables
- They can be used in
	- Lambda function arn
	- http endpoint
	- Parameter mapping templates

Stage variables are like ENV variables for the API gateway
- We use them to change often changing configuration values.
	- Http Endpoint
	- Lambda function ARN
	- Parameter mapping templates
- They are passed to the context object in AWS lambda

API GATEWAY - CANARY DEPLOYMENT
- Possible to enable canary deployments for any stage
- Choose % of traffic canary channel receives
- 

API GATEWAY THROTTLING
- Default is 10000/sec across all resources in the account
- Usage plans can be associated by API key
- Types of throttles are
	- API gateway Throttle
	- Lambda function throttle (default 1000)
	- Usage Plan throttle (Burstable)
- We can use API-Gateway to trigger (expose) step functions.

DOCKER VS VIRTUAL MACHINES


ECS CLUSTERS
- They are a logical grouping of EC2 instances
- They run an ECS agent
- They run a special ECS AMi
- The agents register the instance to the ECS cluster.
- ECS cluster comes with an ASG.
```
ECS_CLUSTER=<cluster-name>

// This should be appended to /etc/ecs/ecs.config
```

ECS TASK DEFINATIONS
- These are metadata in json form to tell ECS how to run a docker container.
- Contains crucial information on:
	- Image Name
	- Port bindings
	- Memory and CPU required
	- Environmental variables
	- Networking information
ECS SERVICE
- How many tasks to be run and how they should be run.

OPSWORKS
- It has 3 services in it
	- OpsWorks stacks
	- OpsWorks for chef automate
	- OpsWorks for Puppet Interprise
- Stacks is about using chef cookbooks for deployments

```
A stack is a set of layers, interfaces and related aws resources whose configuration you want to manage together.
```

STACKS LIFECYCLE EVENTS
- Setup - Fires after instance has finished booting (Can be manually triggered)
- Configure - 
	- Instance enters or leaves the online state
	- Associate/ Disassociate instance with Elastic IP
	- Attach an ELB to a layer, or detach one from a layer
- Deploy, when you run a deploy command, Setup includes deploy
- Undeploy
- Shutdown - for clean up tasks

  CLOUDTRAIL
- Gives the ability to track any api calls made within the account.
- Ensures that the logs are delivered to S3 or cloudwatch logs.
- Can apply to all regions if chosen
- Can filter by event type
- Logs are encrypted by default
- What we get from cloudtrail:
	- Who made the request
	- When and from where
	- What was requested
	- What what was the response
- It has a 15 minute delay, the events are not realtime, for this you would need cloudwatch events
- We can use event history to track what happened in a resource.

CLOUDTRAIL FILE INTEGRITY
- We can check for file modifications through the CLI
- Cloud-trail digest updates every hour.
- We use a command to check the integrity of files.
```


aws cloudtrail validate-logs --start-time 2015-08-27T00:00:00Z 
    --trail-arn arn:aws:cloudtrail:eu-west-1:903077646177:trail/DemoTrail 
    --verbose --profile aws-devops --region eu-west-1

```

CLOUDTRAIL CROSS-ACCOUNT 
- By allowing a specific account to write to a log bucket.
- We use bucket policies to allow other accounts.


KINESIS
- Managed alternative to apache Kafka
- Has real time streaming capability
- 
- Great for
	- Application logs
	- IoT 
	- metrics
	- Click streams
- Great for real-time big-data
- Data is automatically replicated to 3 AZ


- KINESIS STREAMS
	- Divided into shards
	- Producers and consumers
	- Data retention of 1day but can go up to week
	- Ability to reprocess and replay data
	- Multiple applications can consume at the same time.
	- Data inserted cannot be deleted. (Manually)
	- Billing per shard / shards can be scaled
	- Players are :
		- Data Blob - Data being sent, serialized as bytes. Up to 1MB. Can represent anything
		- RecordKey - Set alongside a record, helps to group records in shards. Same key = Same Shard - Use highly distributed key to avoid hot partition issue.
		- Sequence Number - Unique identifier for each records put in shards. Added by kinesis after ingestion.
	- Some of the limitations are as follows:
		- Producers can send 1mb/s or 1000messages/s at write per shard
		- To much activity on 1 shard could cause “ProvisionedThroughPutException"
		- Consumers can 2MB/s per shard across all consumers
		- 5 API calls per shard across all consumers 
- KINESIS ANALYTICS
	- Realtime analytics on data
	- Use the SQL language
	- Can create streams out of realitime analytics.
	- Auto scaling
	- Managed
	- Pay for what goes into this.
- KINESIS FIREHOSE
	- Near real time. 60sec latency minimum for non full batches
	- This is a fully managed service, Automated scaling.
	- Load data into REDSHIFT, S3, ELASTIC SEARCH, SPLUNK
	- Automatic scaling
	- Data transformation through lambda CSV => json
	- Suports compression
	- Pay for amount of data going through firehose  


Kinesis Producers
1. Kinesis SDK
2. Kinesis Producer Library
3. Kinesis Agent
4. Cloudwatch Logs
5. 3rd party libraries

Kinesis Consumers
1. Kinesis SDK
2. Kinesis Client Library
	1. Uses DynamoDB to checkpoint offsets
	2. Uses Dynamo to track other workers and share the work amongst shards
3. Kinesis Connector Library
4. Kinesis Firehose
5. AWS lambda
6. 3rd party libraries

You can not have more KCL applications than shards in your stream.


CLOUDWATCH METRICS
- All services that track some metrics can be found here. (ApiGateway, EC2, etc)
- Detailed monitoring enables metrics every minute. (This is billed)
- Basic monitoring enables metrics every 5 minutes.
- Metrics are only available for up to 15Months
- Some important metrics to know:
	- We don’t have RAM information
	- We can get disk IO
	- You can get aggregate metrics on a collection of instances through ASG metrics. 

CLOUDWATCH CUSTOM METRICS
-  You can chose 1 of 2 options
	- Standard Resolution: with data having 1 minute grantlarity
	- High Resolution: data at a 1 second granularity.
```
aws cloudwatch put-metric-data --metric-name FunnyMetric 
    --namespace Custom --value 1243 --dimensions InstanceId=1-23456789,InstanceType=m1.small 
    --profile aws-devops --region eu-west-1
```
- You can use dimensions and can have up to 10 dimensions in 1 metric.
- You can publish statistics sets (sum, average, mean, etc)

CLOUDWATCH METRICS EXPORT
- You can use the api call get-metric-statistics.
- You get a json back.
```
aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization 
    --dimensions Name=InstanceId,Value=i-06e91bf508fd3bf9f --statistics Maximum --start-time 2019-09-10T00:00:00 
    --end-time 2019-09-11T00:00:00 --period 360 --profile aws-devops --region eu-west-1
```

CLOUDWATCH ALARMS
- Trigger actions based on metrics.
- You cannot combine multiple metrics into 1 alarm
- Alarms can only do 3 things:
	- Send to an SNS topic
	- Trigger an auto-scaling action
	- Trigger an EC2 action.
- You can set an alarm for billing.

UNIFIED CLOUWATCH AGENT
- Used to send both metrics and logs from instances.
- After the setup, a json file is created with the configuration of how the agent should boot up.
- We can also use an SSM parameter to store the configs

CLOUDWATCH METRIC FILTERS & ALARMS
- Filters can be added at the log-group level.

CLOUDWATCH LOGS - LOG SUBSCRIPTION
- We cannot send data to streams or firehose from the console.
- We can use lambda.
- We can create a subscription filter.
- 

CLOUDWATCH EVENTS - INTERGRATION WITH CLOUDTRAIL
- We cannot attach to list get or describe events.
- An example of an event is createImage on EC2
- Target can be anything

EVENT NOTIFICATIONS FROM S3 BUCKETS
- We can react to specific objects being interacted with in the buckets.
- Can send to lambda SNS SQS
- S3 events don’t give bucket level operations. They give object level operations.
- Events need to be tracked in cloud trail also for bucket level events.

CLOUDWATCH DASHBOARDS
- These will be used to display metric results
- They are also multiple-region
- You can correlate data through dashboards

XRAY
- Colleccts data about requests that your application serves
- Helps in a distributed system.
- Creates a service mesh for your application architecture.
- X-ray can detect errors and report when occurred.
- Helps with 3 items
	- Debugging
	- Tracing 
	- Service Map

AMAZON ELASTIC SEARCH
- Has no serverless offering
- Use cases:
	- Log Analytics
	- Real time application monitoring
	- Security Analytics
	- Full text search
	- Clickstream analytics
	- Indexing
- Dashboarding functionality offered
- Kibana provides realtime dashboard capability
- Alternative to cloudwatch dashboards
- Logstash is a log ingestion alternative
	- More retention and more choice on granularity


With Cloudwatch logs

AWS TAGGING STRATEGIES
- These can be used for cost management
- For code deploy to deploy to specific instances
- We can use tag based access control


POLICIES AND STANDARD AUTOMATION
SYSTEMS MANAGER

- Manage EC2 and on-premise systems at scale
- Get operational insights about the state of your infrastructure
- Easily detect problems
- Patching automation for compliance
- Works for both windows and lunux
- Integrated with cloudwatch metrics and dashbpards
- Integrated with aws Cofig
- Free service

Features include
1. Resource groups
2. Insights
	1. dashboard
	2. Inventory: discover and audit the software installed
	3. Compliance
3. Parameter Store
4. Action
	1. Automation (Instance shutdown e.t.c)
	2. Run Command
	3. Session manager
	4. Patch manager
	5. Maintenance window
	6. State manager

We need to install the SSM agent on systems we want to control.
A hybrid instance is an onprem instance that needs ssm agent manually installed. It can also be tagged. Alway starts with mi-######
This is installed by default on Amazon Linux AMIs and other ubuntu AMIs
Instances need to have the appropriate IAM role

RESOURCE GROUPS
- A way to group resources together
- There are 2 types
	- Tag based
	- Cloudformation stack based

SSM RUN COMMAND
- Allows you to run commands on instances managed by ssm
- There are 4 types:
	- Commands
	- Policy
	- Session
	- Automation
- Seems to run ansible playbooks also
- There is also a nice command history to trace 

SSM PARAMETER STORE
- Parameters can be 1 of 3 items
	- String 
		- None sensitive values
	- StringList
	- Secure String
		- Secrets

- SSM can be used for OS patching also
- You can subsequently track the compliance of patches and OS.
- The parameters can be versioned and have full audibility

INVENTORY
- Give information on what’s running on our instances.
- These are packages that are installed
- You can see inventory per instance or for whole account.
- 


AUTOMATIONS
- Simplifies common maintenance and and deployment of tasks 
- This is the opposite of run commands.
- Tight integration with cloudwatch events
- Rescue unreachable instances



AWS CONFIG
- Audit trail and compliance
- Tight relationship between config and cloudtrail
- Configuration can be exported as json
- There is also the compliance timeline concept
- You can get the configuration timeline and relationship. Cloudwatch events and cloud trail.
- All data can be stored on S3.
- We also have a compliance timeline.

AWS CONFIG RULES
- Allows for us to define rules and maintain compliance
- We get an audit trail.
- Each rule costs $1 a month
- Rules can have remediation options.
- We can use cloudwatch events to respond to non-compliant rules and update them.
- Remediation can also be set at the config dashboards and are set through SSM automations.
- We can create custom rules using lambda.
- We can 2 trigger types, periodic and configuration change

AWS CONFIG - MULTI ACCOUNT
- We can aggregate config  data to one account
- Set up under aggregator


SERVICE CATALOG
- Allow to create and manage catalogs of IT services that are approved on AWS
- The products are cloudformation templates.
- Cloudformation is used to ensure consistency and standardization by admins.
- They are assigned to portfolios (teams)
- Teams are presented a self service portal where they can launch  the products.
- All products centrally managed.
- Very important for governance, compliance and consistency
- Users can now launch products without deep AWS knowledge.
- Integrations with self-service portals such as SERVICE NOW
	- Create product from cloudformation template
	- Create portfolio and add product to portfolio
	- Set access to the portfolio for users groups and roles
	- 

A portfolio can be assigned to a user group.


INSPECTOR
- Important when creating AMIs and doing security inspection/analysis.
- You need to install the inspector agent on EC2 instances.
- You can run an assessment for the targets.
- You can analyze the findings.
- There are 2 setups
	- Network assessments - No agent required - if agent is installed it can go further and identify processes reachable on port
	- Host Assessments - Agent needs to be installed
- It can be a target for cloudwatch events to run assessments.
- Creates a scheduled rule for the assement
- For now we can send events into an dns topic
- Inspector does not launch an EC2 instance for you.
- It can be a target of cloudwatch events. We can use a schedule rule

EC2 INSTANCE COMPLIANCE
- AWS CONFIG - 
	- Ensure instance has proper AWS configuration (not open SSH to the world)
	- Audit over time (timeline support)
	- Automation can be implemented using cloudwatch rules.
- INSPECTOR - 
	- Security vulnerability scan from within the OS using the inspector agent.
	- Or outside network scanning.
	- Does not launch instance for you
- SYSTEMS MANAGER - 
	- Allows to run commands and automations, patches , inventory at scale
- SERVICE CATALOGS -
	- When we have beginners and we have strict requirements
	- Minimal configurations
- CONFIGURATION MANAGEMENT TOOLS -
	- SSM, OPSWORKS, Ansible, CHEF, Puppet, USER-DATA
	- Ensure instances have proper configuration files.



SERVICE HEALTH DASHBOARDS
- We can get service information and service history
- We can get an RSS feed into these 
- The SERVICE HEALTH DASHBOARD is a global dashboard that can be used for that.
- The PERSONAL HEALTH DASHBOARD is tailored down to your account.
- These can be attached to cloudwatch events. You can create a rule straight from the dashboard.



TRUSTED ADVISOR
- Automated limits notifications.
- You can refresh every 5 minutes
- It give recommendations based on 5 dimensions:
	- Cost optimization 
	- Perfomance 
	- Security
	- Fault Tolerance
	- Service Limits

TRUSTED ADVISOR AUTOMATION
- You can use a lambda function to pass a notification to slack.
- You can also push data to kinesis data streams for comprehensive data.
- This can be used with cloudwatch events rules.
- Can also be used to delete exposed keys and send notifications to the SNS topic.

TRUSTED ADVISOR AUTOMATION REFRESHES
- You can refresh every 5 minutes manually.
- You can use CLI to refresh refresh-trusted-advisor-check
- You can describe the results of a refresh by using the internal API call describe-trusted-advisor-check-result

GUARD-DUTY
- This is an intelligent threat discovery service.
- Protects the AWS account
- Uses some machine-learning, anormally detection and 3rd party data.
- One click to enable 30 day trial, no need to install software.
- Input data include
	- Cloudtrail logs
	- VPC flowlogs
	- DNS logs
- Notifies you in case of findings
- Integration with AWS Lambda

GUARD-DUTY AUTOMATIONS
- This can be achieved through cloudwatch event rules.
- Events can be handled through lambda


AMAZON MARCIE
- Data visibility security service that helps clarify and protect your sensitive and business critical content.
- You can add many accounts.
- Analyses data in S3 to filter out sensitive data
- For PII data also

SECRET MANAGER
- It differs from parameter store because you can set up rotation and lifecycle events on secrets.
- Has a tight integration with database solutions as RDS, AURORA etc

AWS LICENSE MANAGER
- Manages licences in the account
- Manages licenses from 3rd party vendors such as
	- Microsoft
	- oracle
	- SAP
- You can a associate a license with an AMI
- You can also attach to existing instances.


COST ALLOCATION TAGS

- Tags tracks resources that are related to each other
- With cost allocation tags we can enable detailed costing reports
- They show up as columns in reports.
- AWS cost allocation tags
	- Automatically applied to resources created
	- Start with prefix aws (aws:created_by)
	- Not applied to resources created before the activation
- User Tags
	- Defined by the user
	- Starts with preffix user
- COST ALLOCATIONS TAGS just appear in the billing console
- Takes up to 24 hours for tags to show up in the report

AWS DATA PROTECTION
- TLS for in-transit encryption
- ACM to manage certificates
- Load Balancers
	- ELB, NLB, ALB provide SSL termination
	- Possible to have multiple SSL certs per ALB using Server Name Indication (SNI)
- Cloudfront with SSL
- All AWS resources expose HTTPS endpoints
- At rest , S3 encryption
	- SSE-S3 - aws key
	- SSE-KMS - own kms key
	- SSE-C - own key that aws won’t keep
	- Client side encryption - send encrypted content to aws
- Default encryption on bucket can be enabled
- Glacier encrypted by default
- PHI = PROTECTED HEALTH DATA
- PII = PERSONALLY-IDENTIFYING INFORMATION

INCIDENT AND EVENT RESPONSE
AUTO SCALING GROUPS
- Advanced way of defining instances in different ways
- Templates are going to be versioned and easily managed.
- From launch template you can create
	- Instances
	- ASG
	- Spot Fleet
- You can mix instance types using launch templates
- Templates can be versioned.
- Launch templates can set instance types. (Optional)
- We can do multiple things with launch templates:
	- Launch an instance
	- Create an ASG
	- Create a spot fleet.

Benefits and features 

- Detect when an instance is unhealthy, terminate it, and launch an instance to replace it.
- Save money by dynamically launching instances when they are needed and terminating them when they aren't.
- Ensure that your application always has the right amount of capacity to handle the current traffic demand.



ASG SCHEDULED EVENTS
- When we know traffic patterns in advance
- We can scale in or out based on requirement

ASG SCALING POLICIES
- Default Cooldown - default is 300s
	- This is the number of seconds after a scaling event completes before another can begin
	- This is also called cool down period
- They work with cloudwatch alarms to implement events
- Warm up needs to be lower than cooldown
- You can disable scale-In
- You can create alarms in advance through cloudwatch alarms and attach them to a simple scaling policy.
- You can use steps to auto-scale. 40% => 2 instances, 60% => 4 instances
- We have about 4 metrics to use
	- cpuUtilization
	- networkIn
	- networkOut
	- ALB - request per count

SIMPLE SCALING POLICY
- You need to have a preconfigured alarm for this policy to work
- Give more flexibility and can work with any alarm

STEP POLICY
- Yo define as many steps as you want
	- 60% => add 1 isntance
	- 70% => add 2 instances

ASG - ALB INTEGRATION
- In target groups you can define a slow start mode for newly created instances.
- Canary deployment of instances
- This is set on target group attributes.

ENABLE HTTPS ON ALB
- We can force HTTPS

ASG - SUSPENDING PROCESSES AND TROUBLESHOOTING
- Launch
- Terminate
- Health Check
- Replace Unhealthy
- AZ-REBALANCE  - Make instances balanced across AZ
- Alarm Notification
- Scheduled Actions
- Add to Loadbalancer
We can set scale in protection on instances.


ASG LIFECYCLE HOOKS
- We have one on PENDING
	- Pending wait
	- Pending proceed
- We have another on TERMINATE
	- Terminate wait
	- Terminate proceed
- We can invoke 3 items to cater to the above
	- SNS
	- SQS
	- Lambda through cloudwatch events
- We can use these on a few very important areas
	- Install or configure software on newly launched instances
	- Download logs from instances before they terminate.


TERMINATION POLICIES
- Default
	- Which AZ has the most instances and find one not protected from scale in
	- Evaluate allocation strategy
	- Look for instance with the oldest template
	- If multiple it chooses at random
- Termination policy can be customized - e.g you want to upgrade instances too new instance type.
	- Oldest Instances
	- Newest Instances
	- Oldest Launch Configuration
	- Closest to next Instance Hour (Billing Hour)
	- Oldest Lauch Template
	- Allocation strategy

ASG INTEGRATION WITH SQS
- Scaling based on number of messages in a queue
- We can protect a scale in from happening if its a worker.

ASG MONITORING
- We can get ASG metrics
- We also get EC2 metrics
- We can get notifications also

ASG - CLOUDFORMATION CREATION POLICY
- This is the creation of an ASG
```
Parameters:
  LatestLinuxAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'


Resources: 
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      AvailabilityZones: 
        Fn::GetAZs: 
          Ref: "AWS::Region"
      LaunchConfigurationName:
        Ref: LaunchConfig
      DesiredCapacity: '3'
      MinSize: '1'
      MaxSize: '4'
    CreationPolicy:
      ResourceSignal:
        Count: '3'
        Timeout: PT15M


  LaunchConfig:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
      ImageId: !Ref LatestLinuxAmiId
      InstanceType: t3.micro
      UserData:
        "Fn::Base64":
          !Sub |
            #!/bin/bash -xe
            yum update -y aws-cfn-bootstrap
            /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource AutoScalingGroup --region ${AWS::Region}
```

ASG - CLOUDFORMATION UPDATE POLICY
- Cloudformation can update the ASG but the instances will remain as is.
- In order to update the underlying instances we need to use an update policy attribute.
- We can have 3 options here
	- AutoScallingReplacingUpdate
	- AutoScallingRollingUpdate
	- AutoScalingScheduledAction

ROLLING UPDATE
```
UpdatePolicy:
  AutoScalingRollingUpdate:
    MinInstancesInService: '1'
    MaxBatchSize: '2'
    # how much time to wait for the signal
    PauseTime: PT1M
    WaitOnResourceSignals: 'true'
    # we can suspend processes during the update
    # SuspendProcesses:
    # - list of processes...
  AutoScalingScheduledAction:
    # Prevent Scheduled Actions from modifying min/max/desired for CloudFormation
    IgnoreUnmodifiedGroupSizeProperties: 'true'
```

REPLACE UPDATE
```
UpdatePolicy:
  AutoScalingReplacingUpdate:
    WillReplace: 'true'
```


ASG - CODE-DEPLOY INTEGRATION
- Whenever a new instance is launched in an ASG, code deploy runs and deploys the applications that should be on those instances
- You can also do blue/green deployments. A new ASG is created and gradually replaces the old one.

ASG - CODEDEPLOY INTEGRATION TROUBLESHOOTING
- To avoid scaling events while deploying we can use process suspension until deployment finishes then resume scaling
- This avoids having multiple application versions at the same time.

ASG DEPLOYMENT STRATEGIES
- In-place
	- 1 tg, 1 lb, 1 asg
- Rolling
	- 1lb, 1tg, 1 asg, new instances
	- Both versions available at same time
- Replace
	- 1lb, 1tg, 2 asg, new instances
- Blue/Green
	- 2lb, 2tg, 2asg, new instances, R53

DYNAMODB
- Need a primary key and possibly partition key
- Yo can provision read write capacity
- A dynamoDB stream is a Kinesis stream underlying.


MULTI-REGION SERVICES
- DynamoDB global tables (Enabled by Streams)
- AWS config aggregators
- RDS cross region Read Replicas
- Aurora Global database
- EBS snapshots, AMI, RDS snapshots can be copied to other regions
- VPC peering to allow private traffic between regions
- Route53 uses global network of DNS servers
- S3 cross region replication
- Cloudfron global CDN at edge locations
- Lambda@edge for global lambda
- 


CLOUDFORMATION STACKSETS
- Create update or delete stacks across multiple accounts and regions with a single operation
- Administrator account can create stacksets
- Trusted accounts can create stack instances
- When you update a stack set, all associated stack instances are updated.
- Ability to set a maximum amount of concurrent actions on a target
- Ability to set a failure tolerance

DISASTER RECOVERY
- Any event that has a negative impact on a  company’s business continuity or finances
- DR is about preparing for and recovering from disaster.
	- On premise to On premise
	- On premise to cloud
	- Cloud region A to Cloud region B
- RPO => Recovery Point Objective
	- How often you run backups
	- Determines how much data loss you encounter
- RTO => Recovery Time Objective
	- When will you recover
	- Amount of downtime



- Backup and Restore (High RPO)
	- Not too expensive
- Pilot Light
	- Small version of app is always running in the cloud
	- Useful for the critical core
	- Very similar to backup and restore
	- Faster than backup and restore as critical systems are already up.
- Warm Standby
	- Full system is up and running at minimum size
	- Upon disaster, we can scale to production load.


- Multi-Site / Hot-site
	- Very low RTO -  Very expensive
	- Full production Scale is running on AWS and on-Premise

DISASTER RECOVERY TIPS
- BACKUP
	- EBS, RDS, Snapshots e.t.c
	- Regular pushes to S3
	- Snowball or storage gateway for on-prem
- HIGH AVAILABILITY
	- Route53 to migrate from one region to onother
	- RDS multi-AZ, Elasticache multi-AZ, EFS
	- Site to site VPN as recovery from direct connect
- REPLICATION
	- RDS, AURORA, DYNAMODB global database
	- Database replication from on-prem to RDS
	- Storage Gateway
- Automation
	- Cloudformation / Elastic Beanstalk to recreate environments
	- Cloudwatch Alarms actions to rectify failures
	- AWS lambda functions for custom actions for automations
- CHAOS
	- Test resilience on production workloads


MULTI-REGION DISASTER RECOVERY CHECKLIST
- Is AMI copied and stored in parameter store
- Is cloudformation stack set working in other regions
- What is my RPO and RTO
- Are route53 heath checks working correctly and tied to CW alarm
- Read replication promotion through lambda and CW
- Is my data backed up and where is it.

YOU can download aws linux2 AMI to use on premise
Yon can import/export VM from premium to cloud vise-versa
DATABASE MIGRATION SERVICE - Replicate data to cloud
APPLICATION DISCOVERY SERVICE -Gather information about on-prem servers before migrations
- Server utilization and dependency mapping
- Track with AWS migration hub
AWS SERVER MIGARTION SERVICE - SMS
- Incremental replication of on premise live servers to AWS
- Ongoing migration

AWS MULTI-ACCOUNT 
- This can be a strategy for 
	- Create account per department
	- Per cost center
	- Per Environment
	- Based on regulatory restrictions
	- For better resource isolation
	- To have separate per account service limits
	- Isolated account for logging
- Use tagging standards for billing purposes
- Enable cloud trail on all accounts
- Send cloudwatch logs to central logging system
- We us OU - ORGANIZATIONAL UNITS
- 


Some OUs could look like this 



SERVICE CONROL POLICIES
- Whitelist or blacklist IAM actions
- Apply at the Acc / OU level
- Does not apply to master account
- Applied to all users and roles
- It does not affect service linked roles
- They must have explicit allow to allow actions
- DENY on the parent takes precedence to an ALLOW on a child



MIGRATE ACCOUNTS
- Remove member account from old organisation
- Send invite to new organisation
- Accept the invite to the new organisation from the member account
