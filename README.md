# cloudfoundry-azure-deployment

WIP

https://docs.pivotal.io/pivotalcf/2-1/customizing/azure-deploy-terraform.html#download

Creating a Cloudfoundry deployment that is specific for Azure.
This will include CF Cli and Bosh CLI - 

Pre-requisite - 
Part 1

Must have an azure subscription
Must have access to Pivotal or CF account for Marketplace
Must have a cf login with api endpoint.
Must understand Terraform 
Do not place credentials in Terraform TFVARS <--- this is just an example
It would be good to understand CM - like Bosh or Chef or Ansible (even Tower)


Part 2
Create Azure Resource with Terraform
$ terraform init
$ terraform plan -out=plan
$ terraform apply plan

Part 3
To complete the procedures in this topic, you must have access to the output from when you ran terraform apply to create resources for this deployment. You can view this output at any time by running terraform output. You use the values in your Terraform output to configure the BOSH Director tile.


Step 1: Access Ops Manager
Note: For security, Ops Manager v1.7 and later require that you log in using a fully qualified domain name.

In a web browser, navigate to the fully qualified domain name (FQDN) of the BOSH Director. Use the ops_manager_dns value from running terraform output.

When Ops Manager starts for the first time, you must choose one of the following:

Use an Identity Provider: If you use an Identity Provider, an external identity server maintains your user database.
Internal Authentication: If you use Internal Authentication, PCF maintains your user database.
Select authentication

Use an Identity Provider (IdP)
Log in to your IdP console and download the IdP metadata XML. Optionally, if your IdP supports metadata URL, you can copy the metadata URL instead of the XML.

Copy the IdP metadata XML or URL to the Ops Manager Use an Identity Provider log in page. Meta om

Note: The same IdP metadata URL or XML is applied for the BOSH Director. If you use a separate IdP for BOSH, copy the metadata XML or URL from that IdP and enter it into the BOSH IdP Metadata text box in the Ops Manager log in page.

Enter values for the fields listed below. Failure to provide values in these fields results in a 500 error.

SAML admin group: Enter the name of the SAML group that contains all Ops Manager administrators.
SAML groups attribute: Enter the groups attribute tag name with which you configured the SAML server.
Enter your Decryption passphrase. Read the End User License Agreement, and select the checkbox to accept the terms.

Your Ops Manager log in page appears. Enter your username and password. Click Login.

Download your SAML Service Provider metadata (SAML Relying Party metadata) by navigating to the following URLs:

6a. Ops Manager SAML service provider metadata: https://OPS-MAN-FQDN:443/uaa/saml/metadata
6b. BOSH Director SAML service provider metadata: https://BOSH-IP-ADDRESS:8443/saml/metadata
Note: To retrieve your BOSH-IP-ADDRESS, navigate to the Status tab in the BOSH Director. Record the BOSH Director IP address.

Configure your IdP with your SAML Service Provider metadata. Import the Ops Manager SAML provider metadata from Step 6a above to your IdP. If your IdP does not support importing, provide the values below.

Single sign on URL: https://OPS-MAN-FQDN:443/uaa/saml/SSO/alias/OPS-MAN-FQDN
Audience URI (SP Entity ID): https://OP-MAN-FQDN:443/uaa
Name ID: Email Address
SAML authentication requests are always signed
Import the BOSH Director SAML provider metadata from Step 6b to your IdP. If the IdP does not support an import, provide the values below.

Single sign on URL: https://BOSH-IP-ADDRESS:8443/saml/SSO/alias/BOSH-IP-ADDRESS
Audience URI (SP Entity ID): https://BOSH-IP-ADDRESS:8443
Name ID: Email Address
SAML authentication requests are always signed
Return to the BOSH Director tile and continue with the configuration steps below.

Note: For an example of how to configure SAML integration between Ops Manager and your IdP, see the Configuring Active Directory Federation Services as an Identity Provider topic.

Internal Authentication
When redirected to the Internal Authentication page, you must complete the following steps:

