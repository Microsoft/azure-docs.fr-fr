---
title: Gérer le service Azure IoT Hub Device Provisioning avec Azure CLI et une extension IoT
description: Découvrez comment utiliser Azure CLI et l’extension IoT pour gérer le service IoT Hub Device Provisioning (DPS)
author: chrissie926
ms.author: menchi
ms.date: 01/17/2018
ms.topic: conceptual
ms.service: iot-dps
ms.custom: devx-track-azurecli
services: iot-dps
ms.openlocfilehash: 020b0dbc937885923b26c115f91147437b7a0f9b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964726"
---
# <a name="how-to-use-azure-cli-and-the-iot-extension-to-manage-the-iot-hub-device-provisioning-service"></a>Comment utiliser Azure CLI et l’extension IoT afin de gérer le Service IoT Hub Device Provisioning

[Azure CLI](/cli/azure?view=azure-cli-latest) est un outil en ligne de commande open source et multiplateforme, destiné à la gestion des ressources Azure, telles que IoT Edge. Azure CLI est disponible sous Windows, Linux et macOS. Azure CLI vous permet de gérer les ressources Azure IoT Hub, les instances du service Device Provisioning et les hubs liés prêts à l’utilisation.

L’extension IoT enrichit Azure CLI avec des fonctionnalités telles que la gestion des appareils, et toutes les fonctionnalités de IoT Edge.

Dans ce tutoriel, vous commencez par les étapes de configuration d’Azure CLI et de l’extension IoT. Ensuite, vous voyez comment exécuter des commandes CLI afin d’effectuer des opérations de base dans le service Device Provisioning. 

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="installation"></a>Installation 

### <a name="install-python"></a>Installation de Python

[Python 2.7x ou Python 3.x](https://www.python.org/downloads/) est nécessaire.

### <a name="install-the-azure-cli"></a>Installer l’interface de ligne de commande Microsoft Azure

Suivez les [instructions d’installation](/cli/azure/install-azure-cli?view=azure-cli-latest) pour configurer Azure CLI dans votre environnement. Vous devez utiliser Azure CLI version 2.0.70 (au minimum) ou ultérieure. Utilisez `az –version` pour valider. Cette version prend en charge les commandes d’extension az et introduit l’infrastructure de la commande Knack. Pour une installation sur Windows, le plus simple consiste à télécharger et à installer le [MSI](https://aka.ms/InstallAzureCliWindows).

### <a name="install-iot-extension"></a>Installer l’extension IoT

Le [Lisez-moi de l’extension IoT](https://github.com/Azure/azure-iot-cli-extension) décrit différentes manières d’installer l’extension. Le plus simple consiste à exécuter `az extension add --name azure-iot`. Après l’installation, vous pouvez utiliser `az extension list` pour valider les extensions actuellement installées ou `az extension show --name azure-iot` pour afficher les détails concernant l’extension IoT. Pour supprimer l’extension, vous pouvez utiliser `az extension remove --name azure-iot`.


## <a name="basic-device-provisioning-service-operations"></a>Opérations de base du service Device Provisioning

L’exemple vous montre comment vous connecter à votre compte Azure, créer un groupe de ressources Azure (un conteneur contenant les ressources associées à une solution Azure), créer un IoT Hub, créer un service Device Provisioning, faire la liste des services Device Provisioning existants et créer un hub IoT lié avec les commandes CLI. 

Effectuez les étapes d’installation décrites précédemment avant de commencer. Si vous ne possédez pas de compte Azure, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free/?v=17.39a) dès maintenant. 


### <a name="1-log-in-to-the-azure-account"></a>1. Se connecter au compte Azure
  
```azurecli
az login
```

![Capture d'écran d'une fenêtre d'invite de commandes qui exécute la commande az login.](./media/how-to-manage-dps-with-cli/login.jpg)

### <a name="2-create-a-resource-group-iothubblogdemo-in-eastus"></a>2. Créer un groupe de ressources IoTHubBlogDemo dans eastus

```azurecli
az group create -l eastus -n IoTHubBlogDemo
```

![Créer un groupe de ressources](./media/how-to-manage-dps-with-cli/create-resource-group.jpg)


### <a name="3-create-two-device-provisioning-services"></a>3. Créer deux services Device Provisioning

```azurecli
az iot dps create --resource-group IoTHubBlogDemo --name demodps
```

![Créer un service Device Provisioning](./media/how-to-manage-dps-with-cli/create-dps.jpg)

```azurecli
az iot dps create --resource-group IoTHubBlogDemo --name demodps2
```

### <a name="4-list-all-the-existing-device-provisioning-services-under-this-resource-group"></a>4. Faire la liste de tous les services Device Provisioning sous ce groupe de ressources

```azurecli
az iot dps list --resource-group IoTHubBlogDemo
```

![Faire la liste de tous les services Device Provisioning](./media/how-to-manage-dps-with-cli/list-dps.jpg)


### <a name="5-create-an-iot-hub-blogdemohub-under-the-newly-created-resource-group"></a>5. Créer un hub IoT blogDemoHub sous le groupe de ressources nouvellement créé

```azurecli
az iot hub create --name blogDemoHub --resource-group IoTHubBlogDemo
```

![Créer un hub IoT](./media/how-to-manage-dps-with-cli/create-hub.jpg)

### <a name="6-link-one-existing-iot-hub-to-a-device-provisioning-service"></a>6. Lier un hub IoT existant à un service Device Provisioning

```azurecli
az iot dps linked-hub create --resource-group IoTHubBlogDemo --dps-name demodps --connection-string <connection string> -l westus
```

![Lier un hub](./media/how-to-manage-dps-with-cli/create-hub.jpg)

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Inscrire l’appareil
> * Démarrer l’appareil
> * Vérifier que l’appareil est enregistré

Passez au didacticiel suivant pour apprendre à approvisionner plusieurs appareils sur des hubs à charge équilibrée. 

> [!div class="nextstepaction"]
> [Approvisionner des appareils sur des hubs IoT à charge équilibrée](./tutorial-provision-multiple-hubs.md)