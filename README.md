# Overview

The solution is deployed with AWS CloudFormation. When deployed, an AWS CodeBuild project and an Amazon S3 bucket to store the Prowler generated reports are created. An AWS Lambda function is then used to start the AWS CodeBuild project.

The parameter (user input) defaults will run a basic scan in a single account. However, you can choose different parameters to run more extensive scans or to scan multiple accounts. The deployment process takes less than 5 minutes to complete. The solution’s AWS CloudFormation templates are provided for review in this Github repository.

Once the template is deployed, the CodeBuild project will run. The default assessment takes around 5 minutes to complete. The time to complete a security assessment will vary depending on the number of resources and the scan options selected. At the end of the assessments the reports are delivered to the created S3 Bucket.


## Parameters:

SATv2 can be customized by updating the CloudFormation parameters. This section summarizes the available options and provides a link to the section with more information.


| Parameter                | Description                                                                                                                                  | More information     |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|----------------------|
| ProwlerScanType          | Specify which type of scan to perform. Selecting full without specifying different ProwlerOptions will do a full scan. To perform a specific check, choose Full and append -c to ProwlerOptions. | Scan types           |
| MultiAccountScan         | Set this to true if you want to scan all accounts in your organization. You must have deployed the prerequisite template to provision a role, or specify a different ProwlerRole with the appropriate permissions. | Multi-account scan   |
| Reporting                | Set this to true if you want to summarize the Prowler reports into a single csv and create a presentation. This is helpful when scanning multiple accounts. | Reporting Summary    |
| EmailAddress             | Specify an address if you want to receive an email when the assessment completes.                                                            | Notifications        |
| **Advanced Parameters**  |                                                                                                                                              |                      |
| ConcurrentAccountScans   | For multi-account scans, specify the number of accounts to scan concurrently. This is useful for large organizations with many accounts. Selecting more than three changes the size of the CodeBuild instance and may incur additional costs. |                      |
| CodeBuildTimeout         | Set the timeout for the CodeBuild job. The default is 300 minutes (5 hours).                                                                 |                      |
| MultiAccountListOverride | Specify a space delimited list of accounts to scan. Leaving this blank will scan all accounts in your organization. If you can't provide delegated ListAccount access, you can provide the MultiAccountListOverride parameter. | Multi-account scan   |
| ProwlerOptions           | Specify the parameters for Prowler. The --role and ARN will automatically be added to the end of the parameters you specify. This can also be used to specify a single check. | Full scan            |
| ProwlerRole              | The role that Prowler should assume to perform the scan. Change this if you want to specify your own role with different permissions.         |                      |

## Deploy the solution using AWS Console.


1. Download the 2-sat2-codebuild-prowler.yaml CloudFormation template.
2. Navigate to the AWS CloudFormation console.
3. In the navigation pane, choose Stacks.
4. Choose Create stack.
5. Under Specify template, select Upload a template file.
6. Choose sat2-codebuild-prowler.yaml you downloaded in step 1.
7. Choose Next.
8. For Stack name, enter sat2.
9. Choose Next.
10. On the Configure stack options page, choose Next.
11. On the Review SAS page, select the box I acknowledge that AWS CloudFormation might create IAM resources. and choose Submit.

## Basic Scan
To see a list of checks, review basic checks.

- Maintain current contact details.
- Find obsolete Lambda runtimes.
- Ensure CloudTrail is enabled in all regions.
- Ensure AWS Config is enabled in all regions.
- Ensure no security groups allow ingress from 0.0.0.0/0 or ::/0 to any port.
- Check if GuardDuty is enabled.
- Ensure IAM password policy requires at least one lowercase letter.
- Ensure IAM password policy requires at least one number.
- Ensure IAM password policy requires at least one symbol.
- Ensure IAM password policy requires at least one uppercase letter.
- Ensure MFA is enabled for the root account.
- Ensure access keys are rotated every 90 days or less.
- Ensure there are no S3 buckets open to Everyone or Any AWS user.

## Intermediate scan
To see a list of checks, review intermediate checks.

This scan will add `--severity critical high` to the Prowler scan options. With this selected Prowler will run all security checks that result in critical or high severity.

## Full scan
To see a list of checks, review full checks.

This option doesn't add any additional parameters to the Prowler scan. It will result in Prowler running 359+ checks.

You can also use the full scan to customize the scan however you would like.

For ProwlerScanType choose Full.

For ProwlerOptions, append the check. For example, to check only if GuardDuty is enabled, enter:

`aws --ignore-exit-code-3 -c guardduty_is_enabled`

## Issue Faced and Solution for Single Account

### Issue:
When deploying the solution for a single account, users encountered an issue where the Prowler scan would not complete successfully. This was due to insufficient permissions assigned to the AWS Lambda function, which is responsible for starting the AWS CodeBuild project.

### Solution:
To resolve this issue, we updated the AWS CloudFormation template to include the necessary permissions for the AWS Lambda function. Specifically, we added the following permissions:
- `codebuild:StartBuild` to allow the Lambda function to start the CodeBuild project.
- `s3:PutObject` to allow the Lambda function to store the Prowler generated reports in the S3 bucket.

By including these permissions, the Prowler scan now completes successfully for a single account deployment. Users can deploy the updated CloudFormation template and run the scan without encountering permission-related issues.


## Review the results

Once the solution is deployed, you can review the results in the S3 bucket.

To review the results, follow these steps.

1. Navigate to the Amazon S3 console in the account you deployed Prowler.
2. Select the bucket that starts with sat2-prowler-prowlerfindingsbucket-
3. Choose the folder with the date and time of the scan.
4. For each account, there will be 4 file types (csv, html, json, json-ocsf) in the format prowler-output-<aws-account-id>-<datetime>.
5. Select one of the html objects.
6. Choose Open.


This solution does not integrate with GuardDuty, Security Hub.

## Clean Up
After you run the solution, you should delete the CloudFormation Stacks to remove resources that are no longer needed. The S3 bucket with the Prowler scan results will remain.

To remove the security assessment solution from your account, follow these steps.

1. Navigate to the AWS CloudFormation console in the account you ran the tool from (ProwlerAccountID).

2. In the navigation pane, choose Stacks.

3. Choose the sat2-prowler Stack.

4. Choose Delete.


## References

- https://github.com/awslabs/aws-security-assessment-solution?tab=readme-ov-file
- https://github.com/prowler-cloud/prowler
- https://docs.aws.amazon.com/codebuild/latest/userguide/what-is-codebuild.html
- https://docs.aws.amazon.com/lambda/latest/dg/welcome.html
- https://docs.aws.amazon.com/cloudformation/latest/userguide/what-is-cloudformation.html
- https://docs.aws.amazon.com/s3/index.html
- https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html
- https://docs.aws.amazon.com/codebuild/latest/userguide/what-is-codebuild.html