Enter a Username, Password, and Password confirmation to create an Admin user.
Enter a Decryption passphrase and the Decryption passphrase confirmation. This passphrase encrypts the Ops Manager datastore, and is not recoverable if lost.
If you are using an HTTP proxy or HTTPS proxy, follow the instructions in the Configuring Proxy Settings for the BOSH CPI topic.
Read the End User License Agreement, and select the checkbox to accept the terms.
Click Setup Authentication.
Om login

Log in to Ops Manager with the Admin username and password that you created in the previous step.

Cf login

Step 2: Azure Config Page
Click the BOSH Director tile.

Azure om tile

Select Azure Config.

Azure config

Complete the following fields with information you obtained in the Preparing to Deploy PCF on Azure topic.

Subscription ID: Enter the ID of your Azure subscription.
Tenant ID: Enter your TENANT_ID.
Application ID: Enter the APPLICATION_ID that you created in the Create an Azure Active Directory Application step of the Preparing to Deploy PCF on Azure topic.
Client Secret: Enter your CLIENT_SECRET.
Complete the following fields:

Resource Group Name: Enter the name of your resource group, which is exported from Terraform as the output pcf_resource_group_name.
BOSH Storage Account Name: Enter the name of your storage account, which is exported from Terraform as the output bosh_root_storage_account.
For Cloud Storage Type, select Use Storage Accounts and enter the wildcard_vm_storage_account output from Terraform. Storage accounts

For Default Security Group, enter the bosh_deployed_vms_security_group_name output from Terraform.

For SSH Public Key, enter the ops_manager_ssh_public_key output from Terraform.

For SSH Private Key, enter the ops_manager_ssh_private_key output from Terraform.

For Azure Environment, select the Azure cloud you want to use. Pivotal recommends Azure Commercial Cloud for most Azure environments.

Note: If you selected Azure Stack (Beta) or Azure Government Cloud as your Azure environment, you must set custom VM types. For more information, see Prerequisites for Azure Stack or Azure Government Cloud.

Note: Azure Stack is in beta for this release. Pivotal does not recommend using Azure Stack for a production deployment. For more information, see Azure Stack Support is in Beta for Ops Manager.

Click Save.
Step 3: Director Config Page
Select Director Config to open the Director Config page.

Director config

In the NTP Servers (comma delimited) field, enter a comma-separated list of valid NTP servers.

Leave the JMX Provider IP Address field blank.

Note: Starting from PCF v2.0, BOSH-reported system metrics are available in the Loggregator Firehose by default. Therefore, if you continue to use PCF JMX Bridge for consuming them outside of the Firehose, you may receive duplicate data. To prevent this, leave the JMX Provider IP Address field blank. For additional guidance, see BOSH System Metrics Available in Loggregator Firehose.

Leave the Bosh HM Forwarder IP Address field blank.

Note: Starting from PCF v2.0, BOSH-reported system metrics are available in the Loggregator Firehose by default. Therefore, if you continue to use the BOSH HM Forwarder for consuming them, you may receive duplicate data. To prevent this, leave the Bosh HM Forwarder IP Address field blank. For additional guidance, see BOSH System Metrics Available in Loggregator Firehose.

Select the Enable VM Resurrector Plugin checkbox to enable the Ops Manager Resurrector functionality and increase Pivotal Application Service (PAS) availability.

Select Enable Post Deploy Scripts to run a post-deploy script after deployment. This script allows the job to execute additional commands against a deployment.

Select Recreate all VMs to force BOSH to recreate all VMs on the next deploy. This process does not destroy any persistent disk data.

Select Enable bosh deploy retries if you want Ops Manager to retry failed BOSH operations up to five times.

Select Keep Unreachable Director VMs if you want to preserve BOSH Director VMs after a failed deployment for troubleshooting purposes.

Select HM Pager Duty Plugin to enable Health Monitor integration with PagerDuty.

Director hm pager

Service Key: Enter your API service key from PagerDuty.
HTTP Proxy: Enter an HTTP proxy for use with PagerDuty.
Select HM Email Plugin to enable Health Monitor integration with email.

Director hm email

