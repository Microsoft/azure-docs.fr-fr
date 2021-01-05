---
title: Vérifier la connectivité des appareils dans Azure IoT Hub
description: Utilisez des outils de IoT Hub pour résoudre les problèmes de connectivité des appareils à votre Hub IoT pendant le développement.
services: iot-hub
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.custom:
- mvc
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- devx-track-js
- devx-track-azurecli
ms.date: 02/22/2019
ms.topic: tutorial
ms.service: iot-hub
ms.openlocfilehash: 1eead9bb93fe8b753ace518cde18b240ab1a3cd4
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96572675"
---
# <a name="tutorial-use-a-simulated-device-to-test-connectivity-with-your-iot-hub"></a>Tutoriel : Utiliser un appareil simulé pour tester la connectivité avec votre hub IoT

Dans ce tutoriel, vous utilisez les outils du portail Azure IoT Hub et les commandes Azure CLI pour tester la connectivité de l’appareil. Ce tutoriel utilise également un simulateur d’appareil simple que vous exécutez sur votre ordinateur de bureau.

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Vérifier l’authentification de vos appareils
> * Vérifier la connectivité appareil-à-cloud
> * Vérifiez la connectivité cloud-à-appareil
> * Vérifier la synchronisation de jumeau d’appareil

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

L’application de simulateur d’appareil que vous exécutez dans ce tutoriel est écrit à l’aide de Node.js. Votre ordinateur de développement doit disposer de Node.js v10.x.x ou ultérieur.

