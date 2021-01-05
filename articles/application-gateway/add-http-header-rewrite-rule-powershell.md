---
title: Réécrire les en-têtes HTTP dans une passerelle d’application Azure Application Gateway
description: Cet article fournit des informations sur la façon de réécrire les en-têtes HTTP dans Azure Application Gateway à l’aide d’Azure PowerShell.
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: how-to
ms.date: 04/12/2019
ms.author: absha
ms.openlocfilehash: 6938ad55915286af397fee6d72a333e3bb39a1e6
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397913"
---
# <a name="rewrite-http-request-and-response-headers-with-azure-application-gateway---azure-powershell"></a>Réécrire les en-têtes de requête et de réponse HTTP avec Azure Application Gateway - Azure PowerShell

Cet article explique comment utiliser Azure PowerShell pour configurer une instance de la [référence SKU Application Gateway v2](./application-gateway-autoscaling-zone-redundant.md) afin de réécrire les en-têtes HTTP des requêtes et des réponses.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="before-you-begin"></a>Avant de commencer

- Pour suivre la procédure décrite dans cet article, vous devez exécuter Azure PowerShell localement. Vous devez également avoir installé la version 1.0.0 du module Az ou une version ultérieure. Pour déterminer la version que vous avez installée, exécutez `Import-Module Az`, puis `Get-Module Az`. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Après avoir vérifié la version PowerShell, exécutez `Login-AzAccount` pour créer une connexion avec Azure.
- Vous devez disposer d’une instance de la référence SKU Application Gateway v2. La réécriture des en-têtes n’est pas prise en charge dans la référence SKU v1. Si vous ne disposez pas de la référence SKU v2, créez une instance de [référence SKU Application Gateway v2](./tutorial-autoscale-ps.md) avant de commencer.

## <a name="create-required-objects"></a>Créer les objets requis

Pour configurer la réécriture d’en-tête HTTP, vous devez effectuer ces étapes.

1. Créez les objets nécessaires à la réécriture d’en-tête HTTP :

   - **RequestHeaderConfiguration** : permet de spécifier les champs d’en-tête de requête que vous souhaitez réécrire, ainsi que la nouvelle valeur des en-têtes.

   - **ResponseHeaderConfiguration** : permet de spécifier les champs d’en-tête de réponse que vous souhaitez réécrire, ainsi que la nouvelle valeur des en-têtes.

   - **ActionSet** : contient les configurations des en-têtes de requête et de réponse spécifiés ci-dessus.

   - **Condition**  : configuration facultative. Les conditions de réécriture évaluent le contenu des requêtes et réponses HTTP(S). L’action de réécriture se produit si la requête ou la réponse HTTP(S) correspondent à la condition de réécriture.

     Si vous associez plusieurs conditions à une action, cette dernière ne se produit que lorsque toutes les conditions sont remplies. En d’autres termes, il s’agit d’une opération ET logique.

   - **RewriteRule** : contient plusieurs combinaisons d’actions/conditions de réécriture.

   - **RuleSequence** : configuration facultative qui permet de déterminer l’ordre dans lequel s’exécutent les règles de réécriture. Cette configuration est utile quand vous disposez de plusieurs règles de réécriture dans un jeu de réécritures. Une règle de réécriture qui présente une valeur de séquence de règle inférieure s’exécute en premier. Si vous attribuez la même valeur de séquence de règle à deux règles de réécriture, l’ordre d’exécution n’est pas déterministe.

     Si vous ne configurez pas explicitement l’objet RuleSequence, une valeur par défaut de 100 est définie.

   - **RewriteRuleSet** : contient plusieurs règles de réécriture qui seront associées à une règle d’acheminement de requête.

2. Attachez l’objet RewriteRuleSet à une règle d’acheminement. La configuration de réécriture est attachée à l’écouteur source via la règle de routage. Quand vous utilisez une règle de routage de base, la configuration de réécriture d’en-tête est associée à un écouteur source et correspond à une réécriture d’en-tête globale. Quand vous utilisez une règle de routage basée sur le chemin, la configuration de réécriture d’en-tête est définie sur le mappage du chemin d’URL. Dans ce cas, elle s’applique uniquement à la zone de chemin spécifique d’un site.

