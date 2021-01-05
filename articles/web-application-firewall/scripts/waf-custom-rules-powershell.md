---
title: Exemple de script Azure PowerShell utilisant des règles WAF personnalisées
description: 'Échantillon de script Azure PowerShell : créer des règles personnalisées de pare-feu d’applications web sur Application Gateway'
author: vhorne
ms.service: web-application-firewall
services: web-application-firewall
ms.topic: sample
ms.date: 09/30/2019
ms.author: victorh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8d800e77ed0e609091d31d9b3177af447ac06f5b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89079504"
---
# <a name="create-waf-custom-rules-with-azure-powershell"></a>Créer des règles de pare-feu d’applications web personnalisées avec Azure PowerShell

Ce script crée un pare-feu d’applications web Application Gateway qui utilise des règles personnalisées. Les règles personnalisées bloquent le trafic si l’en-tête de la requête contient User-Agent *evilbot*.

## <a name="prerequisites"></a>Conditions préalables requises

### <a name="azure-powershell-module"></a>Module Azure PowerShell

Si vous choisissez d’installer et d’utiliser Azure PowerShell en local, ce script exige le module Azure PowerShell version 2.1.0 ou ultérieure.

1. Pour connaître la version de l’interface, exécutez `Get-Module -ListAvailable Az`. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).
2. Pour créer une connexion avec Azure, exécutez `Connect-AzAccount`.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Exemple de script

[!code-powershell[main](../../../powershell_scripts/application-gateway/waf-rules/waf-custom-rules.ps1 "Custom WAF rules")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Exécutez la commande suivante pour supprimer le groupe de ressources, la passerelle d’application et toutes les ressources associées.

```powershell
Remove-AzResourceGroup -Name CustomRulesTest
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
| [New-AzApplicationGateway](/powershell/module/az.network/new-azapplicationgateway) | Créer une passerelle d’application |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources et toutes les ressources contenues. |
|[New-AzApplicationGatewayAutoscaleConfiguration](/powershell/module/az.network/New-AzApplicationGatewayAutoscaleConfiguration)|Crée une configuration de mise à l’échelle automatique pour Application Gateway.|
|[New-AzApplicationGatewayFirewallMatchVariable](/powershell/module/az.network/New-AzApplicationGatewayFirewallMatchVariable)|Crée une variable de correspondance pour la condition de pare-feu.|
|[New-AzApplicationGatewayFirewallCondition](/powershell/module/az.network/New-AzApplicationGatewayFirewallCondition)|Crée une condition de correspondance pour la règle personnalisée.|
|[New-AzApplicationGatewayFirewallCustomRule](/powershell/module/az.network/New-AzApplicationGatewayFirewallCustomRule)|Crée une règle personnalisée pour la stratégie de pare-feu de la passerelle d’application.|
|[New-AzApplicationGatewayFirewallPolicy](/powershell/module/az.network/New-AzApplicationGatewayFirewallPolicy)|Crée une stratégie de pare-feu de passerelle d’application.|
|[New-AzApplicationGatewayWebApplicationFirewallConfiguration](/powershell/module/az.network/New-AzApplicationGatewayWebApplicationFirewallConfiguration)|Créer un configuration WAF pour une passerelle d’application.|

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les règles de pare-feu d’applications web personnalisées, consultez [Règles personnalisées pour un pare-feu d’applications web](../ag/custom-waf-rules-overview.md).
- Pour plus d’informations sur le module Azure PowerShell, consultez [Documentation Azure PowerShell](/powershell/azure/).