Vous pouvez télécharger Node.js pour plusieurs plateformes sur [nodejs.org](https://nodejs.org).

Vous pouvez vérifier la version actuelle de Node.js sur votre machine de développement à l’aide de la commande suivante :

```cmd/sh
node --version
```

Téléchargez l’exemple de projet Node.js de simulateur d’appareil à partir de https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip et extrayez l’archive ZIP.

Assurez-vous que le port 8883 est ouvert dans votre pare-feu. L'exemple d’appareil de ce tutoriel utilise le protocole MQTT qui communique sur le port 8883. Dans certains environnements réseau professionnels et scolaires, ce port peut être bloqué. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Créer un hub IoT

Si vous avez créé un Hub IoT de niveau Gratuit ou Standard lors d’un démarrage rapide précédente, vous pouvez ignorer cette étape.

[!INCLUDE [iot-hub-tutorials-create-free-hub](../../includes/iot-hub-tutorials-create-free-hub.md)]

## <a name="check-device-authentication"></a>Vérifier l’authentification des appareils

Un appareil doit s’authentifier auprès de votre Hub avant de pouvoir échanger des données avec le Hub. Vous pouvez utiliser l’outil **Appareils IoT** dans la section **Gestion des appareils** du portail pour gérer vos appareils et vérifier les clés d’authentification qu’ils utilisent. Dans cette section du tutoriel, vous ajoutez un nouvel appareil de test, récupérez sa clé et vérifiez que l’appareil de test peut se connecter au Hub. Ensuite, vous réinitialisez la clé d’authentification pour observer ce qui ce produit lorsqu’un appareil tente d’utiliser une clé obsolète. Cette section du tutoriel utilise le portail Azure pour créer, gérer et surveiller un périphérique, ainsi que l’exemple Node.js de simulateur d’appareil.

Connectez-vous au portail et accédez à votre IoT Hub. Puis accédez à l’outil **Appareils IoT** :

![Outil Appareils IoT](media/tutorial-connectivity/iot-devices-tool.png)

Pour inscrire un nouvel appareil, cliquez sur **+ Ajouter**, définissez **l’ID d’appareil** sur **MyTestDevice**, puis cliquez sur **Enregistrer** :

![Ajouter un nouvel appareil](media/tutorial-connectivity/add-device.png)

Pour récupérer la chaîne de connexion de **MyTestDevice**, cliquez dessus dans la liste d’appareils puis copiez la valeur **Chaîne de connexion-clé primaire**. La chaîne de connexion inclut la *clé d’accès partagé* pour l’appareil.

![Récupérer la chaîne de connexion de l’appareil](media/tutorial-connectivity/copy-connection-string.png)

Pour simuler l’envoi de données de télémétrie de **MyTestDevice** à votre Hub IoT, exécutez l’application d’appareil simulé Node.js que vous avez téléchargée précédemment.

Dans une fenêtre de terminal sur votre machine de développement, accédez au dossier racine de l’exemple de projet Node.js que vous avez téléchargé. Accédez ensuite au dossier **iot-hub\Tutorials\ConnectivityTests**.

Dans la fenêtre de terminal, exécutez les commandes suivantes pour installer les bibliothèques requises et exécuter l’application d’appareil simulé. Utilisez la chaîne de connexion de l’appareil que vous avez notée lorsque vous avez ajouté l’appareil dans le portail.

```cmd/sh
npm install
node SimulatedDevice-1.js "{your device connection string}"
```

La fenêtre de terminal affiche des informations lorsqu’il tente de se connecter à votre Hub :

![Connecter un appareil simulé](media/tutorial-connectivity/sim-1-connected.png)

Vous avez maintenant correctement authentifié depuis un appareil à l’aide d’une clé d’appareil générée par votre Hub IoT.

### <a name="reset-keys"></a>Réinitialiser les clés

Dans cette section, vous réinitialisez la clé d’appareil et observez l’erreur lorsque l’appareil simulé tente de se connecter.

Pour réinitialiser la clé principale d’appareil de **MyTestDevice**, exécutez les commandes suivantes :

```azurecli-interactive
# Generate a new Base64 encoded key using the current date
read key < <(date +%s | sha256sum | base64 | head -c 32)

# Requires the IoT Extension for Azure CLI
# az extension add --name azure-iot

# Reset the primary device key for MyTestDevice
az iot hub device-identity update --device-id MyTestDevice --set authentication.symmetricKey.primaryKey=$key --hub-name {YourIoTHubName}
```

Dans la fenêtre de terminal sur votre machine de développement, exécutez à nouveau l’application d’appareil simulé :

```cmd/sh
npm install
node SimulatedDevice-1.js "{your device connection string}"
```

Cette fois, vous voyez une erreur d’authentification lorsque l’application tente de se connecter :

![Erreur de connexion](media/tutorial-connectivity/sim-1-fail.png)

### <a name="generate-shared-access-signature-sas-token"></a>Générer un jeton de signature d’accès partagé (SAP)

Si votre appareil utilise l’un des Kits de développement logiciel (SDK) de l’appareil IoT Hub, le code de bibliothèque du Kit de développement logiciel (SDK) génère le jeton SAP utilisé pour l’authentification auprès du Hub. Un jeton SAP est généré à partir du nom de votre Hub, du nom de votre appareil et de la clé de l’appareil.

Dans certains scénarios, comme dans une passerelle de protocole cloud ou dans le cadre d’un schéma d’authentification personnalisé, vous devrez peut-être générer le jeton SAP vous-même. Pour résoudre les problèmes liés à votre code de génération SAP, il est utile de générer un jeton SAP vérifié à utiliser pendant le test.

> [!NOTE]
> L’exemple SimulatedDevice-2.js inclut des exemples de génération d’un jeton SAP avec et sans le kit de développement logiciel.

Pour générer un jeton SAP vérifié à l’aide de CLI, exécutez la commande suivante :

```azurecli-interactive
az iot hub generate-sas-token --device-id MyTestDevice --hub-name {YourIoTHubName}
```

Prenez note du texte entier du jeton SAP généré. La sortie a l’aspect suivant : `SharedAccessSignature sr=tutorials-iot-hub.azure-devices.net%2Fdevices%2FMyTestDevice&sig=....&se=1524155307`

Dans une fenêtre de terminal sur votre machine de développement, accédez au dossier racine de l’exemple de projet Node.js que vous avez téléchargé. Accédez ensuite au dossier **iot-hub\Tutorials\ConnectivityTests**.

Dans la fenêtre de terminal, exécutez les commandes suivantes pour installer les bibliothèques requises et exécuter l’application d’appareil simulé :

```cmd/sh
npm install
node SimulatedDevice-2.js "{Your SAS token}"
```

La fenêtre de terminal affiche des informations lorsqu’il tente de se connecter à votre Hub à l’aide du jeton SAP :

![Connecter un appareil simulé à l’aide d’un jeton](media/tutorial-connectivity/sim-2-connected.png)

Vous avez maintenant correctement authentifié depuis un appareil à l’aide d’un jeton de test SAP généré par une commande CLI. Le fichier **SimulatedDevice-2.js** inclut l’exemple de code qui montre comment générer un jeton SAP dans le code.

### <a name="protocols"></a>Protocoles

Un appareil peut utiliser l’un des protocoles suivants pour se connecter à votre Hub IoT :

| Protocol | Port sortant |
| --- | --- |
| MQTT |8883 |
| MQTT sur WebSockets |443 |
| AMQP |5671 |
| AMQP sur WebSockets |443 |
| HTTPS |443 |

Si le port de sortie est bloqué par un pare-feu, l’appareil ne peut pas se connecter :

![Port bloqué](media/tutorial-connectivity/port-blocked.png)

## <a name="check-device-to-cloud-connectivity"></a>Vérifier la connectivité appareil-à-cloud

Après la connexion d’un appareil, ce dernier essaie généralement d’envoyer des données de télémétrie à votre Hub IoT. Cette section vous montre comment vous pouvez vérifier que les donnés de télémétrie envoyées par l’appareil atteignent votre Hub.

Récupérez tout d’abord la chaîne de connexion en cours pour votre appareil simulé à l’aide de la commande suivante :

```azurecli-interactive
az iot hub device-identity show-connection-string --device-id MyTestDevice --output table --hub-name {YourIoTHubName}
```

Pour exécuter un appareil simulé qui envoie des messages, accédez au dossier **iot-hub\Tutorials\ConnectivityTests** dans le code que vous avez téléchargé.

Dans la fenêtre de terminal, exécutez les commandes suivantes pour installer les bibliothèques requises et exécuter l’application d’appareil simulé :

```cmd/sh
npm install
node SimulatedDevice-3.js "{your device connection string}"
```

La fenêtre de terminal affiche des informations lorsqu’il envoie des données de télémétrie à votre Hub :

![Envoi de messages de l’appareil simulé](media/tutorial-connectivity/sim-3-sending.png)

Vous pouvez utiliser les **Métriques** dans le portail pour vérifier que les messages de télémétrie arrivent jusqu’à votre hub IoT. Sélectionnez votre Hub IoT dans la liste déroulante **Ressource**, sélectionnez **Messages de télémétrie envoyés** en tant que métrique et définissez l’intervalle de temps sur **Dernière heure**. Le graphique indique le nombre agrégé de messages envoyés par l’appareil simulé :

![Afficher les métriques IoT Hub](media/tutorial-connectivity/metrics-portal.png)

Après le démarrage de l’appareil simulé, les mesures deviennent disponibles après quelques minutes.

## <a name="check-cloud-to-device-connectivity"></a>Vérifiez la connectivité cloud-à-appareil

Cette section montre comment vous pouvez effectuer un appel de méthode directe test sur un appareil pour vérifier la connectivité cloud-à-appareil. Vous exécutez un appareil simulé sur votre machine de développement pour écouter des appels de méthode directe à partir de votre Hub.

Dans une fenêtre de terminal, exécutez les commandes suivantes pour démarrer l’application d’appareil simulé :

```cmd/sh
node SimulatedDevice-3.js "{your device connection string}"
```

Utiliser une commande CLI pour appeler une méthode directe sur l’appareil :

```azurecli-interactive
az iot hub invoke-device-method --device-id MyTestDevice --method-name TestMethod --timeout 10 --method-payload '{"key":"value"}' --hub-name {YourIoTHubName}
```

L’appareil simulé affiche un message dans la console lorsqu’il reçoit un appel de méthode directe :

![L’appareil simulé reçoit un appel de méthode directe](media/tutorial-connectivity/receive-method-call.png)

Lorsque l’appareil simulé a reçu correctement l’appel de méthode directe, il envoie un accusé de réception au Hub :

![Recevoir l’accusé de réception de méthode directe](media/tutorial-connectivity/method-acknowledgement.png)

## <a name="check-twin-synchronization"></a>Vérifier la synchronisation des jumeaux

Les appareils utilisent des jumeaux pour synchroniser l’état entre le périphérique et le Hub. Dans cette section, vous utilisez des commandes CLI pour envoyer _les propriétés souhaitées_ à un appareil et lire les _propriétés signalées_ envoyées par l’appareil.

L’appareil simulé que vous utilisez dans cette section envoie des propriétés signalées au Hub chaque fois qu’il démarre, et imprime les propriétés souhaitées vers la console chaque fois qu’il les reçoit.

Dans une fenêtre de terminal, exécutez les commandes suivantes pour démarrer l’application d’appareil simulé :

```cmd/sh
node SimulatedDevice-3.js "{your device connection string}"
```

Pour vérifier que le Hub a reçu les propriétés signalées de l’appareil, utilisez la commande CLI suivante :

```azurecli-interactive
az iot hub device-twin show --device-id MyTestDevice --hub-name {YourIoTHubName}
```

Dans la sortie de la commande, vous pouvez voir les propriétés **devicelaststarted** dans la section propriétés déclarées. Cette propriété indique la date et l’heure du dernier démarrage de l’appareil simulé.

![Afficher les propriétés signalées](media/tutorial-connectivity/reported-properties.png)

Pour vérifier que le Hub peut envoyer les valeurs de propriétés souhaitées de l’appareil, utilisez la commande CLI suivante :

```azurecli-interactive
az iot hub device-twin update --set properties.desired='{"mydesiredproperty":"propertyvalue"}' --device-id MyTestDevice --hub-name {YourIoTHubName}
```

L’appareil simulé imprime un message lorsqu’il reçoit une mise à jour de la propriété souhaitée en provenance du Hub :

![Recevoir les propriétés souhaitées](media/tutorial-connectivity/desired-properties.png)

En plus de recevoir les modifications apportées aux propriétés souhaitées lorsqu’elles sont effectuées, l’appareil simulé vérifie automatiquement les propriétés souhaitées lors de son démarrage.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin du hub IoT, supprimez-le ainsi que le groupe de ressources dans le portail. Pour ce faire, sélectionnez le groupe de ressources **tutorials-iot-hub-rg** qui contient votre Hub IoT, puis cliquez sur **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, nous avons vu comment vérifier vos clés d’appareils, vérifier la connectivité appareil-à-cloud, vérifier la connectivité cloud-à-appareil et vérifier la synchronisation de jumeau d’appareil. Pour en savoir plus sur la façon de contrôler votre Hub IoT, consultez l’article sur les procédures de contrôle IoT Hub.

> [!div class="nextstepaction"]
> [Superviser avec IoT Hub](monitor-iot-hub.md)
