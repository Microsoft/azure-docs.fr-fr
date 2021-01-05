---
title: Démarrage rapide – Créer un service Azure IoT Hub Device Provisioning avec un modèle Azure Resource Manager (ARM)
description: Démarrage rapide – Découvrez comment créer un service Azure IoT Hub Device Provisioning avec un modèle Azure Resource Manager (ARM).
author: wesmc7777
ms.author: wesmc
ms.date: 12/03/2020
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.custom: mvc, subject-armqs, devx-track-azurecli
ms.openlocfilehash: 73beed4e4262d911f68c2b4b33bc0c1ee24164f8
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746202"
---
# <a name="quickstart-set-up-the-iot-hub-device-provisioning-service-dps-with-an-arm-template"></a>Démarrage rapide : Configurer le service IoT Hub Device Provisioning avec un modèle ARM

Vous pouvez utiliser un modèle [Azure Resource Manager](../azure-resource-manager/management/overview.md) (ARM) pour configurer par programmation les ressources du cloud Azure nécessaires au provisionnement de vos appareils. Ces étapes montrent comment créer un hub IoT et un service IoT Hub Device Provisioning avec un modèle ARM. Le hub IoT est également lié à la ressource DPS utilisant le modèle. Cette liaison permet à la ressource DPS d’affecter des appareils au hub en fonction des stratégies d’allocation que vous configurez.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Ce démarrage rapide utilise le [portail Azure](../azure-resource-manager/templates/deploy-portal.md) et [Azure CLI](../azure-resource-manager/templates/deploy-cli.md) pour effectuer les étapes de programmation nécessaires pour créer un groupe de ressources et déployer le modèle. Cependant, vous pouvez facilement utiliser [PowerShell](../azure-resource-manager/templates/deploy-powershell.md), .NET, Ruby ou d’autres langages de programmation pour effectuer ces étapes et déployer votre modèle. 

Si votre environnement satisfait les prérequis et si vous êtes déjà habitué à utiliser des modèles ARM, le fait de sélectionner le bouton **Déployer sur Azure** ci-dessous a pour effet d’ouvrir le modèle pour un déploiement sur le portail Azure.

[![Vue d’ensemble du déploiement sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-iothub-device-provisioning%2fazuredeploy.json)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]


## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-iothub-device-provisioning/).

:::code language="json" source="~/quickstart-templates/101-iothub-device-provisioning/azuredeploy.json":::

Deux ressources Azure sont définies dans le modèle ci-dessus :

* [**Microsoft.Devices/iothubs**](/azure/templates/microsoft.devices/iothubs) : Crée un hub IoT Azure.
* [**Microsoft.Devices/provisioningservices**](/azure/templates/microsoft.devices/provisioningservices) : Crée un service Azure IoT Hub Device Provisioning avec le hub IoT nouveau qui lui est déjà lié.


## <a name="deploy-the-template"></a>Déployer le modèle

#### <a name="deploy-with-the-portal"></a>Déployer avec le portail

1. Sélectionnez l’image suivante pour vous connecter à Azure et ouvrir le modèle pour le déploiement. Le modèle crée un hub IoT et une ressource DPS. Le hub est lié dans la ressource DPS.

    [![Étapes de déploiement sur Azure à partir du portail](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-iothub-device-provisioning%2fazuredeploy.json)

2. Sélectionnez ou entrez les valeurs suivantes, puis cliquez sur **Vérifier + créer**.

    ![Paramètres de déploiement de modèle ARM sur le portail](./media/quick-setup-auto-provision-rm/arm-template-deployment-parameters-portal.png)    

    À moins qu’elle soit spécifiée ci-dessous, utilisez la valeur par défaut pour créer le hub IoT et la ressource DPS.

    | Champ | Description |
    | :---- | :---------- |
    | **Abonnement** | Sélectionnez votre abonnement Azure. |
    | **Groupe de ressources** | Cliquez sur **Créer nouveau**, entrez un nom unique pour le groupe de ressources, puis cliquez sur **OK**. |
    | **Région** | Sélectionnez une région pour vos ressources. Par exemple, **USA Est**. |
    | **Nom du hub IoT** | Entrez un nom pour le hub IoT qui doit être globalement unique dans l’espace de noms *.azure-devices.net*. Vous aurez besoin du nom du hub dans la section suivante au moment de valider le déploiement. |
    | **Nom du service de provisionnement** | Entrez un nom pour la nouvelle ressource Device Provisioning Service (DPS). Le nom doit être globalement unique au sein de l’espace de noms *.azure-devices-provisioning.net*. Vous aurez besoin du nom DPS dans la section suivante au moment de valider le déploiement. |
    
3. Dans l’écran suivant, lisez les conditions générales. Si vous acceptez toutes les conditions, cliquez sur **Créer**. 

    Le déploiement prend quelques instants. 

    Outre le portail Azure, vous pouvez également utiliser Azure PowerShell, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-powershell.md).


#### <a name="deploy-with-the-azure-cli"></a>Déployer avec Azure CLI

L’utilisation d’Azure CLI nécessite la version 2.6 ou ultérieure. Si vous exécutez Azure CLI localement, vérifiez votre version en exécutant : `az --version`

