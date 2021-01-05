---
title: Mise à jour d’un microprogramme d’appareil avec Azure IoT Hub | Microsoft Docs
description: Découvrez comment implémenter un processus de mise à jour de microprogramme d’appareil, qui peut être déclenché à partir d’une application back-end connectée à votre hub IoT.
services: iot-hub
author: wesmc7777
ms.author: wesmc
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 06/28/2019
ms.custom:
- mvc
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- devx-track-js
- devx-track-azurecli
ms.openlocfilehash: b4de685accf665c7555a454ef247ddf589c6ba5f
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96572335"
---
# <a name="tutorial-implement-a-device-firmware-update-process"></a>Tutoriel : Implémenter un processus de mise à jour de microprogramme d’appareil

Vous devrez peut-être mettre à jour le microprogramme sur les appareils connectés à votre hub IoT. Par exemple, il peut être nécessaire d’ajouter de nouvelles fonctionnalités au microprogramme ou d’appliquer des correctifs de sécurité. Dans de nombreux scénarios IoT, il est difficile d’intervenir physiquement puis d’appliquer manuellement les mises à jour du microprogramme à vos appareils. Ce tutoriel montre comment démarrer et surveiller le processus de mise à jour du microprogramme à distance via une application back-end connectée à votre hub.

Pour créer et surveiller le processus de mise à jour du microprogramme, l’application back-end de ce tutoriel crée une _configuration_ dans votre hub IoT. La [gestion automatique des appareils](./iot-hub-automatic-device-management.md) IoT Hub utilise cette configuration pour mettre à jour un ensemble de _propriétés souhaitées du jumeau d’appareil_ sur tous vos appareils de refroidissement. Les propriétés souhaitées spécifient les détails de la mise à jour du microprogramme requise. Pendant que les appareils de refroidissement exécutent le processus de mise à jour du microprogramme, ils signalent leur état à l’application back-end à l’aide des _propriétés signalées du jumeau d’appareil_. L’application back-end peut utiliser la configuration pour surveiller les propriétés signalées envoyées depuis l’appareil et suivre le processus de mise à jour du microprogramme jusqu'à la fin :

![Processus de mise à jour du microprogramme](media/tutorial-firmware-update/Process.png)

Dans ce tutoriel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
> * Créer un hub IoT et ajouter un appareil de test dans le registre des identités des appareils.
> * Créer une configuration pour définir la mise à jour du microprogramme.
> * Simuler le processus de mise à jour du microprogramme sur un appareil.
> * Recevoir des mises à jour de l’état de l’appareil tandis que la mise à jour du microprogramme progresse.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

Les deux exemples d’applications que vous exécutez dans ce guide de démarrage rapide sont écrits à l’aide de Node.js. Votre ordinateur de développement doit disposer de Node.js v10.x.x ou ultérieur.

