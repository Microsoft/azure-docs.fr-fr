---
title: 'Démarrage rapide : Créer une machine virtuelle DSVM - Modèle Resource Manager'
titleSuffix: Azure Data Science Virtual Machine
description: Dans ce guide de démarrage rapide, vous utilisez un modèle Azure Resource Manager pour déployer rapidement une machine virtuelle DSVM (Data Science Virtual Machine)
services: machine-learning
author: lobrien
ms.author: laobri
ms.custom: subject-armqs
ms.date: 06/10/2020
ms.service: machine-learning
ms.subservice: data-science-vm
ms.topic: quickstart
ms.openlocfilehash: 8f83cdb09b94fb7eda1f8f700390be8b2c2147e0
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90883156"
---
# <a name="quickstart-create-an-ubuntu-data-science-virtual-machine-using-an-arm-template"></a>Démarrage rapide : Créer une machine virtuelle Data Science Virtual Machine Ubuntu à l’aide d’un modèle Resource Manager

Ce guide de démarrage rapide vous montre comment créer une machine virtuelle Data Science Virtual Machine Ubuntu 18.04 à l’aide d’un modèle Resource Manager (Azure Resource Manager). Les machines virtuelles Data Science Virtual Machine sont des machines virtuelles basées sur le cloud et préchargées avec une suite de frameworks et d’outils de science des données et de machine learning. En cas de déploiement sur des ressources de calcul alimentées par GPU, l’ensemble des outils et bibliothèques sont configurés pour utiliser le GPU.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-ubuntu-DSVM-GPU-or-CPU%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/services/machine-learning/) avant de commencer.

* Pour utiliser les commandes CLI dans ce document à partir de votre **environnement local**, vous avez besoin [d’Azure CLI](/cli/azure/install-azure-cli).

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-vm-ubuntu-DSVM-GPU-or-CPU/).

:::code language="json" source="~/quickstart-templates/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json":::

Les ressources suivantes sont définies dans le modèle :

* [Microsoft.Network/networkInterfaces](/azure/templates/microsoft.network/networkinterfaces)
* [Microsoft.Network/networkSecurityGroups](/azure/templates/microsoft.network/networksecuritygroups)
* [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
* [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.Compute/virtualMachines](/azure/templates/microsoft.compute/virtualmachines) : Créez une machine virtuelle basée sur le cloud. Dans ce modèle, la machine virtuelle est configurée en tant que machine Data Science Virtual Machine exécutant Ubuntu 18.04.

## <a name="deploy-the-template"></a>Déployer le modèle

Pour utiliser le modèle à partir de l’interface Azure CLI, connectez-vous, puis choisissez votre abonnement (Voir [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli)). Ensuite, exécutez :

```azurecli-interactive
read -p "Enter the name of the resource group to create:" resourceGroupName &&
read -p "Enter the Azure location (e.g., centralus):" location &&
read -p "Enter the authentication type (must be 'password' or 'sshPublicKey') :" authenticationType &&
read -p "Enter the login name for the administrator account (may not be 'admin'):" adminUsername &&
read -p "Enter administrator account secure string (value of password or ssh public key):" adminPasswordOrKey &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-ubuntu-DSVM-GPU-or-CPU/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri $templateUri --parameters adminUsername=$adminUsername authenticationType=$authenticationType adminPasswordOrKey=$adminPasswordOrKey &&
echo "Press [ENTER] to continue ..." &&
read
```

Lorsque vous exécutez la commande ci-dessus, indiquez :

1. Le nom du groupe de ressources que vous voulez créer pour contenir la machine virtuelle DSVM et les ressources associées.
1. L’emplacement Azure dans lequel vous souhaitez effectuer le déploiement.
1. Le type d’authentification que vous voulez utiliser (entrez la chaîne `password` ou `sshPublicKey`).
1. Le nom de connexion du compte administrateur (cette valeur ne peut pas être `admin`).
1. La valeur du mot de passe ou de la clé publique SSH pour le compte.

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Pour voir votre machine Data Science Virtual Machine :

1. Accédez au [Portail Azure](https://portal.azure.com).
1. Connectez-vous.
1. Utilisez le groupe de ressources que vous venez de créer.

Les informations du groupe de ressources s’affichent :

:::image type="content" source="media/dsvm-tutorial-resource-manager/resource-group-home.png" alt-text="Capture d’écran d’un groupe de ressources de base contenant une machine virtuelle DSVM":::

Cliquez sur la ressource de machine virtuelle pour accéder à la page d’informations correspondante. Vous y trouverez des informations sur la machine virtuelle, notamment les détails de connexion.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne voulez pas utiliser cette machine virtuelle, supprimez-la. Étant donné que la machine DSVM est associée à d’autres ressources, comme un compte de stockage, vous voudrez probablement supprimer l’intégralité du groupe de ressources que vous avez créé. Vous pouvez supprimer le groupe de ressources par l’intermédiaire du portail en cliquant sur le bouton **Supprimer** et en confirmant ce choix. Vous pouvez également supprimer le groupe de ressources à partir de l’interface CLI avec :

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une machine virtuelle Data Science Virtual Machine à partir d’un modèle Resource Manager.

> [!div class="nextstepaction"]
> [Exemples de programmes et procédures pas à pas sur le machine learning](dsvm-samples-and-walkthroughs.md)
