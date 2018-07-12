## Introduction

3DStreamingToolkit's purpose is to allow building powerful stereoscopic 3D experiences that run on the cloud and stream to low-powered devices. In order to achieve this, a large-scale architecture is required that has all the required WebRTC servers (signaling and TURN) and an orchestrator capable of monitoring and scaling up/down pools of VMs that host the rendering applications. Clients can easily connect to the signaling server and the orchestator will decide what VM should connect to the user.

Key components:
1. Signaling Server Web API  
2. Orchestrator and cloud scaling Web API 
3. Azure Batch 

### Signaling Server Web API

This enables webrtc peer communication across the 3DStreamingToolkit server/client stack. This means that it can be used to faciliate communication between N clients, N peers, and/or both. It uses http as a protocol, and can run over https as well. Further, authentication can be toggled on, requiring clients to provide valid OAuth 2.0 tokens in order to successfully access the service. 

### Orchestrator and cloud scaling Web API 

This is a custom Web API that and will dynamically monitor the overall active users and capacity and decide when to create or delete pools of VMs inside Azure Batch tu sustain a high number of users. Each pool creation will add the correct dependencies, install the custom applications connect the VM to the signaling server and ensure that each application is ready for streaming. 

### Azure Batch

Azure Batch allows you to run large-scale parallel and high-performance computing (HPC) apps efficiently in the cloud. You can schedule compute-intensive work to run on pools of virtual machines, run tasks on those VMs, monitor the state and scale as needed.

More info on Azure Batch: [https://docs.microsoft.com/en-us/dotnet/api/overview/azure/batch?view=azure-dotnet](https://docs.microsoft.com/en-us/dotnet/api/overview/azure/batch?view=azure-dotnet)

## Reference architecture

![architecture](./Images/large-scale-3DSTK-architecture.jpg)






# VMs in 3D Streaming Toolkit

The 3D Streaming Toolkit requires at least 1 Linux virtual machine to act as a [TURN](https://en.wikipedia.org/wiki/Traversal_Using_Relays_around_NAT) server and a pool of Windows Server virtual machines to run the rendering engine on NVIDIA GPUs. Azure Batch allows us to spin up these VMs (i.e. compute nodes) in their own pools, within a designated virtual network (vnet).

Please note that "_The virtual network must be in the same region and subscription as the Azure Batch account. The specified subnet should have enough free IP addresses to accommodate the number of nodes in the pool. If the subnet doesn't have enough free IP addresses, the pool will partially allocate compute nodes, and a resize error will occur._"

More info on setting NetworkConfiguration.SubnetId: [https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.networkconfiguration.subnetid?view=azure-dotnet#Microsoft_Azure_Batch_NetworkConfiguration_SubnetId](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.batch.networkconfiguration.subnetid?view=azure-dotnet#Microsoft_Azure_Batch_NetworkConfiguration_SubnetId)

# API Endpoint for Batch Orchestration

To make it easier to spin up the necessary VMs for a 3DST server environment, a Web API endpoint has been provided. The endpoint can be used to submit an HTTP POST request with the required (and some optional) parameters. A Web API project named "BatchPoolWebApp" is available in the following GitHub repository:
[https://github.com/3DStreamingToolkit/cloud-deploy](https://github.com/3DStreamingToolkit/cloud-deploy)

The Web API project uses `ASP.NET` and it requires [Visual Studio 2017](https://visualstudio.com/vs) (v15.7 or higher).

Below is a sample snippet of the JSON input that can be provided:

```json
{
  "signalingServer": "SIGNALING_URI", // Required
  "signalingServerPort": 80, // Required
  "renderingPoolId": "RENDERING_POOL_ID",
  "renderingJobId": "RENDERING_JOB_ID",
  "dedicatedRenderingNodes": 1,
  "maxUsersPerRenderingNode": 1,
  "turnPoolId": "TURN_POOL_ID",
  "dedicatedTurnNodes": 1
}
```

# Environment Setup

Before you can run the Web API project and call an endpoint, you must set up the following in your Azure environment.
1. Create a [new Virtual Network](https://docs.microsoft.com/en-us/azure/virtual-network/quick-create-portal). 
2. Create a [new Batch account](https://docs.microsoft.com/en-us/azure/batch/batch-account-create-portal).
3. Deploy a [Signaling Server](https://github.com/3DStreamingToolkit/signal-3dstk) which will be used to route incoming client requests to a VM compute node.

# App Settings for ASP .NET Configuration

ASP .NET Core allows you to store your [application settings](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-2.1&tabs=basicconfiguration) in an "appsettings.json" file, and environment-specific files such as "appsettings.Development.json". [Environments](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-2.1&tabs=basicconfiguration) may include: Development, Staging and Production.

Below is a sample of an "appsettings.Development.json" file in the Web API project:

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

The placeholder values beginning with REPLACE_ should be replaced by your actual Batch Account credentials.

# HTTPS and SSL in ASP .NET Web API

By default, ASP .NET Core provides an SSL certificate that can be used during development. This allows the developer to [enforce https](https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-2.1&tabs=visual-studio) when calling any URI or endpoint in the application. 

When testing with a utility such as Postman, you may have to disable SSL certificate verification, in case the dev certificate is being blocked. Below is a screenshot of the Postman settings screen, showing this option.

![postman-ssl](https://user-images.githubusercontent.com/779562/41788944-792426d4-761b-11e8-9e03-c8ad9b34173c.PNG)

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


# Resources & Links:
* Azure Batch website: https://azure.microsoft.com/en-us/services/batch/
* Batch Documentation: https://docs.microsoft.com/en-us/azure/batch/
* Azure Batch in a Virtual Network: https://docs.microsoft.com/en-us/azure/batch/batch-virtual-network
* Batch Samples on GitHub: https://github.com/Azure/azure-batch-samples
* Azure Code Samples: https://azure.microsoft.com/en-us/resources/samples/?service=batch&sort=0
* Batch Pool Projects: https://github.com/shahedc/BatchPoolProjects

