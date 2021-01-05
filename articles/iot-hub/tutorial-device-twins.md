---
title: Synchroniser l’état des appareils à partir d’Azure IoT Hub | Microsoft Docs
description: Découvrez comment utiliser les jumeaux d’appareils pour configurer vos appareils à partir du cloud et recevoir des données d’état et de conformité de vos appareils.
services: iot-hub
author: wesmc7777
ms.author: wesmc
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 06/21/2019
ms.custom:
- mvc
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- devx-track-js
- devx-track-azurecli
ms.openlocfilehash: 9ec2c51f01d6b13f33bc2d537a8f73a6721967d4
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96572522"
---
<!-- **TODO** Update publish config with repo paths before publishing! -->

# <a name="tutorial-configure-your-devices-from-a-back-end-service"></a>Tutoriel : Configurer vos appareils à partir d’un service backend

Outre la réception de données de télémétrie à partir de vos appareils, vous devrez peut-être configurer vos appareils à partir de votre service back-end. Lorsque vous envoyez une configuration souhaitée à vos appareils, vous pouvez choisir de recevoir des mises à jour d’état et de conformité de ces appareils. Par exemple, vous pouvez définir une plage de température de fonctionnement cible pour un appareil ou collecter des informations sur la version du microprogramme auprès de vos appareils.

Pour synchroniser les informations d’état entre un appareil et un hub IoT, vous pouvez utiliser des _jumeaux d’appareil_. Un [jumeau d’appareil](iot-hub-devguide-device-twins.md) est un document JSON associé à un appareil spécifique et stocké par IoT Hub dans le cloud à partir duquel vous pouvez [interroger](iot-hub-devguide-query-language.md) le document. Un jumeau d’appareil contient des _propriétés souhaitées_, des _propriétés rapportées_ et des _balises_. Une propriété souhaitée est définie par une application back-end et lue par un appareil. Une propriété rapportée est définie par un appareil et lue par une application back-end. Une balise est définie par une application back-end et n’est jamais envoyée à un appareil. Les balises permettent d’organiser vos appareils. Ce didacticiel vous explique comment utiliser les propriétés souhaitées et rapportées pour synchroniser les informations d’état :

![Résumé sur les jumeaux d’appareil](media/tutorial-device-twins/DeviceTwins.png)

Dans ce didacticiel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
> * Créer un hub IoT et ajouter un appareil de test dans le registre des identités.
> * Utiliser les propriétés souhaitées pour envoyer des informations d’état à votre appareil simulé.
> * Utiliser les propriétés rapportées pour recevoir des informations d’état de votre appareil simulé.

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

Pour effectuer ce didacticiel, votre abonnement Azure doit contenir un hub IoT avec un appareil ajouté au registre des identités des appareils. L’entrée dans le registre des identités des appareils permet à l’appareil simulé que vous exécutez dans ce didacticiel de se connecter à votre hub.

Si vous n’avez pas encore défini de hub IoT dans votre abonnement, vous pouvez en définir un à l’aide du script d’interface de ligne de commande suivant. Ce script utilise le nom **tutorial-iot-hub** pour le hub IoT. Vous devrez remplacer ce nom par le nom de votre choix au moment d’exécuter le hub. Le script crée le groupe de ressources et le hub dans la région **USA Centre** que vous pouvez remplacer par une région plus proche de chez vous. Le script récupère la chaîne de connexion de service de votre hub IoT, qui vous permet de vous connecter à votre hub IoT depuis l’exemple d’application back-end :

```azurecli-interactive
hubname=tutorial-iot-hub
location=centralus

# Install the IoT extension if it's not already installed:
az extension add --name azure-iot

# Create a resource group:
az group create --name tutorial-iot-hub-rg --location $location

# Create your free-tier IoT Hub. You can only have one free IoT Hub per subscription:
az iot hub create --name $hubname --location $location --resource-group tutorial-iot-hub-rg --sku F1

# Make a note of the service connection string, you need it later:
az iot hub show-connection-string --name $hubname --policy-name service -o table

```

