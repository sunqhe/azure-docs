---
title: Use the Azure CLI to create a policy assignment to identify non-compliant resources in your Azure environment | Microsoft Docs
description: Use PowerShell to create an Azure Policy assignment to identify non-compliant resources.
services: azure-policy
keywords:
author: bandersmsft
ms.author: banders
ms.date: 03/13/2018
ms.topic: quickstart
ms.service: azure-policy
ms.custom: mvc
---

# Create a policy assignment to identify non-compliant resources in your Azure environment with the Azure CLI

The first step in understanding compliance in Azure is to identify the status of your resources. This quickstart steps you through the process of creating a policy assignment to identify virtual machines that aren't using managed disks.

At the end of this process, you'll successfully identify virtual machines that aren't using managed disks. They're *non-compliant* with the policy assignment.

Azure CLI is used to create and manage Azure resources from the command line or in scripts. This guide uses Azure CLI to create a policy assignment and to identify non-compliant resources in your Azure environment.

If you don't have an Azure subscription, create a [free](https://azure.microsoft.com/free/) account before you begin.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

This quickstart requires that you run Azure CLI version 2.0.4 or later to install and use the CLI locally. To find the version, run `az --version`. If you need to install or upgrade, see [Install Azure CLI 2.0]( /cli/azure/install-azure-cli).



## Create a policy assignment

In this quickstart, you create a policy assignment and assign the Audit Virtual Machines without Managed Disks definition. This policy definition identifies resources that don't comply with the conditions set in the policy definition.

Run the following command to create a policy assignment:

```
az policy assignment create --name 'Audit Virtual Machines without Managed Disks Assignment' --scope '<scope>' --policy '<policy definition ID>' --sku 'standard'
```

The preceding command uses the following information:

- **Name** - Display name for the policy assignment. In this case, you're using *Audit Virtual Machines without Managed Disks Assignment*.
- **Policy** – The policy definition ID, based on which you're using to create the assignment. In this case, it is the policy definition – *Audit Virtual Machines without Managed Disks*. To get the policy definition ID, run this command:
        `az policy definition show --name 'Audit Virtual Machines without Managed Disks Assignment'`
- **Scope** - A scope determines what resources or grouping of resources the policy assignment gets enforced on. It could range from a subscription to resource groups. Be sure to replace &lt;scope&gt; with the name of your resource group.
- **Sku** – This command creates a policy assignment with the standard tier. The standard tier enables you to achieve at-scale management, compliance evaluation, and remediation. For additional details about pricing tiers, see [Azure Policy pricing](https://azure.microsoft.com/pricing/details/azure-policy).


## Identify non-compliant resources

To view the resources that aren't compliant under this new assignment, get the policy assignment ID by running the following commands:

```
$policyAssignment = Get-AzureRmPolicyAssignment | where {$_.properties.displayName -eq "Audit Virtual Machines without Managed Disks"}
```

```
$policyAssignment.PolicyAssignmentId
```

For more information about policy assignment IDs, see [Get-AzureRMPolicyAssignment](/powershell/module/azurerm.resources/get-azurermpolicyassignment).

Next, run the following command to get the resource IDs of the non-compliant resources that are output into a JSON file:

```
armclient post "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2017-12-12-preview&$filter=IsCompliant eq false and PolicyAssignmentId eq '<policyAssignmentID>'&$apply=groupby((ResourceId))" > <json file to direct the output with the resource IDs into>
```

Your results resemble the following example:

```
{
"@odata.context":"https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
"@odata.count": 3,
"value": [
{
    "@odata.id": null,
    "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
      "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachineId>"
    },
    {
      "@odata.id": null,
      "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
      "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine2Id>"
   		 },
{
      "@odata.id": null,
      "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
      "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine3ID>"
   		 }

]
}

```

The results are comparable to what you'd typically see listed under **Non-compliant resources** in the Azure portal view.

## Clean up resources

Other guides in this collection build upon this quickstart. If you plan to continue to work with later tutorials, don't clean up the resources created in this quickstart. If you don't plan to continue, delete the assignment you created by running the following command:

```azurecli
az policy assignment delete –name Audit Virtual Machines without Managed Disks Assignment --scope /subscriptions/ <subscriptionID> / <resourceGroupName>
```

## Next steps

In this quickstart, you assigned a policy definition to identify non-compliant resources in your Azure environment.

To learn more about assigning policies and ensure that resources you create in the **future** are compliant, continue to the tutorial for:

> [!div class="nextstepaction"]
> [Creating and managing policies](./create-manage-policy.md)
