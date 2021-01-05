---
title: 'Démarrage rapide : Utiliser un modèle Resource Manager pour créer une machine virtuelle Ubuntu Linux'
description: Dans ce guide de démarrage rapide, apprenez à utiliser un modèle Resource Manager pour créer une machine virtuelle Linux
author: cynthn
ms.service: virtual-machines-linux
ms.topic: quickstart
ms.workload: infrastructure
ms.date: 06/04/2020
ms.author: cynthn
ms.custom: subject-armqs
ms.openlocfilehash: bd22d20703dc72e220f9b479b4a4005033f964ae
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88649769"
---
# <a name="quickstart-create-an-ubuntu-linux-virtual-machine-using-an-arm-template"></a>Démarrage rapide : Créer une machine virtuelle Ubuntu Linux à l’aide d’un modèle ARM

Ce guide de démarrage rapide montre comment utiliser un modèle Azure Resource Manager (modèle ARM) pour déployer une machine virtuelle Ubuntu Linux dans Azure.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-vm-simple-linux%2fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-vm-simple-linux/).

:::code language="json" source="~/quickstart-templates/101-vm-simple-linux/azuredeploy.json":::


Plusieurs ressources sont définies dans le modèle :

- [**Microsoft.Network/virtualNetworks/subnets**](/azure/templates/Microsoft.Network/virtualNetworks/subnets) : permet de créer un sous-réseau.
- [**Microsoft.Storage/storageAccounts**](/azure/templates/Microsoft.Storage/storageAccounts) : permet de créer un compte de stockage.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/Microsoft.Network/networkInterfaces) : permet de créer une carte réseau.
- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/Microsoft.Network/networkSecurityGroups) : permet de créer un groupe de sécurité réseau.
- [**Microsoft.Network/virtualNetworks**](/azure/templates/Microsoft.Network/virtualNetworks) : permet de créer un réseau virtuel.
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/Microsoft.Network/publicIPAddresses) : permet de créer une adresse IP publique.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/Microsoft.Compute/virtualMachines) : permet de créer une machine virtuelle.

## <a name="deploy-the-template"></a>Déployer le modèle

1. Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle. Le modèle crée un coffre de clés et un secret.

    [![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-vm-simple-linux%2fazuredeploy.json)

1. Sélectionnez ou entrez les valeurs suivantes. Utilisez les valeurs par défaut, le cas échéant.

    - **Abonnement** : sélectionnez un abonnement Azure.
    - **Groupe de ressources** : sélectionnez un groupe de ressources existant dans la liste déroulante ou sélectionnez **Créer**, entrez un nom unique pour le groupe de ressources, puis cliquez sur **OK**.
    - **Emplacement** : sélectionnez un emplacement.  Par exemple, **USA Centre**.
    - **Nom d’utilisateur de l’administrateur** : indiquez un nom d’utilisateur, tel que *azureuser*.
    - **Type d'authentification** : Vous pouvez choisir d’utiliser une clé SSH ou un mot de passe.
    - **Clé ou mot de passe d’administrateur** en fonction du type d’authentification choisi :
        - Si vous choisissez **password**, le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](faq.md#what-are-the-password-requirements-when-creating-a-vm).
        - Si vous choisissez **sshPublicKey**, collez le contenu de votre clé publique.
    - **Préfixe d’étiquette DNS** : entrez un identificateur unique à utiliser dans l’étiquette DNS.
    - **Version du système d’exploitation Ubuntu** : sélectionnez la version Ubuntu que vous souhaitez exécuter sur la machine virtuelle.
    - **Emplacement** : la valeur par défaut est le même emplacement que le groupe de ressources, s’il existe déjà.
    - **Taille de la machine virtuelle** : sélectionnez la [taille](../sizes.md) à utiliser pour la machine virtuelle.
    - **Nom du réseau virtuel** : nom à utiliser pour le réseau virtuel.
    - **Nom du sous-réseau** : nom du sous-réseau que la machine virtuelle doit utiliser.
    - **Nom du groupe de sécurité réseau** : nom du groupe de sécurité réseau.
1. Sélectionnez **Revoir + créer**. Une fois la validation terminée, sélectionnez **Créer** pour créer et déployer la machine virtuelle.


Le portail Azure est utilisé pour déployer le modèle. Outre le portail Azure, vous pouvez utiliser Azure CLI, Azure PowerShell et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../../azure-resource-manager/templates/deploy-cli.md).

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Vous pouvez utiliser le portail Azure pour vérifier la machine virtuelle et les autres ressources qui ont été créées. Une fois le déploiement terminé, sélectionnez **Accéder au groupe de ressources** pour afficher la machine virtuelle et d’autres ressources.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui supprime également la machine virtuelle et toutes les ressources du groupe de ressources. 

1. Sélectionnez le **groupe de ressources**.
1. Dans la page du groupe de ressources, sélectionnez **Supprimer**.
1. Lorsque vous y êtes invité, entrez le nom du groupe de ressources, puis sélectionnez **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé une machine virtuelle simple à l’aide d’un modèle ARM. Pour en savoir plus sur les machines virtuelles Azure, suivez le didacticiel pour les machines virtuelles Linux.


> [!div class="nextstepaction"]
> [Didacticiels sur les machines virtuelles Azure Linux](./tutorial-manage-vm.md)