Ce didacticiel utilise un appareil simulé appelé **MyTwinDevice**. Le script suivant ajoute cet appareil dans le registre des identités et récupère sa chaîne de connexion :

```azurecli-interactive
# Set the name of your IoT hub:
hubname=tutorial-iot-hub

# Create the device in the identity registry:
az iot hub device-identity create --device-id MyTwinDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg

# Retrieve the device connection string, you need this later:
az iot hub device-identity show-connection-string --device-id MyTwinDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg -o table

```

## <a name="send-state-information"></a>Envoyer des informations d’état

Vous utilisez les propriétés souhaitées pour envoyer des informations d’état à partir d’une application back-end vers un appareil. Dans cette section, vous allez apprendre à :

* Recevoir et traiter les propriétés souhaitées sur un appareil.
* Envoyer les propriétés souhaitées à partir d’une application back-end.

Pour afficher l’exemple de code de l’appareil simulé qui reçoit les propriétés souhaitées, accédez au dossier **iot-hub/Tutorials/DeviceTwins** dans l’exemple de projet Node.js que vous avez téléchargé. Ouvrez ensuite le fichier SimulatedDevice.js dans un éditeur de texte.

Les sections suivantes décrivent le code qui s’exécute sur l’appareil simulé qui répond aux modifications de propriétés souhaitées envoyées à partir de l’application back end :

### <a name="retrieve-the-device-twin-object"></a>Récupérer l’objet de jumeau d’appareil

Le code suivant établit une connexion à votre hub IoT à l’aide d’une chaîne de connexion de l’appareil :

[!code-javascript[Create IoT Hub client](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=createhubclient&highlight=2 "Create IoT Hub client")]

Le code suivant permet d’obtenir un jumeau de l’objet client :

[!code-javascript[Get twin](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=gettwin&highlight=2 "Get twin")]

### <a name="sample-desired-properties"></a>Exemples de propriétés souhaitées

Vous pouvez structurer vos propriétés souhaitées pour répondre aux besoins de votre application. Cet exemple utilise une propriété de niveau supérieur appelée **fanOn** et regroupe les autres propriétés dans des **composants** distincts. L’extrait de code JSON suivant illustre la structure des propriétés souhaitées utilisée dans ce didacticiel :

[!code[Sample desired properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/desired.json "Sample desired properties")]

### <a name="create-handlers"></a>Créer des gestionnaires

Vous pouvez créer des gestionnaires pour les mises à jour de propriétés souhaitées qui répondent aux mises à jour effectuées à différents niveaux de la hiérarchie JSON. Par exemple, ce gestionnaire voit toutes les modifications de propriétés souhaitées envoyées à l’appareil à partir d’une application back-end. La variable **delta** contient les propriétés souhaitées envoyées depuis le serveur back-end de la solution :

[!code-javascript[Handle all properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=allproperties&highlight=2 "Handle all properties")]

Le gestionnaire suivant réagit uniquement aux modifications apportées à la propriété souhaitée **fanOn** :

[!code-javascript[Handle fan property](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=fanproperty&highlight=2 "Handle fan property")]

### <a name="handlers-for-multiple-properties"></a>Gestionnaires pour propriétés multiples

Dans l’exemple de propriétés JSON souhaitées mentionnées précédemment, le nœud **climat** sous **composants** contient deux propriétés, **minTemperature** et  **maxTemperature**.

Un objet **jumeau** local de l’appareil enregistre un ensemble complet de propriétés souhaitées et rapportées. La valeur **delta** envoyée par l’application back-end peut mettre à jour un seul sous-ensemble de propriétés souhaitées. Dans l’extrait de code suivant, si l’appareil simulé reçoit une mise à jour pour une seule des propriétés **minTemperature** et **maxTemperature**, il utilise la valeur contenue dans le jumeau local pour l’autre valeur afin de configurer l’appareil :

[!code-javascript[Handle climate component](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=climatecomponent&highlight=2 "Handle climate component")]

L’objet **jumeau** local de l’appareil enregistre un ensemble complet de propriétés souhaitées et rapportées. La valeur **delta** envoyée par l’application back-end peut mettre à jour un seul sous-ensemble de propriétés souhaitées.

