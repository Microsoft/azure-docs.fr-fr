---
title: Installer l’agent Azure Monitor
description: Options d’installation de l’agent de Azure Monitor (AMA) sur les machines virtuelles Azure et les serveurs Azure Arc.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/17/2020
ms.openlocfilehash: 893a04f5acd48cf1e6f34033c06a758492e70abf
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516718"
---
# <a name="install-the-azure-monitor-agent-preview"></a>Installer l’agent Azure Monitor (préversion)
Cet article présente les différentes options actuellement disponibles pour installer l’[agent Azure Monitor](azure-monitor-agent-overview.md) sur les machines virtuelles Azure et les serveurs Azure Arc, ainsi que les options permettant de créer des [associations avec des règles de collecte de données](data-collection-rule-azure-monitor-agent.md) qui définissent les données que l’agent doit collecter.

## <a name="prerequisites"></a>Prérequis
Les conditions préalables suivantes sont requises avant l’installation de l’agent Azure Monitor.

- L’[identité du système managé](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md) doit être activée sur les machines virtuelles Azure. Cela n’est pas nécessaire pour les serveurs Azure Arc. L’identité système est activée automatiquement si l’agent est installé dans le cadre du processus de [création et d’affectation d’une règle de collecte de données à l’aide du portail Azure](#install-with-azure-portal).
- L’[étiquette de service AzureResourceManager](../../virtual-network/service-tags-overview.md) doit être activée sur le réseau virtuel pour la machine virtuelle.

## <a name="virtual-machine-extension-details"></a>Détails de l’extension de machine virtuelle
L’agent Azure Monitor est implémenté en tant qu’[extension de machine virtuelle Azure](../../virtual-machines/extensions/overview.md) avec les détails dans le tableau suivant. Il peut être installé à l’aide de l’une des méthodes d’installation des extensions de machine virtuelle, notamment celles décrites dans cet article.

| Propriété | Windows | Linux |
|:---|:---|:---|
| Serveur de publication | Microsoft.Azure.Monitor  | Microsoft.Azure.Monitor |
| Type      | AzureMonitorWindowsAgent | AzureMonitorLinuxAgent  |
| TypeHandlerVersion  | 1.0 | 1.5 |


## <a name="install-with-azure-portal"></a>Installer avec Portail Azure
Pour installer l’agent Azure Monitor à l’aide du portail Azure, suivez le processus de [création d’une règle de collecte de données](data-collection-rule-azure-monitor-agent.md#create-rule-and-association-in-azure-portal) dans le portail Azure. Cela vous permet d’associer la règle de collecte de données à au moins une machine virtuelle Azure ou à des serveurs Azure Arc. L’agent est installé sur l’une de ces machines virtuelles sur laquelle il n’est pas encore installé.


## <a name="install-with-resource-manager-template"></a>Installer avec un modèle Resource Manager
Vous pouvez utiliser des modèles Resource Manager pour installer l’agent Azure Monitor sur des machines virtuelles Azure et sur des serveurs Azure Arc, ainsi que pour créer une association avec des règles de collecte de données. Vous devez créer une règle de collecte des données avant de créer l’association.

Obtenez des exemples de modèles pour l’installation de l’agent et la création de l’association à partir des éléments suivants : 

- [Modèle d’installation de l’agent Azure Monitor (Azure et Azure Arc)](../samples/resource-manager-agent.md#azure-monitor-agent-preview) 
- [Modèle de création d’une association avec une règle de collecte de données](../samples/resource-manager-data-collection-rules.md)

Installez les modèles à l’aide de [n’importe quelle méthode de déploiement pour les modèles Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md) comme les commandes suivantes.

# <a name="powershell"></a>[PowerShell](#tab/ARMAgentPowerShell)
```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "<template-filename.json>" -TemplateParameterFile "<parameter-filename.json>"
```
# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/ARMAgentCLI)
```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "<template-filename.json>" -TemplateParameterFile "<parameter-filename.json>"
```
---

## <a name="install-with-powershell"></a>Installer avec PowerShell
Vous pouvez installer l’agent Azure Monitor sur des machines virtuelles Azure et sur des serveurs Azure Arc à l’aide de la commande PowerShell pour ajouter une extension de machine virtuelle. 

### <a name="azure-virtual-machines"></a>Machines virtuelles Azure
Utilisez les commandes PowerShell suivantes pour installer l’agent Azure Monitor sur des machines virtuelles Azure.
# <a name="windows"></a>[Windows](#tab/PowerShellWindows)
```powershell
Set-AzVMExtension -Name AMAWindows -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -VMName <virtual-machine-name> -Location <location>
```
# <a name="linux"></a>[Linux](#tab/PowerShellLinux)
```powershell
Set-AzVMExtension -Name AMALinux -ExtensionType AzureMonitorLinuxAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -VMName <virtual-machine-name> -Location <location>
```
---

### <a name="azure-arc-enabled-servers"></a>Serveurs avec Azure Arc
Utilisez les commandes PowerShell suivantes pour installer l’agent Azure Monitor sur des serveurs Azure Arc.
# <a name="windows"></a>[Windows](#tab/PowerShellWindowsArc)
```powershell
New-AzConnectedMachineExtension -Name AMAWindows -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -MachineName <virtual-machine-name> -Location <location>
```
# <a name="linux"></a>[Linux](#tab/PowerShellLinuxArc)
```powershell
New-AzConnectedMachineExtension -Name AMALinux -ExtensionType AzureMonitorLinuxAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -MachineName <virtual-machine-name> -Location <location>
```
---
## <a name="azure-cli"></a>Azure CLI
Vous pouvez installer l’agent Azure Monitor sur des machines virtuelles Azure et sur des serveurs Azure Arc à l’aide de la commande Azure CLI pour ajouter une extension de machine virtuelle. 

### <a name="azure-virtual-machines"></a>Machines virtuelles Azure
Utilisez les commandes CLI suivantes pour installer l’agent Azure Monitor sur des machines virtuelles Azure.
# <a name="windows"></a>[Windows](#tab/CLIWindows)
```azurecli
az vm extension set --name AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
# <a name="linux"></a>[Linux](#tab/CLILinux)
```azurecli
az vm extension set --name AzureMonitorLinuxAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
---
### <a name="azure-arc-enabled-servers"></a>Serveurs avec Azure Arc
Utilisez les commandes CLI suivantes pour installer l’agent Azure Monitor sur des serveurs Azure Arc.

# <a name="windows"></a>[Windows](#tab/CLIWindowsArc)
```azurecli
az connectedmachine machine-extension create --name AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
# <a name="linux"></a>[Linux](#tab/CLILinuxArc)
```azurecli
az connectedmachine machine-extension create --name AzureMonitorLinuxAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
---


## <a name="next-steps"></a>Étapes suivantes

- [Créez une règle de collecte de données](data-collection-rule-azure-monitor-agent.md) pour collecter des données à partir de l’agent et les envoyer à Azure Monitor.