Vous pouvez créer plusieurs jeux de réécritures d’en-tête HTTP et appliquer chacun d’eux à différents écouteurs. En revanche, vous ne pouvez appliquer qu’un seul jeu de réécritures à un écouteur spécifique.

## <a name="sign-in-to-azure"></a>Connexion à Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="specify-the-http-header-rewrite-rule-configuration"></a>Spécifier la configuration de règle de réécriture des en-têtes HTTP

Dans cet exemple, nous allons modifier une URL de redirection en réécrivant l’en-tête d’emplacement dans la réponse HTTP chaque fois que cet en-tête contient une référence à azurewebsites.net. Pour ce faire, nous allons ajouter une condition destinée à évaluer si l’en-tête d’emplacement de la réponse contient ou non azurewebsites.net. Nous allons utiliser le modèle `(https?):\/\/.*azurewebsites\.net(.*)$`. Et nous utiliserons `{http_resp_Location_1}://contoso.com{http_resp_Location_2}` comme valeur d’en-tête. Cette valeur remplacera *azurewebsites.net* par *contoso.com* dans l’en-tête d’emplacement.

```azurepowershell
$responseHeaderConfiguration = New-AzApplicationGatewayRewriteRuleHeaderConfiguration -HeaderName "Location" -HeaderValue "{http_resp_Location_1}://contoso.com{http_resp_Location_2}"
$actionSet = New-AzApplicationGatewayRewriteRuleActionSet -RequestHeaderConfiguration $requestHeaderConfiguration -ResponseHeaderConfiguration $responseHeaderConfiguration
$condition = New-AzApplicationGatewayRewriteRuleCondition -Variable "http_resp_Location" -Pattern "(https?):\/\/.*azurewebsites\.net(.*)$" -IgnoreCase
$rewriteRule = New-AzApplicationGatewayRewriteRule -Name LocationHeader -ActionSet $actionSet
$rewriteRuleSet = New-AzApplicationGatewayRewriteRuleSet -Name LocationHeaderRewrite -RewriteRule $rewriteRule
```

## <a name="retrieve-the-configuration-of-your-application-gateway"></a>Récupérer la configuration de votre passerelle d’application

```azurepowershell
$appgw = Get-AzApplicationGateway -Name "AutoscalingAppGw" -ResourceGroupName "<rg name>"
```

## <a name="retrieve-the-configuration-of-your-request-routing-rule"></a>Récupérer la configuration de votre règle d’acheminement des requêtes

```azurepowershell
$reqRoutingRule = Get-AzApplicationGatewayRequestRoutingRule -Name rule1 -ApplicationGateway $appgw
```

## <a name="update-the-application-gateway-with-the-configuration-for-rewriting-http-headers"></a>Mettre à jour la passerelle d’application avec la configuration de réécriture des en-têtes HTTP

```azurepowershell
Add-AzApplicationGatewayRewriteRuleSet -ApplicationGateway $appgw -Name LocationHeaderRewrite -RewriteRule $rewriteRuleSet.RewriteRules
Set-AzApplicationGatewayRequestRoutingRule -ApplicationGateway $appgw -Name rule1 -RuleType $reqRoutingRule.RuleType -BackendHttpSettingsId $reqRoutingRule.BackendHttpSettings.Id -HttpListenerId $reqRoutingRule.HttpListener.Id -BackendAddressPoolId $reqRoutingRule.BackendAddressPool.Id -RewriteRuleSetId $rewriteRuleSet.Id
Set-AzApplicationGateway -ApplicationGateway $appgw
```

## <a name="delete-a-rewrite-rule"></a>Supprimer une règle de réécriture

```azurepowershell
$appgw = Get-AzApplicationGateway -Name "AutoscalingAppGw" -ResourceGroupName "<rg name>"
Remove-AzApplicationGatewayRewriteRuleSet -Name "LocationHeaderRewrite" -ApplicationGateway $appgw
$requestroutingrule= Get-AzApplicationGatewayRequestRoutingRule -Name "rule1" -ApplicationGateway $appgw
$requestroutingrule.RewriteRuleSet= $null
set-AzApplicationGateway -ApplicationGateway $appgw
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la configuration de certains cas d’usage courants, consultez l’article présentant des [scénarios de réécriture d’en-têtes courants](./rewrite-http-headers.md).