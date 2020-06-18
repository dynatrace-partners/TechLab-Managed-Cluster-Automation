**ATTENTION:** This is currently a work in progress. Please check back when the course is complete

![](./images/partner_program.png)

# 1. Overview
Hello and welcome to this Dynatrace Partner TechLab. We are launching these as self-paced training for all our partners. During this session we will focus on automation and complete hands on exercies on how to automate creating a monitoring environment, usergroup and user and auto deploy OneAgent to the new environment.

The goals of this tutorial are;
1. **Environment Management** - Automatically create environments on a managed cluster. Useful for onboarding new customers, training events, creating additional environments etc.
2. **User Group Management** - Automtically create User Groups with the correct access to your freshly created environment. Useful for onboarding new users, training events and user management.
3. **User Management** - Automtically create Users and assign them to the correct user group. Useful for onboarding new users, training events and user management.
4. **API token Management** - Automatically create API tokens with the correct roles. Useful for automated Oneagent deployments, integrations, training events, key rotation etc.
5. **Automated OneAgent Deployment** - Start ec2 instances and automatically install and configure the OneAgent. Useful for fast deployments of the OneAgent and ensuring new resources are implemented. 

First we will run these setps individually so you get a feel for how they operate then we will provide a script that allows you to do it all at once.

## Pre-Requisites
1. You need an AWS account. If you don't have one [get one here](https://aws.amazon.com/)
1. AWS Access key and secret access key for an IAM user with rigths ec2:RunInstances &  ec2:DescribeImages
1. You need a Dynatrace Managed Cluster license. If you don't have one, reach out to your Dynatrace Partner Program Manager (PPM) to be granted a trial license.

Region | Name | Email
------------ | ------------- | -------------
APAC | Dhruv Rajvanshi | [Dhruv.Rajvanshi@dynatrace.com](mailto:Dhruv.Rajvanshi@dynatrace.com)
EMEA | Patricia Jesus Silva | [patricia.jesus.silva@dynatrace.com](mailto:patricia.jesus.silva@dynatrace.com)
LATAM | Luiz Rodrigues | [Luiz.Rodrigues@dynatrace.com](mailto:Luiz.Rodrigues@dynatrace.co)
NORAM | Jamie Mallett | [Jamie.Mallett@dynatrace.com](mailto:Jamie.Mallett@dynatrace.com)

