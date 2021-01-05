---
title: Démarrage rapide – Configurer le service Azure IoT Hub Device Provisioning avec Azure CLI
description: Démarrage rapide - Configurer le service Azure IoT Hub Device Provisioning (DPS) avec Azure CLI
author: wesmc7777
ms.author: wesmc
ms.date: 11/08/2019
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 611068fa020321be88be6e1d6da663266029c658
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660183"
---
# <a name="quickstart-set-up-the-iot-hub-device-provisioning-service-with-azure-cli"></a>Démarrage rapide : Configurer le service IoT Hub Device Provisioning avec Azure CLI

L’interface de ligne de commande (CLI) Azure permet de créer et gérer des ressources Azure à partir de la ligne de commande ou dans les scripts. Ce guide de démarrage rapide explique en détail comment utiliser Azure CLI pour créer un hub IoT et un service IoT Hub Device Provisioning et comment lier les deux. 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

> [!IMPORTANT]
> Le hub IoT et le service de provisionnement que vous créez dans ce guide de démarrage rapide seront détectables publiquement comme points de terminaison DNS. Veillez à éviter toute information sensible si vous décidez de modifier les noms utilisés pour ces ressources.
>

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]


## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. 

L’exemple suivant crée un groupe de ressources nommé *my-sample-resource-group* à l’emplacement *westus*.

```azurecli-interactive 
az group create --name my-sample-resource-group --location westus
```

> [!TIP]
> L’exemple crée le groupe de ressources dans l’emplacement USA Ouest. Vous pouvez afficher une liste des emplacements disponibles en exécutant la commande `az account list-locations -o table`.
>
>

## <a name="create-an-iot-hub"></a>Créer un hub IoT

Créez un hub IoT avec la commande [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create).

L’exemple suivant crée un hub IoT nommé *my-sample-hub* à l’emplacement *westus*. Un nom de hub IoT doit être globalement unique dans Azure. Vous pouvez donc ajouter un préfixe ou un suffixe unique à l’exemple de nom ou choisir un nouveau nom. Vérifiez que votre nom respecte les conventions de nommage pour les hubs IoT, à savoir qu’il doit comprendre entre 3 et 50 caractères et contenir uniquement des caractères alphanumériques en minuscules ou majuscules ou des traits d’union (« - »). 

```azurecli-interactive 
az iot hub create --name my-sample-hub --resource-group my-sample-resource-group --location westus
```

## <a name="create-a-device-provisioning-service"></a>Créer un service Device Provisioning

