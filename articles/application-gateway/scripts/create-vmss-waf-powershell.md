---
title: Exemple de script Azure PowerShell - Limiter le trafic web | Microsoft Docs
description: Exemple de script Azure PowerShell - Créez une passerelle d’application avec un pare-feu d’applications web et un groupe de machines virtuelles identiques qui utilise des règles OWASP pour limiter le trafic.
services: application-gateway
documentationcenter: networking
author: vhorne
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 01/29/2018
ms.author: victorh
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: f20e5271cdca18ad40710e5defdf8175754ef891
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89078671"
---
# <a name="restrict-web-traffic-using-azure-powershell"></a>Limiter le trafic web avec Azure PowerShell

Ce script crée une passerelle d’application avec un pare-feu d’applications web qui utilise un groupe de machines virtuelles identiques pour des serveurs principaux. Le pare-feu d’applications web limite le trafic web en fonction de règles OWASP. Après avoir exécuté le script, vous pouvez tester la passerelle d’application à l’aide de son adresse IP publique.

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Exemple de script

[!code-powershell[main](../../../powershell_scripts/application-gateway/create-vmss/create-vmss-waf.ps1 "Create application gateway with WAF")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement 

Exécutez la commande suivante pour supprimer le groupe de ressources, la passerelle d’application et toutes les ressources associées.

```powershell
Remove-AzResourceGroup -Name myResourceGroupAG
```

## <a name="script-explanation"></a>Explication du script

Ce script a recours aux commandes suivantes pour créer le déploiement. Chaque élément du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crée la configuration de sous-réseau. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crée le réseau virtuel à l’aide des configurations de sous-réseau. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crée l’adresse IP publique pour la passerelle d’application. |
| [New-AzApplicationGatewayIPConfiguration](/powershell/module/az.network/new-azapplicationgatewayipconfiguration) | Crée la configuration qui associe un sous-réseau avec la passerelle d’application. |
| [New-AzApplicationGatewayFrontendIPConfig](/powershell/module/az.network/new-azapplicationgatewayfrontendipconfig) | Crée la configuration qui assigne une adresse IP publique à la passerelle d’application. |
| [New-AzApplicationGatewayFrontendPort](/powershell/module/az.network/new-azapplicationgatewayfrontendport) | Assigne un port à utiliser pour accéder à la passerelle d’application. |
| [New-AzApplicationGatewayBackendAddressPool](/powershell/module/az.network/new-azapplicationgatewaybackendaddresspool) | Crée un pool backend pour une passerelle d’application. |
| [New-AzApplicationGatewayBackendHttpSettings](/powershell/module/az.network/new-azapplicationgatewaybackendhttpsetting) | Configure les paramètres d’un pool backend. |
| [New-AzApplicationGatewayHttpListener](/powershell/module/az.network/new-azapplicationgatewayhttplistener) | Crée un écouteur. |
| [New-AzApplicationGatewayRequestRoutingRule](/powershell/module/az.network/new-azapplicationgatewayrequestroutingrule) | Crée une règle d’acheminement. |
| [New-AzApplicationGatewaySku](/powershell/module/az.network/new-azapplicationgatewaysku) | Spécifie le niveau et la capacité d’une passerelle d’application. |
| [New-AzApplicationGatewayWebApplicationFirewallConfiguration](/powershell/module/az.network/new-azapplicationgatewaywebapplicationfirewallconfiguration) | Crée la configuration du pare-feu d’application web |
| [New-AzApplicationGateway](/powershell/module/az.network/new-azapplicationgateway) | Créer une passerelle d’application |
| [Set-AzVmssStorageProfile](/powershell/module/az.compute/set-azvmssstorageprofile) | Crée un profil de stockage pour le groupe identique. |
| [Set-AzVmssOsProfile](/powershell/module/az.compute/set-azvmssosprofile) | Définit le système d’exploitation du groupe identique. |
| [Add-AzVmssNetworkInterfaceConfiguration](/powershell/module/az.compute/add-azvmssnetworkinterfaceconfiguration) | Définit l’interface réseau du groupe identique. |
| [New-AzVmss](/powershell/module/az.compute/new-azvm) | Crée un groupe de machines virtuelles identiques |
| [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) | Crée un compte de stockage. |
| [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) | Configure les diagnostics pour enregistrer les données. |
| [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) | Obtient l’adresse IP publique d’une passerelle d’application. |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources et toutes les ressources contenues. | 
## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le module Azure PowerShell, consultez [Documentation Azure PowerShell](/powershell/azure/).

Vous trouverez d’autres exemples de scripts PowerShell de passerelle d’application dans la [documentation sur Azure Application Gateway](../powershell-samples.md).