### <a name="handle-insert-update-and-delete-operations"></a>Gérer les opérations d’ajout, de mise à jour et de suppression

Les propriétés souhaitées envoyées depuis l’application back-end n’indiquent pas l’opération qui est effectuée sur une propriété souhaitée spécifique. Votre code doit identifier l’opération à partir de l’ensemble actuel de propriétés souhaitées stockées localement et des modifications envoyées par le hub.

L’extrait de code suivant montre comment l’appareil simulé gère les opérations d’ajout, de mise à jour et de suppression sur la liste de **composants** dans les propriétés souhaitées. Il explique comment utiliser les valeurs **null** pour indiquer qu’un composant doit être supprimé :

[!code-javascript[Handle components](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=components&highlight=2,6,13 "Handle components")]

### <a name="send-desired-properties-to-a-device-from-the-back-end"></a>Envoyer des propriétés souhaitées à un appareil à partir de l’application back-end

Vous avez vu comment un appareil implémente des gestionnaires pour recevoir des mises à jour de propriétés souhaitées. Cette section vous montre comment envoyer des modifications de propriétés souhaitées à un appareil à partir d’une application back-end.

Pour afficher l’exemple de code de l’appareil simulé qui reçoit les propriétés souhaitées, accédez au dossier **iot-hub/Tutorials/DeviceTwins** dans l’exemple de projet Node.js que vous avez téléchargé. Ouvrez ensuite le fichier ServiceClient.js dans un éditeur de texte.

L’extrait de code suivant montre comment établir la connexion au registre des identités des appareils et accéder au jumeau d’un appareil spécifique :

[!code-javascript[Create registry and get twin](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=getregistrytwin&highlight=2,6 "Create registry and get twin")]

L’extrait de code suivant montre les différents *correctifs* de propriétés souhaitées que l’application back-end envoie à l’appareil :

[!code-javascript[Patches sent to device](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=patches&highlight=2,12,26,41,56 "Patches sent to device")]

L’extrait de code suivant montre comment l’application back-end envoie une mise à jour de propriété souhaitée à un appareil :

[!code-javascript[Send desired properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=senddesiredproperties&highlight=2 "Send desired properties")]

### <a name="run-the-applications"></a>Exécution des applications

Dans cette section, vous allez exécuter deux exemples d’applications pour observer comment une application back-end envoie des mises à jour de propriétés souhaitées à une application d’appareil simulé.

Pour exécuter l’appareil simulé et les applications back-end, vous avez besoin des chaînes de connexion du service et de l’appareil. Vous avez noté ces chaînes de connexion lorsque vous avez créé les ressources au début de ce didacticiel.

Pour exécuter l’application d’appareil simulé, ouvrez une fenêtre d’interpréteur de commandes ou d’invite de commandes et accédez au dossier **iot-hub/Tutorials/DeviceTwins** dans le projet Node.js que vous avez téléchargé. Exécutez ensuite les commande suivantes :

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

Pour exécuter l’application back-end, ouvrez une autre fenêtre d’interpréteur de commandes ou d’invite de commandes. Puis, accédez au dossier **iot-hub/Tutorials/DeviceTwins** dans le projet Node.js que vous avez téléchargé. Exécutez ensuite les commande suivantes :

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

La capture d’écran suivante montre le résultat de l’application d’appareil simulé et comment celle-ci gère une mise à jour de la propriété souhaitée **maxTemperature**. Vous pouvez voir de quelle manière le gestionnaire de niveau supérieur et les gestionnaires du composant climat s’exécutent :

![Capture d’écran qui montre comment s’exécutent le gestionnaire de niveau supérieur et les gestionnaires du composant climat.](./media/tutorial-device-twins/SimulatedDevice1.png)

La capture d’écran suivante montre le résultat de l’application back-end et comment celle-ci envoie une mise à jour de la propriété souhaitée **maxTemperature** :

![Capture d’écran qui montre la sortie de l’application back-end et comment celle-ci envoie une mise à jour.](./media/tutorial-device-twins/BackEnd1.png)

## <a name="receive-state-information"></a>Recevoir des informations d’état

