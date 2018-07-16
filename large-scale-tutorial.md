
This tutorial provides instructions on how to set up a large-scale cloud architecture for use with the 3D Streaming Toolkit.
1. Log in to Azure
2. Create a new Virtual Network
3. Deploy a Signaling Server
4. Create a new Azure Batch account
5. Upload streaming application releases to Azure Batch
6. Upload all dependencies and functional tests to Azure Batch
7. Clone/Download the sample Cloud3DSTKDeployment web app/api project
8. Modify/add QA tasks for Linux/TURN pool creation
9. Modify/add QA tasks for Windows/Rendering pool creation
10. Run the Web project, locally or in the cloud
11. Invoke the Cloud3DSTKDeployment endpoints with HTTP POST
12. Browse the Batch account using the Portal or Batch Explorer
13. Connect the clients to the VMs

Extra customization: 
14. Customizing your Compute Nodes
15. Setting up Azure AD for Batch usage
16. Network Configuration for Batch usage

# 1. Log in to Azure

Log in to the Azure Portal at: https://portal.azure.com

![01-login-portal](https://user-images.githubusercontent.com/779562/42053151-c261c35c-7ade-11e8-953a-6560c88a54c2.png)

# 2. Create a new Virtual Network

1. Click + "Create a resource".
2. Search for Virtual Network and click to proceed.
3. Complete the fields for the network's Name, Address Space, Subscription, Resource Group, Location, Subnet, Address Range, DDoS Protection Level, and any Service Endpoints. 
4. For more information, refer to the following guide: https://docs.microsoft.com/en-us/azure/virtual-network/quick-create-portal

![02-virtual-network-create](https://user-images.githubusercontent.com/779562/42053400-667801c2-7adf-11e8-8bc5-ab938cddb955.png)
 
# 3. Deploy a Signaling Server

1. Refer to the Signaling Server from the following repo: https://github.com/3DStreamingToolkit/signal-3dstk
2. Clone the repo locally if you wish to download the code.
3. Click the "Deploy to Azure" button directly from the repo's Readme page.

![03-signaling-server-deploy](https://user-images.githubusercontent.com/10086264/42473193-ef51b3d4-8391-11e8-8342-72770b5279c1.png)
 
# 4. Create a new Azure Batch account

1. Click + "Create a resource".
2. Search for Batch Service and click to proceed.
3. Complete the fields for the Batch Account's Name, Subscription, Resource Group (use existing), Location, Storage Account, Pool Allocation Mode (Batch Service or User Subscription). 
4. For more information, refer to the upper sections of the following guide: https://docs.microsoft.com/en-us/azure/batch/quick-create-portal
5. NOTE: you won't have to create any pools or nodes manually because we will do this programmatically using the Batch SDK.

![04-batch-create](https://user-images.githubusercontent.com/779562/42054261-e0fadb2a-7ae1-11e8-8e76-e19c36a7dd0d.PNG)

# 5. Upload streaming application releases to Azure Batch

1. Visit the Azure Portal at: https://portal.azure.com
2. Click on the Batch account you created earlier.
3. Browse the sections to see Applications within that Batch account.
4. Upload the application releases with desired versioning. For each pool creation, Azure Batch will automatically copy a specific or latest version of the application and unzip the content.  

# 6. Upload all dependencies and functional tests to Azure Batch

1. Browse to the same Applications section as step 6
2. Upload the desired Nvidia driver install. For Azure NV series get the latest [here](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/n-series-driver-setup)
3. Upload all dependency installers required for your streaming application. For example, for our DirectX sample servers, we require the [Visual C++ Redistributable for Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48145) 
4. Retrieve and upload the latest end-to-end functional test from [3D Streaming toolkit releases](https://github.com/3DStreamingToolkit/3DStreamingToolkit/releases). The latest version is [3DStreamingToolkit-FunctionalTests-v2.0](https://github.com/3DStreamingToolkit/3DStreamingToolkit/releases/download/v2.0/3DStreamingToolkit-FunctionalTests-v2.0.zip)
6. Upload the [latest server deployment script](https://github.com/3DStreamingToolkit/cloud-deploy/tree/master/batch-application-packages/streaming-server-deployment)
7. You can upload any other scripts or installers that you require for your VM creation

![06-batch-applications](./Images/batch-applications.jpg) 
 
# 7. Clone/Download the sample Cloud3DSTKDeployment web app/api project

1. Refer to the Cloud3DSTKDeployment project from the following repo: https://github.com/3DStreamingToolkit/cloud-deploy
2. Clone the repo locally and open `Cloud3DSTKDeployment.sln` in Visual Studio 2017.

![07-cloud-deploy](./Images/clouddeploy.jpg)

# 8. Modify/add QA tasks for Linux/TURN pool creation

1. Open to `Services/BatchService.cs` and look at `CreateTurnPool` method
2. Set the desired VM type and Ubuntu version for your TURN server nodes. Be default, we use the `STANDARD_A1` machines and `Ubuntu Server 14.04.5-LTS`. See the [Azure Batch documentation](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) for more details.
```
pool = this.batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: dedicatedNodes,
    virtualMachineSize: "STANDARD_A1",
    virtualMachineConfiguration: new VirtualMachineConfiguration(
        new ImageReference(
            offer: "UbuntuServer",
            publisher: "Canonical",
            sku: "14.04.5-LTS"),
        nodeAgentSkuId: "batch.node.ubuntu 14.04"));
```
3. Modify the StartTask if you wish to install a different type of TURN server. In our sample we install a simple docker image running at port 3478 and hardcoded username and password. This is not the recommended approach if you wish to use authentication. See [our code story](https://www.microsoft.com/developerblog/2018/01/29/orchestrating-turn-servers-cloud-deployment/) for more options. 
```
pool.StartTask = new StartTask
{
    // Run a command to install docker and get latest TURN server implementation
    CommandLine = "bash -c \"sudo apt-get update && sudo apt-get -y install docker.io && sudo docker run -d -p 3478:3478 -p 3478:3478/udp --restart=always zolochevska/turn-server username password realm\"",
    UserIdentity = new UserIdentity(new AutoUserSpecification(AutoUserScope.Task, ElevationLevel.Admin)),
    WaitForSuccess = true,
    MaxTaskRetryCount = 2
};
```

# 9. Modify/add QA tasks for Windows/Rendering pool creation
1. Open `Services/BatchService.cs` and look at `CreateRenderingPool` method
2. Set the desired VM size and Windows version. For optimal streaming, we recommended using the `Standard_NV6 VM` and `Windows Server 2016`.  
```
pool = this.batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: dedicatedNodes,
    virtualMachineSize: "Standard_NV6",  // NV-series, 6 CPU, 1 GPU, 56 GB RAM 
    virtualMachineConfiguration: new VirtualMachineConfiguration(
        new ImageReference(
            offer: "WindowsServer",
            publisher: "MicrosoftWindowsServer",
            sku: "2016-DataCenter",
            version: "latest"),
        nodeAgentSkuId: "batch.node.windows amd64"));
``` 
3. Modify the application packages that you wish to include for each pool. Here is an example based on the applications uploaded above:
```
// Specify the application and version to install on the compute nodes
pool.ApplicationPackageReferences = new List<ApplicationPackageReference>
    {
        new ApplicationPackageReference
        {
            ApplicationId = "NVIDIA",
            Version = "391.58"
        },
        new ApplicationPackageReference
        {
            ApplicationId = "native-server-tests",
            Version = "1"
        },
        new ApplicationPackageReference
        {
            ApplicationId = "sample-server",
            Version = "1.0"
        },
         new ApplicationPackageReference
         {
            ApplicationId = "vc-redist",
            Version = "2015"
         },
         new ApplicationPackageReference
         {
            ApplicationId = "server-deploy-script",
            Version = "1.0"
         }
    };
```
4. Modify the `StartTask` to install all your dependencies and run the unit tests. In this example, we copy the rendering application to a desired path, install vc-redist, nvidia drivers and run the end-to-end functional tests:
```
// Create and assign the StartTask that will be executed when compute nodes join the pool.
pool.StartTask = new StartTask
{
    // Install all packages and run Unit tests to ensure the node is ready for streaming
    CommandLine = string.Format(
        "cmd /c robocopy %AZ_BATCH_APP_PACKAGE_sample-server#1.0% {0} /E && " +
        "cmd /c %AZ_BATCH_APP_PACKAGE_vc-redist#2015%\\vc_redist.x64.exe /install /passive /norestart && " +
        "cmd /c %AZ_BATCH_APP_PACKAGE_NVIDIA#391.58%\\setup.exe /s && " +
        "cmd /c %AZ_BATCH_APP_PACKAGE_native-server-tests#1%\\NativeServerTests\\NativeServer.Tests.exe --gtest_also_run_disabled_tests --gtest_filter=\"*Driver*:*Hardware*\"",
    this.serverPath),
    UserIdentity = new UserIdentity(new AutoUserSpecification(AutoUserScope.Task, ElevationLevel.Admin)),
    WaitForSuccess = true,
    MaxTaskRetryCount = 2
};
```
5. If required, add extra tasks that each pool will perform in the `AddRenderingTasksAsync` method. In our example, we run the server-deploy-script to set the correct signaling/TURN server information and start the rendering application as a Windows service. 
```
var startRenderingCommand = string.Format(
        "cmd /c powershell -command \"start-process powershell -verb runAs -ArgumentList '-ExecutionPolicy Unrestricted -file %AZ_BATCH_APP_PACKAGE_server-deploy-script#1.0%\\server_deploy.ps1 {1} {2} {3} {4} {5} {6} {7} {0} '\"",
        this.serverPath,
        string.Format("turn:{0}:3478", turnServerIp),
        "username",
        "password",
        signalingServerURL,
        signalingServerPort,
        5000,
        serverCapacity);
```

# 10. Run the Web project, locally or in the cloud.

1. Update the JSON configuration file for your environment
3. Refer to the sample settings below and replace with your own values
4. Run the Web App project locally or after deploying to your Azure account

```json
{
  "BatchAccountName": "REPLACE_BATCH_ACCOUNT_NAME",
  "BatchAccountKey": "REPLACE_BATCH_ACCOUNT_KEY",
  "BatchAccountUrl": "REPLACE_BATCH_ACCOUNT_URL",
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    }
  }
}
```
 
# 11. Invoke the Cloud3DSTKDeployment endpoints with HTTP POST

1. Make a note of the Web API endpoint and append the API route path, e.g. https://localhost:44329/api/create
2. Prepare to call the endpoint with a POST request, e.g. using a tool such as Postman
3. Use the JSON sample below and replace the values to provide your own input.
4. Call the endpoint and observe the response.

```json
{
  "signalingServer": "SIGNALING_URI", // Required
  "signalingServerPort": 80, // Required
  "vnet": "/subscriptions/{subscription}/resourceGroups/{group}/providers/{provider}/virtualNetworks/{network}/subnets/{subnet}", 
  "renderingPoolId": "RENDERING_POOL_ID",
  "renderingJobId": "RENDERING_JOB_ID",
  "dedicatedRenderingNodes": 1,
  "maxUsersPerRenderingNode": 1,
  "turnPoolId": "TURN_POOL_ID",
  "dedicatedTurnNodes": 1
}
```

# 12. Browse the Batch account using the Portal or Batch Explorer

1. Revisit the Azure Portal at: https://portal.azure.com
2. Click on the Batch account you created earlier.
3. Browse the sections to see Applications, Pools and Jobs within that Batch account
4. Check the Pools created with the POST request above
5. Monitor the Nodes in each Pool and wait until the status reached the IDLE state
6. The Server Rendering applications are now running on the VM and clients are able to connect by joining the signaling server 

# 13. Connect the clients to the VMs

1. Look at the [Get Started section](./index.md#get-started) to setup a client. 
2. The WebRTC configuration should use the signaling server deployed in step 3. The TURN server credentials are automatically passed down to the clients (DirectX and WebClient samples) on connection.
3. Depending on the servers capacity, dedicated nodes and pools, you can now connect multiple clients and easily scale up/down inside the Azure Batch portal or by triggering the API endpoints from step 11. 

## Extra customization
 
# 14. Customizing your Compute Nodes.

When you call the [CreatePool() method](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.pooloperations.createpool?view=azure-dotnet) in the Batch SDK, you have several options how you want to create the Virtual Machines as Compute Nodes within your pool.

![14a-azure-batch-createpool-options](https://user-images.githubusercontent.com/779562/42060961-90c86a36-7af6-11e8-96ef-c5afd4e1df25.png)

* Option A: [Cloud Service Configuration](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.cloudserviceconfiguration?view=azure-dotnet): Windows Servers only
* Option B: [VM Configuration](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration?view=azure-dotnet): Both Windows and Linux Servers, where Batch uses Scale Sets for Linux Servers.

**Option A**

When using a Cloud Service Configuration, you will have to pick a virtualMachineSize (e.g. standard_d1_v2) and an osFamily (e.g. 5 for Windows Server 2016). You may find the complete list of options here:
https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-sizes-specs

Sample Code:
```C#
pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 3, // 3 compute nodes
    virtualMachineSize: "standard_d1_v2",  // single-core, 3.5 GB memory, 50 GB disk
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5")); // Windows Server 2016
```
**Option B**

When using a Virtual Machine Configuration, you will get additional options to use an ImageReference, either from the Azure directory or from a custom image.

* If you use an ImageReference, you will have to provide values for the Publisher, Offer, SKU and NodeAgentSkuId. The quickest way to find out these values is in the Azure portal. Simply browse your Batch account in the Azure portal and click Add Pool to see what options are available to you.

```C#
var pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 1,
    virtualMachineSize: "STANDARD_A1", 
    virtualMachineConfiguration: new VirtualMachineConfiguration(
        new ImageReference(
            offer: "UbuntuServer",
            publisher: "Canonical",
            sku: "14.04.5-LTS"),
        nodeAgentSkuId: "batch.node.ubuntu 14.04")
```

![14b-batch-pool-vm-options](https://user-images.githubusercontent.com/779562/42060992-ac10976e-7af6-11e8-852c-f288a78f97eb.PNG)

* If you choose to use a custom image, you will have to provide a path to the custom VM image. This VM image must be in 
the _same region and subscription as the Azure Batch account_. This makes your call to VirtualMachineConfiguration a lot simpler:

```C#

virtualMachineConfiguration: new VirtualMachineConfiguration(
    new ImageReference(virtualMachineImageId: VirtualMachineImageId),
    nodeAgentSkuId: "batch.node.windows amd64")

// where VirtualMachineImageId is in the following format 
// /subscriptions/xxxx-xxxxxx-xxxxx-xxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage
```

NOTE: When using a custom image, you may use Azure AD authentication to access the VM image. 

# 15. Setting up Azure AD for Batch usage.

To set up Azure AD for Batch usage, follow the instructions provided at:
* https://docs.microsoft.com/en-us/azure/batch/batch-aad-auth

This involves the following steps:
1. Register your application with a tenant
2. Get the tenant ID for your Active Directory
3. Use integrated authentication (not Service Principal)

When registering your Batch App, you will have to add it as either a Web App/API or a Native App. 
* If you choose the Web option, you will have to provide API Access Keys. 
* If you have a Console app (or other Native app), you should choose the Native option, which does not require any API Access Keys. 

When adding integrated authentication, you will have the option to select the Batch API by searching for one of the following:
a. MicrosoftAzureBatch (no spaces)
b. Microsoft Azure Batch (with spaces)
c. ddbf3205-c6bd-46ae-8127-60eb93363864 (the ID for the Batch API) 

As of this writing, Option B is the best way to add the Batch API, i.e. searching for Microsoft Azure Batch (with spaces). Registering your Batch application will allow you to use your custom VM image as described above.

After you've registered your Batch application, you may browse the Azure Portal under Azure Active Directory, then App Registrations to see a list of registered apps.

![15-azure-ad-batch](https://user-images.githubusercontent.com/779562/42062008-2307277c-7afa-11e8-8724-af02caae88a3.png)
 
# 16. Network Configuration for Batch usage.

In the very first step, we set up a Virtual Network. In order to use it programmatically with the Batch SDK, simply set the [NetworkConfiguration.SubnetId](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.networkconfiguration.subnetid?view=azure-dotnet#Microsoft_Azure_Batch_NetworkConfiguration_SubnetId) property to the Subnet Id value of your Virtual Network. You can also customize the Port/IP restrictions for your virtual network. 

Sample Code to set NetworkConfiguration for a Batch pool
```C#
pool.NetworkConfiguration = GetNetworkConfiguration();
```

Sample Code to set up a Network Configuration 
```
var networkConfiguration = new NetworkConfiguration
{
    EndpointConfiguration = new PoolEndpointConfiguration(new InboundNatPool[]
       {
        new InboundNatPool(
            name: "UDP_3478",
            protocol: InboundEndpointProtocol.Udp,
            backendPort: 3478,
            frontendPortRangeStart: 3478,
            frontendPortRangeEnd: 3578),
        new InboundNatPool(
            name: "UDP_5349",
            protocol: InboundEndpointProtocol.Udp,
            backendPort: 5349,
            frontendPortRangeStart: 5349,
            frontendPortRangeEnd: 5449),
       }.ToList())
};
```

There are some guidelines and restrictions when using a creating a Virtual Network for use with Batch. Note that these may be subject to change.  
* The specified Virtual Network (VNet) must be in the same Azure region as the Azure Batch account.
* The specified VNet must be in the same subscription as the Azure Batch account.
* The specified VNet must be a Classic VNet. VNets created via Azure Resource Manager are not supported.
* The specified subnet should have enough free IP addresses to accommodate the "targetDedicated" property. If the subnet doesn't have enough free IP addresses, the pool will partially allocate compute nodes, and a resize error will occur.
* The "MicrosoftAzureBatch" service principal must have the "Classic Virtual Machine Contributor" Role-Based Access Control (RBAC) role for the specified VNet. If the specified RBAC role is not given, the Batch service returns 400 (Bad Request).
* The specified subnet must allow communication from the Azure Batch service to be able to schedule tasks on the compute nodes. This can be verified by checking if the specified VNet has any associated Network Security Groups (NSG). If communication to the compute nodes in the specified subnet is denied by an NSG, then the Batch service will set the state of the compute nodes to unusable.
* This property can be specified only for pools created with cloudServiceConfiguration. If this is specified on pools created with the virtualMachineConfiguration property, the Batch service returns 400 (Bad Request).

**SOURCE**: https://blogs.msdn.microsoft.com/maheshk/2016/11/25/how-to-specify-vnetclassic-when-creating-new-azurebatchpool-compute-nodes/

# Features & Limitations

There are some features and limitations to be aware of when using Azure Batch:
* When using [CloudServiceConfiguration](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.cloudserviceconfiguration?view=azure-dotnet), _only_ Windows VMs can be generated.
* When using [VirtualMachineConfiguration](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration?view=azure-dotnet), _both_ Windows and Linux VMs can be generated.
* The Batch Service use [VM Scale Sets](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview) to provide Linux compute nodes. 
* When using a VM Configuration, an image can be used from the [Azure Marketplace](https://azure.microsoft.com/marketplace/virtual-machines/) or from a custom image.
* Windows VM sizes must be selected from [sizes available for Windows on Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 
* Linux VM sizes must be selected from one of the [sizes available for Linux on Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 
* If using a [custom image](https://docs.microsoft.com/en-us/azure/batch/batch-custom-images), the VM image must reside in the same region and subscription as the Azure Batch account.
* For the Web API implementation, [SharedKeyCredentials](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.auth.batchsharedkeycredentials?view=azure-dotnet) are enough to create a VM in Batch, but a custom image would require [TokenCredentials](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.auth.batchtokencredentials?view=azure-dotnet) which needs [Azure AD](https://docs.microsoft.com/en-us/azure/batch/batch-aad-auth) for authentication.

# Related resources

* Azure Batch website: https://azure.microsoft.com/en-us/services/batch/
* Batch Documentation: https://docs.microsoft.com/en-us/azure/batch/
* Azure Batch in a Virtual Network: https://docs.microsoft.com/en-us/azure/batch/batch-virtual-network
* Batch Samples on GitHub: https://github.com/Azure/azure-batch-samples
* Azure Code Samples: https://azure.microsoft.com/en-us/resources/samples/?service=batch&sort=0