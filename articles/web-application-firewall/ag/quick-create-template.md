---
title: 'Démarrage rapide : Créer un pare-feu d’application web (WAF) v2 sur Application Gateway - Modèle Azure Resource Manager'
titleSuffix: Azure Application Gateway
description: Découvrez comment utiliser un modèle de démarrage rapide Azure Resource Manager (modèle ARM) pour créer un pare-feu d’applications web v2 sur Azure Application Gateway.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: quickstart
ms.date: 09/16/2020
ms.author: victorh
ms.custom: subject-armqs
ms.openlocfilehash: 4c5d5c4cafee9402e2afd7ddc047fd3b68e84540
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92091319"
---
# <a name="quickstart-create-an-azure-waf-v2-on-application-gateway-using-an-arm-template"></a>Démarrage rapide : Créer un pare-feu d’application web (WAF) v2 sur Application Gateway à l’aide d’un modèle Resource Manager

Dans ce guide de démarrage rapide, vous utilisez un modèle Azure Resource Manager (modèle ARM) pour créer un pare-feu d’applications web Azure v2 sur Application Gateway.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-wafv2%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Vérifier le modèle

Ce modèle crée un pare-feu d’applications web v2 simple sur Azure Application Gateway. Cela comprend l’adresse IP d’un front-end d’adresses IP publiques, des paramètres HTTP, une règle avec un écouteur de base sur le port 80 et un pool back-end. Une stratégie WAF avec une règle personnalisée est créée pour bloquer le trafic vers le pool back-end en fonction d’un type de correspondance d’adresse IP.

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/ag-docs-wafv2/).

:::code language="json" source="~/quickstart-templates/ag-docs-wafv2/azuredeploy.json":::

Plusieurs ressources Azure sont définies dans le modèle :

- [**Microsoft.Network/applicationgateways**](/azure/templates/microsoft.network/applicationgateways)
- [**Microsoft.Network/ApplicationGatewayWebApplicationFirewallPolicies**](/azure/templates/microsoft.network/ApplicationGatewayWebApplicationFirewallPolicies)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses) : une pour la passerelle d’application et deux pour les machines virtuelles.
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines) : deux machines virtuelles
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces) : deux pour les machines virtuelles
- [**Microsoft.Compute/virtualMachine/extensions**](/azure/templates/microsoft.compute/virtualmachines/extensions) : pour configurer IIS et les pages web

## <a name="deploy-the-template"></a>Déployer le modèle

Déployez le modèle ARM sur Azure :

1. Sélectionnez **Déployer sur Azure** pour vous connecter à Azure et ouvrir le modèle. Le modèle crée une passerelle d’application, l’infrastructure réseau et deux machines virtuelles dans le pool de back-ends exécutant IIS.

   [![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fag-docs-wafv2%2Fazuredeploy.json)

2. Sélectionnez ou créez votre groupe de ressources.
3. Sélectionnez **J’accepte les conditions générales mentionnées ci-dessus**, puis **Acheter**. Le déploiement peut prendre 10 minutes ou plus.

## <a name="validate-the-deployment"></a>Valider le déploiement

Même si IIS n’est pas obligatoire pour créer la passerelle d’application, il est installé sur les serveurs back-end pour vérifier si Azure a bien créé un pare-feu d’applications web (WAF) v2 sur la passerelle d’application.

Utilisez IIS pour tester la passerelle d’application :

1. Trouvez l’adresse IP publique de la passerelle d’application dans la page **Vue d’ensemble** correspondante. ![Enregistrez l’adresse IP publique de la passerelle d’application](../../application-gateway/media/application-gateway-create-gateway-portal/application-gateway-record-ag-address.png) Vous pouvez aussi sélectionner **Toutes les ressources**, entrer *myAGPublicIPAddress* dans la zone de recherche, puis sélectionner l’adresse IP publique dans les résultats de la recherche. Azure affiche l’adresse IP publique dans la page **Vue d’ensemble**.
2. Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur pour y accéder.
3. Vérifiez la réponse. Une réponse **403 Interdit** vérifie que le pare-feu d’applications web (WAF) a bien été créé et qu’il bloque les connexions au pool back-end.
4. Changez la règle personnalisée pour définir **Autoriser le trafic**.
  Exécutez le script Azure PowerShell suivant, en remplaçant le nom de votre groupe de ressources :
   ```azurepowershell
   $rg = "<your resource group name>"
   $AppGW = Get-AzApplicationGateway -Name myAppGateway -ResourceGroupName $rg
   $pol = Get-AzApplicationGatewayFirewallPolicy -Name WafPol01 -ResourceGroupName $rg
   $pol[0].customrules[0].action = "allow"
   $rule = $pol.CustomRules
   Set-AzApplicationGatewayFirewallPolicy -Name WafPol01 -ResourceGroupName $rg -CustomRule $rule
   $AppGW.FirewallPolicy = $pol
   Set-AzApplicationGateway -ApplicationGateway $AppGW
   ```

   Actualisez plusieurs fois votre navigateur. Vous devez alors voir les connexions à myVM1 et à myVM2.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec la passerelle d’application, supprimez le groupe de ressources. Cela supprime la passerelle d’application et toutes les ressources associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Créer une passerelle d’application avec un pare-feu d’applications web à l’aide du portail Azure](application-gateway-web-application-firewall-portal.md)