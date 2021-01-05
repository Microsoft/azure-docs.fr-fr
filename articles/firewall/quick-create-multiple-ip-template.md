---
title: 'Démarrage rapide : Créer un pare-feu Azure avec plusieurs adresses IP publiques - modèle Resource Manager'
description: Dans ce guide de démarrage rapide, découvrez comment utiliser un modèle Azure Resource Manager (modèle ARM) pour créer un pare-feu Azure avec plusieurs adresses IP publiques.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 08/28/2020
ms.author: victorh
ms.openlocfilehash: 33d110f327a381241fd9dbf55996a6e3f9fd3a43
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057975"
---
# <a name="quickstart-create-an-azure-firewall-with-multiple-public-ip-addresses---arm-template"></a>Démarrage rapide : Créer un pare-feu Azure avec plusieurs adresses IP publiques - Modèle ARM

Dans ce guide de démarrage rapide, vous utilisez un modèle Azure Resource Manager (modèle ARM) pour déployer un pare-feu Azure avec plusieurs adresses IP publiques à partir d’un préfixe d’adresses IP publique. Le pare-feu déployé présente des règles de collection de règles NAT qui autorisent les connexions RDP à deux machines virtuelles Windows Server 2019.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Pour plus d’informations sur le pare-feu Azure avec plusieurs adresses IP publiques, consultez [Déployer un pare-feu Azure avec plusieurs adresses IP publiques à l’aide d’Azure PowerShell](deploy-multi-public-ip-powershell.md).

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffw-docs-qs%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Vérifier le modèle

Ce modèle crée un pare-feu Azure avec deux adresses IP publiques ainsi que les ressources nécessaires à la prise en charge du pare-feu Azure.

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/fw-docs-qs).

:::code language="json" source="~/quickstart-templates/fw-docs-qs/azuredeploy.json":::

Plusieurs ressources Azure sont définies dans le modèle :

- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/publicIPPrefix**](/azure/templates/microsoft.network/publicipprefixes)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines)
- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageAccounts)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.network/routeTables)

## <a name="deploy-the-template"></a>Déployer le modèle

Déployez le modèle ARM sur Azure :

1. Sélectionnez **Déployer sur Azure** pour vous connecter à Azure et ouvrir le modèle. Le modèle crée un pare-feu Azure, l’infrastructure réseau et deux machines virtuelles.

   [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffw-docs-qs%2Fazuredeploy.json)

2. Dans le portail, affichez la page **Créer un pare-feu Azure avec plusieurs adresses IP publiques**, puis tapez ou sélectionnez les valeurs suivantes :
   - Abonnement : sélectionnez à partir d’abonnements existants. 
   - Groupe de ressources :  sélectionnez à partir de groupe de ressources existants, ou sélectionnez **Créer**, puis **OK**.
   - Localisation : Sélectionner un emplacement
   - Nom de l’utilisateur administrateur : tapez le nom d’utilisateur du compte de l’utilisateur administrateur. 
   - Mot de passe administrateur : tapez une clé ou un mot de passe administrateur.

3. Sélectionnez **J’accepte les conditions générales mentionnées ci-dessus**, puis **Acheter**. Le déploiement peut prendre 10 minutes ou plus.

## <a name="validate-the-deployment"></a>Valider le déploiement

Dans le portail Azure, passez en revue les ressources déployées. Notez les adresses IP publiques du pare-feu.  

Utilisez une connexion Bureau à distance pour vous connecter aux adresses IP publiques du pare-feu. Pour que les connexions réussissent, les règles NAT de pare-feu doivent autoriser la connexion aux serveurs back-end.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec le pare-feu, supprimez le groupe de ressources. Vous supprimez ainsi le pare-feu et toutes les ressources associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Déployer et configurer un pare-feu Azure dans un réseau hybride à l’aide du portail Azure](tutorial-hybrid-portal.md)
