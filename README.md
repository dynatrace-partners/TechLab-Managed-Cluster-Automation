**ATTENTION:** This is currently a work in progress. Please check back when the course is complete

![](./images/partner_program.png)

# Overview
Hello and welcome to this Dynatrace Partner TechLab. We are launching these as self-paced training for all our partners. During this session we will focus on automation and complete hands on exercises on how to automate creating a monitoring environment, usergroup and user and auto deploy OneAgent to the new environment.

The goals of this tutorial are;
1. **Environment Management** - Automatically create environments on a managed cluster. Useful for onboarding new customers, training events, creating additional environments etc.
2. **User Group Management** - Automatically create User Groups with the correct access to your freshly created environment. Useful for onboarding new users, training events and user management.
3. **User Management** - Automatically create Users and assign them to the correct user group. Useful for onboarding new users, training events and user management.
4. **Environment API token Management** - Automatically create environment API tokens with the correct roles. Useful for automated OneAgent deployments, integrations, training events, key rotation etc.
5. **Automated OneAgent Deployment** - Start ec2 instances and automatically install and configure the OneAgent. Useful for fast deployments of the OneAgent and ensuring new resources are implemented. 

First we will run these steps individually so you get a feel for how they operate then we will provide a script that allows you to do it all at once.

## Pre-Requisites
1. You need an AWS account. If you don't have one [get one here](https://aws.amazon.com/)
2. AWS Access key and secret access key for an IAM user with rights ec2:RunInstances,  ec2:DescribeImages & ec2:CreateTags. If you don't know how to do this don't worry it is explained in the next section.
3. You need a Dynatrace Managed Cluster license. If you don't have one, reach out to your Dynatrace Partner Program Manager (PPM) to be granted a trial license.

Region | Name | Email
------------ | ------------- | -------------
APAC | Dhruv Rajvanshi | [Dhruv.Rajvanshi@dynatrace.com](mailto:Dhruv.Rajvanshi@dynatrace.com)
EMEA | Patricia Jesus Silva | [patricia.jesus.silva@dynatrace.com](mailto:patricia.jesus.silva@dynatrace.com)
LATAM | Luiz Rodrigues | [Luiz.Rodrigues@dynatrace.com](mailto:Luiz.Rodrigues@dynatrace.co)
NORAM | Jamie Mallett | [Jamie.Mallett@dynatrace.com](mailto:Jamie.Mallett@dynatrace.com)

