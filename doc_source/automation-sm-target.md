# Running Automation Workflows with Triggers using State Manager<a name="automation-sm-target"></a>

You can start an Automation workflow by creating a State Manager Association with an Automation document\. By creating a State Manager Association with an Automation document, you can target different types of AWS resources\. For example, you can create associations that enforce a desired state on an AWS resource, including the following:
+ Attach a Systems Manager role to Amazon EC2 instances to make them *managed instances*\.
+ Enforce desired ingress and egress rules for a security group\.
+ Create or delete Amazon DynamoDB \(DynamoDB\) backups\.
+ Create or delete Amazon Elastic Block Store \(Amazon EBS\) snapshots\.
+ Disable read and write permissions on Amazon Simple Storage Service \(Amazon S3\) buckets\.
+ Start, restart, or stop managed instances and Amazon Relational Database Service \(Amazon RDS\) instances\.
+ Patch Windows and Linux AMIs\.

**Before You Begin**  
Be aware of the following important details before you run Automation workflows by using State Manager\.
+ Before you can create an association that runs an Automation document, verify that you configured permissions for Systems Manager Automation\. For more information, see [Getting Started with Automation](automation-setup.md)\.
+ State Manager associations that run Automation documents contribute to the maximum number of concurrently running Automations in your AWS account\. You can have a maximum of 25 concurrent Automations running at one time\.
+ Systems Manager automatically creates a service\-linked role so that State Manager has permission to call Systems Manager Automation API actions\. If you want, you can create the service\-linked role yourself by running the following command from the AWS CLI\.

  ```
  aws iam create-service-linked-role --aws-service-name ssm.amazonaws.com
  ```

  For more information about service\-linked roles, see [Service\-Linked Role Permissions for Systems Manager](using-service-linked-roles.md#slr-permissions)\.

## Creating an Association that Runs an Automation Workflow \(Console\)<a name="automation-sm-target-console"></a>

The following procedure describes how to use the Systems Manager console to create a State Manager association that runs an Automation workflow\.

**To create a State Manager association that runs a Systems Manager Automation Workflow \(Console\)**

1. Open the AWS Systems Manager console at [https://console\.aws\.amazon\.com/systems\-manager/](https://console.aws.amazon.com/systems-manager/)\.

1. In the navigation pane, choose **State Manager**, and then choose **Create association**\.

1. In the **Name** field, specify a name\. This is optional, but recommended\.

1. In the **Document** list, choose a document\. Use the Search bar to filter on **Document type : Equal : Automation** documents\. To view more Automation documents, use the numbers to the right of the Search bar\. 
**Note**  
You can view information about a document by choosing the document name\.

1. Choose **Simple execution** to run the automation on one or more targets by specifying the resource ID for those targets\. Choose **Rate control** to run the automation across a fleet of AWS resources by specifying a targeting option such as tags or AWS Resource Groups\. You can also control the execution of the automation across your resources by specifying concurrency and error thresholds\.

   If you chose **Rate control**, the **Targets** section appears\.

1. In the **Targets** section, choose a method for targeting resources\.

   1. \(Required\) In the **Parameter** list, choose a parameter\. The items in the **Parameter** list are determined by the parameters in the Automation document that you selected at the start of this procedure\. By choosing a parameter, you define the type of resource on which the Automation workflow runs\. 

   1. \(Required\) In the **Targets** list, choose a method for targeting the resources\.
      + **Resource Group**: Choose the name of the group from the **Resource Group** list\. For more information about targeting AWS Resource Groups in Automation documents, see [Targeting AWS Resource Groups](automation-working-targets.md#automation-working-targets-resource-groups)\.
      + **Tags**: Enter the tag key and \(optionally\) the tag value in the fields provided\. Choose **Add**\. For more information about targeting tags in Automation documents, see [Targeting Tags](automation-working-targets.md#automation-working-targets-tags)\.
      + **Parameter Values**: Enter values in the **Input parameters** section\. If you specify multiple values, Systems Manager runs a child Automation workflow on each value specified\.

        For example, say that your Automation document includes an **InstanceID** parameter\. If you target the values of the **InstanceID** parameter when you run the Automation, then Systems Manager runs a child Automation for each instance ID value specified\. The parent Automation is complete when the Automation finishes running each specified instance, or if the execution fails\. You can target a maximum of 50 parameter values\. For more information about targeting parameter values in Automation documents, see [Targeting Parameter Values](automation-working-targets.md#automation-working-targets-parameter-values)\.

1. In the **Input parameters** section, specify the required input parameters\.

   If you chose to target resources by using tags or a resource group, then you may not need to choose some of the options in the **Input parameters** section\. For example, if you chose the AWS\-RestartEC2Instance document, and you chose to target instances by using tags, then you don't need to specify or choose instance IDs in the **Input parameters** section\. The Automation execution locates the instances to restart by using the tags you specified\. 
**Important**  
You must specify a role ARN in the **AutomationAssumeRole** field\. State Manager uses the assume role to call AWS services specified in the Automation document and run Automation associations on your behalf\. For more information, see [Running an Automation Workflow by Using an IAM Service Role](automation-walk-security-assume.md)\. 

1. In the **Specify schedule** section, choose **On Schedule** if you want to run the association at regular intervals\. If you choose this option, then use the options provided to create the schedule using Cron or Rate expressions\. For more information about Cron and Rate expressions for State Manager, see [Cron and Rate Expressions for Associations](reference-cron-and-rate-expressions.md#reference-cron-and-rate-expressions-association)\. 
**Note**  
Rate expressions are the preferred scheduling mechanism for State Manager associations that run Automation documents\. Rate expressions allow more flexibility for running associations in the event that you reach the maximum number of concurrently running Automations\. With a rate schedule, Systems Manager can retry the Automation shortly after receiving notification that concurrent Automations have reached their maximum and have been throttled\.

   Choose **No schedule** if you want to run the association one time\. 

1. \(Optional\) In the **Rate Control** section, choose **Concurrency** and **Error threshold** options to control the Automation execution across your AWS resources\.

   1. In the **Concurrency** section, choose an option: 
      + Choose **targets** to enter an absolute number of targets that can run the Automation workflow simultaneously\.
      + Choose **percentage** to enter a percentage of the target set that can run the Automation workflow simultaneously\.

   1. In the **Error threshold** section, choose an option:
      + Choose **errors** to enter an absolute number of errors allowed before Automation stops sending the workflow to other resources\.
      + Choose **percentage** to enter a percentage of errors allowed before Automation stops sending the workflow to other resources\.

   For more information about using targets and rate controls with Automation, see [Running Automation Workflows that Use Targets and Rate Controls](automation-working-targets-and-rate-controls.md)\.

1. Choose **Create Association**\. 
**Important**  
When you create an association, the association immediately runs against the specified targets\. The association then runs based on the cron or rate expression you chose\. If you chose **No schedule**, the association does not run again\.

## Creating an Association that Runs an Automation Workflow \(CLI\)<a name="automation-sm-target-cli"></a>

Use the following format to create an AWS CLI command that creates a State Manager association that runs an Automation workflow\.

**Note**  
If you create an association by using the AWS CLI, use the `--Targets` parameter to target instances for the association\. Don't use the `--InstanceID` parameter\. The `--InstanceID` parameter is a legacy parameter\. 

```
aws ssm create-association --association-name AssociationName --targets Key=tag:TagKey,Values=TagValue --name DocumentName --parameters (if any) --automation-target-parameter-name (parameter to target) --schedule "cron_or_rate_expression"
```

The following example creates an association to start Amazon EC2 instances tagged with `"Environment,Linux"`\. The association uses the `AWS-StartEC2Instance` document and the specified automation assume role to start the targeted instances at 7:00 every Monday morning\. This association runs simultaneously on 10 instances maximum at any given time\. Also, this association stops running on more instances for a particular execution interval if the error count exceeds 5\. For compliance reporting, this association is assigned a severity level of Medium\.

```
aws ssm create-association --association-name Start_EC2_Instances --targets Key=tag:Environment,Values=Linux --name AWS-StartEC2Instance --parameters  AutomationAssumeRole=arn:aws:iam::123456789012:role/AmazonSSMAutomationRole --automation-target-parameter-name InstanceId  --schedule "cron(0 7 ? * MON *)" --max-errors "5" --max-concurrency "10" --compliance-severity "MEDIUM"
```

**Note**  
If you use tags to create an association on one or more target instances, and then you remove the tags from an instance, that instance no longer runs the association\. The instance is disassociated from the State Manager document\. 

## Creating an Association that Runs an Automation Workflow \(PowerShell\)<a name="automation-sm-target-ps"></a>

Use the following format to create an AWS Tools for PowerShell command that creates a State Manager association that runs an Automation workflow\.

**Note**  
If you create an association by using AWS Tools for Windows PowerShell, use the `-Target` parameter to target instances for the association\. Don't use the `-InstanceID` parameter\. The `-InstanceID` parameter is a legacy parameter\. 

```
New-SSMAssociation -AssociationName AssociationName -Target Targets -Name DocumentName -Parameters (if any) -AutomationTargetParameterName (parameter to target) -ScheduleExpression "cron_or_rate_expression"
```

The following example creates an association to start Amazon EC2 instances tagged with `"Environment,Linux"`\. The association uses the `AWS-StartEC2Instance` document and the specified automation assume role to start the targeted instances at 7:00 every Monday morning\. This association runs simultaneously on 10 instances maximum at any given time\. Also, this association stops running on more instances for a particular execution interval if the error count exceeds 5\. For compliance reporting, this association is assigned a severity level of Medium\.

```
$Target = New-Object Amazon.SimpleSystemsManagement.Model.Target
$Target.Key = "tag:Environment"
$Target.Values = "Linux"

New-SSMAssociation -AssociationName "Start_EC2_Instances" -Target $Target -Name "AWS-StartEC2Instance" -Parameters @{"AutomationAssumeRole"="arn:aws:iam::123456789012:role/AmazonSSMAutomationRole"} -AutomationTargetParameterName "InstanceId" -ScheduleExpression "cron(0 2 ? * SUN *)" -ComplianceSeverity MEDIUM -MaxConcurrency 10 -MaxError 5
```

**Note**  
If you use tags to create an association on one or more target instances, and then you remove the tags from an instance, that instance no longer runs the association\. The instance is disassociated from the State Manager document\. 

## Troubleshooting State Manager Automation Executions<a name="systems-manager-state-manager-automation-documents-troubleshooting"></a>

Systems Manager Automation enforces a limit of 25 concurrent executions and 75 queued executions per account, per Region\. If a State Manager association that runs an Automation document shows a status of **Failed** and a detailed status of **AutomationExecutionLimitExceeded**, then your execution may have reached the limit\. As a result, Systems Manager throttles the executions\. To resolve this issue, do the following:
+ Use a different rate or cron expression for your association\. For example, if the association is scheduled to run every 30 minutes, then change the expression so that it runs every hour or two\.
+ Delete existing Automation executions that have a status of **Pending**\. By deleting these executions, you clear the current queue\.