Créez un service Device Provisioning avec la commande [az iot dps create](/cli/azure/iot/dps#az-iot-dps-create). 

L’exemple suivant crée un service de provisionnement nommé *my-sample-dps* à l’emplacement *westus*. Vous devez également choisir un nom global unique pour votre propre service de provisionnement. Vérifiez qu’il respecte les conventions de nommage pour les services IoT Hub Device Provisioning, à savoir qu’il doit comprendre entre 3 et 64 caractères et contenir uniquement des caractères alphanumériques en minuscules ou majuscules ou des traits d’union (« - »).

```azurecli-interactive 
az iot dps create --name my-sample-dps --resource-group my-sample-resource-group --location westus
```

> [!TIP]
> L’exemple crée le service d’approvisionnement dans l’emplacement USA Ouest. Vous pouvez afficher une liste des emplacements disponibles en exécutant la commande `az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table` ou en vous rendant sur la page [Statut Azure](https://azure.microsoft.com/status/) et en recherchant « Device Provisioning ». Dans les commandes, les emplacements peuvent être spécifiés dans un format d’un mot ou de plusieurs mots. Par exemple : westus, West US, WEST US, etc. La valeur ne respecte pas la casse. Si vous utilisez un format de plusieurs mots pour spécifier l’emplacement, entourez la valeur d’apostrophes. Par exemple, `--location "West US"`.
>

## <a name="get-the-connection-string-for-the-iot-hub"></a>Obtenir la chaîne de connexion pour le hub IoT

Il faut que la chaîne de connexion de votre hub IoT lie ce dernier au service Device Provisioning. Utilisez la commande [az iot hub show-connection-string](/cli/azure/iot/hub#az-iot-hub-show-connection-string) pour obtenir la chaîne de connexion et utilisez sa sortie pour définir une variable à utiliser quand vous lierez les deux ressources. 

L’exemple suivant définit la variable *hubConnectionString* avec la valeur de la chaîne de connexion pour la clé primaire de la stratégie *iothubowner* du hub (le paramètre `--policy-name` peut être utilisé pour spécifier une suivre différente). Remplacez *my-sample-hub* par le nom unique du hub IoT que vous avez choisi précédemment. La commande utilise les options de [requête](/cli/azure/query-azure-cli) et de [sortie](/cli/azure/format-output-azure-cli#tsv-output-format) d’Azure CLI pour extraire la chaîne de connexion de la sortie de la commande.

```azurecli-interactive 
hubConnectionString=$(az iot hub show-connection-string --name my-sample-hub --key primary --query connectionString -o tsv)
```

Vous pouvez utiliser la commande `echo` pour voir la chaîne de connexion.

```azurecli-interactive 
echo $hubConnectionString
```

> [!NOTE]
> Ces deux commandes sont valides pour un hôte exécuté par Bash.
> 
> Si vous utilisez un interpréteur de commandes Windows/CMD local ou un hôte PowerShell, modifiez les commandes pour utiliser la syntaxe appropriée à cet environnement.
>
> Si vous utilisez Azure Cloud Shell, vérifiez que la liste déroulante de l’environnement sur le côté gauche de la fenêtre de l’interpréteur de commandes indique **Bash**.
>

## <a name="link-the-iot-hub-and-the-provisioning-service"></a>Lier le hub IoT et le service d’approvisionnement

Liez le hub IoT et votre service d’approvisionnement avec la commande [az iot dps linked-hub create](/cli/azure/iot/dps/linked-hub#az-iot-dps-linked-hub-create). 

L’exemple suivant lie un hub IoT nommé *my-sample-hub* à l’emplacement *westus* et un service Device Provisioning nommé *my-sample-dps*. Remplacez ces noms par les noms uniques du hub IoT et du service Device Provisioning que vous avez choisis précédemment. La commande utilise la chaîne de connexion pour votre hub IoT qui a été stockée dans la variable *hubConnectionString* à l’étape précédente.

```azurecli-interactive 
az iot dps linked-hub create --dps-name my-sample-dps --resource-group my-sample-resource-group --connection-string $hubConnectionString --location westus
```

La commande peut prendre quelques minutes.

## <a name="verify-the-provisioning-service"></a>Vérifier le service d’approvisionnement

Obtenez les informations de votre service d’approvisionnement avec la commande [az iot dps show](/cli/azure/iot/dps#az-iot-dps-show).

L’exemple suivant obtient les informations d’un service d’approvisionnement nommé *my-sample-dps*. Remplacez ce nom par le nom de votre propre service Device Provisioning.

```azurecli-interactive
az iot dps show --name my-sample-dps
```
Le hub IoT lié est affiché dans la collection *properties.iotHubs*.

![Vérifier le service de provisionnement](./media/quick-setup-auto-provision-cli/verify-provisioning-service.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Les autres démarrages rapides de cette collection reposent sur ce démarrage rapide. Si vous souhaitez continuer à utiliser d’autres démarrages rapides ou les didacticiels, ne nettoyez pas les ressources créées dans ce démarrage rapide. Si vous n’envisagez pas de continuer, vous pouvez utiliser les commandes suivantes pour supprimer le service d’approvisionnement, le hub IoT et le groupe de ressources et toutes ses ressources. Remplacez les noms des ressources écrites ci-dessous par les noms de vos propres ressources.

Pour supprimer le service d’approvisionnement, exécutez la commande [az iot dps delete](/cli/azure/iot/dps#az-iot-dps-delete) :

```azurecli-interactive
az iot dps delete --name my-sample-dps --resource-group my-sample-resource-group
```
Pour supprimer le hub IoT, exécutez la commande [az iot hub delete](/cli/azure/iot/hub#az-iot-hub-delete) :

```azurecli-interactive
az iot hub delete --name my-sample-hub --resource-group my-sample-resource-group
```

Pour supprimer un groupe de ressources et toutes ses ressources, exécutez la commande [az group delete](/cli/azure/group#az-group-delete) :

```azurecli-interactive
az group delete --name my-sample-resource-group
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé un hub IoT et une instance de service Device Provisioning, puis vous avez lié ces deux ressources. Pour savoir comment utiliser cette configuration pour provisionner un appareil simulé, passez au guide de démarrage rapide relatif à la création d’un appareil simulé.

> [!div class="nextstepaction"]
> [Démarrage rapide pour créer un appareil simulé](./quick-create-simulated-device.md)