4. You need a Managed Cluster. A single trial node is sufficient. This could be run on a VM or in your EC2 account [view specs](https://www.dynatrace.com/support/help/setup-and-configuration/dynatrace-managed/installation/dynatrace-managed-hardware-and-system-requirements/)
5. Optional: ActiveGates - An ActiveGate is not required to complete these exercises, but depending on your install and security requirements you may also require a Cluster and/or Environment ActiveGate. For more info see [When do I need to install an ActiveGate?](https://www.dynatrace.com/support/help/setup-and-configuration/dynatrace-activegate/basic-concepts/when-do-i-need-to-install-an-activegate/)
6. [Postman](https://www.postman.com/downloads/) installed
7. You need to clone or copy the content of this GitHub repo to your local disk!

## Preparation

**Amazon Web Services (AWS)**

As we are going to use AWS EC2. We have tested this tutorial on eu-west-1 (Ireland), Paris (eu-west-3) & ..... . To be on the safe side we suggest you pick one of these regions!

Ensure you have created an IAM user  with rights ec2:RunInstances, ec2:DescribeImages & ec2:CreateTags I recommend you create a user for this with only these rights that can be removed in the future.

Login to your AWS account and navigate to IAM > Users > Add user and complete the following
1. Give your user a name and select Programmatic access

![](./images/preparation/awsconfigure.png)

2. Select Create Group

![](./images/preparation/awsCreateGroup.png)

3. Name your group and select Create policy. The Policy creater should open in a new tab.

![](./images/preparation/awsGroupName.png)

4. Choose the service ec2 and under actions search and add RunInstances, DescribeImages & CreateTags. Then click Review policy

![](./images/preparation/awsCreatePolicy.png)

5. It should look like the following. Select Create Policy

![](./images/preparation/awsPolicy.png)

6. Return to your previous tab, click Refresh and search for you new policy. Tick the policy and select Create Group

![](./images/preparation/awsPolicyGroup.png)

7. Ensure your group has been added for your user and the policy is attached. Click Next:tags. Tags are optional so add one if you wish and click Next:Review

![](./images/preparation/awsSetPer.png)

8. Your new user should look like the following. Click Create User. Store your credentials, you can download csv to save them in a file. 

![](./images/preparation/awsUserSum.png)

Once you have your IAM user and role we need a couple of things
1. Your AWS Access Key ID. It should look something like `AKIAIOSFODNN7EXAMPLE`
2. Your AWS Secret key. It should look something like `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`
3. Your AWS region. It should look something like `eu-west-1`

**Dynatrace**

You will need to have provisioned your dynatrace managed cluster. For installation instructions please follow the [documentation](https://www.dynatrace.com/support/help/setup-and-configuration/dynatrace-managed/installation/set-up-a-cluster/)

Once you have your cluster provisioned we need a couple of things
1. Your dynatrace managed URL. It should look something like `https://xxxxxx.dynatrace-managed.com`
2. Verify the node enpoint by clicking on the node on the home screen in CMC. Ensure the value in `IP address and port for OneAgent traffic` will be reachable by the OneAgent. Safeguard is to configure the cluster domain name.

![](./images/preparation/nodeEndpoints.png)

3. Cluster API token. Inside CMC go to Settings -> API tokens -> Cluster tokens and select Generate token. Grant it the scope of Service Provider API

![](./images/preparation/clusterapi.png)

Expand the token and copy the API key

![](./images/preparation/clusterapitoken.png)

**Postman**

For this tutorial we will leverage postman to make our API calls. If you would like to take these concepts and incorporate them into your dynatrace provisioning process you can choose any tool you like providing it can make API calls. 

Once you have installed Postman we need a couple of things
1. Upload the <a id="raw-url" href="https://raw.githubusercontent.com/dynatrace-partners/TechLab-Managed-Cluster-Automation/master/postman/TechLab-Managed-Cluster-Automation.postman_environment.json">postman environment</a> we have provided.

    Inside Postman click on import, select file and upload TechLab-Managed-Cluster-Automation.postman_environment.json

    ![](./images/preparation/postmanEnv.png)

2. Upload the [<a id="raw-url" href="https://raw.githubusercontent.com/dynatrace-partners/TechLab-Managed-Cluster-Automation/master/postman/TechLab-Managed-Cluster-Automation.postman_collection.json">postman collection</a> we have provided.

    Repeat the same process to import TechLab-Managed-Cluster-Automation.postman_collection.json

    ![](./images/preparation/postmanCollection.png)

3. Configure your postman environment variables

    * Click on the ![](./images/preparation/manageEnvironments.png) to manage your environments

    * Click on the environment name TechLab-Managed-Cluster-Automation

    * Set all the 5 environment variables to your values. Ensure to set both the initial and current values. For dtManaged it should look like 'xxxxxx.dynatrace-managed.com' do not include the 'https://'

    ![](./images/preparation/environmentVars.png)

    * Click on update to save your changes

4. Select your postman environment in the upper right hand corner dropdown

    ![](./images/preparation/postmanSelEnv.png)

# 1. Creating a new monitoring environment

**ATTENTION:** You must execute these requests in the order they are listed in here as the requests gather response data and create new environment variables that are used in later requests.

To create a new monitoring environment ensure you have set-up your managed cluster, have generated a Cluster API Token and configured Postman.

## Creating a new monitoring environment background 

**What is a monitoring environment?**

Your Dynatrace monitoring environment is where all your Dynatrace performance analysis takes place. Dynatrace OneAgent sends all captured monitoring data to your monitoring environment for analysis. A monitoring environment is analogous to an analysis server that provides all Dynatrace application-performance analysis functionality, including all dashboards, charts, reports and other tools.

**When would you create a new monitoring environment?**

It's common to set up multiple monitoring environments so that related entities can be grouped for discrete analysis. For example, you might set up one monitoring environment to monitor and analyze the performance of your production clusters. You might set up a second environment that's dedicated to the performance of your developers' machines and a third environment for your staging servers.
You might also be running managed as a service for your customers. In that case you would setup separate environments for each customer to ensure their data is segregated.

**How do you create a new monitoring environment?**

Monitoring environments can be created using the Cluster Management Console, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create a new monitoring environment and token management API token via an API call.

## Creating a new monitoring environment configuration

**Request configuration**

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the cluster v2 API endpoint environments. By making a request to this API endpoint we will create a single new environment on our managed cluster. The environment will be named `TechLab-Managed-Cluster-Automation-x` where the x is  unique number that starts at 1 and will count upward each time you execute the request to make sure the value us unique. Each request will create a single environment, separate requests are required if you wish to create multiple environments.

**Parameters**

By supplying the parameter createToken=true a token with the permission `Token management` is created when creating a new environment. This token is then returned in the response body. It can be used within the newly created environment to create other tokens for configuring this environment.
We are creating this token so we can use it in our next request to create an API token on our new environment with agent install rights.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our cluster API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\}
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server.
Key | Value | Description
------------ | ------------- | -------------
name | TechLab-Managed-Cluster-Automation-\{\{envNumber\}\} | This will be the name assigned to your environment and it should be unique. This name will be displayed to the users of the environment so in real customer projects ensure to choose and appropriate name. In our case I have set it to the same name as the techLab. The variable \{\{envNumber\}\} will be automatically created if it doesn't exist and set to 1. Each time this request is run the variable will increase by one to ensure the name is unique.
state | ENABLED | This sets our new environment in the enabled state meaning it will be accessible and immediately ready for use. 
tags | customerA, production | Tags are not visible in CMC, but they are via the cluster API. In large clusters with lots of environments proper tag assignments will make management much easier. In our case we will tag our environment as customerA and production, in the future this would allow me to filer for this exact environment if I specified both tags or I could filter for all customerA's environments or all production environments.

**Pre-request script**

This is part of postman and not a requirement to create an environment via an API call. You can use pre-request scripts in Postman to execute JavaScript before a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/pre-request-scripts/)
In our case the script first deletes any dynamic environment variables that would have been set if this collection has been run before, except envNumber as this should remain if it has been used before to keep the IDs unique. Then it checks to see if envNumber exists or not. If it doesn't exist \(and it won't the first time you do this exercise\) it creates it and sets the value to 1. If the variable does exist \(i.e. you have run the request before\) then it will increment the value by one. As detailed above this variable is used in our environment name.

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script parses the JSON response body for the ID and API token of our new environment and sets them as environment variables envID and envTokenManagementToken so we can use them in our later requests. 

**Executing the request**
1. Open the Create Environment request.
2. Verify your environment variables are set and selected.
    * Verify that you have selected your TechLab-Managed-Cluster-Automation environment then hover over the variable dtManaged and verify both the initial and current values are set, it should look like 'xxxxxx.dynatrace-managed.com' without the `https://`

    ![](./images/monitoringenvironments/createEnvironment.png)

    * On the Headers tab hover over \{\{dtAPI\}\} to ensure it has been sent
3. Click on `Send` to execute the request.
4. Check that the request received a `201 Created` response and the response body contains the id, name and token for your new environment

![](./images/monitoringenvironments/creteEnvResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

5. Check your new environment now exists in CMC

![](./images/monitoringenvironments/createEnvCMC.png)

6. Access your new environment by clicking on its name in CMC and then Go to environment
7. Verify your new envNumber,  envID and envTokenManagementToken have been set as environment variables in postman

![](./images/monitoringenvironments/creteEnvEnvVars.png)

Congratulations you have just created a new environment via an API call. Now Let's create a usergroup to grant access to this environment.

# 2. Creating a new user group

To create a new user group ensure you have completed the creating a new monitoring environment exercise.

## Creating a new user group background 

**What is a user group?**

You need to configure user groups in Dynatrace Managed to allow access to your monitoring environments. You can assign a pre-defined set of permissions to a group. Once a group is defined, you can add users to the group. Added users inherit the permissions of the groups they are assigned to. Any group can be modified to fit your needs. You can even create new groups and assign permissions to them.

**When would you create a new user group?**

User groups are created to control access. In managed, user groups can be assigned environment or cluster permissions. User groups can be created manually or via LDAP/SAML/OpenID integration. You would normally create a user group with cluster permissions for your dynatrace administrators so they  have access to Cluster Management Console, and can manage your monitoring environments and Dynatrace Server.

Environment permissions are granted to users of monitoring environments. They allow you to control which environments can be accessed and to which level. You would normally require multiple user groups for different user access levels. For example, you may require one with read only permissions for application owners and another with change monitoring settings for your DevOps team.

**How do you create a new user group?**

Assuming you are not using any user integration then user groups can be created using the Cluster Management Console, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create a new user group and assign full environment permissions to our previously created environment via an API call.

## Creating a new user group configuration

**Request configuration**

**ATTENTION:** There is currently no user group V2 API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the cluster v1 API endpoint groups. By making a request to this API endpoint we will create a single new user group on our managed cluster with complete environment rights to our previously created monitoring environment. The environment will be named in the same format as the environment, as in  `TechLab-Managed-Cluster-Automation-x` where the x is  unique number that starts at 1 and will count upward each time you execute the request to make sure the value us unique. Each request will create a single user group, separate requests are required if you wish to create multiple user groups.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our cluster API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\}
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server.
Key | Value | Description
------------ | ------------- | -------------
id | blank | This will be the ID of you user group which can be used in the future to assign users or make changes to it. It must be blank as it will be auto created from the name, but any spaces would be removed. If you have spaces in the name then you would need to capture the returned ID to use it to assign users.
name | TechLab-Managed-Cluster-Automation-\{\{envNumber\}\} | This will be the name assigned to your user group and it should be unique. In our case I have set it to the same name as the environment. The variable \{\{envNumber\}\} will be automatically created if it doesn't exist and set to 1. Each time this request is run the variable will increase by one to ensure the name is unique.
ldapGroupNames | blank | This is an LDAP group name. We are not integrating with LDAP so we will leave it blank.
accessRight | multiple | There are 7 access right levels you can assign. You always specify all of them, but only add the environment ID that you wish to assign to each role. In our case we will assign all environments to our newly created environment.
isClusterAdminGroup | false | This determines if the user is given cluster admin rights so they could access CMC and manage the cluster. In our case we will not grant this role.

**Executing the request**
1. Open the User Group request.
2. Verify your additional environment variables envNumber & envID have been set (these are automatically created and set when you executed the Create Environment request).
3. Click on `Send` to execute the request.
4. Check that the request received a `200 OK` response.

![](./images/usergroups/createUgResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

5. Check your new user group now exists in CMC and that all access rights for your matching environment have been granted

![](./images/usergroups/createUgCMC.png)

Congratulations you have just created a new user group via an API call. Now Let's create a user and assign them to this group to grant access to the environment.

# 3. Creating a new user

To create a new user ensure you have completed the creating a new monitoring environment & user group exercises.

## Creating a new user background 

**What is a user?**

You need to add users to your managed cluster to allow them to log into their monitoring environments. User access rights are controled by the User Group they are assigned to.

**When would you create a new user?**

User are created to allow access. User accounts should be specific to individual users. When a new user needs access you should create them their own account and not use shared accounts. By leveraging API calls you could automate this process and allow self-service style registrations.

**How do you create a new user?**

Assuming you are not using any user integration then users can be created using the Cluster Management Console, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create a new user and assign it to our previously created user group via an API call.

## Creating a new user configuration

**Request configuration**

**ATTENTION:** There is currently no user V2 API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the cluster v1 API endpoint users. By making a request to this API endpoint we will create a single new user on our managed cluster assigned to our previously created user group. The id will be in the  format `userx` where the x is  unique number that starts at 1 and will count upward each time you execute the create environment request. Each request will create a single user, separate requests are required if you wish to create multiple users. In our setup it will only support a single user creation per environment as the values do not change.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our cluster API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\}
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information.
Key | Value | Description
------------ | ------------- | -------------
id | user\{\{envNumber\}\} | This will be both the ID and username for the login.
email | test\{\{envNumber\}\}@test\{\{envNumber\}\}.com | This is the email for the user where the environment invitation will be sent. If you wish to use this account and complete the registration, please change this to a real email address you can access. Please note each email address can only be assigned to a single user on a cluster.
firstName | test | The first name of the user
lastName | user | The last name of the user
groups | TechLab-Managed-Cluster-Automation-\{\{envNumber\}\} | The ID of the groups to assign the user to. Please note this must be the IDs and not the names of the groups.

**Executing the request**
1. Open the Create User request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response.

![](./images/users/createUserResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

5. Check your new user group now exists in CMC and that all access rights for your matching environment have been granted

![](./images/users/createUserCMC.png)

Congratulations you have just created a new user via an API call. Now Let's create an environment API token so we can automate our OneAgent installation.

# 4. Creating a dynatrace environment API token

To create a dynatrace environment API token ensure you have completed the creating a new monitoring environment exercise.

## Creating a dynatrace environment API token background 

**What is a dynatrace environment API token?**

To get authenticated to use the Dynatrace API, you need a valid API token. Access to the API is fine-grained, meaning that you also need the proper permissions assigned to the token. In our example we want to automatically install the OneAgent on an ec2 host, so we need to create a token with the InstallerDownload permission.

**When would you create a dynatrace environment API token?**

An environment API token is required when you wish to access the dynatace APIs for a given environment. You can assign multiple permissions to a single token, or you can generate several tokens, each with different access levels and use them accordingly—check your organization's security policies for the best practice.

**How do you create a dynatrace environment API token?**

API tokens can be generated manually inside the settings of the environment UI, but this can be cumbersome if you have a large amount to manage.

In this exercise we will create a new API token with InstallerDownload permissions so we can automate the installation of the OneAgent when we start our new EC2 instance.

## Creating a dynatrace environment API token configuration

**Request configuration**

**ATTENTION:** There is currently no tokens V2 environment API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the environment v1 API endpoint tokens. This is different than the previous endpoints as this is now and environment endpoint where we have previously used cluster endpoints. Environment endpoints apply specifically to the specified environment. You will notice in the URL we are now specifying an environment in the parameter \{\{envID\}\}, this was extracted from the response when the create environment request was executed. 

By making a request to this API endpoint we will create a new API token on our new environment with the rights to download the OneAgent Installer. For security we will set the token to expire in 4 hours. This means you need to create your ec2 instance within this time.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{envTokenManagementToken\}\} | This provides our environment API token to authenticate the request. The token is stored in the environment variable \{\{envTokenManagementToken\}\} which was created when we ran our Create Environment request.
Content-Type | application/json | The response contains JSON payload

**Body**

The JSON body of the request provides the required information. The body must not provide an ID as it will be automatically assigned by the Dynatrace server.
Key | Value | Description
------------ | ------------- | -------------
name | TechLab-Managed-Cluster-Automation | This will be the name of our token. Dynatrace doesn't enforce unique token names. You can create multiple tokens with the same name. Be sure to provide a meaningful name for each token you generate. Proper token naming helps you to efficiently manage your tokens and perhaps delete them when they're no longer needed.
scopes | InstallerDownload | This is the permissions the token will hold. In our case we only require the installer download. For a full list of permissions see [token permission](https://www.dynatrace.com/support/help/dynatrace-api/basics/dynatrace-api-authentication#token-permissions)
expiresIn | 4 HOURS | This is an optional parameter that will cause the token to expire after a given time. It is strongly recommended to rotate your tokens frequently and this parameter can help you with that, but be careful when using integrations. If not set then the token never expires.

**Tests**

This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script parses the JSON response body for the new API token and sets it as environment variable paasToken so we can use it in the Launch AWS easyTravel Instances request to auto deploy the OneAgent.

**Executing the request**
1. Open the Create Installer Token request.
2. Click on `Send` to execute the request.
3. Check that the request received a `201 Created` response.

![](./images/envtoken/createInstTokResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set, verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

5. Check that the paasToken environment variable has been set in postman

![](./images/envtoken/createInstTokVars.png)

Congratulations you have just created a new API token with installer download permission via an API call. Now Let's launch some EC2 instances and automatically deploy the OneAgent.

# 5. Get AWS AMI ID

As this is not a dynatrace API call and is purely to get an AMI ID we will not cover this in as much detail. For more information on it please see the [aws documentation](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeImages.html). This request polls the AWS API to get the latest Ubuntu AMI in your region, it is filtered to the Ubuntu Cloud Account. This way when you launch an ec2 instance in the next step it will use this AMI to ensure you are on an up-to-date version. When you send the request the AMI ID will be stored as environment variable ImageId.

**Executing the request**
1. Open the Get AWS AMI ID request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response.

![](./images/ami/getAMIResp.png)

    If you get a 401 error check the value of your accessKeyID, secretAccessKey and region environment variables. Ensure both the initial and current values are set and the same. If they are verify you have added the correct roles in IAM.

5. Check that the ImageId environment variable has been set in postman

![](./images/ami/getAMIVars.png)

Now that we have the latest Ubunntu image AMI we can launch our ec2 instances.

# 6. Launching easyTravel ec2 Instances and auto deploying the OneAgent

To launch the easytravel ec2 instances ensure you have created your environment, installer token and got the AWS AMI for the latest Ubuntu release. 
As this is not a dynatrace API call and is used to start some ec2 we will not cover this in as much detail, but we will cove how it auto deploys the OneAgent. For more information on it please see the [aws documentation](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RunInstances.html)

## Launching easyTravel ec2 Instances and auto deploying the OneAgent 

**How do you launch an easyTravel ec2 Instances and auto deploying the OneAgent?**

We will leverage the aws RunInstances API call to start 2 instances of the latest Ubuntu AMI that we gathered in the previous call. This will be a blank Ubuntu image so we will leverage [AWS UserData](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) to install and configure both easyTravel and the OneAgent. The UserData is sent as base64-encoded text, but if you want to see the commands that are executed I have provided a sample script [here](./aws/userData.txt). This sample is not exactly the same as the Postman execution as we dynamically set some values in the postman execution we will pass the correct dynatrace environment and API token details and we will set a host group. We will execute this request twice, the first will create an instance with the host group production and the second will create another instance with the host group test.

## Launching easyTravel ec2 Instances and auto deploying the OneAgent configuration

**Request configuration**

**ATTENTION:** There are some values that are unique to your ec2 account that you will have to set for this request to work.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the aws API endpoint to execute the action runInstances. This API call will start an instance of the latest Ubuntu image from the AMI that was gathered in the previous step.

**Parameters**

There are a lot of Parameters in this request, but I will only cover the ones relevant to dynatrace or that you may need to change for your environment.

You may need to set the following Parameters. Determine if you need to set them by reading the table below. If you do need to set them then add the value in postman and ensure you tick the box to the left of the key name to enable it. 
Key | Value | Requirement | Description
------------ | ------------- | ------------- | -------------
KeyName | The name of your key pair | Optional | The name of the key pair used to connect to your instance. You can create a key pair using [CreateKeyPair](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateKeyPair.html). If you do not specify a key pair, you can't connect to the instance. In our use case there is no direct requirement to access the environment, but you may find it useful if you want to perform troubleshooting or use the instance in the future for other things.
SecurityGroupId | The ID of your security group | Optional | If you don't specify a security group ID, aws use the default security group. For more information, see [Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html). Depending on your setup this may be required. A security group acts as a virtual firewall for your instance to control incoming and outgoing traffic so you may require a specific one to allow the correct access to and from your instance. At a minimum you will need inbound on 9080 & 8079 to access the easyTravel & easyTravel Angular front ends and outboun either 443 for direct OneAgent communication or 9999 via an ActiveGate. 

The secret sauce here that you should be aware of is [AWS UserData](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html). When we launch this request we supply a set of commands to be executed on the ec2 instance at start up. The userdata is preconfigured, do not make changes or it may no longer work. In our case these commands will do the following;
1. Updates the package lists by running apt-get update
2. Installs the required packages
3. Downloads and installs the latest version of easyTravel
4. Makes required config changes to easyTravel including API call to set correct public DNS of instance for the source of the traffic that is generated.
5. Installs the OneAgent for your newly created environment leveraging the API token you created in exercise 5
6. Starts the easyTravel angular app. This means you don't even need to access the environment.

**Pre-request script**
This is part of postman. You can use pre-request scripts in Postman to execute JavaScript before a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/pre-request-scripts/)
In our case the script sets the correct host group when requests are executed and updates the userdata to be sent to aws.

**ATTENTION:** When we create these hosts they are set to auto terminate after 8 hours. If you do not want them to auto terminate then please comment out lines 34 & 35. The host is also set to auto terminate on shutdown so keep in mind even if you comment this line out the host would still terminate if you shut it down.

To change the shutdown behaviour of an instance using the console (only after you have started your instance)

1. Open the Amazon EC2 console at `https://console.aws.amazon.com/ec2/`.
2. In the navigation pane, choose Instances.
3. Select the instance, and choose Actions, Instance Settings, Change Shutdown Behaviour. The current behaviour is already selected.
4. To change the behaviour, select an option from the Shutdown behaviour list, and then choose Apply.
![](./images/runinstances/shutdown_behavior_dialog.png)


**Executing the request**
This request is designed to start 2 instance of easyTravel, one with the host group production and the second with the host group test. We will execute the request twice to achieve this. No changes are required as they are done automatically
1. Open the Launch AWS easyTravel Instances request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response and the Production instance name tag.

![](./images/runinstances/runInstancesRespProd.png)

    If you get a 401 error check the value of your accessKeyID, secretAccessKey and region environment variables. Ensure both the initial and current values are set and the same. If they are verify you have added the correct roles in IAM.

5. Execute the request again to start the second instance.
6. Check that the request received a `200 OK` response and the Test instance name tag.

![](./images/runinstances/runInstancesRespTest.png)

    If you get a 401 error check the value of your accessKeyID, secretAccessKey and region environment variables. Ensure both the initial and current values are set and the same. If they are verify you have added the correct roles in IAM.

7. Inside your AWS console check the instance have been created

![](./images/runinstances/runInstancesEC2.png)

8. Check the deployment status in dynatrace to see if your hosts are now monitored. Please note this can take 5-10 mins. 

![](./images/runinstances/runInstancesDT.png)

Congratulations you have just created 2 new ec2 instances and auto deployed the OneAgent.

**Troubleshooting**
If after 10 mins your hosts have not appeared in dynatrace check the following.
1. In the AWS EC2 console select one of your instances and click on Actions > Instance settings > View/Change User Data.
    * Verify the userdata has not been corrupted
    * Check the values for your managed environment URL add API token in the wget command
2. Log on to your instance and view log /var/log/cloud-init-output.log to check for errors in downloading, installing or connection of your OneAgent. You can do this by executing `Tail -f /var/log/cloud-init-output.log`. Please note to access the instance you will need to have specified KeyName when you created the instance. 

# 7. Deleting a user

To delete a user ensure you have completed the creating a new user exercise.

## Deleting a user background 

**When would you delete user?**

User are created to allow access. If a user leaves a company or does no longer require access then their account should be deleted.

**How do you delete a user?**

Assuming you are not using any user integration then users can be delete using the Cluster Management Console, but this can be cumbersome if you have a large amount to manage.

In this exercise we will delete the previously created user via an API call. In a real-world scenario this call could be automatically trigger if a user left the company.

## Deleting a user configuration

**Request configuration**

**ATTENTION:** There is currently no user V2 API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Delete request that leverages the cluster v1 API endpoint users. By making a request to this API endpoint we will delete the previously created user. We will supply the ID of the previously created user in the URL.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our cluster API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\}
Content-Type | application/json | The response contains JSON payload

**Executing the request**
1. Open the Delete User request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response.

![](./images/users/deleteUserResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

4. Check your new user group no longer exists in CMC.

Congratulations you have just deleted a  user via an API call. Now Let's delete the User Group. In real life you probably wouldn't have to delete the UG if a user left as they are not user specific.

# 8. Deleting a user group

To delete a user group ensure you have completed the creating a new user group exercise.

## Deleting a user group background 

**When would you delete a user group group?**

User groups are created to control access. You would delete a user group if it is no longer required. This could be if all environments assigned to it have been deleted.

**How do you delete a user group?**

Assuming you are not using any user integration then user groups can be deleted using the Cluster Management Console, but this can be cumbersome if you have a large amount to manage.

In this exercise we will delete the previously created user group via an API call.

## Deleting a user group configuration

**Request configuration**

**ATTENTION:** There is currently no user group V2 API so we will use the V1 API. In the future we will migrate this guide to V2 when it becomes available.

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Delete request that leverages the cluster v1 API endpoint groups. By making a request to this API endpoint we will delete the previously created user group. We will supply the ID of the previously created user group in the URL.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our cluster API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\}
Content-Type | application/json | The response contains JSON payload

**Executing the request**
1. Open the Delete User Group request.
2. Click on `Send` to execute the request.
3. Check that the request received a `200 OK` response.

![](./images/usergroups/deleteUgResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

4. Check your new user group no longer exists in CMC.

Congratulations you have just deleted a  user group via an API call. Now Let's disable our previously created environment.

# 9. Disable an environment

To disable an environment ensure you have completed the creating an environment exercise.

## Disable an environment background 

**When would you disable an environment?**

You must first disable an environment if you wish to delete it.

**How do you disable an environment?**

Environments can be disabled using the Cluster Management Console, but this can be cumbersome if you have a large amount to manage.

In this exercise we will disable the previously created environment via an API call.

## Disable an environment configuration

**Request configuration**

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Put request that leverages the cluster v2 API endpoint environments. By making a request to this API endpoint we will disable the previously created environment. We will supply the ID of the previously created environment in the URL.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our cluster API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\}
Content-Type | application/json | The response contains JSON payload

**Body**
For this request we will supply the same Body as the create environment request, but with the state set as `DISABLED` this time.

**Executing the request**
1. Open the Disable Environment request.
2. Click on `Send` to execute the request.
3. Check that the request received a `204 No Content` response.

![](./images/monitoringenvironments/disableEnvResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

Congratulations you have just disabled an environment via an API call. Now Let's delete it.

# 10. Delete an environment

To delete an environment ensure you have completed the disable an environment exercise.

## Delete an environment background 

**When would you delete an environment?**

You would delete an environment when both the environment and historical data are no longer required.

**How do you delete an environment?**

Environments can be deleted using the Cluster Management Console, but this can be cumbersome if you have a large amount to manage.

In this exercise we will delete the previously created environment via an API call.

## Delete an environment configuration

**Request configuration**

Let's have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Delete request that leverages the cluster v2 API endpoint environments. By making a request to this API endpoint we will delete the previously disabled environment. We will supply the ID of the previously disabled environment in the URL.

**Headers**

We will supply 2 headers in this request;
Key | Value | Description
------------ | ------------- | -------------
Authorization | Api-Token \{\{dtAPI\}\} | This provides our cluster API token to authenticate the request. The token is stored in the environment variable \{\{dtAPI\}\}
Content-Type | application/json | The response contains JSON payload

**Tests**
To clean up postman so you can run through these exercises in the future all the environment variables that were dynamically created are deleted.

**Executing the request**
1. Open the Delete Environment request.
2. Click on `Send` to execute the request.
3. Check that the request received a `204 No Content` response.

![](./images/monitoringenvironments/deleteEnvResp.png)

    If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.

    If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.

Congratulations you have just deleted an environment via an API call.

We hope you have found this course useful.