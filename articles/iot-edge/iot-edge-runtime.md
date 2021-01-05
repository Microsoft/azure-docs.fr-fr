---
title: Découvrir comment le runtime gère les appareils - Azure IoT Edge | Microsoft Docs
description: Découvrez comment runtime IoT Edge gère les modules, la sécurité, la communication et les rapports sur vos appareils
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/10/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: amqp, mqtt, devx-track-csharp
ms.openlocfilehash: c0c3a452c93b88483ac7027405665c26ceab8183
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368502"
---
# <a name="understand-the-azure-iot-edge-runtime-and-its-architecture"></a>Présentation du runtime Azure IoT Edge et de son architecture

Le runtime IoT Edge est une collection de programmes qui transforme un appareil en appareil IoT Edge. Collectivement, les composants du runtime IoT Edge permettent aux appareils IoT Edge de recevoir du code à exécuter dans l’arête et de communiquer les résultats.

Le runtime IoT Edge est responsable des fonctions suivantes sur les appareils IoT Edge :

* Installer et mettre à jour des charges de travail sur l’appareil.

* Tenir à jour les normes de sécurité Azure IoT Edge sur l’appareil.

* Garantir que les [modules IoT Edge](iot-edge-modules.md) sont toujours en cours d’exécution.

* Envoyer des rapports d’intégrité du module dans le cloud pour la supervision à distance.

* Gérer la communication entre les appareils en aval et les appareils IoT Edge.

* Gérer la communication entre les modules sur un appareil IoT Edge.

* Gérer la communication entre un appareil IoT Edge et le cloud.
<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"
* Gérer la communication entre des appareils IoT Edge.
::: moniker-end

![Le runtime communique des insights et des données sur l’intégrité des modules à IoT Hub](./media/iot-edge-runtime/Pipeline.png)

Les responsabilités du runtime IoT Edge se répartissent en deux catégories : communication et gestion des modules. Ces deux rôles sont remplis par deux composants qui font partie du runtime IoT Edge. L’*agent IoT Edge* déploie et surveille les modules, tandis que le *hub IoT Edge* est responsable de la communication.

L’agent IoT Edge et le hub IoT Edge sont tous deux des modules, comme n’importe quel autre module exécuté sur un appareil IoT Edge. Ils sont parfois appelés *modules runtime*.

## <a name="iot-edge-agent"></a>Agent IoT Edge

L’agent IoT Edge est l’un des deux modules qui composent le runtime Azure IoT Edge. Il est responsable de l’instanciation des modules, vérifie qu’ils continuent à s’exécuter, et signale l’état des modules à IoT Hub. Ces données de configuration sont écrites en tant que propriété du jumeau de module de l’agent IoT Edge.

Le [démon de sécurité IoT Edge](iot-edge-security-manager.md) démarre l’agent IoT Edge au démarrage de l’appareil. L’agent récupère son jumeau de module à partir d’IoT Hub et inspecte le manifeste de déploiement. Le manifeste de déploiement est un fichier JSON qui déclare les modules qui doivent être démarrés.

Chaque élément du manifeste de déploiement contient des informations sur un module, et est utilisé par l’agent IoT Edge pour contrôler le cycle de vie du module. Pour plus d’informations sur toutes les propriétés utilisées par l’agent IoT Edge pour contrôler les modules, consultez les [propriétés des jumeaux de module de l’agent IoT Edge et du hub IoT Edge](module-edgeagent-edgehub.md).

L’agent IoT Edge envoie la réponse d’exécution à IoT Hub. Voici une liste de réponses possibles :
  
* 200 - OK
* 400 - La configuration de déploiement a un format incorrect ou n’est pas valide.
* 417 - Aucune configuration de déploiement n’est définie pour l’appareil.
* 412 - La version de schéma dans la configuration de déploiement n’est pas valide.
* 406 - L’appareil IoT Edge est hors connexion ou n’envoie pas de rapports d’état.
* 500 - Une erreur s’est produite dans le runtime IoT Edge.

Pour plus d’informations sur la création de manifestes de déploiement, consultez [Découvrir comment déployer des modules et établir des routes dans IoT Edge](module-composition.md).

### <a name="security"></a>Sécurité

L’agent IoT Edge joue un rôle essentiel dans la sécurité d’un appareil IoT Edge. Par exemple, il effectue des actions telles que la vérification de l’image d’un module avant de le démarrer.

