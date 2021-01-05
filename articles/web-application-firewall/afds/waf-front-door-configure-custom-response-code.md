---
title: Configurer une réponse personnalisée pour le pare-feu d’applications web (WAF) avec Azure Front Door
description: Découvrez comment configurer un code et un message de réponse personnalisée quand le pare-feu d’applications web bloque une requête.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 06/10/2020
ms.author: victorh
ms.reviewer: tyao
ms.openlocfilehash: 8fc6e71494df36cd6f823661b18e4a3d8ce2938c
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563679"
---
# <a name="configure-a-custom-response-for-azure-web-application-firewall-waf"></a>Configurer une réponse personnalisée pour le pare-feu d’applications web (WAF) Azure

Par défaut, quand WAF bloque une requête en raison d’une mise en correspondance de règle, il retourne un code d’état 403 avec le message **La requête est bloquée**. Le message par défaut comprend également la chaîne de référence de suivi qui peut être utilisée pour établir un lien vers des [entrées de journal](./waf-front-door-monitor.md) pour la requête.  Vous pouvez configurer un code d’état de réponse personnalisé et un message personnalisé avec une chaîne de référence pour votre cas d’usage. Cet article décrit comment configurer une page de réponse personnalisée quand une requête est bloquée par le pare-feu d’applications web (WAF).

## <a name="configure-custom-response-status-code-and-message-use-portal"></a>Configurer un code d’état et un message de réponse personnalisés à l’aide du portail

Vous pouvez configurer un corps et un code d’état de réponse personnalisé sous « Paramètres de stratégie » à partir du portail WAF.

:::image type="content" source="../media/waf-front-door-configure-custom-response-code/custom-response-settings.png" alt-text="Paramètres de stratégie WAF":::

Dans l’exemple ci-dessus, nous avons gardé le code de réponse 403 et configuré un message bref « Please contact us », comme illustré dans l’image ci-dessous :

:::image type="content" source="../media/waf-front-door-configure-custom-response-code/custom-response.png" alt-text="Exemple de réponse personnalisée":::

« {{Azure-Ref}} » insère la chaîne de référence unique dans le corps de la réponse. La valeur correspond au champ TrackingReference dans les journaux `FrontdoorAccessLog` et `FrontdoorWebApplicationFirewallLog`.

## <a name="configure-custom-response-status-code-and-message-use-powershell"></a>Configurer un code d’état et un message de réponse personnalisés à l’aide de PowerShell

### <a name="set-up-your-powershell-environment"></a>Configurer votre environnement PowerShell

Azure PowerShell fournit un ensemble d’applets de commande qui utilisent le modèle [Azure Resource Manager](../../azure-resource-manager/management/overview.md) pour gérer vos ressources Azure. 

Vous pouvez installer [Azure PowerShell](/powershell/azure/) sur votre ordinateur local et l’utiliser sur n’importe quelle session PowerShell. Suivez les instructions sur la page pour vous connecter avec vos informations d’identification Azure, puis installez le module Az PowerShell.

### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Se connecter à Azure avec une boîte de dialogue interactive

```
Connect-AzAccount
Install-Module -Name Az

```
Vérifiez que la version actuelle de PowerShellGet est installée. Exécutez la commande ci-dessous, puis rouvrez PowerShell.
```
Install-Module PowerShellGet -Force -AllowClobber
``` 
### <a name="install-azfrontdoor-module"></a>Installer le module Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Dans Azure, vous allouez les ressources associées à un groupe de ressources. Ici nous créons un groupe de ressources à l’aide de [New-AzResourceGroup](/powershell/module/Az.resources/new-Azresourcegroup).

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupWAF
```

### <a name="create-a-new-waf-policy-with-custom-response"></a>Créer une nouvelle stratégie de pare-feu d’applications web avec une réponse personnalisée 

Voici un exemple de création de stratégie de pare-feu d’applications web avec le code d’état de réponse personnalisé 405 et le message **You are blocked.** , à l’aide de [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy).

```azurepowershell
# WAF policy setting
New-AzFrontDoorWafPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-EnabledState enabled `
-Mode Detection `
-CustomBlockResponseStatusCode 405 `
-CustomBlockResponseBody "<html><head><title>You are blocked.</title></head><body></body></html>"
```

Modifiez le code de réponse personnalisé ou des paramètres du corps de réponse d’une stratégie de pare-feu d’applications web existante à l’aide de [Update-AzFrontDoorFireWallPolicy](/powershell/module/az.frontdoor/Update-AzFrontDoorWafPolicy).

```azurepowershell
# modify WAF response code
Update-AzFrontDoorFireWallPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-EnabledState enabled `
-Mode Detection `
-CustomBlockResponseStatusCode 403
```

```azurepowershell
# modify WAF response body
Update-AzFrontDoorFireWallPolicy `
-Name myWAFPolicy `
-ResourceGroupName myResourceGroupWAF `
-CustomBlockResponseBody "<html><head><title>Forbidden</title></head><body>{{azure-ref}}</body></html>"
```

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur le [pare-feu d’applications web avec Azure Front Door](../afds/afds-overview.md)