Host: Enter your email hostname.
Port: Enter your email port number.
Domain: Enter your domain.
From: Enter the address for the sender.
Recipients: Enter comma-separated addresses of intended recipients.
Username: Enter the username for your email server.
Password: Enter the password for your email server.
Enable TLS: Select this checkbox to enable Transport Layer Security.
For CredHub Encryption Provider, you can choose whether BOSH CredHub stores its encryption key internally on the BOSH Director and CredHub VM, or in an external hardware security module (HSM). The HSM option is more secure. Before configuring an HSM encryption provider in the Director Config pane, you must follow the procedures and collect information as described in Preparing CredHub HSMs for Configuration.

Note: After you deploy Ops Manager with an HSM encryption provider, you cannot change BOSH CredHub to store encryption keys internally.

CredHub Encryption Provider options in the Director Config pane
Internal: Select this option for internal CredHub key storage. This option is selected by default and requires no additional configuration.
Luna HSM: Select this option to use a SafeNet Luna HSM as your permanent CredHub encryption provider, and fill in the following fields:
Encryption Key Name: Any name to identify the key that the HSM uses to encrypt and decrypt the CredHub data. Changing this key name after you deploy Ops Manager could cause service downtime.
Provider Partition: The partition that stores your encryption key. Changing this partition after you deploy Ops Manager could cause service downtime. For this value and the ones below, use values gathered from Preparing CredHub HSMs for Configuration.
Provider Partition Password
Provider Client Certificate: The certificate that validates the identity of the HSM when CredHub connects as a client.
Provider Client Certificate Private Key
HSM Host Address
HSM Port Address: If you don’t know your port address, enter 1792.
Partition Serial Number
HSM Certificate: The certificate that the HSM presents to CredHub to establish a two-way mTLS connection.
Select a Blobstore Location to either configure the blobstore as an internal server or an external endpoint. Because the internal server is unscalable and less secure, Pivotal recommends you configure an external blobstore.

Note: After you deploy Ops Manager, you cannot change the blobstore location.

Blobstore location options in the Director Config pane
Internal: Select this option to use an internal blobstore. Ops Manager creates a new VM for blob storage. No additional configuration is required.
S3 Compatible Blobstore: Select this option to use an external S3-compatible endpoint. Follow the procedures in Sign up for Amazon S3 and Creating a Bucket from the AWS documentation. When you have created an S3 bucket, complete the following steps:
S3 Endpoint: Navigate to the Regions and Endpoints topic in the AWS documentation. Locate the endpoint for your region in the Amazon Simple Storage Service (S3) table and construct a URL using your region’s endpoint. For example, if you are using the us-west-2 region, the URL you create would be https://s3-us-west-2.amazonaws.com. Enter this URL into the S3 Endpoint field in Ops Manager.
Bucket Name: Enter the name of the S3 bucket.
Access Key and Secret Key: Enter the keys you generated when creating your S3 bucket.
Select V2 Signature or V4 Signature. If you select V4 Signature, enter your Region.
Note: AWS recommends using Signature Version 4. For more information about AWS S3 Signatures, see the Authenticating Requests documentation.

GCS Blobstore: Select this option to use an external Google Cloud Storage (GCS) endpoint. To create a GCS bucket, you will need a GCS account. Then follow the procedures in Creating Storage Buckets. When you have created a GCS bucket, complete the following steps:
Bucket Name: Enter the name of your GCS bucket.
Storage Class: Select the storage class for your GCS bucket. See Storage Classes in the GCP documentation for more information.
Service Account Key: Follow the steps in the Set Up an IAM Service Account section to download a JSON file with a private key. Then enter the contents of the JSON file into the field.
For Database Location, select Internal.

(Optional) Director Workers sets the number of workers available to execute Director tasks. This field defaults to 5.

(Optional) Max Threads sets the maximum number of threads that the BOSH Director can run simultaneously. Pivotal recommends that you leave the field blank to use the default value, unless doing so results in rate limiting or errors on your IaaS.

(Optional) To add a custom URL for your BOSH Director, enter a valid hostname in Director Hostname. You can also use this field to configure a load balancer in front of your BOSH Director. For more information, see How to set up a load balancer in front of BOSH Director in the Pivotal Knowledge Base.

