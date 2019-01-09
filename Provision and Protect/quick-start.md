# Use Case: Provision and Protect with vRealize Automation
The Provision and Protect use case highlights how Rubrik can be integrated into existing or new vRealize Automation (vRA) blueprints surrounding VM Provisioning, requiring users to select an SLA Domain Name during the provisioning process. This ensures that each and every time a vRA catalog item is requested, data protection policies are applied to the newly provisioned VMs. 

## Abstract
vRA delivers a robust self-service architecture allowing end users, developers, and customers to request and deploy new IT services with increased efficiency. End users are able to gain access to services much faster, but this often leads to important business functions such as data protection being left by the wayside. Coupling Rubrik with vRA ensures that data protection policies are not an afterthought as new workloads are created. Instead, new workload requests are set up to require the selection of a Rubrik SLA Domain, giving IT Operations the guarantee that any data provisioned through vRA’s self-service catalog is protected and available. 

At a high level, the Provision and Protect use case outlined in this document adheres to the following workflow:

1. User requests catalog item, which deploys a new Windows 2012 R2 Server
2. During the request process, a user is required to select which Rubrik SLA Domain to assign to the newly provisioned VM
3. The VM is provisioned according to vRA restrictions and/or reservations
4. A custom built vRealize Orchestrator (vRO) workflow is executed post-deployment, assigned the chosen SLA Domain Name to the newly provisioned VM

## Prerequisites
The following are the prerequisites in order to successfully implement the blueprint outlined in this guide:

