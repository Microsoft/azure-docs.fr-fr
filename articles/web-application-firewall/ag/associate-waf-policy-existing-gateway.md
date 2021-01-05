---
title: Associer une stratégie de pare-feu d’applications web à une Application Gateway Azure existante
description: Découvrez comment associer une stratégie de pare-feu d’applications web à une Application Gateway Azure existante.
services: web-application-firewall
ms.topic: article
author: vhorne
ms.service: web-application-firewall
ms.date: 10/25/2019
ms.author: victorh
ms.openlocfilehash: 1ed2e0cf8cc8cd841d8779462d62ba4852774a3a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74083902"
---
# <a name="associate-a-waf-policy-with-an-existing-application-gateway"></a>Associer une stratégie WAF à une Application Gateway existante

Vous pouvez utiliser Azure PowerShell pour [créer une stratégie WAF](create-waf-policy-ag.md), mais vous disposez peut-être déjà d’une Application Gateway et souhaitez simplement lui associer une stratégie WAF. Dans cet article, vous n’avez rien à faire. vous créez une stratégie WAF et l’associez à une Application Gateway déjà existante. 

1. Obtenez votre Application Gateway et Stratégie de pare-feu. Si vous n’avez pas de stratégie de pare-feu existante, consultez l’étape 2. 

   ```azurepowershell-interactive
      Connect-AzAccount
      Select-AzSubscription -Subscription "<sub id>"`

      #Get Application Gateway and existing policy object or create new`
      $gw = Get-AzApplicationGateway -Name <Waf v2> -ResourceGroupName <RG name>`
      $policy = Get-AzApplicationGatewayFirewallPolicy -Name <policy name> -ResourceGroupName <RG name>`
   ```

2. (Facultatif) créer une stratégie de pare-feu.

   ```azurepowershell-interactive
      New-AzApplicationGatewayFirewallPolicy -Name <policy name> -ResourceGroupName <RG name>'
      $policy = Get-AzApplicationGatewayFirewallPolicy -Name <policy name> -ResourceGroupName <RG name>`
   ```
   > [!NOTE]
   > Si vous créez cette stratégie WAF pour passer d’une configuration WAF à une stratégie WAF, la stratégie doit être une copie exacte de votre ancienne configuration. Cela signifie que toutes les exclusions, règles personnalisées, groupes de règles désactivés, etc. doivent être exactement identiques à celles de la configuration WAF.
3. (Facultatif) Vous pouvez configurer la stratégie WAF en fonction de vos besoins. Cela comprend les règles personnalisées, la désactivation des règles/groupes de règles, les exclusions, la définition des limites de chargement de fichier, etc. Si vous ignorez cette étape, toutes les valeurs par défaut sont sélectionnées. 
   
4. Enregistrez la stratégie et joignez-la à votre Application Gateway. 
   
   ```azurepowershell-interactive
      #Save the policy itself
      Set-AzApplicationGatewayFirewallPolicy -InputObject $policy`
   
      #Attach the policy to an Application Gateway
      $gw.FirewallPolicy = $policy`
   
      #Save the Application Gateway
      Set-AzApplicationGateway -ApplicationGateway $gw`
   ```

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur les Règles personnalisées.](configure-waf-custom-rules.md)
