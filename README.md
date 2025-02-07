# Prebuilt Promotion

## Table of Contents

- [Prebuilt Promotion](#prebuilt-promotion)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Supported IAP Versions](#supported-iap-versions)
  - [Getting Started](#getting-started)
    - [Prerequisites](#prerequisites)
    - [Capabilities](#capabilities)
    - [How to Install](#how-to-install)
    - [Testing](#testing)
  - [Using This Pre-Built](#using-this-pre-built)
    - [Input Schema](#input-schema)
    - [Git Credentials (Bitbucket and GitLab)](#git-credentials-bitbucket-and-gitlab)
    - [Host Override (optional) (Bitbucket and GitLab)](#host-override-optional-bitbucket-and-gitlab)
    - [IAP Credentials (for Promote stage)](#iap-credentials-for-promote-stage)
    - [Integrating with GitLab](#integrating-with-gitlab)
    - [Integrating with Bitbucket](#integrating-with-bitbucket)
    - [Integrating with Github](#integrating-with-github)
    - [Pipeline Stages](#pipeline-stages)
  - [Additional Information](#additional-information)

## Overview

**Prebuilt Promotion** takes a pre-built installed on IAP in Admin Essentials and creates a new project or updates an existing project by way of a merge request or pull request in the specified version control platform (i.e. GitLab, Github, Bitbucket) using the pre-built's IAP components and other files for CI/CD of maintaining those resources.

In either case of a new project or an MR or PR created, a pipeline will be started that promotes the pre-built into a specified IAP environment by way of the version control's CI/CD environment variables. More details on the different stages of the pipeline can be found under [pipeline stages](#pipeline-stages) below.

## Supported IAP Versions

Users must satisfy the following pre-requisites:

- Itential Automation Platform
  - `^2023.1`
- App-Artifacts
  - `^6.5.1-2023.1.0`
- One of the following version control adapters
  - [GitLab Adapter](https://gitlab.com/itentialopensource/adapters/devops-netops/adapter-gitlab) (`^v0.8.1`)
  - [Bitbucket Adapter](https://gitlab.com/itentialopensource/adapters/devops-netops/adapter-bitbucket) (`^v0.4.0`)
  - [GitHub Adapter](https://gitlab.com/itentialopensource/adapters/devops-netops/adapter-github) (`^v0.3.0`)
  
## Getting Started

### Prerequisites

Running `Prebuilt Promotion` successfully requires the following:

- Target pre-built to push to version control service already installed in Admin Essentials on IAP instance where `Prebuilt Promotion` is being run.
- The pipeline runners from the version control service need to have network access to the target IAP instance for the promotion from the version control service to the target IAP instance to succeed.
- Defining the version control service CI/CD variables listed below for promotion of the pre-built to the target IAP instance.

### Capabilities

- Performs rediscover on the pre-built if components are deleted or additional components are added
- Includes the necessary configuration files and scripts to run a CI/CD pipeline process that will:
  - Validate security of code and quality of manifest.json
  - Version bump the pre-built bundle when pushing to master
  - Generate an updated `artifact.json` file reflecting the changes made in the up-to-date artifact bundle
  - Promote the `artifact.json` to the next IAP environment in the CI/CD process (e.g. staging). This step will overwrite exising components in IAP.

### How to Install

To install this pre-built:

- Verify that you are running the documented [prerequisites](#prerequisites) in order to install the pre-built.

- Follow the instructions on the Itential Documentation site for [installing a pre-built](https://docs.itential.com/docs/installing-uninstalling-a-prebuilt).

### Testing

While Itential tests this pre-built and its capabilities, it is often the case the customer environments offer their own unique circumstances. Therefore, it is our recommendation that you deploy this pre-built into a development/testing environment in which you can test the pre-built.

## Using This Pre-Built

This pre-built can be run from a standalone Operations Manager automation or a [childJob task](https://docs.itential.com/docs/childjob-3).

**Note**: To run this pre-built from Operations Manager, use the automation `Prebuilt Promotion`. The entry point workflow used to run this pre-built in a childJob task is called `Prebuilt Promotion`.

### Input Schema

If running this pre-built from Operations Manager, refer to the following list of JSON Form inputs that correspond to image below.

<table><tr><td>
  <img src="https://gitlab.com/itentialopensource/pre-built-automations/prebuilt-promotion/-/raw/release/2023.1/images/sampleFormData.png" alt="sampleForm" width="600px">
</td></tr></table>

The following table details the property keys of the input object.
| key                                      | type    | required | description |
|------------------------------------------|---------|----------|---------------------------------------------------------|
| Check In Prebuilt -> Version Control Service                  | string   | yes      | The version control adapter used to push the pre-built files to the respective version control service such as GitLab or GitHub. <br><br>This value is also used to populate the `respository.type` field of the created Pre-Built. Further, the adapter associated with this value’s `hostname` is used to populate the `repository.hostname` field of the created Pre-Built. For more information about the Pre-Built repository setting, see [here](https://docs.itential.com/docs/configuring-multiple-prebuilt-repositories-1#managing-prebuilt-repository-configurations).|
| Check In Prebuilt -> Project Name    | string  | yes | The project name to Update/Create. Avoid using spaces in the name for Github Projects.  |
| Check In Prebuilt -> Make Project Private (Github Only) | boolean | no | If checked or set to true, will make project private in GitHub. If left unchecked, will make project public in GitHub |
| Check In Prebuilt -> Group Path | string  | yes | The group path in which the project will be created. <br><br> This value is also used to populate the `respository.path` field of the created Pre-Built. For more information about the Pre-Built repository setting, see [here](https://docs.itential.com/docs/configuring-multiple-prebuilt-repositories-1#managing-prebuilt-repository-configurations). |
| Check In Prebuilt -> Re-Discover Prebuilt | boolean  | no | If checked or set to true, the pre-built will perform re-discovery of IAP components by reference from existing IAP components in pre-built. This is needed if any IAP components have been added to or deleted from the pre-built since committing changes to version control service. For example if adding a new childJob task that has a new workflow, ensure this box is checked for that workflow to be included in pre-built changes to push to version control service. If unchecked, will commit IAP components already associated with pre-built as found in IAP Admin Essentials. |
| Check In Prebuilt -> Prebuilt | string  | yes  | The pre-built found on the IAP instance's Admin Essentials to push to the respective version control service. |
| Check In Prebuilt -> Project Path | string  | no  | Bitbucket project to create repo under. Only valid for Bitbucket projects. |
| Check In Prebuilt -> For Existing Projects -> MR Type | string  | yes  | The type of merge request change being made: patch, minor, or major. |
| Check In Prebuilt -> For Existing Projects -> Commit Message  | object  | yes  | The commit message to use in version control commit.  |
| Check In Prebuilt -> For Existing Projects -> Target Branch   | object  | yes  | The branch to target pre-built changes to. |

### Git Credentials (Bitbucket and GitLab)

The following tables indicate version control CI/CD variables that need to be set within the version control service (i.e. Bitbucket or GitLab) for the pipeline created by Prebuilt Promotion to work as expected.

|Variable            |Example Value         |Description |
|--------------------|----------------------|------------|
|`CI_GIT_EMAIL`      |`example@gitlab.com`  |Email for user account running the pipeline. This will be used to set-up version control access within your pipeline environment.|
|`CI_GIT_USERNAME`     |`ci-bot-user`         |Username of user account running the pipeline. This will be used to set-up version control access within your pipeline environment.|
|`ID_RSA`              |Contents of private id_rsa key|The value for ID_RSA is the private key from an SSH key created by a Windows, Linux, or macOS machine. The corresponding public key generated should be added to the account you want to use for committing the generated artifact.json from the pipeline to the target project of your version control system. Refer to vendor documentation [here](https://docs.gitlab.com/ee/user/ssh.html#add-an-ssh-key-to-your-gitlab-account) for adding public SSH key to account in GitLab and [here](https://support.atlassian.com/bitbucket-cloud/docs/set-up-personal-ssh-keys-on-windows/#Provide-Bitbucket-Cloud-with-your-public-key) for adding public SSH key to account in Bitbucket Cloud.|

### Host Override (optional) (Bitbucket and GitLab)

|Variable            |Example Value         |Description |
|--------------------|----------------------|------------|
|`CI_ORIGIN_HOSTNAME` (optional) |`git@self-hosted-gitlab.com`                      |GitLab hostname. This will be used to setup GitLab access within your pipeline environment. When this variable is not set, the hostname will default to `git@gitlab.com`|
|`CI_ORIGIN_SSH_PORT` (optional)| `2224` | Port that GitLab is running SSH on. This variable should be set if the SSH client connection is not accessible via the default 22 port.

### IAP Credentials (for Promote stage)

If you do not want to promote your pre-built bundle to a higher IAP environment, set `PROMOTE = False` and disregard the variables in this section.
|Variable            |Example Value         |Description |
|--------------------|----------------------|------------|
|`PROMOTE` (optional, defaults to `True`)     |`True` or `False`| Determines if "promote" stage of pipeline will run.
|`IAP_HOSTNAME`        |`https://ACME.itential.com`| Used in promotion stage to connect to IAP|
|`IAP_TOKEN`          |`123_sample_token`| Required if using token authentication and NOT basic authentication|
|`IAP_USERNAME`        |`iap_user@itential.com`| Required if using basic authentication|
|`IAP_PW`              |`itential123`| Required if using basic authentication
|`IAP_SSL_CERT`        | `-----BEGIN CERTIFICATE-----...-----END CERTIFICATE-----`| Required if using two-way SSL verification for HTTPS for target IAP promotion
|`IAP_PUSH_TO_LOCAL` (optional) |`True` or `False` | Determines the scope of the pre-built. If this variable is not set, the scope value will default to `local`. If this variable is set to `False`, the scope is set to the repository specified in the artifact.json file. <br><br>When `IAP_PUSH_TO_LOCAL` is set to `True` the object below replaces whatever the `repository` field is set to in the Pre-Built. Further, the local `repository` configuration shown below will be used if on import of a Pre-Built the repository configuration provided is not found on the target IAP. For more information about the Pre-Built repository setting, see [here](https://docs.itential.com/docs/configuring-multiple-prebuilt-repositories-1#managing-prebuilt-repository-configurations).<pre>{<br>  "type": "local", <br>  "hostname": "localhost", <br>  "path": "/"<br>}</pre>|

### Integrating with GitLab

From the GitLab UI, ensure that the necessary CI/CD variables are defined within the scope of your project. 

<table><tr><td>
  <img src="https://gitlab.com/itentialopensource/pre-built-automations/prebuilt-promotion/-/raw/release/2023.1/images/setEnvVarGitlab.png" alt="envVarPageGitlab" width="800px">
</td></tr></table>

### Integrating with Bitbucket

Ensure that the necessary Workspace variables are set in your Workspace settings in Bitbucket.

<table><tr><td>
  <img src="https://gitlab.com/itentialopensource/pre-built-automations/prebuilt-promotion/-/raw/release/2023.1/images/setEnvVarBitbucket.png" alt="envVarPageBitbucket" width="800px">
</td></tr></table>

A Bitbucket runner has to be registered in order to run builds in pipelines. The runner can be registered either for a repository or a workspace. A repository runner is used to run builds for that specific repository in a workspace. A workspace runner is used to run builds in pipelines for any repository in the workspace.

The steps to register a runner are in Bitbucket Cloud's official documentation page https://support.atlassian.com/bitbucket-cloud/docs/adding-a-new-runner-in-bitbucket/

<table><tr><td>
  <img src="https://gitlab.com/itentialopensource/pre-built-automations/prebuilt-promotion/-/raw/release/2023.1/images/registerBitbucketRunner.png" alt="registerBitbucketRunner" width="800px">
</td></tr></table>

### Integrating with Github

Under your Github organization settings, ensure that the necessary environmental variables are defined as Action secrets.

<table><tr><td>
  <img src="https://gitlab.com/itentialopensource/pre-built-automations/prebuilt-promotion/-/raw/release/2023.1/images/setEnvVarGithub.png" alt="envVarPageGithub" width="800px">
</td></tr></table>

### Pipeline Stages

|Stage    |Description     |
|---------|----------------|
|quality  | Lints code, test code quality, and ensures that there are no security vulnerabilities|
|test     | Validates path links and schema of `manifest.json`. |
|bump     | Bumps pre-built version (only done on `master` branch). |
|generate | Generates updated `artifact.json` file. |
|promote  | Promotes updated `artifact.json` to specified IAP environment using the credentials and hostname specified in the environmental variables. |

## Additional Information

Please use your Itential Customer Success account if you need support when using this Pre-Built.
