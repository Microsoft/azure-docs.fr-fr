---
title: Publier, s’abonner aux événements localement - Azure Event Grid IoT Edge | Microsoft Docs
description: Publier, s’abonner aux événements localement à l’aide de webhook avec Event Grid sur IoT Edge
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 2a7cc864366bd9a35c96dd453c0dc68f77d8abd9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171446"
---
# <a name="tutorial-publish-subscribe-to-events-locally"></a>Tutoriel : Publier, s'abonner à des événements localement

Cet article vous guide tout au long des étapes nécessaires pour publier et s’abonner à des événements avec Event Grid sur IoT Edge.

> [!NOTE]
> Pour en savoir plus sur les rubriques et les abonnements d’Azure Event Grid, consultez [Concepts d’Event Grid](concepts.md).

## <a name="prerequisites"></a>Prérequis 
Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* **Abonnement Azure** : Créez un [compte gratuit](https://azure.microsoft.com/free) si vous n’en avez pas encore. 
* **Appareil Azure IoT Hub et IoT Edge** : suivez les étapes du guide de démarrage rapide pour les appareils [Linux](../../iot-edge/quickstart-linux.md) ou [Windows](../../iot-edge/quickstart.md) si vous n’en avez pas encore.

## <a name="deploy-event-grid-iot-edge-module"></a>Déployer le module Event Grid sur IoT Edge

Il existe plusieurs moyens de déployer des modules sur un appareil IoT Edge ; tous fonctionnent pour Azure Event Grid sur IoT Edge. Cet article décrit les étapes à suivre pour déployer Event Grid sur IoT Edge sur le Portail Azure.

>[!NOTE]
> Dans ce tutoriel, vous allez déployer le module Event Grid sans persistance. Cela signifie que toutes les rubriques et tous les abonnements créés dans ce tutoriel seront supprimés lors du redéploiement du module. Pour savoir comment configurer la persistance, voir les articles suivants : [Conserver l’état dans Linux](persist-state-linux.md) ou [Conserver l’état dans Windows](persist-state-windows.md). Pour les charges de travail de production, nous vous recommandons d’installer le module Event Grid avec la persistance.


### <a name="select-your-iot-edge-device"></a>Sélectionnez votre appareil IoT Edge

1. Connectez-vous au [portail Azure](https://portal.azure.com)
1. Accédez à votre IoT Hub.
1. Sélectionnez **IoT Edge** dans le menu de la section **Gestion automatique des appareils**. 
1. Cliquez sur l’ID de l’appareil cible dans la liste des appareils
1. Sélectionnez **Définir modules**. Laissez la page ouverte. Vous continuerez avec les étapes de la prochaine section.

### <a name="configure-a-deployment-manifest"></a>Configurer un manifeste de déploiement

Un manifeste de déploiement est un document JSON qui décrit les modules à déployer, le flux des données entre les modules et les propriétés souhaitées des jumeaux de module. Le Portail Azure comprend un Assistant qui vous guide à travers la création du manifeste de déploiement, vous évitant de générer le document JSON manuellement.  Il comporte trois étapes : **Ajouter des modules**, **Spécifier des routes** et **Vérifier le déploiement**.

### <a name="add-modules"></a>Ajouter des modules

1. Dans la section **Modules de déploiement**, sélectionnez **Ajouter**
1. Dans la liste déroulante des types de modules, sélectionnez **Module IoT Edge**
1. Indiquez le nom, l’image et les options de création du conteneur :

   * **Nom** : eventgridmodule
   * **URI de l’image**  : `mcr.microsoft.com/azure-event-grid/iotedge:latest`
   * **Options de création de conteneur** :

   [!INCLUDE [event-grid-edge-module-version-update](../../../includes/event-grid-edge-module-version-update.md)]

    ```json
        {
          "Env": [
            "inbound__clientAuth__clientCert__enabled=false"
          ],
          "HostConfig": {
            "PortBindings": {
              "4438/tcp": [
                {
                    "HostPort": "4438"
                }
              ]
            }
          }
        }
    ```    
 1. Cliquez sur **Enregistrer**.
 1. Passez à la section suivante pour ajouter le module d’abonné Azure Event Grid avant de les déployer ensemble.

    >[!IMPORTANT]
    > Dans ce tutoriel, vous allez déployer le module Event Grid avec l’authentification du client désactivée. Pour les charges de travail de production, nous vous recommandons d’activer l’authentification client. Pour savoir comment configurer le module Event Grid de manière sécurisée, voir [Sécurité et authentification](security-authentication.md).
    > 
    > Si vous utilisez une machine virtuelle Azure comme appareil Edge, ajoutez une règle de port entrant pour autoriser le trafic entrant sur le port 4438. Pour savoir comment ajouter la règle, voir [Guide pratique pour ouvrir des ports sur une machine virtuelle](../../virtual-machines/windows/nsg-quickstart-portal.md).
    

## <a name="deploy-event-grid-subscriber-iot-edge-module"></a>Déployer le module IoT Edge d’abonné Event Grid

Cette section montre comment déployer un autre module IoT qui agirait en tant que gestionnaire d’événements auquel des événements peuvent être remis.

### <a name="add-modules"></a>Ajouter des modules

1. Dans la section **Modules de déploiement**, sélectionnez à nouveau **Ajouter**. 
1. Dans la liste déroulante des types de modules, sélectionnez **Module IoT Edge**
1. Fournissez le nom, l’image et les options de création du conteneur :

   * **Nom** : abonné
   * **URI de l’image**  : `mcr.microsoft.com/azure-event-grid/iotedge-samplesubscriber:latest`
   * **Options de création de conteneur** : None
1. Cliquez sur **Enregistrer**.
1. Cliquez sur **Suivant** pour passer à la section des itinéraires

 ### <a name="setup-routes"></a>Configurer les itinéraires

Conservez les itinéraires par défaut, puis sélectionnez **Suivant** pour passer à la section de vérification

### <a name="submit-the-deployment-request"></a>Envoyer la demande de déploiement

1. La section de vérification vous montre le manifeste de déploiement JSON qui a été créé en fonction de vos sélections dans la section précédente. Vérifiez que vous voyez les deux modules : **eventgridmodule** et **subscriber** répertoriés dans le JSON. 
1. Passez en revue les informations de votre déploiement, puis sélectionnez **Envoyer**. Une fois que vous envoyez le déploiement, vous revenez à la page **appareil**.
1. Dans la **section Modules**, vérifiez que les modules **eventgrid** et **subscriber** sont répertoriés. Et vérifiez que les colonnes **Spécifié dans de déploiement** et **Signalé par l’appareil** ont la valeur **Oui**.

    Il peut s’écouler quelques instants avant que le module ne démarre sur l’appareil et qu’il ne soit renvoyé à IoT Hub. Actualisez la page pour afficher un état mis à jour.

## <a name="create-a-topic"></a>Création d'une rubrique

En tant qu’éditeur d’un événement, vous devez créer une rubrique Event Grid. Dans Azure Event Grid, une rubrique fait référence à un point de terminaison auquel les éditeurs peuvent envoyer des événements.

1. Créez topic.json avec le contenu suivant. Pour plus d’informations sur la charge utile, consultez notre [documentation sur l’API](api.md).

    ```json
        {
          "name": "sampleTopic1",
          "properties": {
            "inputschema": "eventGridSchema"
          }
        }
    ```

1. Exécutez la commande suivante pour créer une rubrique de grille d’événements. Vérifiez que le code d’état HTTP est `200 OK`.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1?api-version=2019-01-01-preview
    ```

1. Exécutez la commande suivante pour vérifier la bonne création de la rubrique. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1?api-version=2019-01-01-preview
    ```

   Exemple de sortie :

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic1",
            "name": "sampleTopic1",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic1/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```

## <a name="create-an-event-subscription"></a>Créer un abonnement d’événement

Les abonnés peuvent s’inscrire aux événements publiés dans une rubrique. Pour recevoir des événements, vous devez créer un abonnement Event Grid pour un sujet qui vous intéresse.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Créez subscription.json avec le contenu suivant. Pour plus d’informations sur la charge utile, consultez notre [documentation sur l’API](api.md)

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
    ```

    >[!NOTE]
    > La propriété **endpointType** spécifie que l’abonné est un **webhook**.  **endpointUrl** spécifie l’URL à laquelle l’abonné écoute les événements. Cette URL correspond à l’exemple d’abonné Azure que vous avez déployé précédemment.
2. Exécutez la commande suivante pour créer un abonnement pour la rubrique. Vérifiez que le code d’état HTTP est `200 OK`.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1/eventSubscriptions/sampleSubscription1?api-version=2019-01-01-preview
    ```
3. Exécutez la commande suivante pour vérifier la bonne création de l’abonnement. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1/eventSubscriptions/sampleSubscription1?api-version=2019-01-01-preview
    ```

    Exemple de sortie :

   ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic1/eventSubscriptions/sampleSubscription1",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription1",
          "properties": {
            "Topic": "sampleTopic1",
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
    ```

## <a name="publish-an-event"></a>Publier un événement

1. Créez event.json avec le contenu suivant. Pour plus d’informations sur la charge utile, consultez notre [documentation sur l’API](api.md).

    ```json
        [
          {
            "id": "eventId-func-0",
            "eventType": "recordInserted",
            "subject": "myapp/vehicles/motorcycles",
            "eventTime": "2019-07-28T21:03:07+00:00",
            "dataVersion": "1.0",
            "data": {
              "make": "Ducati",
              "model": "Monster"
            }
          }
        ]
    ```
1. Exécutez la commande suivante pour publier un événement.

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1/events?api-version=2019-01-01-preview
    ```

## <a name="verify-event-delivery"></a>Vérifier la livraison de l’événement

1. SSH ou RDP dans votre machine virtuelle IoT Edge.
1. Vérifiez les journaux des abonnés :

    Sur Windows, exécutez la commande ci-dessous :

    ```sh
    docker -H npipe:////./pipe/iotedge_moby_engine container logs subscriber
    ```

   Sur Linux, exécutez la commande ci-dessous :

    ```sh
    sudo docker logs subscriber
    ```

    Exemple de sortie :

    ```sh
        Received Event:
            {
              "id": "eventId-func-0",
              "topic": "sampleTopic1",
              "subject": "myapp/vehicles/motorcycles",
              "eventType": "recordInserted",
              "eventTime": "2019-07-28T21:03:07+00:00",
              "dataVersion": "1.0",
              "metadataVersion": "1",
              "data": {
                "make": "Ducati",
                "model": "Monster"
              }
            }
    ```

## <a name="cleanup-resources"></a>Nettoyer les ressources

* Exécutez la commande suivante pour supprimer la rubrique et tous ses abonnements.

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic1?api-version=2019-01-01-preview
    ```
* Supprimez le module d’abonnés de votre appareil IoT Edge.


## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez créé une rubrique Event Grid, un abonnement et des événements publiés. Maintenant que vous connaissez les étapes de base, consultez les articles suivants : 

- Pour résoudre les problèmes liés à l’utilisation d’Azure Event Grid sur IoT Edge, voir le [Guide de dépannage](troubleshoot.md).
- Créer/mettre à jour un abonnement avec des [filtres](advanced-filtering.md).
- Activer la persistance du module Event Grid module sur [Linux](persist-state-linux.md) ou [Windows](persist-state-windows.md)
- Suivre la [documentation](configure-client-auth.md) pour configurer l’authentification du client
- Transférer les événements à Azure Functions dans le cloud en suivant ce [didacticiel](pub-sub-events-webhook-cloud.md)
- [Réagir à des événements Storage Blob sur IoT Edge](react-blob-storage-events-locally.md)
- [Superviser les rubriques et les abonnements à la périphérie](monitor-topics-subscriptions.md)

