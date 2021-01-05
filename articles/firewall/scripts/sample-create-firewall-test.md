---
title: Créer un environnement de test pour Pare-feu Azure
description: Cet exemple de script crée un pare-feu et un environnement réseau de test. Le réseau a un réseau virtuel avec trois sous-réseaux.
services: virtual-network
author: vhorne
ms.service: firewall
ms.devlang: powershell
ms.topic: sample
ms.date: 11/19/2019
ms.author: victorh
ms.openlocfilehash: 4158bc07373a2d0aa6fb6ceaf2dce62b50bb6bd7
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658364"
---
# <a name="create-an-azure-firewall-test-environment"></a>Créer un environnement de test pour Pare-feu Azure

Cet exemple de script crée un pare-feu et un environnement réseau de test. Le réseau dispose d’un réseau virtuel, doté de trois sous-réseaux : *AzureFirewallSubnet*, *ServersSubnet* et *JumpboxSubnet*. ServersSubnet et JumpboxSubnet sont chacun équipés d’un serveur Windows Server à 2 cœurs.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Le pare-feu se trouve dans AzureFirewallSubnet et est configuré à l’aide d’une collection de règles d’application avec une seule règle qui autorise l’accès à `www.microsoft.com`.

Un itinéraire défini par l’utilisateur est créé : à partir de ServersSubnet, il pointe en direction du trafic réseau via le pare-feu, où les règles de pare-feu sont appliquées.

Vous pouvez exécuter le script à partir d’Azure [Cloud Shell](https://shell.azure.com/powershell) ou à partir d’une installation PowerShell locale. 

Si vous utilisez PowerShell en local, ce script requiert Azure PowerShell. Pour trouver la version installée, exécutez `Get-Module -ListAvailable Az`. 

Si vous devez procéder à une mise à niveau, vous pouvez utiliser `PowerShellGet`, qui est intégré à Windows 10 et Windows Server 2016.

> [!NOTE]
>Les autres versions de Windows vous obligent à installer `PowerShellGet` avant de pouvoir les utiliser. Vous pouvez exécuter `Get-Module -Name PowerShellGet -ListAvailable | Select-Object -Property Name,Version,Path` pour déterminer s’il est installé sur votre système. Si la sortie est vide, vous devez installer la dernière version de [Windows Management Framework](https://www.microsoft.com/download/details.aspx?id=54616).

Pour plus d’informations, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/install-Az-ps).

Toute installation d’Azure PowerShell existante effectuée avec l’outil Web Platform Installer sera en conflit avec l’installation de PowerShellGet et devra être supprimée.

Rappelez-vous que si vous exécutez PowerShell en local, vous devez également exécuter `Connect-AzAccount` pour créer une connexion avec Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Exemple de script


[!code-azurepowershell-interactive[main](../../../powershell_scripts/firewall/create-fw-test.ps1  "Create a firewall test environment")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement 

Exécutez la commande suivante pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées :

```powershell
Remove-AzResourceGroup -Name AzfwSampleScriptEastUS -Force
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes pour créer un groupe de ressources, un réseau virtuel et les groupes de sécurité réseau. Chaque commande du tableau suivant renvoie à une documentation spécifique :

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crée un objet de configuration de sous-réseau. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crée un réseau virtuel et un sous-réseau frontal Azure. |
| [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) | Crée des règles de sécurité à attribuer à un groupe de sécurité réseau. |
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) |Crée des règles NSG qui autorisent ou bloquent des ports spécifiques sur des sous-réseaux donnés. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Associe les groupes de sécurité réseau à des sous-réseaux. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crée une adresse IP publique pour accéder à la machine virtuelle à partir d’Internet. |
| [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) | Crée des interfaces réseau virtuelles et les attache aux sous-réseaux frontaux et principaux du réseau virtuel. |
| [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig) | Crée une configuration de machine virtuelle. Cette configuration inclut des informations telles que le nom de la machine virtuelle, le système d’exploitation et les informations d’identification d’administration. La configuration est utilisée lors de la création de machines virtuelles. |
| [New-AzVM](/powershell/module/az.compute/new-azvm) | Création d’une machine virtuelle |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources et toutes les ressources contenues. |
|[New-AzFirewall](/powershell/module/az.network/new-azfirewall)| Crée un nouveau pare-feu Azure.|
|[Get-AzFirewall](/powershell/module/az.network/get-azfirewall)|Obtient un objet de pare-feu Azure.|
|[New-AzFirewallApplicationRule](/powershell/module/az.network/new-azfirewallapplicationrule)|Crée une nouvelle règle d’application de pare-feu Azure.|
|[Set-AzFirewall](/powershell/module/az.network/set-azfirewall)|Valide les modifications apportées à l’objet de pare-feu Azure.|

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure PowerShell, consultez la [documentation Azure PowerShell](/powershell/azure/).