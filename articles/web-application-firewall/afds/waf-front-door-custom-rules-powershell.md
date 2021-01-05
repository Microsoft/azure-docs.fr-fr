---
title: Configurer des règles personnalisées WAF & ensemble de règles par défaut pour Azure Front Door
description: Découvrez comment configurer une stratégie WAF constituée de règles personnalisées et managées pour un point de terminaison Front Door existant.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 09/05/2019
ms.author: victorh
ms.openlocfilehash: 9a0e262db9f5c37189a589eefc451a88dd5ea8c6
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563407"
---
# <a name="configure-a-web-application-firewall-policy-using-azure-powershell"></a>Configurer une stratégie de pare-feu d’applications web à l’aide d’Azure PowerShell

Une stratégie de pare-feu d’applications web (WAF) définit les contrôles à effectuer à la réception d’une requête sur Front Door.
Cet article explique comment configurer une stratégie WAF qui est constituée de règles personnalisées et pour laquelle un ensemble de règles par défaut managées par Azure est activé.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Avant de configurer une stratégie de limitation du débit, configurez votre environnement PowerShell et créez un profil Front Door.

### <a name="set-up-your-powershell-environment"></a>Configurer votre environnement PowerShell

Azure PowerShell fournit un ensemble d’applets de commande qui utilisent le modèle [Azure Resource Manager](../../azure-resource-manager/management/overview.md) pour gérer vos ressources Azure. 

Vous pouvez installer [Azure PowerShell](/powershell/azure/) sur votre ordinateur local et l’utiliser sur n’importe quelle session PowerShell. Suivez les instructions données pour vous connecter avec vos informations d’identification Azure, puis installer le module Az PowerShell.

#### <a name="sign-in-to-azure"></a>Connexion à Azure

```
Connect-AzAccount

```
Avant d’installer le module Front Door, assurez-vous que vous avez installé la version actuelle de PowerShellGet. Exécutez la commande ci-dessous, puis rouvrez PowerShell.

```
Install-Module PowerShellGet -Force -AllowClobber
``` 

#### <a name="install-azfrontdoor-module"></a>Installer le module Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```
### <a name="create-a-front-door-profile"></a>Créer un profil Front Door

Créez un profil Front Door en suivant les instructions décrites dans [Démarrage rapide : Créer un profil Front Door](../../frontdoor/quickstart-create-front-door.md)

## <a name="custom-rule-based-on-http-parameters"></a>Règle personnalisée en fonction des paramètres HTTP

L’exemple suivant montre comment configurer une règle personnalisée avec deux conditions de correspondance en utilisant [New-AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject). Les requêtes proviennent d’un site spécifié, tel que défini par le point d’accès, et la chaîne de requête ne contient pas le terme « password » (« mot de passe »). 

```powershell-interactive
$referer = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestHeader -OperatorProperty Equal -Selector "Referer" -MatchValue "www.mytrustedsites.com/referpage.html"
$password = New-AzFrontDoorWafMatchConditionObject -MatchVariable QueryString -OperatorProperty Contains -MatchValue "password"
$AllowFromTrustedSites = New-AzFrontDoorWafCustomRuleObject -Name "AllowFromTrustedSites" -RuleType MatchRule -MatchCondition $referer,$password -Action Allow -Priority 1
```

## <a name="custom-rule-based-on-http-request-method"></a>Règle personnalisée en fonction de la méthode de requête HTTP

Créez une règle de blocage de la méthode « PUT » en utilisant [New-AzFrontDoorWafCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject) comme suit :

```powershell-interactive
$put = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestMethod -OperatorProperty Equal -MatchValue PUT
$BlockPUT = New-AzFrontDoorWafCustomRuleObject -Name "BlockPUT" -RuleType MatchRule -MatchCondition $put -Action Block -Priority 2
```

## <a name="create-a-custom-rule-based-on-size-constraint"></a>Créer une règle personnalisée en fonction de la contrainte de taille

L’exemple suivant utilise Azure PowerShell pour créer une règle qui bloque les requêtes dont l’URL comporte plus de 100 caractères :
```powershell-interactive
$url = New-AzFrontDoorWafMatchConditionObject -MatchVariable RequestUri -OperatorProperty GreaterThanOrEqual -MatchValue 100
$URLOver100 = New-AzFrontDoorWafCustomRuleObject -Name "URLOver100" -RuleType MatchRule -MatchCondition $url -Action Block -Priority 3
```
## <a name="add-managed-default-rule-set"></a>Ajouter un ensemble de règles managées par défaut

L’exemple suivant crée un ensemble de règles managées par défaut à l’aide d’Azure PowerShell :
```powershell-interactive
$managedRules =  New-AzFrontDoorWafManagedRuleObject -Type DefaultRuleSet -Version 1.0
```
## <a name="configure-a-security-policy"></a>Configurer une stratégie de sécurité

Recherchez le nom du groupe de ressources qui contient le profil Front Door à l’aide de `Get-AzResourceGroup`. Ensuite, configurez une stratégie de sécurité avec les règles créées aux étapes précédentes en utilisant [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) dans le groupe de ressources spécifié qui abrite le profil Front Door.

```powershell-interactive
$myWAFPolicy=New-AzFrontDoorWafPolicy -Name $policyName -ResourceGroupName $resourceGroupName -Customrule $AllowFromTrustedSites,$BlockPUT,$URLOver100 -ManagedRule $managedRules -EnabledState Enabled -Mode Prevention
```

## <a name="link-policy-to-a-front-door-front-end-host"></a>Lier une stratégie à un hôte front-end Front Door

Liez l’objet de stratégie de sécurité à l’hôte front-end Front Door existant et mettez à jour les propriétés Front Door. Commencez par récupérer l’objet Front Door avec [Get-AzFrontDoor](/powershell/module/Az.FrontDoor/Get-AzFrontDoor).
Définissez ensuite la propriété *WebApplicationFirewallPolicyLink* du front-end sur la valeur *resourceId* du "$myWAFPolicy$" créé à l’étape précédente, en utilisant [Set-AzFrontDoor](/powershell/module/Az.FrontDoor/Set-AzFrontDoor). 

L’exemple ci-dessous utilise le nom de groupe de ressources *myResourceGroupFD1* en partant du principe que vous avez créé le profil Front Door à l’aide des instructions fournies dans l’article [Démarrage rapide : Créer un profil Front Door](../../frontdoor/quickstart-create-front-door.md). De plus, dans l’exemple ci-dessous, remplacez $frontDoorName par le nom de votre profil Front Door. 

```powershell-interactive
   $FrontDoorObjectExample = Get-AzFrontDoor `
     -ResourceGroupName myResourceGroupFD1 `
     -Name $frontDoorName
   $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $myWAFPolicy.Id
   Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
 ```

> [!NOTE]
> Il vous suffit de définir la propriété *WebApplicationFirewallPolicyLink* une seule fois pour lier une stratégie de sécurité à un hôte front-end Front Door. Les mises à jour suivantes de la stratégie sont automatiquement appliquées au front-end.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Front Door](../../frontdoor/front-door-overview.md) 
- En savoir plus sur [WAF pour Front Door](afds-overview.md)