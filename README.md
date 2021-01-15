# On-premises Data Gateway Best Practises
A collection of best practises for the on-premises data gateway used by the Power Platform and Power BI

## What is the Gateway?
The on-premises data gateway acts as a bridge to provide quick and secure data transfer between on-premises data (data that isn't in the cloud) and several Microsoft cloud services. These cloud services include Power BI, Power Apps, Power Automate, Azure Analysis Services, and Azure Logic Apps. By using a gateway, organizations can keep databases and other data sources on their on-premises networks, yet securely use that on-premises data in cloud services. **The same on-premises data gateway can be used for all services.**

![Gateway used by different services](images/Picture1.png)

In this document we will focus on the best practises for the on-premises data gateway. We focus on highlighting these pratices for the Power Platform and Power BI. More information on how the on-premises data gateway works, check out the documentation for the different services:
* [Gernal Content](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-onprem)
* [Power BI](https://docs.microsoft.com/en-us/power-bi/connect-data/service-gateway-onprem)
* [Power Apps](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/gateway-reference)
* [Power Automate](https://docs.microsoft.com/en-us/power-automate/gateway-reference)
* [Logic Apps](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install)
* [Azure Analysis Services](https://docs.microsoft.com/en-us/azure/analysis-services/analysis-services-gateway )

### What is the difference between the on-premises data gateway and the Azure VPN gateway and the Application gateway?

Within Microsoft, there are three different gateways: The on-premises data gateway, the [VPN gateway](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways) and the [Application gateway](https://docs.microsoft.com/en-us/azure/application-gateway/overview). All gateways are slightly different and designed for different solutions. 
In this document we will focus on the on-premises gateway. This gateway is used by services on the Power Plarform, Power BI, Logic Apps and Azure Analysis Services. You can use the VPN gateway when you are connecting your on-premises data directly to Azure. The Application gateway is used for applications with a public facing browser, to create a safe connection between the internet and Azure. 

### Can the VPN gateway and the on premises gateway run on the same machine?
Yes, both gateways can run on the same machine, but it is not recommended. Both applications will probably transfer a lot of data. It is also harder to troubleshoot any of the two gateways when both are running on the same machine.

## When to use a on-premises data gateway?
On-premises data gateways are required when any of the above-mentioned services need to connect to data that has any of the following conditions:
* On-premises data sources
* Data sources/platform-as-a-service solutions which reside in a virtual machine
* Cloud data sources which reside within a virtual network
* Data sources that are in the same query/dataflow or dataset as datasources that require an on-premises data gateway
* When the [Web.Page()](https://docs.microsoft.com/en-us/powerquery-m/web-page) function is used in a query

A on-premises data gateway is however **not necessary** to access:
* Platform-as-a-service solutions in the cloud *(public ports)*
* Software-as-a-service solutions *(Salesforce/Google Analytics)*

### What is the best way to combine data from the cloud and data that requires an on-premises data gateway?
If you are using dataflows and are using both cloud data sources and data sources that require an on-premises data gateway, it is recommended to first create a seperate dataflow to load data from on-prem into the cloud (into a Data Lake thorugh analytical dataflows). For more information, look at these [best practises for re-using dataflows](https://docs.microsoft.com/en-us/power-query/dataflows/best-practices-reusing-dataflows). Now, when you want to join the dataflow with the cloud data source, the join will happen in the cloud and not on the gateway. This can help with taking off load of the gateway. 

![Gateway used by different services](images/Picture2.png)

### What types of queries exist in every application?
Across the products, there are different query types. Understanding different query types and their performance inmpact is useful for choosing the right gateway machin and while monitoring performance od the gateway. Types of queries supported by product:
* **Power BI Datsets**: Import, Direct Query, Live *(via Analysis Services)*
* **Power BI Dataflows**: Import
* **Power Apps**: Direct Query
* **Power Platform Dataflows**: Import
* **Power Automete**: Direct Query
* **Azure Analysis Services**: Import, Live 

## Types of Gateway
There are two different types of the on-premises data gateway.
* Personal gateway
* Standard gateway (previously know as enterprise gateway)

The personal mode is **only avaiable in Power BI**. In all other services, you can only use the standard gateway. The main  difference between the personal gateway and the standard gateway, is that the personal gateway can only be used by one individual, whereas the standard gateway can be used by anyone with permission. To learn more, take a look at the [documentation](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-onprem#types-of-gateways).

### Why and when should I use a personal gatway?
At tenant level, the adminstator can control who can install gateways. allowing users to install a personal gateway on their own personal devices, makes it very easy for users to start connecting to on-premises data sources. The personal gateway is fast to set up and it makes it easy to use for prototyping or personal projects in the cloud. For more information on personal gateways, check the [documentation](https://docs.microsoft.com/en-us/power-bi/connect-data/service-gateway-personal-mode).

*Note: If you are usin R or Python scripts in ypur data preperation step in Power Query and you want to publish uour report to the cloud, you can only make use of the personal gateway.*

# Why do I need a personal gateway when using Python or R scripts?
This is because of the security risks of running R and python scripts on a shared platform.

### When do I need the standard gateway?
In all other applications, you can only install the standard gateway. Some Advantages/capabilities from the standard gateway are: 

* Multiple applications (like Power BI and Power Apps) can use the same gateway. 
* The gateway is intended to run on a server instead of a personal computer. 
* Support for Refresh, Direct Query and Live, while personal  has only support for Refresh 
* Privacy levels per data source can be configured. 
* Clustering for high availability and disaster recovery possible 
* Logging for activity and health monitoring 

### Should I use the same gateway cluster with different applications like Power Apps and Power BI?
With the standard gateway it is possible to attach multiple applciation to the same cluster. However, this is not always optimal.
Reasons to seperate the clusters:
* Power BI and Power Apps can have different hardware requirements (CPU or memory optimized)
* Different security roles in Power BI and Power Automate.
* Can be easier for trroubleshooting and management

### What is the difference between gateway application, gateway machine and gateway cluster?
* A **gateway application** is the application which helps woth on-prem onnectivity
* A **gateway machine** is the machine on which the gateway application is installed
* A **gateway cluster** is a logical grouping of gateway machines

![Gateway used by different services](images/gateway.PNG)

### What are the benifits of high availability and load balancing?
If you have only one gateway member, then you have a single point of failure. Therefore, we recommenc having a least 2 gateway members on a cluster with high availability enabled. Addinionally enabling load balancing helps utilize recoucres of both gateway members effectively. 
* **High availability** eliminates having a simple point of failure
* **Load balancing** automatically distributes the workload across all gateway members in the cluster

## Should I use different clusters for dev/test and prid environment?
We would recommend using at least two separate clusters:
* 1 cluster for your dev/test environmtent
* 1 cluster for your prod environment

By doing this, anay changes that uou make, like adding a new data source, you can first test on your dev/test cluster before appying it in your production entironment. This will ensure that you production environmentnis not impacted by these changes.

![Gateway used by different services](images/picture4.png)

### Do I need more than two clusters?
In principle, you do not need more than two clusters. If you need to scale, you can add new gateways to the same cluster or upgrade to bigger machines. There might be cases where you want to have more than two clusters:
* when you have a dev/test and prod cluster
* you want to have seperate clusters for different applications for reasons described earlier

## How to choose your gateway machine
The gateway is not only used for data connectivity. In most cases, the query results from the data sources atre processed on the gateway machine. Therefore, a gateway machine requires the resources to processthese data merges and transformation succesfully.

When you need to choose a server size, there are two aspects that are most important: Memory and CPU. It is important that the size of your gateway machine is big enough to handle the workload at its busiest time. For more information, see the "monitoring your gateway".

## How to determine the ideal machine size?
As long as your gateway can handle the peak workload, you are good. Howwer, to plan your machine sizes, you might want to get some insights beforehand. Different query types have different impact on your gateway machine:
* **Refresh** is usually more heavy on memory
Refreshes usually run longer and utilize more memory to accomodate for the data transformations and merges.
* **Direct Query** and **Live** are usually more heavy on CPU
Direct queries and the live connection are mostly CPU heavy. In most cases, direct queries are short and fetch smaller volumes of data. Based on the small data volume, Direct Query is normally not memory intense, but these queries get executed manu tomes on demand, which can be CPU intens.

## Gateway security
The gateway offers multiple securty roles which differ across the services.
* **Power BI**: Admin
* **Power Apps and Power Autmate**: Admin, Can Use, Can Use + Share

The Gateway Admin role is available in for all services. Users in this role can update, change configurations, add other admins to the gateway.

A gateway admin can also create data sources (in Power BI) or connections (in Power Apps and Power Automate) and also share these to users, who can use them in reports, dataflows, Apps or flows.

Additionally, in Power Apps and Power Automate, gateway admins can share gateways to users under the “Can Use” and “Can Use + Share” roles.  Users in these roles do not have other admin capabilities but are allowed to create connections on the gateway. Users in Can Use + Share roles can additionally share the gateway with other users.

### Can you restrics acess to specific connectors via the gateway in Power Apps and Power Automate?
While sharing a gateway, you also share it for specific connectors. For instance, if a gateway was shared for only SQL Server, then the user can create only SQL connections using that gateway. This will ensure users don’t use Power platform with unauthorized data sources on your network.

### Where can I manage my gateways?
You can manage gateways in each service. For instance, the Gateway page in Power Apps and Power Automate while the Manage Gateways page in Power BI helps you manage gateways.
Additionally you could also manage gateways in Power Platform admin center.

### Where can I see all gateways installed in a tenant? 
As a Tenant admin and a service admin(PBI and Power platform) , you can see all gateways installed in your tenant and used across all services in the Power Platform admin center. Please make sure the tenant administration toggle is turned on. By default, you would see only gateways you are an administrator of.

### How many admins should a gateway have?
We recommend you add a security group as the admin of a gateway containing more than one admin so the gateway isn’t orphaned when the admin leaves the company. However, if a gateway is orphaned, the tenant admin will be able to add additional admins from the power platform admin center.

## Installation and Configuration and Updating the Gateway

### Where should I install my gateway?
A gateway needs to be installed on a machine that is running Windows and need to have access to the data source. It is recommended that your gateway machine is physically close to the data source.

### Does the physical location of the machine where I install the gateway need to match the region chosen while installing the gateway? 
The region chosen while installing your gateway is the region where all metadata related to your gateway is stored. The region you choose may impact the service you may be able to use the gateway with. 
* **Power BI**: Gateway needs to be installed in the same region as your tenant’s Power BI home region
* **Power Apps and Power Automate**: Gateway needs to be in the same region as the environment

### What if my Power BI home region is in the US and my PowerApps environment in Europe? 
In this scenario, you would need to create two separate gateway clusters, one cluster that is deployed in the same region as the Power BI home region and the other one in the same region in PowerApps. 

### What if my Power BI home region is in the US and my PowerApps environment in Europe? 
In this scenario, you would need to create two separate gateway clusters, one cluster that is deployed in the same region as the Power BI home region and the other one in the same region in PowerApps. 

### What if I have two PowerApps environments in two different regions? 
In this scenario, you would also need to create two separate gateway clusters, as the application can only communicate with a gateway cluster that is deployed in the same region as the environment. 

### What are the requiremets to install the gateway succesfully?
The full list or requirements you can find in the [documentation](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-install)

### Can I install my gateways in an automated way?
Yes, this can be done with PowerShell. For more information, see the [documentation](https://docs.microsoft.com/en-us/powershell/module/datagateway/get-datagatewayinstaller?view=datagateway-ps).

### Is internet connectivity important for the server? 
Yes, internet connectivity is very important for the server. We therefore recommend using a wired machine as the gateway server over a wireless one. 

### What is the gateway service account?
When the data gateway is installed, a *PBIgwService* local account is automatically configured on the local machine. The gateway service then runs in the context of this account and is thus granted *“Log On As A Service”* permissions during installation.  

Additionally service accounts also need the *SeImpersonatePrivilege*. All service accounts have this by default (see screenshot):

![Gateway used by different services](images/picture5.png)

### Can I change the gateway service account?
Some organizations may want to change this service account to run instead as a domain service account: 

![Gateway used by different services](images/picture7.png)

When changing the service account, make sure the service account has:
* Full control permissions for the installation location, *Default is located at: C:\Program Files\On-premises data gateway*, as well as the location where logs are retained *if modified from the default path*  
* Local securti policy
* User rights assigments: a) impersonate a client after authentification. b) log on as a service

More information on gow to change te service account can be found [here](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-service-account).

### Who can install the gateway?
 A person that wants to install the standard gateway needs to have access to a server and admin privileges on the server to install it. 

 ### Can I control who can install the standard gateway in my organization?
 As a tentant admin or service admin (both Power BI and Power Platform), you can manages standardc gateway installers in your organization. For documentation on how to control who can install the gateway, check this [page](https://docs.microsoft.com/en-us/power-platform/admin/onpremises-data-gateway-management#manage-gateway-installers).

 ### As a tenant or service admin, can I also restrict personal gateway installation in my company?
 Yes, you can manage who can install personal gateway using PowerShell. For more details, see the documentation [here](https://docs.microsoft.com/en-us/powershell/module/datagateway/set-datagatewayinstaller?view=datagateway-ps).

### Is it possible to do a silent install of the standard data gateway?
Yes, it possible yo do silent install of the standard data gateway, using [these](https://www.powershellgallery.com/packages/DataGateway/3000.37.39/Content/Samples%5CInstallAndAddDataGateway-Sample.ps1) PowerSjell cmdlets.

 ## Updating the gateway

 ### How ofter are the new gateway releases?
We release a new version of the gateway every month and the last months of gateway versions are supported. While we test each of the gateway versions thoroughly, we do recommend that you update your test gateway servers first and test your critical scenarios before updating production. Also be aware that the most recent features available in Power BI desktop may not work on the gateway if gateways aren’t updated. 

### Do I need to manually update my gateway?
yes, you need to manually update the gateway.

### How do I update my gateway member in the cluster?
Please follow the following steps while updating a gateway cluster with two or more members:
1. Diasable one gateway member.
2. Update the gateway member to the new version.
3. Enable the updated gateway member.
4. Repeat step 1-3 untill all gateway members are updated

Disabeling a gateway makes sure the load balancer does not try to excecute queries on the member you are updating, hence reducing delays and failures.

### Can gateway members in a cluster be on different versions?
Yes, but we recommend that you update gateway members one after the other without a long lag. This will reduce sporadic failures as a query may succees on one gateway member, but not on the other, based on its version.

### Can I updated the gateway configuration files?
You can update gateway configurations For more information of the [configuration files](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-proxy).

## Naming Conventions

### What are recommended naming conventions for the gateway cluster and gateway member names?
* It is helpfull to have some variation of "Data Gateway" or "GW" in the name.
This will be usefull for troubleshooting, particularay of the data gateway resides on a multi-purpose machine.
* Include specific the specific purpose in the name, if tthe gateway os dedicated to supporting certain types of operations.
* Indicate in the gateway clusters in in a dev/test or production environment
* Alligm member names closely to the cluster name

Example:
* DataGW-payroll-dev-cluster
* DataGW-payoll-dev-member1
* DataGW-payroll-dev-member2

Additionally, we have other attributes like department, description, and contact information which provide context to gateway admins and users. For instance while setting a data refresh schedule, the Power BI dataset/dataflow owner can see the gateway name, department, description, and contact information which could help them decide which gateway they should select. 

### What are recommended data source names?
The data source name should equate as closely as possible to the source itself or the any logical term for its purpose, to avoid any ambiguity. If applicable, other clarifications (such as Dev/Test/Prod) can be included in the data source name, if non-production and production environments are set up on a single gateway rather than multiple gateways. The data source name may be modified after it is created . However, the type, server, and database may not be changed after it is initially configured. 

## Management 

### How do I manage data sources in Power BI?
In Power BI Desktop, you can just connect to  on- premises sources, without a gateway. But when you publish your report to the Power BI Service, you need to use a gateway for the Power BI service to be able to connect to the on-premises data source.

![Gateway used by different services](images/picture8.png)

In Power BI, a gateway admin is required to create data sources on the gateway and add data source users. As long as a data set owner has access to the data source, they can bind the one or more datasets to the data source. Similary, a dataflow author/owner can also connect to a gateway data source as long he/she has been added as a valid data source user.

For more information on how to connect a data source to the gateway cluster in Power BI, check the [documentation](https://docs.microsoft.com/en-us/power-bi/connect-data/service-gateway-data-sources).

### If I create a data source, can it be used in both dataflows and datasets?
Yes, as long as a user has access to the data source, they can use it both for Power BI datasets and dataflows.

### What requirements does a data source connection has?
It is important that the database attribute in the data connection is the same as that you use in the Power BI connection. This means that you need to create a new connection to the gateway for every database that is used in the server. This would be similar for every path to a local folder or file or any database in SAP.  

### Can I add data sources with PowerShell?
No, but it is possible to perform data source management using [Power BI REST API's (Gateways)](https://docs.microsoft.com/en-us/rest/api/power-bi/gateways).

### How do I manage data sources fotr gateways in Power Apps? 
Connecting data sources from the power platform is different than in Power BI. If you want to connect a on-premises data source in Power Apps dataflows, or Apps you can do this without registering the connection first. If you can connect to a certain data source is determined by your rights to the gateway. If you are an admin, you can connect to all sources. If you have “can use” right, the admin has determined for you which data sources you can connect to. As an admin of the gateway, I can specify this permission when adding new users to the gateway.

### How do I manage data sources fotr gateways in Power Automate? 
For Power Automate, adding data sources to the gateway is a similar process as in PowerApps 

## Security

### Why and how do I use the gateway recovery key?
While installing the gateway, you need to provide a recovery key and this key is used fot encrypting credentials for your on-premises data sources. For security reasons, Microsoft does not have access to this key so make sure you store thsi key safely. 

You will require this key when you want to move or recover the gateway on a different machine. You will also need this key when you want to add new gateway members to the gateway cluster.

Microsoft is not providing any restriction on choosing your key. It is therefore important that you create a difficult key following standard practices, like:
* Use combination of uppercase and lowercase letters, symbols and numbers.
* The more characters you use, the saver.
* Never use any “real” words in any language 

Next is to store the ket in a safe location. Never write your key down. It is also important to update your key regulary based on your organizations policy. [Here](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-recovery-key) is how you can change your recovery ker.

Note: Updating the recovery key re-encrypts the Power BI credentials but credentials for the other applications, end users are required to re-enter credentials so they can be re-encrypted using the new key. 

### How are my credentials handled by the gateway?
Within the data source configuration in the Power BI service, an authentication method, username, and password need to be specified for access to the data source. These credentials are encrypted (using asymmetric encryption) on the data gateway, which also handles decrypting those credentials when the data source is accessed. Whenever possible, a domain-based service account should be used for data source credentials rather than an individual user account. User accounts become problematic when password changes occur or upon terminations and transfers. The service account should have a password that does not expire (or is managed). Using a service account will also be helpful for logging and auditing purposes. This account typically requires only read permissions on the data source. 

One exception is when specifying an Analysis Services data source: Analysis Services requires the account to have administrator privileges on the instance. With Analysis Services, the account for the user interacting with the report is sent via the EffectiveUserName connection string property in the form of an e-mail address. The domain in this e-mail address must match what is in the Azure Active Directory associated with Power BI or match a user principal name mapping.

For DirectQuery, single sign-on may be configured so that user permissions in the source system are honored. For import datasets, the credentials are inherited from the data source as configured in the data gateway. This is extremely important point because it may impact permissions on data retrieved from source systems. As an alternative, the dataset owner’s identity may be used for import queries if the data source is configured to allow it.

The SSO via Kerberos setting is very helpful for re-using one single data source for a wide number of people/teams who would retrieve different data based on their underlying permissions. This avoids having to set up the same data source multiple teams, with different credentials, and different user populations assigned.

## What are the firewalls and ports used by the gateway?
The On-Premises Data Gateway communicates with the Azure Service Bus via TCP on outbound ports 443 by default. It can also use 5671, 5672, or 9350 – 9354. With TCP, the On-Premises Data Gateway communicates with the Azure Service Bus with a combination of IP addresses and domain names   

After the gateway is installed and registered, the only required ports and IP addresses are those needed by Service Bus, as described for servicebus.windows.net in the preceding table. You can get the list of required ports by performing the Network ports test periodically in the gateway app. You can also force the gateway to communicate using HTTPS.

For more information, see the [documentation](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-communication).

### What communication protocols are used?
As of mid-2019, the default for new gateway installations changed to be HTTPS. This was done to improve customer experience, security and reduce connectivity failures for new installations . There are two options for setting the gateway communication protocol:
* HTTPS (the default). When enabled, an additional layer of SSL security is added on top of the transport security. This should have only a nominal performance impact. 
* Direct TCP. With this method, the gateway uses TLS (Transport Layer Security) to communicate with the Power BI Service. Data is encrypted in transit by the transport protocol which is TLS 1.2.

## Monitoring the gateway
A gateway admin would require visibility into what is running on the gateway to be able to troubleshoot long running queries or proactivity plan for capacity. 


### How do I monitor my gateway
You can monitor the gateway, by following [these](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-performance) instructions.

When monitoring the gateway, we can split the monitoring task into two catregories:
1. Monitoring the Gateway performance and load
2. Investigating a specific long running query

When monitoring the gateway performance, there are multiple metrics we can looka t. We divide these metrics into 4 categories:
* Query Diagnostics
* Hardware Diagnostics
* Network Diagnostics
* Software Diagnostics

### How to monitor query diagnostics?
Query diagnostics are all logs and performance counters that are related to the queries that get executed on the gateway. The three most important query counters are:
* **Number of Failed Queries**
The number of failed queries in a counter of the number of queries that failed. You can map failed queries over time or look at aggregated values. If a lot of queries are failing at the same time for example, this could indicate that there is something wrong on the gateway. While, if just one query fails, there might be something wrong with the query. 
* **Query Count**
The query count is the number of queries that gets executed on the gateway at a given point in time. This metric can help you understand the load on the gateway, but not completely, since query count is not telling you anything about query duration, CPU or memory consumption.

### How to monitor hardware diagnostics?
Query diagnostics are all logs and performance counters that are related to the queries that get executed on the gateway. The three most important query counters are:
* **Gateway application memory usage (working set)** Gateway application memory usage is the memory usage by the gateway application. When configuring the gateway application on your machine, the gateway can be configured to only use a percentage of the total memory available on the machine. When the maximum of the allocated memory  for the gateway gets reached, your queries will start running slow or fail. It is therefor important to monitor this metric and make sure you have enough memory capacity.
* **Mashup memory usage (working set)** Mashup memory usage is the memory usage of the mashup container. Most queries ruse mashup container to excecute. So the number of mashup containers determines the number of queries that can be excecuted in parallel. A working set defines the memory allocated to each container. It is therefore important to monitor the mashup memory usage, When queries grow over time, the container needs to have enough memory allocated to excecute the query.
* **Machine memory usage** System memory usage is the total memory usage of the machine. This metric can be usefull to track if you can increase the application memory usage of the machine. For example, you allocated 50% of the machines memory to the gateway applciation. If you have reached the maximum capacity of the application memory, you have three options:
    1. Reduce the number of queries 
    2. Scale out/up the gateway machines
    3. Allocate more memory from the machien to the gateway application

    For the third scenario, you need to make sure your machine has enough total memory left to allocate more to the gateway applicaiton. You can use this metric to determine this.

* **Gateway CPU usage** Gateway CPU usage is the CPU usage on the gateway machine. CPUs are designed to run safely at 100% CPU utilization. However, when coming close to this limit, this can cause slowness int the execution of your queries. It is therefore important to monitor the CPU usage and add more nodes to the cluster when this is causing slower running queries.

### How to monitor network diagnostics?
Query diagnostics are all logs and performance counters that are related to the queries that get executed on the gateway. The two most important query counters is:
* **Gateway uptime** Gateway uptime is the the time the gateway is active and reachable. It is important that when queries need to be excecuted the gateway is online and can be reached.
* **Network connection** It is important that your gateway has good connection to the internet to avaoid any latency.







### How do I know when to scale up or out the gateway?

Scaling up is adding additional resources to the gateway machine and scaling out is adding additional gateway members to the cluster. 
You would be required to scale up if the gateway machine reaches maximum CPU or memory utilization when you are executing just one query. 

![Gateway used by different services](images/picture9.png)

You would be required to scale up or out when the system is at maximum CPU or memory while multiple queries are being executed at the same time. When load balancing is enabled,  the selection of a gateway member for executing a query is random If the gateway member to which a query is sent is busy, the query is routed over at random to another gateway member in the cluster and this is done until the query can be executed. When all gateway members are busy, then the query fails.

![Gateway used by different services](images/picture10.png)



### QUERY DIAGNOSTICS DETAILS

* **Query Duration**
The query duration is the duration of the query excecution on the gateway. Over time the duration of a certain query can increase. This could be, because the query started processing more data over time, resulting in the longer running query. The use of incremental refresh could solve the long running query.





