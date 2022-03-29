# configure amg
## Prerequisite
AMG requires AWS SSO enabled in your account. AWS SSO is used as the authentication provider to sign into the AMG workspace.

Follow the steps below to enable AWS SSO in your account
* Sign in to the AWS Management Console with your AWS Organizations management account credentials.
* Open the AWS SSO console.
* Choose Enable AWS SSO.

If you have not yet set up AWS Organizations, you will be prompted to create an organization. Choose Create AWS organization to complete this process.

Now go ahead and create a new AWS SSO user that we will use to provide access to the AMG workspace later.

## Create AMG workspace
Go to the AMG console and provide a workspace name as shown below
![image](https://user-images.githubusercontent.com/97269758/160515632-e009d827-9078-4b79-8c29-583378a8ec51.png)
Choose Service managed in the Configure Settings page and click Next. Choosing this option will allow the wizard to automatically provision the permissions for you based on the AWS services we will choose later on.

In the Service managed permission settings screen, you can choose to configure Grafana to monitor resources in the same account where you are creating the workspace or allow Grafana to reach into multiple AWS accounts by choosing the Organization option and providing the necessary OU IDs.
![image](https://user-images.githubusercontent.com/97269758/160515690-e73a6f28-ffec-4e33-9550-0486e95d1212.png)

We will simply leave the option to Current account and select all the Data sources and the Notification channels. Click Next
![image](https://user-images.githubusercontent.com/97269758/160515751-1b7c132c-7502-4542-985b-6a60c64ab75e.png)

In the Review screen, take a look at the options and click on Create workspace
## Add Users
Once the AMG workspace turns to ACTIVE, click on Assign user and select the SSO user created in previously. Click Assign user
![image](https://user-images.githubusercontent.com/97269758/160515876-358e3c91-9e62-4f46-959b-90f661ebf902.png)

By default, all newly assigned users are added as Viewers that only provides read-only permissions on Grafana. To make the user as Administrator, select the user under Users and select Make admin. Now you should see that the user is an Administrator.
![image](https://user-images.githubusercontent.com/97269758/160515905-4f1744c1-125b-4fd1-b423-59f94a65013b.png)

## Login to AMG workspace
Click on the Grafana workspace URL in the Summary section
![image](https://user-images.githubusercontent.com/97269758/160515952-32fb2fb4-0b78-4b19-87b3-a641f757907f.png)

This will take you to the AWS SSO login screen, where you can provide the UserId and Password that you created as part of prerequisites.
![image](https://user-images.githubusercontent.com/97269758/160515988-94635310-0dfd-45b6-aee0-94e203fee8a6.png)

## Configure AMP data source
Select AWS services from the AWS logo on the left navigation bar, which will take you to the screen as shown below showing all the AWS data sources available for you to choose from.
![image](https://user-images.githubusercontent.com/97269758/160516046-bb3a3a6e-281f-4b21-87c3-e46510e50ad3.png)

Select Prometheus from the list, select the AWS Region where you created the AMP workspace. This will automatically populate the AMP workspaces available in that Region as shown below.
![image](https://user-images.githubusercontent.com/97269758/160516160-8b7a0f0e-f525-4ede-a997-f0f797e06a0a.png)

Simply select the AMP workspace from the list and click Add data sources. Once added you will able to see that the AMP data source is authenticated through SigV4 protocol. Grafana (7.3.5 and above) has the AWS SigV4 proxy built-in as a plugin which makes this possible.
![image](https://user-images.githubusercontent.com/97269758/160516217-c1f77567-b70c-45f4-ac29-5692c5d24c19.png)

Query Metrics
In this section we will be importing a public Grafana dashboard that allows us to visualize metrics from a Kubernetes environment.

Go to the plus sign on the left navigation bar and select Import.

![image](https://user-images.githubusercontent.com/97269758/160516250-9433c136-4e8c-4136-9cc2-f1a988cdd28a.png)


In the Import screen, type 3119 in Import via grafana.com textbox and click Load

Select the AMP data source in the drop down at the bottom and click on Import


Once complete, you will be able to see the Grafana dashboard showing metrics from the EKS cluster through AMP data source as shown below.

![image](https://user-images.githubusercontent.com/97269758/160516326-2b80430d-dcbd-4c36-8545-7e151cc862c5.png)

You can also create your own custom dashboard using PromQL by creating a custom dashboard and adding a panel connecting AMP as the data source.