Vous pouvez télécharger Node.js pour plusieurs plateformes sur [nodejs.org](https://nodejs.org).

Vous pouvez vérifier la version actuelle de Node.js sur votre machine de développement à l’aide de la commande suivante :

```cmd/sh
node --version
```

Téléchargez l’exemple de projet Node.js à partir de https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip et extrayez l’archive ZIP.

Assurez-vous que le port 8883 est ouvert dans votre pare-feu. L'exemple d’appareil de ce tutoriel utilise le protocole MQTT qui communique sur le port 8883. Dans certains environnements réseau professionnels et scolaires, ce port peut être bloqué. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="set-up-azure-resources"></a>Configurer les ressources Azure

Pour effectuer ce tutoriel, votre abonnement Azure doit contenir un hub IoT avec un appareil ajouté au registre des identités des appareils. L’entrée dans le registre des identités des appareils permet à l’appareil simulé que vous exécutez dans ce didacticiel de se connecter à votre hub.

Si vous n’avez pas encore défini de hub IoT dans votre abonnement, vous pouvez en définir un à l’aide du script d’interface de ligne de commande suivant. Ce script utilise le nom **tutorial-iot-hub** pour le hub IoT. Vous devrez remplacer ce nom par le nom de votre choix au moment d’exécuter le hub. Le script crée le groupe de ressources et le hub dans la région **USA Centre** que vous pouvez remplacer par une région plus proche de chez vous. Le script récupère la chaîne de connexion de service de votre hub IoT, qui vous permet de vous connecter à votre hub IoT depuis l’exemple d’application back-end :

```azurecli-interactive
hubname=tutorial-iot-hub
location=centralus

# Install the IoT extension if it's not already installed
az extension add --name azure-iot

# Create a resource group
az group create --name tutorial-iot-hub-rg --location $location

# Create your free-tier IoT Hub. You can only have one free IoT Hub per subscription
az iot hub create --name $hubname --location $location --resource-group tutorial-iot-hub-rg --sku F1

# Make a note of the service connection string, you need it later
az iot hub show-connection-string --name $hubname --policy-name service -o table

```

Ce tutoriel utilise un appareil simulé appelé **MyFirmwareUpdateDevice**. Le script suivant ajoute cet appareil dans le registre des identités des appareils, définit une valeur de balise et récupère sa chaîne de connexion :

```azurecli-interactive
# Set the name of your IoT hub
hubname=tutorial-iot-hub

# Create the device in the identity registry
az iot hub device-identity create --device-id MyFirmwareUpdateDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg

# Add a device type tag
az iot hub device-twin update --device-id MyFirmwareUpdateDevice --hub-name $hubname --set tags='{"devicetype":"chiller"}'

# Retrieve the device connection string, you need this later
az iot hub device-identity show-connection-string --device-id MyFirmwareUpdateDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg -o table

```

> [!TIP]
> Si vous exécutez ces commandes à partir d’une invite de commandes Windows ou d’une invite de commandes Powershell, consultez la page [azure-iot-cli-extension tips](https://github.com/Azure/azure-iot-cli-extension/wiki/Tips) pour plus d’informations sur la façon de citer les chaînes JSON.

## <a name="start-the-firmware-update"></a>Démarrer la mise à jour du microprogramme

Vous devez créer une [configuration de gestion automatique des appareils](iot-hub-automatic-device-management.md#create-a-configuration) dans l’application back-end pour commencer le processus de mise à jour du microprogramme sur tous les appareils comportant un **devicetype** de refroidissement. Dans cette section, vous allez apprendre à :

* Créer une configuration à partir d’une application back-end.
* Surveiller la tâche jusqu'à son achèvement.

### <a name="use-desired-properties-to-start-the-firmware-upgrade-from-the-back-end-application"></a>Utiliser les propriétés souhaitées pour démarrer la mise à jour du microprogramme à partir de l’application back-end

Pour afficher le code d’application back-end qui crée la configuration, accédez au dossier **iot-hub/Tutorials/FirmwareUpdate** dans l’exemple de projet Node.js que vous avez téléchargé. Ouvrez ensuite le fichier ServiceClient.js dans un éditeur de texte.

L’application back-end crée la configuration suivante :

[!code-javascript[Automatic device management configuration](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=configuration "Automatic device management configuration")]

La configuration inclut les sections suivantes :

* `content` spécifie les propriétés souhaitées du microprogramme envoyées aux appareils sélectionnés.
* `metrics` spécifie les requêtes qui vont exécuter le rapport d’état de la mise à jour du microprogramme.
* `targetCondition` sélectionne les appareils qui vont recevoir la mise à jour du microprogramme.
* `priorty` définit la priorité relative de cette configuration sur d’autres configurations.

L’application back-end utilise le code suivant pour créer la configuration afin de définir les propriétés souhaitées :

[!code-javascript[Create configuration](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=createConfiguration "Create configuration")]

Après avoir créé la configuration, l’application surveille la mise à jour du microprogramme :

[!code-javascript[Monitor firmware update](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=monitorConfiguration "Monitor firmware update")]

Une configuration signale deux types de métriques :

* Les métriques système qui indiquent combien d’appareils sont ciblés et combien d’appareils sont déjà mis à jour.
* Les mesures personnalisées générées par les requêtes que vous ajoutez à la configuration. Dans ce tutoriel, les requêtes indiquent combien d’appareils sont à chaque étape du processus de mise à jour.

### <a name="respond-to-the-firmware-upgrade-request-on-the-device"></a>Répondre à la requête de mise à niveau du microprogramme sur l’appareil

Pour afficher le code d’appareil simulé qui gère les propriétés souhaitées du microprogramme envoyées à l’application back-end, accédez au dossier **iot-hub/Tutorials/FirmwareUpdate** dans l’exemple de projet Node.js que vous avez téléchargé. Ouvrez ensuite le fichier SimulatedDevice.js dans un éditeur de texte.

L’application d’appareil simulé crée un gestionnaire de mises à jour des propriétés souhaitées **properties.desired.firmware** dans le jumeau d’appareil. Dans le gestionnaire, des vérifications basiques sont exécutées sur les propriétés souhaitées avant le début du processus de mise à jour :

[!code-javascript[Handle desired property update](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=initiateUpdate "Handle desired property update")]

## <a name="update-the-firmware"></a>Mettre à jour le microprogramme

La fonction **initiateFirmwareUpdateFlow** exécute la mise à jour. Cette fonction utilise la fonction **cascade** pour exécuter les phases du processus de mise à jour dans l’ordre. Dans cet exemple, la mise à jour du microprogramme comporte quatre phases. La première phase télécharge l’image, la seconde phase vérifie l’image à l’aide d’une somme de contrôle, la troisième phase applique l’image et la dernière phase redémarre l’appareil :

[!code-javascript[Firmware update flow](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=firmwareupdateflow "Firmware update flow")]

Pendant le processus de mise à jour, l’appareil simulé génère des rapports sur la progression à l’aide de propriétés signalées :

[!code-javascript[Reported properties](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=reportedProperties "Reported properties")]

L’extrait de code suivant illustre l’implémentation de la phase de téléchargement. Pendant la phase de téléchargement, l’appareil simulé utilise les propriétés signalées pour envoyer des informations sur l’état à l’application back-end :

[!code-javascript[Download image phase](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=downloadimagephase "Download image phase")]

## <a name="run-the-sample"></a>Exécution de l'exemple

Dans cette section, vous exécutez deux exemples d’applications pour observer une application back-end créer la configuration pour gérer le processus de mise à jour du microprogramme sur l’appareil simulé.

Pour exécuter l’appareil simulé et les applications back-end, vous avez besoin des chaînes de connexion du service et de l’appareil. Vous avez noté ces chaînes de connexion lorsque vous avez créé les ressources au début de ce didacticiel.

Pour exécuter l’application d’appareil simulé, ouvrez une fenêtre d’interpréteur de commandes ou d’invite de commandes et accédez au dossier **iot-hub/Tutorials/FirmwareUpdate** dans le projet Node.js que vous avez téléchargé. Exécutez ensuite les commande suivantes :

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

Pour exécuter l’application back-end, ouvrez une autre fenêtre d’interpréteur de commandes ou d’invite de commandes. Puis, accédez au dossier **iot-hub/Tutorials/FirmwareUpdate** dans le projet Node.js que vous avez téléchargé. Exécutez ensuite les commande suivantes :

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

La capture d’écran suivante montre le résultat de l’application d’appareil simulé et comment celle-ci répond à la mise à jour des propriétés souhaitées du microprogramme à partir de l’application back-end :

![Appareil simulé](./media/tutorial-firmware-update/SimulatedDevice.png)

La capture d’écran suivante montre le résultat de l’application back-end et comment celle-ci crée la configuration pour mettre à jour les propriétés souhaitées du microprogramme :

![Capture d’écran montrant la sortie de l’application back-end.](./media/tutorial-firmware-update/BackEnd1.png)

La capture d’écran suivante montre le résultat de l’application back-end et comment celle-ci surveille les métriques de mises à jour du microprogramme à partir de l’appareil simulé :

![Application back-end](./media/tutorial-firmware-update/BackEnd2.png)

Comme les configurations d’appareil automatiques s’exécutent au moment de la création, puis toutes les cinq minutes, vous ne voyez pas nécessaire chaque mise à jour d’état envoyée à l’application back-end. Vous pouvez également afficher les métriques dans le portail dans la section **Gestion automatique des appareils -> Configuration de l’appareil IoT** de votre hub IoT :

![Afficher la configuration dans le portail](./media/tutorial-firmware-update/portalview.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’effectuer le didacticiel suivant, conservez le groupe de ressources et l’IoT Hub afin de les réutiliser ultérieurement.

Si vous n’avez plus besoin du hub IoT, supprimez-le ainsi que le groupe de ressources dans le portail. Pour ce faire, sélectionnez le groupe de ressources **tutorial-iot-hub-rg** qui contient votre hub IoT, puis cliquez sur **Supprimer**.

Vous pouvez également utiliser l’interface de ligne de commande :

```azurecli-interactive
# Delete your resource group and its contents
az group delete --name tutorial-iot-hub-rg
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à implémenter un processus de mise à jour de microprogramme pour vos appareils connectés. Passez au tutoriel suivant pour apprendre à utiliser les outils du portail Azure IoT Hub et les commandes Azure CLI pour tester la connectivité de l’appareil.

> [!div class="nextstepaction"]
> [Utiliser un appareil simulé pour tester la connectivité avec votre hub IoT](tutorial-connectivity.md)