1. You need a Managed Cluster. A single trial node is sufficient. Thic could be run on a VM or in your EC2 account [view specs](https://www.dynatrace.com/support/help/setup-and-configuration/dynatrace-managed/installation/dynatrace-managed-hardware-and-system-requirements/)
1. Optional: ActiveGates - An ActiveGate is not required to complete these exercises but depending on your install and security requirements you may also require a CLuster and/or Environment ActiveGate. For more info see [When do I need to install an ActiveGate?](https://www.dynatrace.com/support/help/setup-and-configuration/dynatrace-activegate/basic-concepts/when-do-i-need-to-install-an-activegate/)
1. [Postman](https://www.postman.com/downloads/) installed
1. You need to clone or copy the content of this GitHub repo to your local disk!

## Preparation

**Amazon Web Services (AWS)**

As we are going to use AWS EC2. We have tested this tutorial on eu-west-1 (Ireland) & ..... . To be on the safe side we suggest you pick one of these regions!
Ensure you have created an IAM user  with rigths ec2:RunInstances &  ec2:DescribeImages 
Login to your AWS account and navigate to IAM
![](./images/preparation/awsconfigure.png)

**Dynatrace**

You will need to have provisioned your dynatrace managed cluster. For installation instructions please follow the [documentation](https://www.dynatrace.com/support/help/setup-and-configuration/dynatrace-managed/installation/set-up-a-cluster/)

Once you have have your cluster provisioned we need a couple of things
1. Your dynatrace managed URL. That should look something like `https://xxxxxx.dynatrace-managed.com`
2. Cluster API token. Inside CMC go to Settings -> API tokens -> Cluster tokens and select Generate token. Grant it the scope of Service Provider API

![](./images/preparation/clusterapi.png)

Expand the token and copy the API key

![](./images/preparation/clusterapitoken.png)

**Postman**

For this tutorial we will leverage postman to make our API calls. If you would like to take these concepts and encorporate them into your dynatrace provisioning process you can choose any tool you like providing it can make API calls. 

Once you have installed Postman we need a couple of things
1. Upload the [postman environment](./postman/TechLab-Managed-Cluster-Automation.postman_environment.json) we have provided.
Inside Postman click on import, select file and upload TechLab-Managed-Cluster-Automation.postman_environment.json
2. Upload the [postman collection](./postman/TechLab-Managed-Cluster-Automation.postman_collection.json) we have provided.
Repeat the same process to import TechLab-Managed-Cluster-Automation.postman_collection.json
3. Configure your postman environment variables

    * Click on the ![](./images/preparation/manageEnvironments.png) to manage your environments

    * Click on the environment name TechLab-Managed-Cluster-Automation

    * Set all the 5 environment variables to your values. Ensure to set both the initial and current values. For dtManaged it should look like 'xxxxxx.dynatrace-managed.com' do not include the 'https://'

![](./images/preparation/environmentVars.png)

Click on update to save your changes

# 2. Creating a new monitoring environment

**ATTENTION:** You must execute these requests in the order they are listed in here as the requests gather response data and create new environment variables that are used in later requests.

To create a new monitoring environment ensure you have set-up your managed cluster, have generated a Cluster API Token and configured Postman.

## Creating a new monitoring environment background 

**What is a monitoring environment?**

Your Dynatrace monitoring environment is where all your Dynatrace performance analysis takes place. Dynatrace OneAgent sends all captured monitoring data to your monitoring environment for analysis. A monitoring environment is analogous to an analysis server that provides all Dynatrace application-performance analysis functionality, including all dashboards, charts, reports and other tools.

**When would you create a new monitoring environment?**

It's common to set up multiple monitoring environments so that related entities can be grouped for discrete analysis. For example, you might set up one monitoring environment to monitor and analyze the performance of your production clusters. You might set up a second environment that's dedicated to the performance of your developers' machines and a third environment for your staging servers.
You might also be running managed as a service for your customers. In that case you would setup separate environments for each customer to ensure their data is segrated.

**How do you create a new monitoring environment?**

Monitorining environments can be created using the Cluster Management Console but this can be cumbersome if you have a large amount to manage.

In this exercise we will create a new monitoring environment and assign the appropriate license limits to it via an API call.

## Creating a new monitoring environment configuration

**Request configuration**

Lets have a look at the configuration of this request so we can understand what will happen when we execute it.

This is a Post request that leverages the cluster v2 API endpoint environments. By making a request to this API enpoint we will create a single new environment on our managed cluster. The environment will be named `TechLab-Managed-Cluster-Automation-x` where the x is  unique number that starts at 1 and will cound upward eachtime your execute the request to make sure the value us unique. Each request will create a single environment, seprate request are required if you wish to create multiple environments.

**Paramaters**

By supplying the paramater createToken=true a token with the permission `Token management` is created when creating a new environment. This token is then returned in the response body. It can be used within the newly created environment to create other tokens for configuring this environment.
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
name | TechLab-Managed-Cluster-Automation-\{\{envNumber\}\} | This will be the name assigned to your environment and it should be unique. This name will be displayed to the users of the environment so in real customer projects ensure to choose and appropriate name. In our case i have set it to the same name as the techLab. The variable \{\{envNumber\}\} will be automatically created if it doesn't exist and set to 1. Each time this request is run the variable will increase by one to ensure teh name is unique.
state | ENABLED | This sets our new environment in the enabled state meaning it will be accessible and immediately ready for use. 
tags | customerA, production | Tags are not visable in CMC but they are via the cluster API. In large clusters with lots of environments proper tag assignments will make management much easier. In our case we will tag our environment as customerA and production, in the future this would allow me to filer for this exact environment if i specified both tags or I could filter for all customerA's environments or all porduction environments.

**Pre-request script**
This is part of postman and not a requirement to create an environment via an API call. You can use pre-request scripts in Postman to execute JavaScript before a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/pre-request-scripts/)
In our case the script checks to see if the environment variable envNumber exists or not. If it doesn't exist \(and it won't the first time you do this exercise\) it creates it and sets the value to 1. If the variable does exist \(i.e. you have run the request before\) then it will increment the value by one. As detailed above this variable is used in our environment name.

**Tests**
This is part of postman and not a requirement to create an environment via an API call. You can use Tests in Postman to execute JavaScript after a request runs. You can find more details [here](https://learning.postman.com/docs/postman/scripts/test-scripts/)
In our case the script parses the JSON reponse body for the ID and API token of our new environment and sets them as environment variables envID and envTokenManagementToken so we can use them 

**Executing the request**
1. Open the Create Environment request.
Verify that you have selected your TechLab-Managed-Cluster-Automation environment then hover over the variable dtManaged and verify both the initial and current values are set, it should look like 'xxxxxx.dynatrace-managed.com' without the `https://`
![](./images/monitoringenvironments/createEnvironment.png)
On the Headers tab hover over \{\{dtAPI\}\} to ensure it has been sent
2. Click on `Send` to execute the request.
3. Check that the request received a 201 Created response and the response body contains the id, name and token for your new environment
![](./images/monitoringenvironments/creteEnvResp.png)
If you get a could not send request error check the value of your dtManaged environment variable and ensure it is in the format of `xxxxxx.dynatrace-managed.com` without the `https://`. Ensure both the initial and current values are set and the same.
If you get a 401 error check the value of your dtAPI environment variable. Ensure both the initial and current values are set and the same. If they are set verify the token is correct in CMC and it has the Service Provider API role. Be careful if your token ends with a = as this can get cut off when copying and pasting.
4. Check your new environment now exists in CMC
5. Access your new environment by clicking on it's name in CMC and then Go to environment
6. Verify your new environment ID and Token have been set as environment variables in postman

Congratulations you have just created a new environment via an API call. Now lets create a usergroup to grant access to this environment.

**ATTENTION:** There is now an Early Adopter for the Environments V2 API. In the future we will migrate this guide to it.