Pour plus d’informations sur le framework de sécurité Azure IoT Edge, consultez [Gestionnaire de sécurité IoT Edge](iot-edge-security-manager.md).

## <a name="iot-edge-hub"></a>Hub IoT Edge

Le hub IoT Edge est l’autre module qui compose le runtime Azure IoT Edge. Il joue le rôle de proxy local pour IoT Hub en exposant les mêmes points de terminaison de protocole qu’IoT Hub. Cette cohérence signifie que les clients peuvent se connecter au runtime IoT Edge comme ils le feraient avec IoT Hub.

Le hub IoT Edge n’est pas une version complète d’IoT Hub s’exécutant localement. Le hub IoT Edge délègue silencieusement certaines tâches à l’IoT Hub. Par exemple, le hub IoT Edge télécharge automatiquement les informations d’autorisation à partir d’IoT Hub lors de sa première connexion pour permettre à un appareil de se connecter. Une fois la première connexion établie, les informations d’autorisation sont mises en cache localement par le hub IoT Edge. Les connexions ultérieures à partir de cet appareil sont autorisées sans avoir à télécharger à nouveau les informations d’autorisation à partir du cloud.

### <a name="cloud-communication"></a>Communication cloud

Pour réduire la bande passante qu’utilise votre solution IoT Edge, le hub IoT Edge optimise le nombre de connexions au cloud. Le hub IoT Edge accepte des connexions logiques de modules ou d’appareils en aval, et les combine pour établir une connexion physique unique au cloud. Les détails de ce processus sont transparents pour le reste de la solution. Les clients pensent avoir leur propre connexion au cloud, alors qu’ils passent tous par la même connexion. Le hub IoT Edge peut utiliser le protocole AMQP ou MQTT pour communiquer en amont avec le cloud, indépendamment des protocoles utilisés par les appareils en aval. Toutefois, le hub IoT Edge ne permet actuellement que de combiner des connexions logiques en une seule connexion physique en utilisant AMQP comme protocole en amont et ses capacités de multiplexage. AMQP est le protocole en amont par défaut.

![Le hub IoT Edge est une passerelle entre les appareils physiques et IoT Hub](./media/iot-edge-runtime/gateway-communication.png)

Le hub IoT Edge peut déterminer s’il est connecté à IoT Hub. Si la connexion est perdue, le hub IoT Edge enregistre localement les messages ou les mises à jour de jumeau. Une fois la connexion rétablie, il synchronise toutes les données. L’emplacement utilisé pour ce cache temporaire est déterminé par une propriété du jumeau de module du hub IoT Edge. La taille du cache n’est pas limitée et augmente tant que l’appareil a une capacité de stockage.  Pour plus d’informations, voir [Fonctionnalités hors connexion](offline-capabilities.md).

<!-- <1.1> -->
::: moniker range="iotedge-2018-06"

### <a name="module-communication"></a>Communication des modules

Le hub IoT Edge facilite la communication entre les modules. L’utilisation du hub IoT Edge comme un répartiteur de messages permet aux modules de rester indépendants les uns des autres. Il suffit aux modules de spécifier les entrées sur lesquelles ils acceptent des messages et les sorties vers lesquelles ils écrivent des messages. Un développeur de solutions peut assembler ces entrées et sorties afin que les modules traitent les données dans l’ordre propre à cette solution.

![Le hub IoT Edge facilite la communication entre les modules](./media/iot-edge-runtime/module-endpoints.png)

Pour envoyer des données au hub IoT Edge, un module appelle la méthode SendEventAsync. Le premier argument spécifie sur quelle sortie envoyer le message. Le pseudo-code suivant envoie un message sur **output1** :

   ```csharp
   ModuleClient client = await ModuleClient.CreateFromEnvironmentAsync(transportSettings);
   await client.OpenAsync();
   await client.SendEventAsync("output1", message);
   ```

Pour recevoir un message, inscrivez un rappel qui traite les messages entrant sur une entrée spécifique. Le pseudo-code suivant inscrit la fonction messageProcessor à utiliser pour le traitement de tous les messages reçus sur **input1** :

   ```csharp
   await client.SetInputMessageHandlerAsync("input1", messageProcessor, userContext);
   ```

