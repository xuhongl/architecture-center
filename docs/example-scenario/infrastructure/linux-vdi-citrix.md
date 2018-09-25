---
title: Linux Virtual Desktops with Citrix
description: Proven scenario for building a VDI environment for Linux Desktops using Citrix on Azure.
author: miguelangelopereira 
ms.date: 09/12/2018 
---
# Linux Virtual Desktops with Citrix

This scenario is applicable to any industry that needs a Virtual Desktop Infrastructure (VDI) for Linux Desktops.  VDI refers to the process of running a user desktop inside a virtual machine that lives on a server in the datacenter. The customer in this sample scenario chose to use a Citrix-based solution for their VDI needs.

It's common for organizations to have heterogeneous environments with multiple devices and operating systems being used by employees. Providing consistent access to applications while maintaining a high level of security can be a challenge. A VDI solution for Linux desktops will allow your organization to provide access independently of the device or OS utilized by the end user.

Some benefits for this sample solution include:

- Increased ROI with shared Linux virtual desktops by giving more users access to the same infrastructure. By consolidating resources on a centralized VDI environment, the end user devices don't need to be as powerful.
- Performance will be consistent regardless of the end user device.
- Provides access to Linux applications on any device (including Non-Linux).
- Sensitive data can be secured in the Azure datacenter for all distributed employees.

## Potential use cases

Consider this scenario for the following use case:

- Provide secure access to mission-critical, specialized Linux VDI desktops from Linux or non-Linux devices

## Architecture