* VMware vRealize Automation 7.3 or above
* [Rubrik Plugin for vRealize](https://github.com/rubrikinc/rubrik-vrealize)
* Rubrik CDM 4.1 or above

|**Note:** For more information on how to install and configure the Rubrik Plugin for vRealize see the [Rubrik Plugin for vRealize Quickstart on GitHub](https://github.com/rubrikinc/rubrik-vrealize).|

## Creating the Provision and Protect Catalog Item
This section will outline the steps to create a vRA blueprint which involves adding a catalog item for provisioning and protecting a VM. This process can be broken down into three major categories:

* Creating the appropriate Rubrik SLA Domain Property Definitions
* Creating or assigning the Rubrik SLA Domain Property Definitions to a new or existing vRA blueprint
* Creating the Workflow Subscription

### Creating Rubrik SLA Domain Property Definitions
When requiring custom information from an end user during a self-service request, vRA requires that it be delivered through a Property Definition. The following section walks through the steps to require an SLA Domain assignment during provisioning by creating two Property Definitions and group.

Login to vRA with a user assigned to the IaaS administrator role. Select **Administration**, the expand **Property Dictionary** selecting **Property Definition**s. Click **New** to add a new property definition.

![alt text](/images/image1.png)

The first **Property Definition** created will support the selection of a configured Rubrik cluster within vRO. Input the following information describing the property definition. Click **Select** to assign a vRO action to the property values.

* **Name**: `rubrik.cluster`
* **Label**: `Rubrik Cluster`
* **Display Order**: `1`
* **Data Type**: `String`
* **Required**: `Yes`
* **Display as**: `Dropdown`
* **Values**: `External values`

![alt text](/Provision and Protect/images/image2.png)

Expand the `com.rubrik.devops.actions` folder and select the `rubrik_GetRestHosts` action from the list. Click **OK** to continue.

![alt text](/Provision and Protect/images/image3.png)

Click **OK** to save the property definition

![alt text](/Provision and Protect/images/image4.png)

Click **New** to add a new property definition.

![alt text](/Provision and Protect/images/image5.png)

Input the following information describing the property definition. Click **Select** to assign a vRO action to the property values.

* **Name**: `rubrik.sla_name`
* **Label**: `SLA Domain Name`
* **Display Order**: `2`
* **Data Type**: `String`
* **Required**: `Yes`
* **Display as**: `Dropdown`
* **Values**: `External values`

![alt text](/Provision and Protect/images/image6.png)

Expand the `com.rubrik.devops.actions` folder and select the `rubrik_GetSlaList` action from the list. Click `OK` to continue

![alt text](/Provision and Protect/images/image7.png)

On the **Property Definition** screen, select **Edit** in the **Input parameters** section. Check the **Bind** checkbox and select `rubrik.cluster` from the dropdown for the value of the `rubrik_host` input parameter. This allows the SLA Domain list to be dynamically populated depending on the selected Rubrik cluster. Click **OK**.

![alt text](/Provision and Protect/images/image8.png)

Click **OK** to save the property definition

![alt text](/Provision and Protect/images/image9.png)

Log into vRA with a user assigned to the IaaS administrator role. Select **Administration**, the expand **Property Dictionary** selecting **Property Groups**. Click **New** to add a new property group.

![alt text](/Provision and Protect/images/image10.png)

Give the new property group a name and click **New** to add a new property to the group.

![alt text](/Provision and Protect/images/image11.png)

Create the following three properties in the group and click **OK** to save the property group:

* **Name**: `Extensibility.Lifecycle.Properties.VMPSMasterWorkflow32.MachineProvisioned`
* **Value**: `__*,*`
* **Encrypted**: `No`
* **Show in Request**: `No`
	
* **Name**: `rubrik.cluster`
* **Value**: 
* **Encrypted**: `No`
* **Show In Request**: `Yes`

* **Name**: `rubrik.sla_name`
* **Value**: 
* **Encrypted**: `No`
* **Show In Request**: `Yes`

![alt text](/Provision and Protect/images/image12.png)

The required property definitions and groups have been created. The next step is to create a blueprint and associate the property group to it.

### Creating a Provision and Protect Blueprint
The next step in the provision and protect use case is to create a blueprint which provisions a VM and sets the requirement of obtaining the Rubrik SLA Domain Name custom property definition. The following outlines the steps to create a provision and protect blueprint.

Log into vRA with credentials containing the privileges to create and publish blueprints. Select **Design**, then **Blueprints** and click **New** to create a new blueprint

![alt text](/Provision and Protect/images/image13.png)

On the **General** tab of the **New Blueprint** dialog, input a **Name** and **Description**. Select the **Properties** tab.

![alt text](/Provision and Protect/images/image14.png)

On the **Properties** tab click **Add** under the **Property Groups** section.  When prompted select the **Property Group** created in the previous section and click **OK** twice to proceed to the blueprint design canvas.

![alt text](/Provision and Protect/images/image15.png)

Drag the **vSphere (vCenter) Machine** entity from the left-hand listing onto the blueprint design canvas.

![alt text](/Provision and Protect/images/image16.png)

Set the desired information on the **General** tab. On the **Build Information** tab, configure the **Action** to `Clone` and select a valid template for the **Clone from** section. Click **Finish** to save the blueprint.

![alt text](/Provision and Protect/images/image17.png)

Select the blueprint from the list and click **Publish**.

![alt text](/Provision and Protect/images/image18.png)

The blueprint is now configured to require end users to select a Rubrik SLA Domain name during the provisioning requests. Ensure the blueprint and catalog item is configured with proper entitlements and services to provide access to the catalog. The final step is **Create a Workflow Subscription** to run the applicable vRO workflow to add the VM to an SLA Domain.

### Create a Workflow Subscription
A vRA Workflow Subscription provides a means to run various vRO workflows at specific states of the blueprints deployment. For example, a Workflow Subscription could trigger a vRO workflow which retrieves an IP address from an IPAM solution before the VM is provisioned.

In the Provision and Protect blueprint, a Workflow Subscription is required to trigger a vRO workflow which assigns the VM to an SLA Domain after the VM has been provisioned. The vRO workflow to accomplish this is supplied by Rubrik and named “Rubrik Post-Provisioning Workflow”. The workflow takes a VM and SLA Domain name as inputs and executes as follows:

1. Determine which vCenter Server instance is hosting the supplied VM
2. Refresh the vCenter Server inventory within Rubrik CDM so the VM is available for policy assignment
3. Adds the VM to the protected by specified SLA Domain 

This section will outline the steps required to create the required Workflow Subscription:

From the **Administrator** tab, select **Events**, then **Subscriptions**. Click **New** to create a new **Workflow Subscription**.

![alt text](/Provision and Protect/images/image19.png)

Select **Machine Provisioning** and click **Next**.

![alt text](/Provision and Protect/images/image20.png)

Select **Run based on conditions** then select **All** of the variables defined in the following table. Input the following three clauses and click **Next**.

|Clause
Operator
Value
Data > Machine > Machine Type
Equals
Constant > Virtual Machine
Data > Lifecycle state > Lifecycle state name
Equals
Constant > VMPSMasterWorkflow32.MachineProvisioned
Data > Lifecycle state > State phase
Equals
Constant > POST|

![alt text](/Provision and Protect/images/image21.png)

Expand the navigation tree to **Orchestrator** > **Library** > **Rubrik-DevOps** > **Helper Workflows** and select `Rubrik - Post-Provisioning Workflow` and click **Next**.

![alt text](/Provision and Protect/images/image22.png)

Give the **Workflow Subscription** a name and select **Finish** to save.

![alt text](/Provision and Protect/images/image23png)

Select the **Workflow Subscription** from the list and click **Publish** to publish the subscription.

![alt text](/Provision and Protect/images/image24.png)

The **Workflow Subscription** has now been configured. Users requesting the provision and protect blueprint will now be required to select an SLA Domain Name, which is automatically assigned to the provisioned VM.

## Requesting the Catalog Item
The following illustrates the steps required and processes executed in requesting the Provision and Protect catalog item within vRA.

Log into vRA with credentials which can access the catalog. From the **Catalog** tab, click **Request** on the created catalog item.

![alt text](/Provision and Protect/images/image25.png)

Complete the required information on the request form. The **SLA Domain Name** field is auto-populated with a list of SLA Domains residing selected Rubrik cluster. Select the desired **Rubrik Cluster** and **SLA Domain Name** and click **Submit**.

![alt text](/Provision and Protect/images/image26.png)

The VM provisioning process has begun. More information around the status can be viewed by clicking **In Progress**.

![alt text](/Provision and Protect/images/image27.png)

![alt text](/Provision and Protect/images/image28.png)

Once the provisioning process has finished the status of the vRO workflow to add the VM to an SLA Domain can be found by browsing **Library** > **Rubrik-DevOps** > **Helper Workflows** and selecting the associated run session from the **Workflows** tab. The workflow will be marked with either a green play icon indicating it is currently running, a red x icon indicating it has failed, or a green checkmark indicating it has successfully completed. 

![alt text](/Provision and Protect/images/image29.png)

Selecting the **Variables** tab shows the data that was passed from vRA to the vRO workflow.

![alt text](/Provision and Protect/images/image30.png)

Selecting the **Logs** tab shows the result from all portions of the `Rubrik - Post Provisioning Workflow` workflow.

![alt text](/Provision and Protect/images/image31.png)

The Rubrik CDM UI successfully shows that the newly provisioned virtual machine has been assigned the selected SLA Domain.

![alt text](/Provision and Protect/images/image32.png)


## Conclusion
Rubrik and vRA to work in harmony to provide complete automation around data management. Integrating Rubrik and vRA to build Provision and Protect blueprints delivers various self-serve options around backup and recovery to end-users, while still allowing IT teams to maintain the governance and control they require. For more information on the integration and architecture behind Rubrik and vRA see the Rubrik and vRA Reference Architecture.