WARNING: If you change the Director Hostname after your initial deployment, VMs become unavailable. This causes PCF downtime. To restore VM availability, enable Recreate All VMs and redeploy.

(Optional) Enter your list of comma-separated Excluded Recursors to declare which IP addresses and ports should not be used by the DNS server.

(Optional) To disable BOSH DNS, select the Disable BOSH DNS server for troubleshooting purposes checkbox. For more information about the BOSH DNS service discovery mechanism, see BOSH DNS Service Discovery for Application Containers in the PAS Release Notes.

Breaking Change: Do not disable BOSH DNS without consulting Pivotal Support. For more information about disabling BOSH DNS, see Disabling or Opting Out of BOSH DNS in PCF in the Pivotal Knowledge Base.

(Optional) To set a custom banner that users see when logging in to the Director using SSH, enter text in the Custom SSH Banner field. Disable bosh dns

Click Save.

Step 4: Create Networks Page
Select Create Networks and follow the procedures in this section to add the network configuration you created for your VPC.

Create the Management Network
Click Add Network.

(Optional) Select Enable ICMP checks to enable ICMP on your networks. Ops Manager uses ICMP checks to confirm that components within your network are reachable.

For Name, enter the name of the network you want to create. For example, Management.

Under Subnets, complete the following fields:

Azure Network Name: Enter NETWORK-NAME/SUBNET-NAME, where NETWORK-NAME is the network_name output from Terraform and SUBNET-NAME is the management_subnet_name output from Terraform.
Note: The Azure portal sometimes displays the names of resources with incorrect capitalization. Always use the Azure CLI to retrieve the correctly capitalized name of a resource.

CIDR: Enter the CIDR listed under management_subnet_cidrs output from Terraform. For example, 10.0.8.0/26.
Reserved IP Ranges: Enter the first 9 IP addresses of the subnet. For example, 10.0.8.1-10.0.8.9.
DNS: Enter 168.63.129.16.
Gateway: Enter the first IP address of the subnet. For example, 10.0.8.1. Create networks management
Click Save.

Note: After you deploy Ops Manager, you add subnets with overlapping AZs to expand your network. For more information on how to configure additional subnets, see Expanding Your Network with Additional Subnets.

Create the PAS Network
Click Add Network.

(Optional) Select Enable ICMP checks to enable ICMP on your networks. Ops Manager uses ICMP checks to confirm that components within your network are reachable.

For Name, enter the name of the network you want to create. For example, PAS.

Under Subnets, complete the following fields:

Azure Network Name: Enter NETWORK-NAME/SUBNET-NAME, where NETWORK-NAME is the network_name output from Terraform and SUBNET-NAME is the pas_subnet_name output from Terraform.
CIDR: Enter the CIDR listed under pas_subnet_cidrs output from Terraform. For example, 10.0.0.0/22.
Reserved IP Ranges: Enter the first 9 IP addresses of the subnet. For example, 10.0.0.1-10.0.0.9.
DNS: Enter 168.63.129.16.
Gateway: Enter the first IP address of the subnet. For example, 10.0.0.1. Create networks deployment
Click Save.

Create the Services Network
(Optional) Select Enable ICMP checks to enable ICMP on your networks. Ops Manager uses ICMP checks to confirm that components within your network are reachable.

For Name, enter the name of the network you want to create. For example, Services.

Under Subnets, complete the following fields:

Azure Network Name: Enter NETWORK-NAME/SUBNET-NAME, where NETWORK-NAME is the network_name output from Terraform and SUBNET-NAME is the services_subnet_name output from Terraform.
CIDR: Enter the CIDR listed under services_subnet_cidrs output from Terraform. For example, 10.0.4.0/22.
Reserved IP Ranges: Enter the first 9 IP addresses of the subnet. For example, 10.0.4.1-10.0.4.9.
DNS: Enter 168.63.129.16.
Gateway: Enter the first IP address of the subnet. For example, 10.0.4.1. Create networks services
Click Save. If you do not have Enable ICMP checks selected, you may see red warnings which you can safely ignore.