[![](./media/azure-citrix-sample-diagram.png "Architecture Diagram")](./media/azure-citrix-sample-diagram.png#lightbox)

This sample solution will allow the corporate network access to Linux Virtual Desktops:

- An ExpressRoute is established between the On-Premises environment and Azure for fast and reliable connectivity to the Cloud
- Citrix XenDeskop solution deployed for VDI
- The CitrixVDA run on Ubuntu (or another supported distro)
- Azure Network Security Groups will apply the correct network ACLs
- Citrix ADC (Netscaler) will publish and load balance all the Citrix services
- Active Directory Domain Services will be used to domain join the Citrix Servers. VDA servers will not be domain joined.
- Azure Hybrid File Sync will enable shared storage across the solution. For example, it can be used in remote /home solutions.

For this sample solution, the following SKUs are used:

- Citrix ADC (Netscaler): 2 x D4sv3 with [NetScaler 12.0 VPX Standard Edition 200 MBPS PAYG image](https://azuremarketplace.microsoft.com/pt-br/marketplace/apps/citrix.netscalervpx-120?tab=PlansAndPrice)
- Citrix License Server: 1 x D2s v3
- Citrix VDA: 4 x D8s v3
- Citrix Storefront: 2 x D2s v3
- Citrix Delivery Controller: 2 x D2s v3
- Domain Controllers: 2 x D2sv3
- Azure File Servers: 2 x D2sv3

> [!Note]
> All the licenses (besides Netscaler) are bring-your-own-license (BYOL)

### Components

- [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) allows resources such as VMs to securely communicate with each other, the Internet, and on-premises networks. Virtual networks provide isolation and segmentation, filter and route traffic, and allow connection between locations. One Virtual Network will be used  for all resources in the sample scenario.
- [Azure network security groups](/azure/virtual-network/security-overview) contain a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol. The virtual networks in this scenario are secured with network security group rules that restrict the flow of traffic between the application components.
- [Azure load balancer](/azure/application-gateway/overview) distributes inbound traffic according to rules and health probes. A load balancer provides low latency and high throughput, and scales up to millions of flows for all TCP and UDP applications. An internal load balancer is used in this scenario to distribute traffic on the Citrix Netscaler.
- [Azure Hybrid File Sync](https://github.com/MicrosoftDocs/azure-docs/edit/master/articles/storage/files/storage-sync-files-planning.md) will be used for all shared storage. The storage will replicate to two file servers using Hybrid File Sync.
- [Azure SQL Database](https://docs.microsoft.com/en-us/azure/sql-database/) is a relational database-as-a-service (DBaaS) based on the latest stable version of Microsoft SQL Server Database Engine. It will be used for hosting Citrix databases.
- [ExpressRoute](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider. 
- [Active Directory Domain Services is used for Directory Services and user authentication
- [Azure Availabilty Sets](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-availability-sets) will ensure that the VMs you deploy on Azure are distributed across multiple isolated hardware nodes in a cluster. Doing this ensures that if a hardware or software failure within Azure happens, only a subset of your VMs are impacted and that your overall solution remains available and operational. 
- [Citrix ADC (Netscaler)](https://www.citrix.com/products/citrix-adc/) is an application delivery controller that performs application-specific traffic analysis to intelligently distribute, optimize, and secure Layer 4-Layer 7 (L4–L7) network traffic for web applications. 
- [Citrix Storefront](https://www.citrix.com/products/citrix-virtual-apps-and-desktops/citrix-storefront.html) is an enterprise app store that improves security and simplifies deployments, delivering a modern, unmatched near-native user experience across Citrix Receiver on any platform. StoreFront makes it easy to manage multi-site and multi-version Citrix Virtual Apps and Desktops environments. 
- [Citrix License Server](https://www.citrix.com/buy/licensing/overview.html) will manage the licenses for Citrix Products.
- [Citrix XenDesktops VDA](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops-service.html) enables connections to applications and desktops. The VDA is installed on the machine that runs the applications or virtual desktops for the user. It enables the machines to register with Delivery Controllers and manage the High Definition eXperience (HDX) connection to a user device.
- [Citrix Delivery Controller](https://docs.citrix.com/en-us/xenapp-and-xendesktop/7-15-ltsr/manage-deployment/delivery-controllers.html) is the server-side component that is responsible for managing user access, plus brokering and optimizing connections. Controllers also provide the Machine Creation Services that create desktop and server images.

### Alternatives

- There are multiple partners with VDI solutions that supported in Azure such as VMware, Workspot, and others. This specific sample architecture is based on a deployed project that used Citrix.
- Citrix provides a Cloud Service that abstracts part of this architecture. It could be an alternative for this solution. See [Citrix Cloud](https://www.citrix.com/products/citrix-cloud/)

## Considerations

- Check the [Citrix Linux Requirements](https://docs.citrix.com/en-us/linux-virtual-delivery-agent/current-release/system-requirements.html) 
- Latency can have impact on the overall solution. For production environment, test accordingly.
- Depending on the scenario, the solution may need VMs with GPUs for VDA. For this solution, it is assumed that GPU is not a requirement.

### Availability, Scalability, and Security

- This sample solution is designed for High Availability for all roles besides the licensing server. Because the environment continues to function in a 30-day grace period if the license server is offline, no additional redundancy is required on that server.
- All servers providing similar roles should be deployed in [Availability Sets](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy).
- This sample solution does not include Disaster Recovery capabilities. [Azure Site Recovery](https://docs.microsoft.com/en-us/azure/site-recovery/site-recovery-overview) could be a good add-on to this design.
- For a production deployment management solution should be implemented such as [backup](https://docs.microsoft.com/en-us/azure/backup/backup-introduction-to-azure-backup), [monitoring](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-overview) and [update management](https://docs.microsoft.com/en-us/azure/automation/automation-update-management).
- This sample solution should work for about 250 concurrent (about 50-60 per VDA server) users with a mixed usage. But that will greatly depended on the type of applications being used. For production use, rigorous load testing should be performed.

## Deploy this scenario

For deployment information see the official [Citrix documentation](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure.html).

## Pricing

- The Citrix XenDestop licenses are not included in Azure service charges
- The Citrix Netscaler license is included in a pay-as-you-go model
- Using reserved instances will greatly reduce the compute cost for the solution
- The ExpressRoute cost is not included

## Next Steps

- Check Citrix documentation for planning and deployment [here](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure.html)
- For deploying Citrix ADC (Netscaler) in Azure. Check the Resource Manager templates provided by Citrix [here](https://github.com/citrix/netscaler-azure-templates). 