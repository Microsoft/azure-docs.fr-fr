---
title: Script PowerShell - Trouver le coffre d’un compte de stockage
description: Découvrez comment utiliser un script Azure PowerShell pour rechercher le coffre Recovery Services dans lequel votre compte de stockage est inscrit.
ms.topic: sample
ms.date: 1/28/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 40859c1ea05210d27fcdcf33ba9d4f961965ea22
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89075696"
---
# <a name="powershell-script-to-find-the-recovery-services-vault-where-a-storage-account-is-registered"></a>Script PowerShell permettant de rechercher le coffre Recovery Services dans lequel un compte de stockage est inscrit

Ce script vous aide à rechercher le coffre Recovery Services dans lequel votre compte de stockage est inscrit.

## <a name="sample-script"></a>Exemple de script

```powershell
Param(
        [Parameter(Mandatory=$True)][System.String] $ResourceGroupName,
        [Parameter(Mandatory=$True)][System.String] $StorageAccountName,
        [Parameter(Mandatory=$True)][System.String] $SubscriptionId
    )

Connect-AzAccount
Select-AzSubscription -Subscription $SubscriptionId
$vaults = Get-AzRecoveryServicesVault
$found = $false
foreach($vault in $vaults)
{
  Write-Verbose "Checking vault: $($vault.Id)" -Verbose
  
  $containers = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -FriendlyName $StorageAccountName -ResourceGroupName $ResourceGroupName -VaultId $vault.Id -Status Registered
  
  if($containers -ne $null)
  {
    $found = $True
    Write-Information "Found Storage account $StorageAccountName registered in vault: $($vault.Id)" -InformationAction Continue
    break;
  }
}

if(!$found)
{
     Write-Information "Storage account: $StorageAccountName is not registered in any vault of this subscription" -InformationAction Continue
}
```

## <a name="how-to-execute-the-script"></a>Comment exécuter le script

1. Enregistrez le script ci-dessus sur votre ordinateur avec le nom de votre choix. Dans cet exemple, nous l’avons enregistré sous le nom *FindRegisteredStorageAccount.ps1*.
2. Exécutez le script en indiquant les paramètres suivants :

    * **-ResourceGroupName** : groupe de ressources du compte de stockage
    * **-StorageAccountName** : nom du compte de stockage
    * **-SubscriptionID** : ID de l’abonnement dans lequel le compte de stockage est présent

L’exemple suivant tente de trouver le coffre Recovery Services dans lequel le compte de stockage *afsaccount* est inscrit :

```powershell
.\FindRegisteredStorageAccount.ps1 -ResourceGroupName AzureFiles -StorageAccountName afsaccount -SubscriptionId ef4ad5a7-c2c0-4304-af80-af49f49af3d1
```

## <a name="output"></a>Output

La sortie affiche le chemin d’accès complet du coffre Recovery Services dans lequel le compte de stockage est inscrit. Voici un exemple de sortie :

```output
Found Storage account afsaccount registered in vault: /subscriptions/ ef4ad5a7-c2c0-4304-af80-af49f49af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault123
```

## <a name="next-steps"></a>Étapes suivantes

Découvrir comment [sauvegarder des partages de fichiers Azure à partir du portail Azure](../backup-afs.md)