Step 5: Assign Networks Page
Select Assign Networks.

Assign networks

Use the drop-down menu to select the management network for your BOSH Director.

Click Save.

Step 6: Security Page
Select Security.

Om security

In Trusted Certificates, enter your custom certificate authority (CA) certificates to insert into your organization’s certificate trust chain. This feature enables all BOSH-deployed components in your deployment to trust custom root certificates. 

To enter multiple certificates, paste your certificates one after the other. For example, format your certificates like the following:

-----BEGIN CERTIFICATE-----
ABCDEFGH12345678ABCDEFGH12345678ABCDEFGH12345678AB
EFGH12345678ABCDEFGH12345678ABCDEFGH12345678ABCDEF
GH12345678ABCDEFGH12345678ABCDEFGH12345678...
------END CERTIFICATE------
-----BEGIN CERTIFICATE-----
BCDEFGH12345678ABCDEFGH12345678ABCDEFGH12345678ABB
EFGH12345678ABCDEFGH12345678ABCDEFGH12345678ABCDEF
GH12345678ABCDEFGH12345678ABCDEFGH12345678...
------END CERTIFICATE------
-----BEGIN CERTIFICATE-----
CDEFGH12345678ABCDEFGH12345678ABCDEFGH12345678ABBB
EFGH12345678ABCDEFGH12345678ABCDEFGH12345678ABCDEF
GH12345678ABCDEFGH12345678ABCDEFGH12345678...
------END CERTIFICATE------
Note: If you want to use Docker Registries for running app instances in Docker containers, enter the certificate for your private Docker Registry in this field. See the Using Docker Registries topic for more information.

Choose Generate passwords or Use default BOSH password. Pivotal recommends that you use the Generate passwords option for greater security.

Click Save. To view your saved Director password, click the Credentials tab.

Step 8: Syslog Page
Select Syslog. Syslog bosh

(Optional) To send BOSH Director system logs to a remote server, select Yes.

In the Address field, enter the IP address or DNS name for the remote server.

In the Port field, enter the port number that the remote server listens on.

In the Transport Protocol dropdown menu, select TCP, UDP, or RELP. This selection determines which transport protocol is used to send the logs to the remote server.

(Optional) Pivotal strongly recommends that you enable TLS encryption when forwarding logs as they may contain sensitive information. For example, these logs may contain cloud provider credentials. To enable TLS, perform the following steps.

In the Permitted Peer field, enter either the name or SHA1 fingerprint of the remote peer.
In the SSL Certificate field, enter the SSL certificate for the remote server.
Click Save.

Step 9: Resource Config Page
Select Resource Config. Om resource config

Adjust any values as necessary for your deployment. Under the Instances, Persistent Disk Type, and VM Type fields, choose Automatic from the drop-down menu to allocate the recommended resources for the job. If the Persistent Disk Type field reads None, the job does not require persistent disk space.

Note: Pivotal recommends provisioning a BOSH Director VM with at least 8 GB memory.

Note: If you set a field to Automatic and the recommended resource allocation changes in a future version, Ops Manager automatically uses the new recommended allocation.

(Optional) For Load Balancers, enter your Azure application gateway name for each associated job. To create an application gateway, follow the procedures in Configure an application gateway for SSL offload by using Azure Resource Manager from the Azure documentation. When you create the application gateway, associate the gateway’s IP with your PCF system domain.

Click Save.

Step 10: (Optional) Add Custom VM Extensions
Use the Ops Manager API to add custom properties to your VMs such as associated security groups and load balancers. For more information, see Managing Custom VM Extensions.

Step 11: Complete the BOSH Director Installation
Click the Installation Dashboard link to return to the Installation Dashboard.

Click Apply Changes. If the following ICMP error message appears, click Ignore errors and start the install.

Install error

BOSH Director installs. This may take a few moments. When the installation process successfully completes, the Changes Applied window appears.

Ops manager complete

After you complete this procedure, follow the instructions in the Deploying PAS on Azure Using Terraform topic



