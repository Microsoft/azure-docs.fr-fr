---
title: Obtenir des notifications de maintenance de machines virtuelles Azure à l’aide de PowerShell
description: Affichez les notifications de maintenance pour les machines virtuelles s’exécutant dans Azure et commencez la maintenance en libre-service à l’aide de PowerShell.
author: shants123
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/19/2019
ms.author: shants
ms.openlocfilehash: 9942ad359bf1e1e7a02f30979ba4a9325d90d484
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91336052"
---
# <a name="handling-planned-maintenance-using-powershell"></a>Gestion de la maintenance planifiée à l’aide de PowerShell

**Cet article s’applique aux machines virtuelles exécutant Linux et Windows.**

Vous pouvez utiliser Azure PowerShell pour savoir quand la [maintenance](maintenance-notifications.md) des machines virtuelles a été planifiée. Les informations de maintenance planifiée sont disponibles à partir de la cmdlet [Get-AzVM](/powershell/module/az.compute/get-azvm) lorsque vous utilisez le paramètre `-status`.
  
Les informations de maintenance sont retournées uniquement en cas de maintenance planifiée. Si aucune maintenance planifiée n’impacte la machine virtuelle, l’applet de commande ne retourne pas d’informations de maintenance. 


```powershell
Get-AzVM -ResourceGroupName myResourceGroup -Name myVM -Status
```

Sortie

```
MaintenanceRedeployStatus               : 
  IsCustomerInitiatedMaintenanceAllowed : True
  PreMaintenanceWindowStartTime         : 5/14/2018 12:30:00 PM
  PreMaintenanceWindowEndTime           : 5/19/2018 12:30:00 PM
  MaintenanceWindowStartTime            : 5/21/2018 4:30:00 PM
  MaintenanceWindowEndTime              : 6/4/2018 4:30
  LastOperationResultCode               : None 
```

Les propriétés suivantes sont retournées sous MaintenanceRedeployStatus : 

| Valeur | Description   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | Indique si vous pouvez démarrer la maintenance sur la machine virtuelle maintenant |
| PreMaintenanceWindowStartTime         | Début de la fenêtre de maintenance en libre-service lorsque vous pouvez lancer la maintenance sur votre machine virtuelle |
| PreMaintenanceWindowEndTime           | Fin de la fenêtre de maintenance en libre-service lorsque vous pouvez lancer la maintenance sur votre machine virtuelle |
| MaintenanceWindowStartTime            | Début de la maintenance planifiée pendant laquelle Azure lance la maintenance sur votre machine virtuelle |
| MaintenanceWindowEndTime              | Fin de la fenêtre de maintenance planifiée pendant laquelle Azure lance la maintenance sur votre machine virtuelle |
| LastOperationResultCode               | Résultat de la dernière tentative de lancement de la maintenance sur la machine virtuelle |



Vous pouvez également obtenir l'état de maintenance de toutes les machines virtuelles d'un groupe de ressources à l'aide de [Get-AzVM](/powershell/module/az.compute/get-azvm) sans spécifier de machine virtuelle.
 
```powershell
Get-AzVM -ResourceGroupName myResourceGroup -Status
```

L’exemple PowerShell suivant prend votre ID d’abonnement et retourne la liste des machines virtuelles dont la maintenance est planifiée.

```powershell

function MaintenanceIterator
{
    Select-AzSubscription -SubscriptionId $args[0]

    $rgList= Get-AzResourceGroup 

    for ($rgIdx=0; $rgIdx -lt $rgList.Length ; $rgIdx++)
    {
        $rg = $rgList[$rgIdx]        
    $vmList = Get-AzVM -ResourceGroupName $rg.ResourceGroupName 
        for ($vmIdx=0; $vmIdx -lt $vmList.Length ; $vmIdx++)
        {
            $vm = $vmList[$vmIdx]
            $vmDetails = Get-AzVM -ResourceGroupName $rg.ResourceGroupName -Name $vm.Name -Status
              if ($vmDetails.MaintenanceRedeployStatus )
            {
                Write-Output "VM: $($vmDetails.Name)  IsCustomerInitiatedMaintenanceAllowed: $($vmDetails.MaintenanceRedeployStatus.IsCustomerInitiatedMaintenanceAllowed) $($vmDetails.MaintenanceRedeployStatus.LastOperationMessage)"               
            }
          }
    }
}

```

### <a name="start-maintenance-on-your-vm-using-powershell"></a>Démarrer la maintenance sur votre machine virtuelle à l’aide de PowerShell

À l’aide des informations de la fonction de la section précédente, la commande suivante démarre la maintenance sur une machine virtuelle si **IsCustomerInitiatedMaintenanceAllowed** est défini sur true.

```powershell
Restart-AzVM -PerformMaintenance -name $vm.Name -ResourceGroupName $rg.ResourceGroupName 
```

## <a name="classic-deployments"></a>Déploiements Classic

[!INCLUDE [classic-vm-deprecation](../../includes/classic-vm-deprecation.md)]

Si vous avez encore des machines virtuelles qui ont été déployées à l’aide du modèle de déploiement Classic, vous pouvez utiliser PowerShell pour interroger ces machines virtuelles et démarrer la maintenance.

Pour obtenir l’état de la maintenance d’une machine virtuelle, tapez :

```
Get-AzureVM -ServiceName <Service name> -Name <VM name>
```

Pour démarrer la maintenance sur votre machine virtuelle classique, tapez :

```
Restart-AzureVM -InitiateMaintenance -ServiceName <service name> -Name <VM name>
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également gérer la maintenance planifiée à l’aide d’[Azure CLI](maintenance-notifications-cli.md) ou du [portail](maintenance-notifications-portal.md).
