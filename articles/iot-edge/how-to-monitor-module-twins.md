---
title: Superviser les jumeaux de module - Azure IoT Edge
description: Comment interpréter les jumeaux d'appareil et jumeaux de module pour déterminer la connectivité et l’intégrité.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 05/29/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 1a11d3a9a972188af4cf8f054349da98d69691a3
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91876156"
---
# <a name="monitor-module-twins"></a>Superviser les jumeaux de module

Dans Azure IoT Hub, les jumeaux de module permettent de surveiller la connectivité et l’intégrité de vos déploiements IoT Edge. Les jumeaux de module stockent dans votre hub IoT des informations utiles sur les performances de vos modules en cours d’exécution. Les modules runtime de l’[agent IoT Edge](iot-edge-runtime.md#iot-edge-agent) et du [hub IoT Edge](iot-edge-runtime.md#iot-edge-hub) gèrent leurs jumeaux de module, `$edgeAgent` et `$edgeHub`, respectivement :

* `$edgeAgent` contient les données d’intégrité et de connectivité relatives aux modules runtime de l’agent IoT Edge et du hub IoT Edge Hub, ainsi qu’à vos modules personnalisés. L’agent IoT Edge est chargé de déployer les modules, de les surveiller et de générer des rapports sur l’état de la connexion à votre Azure IoT Hub.
* `$edgeHub` contient des données sur les communications entre le hub IoT Edge s’exécutant sur un appareil et votre Azure IoT Hub. Cela comprend le traitement des messages provenant des appareils en aval. Le hub IoT Edge est chargé de traiter les communications entre Azure IoT Hub et les appareils et modules IoT Edge.

Les données sont organisées dans des métadonnées, des balises, ainsi que des jeux de propriétés souhaitées et signalées dans les structures JSON des jumeaux de module. Les propriétés souhaitées que vous avez spécifiées dans votre fichier deployment.json sont copiées dans les jumeaux de module. L’agent IoT Edge et le hub IoT Edge mettent à jour les propriétés signalées pour leurs modules.

De même, les propriétés souhaitées spécifiées pour vos modules personnalisés dans le fichier deployment.json sont copiées dans leur jumeau de module, mais votre solution se doit de fournir les valeurs des propriétés signalées.

Cet article explique comment examiner les jumeaux de module dans le portail Azure, Azure CLI, ainsi que dans Visual Studio Code. Pour plus d’informations sur la supervision de la façon dont vos appareils reçoivent les déploiements, consultez [Superviser les déploiements IoT Edge](how-to-monitor-iot-edge-deployments.md). Pour obtenir une vue d’ensemble du concept de jumeaux de module, consultez [Comprendre et utiliser les jumeaux de module dans IoT Hub](../iot-hub/iot-hub-devguide-module-twins.md).

> [!TIP]
> Les propriétés signalées d’un module runtime peuvent s’avérer obsolètes si un appareil IoT Edge est déconnecté de son hub IoT. Vous pouvez [effectuer un test ping](how-to-edgeagent-direct-method.md#ping) du module `$edgeAgent` afin de déterminer si la connexion a été perdue.

## <a name="monitor-runtime-module-twins"></a>Surveiller les jumeaux de module du runtime

Pour résoudre les problèmes de connectivité de déploiement, examinez les jumeaux de module du runtime de l’agent IoT Edge et du hub IoT Edge, puis explorez d’autres modules.

### <a name="monitor-iot-edge-agent-module-twin"></a>Surveiller le jumeau de module de l’agent IoT Edge

Le JSON suivant illustre le jumeau de module `$edgeAgent` dans Visual Studio Code avec la plupart des sections JSON réduites.

```json
{
  "deviceId": "Windows109",
  "moduleId": "$edgeAgent",
  "etag": "AAAAAAAAAAU=",
  "deviceEtag": "NzgwNjA1MDUz",
  "status": "enabled",
  "statusUpdateTime": "0001-01-01T00:00:00Z",
  "connectionState": "Disconnected",
  "lastActivityTime": "0001-01-01T00:00:00Z",
  "cloudToDeviceMessageCount": 0,
  "authenticationType": "sas",
  "x509Thumbprint": {
    "primaryThumbprint": null,
    "secondaryThumbprint": null
  },
  "version": 53,
  "properties": {
    "desired": { "···" },
    "reported": {
      "schemaVersion": "1.0",
      "version": { "···" },
      "lastDesiredStatus": { "···" },
      "runtime": { "···" },
      "systemModules": {
        "edgeAgent": { "···" },
        "edgeHub": { "···" }
      },
      "lastDesiredVersion": 5,
      "modules": {
        "SimulatedTemperatureSensor": { "···" }
      },
      "$metadata": { "···" },
      "$version": 48
    }
  }
}
```

Le JSON peut être décrit dans les sections suivantes, en commençant par le haut :

* Metadata : contient les données de connectivité. Il est intéressant de constater que l’état de connexion de l’agent IoT Edge est toujours dans un état déconnecté : `"connectionState": "Disconnected"`. En effet, l’état de la connexion concerne les messages appareil-à-cloud (D2C) et l’agent IoT Edge n’envoient pas de messages D2C.
* Properties : contient les sous-sections `desired` et `reported`.
* Properties.desired : (présentation réduite) valeurs des propriétés attendues définies par l’opérateur dans le fichier deployment.json.
* Properties.reported : valeurs des propriétés les plus récentes signalées par l’agent IoT Edge.

Les sections `properties.desired` et `properties.reported` affichent une structure similaire et contiennent des métadonnées supplémentaires pour les informations relatives au schéma, à la version et au runtime. La section `modules` est également incluse pour tous les modules personnalisés (tels que `SimulatedTemperatureSensor`) et la section `systemModules` pour `$edgeAgent` et les modules runtime `$edgeHub`.

En comparant les valeurs des propriétés signalées aux valeurs souhaitées, vous pouvez identifier les anomalies et les déconnections afin de faciliter la résolution des problèmes. Lors de ces comparaisons, vérifiez la valeur signalée `$lastUpdated` de la section `metadata` pour la propriété que vous examinez.

À des fins de résolution des problèmes, il est important d’examiner les propriétés suivantes :

* **exitcode** : toute valeur autre que zéro indique que le module s’est arrêté en raison d’une défaillance. Cela étant, les codes d’erreur 137 ou 143 sont utilisés si un module a été volontairement défini sur un état arrêté.

* **lastStartTimeUtc** : affiche la valeur **DateTime** à laquelle le conteneur a été démarré pour la dernière fois. Cette valeur est 0001-01-01T00:00:00Z si le conteneur n’a pas été démarré.

* **lastExitTimeUtc** : affiche la valeur **DateTime** à laquelle le conteneur s’est terminé pour la dernière fois. Cette valeur est 0001-01-01T00:00:00Z si le conteneur est en cours d’exécution et n’a jamais été arrêté.

* **runtimeStatus** : les valeurs possibles sont les suivantes :

    | Valeur | Description |
    | --- | --- |
    | unknown | État par défaut jusqu’à la création du déploiement. |
    | backoff | Le module est planifié pour démarrer, mais il n’est pas en cours d’exécution. Cette valeur est utile pour un module rencontrant des changements d’état au redémarrage. Lorsqu’un module en échec attend de redémarrer pendant la période de refroidissement, il est dans un état backoff. |
    | exécution en cours | Indique que le module est en cours d’exécution. |
    | unhealthy | Indique qu’une vérification de la sonde d’intégrité a échoué ou expiré. |
    | stopped | Indique que le module s’est terminé avec succès (code de sortie égal à zéro). |
    | échec | Indique que le module s’est terminé avec un code de sortie d’échec (autre que zéro). Le module peut revenir à l’état backoff en fonction de la stratégie de redémarrage en vigueur. Cet état peut indiquer que le module s’est heurté à une erreur irrécupérable. L’échec se produit lorsque Microsoft Monitoring Agent (MMA) ne peut plus activer le module, ce qui requiert un nouveau déploiement. |

Pour plus d’informations, consultez [Propriétés signalées pour EdgeAgent](module-edgeagent-edgehub.md#edgeagent-reported-properties).

### <a name="monitor-iot-edge-hub-module-twin"></a>Surveiller le jumeau de module du hub IoT Edge

Le JSON suivant illustre le jumeau de module `$edgeHub` dans Visual Studio Code avec la plupart des sections JSON réduites.

```json
{
  "deviceId": "Windows109",
  "moduleId": "$edgeHub",
  "etag": "AAAAAAAAAAU=",
  "deviceEtag": "NzgwNjA1MDU2",
  "status": "enabled",
  "statusUpdateTime": "0001-01-01T00:00:00Z",
  "connectionState": "Connected",
  "lastActivityTime": "0001-01-01T00:00:00Z",
  "cloudToDeviceMessageCount": 0,
  "authenticationType": "sas",
  "x509Thumbprint": {
    "primaryThumbprint": null,
    "secondaryThumbprint": null
  },
  "version": 102,
    "properties": {
      "desired": { "···" },
      "reported": {
        "schemaVersion": "1.0",
        "version": { "···" },
      "lastDesiredVersion": 5,
      "lastDesiredStatus": { "···" },
      "clients": {
        "Windows109/SimulatedTemperatureSensor": {
          "status": "Disconnected",
          "lastConnectedTimeUtc": "2020-04-08T21:42:42.1743956Z",
          "lastDisconnectedTimeUtc": "2020-04-09T07:02:42.1398325Z"
        }
      },
      "$metadata": { "···" },
      "$version": 97
    }
  }
}

```

Le JSON peut être décrit dans les sections suivantes, en commençant par le haut :

* Metadata : contient les données de connectivité.

* Properties : contient les sous-sections `desired` et `reported`.
* Properties.desired : (présentation réduite) valeurs des propriétés attendues définies par l’opérateur dans le fichier deployment.json.
* Properties.reported : valeurs des propriétés les plus récentes signalées par le hub IoT Edge.

Si vous rencontrez des problèmes liés à vos appareils en aval, l’examen de ces données constitue un bon point de départ.

## <a name="monitor-custom-module-twins"></a>Surveiller les jumeaux de module personnalisés

Les informations relatives à la connectivité de vos modules personnalisés sont gérées dans le jumeau de module de l’agent IoT Edge. Le jumeau de module de votre module personnalisé est principalement utilisé pour gérer les données de votre solution. Les propriétés souhaitées que vous avez définies dans votre fichier deployment.json sont répercutées dans le jumeau de module, et votre module peut mettre à jour les valeurs des propriétés signalées, si besoin.

Vous pouvez utiliser votre langage de programmation préféré avec les [kits de développement logiciel (SDK) Azure IoT Hub Device](../iot-hub/iot-hub-devguide-sdks.md#azure-iot-hub-device-sdks) pour mettre à jour les valeurs des propriétés signalées dans le jumeau de module, selon le code d’application de votre module. Pour ce faire, la procédure suivante utilise le kit de développement logiciel (SDK) Azure pour .NET, à l’aide du code issu du module [SimulatedTemperatureSensor](https://github.com/Azure/iotedge/blob/dd5be125df165783e4e1800f393be18e6a8275a3/edge-modules/SimulatedTemperatureSensor/src/Program.cs) :

1. Créer une instance de [ModuleClient](/dotnet/api/microsoft.azure.devices.client.moduleclient) avec la méthode [CreateFromEnvironmentAysnc](/dotnet/api/microsoft.azure.devices.client.moduleclient.createfromenvironmentasync).

1. Obtenir une collection des propriétés du jumeau de module avec la méthode [GetTwinAsync](/dotnet/api/microsoft.azure.devices.client.moduleclient.gettwinasync).

1. Créer un écouteur (en passant un rappel) pour intercepter les modifications apportées aux propriétés souhaitées à l’aide de la méthode [SetDesiredPropertyUpdateCallbackAsync](/dotnet/api/microsoft.azure.devices.client.deviceclient.setdesiredpropertyupdatecallbackasync).

1. Dans votre méthode de rappel, mettez à jour les propriétés signalées dans le jumeau de module à l’aide de la méthode [UpdateReportedPropertiesAsync](/dotnet/api/microsoft.azure.devices.client.moduleclient), en passant un objet[TwinCollection](/dotnet/api/microsoft.azure.devices.shared.twincollection) des valeurs des propriétés que vous souhaitez définir.

## <a name="access-the-module-twins"></a>Accéder aux jumeaux de module

Vous pouvez consulter le JSON des jumeaux de module dans Azure IoT Hub, Visual Studio Code, ainsi qu’avec Azure CLI.

### <a name="monitor-in-azure-iot-hub"></a>Surveiller dans Azure IoT Hub

Pour afficher le JSON du jumeau de module :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre IoT Hub.
1. Sélectionnez **IoT Edge** dans le menu du volet gauche.
1. Sous l’onglet **Appareils IoT Edge**, sélectionnez l’**ID d’appareil** de l’appareil doté des modules que vous souhaitez surveiller.
1. Sous l’onglet **Modules**, sélectionnez le nom du module, puis **Jumeau d'identité de module** dans la barre de menus supérieure.

  ![Sélectionner un jumeau de module à afficher dans le portail Azure](./media/how-to-monitor-module-twins/select-module-twin.png)

Le message « Il n’existe pas d’identité de module pour ce module » indique que la solution backend à l’origine de la création de l’identité n’est plus disponible.

### <a name="monitor-module-twins-in-visual-studio-code"></a>Surveillez les jumeaux de module dans Visual Studio Code

Pour examiner et modifier un jumeau de module :

1. Si elle ne l’est pas déjà, installez l’[extension Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) pour Visual Studio Code.
1. Dans l’**Explorateur**, développez **Azure IoT Hub**, puis l’appareil doté du module que vous souhaitez surveiller.
1. Cliquez avec le bouton droit sur le module et sélectionnez **Modifier le jumeau de module**. Un fichier temporaire du jumeau de module est téléchargé sur votre ordinateur et affiché dans Visual Studio Code.

  ![Obtenir un jumeau de module à modifier dans Visual Studio Code](./media/how-to-monitor-module-twins/edit-module-twin-vscode.png)

Si vous apportez des modifications, sélectionnez **Mettre à jour le jumeau de module** au-dessus du code dans l’éditeur pour enregistrer les modifications apportées à votre hub IoT.

  ![Mettre à jour un jumeau de module dans Visual Studio Code](./media/how-to-monitor-module-twins/update-module-twin-vscode.png)

### <a name="monitor-module-twins-in-azure-cli"></a>Surveiller les jumeaux de module dans Azure CLI

Pour savoir si IoT Edge est en cours d’exécution, utilisez la méthode [az iot hub invoke-module-method](how-to-edgeagent-direct-method.md#ping) pour effectuer un test ping sur l’agent IoT Edge.

La structure [az iot hub module-twin](/cli/azure/ext/azure-cli-iot-ext/iot/hub/module-twin) fournit les commandes suivantes :

* **az iot hub module-twin show** : afficher une définition de jumeau de module.
* **az iot hub module-twin update** : mettre à jour une définition de jumeau de module.
* **az iot hub module-twin replace** : remplacer une définition de jumeau de module par un JSON cible.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [communiquer avec edgeAgent à l’aide de méthodes directes intégrées](how-to-edgeagent-direct-method.md).
