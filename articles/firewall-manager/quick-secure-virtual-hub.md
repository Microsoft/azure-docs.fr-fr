---
title: 'Démarrage rapide : Sécuriser un hub virtuel à l’aide d’Azure Firewall Manager – Modèle Resource Manager'
description: Dans ce guide de démarrage rapide, découvrez comment sécuriser votre hub virtuel à l’aide d’Azure Firewall Manager.
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 08/28/2020
ms.author: victorh
ms.openlocfilehash: 050dfa737ad04127f8c72158fbecb2e8e70e05c1
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89075339"
---
# <a name="quickstart-secure-your-virtual-hub-using-azure-firewall-manager---arm-template"></a>Démarrage rapide : Sécuriser votre hub virtuel avec Azure Firewall Manager - Modèle ARM

Dans ce guide de démarrage rapide, vous allez utiliser un modèle Azure Resource Manager (modèle ARM) pour sécuriser un hub virtuel à l’aide d’Azure Firewall Manager. Le pare-feu déployé comprend une règle d’application qui autorise les connexions à `www.microsoft.com`. Deux machines virtuelles Windows Server 2019 sont déployées pour tester le pare-feu. Un serveur de rebond est utilisé pour se connecter au serveur de charge de travail. À partir du serveur de charge de travail, il n’est possible de se connecter qu’à `www.microsoft.com`.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Pour plus d’informations sur Azure Firewall Manager, consultez [Qu’est-ce qu’Azure Firewall Manager ?](overview.md)

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffwm-docs-qs%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Vérifier le modèle

Ce modèle crée un hub virtuel sécurisé à l’aide d’Azure Firewall Manager, ainsi que les ressources nécessaires au scénario.

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/fwm-docs-qs/).

:::code language="json" source="~/quickstart-templates/fwm-docs-qs/azuredeploy.json":::

Plusieurs ressources Azure sont définies dans le modèle :

- [**Microsoft.Network/virtualWans**](/azure/templates/microsoft.network/virtualWans)
- [**Microsoft.Network/virtualHubs**](/azure/templates/microsoft.network/virtualHubs)
- [**Microsoft.Network/firewallPolicies**](/azure/templates/microsoft.network/firewallPolicies)
- [**Microsoft.Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines)
- [**Microsoft.Storage/storageAccounts**](/azure/templates/microsoft.storage/storageAccounts)
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.network/routeTables)

## <a name="deploy-the-template"></a>Déployer le modèle

Déployez le modèle ARM sur Azure :

1. Sélectionnez **Déployer sur Azure** pour vous connecter à Azure et ouvrir le modèle. Le modèle crée un Pare-feu Azure, un réseau WAN virtuel et un hub virtuel, l’infrastructure réseau et deux machines virtuelles.

   [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffwm-docs-qs%2Fazuredeploy.json)

2. Sur la page **Hubs virtuels sécurisés** du portail, tapez ou sélectionnez les valeurs suivantes :
   - Abonnement : sélectionnez à partir d’abonnements existants. 
   - Groupe de ressources :  sélectionnez à partir de groupe de ressources existants, ou sélectionnez **Créer**, puis **OK**.
   - Localisation : Sélectionner un emplacement
   - Nom de l’utilisateur administrateur : tapez le nom d’utilisateur du compte de l’utilisateur administrateur. 
   - Mot de passe administrateur : tapez une clé ou un mot de passe administrateur.

3. Sélectionnez **Vérifier + créer**, puis **Créer**. Le déploiement peut prendre 10 minutes ou plus.

## <a name="validate-the-deployment"></a>Valider le déploiement

À présent, testez les règles de pare-feu pour vérifier qu’elles fonctionnent comme prévu.

1. Dans le portail Azure, passez en revue les paramètres réseau pour la machine virtuelle **Workload-Srv** et notez l’adresse IP privée.
2. Connectez un bureau à distance à la machine virtuelle **Jump-Srv**, puis connectez-vous. À partir de là, ouvrez une connexion de bureau distant à l’adresse IP privée **Workload-Srv**.

3. Ouvrez Internet Explorer et accédez à `www.microsoft.com`.
4. Sélectionnez **OK** > **Fermer** sur les alertes de sécurité d’Internet Explorer.

   La page d’accueil de Microsoft doit s’afficher.

5. Accédez à `www.google.com`.

   Vous devriez être bloqué par le pare-feu.

Maintenant que vous avez vérifié que les règles de pare-feu fonctionnent :

- Vous pouvez accéder au nom de domaine complet autorisé, mais pas à d’autres.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec le pare-feu, supprimez le groupe de ressources. Vous supprimez ainsi le pare-feu et toutes les ressources associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les fournisseurs de partenaire de sécurité](trusted-security-partners.md)
