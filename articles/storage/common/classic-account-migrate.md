---
title: How to migrate your classic storage accounts to Azure Resource Manager
titleSuffix: Azure Storage
description: Learn how to migrate your classic storage accounts to the Azure Resource Manager deployment model. All classic accounts must be migrated by August 31, 2024.
services: storage
author: tamram

ms.service: storage
ms.topic: how-to
ms.date: 04/06/2023
ms.author: tamram
ms.subservice: common
---

# How to migrate your classic storage accounts to Azure Resource Manager

Microsoft will retire classic storage accounts on August 31, 2024. To preserve the data in any classic storage accounts, you must migrate them to the Azure Resource Manager deployment model by that date. After you migrate your account, all of the benefits of the Azure Resource Manager deployment model will be available for that account. For more information about the deployment models, see [Resource Manager and classic deployment](../../azure-resource-manager/management/deployment-models.md).

This article describes how to migrate your classic storage accounts to the Azure Resource Manager deployment model. For more information, see [Migrate your classic storage accounts to Azure Resource Manager by August 31, 2024](classic-account-migration-overview.md).

## Identify classic storage accounts in your subscription

# [Portal](#tab/azure-portal)

To list classic storage accounts in your subscription with the Azure portal:

1. Navigate to the list of your storage accounts in the Azure portal.
1. Select **Add filter**. In the **Filter** dialog, set the **Filter** field to **Type** and the **Operator** field to **Equals**. Then set the **Value** field to **microsoft.classicstorage/storageaccounts**.

    :::image type="content" source="media/classic-account-migrate/classic-accounts-list-portal.png" alt-text="Screenshot showing how to list classic storage accounts in Azure portal." lightbox="media/classic-account-migrate/classic-accounts-list-portal.png":::

# [PowerShell](#tab/azure-powershell)

To list classic storage accounts in your subscription with PowerShell, run the following command:

```azurepowershell
Get-AzResource -ResourceType Microsoft.ClassicStorage/storageAccounts
```

---

## Migrate a classic storage account

The process of migrating a classic storage account involves four steps:

1. **Validate**. During the Validation phase, Azure checks the storage account to ensure that it can be migrated.
1. **Prepare**. In the Prepare phase, Azure creates a new general-purpose v1 storage account and alerts you to any problems that may have occurred. The new account is created in a new resource group in the same region as your classic account.

    At this point your classic storage account still exists. If there are any problems reported, you can correct them or abort the process.

1. **Check manually**. It's a good idea to make a manual check of the new storage account to make sure that the output is as you expect.
1. **Commit or abort**. If you are satisfied that the migration has been successful, then you can commit the migration. Committing the migration permanently deletes the classic storage account.

    If there are any problems with the migration, then you can abort the migration at this point. If you choose to abort, the new resource group and new storage account are deleted. Your classic account remains available. You can address any problems and attempt the migration again.

For more information about the migration process, see [Understand storage account migration from the classic deployment model to Azure Resource Manager](classic-account-migration-process.md)

You can migrate a classic storage account to the Azure Resource Manager deployment model with the Azure portal or PowerShell.

# [Portal](#tab/azure-portal)

To migrate a classic storage account to the Azure Resource Manager deployment model with the Azure portal:

1. Navigate to your classic storage account in the Azure portal.
1. In the **Settings** section, select **Migrate to ARM**.
1. Select **Validate** to determine migration feasibility.

   :::image type="content" source="./media/classic-account-migrate/validate-storage-account.png" alt-text="Screenshot showing how to migrate your classic storage account to Azure Resource Manager." lightbox="./media/classic-account-migrate/validate-storage-account.png":::

1. After a successful validation, select **Prepare** button to simulate the migration.

  > [!IMPORTANT]
  > There may be a delay of a few minutes after validation is complete before the Prepare button is enabled.

1. If the Prepare step completes successfully, you'll see a link to the new resource group. Select that link to navigate to the new resource group. The migrated storage account appears under the **Resources** tab in the **Overview** page for the new resource group.

    At this point you can compare the configuration and data in the classic storage account to the newly migrated storage account. You'll see both in the list of storage accounts in the portal. Both the classic account and the migrated account have the same name.

    :::image type="content" source="media/classic-account-migrate/compare-classic-migrated-accounts.png" alt-text="Screenshot showing the results of the Prepare step in the Azure portal." lightbox="media/classic-account-migrate/compare-classic-migrated-accounts.png":::

