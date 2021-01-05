---
title: Créer un service de liaison privée dans Azure Private Link
description: Dans ce guide de démarrage rapide, vous utilisez un modèle Azure Resource Manager (modèle ARM) pour créer un service de liaison privée.
services: private-link
author: mblanco77
ms.service: private-link
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 05/29/2020
ms.author: allensu
ms.openlocfilehash: dce80d70af7cd711cf852a60b98ad65b6d21117f
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88705246"
---
# <a name="quickstart-create-a-private-link-service-by-using-an-arm-template"></a>Démarrage rapide : Créer un service de liaison privée à l’aide d’un modèle ARM

Dans ce guide de démarrage rapide, vous utilisez un modèle Azure Resource Manager (modèle ARM) pour créer un service de liaison privée.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Vous pouvez également suivre ce guide de démarrage rapide en utilisant le [Portail Azure](create-private-link-service-portal.md), [Azure PowerShell](create-private-link-service-powershell.md) ou l’interface [Azure CLI](create-private-link-service-cli.md).

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-privatelink-service%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Vous devez avoir un compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Vérifier le modèle

Ce modèle crée un service de liaison privée.

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-privatelink-service/).

:::code language="json" source="~/quickstart-templates/101-privatelink-service/azuredeploy.json":::

Plusieurs ressources Azure sont définies dans le modèle :

- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks) : Il y a un réseau virtuel pour chaque machine virtuelle.
- [**Microsoft.Network/loadBalancers**](/azure/templates/microsoft.network/loadBalancers) : l’équilibreur de charge qui expose les machines virtuelles hébergeant le service.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces) : il y a deux interfaces réseau, une pour chaque machine virtuelle.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines) : il y a deux machines virtuelles, une qui héberge le service et une autre qui teste la connexion au point de terminaison privé.
- [**Microsoft.Compute/virtualMachines/extensions**](/azure/templates/Microsoft.Compute/virtualMachines/extensions) : l’extension qui installe un serveur web.
- [**Microsoft.Network/privateLinkServices**](/azure/templates/microsoft.network/privateLinkServices) : le service de liaison privée pour exposer le service.
- [**Microsoft.Network/publicIpAddresses**](/azure/templates/microsoft.network/publicIpAddresses) : il y a deux adresses IP publiques, une pour chaque machine virtuelle.
- [**Microsoft.Network/privateendpoints**](/azure/templates/microsoft.network/privateendpoints) : le point de terminaison privé pour accéder au service.

## <a name="deploy-the-template"></a>Déployer le modèle

Voici comment déployer le modèle ARM sur Azure :

1. Pour vous connecter à Azure et ouvrir le modèle, sélectionnez **Déployer sur Azure**. Le modèle crée une machine virtuelle, un équilibreur de charge standard, un service de liaison privée, un point de terminaison privé, un réseau et une machine virtuelle à valider.

   [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-privatelink-service%2Fazuredeploy.json)

2. Sélectionnez ou créez votre groupe de ressources.
3. Saisissez le nom d’utilisateur et le mot de passe administrateur de la machine virtuelle.
4. Lisez la déclaration des conditions générales. Si vous acceptez, sélectionnez **J’accepte les termes et conditions mentionnés ci-dessus** > **Acheter**.

## <a name="validate-the-deployment"></a>Valider le déploiement

> [!NOTE]
> Le modèle ARM génère un nom unique pour la ressource de la machine virtuelle myConsumerVm<b>{uniqueid}</b>. Remplacez la valeur générée pour **{uniqueid}** .

### <a name="connect-to-a-vm-from-the-internet"></a>Se connecter à une machine virtuelle à partir d’Internet

Connectez-vous à la machine virtuelle _myConsumerVm{uniqueid}_ via Internet comme suit :

1.  Dans la barre de recherche du portail, saisissez _myConsumerVm{uniqueid}_ .

2.  Sélectionnez **Connecter**. **Se connecter à une machine virtuelle** s’ouvre.

3.  Sélectionnez **Télécharger le fichier RDP**. Azure crée un fichier de protocole RDP (Remote Desktop Protocol) ( _.rdp_) et le télécharge sur votre ordinateur.

4.  Ouvrez le fichier .rdp téléchargé.

    a. Si vous y êtes invité, sélectionnez **Connexion**.

    b. Entrez le nom d’utilisateur et le mot de passe spécifiés quand vous avez créé la machine virtuelle.
    
    > [!NOTE]
    > Vous devrez peut-être sélectionner **Plus de choix** > **Utiliser un autre compte** pour spécifier les informations d’identification que vous avez entrées lors de la création de la machine virtuelle.

5.  Sélectionnez **OK**.

6.  Un avertissement de certificat peut s’afficher pendant le processus de connexion. Si vous recevez un avertissement de certificat, sélectionnez **Oui** ou **Continuer**.

7.  Quand le bureau de la machine virtuelle s’affiche, réduisez-le pour revenir à votre bureau local.

### <a name="access-the-http-service-privately-from-the-vm"></a>Accéder au service http en privé à partir de la machine virtuelle

Voici comment vous connecter au service http à partir de la machine virtuelle en utilisant le point de terminaison privé.

1.  Accédez au Bureau à distance de la machine virtuelle _myConsumerVm{uniqueid}_ .
2.  Ouvrez un navigateur et entrez l’adresse du point de terminaison privé : `http://10.0.0.5/`.
3.  La page IIS par défaut s’affiche.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec le service de liaison privée, supprimez le groupe de ressources. Cette action supprime le service de liaison privée et toutes les ressources qui y sont associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [Azure Private Link](private-link-overview.md).