Votre application back-end reçoit des informations d’état d’un appareil sous la forme de propriétés rapportées. L’appareil définit les propriétés rapportées et les envoie à votre hub. Une application back-end peut lire les valeurs actuelles des propriétés rapportées à partir du jumeau d’appareil stocké dans votre hub.

### <a name="send-reported-properties-from-a-device"></a>Envoyer des propriétés rapportées depuis un appareil

Vous pouvez envoyer des mises à jour de propriétés rapportées sous la forme d’un correctif. L’extrait de code suivant montre un modèle de correctif envoyé par l’appareil simulé. L’appareil simulé met à jour les champs dans le correctif avant de l’envoyer au hub :

[!code-javascript[Reported properties patches](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=reportedpatch&highlight=2 "Reported properties patches")]

L’appareil simulé utilise la fonction suivante pour envoyer le correctif contenant les propriétés rapportées au hub :

[!code-javascript[Send reported properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=sendreportedproperties&highlight=2 "Send reported properties")]

### <a name="process-reported-properties"></a>Traiter des propriétés rapportées

Une application back-end accède aux valeurs actuelles des propriétés rapportées d’un appareil via le jumeau d’appareil. L’extrait de code suivant montre comment l’application back-end lit les valeurs des propriétés rapportées pour l’appareil simulé :

[!code-javascript[Display reported properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=displayreportedproperties&highlight=2 "Display reported properties")]

### <a name="run-the-applications"></a>Exécution des applications

Dans cette section, vous allez exécuter deux exemples d’applications pour observer comment une application d’appareil simulé envoie des mises à jour de propriétés rapportées à une application back-end.

Vous allez exécuter les deux mêmes exemples d’applications que vous avez exécutés pour voir comment les propriétés souhaitées sont envoyées à un appareil.

Pour exécuter l’appareil simulé et les applications back-end, vous avez besoin des chaînes de connexion du service et de l’appareil. Vous avez noté ces chaînes de connexion lorsque vous avez créé les ressources au début de ce didacticiel.

Pour exécuter l’application d’appareil simulé, ouvrez une fenêtre d’interpréteur de commandes ou d’invite de commandes et accédez au dossier **iot-hub/Tutorials/DeviceTwins** dans le projet Node.js que vous avez téléchargé. Exécutez ensuite les commande suivantes :

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

Pour exécuter l’application back-end, ouvrez une autre fenêtre d’interpréteur de commandes ou d’invite de commandes. Puis, accédez au dossier **iot-hub/Tutorials/DeviceTwins** dans le projet Node.js que vous avez téléchargé. Exécutez ensuite les commande suivantes :

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

La capture d’écran suivante montre le résultat de l’application d’appareil simulé et comment celle-ci envoie une mise à jour de propriété rapportée à votre hub :

![Appareil simulé](./media/tutorial-device-twins/SimulatedDevice2.png)

La capture d’écran suivante montre le résultat de l’application back-end et comment celle-ci reçoit et traite une mise à jour de propriété rapportée envoyée à partir d’un appareil :

![Application back-end](./media/tutorial-device-twins/BackEnd2.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’effectuer le didacticiel suivant, conservez le groupe de ressources et l’IoT Hub afin de les réutiliser ultérieurement.

Si vous n’avez plus besoin du hub IoT, supprimez-le ainsi que le groupe de ressources dans le portail. Pour ce faire, sélectionnez le groupe de ressources **tutorial-iot-hub-rg** qui contient votre hub IoT, puis cliquez sur **Supprimer**.

Vous pouvez également utiliser l’interface de ligne de commande :

```azurecli-interactive
# Delete your resource group and its contents
az group delete --name tutorial-iot-hub-rg
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel vous avez appris à synchroniser les informations d’état entre vos appareils et votre hub IoT. Passez au didacticiel suivant pour découvrir comment utiliser les jumeaux d’appareil pour implémenter un processus de mise à jour du microprogramme.

> [!div class="nextstepaction"]
> [Implémenter un processus de mise à jour de microprogramme d’appareil](tutorial-firmware-update.md)