Pour plus d’informations sur la classe ModuleClient et ses méthodes de communication, reportez-vous aux informations de référence sur l’API du langage de votre SDK préféré : [C#](/dotnet/api/microsoft.azure.devices.client.moduleclient), [C](/azure/iot-hub/iot-c-sdk-ref/iothub-module-client-h), [Python](/python/api/azure-iot-device/azure.iot.device.iothubmoduleclient), [Java](/java/api/com.microsoft.azure.sdk.iot.device.moduleclient) ou [Node.js](/javascript/api/azure-iot-device/moduleclient).

Le développeur de solution doit spécifier les règles qui déterminent la façon dont le hub IoT Edge transmet les messages d’un module à l’autre. Les règles d’acheminement sont définies dans le cloud et envoyées (push) vers le hub IoT Edge dans son jumeau de module. La même syntaxe pour les itinéraires IoT Hub est utilisée pour définir les itinéraires entre les modules dans Azure IoT Edge. Pour plus d’informations, consultez [Déployer des modules et établir des routes dans IoT Edge](module-composition.md).

![Les routes entre modules passent par le hub IoT Edge](./media/iot-edge-runtime/module-endpoints-routing.png)
::: moniker-end

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

### <a name="local-communication"></a>Communication locale

Le hub IoT Edge facilite la communication locale. Il permet la communication appareil-à-module, module-à-module et appareil-à-appareil en répartissant les messages pour que les appareils et les modules restent indépendants les uns des autres.

>[!NOTE]
> La fonctionnalité de répartiteur MQTT est disponible en préversion publique avec IoT Edge version 1.2. Elle doit être activée de manière explicite.

Le hub IoT Edge prend en charge deux mécanismes de répartition :

1. Les [fonctionnalités de routage des messages prises en charge par IoT Hub](../iot-hub/iot-hub-devguide-messages-d2c.md) ; et
2. Un répartiteur MQTT à usage général qui répond aux [normes MQTT v 3.1.1](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html).

#### <a name="using-routing"></a>Utilisation du routage

Le premier mécanisme de répartition utilise les mêmes fonctionnalités de routage qu’IoT Hub pour spécifier la façon dont les messages sont transmis entre les appareils ou les modules. Les premiers appareils ou modules spécifient les entrées sur lesquelles ils acceptent des messages et les sorties vers lesquelles ils écrivent des messages. Un développeur de solutions peut ensuite acheminer des messages entre une source (par exemple, des sorties) et une destination (par exemple, des entrées) avec des filtres potentiels.

![Les routes entre modules passent par le hub IoT Edge](./media/iot-edge-runtime/module-endpoints-routing.png)

Le routage peut être utilisé par des appareils ou des modules créés avec les kits Azure IoT device SDK via le protocole AMQP ou MQTT. Toutes les primitives IoT Hub de messagerie (par exemple : la télémétrie, les méthodes directes, C2D, les jumeaux) sont prises en charge, mais la communication sur les rubriques définies par l’utilisateur n’est pas prise en charge.

Pour plus d’informations sur les itinéraires, consultez [Découvrir comment déployer des modules et établir des routes dans IoT Edge](module-composition.md).

#### <a name="using-the-mqtt-broker"></a>Utilisation du répartiteur MQTT

Le deuxième mécanisme de répartition est basé sur un répartiteur MQTT standard. MQTT est un protocole léger de transfert de messages qui garantit des performances optimales sur les appareils à ressources restreintes et est une norme de publication et d’abonnement populaire. Les appareils ou les modules s’abonnent à des rubriques pour recevoir les messages publiés par d’autres appareils ou modules. Le hub IoT Edge implémente son propre répartiteur MQTT qui suit [les spécifications de la version 3.1.1 de MQTT](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html).

Le répartiteur MQTT active deux modèles de communication supplémentaires par rapport au routage par diffusion locale et à la communication point à point. La diffusion locale est utile quand un appareil ou un module doit alerter localement plusieurs autres appareils ou modules. La communication point à point permet à deux appareils IoT Edge ou deux appareils IoT de communiquer localement sans aller et venir par le cloud.

![Publier et s’abonner localement à l’aide du hub IoT Edge](./media/iot-edge-runtime/local-communnication-mqtt-broker.png)

Le répartiteur MQTT peut être utilisé par des appareils ou des modules créés avec les kits Azure IoT device SDK qui communiquent par le biais du protocole MQTT ou de tout client MQTT à usage général. À l’exception de C2D, toutes les primitives IoT Hub de messagerie, par exemple la télémétrie, les méthodes directes et les jumeaux, sont prises en charge. Les rubriques spéciales IoT Hub utilisées par les primitives IoT Hub sont prises en charge et sont donc des rubriques définies par l’utilisateur.
Il peut s’agir d’une rubrique spéciale IoT Hub ou d’une rubrique définie par l’utilisateur.

Contrairement au mécanisme de routage, la mise en ordre des messages n’est pas garantie et le filtrage des messages n’est pas pris en charge par le répartiteur. L’absence de ces fonctionnalités permet toutefois au répartiteur MQTT d’être plus rapide que le routage.

Pour plus d’informations sur le répartiteur MQTT, consultez [Publier et s’abonner avec IoT Edge](how-to-publish-subscribe.md).

#### <a name="comparison-between-brokering-mechanisms"></a>Comparaison entre les mécanismes de répartition

Voici les fonctionnalités disponibles avec chaque mécanisme de répartition :

|Fonctionnalités  | Routage  | Répartiteur MQTT  |
|---------|---------|---------|
|Télémétrie D2C    |     &#10004;    |         |
|Télémétrie locale     |     &#10004;    |    &#10004;     |
|DirectMethods     |    &#10004;     |    &#10004;     |
|Jumeau     |    &#10004;     |    &#10004;     |
|C2D pour les appareils     |   &#10004;      |         |
|Classement     |    &#10004;     |         |
|Filtrage     |     &#10004;    |         |
|Rubriques définies par l’utilisateur     |         |    &#10004;     |
|Appareil-à-appareil     |         |    &#10004;     |
|Diffusion locale     |         |    &#10004;     |
|Performances     |         |    &#10004;     |

### <a name="connecting-to-the-iot-edge-hub"></a>Connexion au hub IoT Edge

Le hub IoT Edge accepte les connexions à partir de clients d’appareil ou de module, soit via le protocole MQTT, soit via le protocole AMQP.

>[!NOTE]
> Le hub IoT Edge prend en charge les clients qui se connectent à l’aide de MQTT ou de AMQP. Il ne prend pas en charge les clients qui utilisent HTTP.

Lorsqu’un client se connecte au hub IoT Edge, voici ce qui se produit :

1. Si le protocole TLS (Transport Layer Security) est utilisé (recommandé), un canal TLS est créé pour établir une communication chiffrée entre le client et le hub IoT Edge.
2. Les informations d’authentification sont envoyées par le client au hub IoT Edge pour s’identifier.
3. Le hub IoT Edge autorise ou rejette la connexion en fonction de sa stratégie d’autorisation.

#### <a name="secure-connections-tls"></a>Connexions sécurisées (TLS)

Par défaut, le hub IoT Edge n’accepte que les connexions sécurisées par le protocole TLS (Transport Layer Security), par exemple, les connexions chiffrées qu’un tiers ne peut pas déchiffrer.

Si un client se connecte sur le port 8883 (MQTTS) ou 5671 (AMQPS) au hub IoT Edge, un canal TLS doit être généré. Pendant l’établissement d’une liaison TLS, le hub IoT Edge envoie sa chaîne de certificats que le client doit valider. Pour valider la chaîne de certificats, le certificat racine du hub IoT Edge doit être installé en tant que certificat approuvé sur le client. Si le certificat racine n’est pas approuvé, la bibliothèque de client est rejetée par le hub IoT Edge avec une erreur de vérification du certificat.

Les étapes à suivre pour installer ce certificat racine du répartiteur sur les clients d’appareil sont décrites dans la documentation relative à la [passerelle transparente](how-to-create-transparent-gateway.md) et à la [préparation d’un appareil en aval](how-to-connect-downstream-device.md#prepare-a-downstream-device). Les modules peuvent utiliser le même certificat racine que le hub IoT Edge en tirant parti de l’API de démon IoT Edge.

#### <a name="authentication"></a>Authentification

Le hub IoT Edge accepte uniquement les connexions à partir d’appareils ou de modules qui ont une identité IoT Hub, par exemple, qui ont été inscrits dans IoT Hub et qui disposent de l’une des trois méthodes d’authentification client prises en charge par IoT Hub pour prouver leur identité : [authentification par clé symétrique](how-to-authenticate-downstream-device.md#symmetric-key-authentication), [authentification auto-signée X.509](how-to-authenticate-downstream-device.md#x509-self-signed-authentication) et [authentification signée par une autorité de certification X.509](how-to-authenticate-downstream-device.md#x509-ca-signed-authentication).  Ces identités IoT Hub peuvent être vérifiées localement par le hub IoT Edge afin que les connexions puissent toujours être effectuées en mode hors connexion.

Remarques :

* Les modules IoT Edge prennent uniquement en charge l’authentification par clé symétrique pour le moment.
* Les clients MQTT ayant un nom d’utilisateur et un mot de passe locaux ne sont pas acceptés par le répartiteur MQTT du hub IoT Edge ; ils doivent utiliser des identités IoT Hub.

#### <a name="authorization"></a>Autorisation

Une fois authentifié, le hub IoT Edge dispose de deux méthodes pour autoriser les connexions client :

* En vérifiant qu’un client appartient à son ensemble de clients approuvés définis dans IoT Hub. L’ensemble des clients approuvés est spécifié en configurant des relations parent/enfant ou appareil/module dans IoT Hub. Lorsqu’un module est créé dans IoT Edge, une relation de confiance est établie automatiquement entre ce module et son appareil IoT Edge. Il s’agit du seul modèle d’autorisation pris en charge par le mécanisme de répartition du routage.

* En définissant une stratégie d’autorisation. Cette stratégie d’autorisation est un document répertoriant toutes les identités de clients autorisés qui peuvent accéder aux ressources sur le hub IoT Edge. Il s’agit du principal modèle d’autorisation utilisé par le répartiteur MQTT du hub IoT Edge, bien que les relations parent/enfant et appareil/module puissent également être comprises par le répartiteur MQTT pour les rubriques IoT Hub.

### <a name="remote-configuration"></a>Configuration distante

Le hub IoT Edge est entièrement contrôlé par le cloud. Il obtient sa configuration à partir d’IoT Hub par le biais de son [jumeau de module](iot-edge-modules.md#module-twins). Il inclut :

* Configuration des itinéraires
* Vous pouvez également utiliser les stratégies d'autorisation
* Configuration de pont MQTT

En outre, plusieurs configurations peuvent être définies en configurant [des variables d’environnement sur le hub IoT Edge](https://github.com/Azure/iotedge/blob/master/doc/EnvironmentVariables.md).
<!-- </1.2> -->
::: moniker-end

## <a name="runtime-quality-telemetry"></a>Télémétrie de qualité du runtime

IoT Edge collecte des données de télémétrie anonymes à partir du runtime hôte et des modules système pour améliorer la qualité du produit. Ces informations sont appelées « télémétrie de qualité du runtime ». Les données de télémétrie collectées sont régulièrement envoyées sous forme de messages appareil-à-cloud à IoT Hub à partir de l’agent IoT Edge. Ces messages n’apparaissent pas dans la télémétrie régulière du client et ne consomment aucun quota de messages.

L’agent IoT Edge et le hub génèrent des métriques que vous pouvez collecter pour comprendre les performances de l’appareil. Un sous-ensemble de ces métriques est collecté par l’agent IoT Edge dans le cadre de la télémétrie de qualité du runtime. Les métriques collectées pour la télémétrie de qualité du runtime sont marquées de la balise `ms_telemetry`. Pour plus d’informations sur toutes les métriques disponibles, consultez [Accéder aux métriques intégrées](how-to-access-built-in-metrics.md).

Les informations permettant d’identifier une personne ou une organisation, telles que les noms d’appareils et de modules, sont supprimées avant le chargement pour garantir la nature anonyme de la télémétrie de qualité du runtime.

L’agent IoT Edge collecte les données de télémétrie toutes les heures et envoie un message à IoT Hub toutes les 24 heures.

Si vous souhaitez refuser l’envoi de données de télémétrie du runtime à partir de vos appareils, deux méthodes sont possibles :

* Définissez la variable d’environnement `SendRuntimeQualityTelemetry` sur `false` pour **edgeAgent**, ou
* Décochez l’option dans le portail Azure pendant le déploiement.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation des modules Azure IoT Edge](iot-edge-modules.md)
* [Découvrir comment déployer des modules et établir des routes dans IoT Edge](module-composition.md)
* [Découvrir comment publier et s’abonner avec IoT Edge](how-to-publish-subscribe.md)
* [En savoir plus sur les métriques du runtime IoT Edge](how-to-access-built-in-metrics.md)
