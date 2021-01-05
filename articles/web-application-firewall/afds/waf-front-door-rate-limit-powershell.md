---
title: Configurer une règle de limitation du débit WAF pour Front Door – Azure PowerShell
description: Découvrez comment configurer une règle de limitation du débit pour un point de terminaison Front Door existant.
author: vhorne
ms.service: web-application-firewall
ms.topic: article
services: web-application-firewall
ms.date: 02/26/2020
ms.author: victorh
ms.openlocfilehash: 23b893bad591af5f1e923b68e8d30453f859792b
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563475"
---
# <a name="configure-a-web-application-firewall-rate-limit-rule-using-azure-powershell"></a>Configurer une règle de limitation du débit du pare-feu d’applications web à l’aide d’Azure PowerShell
La règle de limitation du débit du pare-feu d'applications web (WAF) Azure pour Azure Front Door contrôle le nombre de requêtes autorisées de la part des clients sur une durée d'une minute.
Cet article explique comment utiliser Azure PowerShell pour configurer une règle de limitation du débit du WAF afin de contrôler le nombre de requêtes que les clients sont autorisés à envoyer à une application web dont l'URL contient */promo*.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

> [!NOTE]
> Les limitations de débit sont appliquées pour chaque adresse IP de client. Si vous avez plusieurs clients qui accèdent à votre instance Front Door à partir d’adresses IP différentes, leurs propres limitations de débit sont appliquées.

## <a name="prerequisites"></a>Prérequis
Avant de configurer une stratégie de limitation du débit, configurez votre environnement PowerShell et créez un profil Front Door.
### <a name="set-up-your-powershell-environment"></a>Configurer votre environnement PowerShell
Azure PowerShell fournit un ensemble d’applets de commande qui utilisent le modèle [Azure Resource Manager](../../azure-resource-manager/management/overview.md) pour gérer vos ressources Azure. 

Vous pouvez installer [Azure PowerShell](/powershell/azure/) sur votre ordinateur local et l’utiliser sur n’importe quelle session PowerShell. Suivez les instructions données pour vous connecter avec vos informations d’identification Azure, puis installer le module Az PowerShell.

#### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Se connecter à Azure avec une boîte de dialogue interactive
```
Connect-AzAccount

```
Avant d'installer le module Front Door, assurez-vous que la version actuelle de PowerShellGet est installée. Exécutez la commande suivante et rouvrez PowerShell.

```
Install-Module PowerShellGet -Force -AllowClobber
``` 

#### <a name="install-azfrontdoor-module"></a>Installer le module Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```
### <a name="create-a-front-door-profile"></a>Créer un profil Front Door
Créez un profil Front Door en suivant les instructions décrites dans [Démarrage rapide : Créer un profil Front Door](../../frontdoor/quickstart-create-front-door.md)

## <a name="define-url-match-conditions"></a>Définir les conditions de correspondance d’URL
Définissez une condition de correspondance d’URL (URL contenant /promo) avec [New-AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject).
L’exemple suivant définit */promo* comme valeur de correspondance de la variable *RequestUri* :

```powershell-interactive
   $promoMatchCondition = New-AzFrontDoorWafMatchConditionObject `
     -MatchVariable RequestUri `
     -OperatorProperty Contains `
     -MatchValue "/promo"
```
## <a name="create-a-custom-rate-limit-rule"></a>Créer une règle personnalisée de limitation du débit
Définissez une limitation du débit avec [New-AzFrontDoorWafCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject). Dans l’exemple suivant, la limite est définie à 1000. Quand un client effectue plus de mille requêtes vers la page promo dans un intervalle d’une minute, les requêtes en plus sont bloquées jusqu’au prochain intervalle d’une minute.

```powershell-interactive
   $promoRateLimitRule = New-AzFrontDoorWafCustomRuleObject `
     -Name "rateLimitRule" `
     -RuleType RateLimitRule `
     -MatchCondition $promoMatchCondition `
     -RateLimitThreshold 1000 `
     -Action Block -Priority 1
```


## <a name="configure-a-security-policy"></a>Configurer une stratégie de sécurité

Recherchez le nom du groupe de ressources qui contient le profil Front Door à l’aide de `Get-AzureRmResourceGroup`. Ensuite, configurez une stratégie de sécurité avec une règle personnalisée de limitation du débit en utilisant [ New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) dans le groupe de ressources spécifié qui abrite le profil Front Door.

L’exemple ci-dessous utilise le nom de groupe de ressources *myResourceGroupFD1* en partant du principe que vous avez créé le profil Front Door à l’aide des instructions fournies dans l’article [Démarrage rapide : Créer une instance Front Door](../../frontdoor/quickstart-create-front-door.md) avec [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy).

```powershell-interactive
   $ratePolicy = New-AzFrontDoorWafPolicy `
     -Name "RateLimitPolicyExamplePS" `
     -resourceGroupName myResourceGroupFD1 `
     -Customrule $promoRateLimitRule `
     -Mode Prevention `
     -EnabledState Enabled
```
## <a name="link-policy-to-a-front-door-front-end-host"></a>Lier une stratégie à un hôte front-end Front Door
Liez l’objet de stratégie de sécurité à l’hôte front-end Front Door existant et mettez à jour les propriétés Front Door. Commencez par récupérer l’objet Front Door à l’aide de la commande [Get-AzFrontDoor](/powershell/module/Az.FrontDoor/Get-AzFrontDoor).
Définissez ensuite la propriété *WebApplicationFirewallPolicyLink* du front-end sur la valeur *resourceId* du "$ratePolicy" créé à l’étape précédente, en utilisant la commande [Set-AzFrontDoor](/powershell/module/Az.FrontDoor/Set-AzFrontDoor). 

L’exemple ci-dessous utilise le nom de groupe de ressources *myResourceGroupFD1* en partant du principe que vous avez créé le profil Front Door à l’aide des instructions fournies dans l’article [Démarrage rapide : Créer un profil Front Door](../../frontdoor/quickstart-create-front-door.md). De plus, dans l’exemple ci-dessous, remplacez $frontDoorName par le nom de votre profil Front Door. 

```powershell-interactive
   $FrontDoorObjectExample = Get-AzFrontDoor `
     -ResourceGroupName myResourceGroupFD1 `
     -Name $frontDoorName
   $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $ratePolicy.Id
   Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
 ```

> [!NOTE]
> Il vous suffit de définir la propriété *WebApplicationFirewallPolicyLink* une seule fois pour lier une stratégie de sécurité à un hôte front-end Front Door. Les mises à jour suivantes de la stratégie sont automatiquement appliquées au front-end.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Front Door](../../frontdoor/front-door-overview.md).