Vous connecter à votre compte Azure et sélectionner votre abonnement.

1. Si vous exécutez Azure CLI localement au lieu de l’exécuter sur le portail, vous devez vous connecter. Pour vous connecter à l’invite de commande, exécutez la [commande login](/cli/azure/get-started-with-az-cli2) :
    
    ```azurecli
    az login
    ```

    Suivez les instructions pour vous authentifier à l’aide du code et vous connecter à votre compte Azure via un navigateur web.

2. Si vous possédez plusieurs abonnements Azure, la connexion à Azure vous donne accès à tous les abonnements Azure associés à vos informations d’identification. Utilisez la [commande pour répertorier les comptes Azure](/cli/azure/account) ci-dessous :
    
    ```azurecli
    az account list -o table
    ```

    Utilisez la commande suivante pour sélectionner l’abonnement que vous souhaitez utiliser pour exécuter les commandes en vue de créer votre hub IoT et vos ressources DPS. Vous pouvez utiliser le nom de l’abonnement ou l’ID de la sortie de la commande précédente :

    ```azurecli
    az account set --subscription {your subscription name or id}
    ```

3. Copiez et collez les commandes suivantes dans votre invite CLI. Exécutez ensuite les commandes en appuyant sur **ENTRÉE**.
   
    > [!TIP]
    > Les commandes vous invitent à entrer un emplacement de groupe de ressources. Vous pouvez afficher la liste des emplacements disponibles en exécutant d’abord la commande :
    >
    > `az account list-locations -o table`
    >
    >
    
    ```azurecli-interactive
    read -p "Enter a project name that is used for generating resource names:" projectName &&
    read -p "Enter the location (i.e. centralus):" location &&
    templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-iothub-device-provisioning/azuredeploy.json" &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location "$location" &&
    az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
    echo "Press [ENTER] to continue ..." &&
    read
    ```

4. Les commandes vous invitent à entrer les informations suivantes. Fournissez chaque valeur et appuyez sur **ENTRÉE**.

    | Paramètre | Description |
    | :-------- | :---------- |
    | **Nom du projet** | La valeur de ce paramètre sera utilisée pour créer un groupe de ressources qui contiendra toutes les ressources. La chaîne `rg` sera ajoutée à la fin de la valeur de nom de votre groupe de ressources. |
    | **location** | Cette valeur correspond à la région où résident toutes les ressources. |
    | **iotHubName** | Entrez un nom pour le hub IoT qui doit être globalement unique dans l’espace de noms *.azure-devices.net*. Vous aurez besoin du nom du hub dans la section suivante au moment de valider le déploiement. |
    | **provisioningServiceName** | Entrez un nom pour la nouvelle ressource Device Provisioning Service (DPS). Le nom doit être globalement unique au sein de l’espace de noms *.azure-devices-provisioning.net*. Vous aurez besoin du nom DPS dans la section suivante au moment de valider le déploiement. |

    Azure CLI est utilisé pour déployer le modèle. Outre Azure CLI, vous pouvez aussi utiliser Azure PowerShell, le portail Azure et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-powershell.md).


## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

1. Pour vérifier le déploiement, exécutez la [commande suivante pour lister les ressources](/cli/azure/resource?view=azure-cli-latest#az-resource-list&preserve-view=true) et recherchez le nouveau service de provisionnement et le hub IoT dans la sortie :

    ```azurecli
     az resource list -g "${projectName}rg"
    ```

2. Pour vérifier que le hub est déjà lié à la ressource DPS, exécutez la [commande show suivante de l’extension DPS](/cli/azure/iot/dps?view=azure-cli-latest#az_iot_dps_show&preserve-view=true).

    ```azurecli
     az iot dps show --name <Your provisioningServiceName>
    ```

    Remarquez les hubs qui sont liés sur le membre `iotHubs`.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Les autres démarrages rapides de cette collection reposent sur ce démarrage rapide. Si vous souhaitez continuer à utiliser d’autres démarrages rapides ou les didacticiels, ne nettoyez pas les ressources créées dans ce démarrage rapide. Si vous n’envisagez pas de continuer, vous pouvez utiliser le portail Azure ou Azure CLI pour supprimer le groupe de ressources et toutes ses ressources.

Pour supprimer un groupe de ressources et toutes ses ressources à partir du portail Azure, ouvrez simplement le groupe de ressources, puis cliquez sur **Supprimer le groupe de ressources** dans la partie supérieure.

Pour supprimer le groupe de ressources déployé à l’aide d’Azure CLI :

```azurecli
az group delete --name "${projectName}rg"
```

Vous pouvez également supprimer des groupes de ressources et des ressources individuelles à l’aide du portail Azure, de PowerShell, des API REST ou encore des kits SDK de plateforme prise en charge publiés pour Azure Resource Manager ou le service IoT Hub Device Provisioning.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé un hub IoT et une instance de service Device Provisioning, puis vous avez lié ces deux ressources. Pour savoir comment utiliser cette configuration pour provisionner un appareil, passez au démarrage rapide traitant de la création d’un appareil.

> [!div class="nextstepaction"]
> [Démarrage rapide pour provisionner un appareil](./quick-create-simulated-device-symm-key.md)

