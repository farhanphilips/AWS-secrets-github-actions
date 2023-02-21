# Use AWS Secrets Manager secrets in GitHub jobs

To Pull secrets from AWS in GitHub Actions workflow, we can use the AWS secrets manager service. Here are the general steps to follow: 
1.	Create an IAM user in the AWS account with necessary permissions to access the secrets in the AWS Secrets Manager. The user should have the ‘secretsmanager:GetSecretValue’ permission at the minimum.
2.	Create a new secret in AWS Secrets Manager or use an existing one.
3.	In the GitHub repository with the reusable workflows, go to the settings tab and navigate to he secrets page.
4.	Click on “New Secret” button and enter the name and value of IAM user Credentials of AWS, In our case its ‘AWS_ReadSecrets_Role’
5.	Fetching Secrets from AWS is a two-part process –

    a.	We need to configure the aws credentials, this step will allow us to use a short-lived credential and avoid storing additional keys outside of the AWS Secrets Manager, make sure IAM role has the following permissions embedded in it in order to correctly configure the aws credentials using the actions -

        i.	GetSecretValue on the secrets we want to retrieve

        ii. 	ListSecrets on all secrets
    
    
    <img src="https://user-images.githubusercontent.com/122483390/220396268-4dd0fcb9-fd22-470e-91a1-ccf9a9c3a358.png" alt="carbon (1)" width="550" height="220">

    Here the role-to-assume parameter provides the short-lived credentials needed for the action and the role-duration-seconds is the time duration specified for the session.

     b.	In the second step we need to get the secrets from the AWS Secrets Manager. We can have another action that will retrieve the secrets from AWS Secrets manager and add them as masked Environment variables in the GitHub Workflow.

    <img src="https://user-images.githubusercontent.com/122483390/220397642-bcc8264a-f6f4-4d38-b645-63d4f3f2205a.png" alt="carbon (2)" width="450" height="220">

    **Parameters:** 

        i.secret-ids: Secret ARNS, names, and name prefixes.

        ii.By default, the step creates each environment variable name from the secret name, transformed to include only uppercase letters, numbers, and underscores, and so that it doesn't begin with a number.

        iii.To set the environment variable name, enter it before the secret ID, followed by a comma. For example, ENV_VAR_1, secretId creates an environment variable named ENV_VAR_1 from the secret secretId.

        iv.The environment variable name can consist of uppercase letters, numbers, and underscores.

        v.To use a prefix, enter at least three characters followed by an asterisk. For example, dev* matches all secrets with a name beginning in dev. The maximum number of matching secrets that can be retrieved is 100. If you set the variable name, and the prefix matches multiple secrets, then the action fails.

        vi.parse-json-secrets: (Optional - default false) By default, the action sets the environment variable value to the entire JSON string in the secret value. Set parse-json-secrets to true to create environment variables for each key/value pair in the JSON.


    Note that if the JSON uses case-sensitive keys such as "name" and "Name", the action will have duplicate name conflicts. In this case, set parse-json-secrets to false and parse the JSON secret value separately.




6.	 If we wish to reuse an environment variable which has a dependency on AWS secrets, we can create a separate step in a workflow so that the subsequent steps in a job can access the declared global environment variable

![image](https://user-images.githubusercontent.com/122483390/220398806-9d672a06-7473-4b09-a3b1-f1f3beffae01.png)



As we can see above, a new action step in the job allows the ‘VSS_NUGET_EXTERNAL_FEED_ENDPOINTS’  to be accessible globally as an environment variable and also be fetched explicitly
In case the set environment variable step run command requires escape character formatting, be sure to do so as the step will fail without it.

**As a rule of thumb -**

```
For Linux Runners:

The character escaping can be formatted using the double quotes – “ before every / present in string

To have this environment variable be accessible to consequent steps make sure to use the $GITHUB_ENV

For Windows Runner:
              The character escaping can be formatted using the back quote - ` before every / present in string
To have this environment variable be accessible to consequent steps make sure to use the $env:GITHUB_ENV
```

# Calling reusable workflows from another repository to pull secrets from AWS

In order to call a reusable workflow from another repository which consists of steps to pull secrets from AWS Secrets Manager, we can have the following YAML in our current repository. For example, lets look at this Black duck workflow - 


![image](https://user-images.githubusercontent.com/122483390/220404080-fd08a6b0-e6ae-41fa-8a05-4935ae7e967d.png)

**Point 1**

This GitHub Action is named "Blackduck" and has three types of triggers: schedule, push, and pull_request. The schedule trigger CRON Job is set to run every Saturday at 11:30 PM UTC (cron syntax: '30 23 * * 6').

**Point 2**

The push trigger will run the workflow on any push event that affects either the "master" branch or any branch that starts with "release/".
The pull_request trigger will run the workflow on any pull request that affects either the "master" branch or any branch that starts with "release/".

**Point 3**

This GitHub Action defines a single job named "Blackduck" which uses a pre-existing workflow located in the philips-internal/ix-workflows repository at .github/workflows/blackduck.yml and the release for that repository currently stands at ‘v2’. 

**Point 4**

This job specifies three secrets (AWS_ReadSecrets_Role, ACR_USERNAME, and ACR_PASSWORD) that are used in the execution of the job in the reusable workflow.
The purpose and steps of the workflow itself are not defined in this YAML file but can be found in the referenced workflow in the philips-internal/ix-workflows repository.


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the Apache 2.0 License. See the LICENSE file.

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
