---
title: 'Tutoriel : Configurer une stratégie WAF de géofiltrage - Azure Front Door'
description: Dans ce tutoriel, vous allez apprendre à créer une stratégie WAF de géofiltrage et à l’associer à votre hôte front-end Front Door existant.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/14/2020
ms.author: duau
ms.openlocfilehash: f9b5869f7dd472c50ffb2c1c0dce765200f53882
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91324023"
---
# <a name="tutorial-how-to-set-up-a-geo-filtering-waf-policy-for-your-front-door"></a>Tutoriel : Guide pratique pour configurer une stratégie WAF de filtrage géographique pour votre porte d’entrée
Dans ce didacticiel, vous allez apprendre à utiliser Azure PowerShell pour créer un exemple de stratégie de filtrage géographique et à l’associer à votre hôte frontend Front Door existant. Cet exemple de stratégie de filtrage géographique bloquera les demandes de tous les autres pays/régions, à l’exception des États-Unis.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Définir une condition de correspondance de géofiltrage.
> - Ajouter une condition de correspondance de géofiltrage à une règle.
> - Ajouter des règles à une stratégie.
> - Lier une stratégie WAF à un hôte front-end Front Door.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis
* Avant de définir une stratégie de filtrage géographique, configurez votre environnement PowerShell et créez un profil Front Door.
* Créez un profil Front Door en suivant les instructions décrites dans [Démarrage rapide : créer un profil Front Door](quickstart-create-front-door.md).

## <a name="define-geo-filtering-match-condition"></a>Définir la condition de correspondance du filtrage géographique

Créez un exemple de condition de correspondance qui sélectionne les demandes ne provenant pas de « US » en utilisant [New-AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject) sur les paramètres lors de la création d’une condition de correspondance. Les code de pays/région à deux lettres pour effectuer un mappage pays/région sont fournis [ici](front-door-geo-filtering.md).

```azurepowershell-interactive
$nonUSGeoMatchCondition = New-AzFrontDoorWafMatchConditionObject `
-MatchVariable RemoteAddr `
-OperatorProperty GeoMatch `
-NegateCondition $true `
-MatchValue "US"
```
## <a name="add-geo-filtering-match-condition-to-a-rule-with-action-and-priority"></a>Ajouter une condition de correspondance du filtrage géographique à une règle à l’aide d’une action et d’une priorité

Créez un objet CustomRule `nonUSBlockRule` basé sur la condition de correspondance, sur une action et sur une priorité à l’aide de [New-AzFrontDoorWafCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject).  Un CustomRule peut avoir plusieurs MatchCondition.  Dans cet exemple, une action est définie pour bloquer et une priorité est définie comme 1, il s’agit de la priorité la plus élevée.

```
$nonUSBlockRule = New-AzFrontDoorWafCustomRuleObject `
-Name "geoFilterRule" `
-RuleType MatchRule `
-MatchCondition $nonUSGeoMatchCondition `
-Action Block `
-Priority 1
```
## <a name="add-rules-to-a-policy"></a>Ajouter des règles à une stratégie
Recherchez le nom du groupe de ressources qui contient le profil Front Door à l’aide de `Get-AzResourceGroup`. Ensuite, créez un objet de stratégie `geoPolicy` contenant `nonUSBlockRule` à l’aide de [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) dans le groupe de ressources spécifié qui abrite le profil Front Door. Vous devez fournir un nom unique pour la stratégie de géofiltrage. 

L’exemple ci-dessous utilise le nom de groupe de ressources *FrontDoorQS_rg0* en partant du principe que vous avez créé le profil Front Door à l’aide des instructions fournies dans l’article [Démarrage rapide : Créer un profil Front Door](quickstart-create-front-door.md). Dans l’exemple ci-dessous, remplacez le nom de la stratégie *geoPolicyAllowUSOnly* par un nom de stratégie unique.

```
$geoPolicy = New-AzFrontDoorWafPolicy `
-Name "geoPolicyAllowUSOnly" `
-resourceGroupName FrontDoorQS_rg0 `
-Customrule $nonUSBlockRule  `
-Mode Prevention `
-EnabledState Enabled
```
## <a name="link-waf-policy-to-a-front-door-frontend-host"></a>Lier une stratégie WAF et un hôte front-end Front Door
Liez l’objet de stratégie WAF à l’hôte front-end Front Door existant et mettez à jour les propriétés de la porte d’entrée. 

Pour ce faire, commencez par récupérer votre objet Front Door à l’aide de [Get-AzFrontDoor](/powershell/module/az.frontdoor/get-azfrontdoor). 

```
$geoFrontDoorObjectExample = Get-AzFrontDoor -ResourceGroupName FrontDoorQS_rg0
$geoFrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $geoPolicy.Id
```
Ensuite, définissez la propriété WebApplicationFirewallPolicyLink front-end sur l’ID de ressource de `geoPolicy` à l’aide de [Set-AzFrontDoor](/powershell/module/az.frontdoor/set-azfrontdoor).

```
Set-AzFrontDoor -InputObject $geoFrontDoorObjectExample[0]
```

> [!NOTE] 
> Vous ne devrez définir la propriété WebApplicationFirewallPolicyLink qu’une seule fois pour lier une stratégie WAF à un hôte front-end Front Door. Les mises à jour de stratégie suivantes sont automatiquement appliquées à l’hôte front-end.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans les étapes précédentes, vous avez configuré une règle de géofiltrage qui est associée à une stratégie WAF. Vous avez ensuite lié la stratégie à un hôte front-end de votre profil Front Door. Si vous n’avez plus besoin de la règle de géofiltrage ni de la stratégie WAF, vous devez d’abord dissocier la stratégie de l’hôte front-end avant de pouvoir supprimer la stratégie WAF.

:::image type="content" source="media/front-door-geo-filtering/front-door-disassociate-policy.png" alt-text="Dissocier la stratégie WAF":::

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment configurer un pare-feu d’applications web pour votre profil Front Door, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Pare-feu d’applications web et Front Door](front-door-waf.md)
