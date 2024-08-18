# Static Resource Deployment Pipeline

Deploy static resources to Salesforce sandboxes.

## Metadata Information

During deployment, the workflow checks for a _METADATA.INI file to extract the version number. If the file is missing or lacks a version, it defaults to "0.0.0 (UNDEFINED)". The description for the Salesforce resource is then created in the format: `v$VERSION - {{ github.event.sender.login }} - $(date '+%Y-%m-%d %H:%M:%S')`. For example, it might generate `v0.0.1 - dmoruzzi - 2023-12-31 13:12:11`.

### Sample Configuration File

Save `_METADATA.INI` in the root of the ZIP file with the following contents:
```ini
VERSION = "0.0.1"
```

## Roles and Responsibilities

Type | Description | Team | Frequency
--- | --- | --- | ---
Usage | Commit static resource ZIP content to `origin/main` branch | Static Resource Team | Per Usage
Usage | Validate the correct sandbox selection for the target sandbox | Static Resource Team | Per Usage
Usage | Validate the successful deployment of static resources | Static Resource Team | Per Usage
Usage | Initial failure triage and notification of deployment issues | Static Resource Team | Per Issue
Sustainment | Passphrase rotation of Salesforce CLI authentication tokens + GitHub Secrets configuration | DevOps | Yearly
Sustainment | Passphrase rotation of Service Integration GitHub authentication tokens + GitHub Secrets configuration on Static Resource Team GitHub repository | DevOps | Yearly
Innovation | Adding support to new salesforce sandboxes to meet new quality acceptance sandboxes requirements | DevOps | Project Allocation
Innovation | Integrating deployment pipeline with requested features | DevOps | Project Allocation
Innovation | Resolving deployment pipeline issues | DevOps | _Reasonable Effort_ and/or Project Allocation
Quality Gate | Right to suspend deployment pipeline | DevOps | As Needed
Quality Gate | Right to revoke Service Integration token authorization | DevOps | As Needed
Quality Gate | Right to revoke Salesforce CLI token authorization per sandbox | DevOps | As Needed
Quality Gate | Right to revoke individual contributor authorization to pipeline | DevOps | As Needed

This workflow is a quality-improvement pipeline and is **not** considered a production pipeline. As such, there are no service level guarantees or continued support agreements. In the event of an issue, corrections and enhancements are made on a reasonable effort basis but are not guaranteed.

In the event the static resource pipeline is unavailable, please contact the DevOps team to engage a manual deployment of the static resources.

## Workflow Details

- **Trigger**: Manual trigger through `workflow_dispatch` with a required input for the target environment (`int`, `qa`, or `training`).
- _\_METADATA.INI_ file is an _optional_*[1] input on the root of the ZIP file containing metadata associated with the static resource.

### Steps

1. **Prepare Job Variables & Structure**: Set environment variables and create necessary directories.
2. **Checkout Code**: Checkout the `main` branch of the repository.
3. **Verify Target Application File**: Check for the existence of the target application file and create an issue if it's missing.
4. **Cache Salesforce CLI**: Cache the Salesforce CLI to improve performance and avoid frequent downloads.
5. **Install Salesforce CLI**: Install the Salesforce CLI if it is not cached.
6. **Add Salesforce CLI to PATH**: Ensure the Salesforce CLI is available in the system path.
7. **Display Salesforce CLI Version**: Display the version of the installed Salesforce CLI.
8. **Generate Placeholder Salesforce Project**: Generate a placeholder Salesforce project to facilitate deployments.
9. **Make Static Resource Folder Structure**: Create the static resource folder structure.
10. **Extract Target Application File**: Unzip the target application file into the static resource directory.
11. **Parse Metadata Information**: Read metadata information and create a `.resource-meta.xml` file.
12. **Authenticate to Salesforce**: Log in to the appropriate Salesforce environment based on the target.
13. **Check Resource Deployability**: Perform a dry-run deployment to check for issues.
14. **Display Deployment Check Log**: Review and display logs from the deployment check.
15. **Deploy Static Resource**: Deploy the static resource to Salesforce.
16. **Upload Deployment Log**: Upload the deployment log for later review.

### Requirements

Environment | Secret Name | Description
--- | --- | ---
int | `AUTH_INT` | Salesforce sfdxAuthUrl; restrict secret to deployment environment & restrict access to `origin/main` branch only
qa | `AUTH_QA` | Salesforce sfdxAuthUrl; restrict secret to deployment environment & restrict access to `origin/main` branch only
training | `AUTH_TRAINING` | Salesforce sfdxAuthUrl; restrict secret to deployment environment & restrict access to `origin/main` branch only
training | `AUTH_UAT` | Salesforce sfdxAuthUrl; restrict secret to deployment environment & restrict access to `origin/main` branch only -- used strictly to validate (not deploy) UAT environment for training deployments

### Troubleshooting

Issues are logged as GitHub issues with appropriate labels and descriptions. Please refer to the workflow run for more details.

## Points of Contact

NetId | Name | Description | Team
--- | --- | --- | ---
dmoruzzi | David Moruzzi | Deployment pipeline maintainer | DevOps
scarrion | Scott Carrion | Service Integration owner | DevOps
pharrington | Patrick Harrington | Lead requirements engineer | Static Resource Team
gkelly | Gabriel Kelly | Integration pipeline maintainer | Static Resource Team

## Future Discussions

The below are some of the areas that were brought up during the development design process:

- Onboard an additional service integration account for GitHub accounts to better manage access; grant SI_2 access to write to `develop` and then grant SI_1 access to: Actions=Read/Write,  Contents=Read/Write,Variables=Read/Write and then have a separate GitHub Workflow tracking pushes to the `deploy` branch; if push contains `int.zip`, `qa.zip`, or `training.zip` then write to the `main` branch and then trigger the deployment pipeline. 
- Centralize deployment frequency metrics for time/cost savings calculations to accurately priorize pipeline innovation time compared to other areas of innovation.
- Offload GitHub event senders access control to Active Directory groups synced with GitHub Teams.
- Automate Azure DevOps (ADO) comments on pre-defined work items per metadata information.
- Automate Jira issue transitions to move work items to an appropriate state per metadata information.
- Deployment pipeline of static resources to children development sandboxes.


____

[1]: This field is likely to become mandatory in the future.