1. If you're not satisfied with the results of the migration, select **Abort** to delete the new storage account and resource group. You can then address any problems and try again.
1. When you're ready to commit, type **yes** to confirm, then select **Commit** to complete the migration.

# [PowerShell](#tab/azure-powershell)

To migrate a classic storage account to the Azure Resource Manager deployment model with PowerShell, you'll need to use the Azure PowerShell Service Management module. To learn how to install this module, see [Install and configure the Azure PowerShell Service Management module](/powershell/azure/servicemanagement/install-azure-ps#checking-the-version-of-azure-powershell). The key steps are included here for convenience.

> [!NOTE]
> The cmdlets in the Azure Service Management module are for managing legacy Azure resources that use Service Management APIs, including classic storage accounts. This module includes the commands needed to migrate a classic storage account to Azure Resource Manager.
>
> To manage Azure Resource Manager resources, we recommend that you use the Az PowerShell module. The Az module replaces the deprecated AzureRM module. For more information about moving from the AzureRM module to the Az module, see [Migrate Azure PowerShell scripts from AzureRM to Az](/powershell/azure/migrate-from-azurerm-to-az).

First, install PowerShellGet if you do not already have it installed. For more information on how to install PowerShellGet, see [Installing PowerShellGet](/powershell/scripting/gallery/installing-psget#installing-the-latest-version-of-powershellget). After you install PowerShellGet, close and reopen the PowerShell console.

Next, install the Azure Service Management module. If you also have the AzureRM module installed, you'll need to include the `-AllowClobber` parameter, as described in [Step 2: Install Azure PowerShell](/powershell/azure/servicemanagement/install-azure-ps#step-2-install-azure-powershell). After the installation is complete, import the Azure Service Management module.

```azurepowershell
Install-Module -Name Azure -AllowClobber
Import-Module -Name Azure
```

Log into your account by running [Add-AzureAccount](/powershell/module/servicemanagement/azure/add-azureaccount). Then call [Select-AzureSubscription](/powershell/module/servicemanagement/azure/select-azuresubscription) to set your current subscription. Remember to replace the placeholder values in brackets with your subscription name:

```azurepowershell
Add-AzureAccount
Select-AzureSubscription -SubscriptionName <subscription-name>
```

To validate that the account is ready for migration, call the [Move-AzureStorageAccount](/powershell/module/servicemanagement/azure/move-azurestorageaccount) command. Remember to replace the placeholder values in brackets with the name of your classic storage account:

```azurepowershell
$accountName = "<storage-account>"
Move-AzureStorageAccount -Validate -StorageAccountName $accountName
```

Next, prepare the account for migration:

```azurepowershell
Move-AzureStorageAccount -Prepare -StorageAccountName $accountName
```

Check the configuration for the prepared storage account with either Azure PowerShell or the Azure portal. If you're not ready for migration, use the following command to revert your account to its previous state:

```azurepowershell
Move-AzureStorageAccount -Abort -StorageAccountName $accountName
```

Finally, when you are satisfied with the prepared configuration, move forward with the migration and commit the resources with the following command:

```azurepowershell
Move-AzureStorageAccount -Commit -StorageAccountName $accountName
```

---

### Locate and delete disk artifacts in a classic account

Classic storage accounts may contain classic (unmanaged) disks, virtual machine images, and operating system (OS) images. To migrate the account, you may need to delete these artifacts first.

To delete disk artifacts from the Azure portal, follow these steps:

1. Navigate to the Azure portal.
1. In the **Search** bar at the top, search for **Disks (classic)**, **OS Images (classic)**, or **VM Images (classic)** to display classic disk artifacts.
1. Locate the classic disk artifact to delete, and select it to view its properties.
1. Select the **Delete** button to delete the disk artifact.

    :::image type="content" source="media/classic-account-migrate/delete-disk-artifacts-portal.png" alt-text="Screenshot showing how to delete classic disk artifacts in Azure portal." lightbox="media/classic-account-migrate/delete-disk-artifacts-portal.png":::

For more information about errors that may occur when deleting disk artifacts and how to address them, see [Troubleshoot errors when you delete Azure classic storage accounts, containers, or VHDs](/troubleshoot/azure/virtual-machines/storage-classic-cannot-delete-storage-account-container-vhd).

## See also

- [Migrate your classic storage accounts to Azure Resource Manager by August 31, 2024](classic-account-migration-overview.md)
- [Understand storage account migration from the classic deployment model to Azure Resource Manager](classic-account-migration-process.md)
