---
title: Réagir aux événements du module Stockage Blob – Azure Event Grid sur IoT Edge | Microsoft Docs
description: Réagir aux événements du module Stockage Blob
author: arduppal
manager: brymat
ms.author: arduppal
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 230e158a970f8c815b1575403c013e30749124c5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005059"
---
# <a name="tutorial-react-to-blob-storage-events-on-iot-edge-preview"></a>Tutoriel : Réagir aux événements du module Stockage Blob sur IoT Edge (préversion)
Cet article explique comment déployer le Stockage Blob Azure sur le module IoT, qui agirait en tant que serveur de publication Event Grid pour envoyer des événements lors de la création et de la suppression d’objets Blob à Event Grid.  

Pour obtenir une vue d’ensemble du Stockage Blob Azure sur IoT Edge, voir [Stockage d’objets Blob Azure sur IoT Edge](../../iot-edge/how-to-store-data-blob.md) et ses fonctionnalités.

> [!WARNING]
> L’intégration de Stockage Blob Azure sur IoT Edge avec Event Grid est en préversion

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

    ```json
        {
          "Env": [
           "inbound__serverAuth__tlsPolicy=enabled",
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
    > Dans ce tutoriel, vous allez découvrir comment déployer le module Event Grid pour autoriser des requêtes HTTP/HTTPS avec l’authentification du client désactivée. Dans le cas de charges de travail de production, nous vous recommandons d’autoriser uniquement les requêtes HTTPs et les abonnés pour lesquels l’authentification client est activée. Pour savoir comment configurer le module Event Grid de manière sécurisée, voir [Sécurité et authentification](security-authentication.md).
    

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
1. Passez à la section suivante pour ajouter le module Stockage Blob Azure

## <a name="deploy-azure-blob-storage-module"></a>Déployer le module Stockage Blob Azure

Cette section vous montre comment déployer le module Stockage Blob Azure qui agira comme un serveur de publication Event Grid qui publie des événements de création et de suppression d’objets blob.

### <a name="add-modules"></a>Ajouter des modules

1. Dans la section **Modules de déploiement**, sélectionnez **Ajouter**
2. Dans la liste déroulante des types de modules, sélectionnez **Module IoT Edge**
3. Fournissez le nom, l’image et les options de création du conteneur :

   * **Nom** : azureblobstorageoniotedge
   * **URI de l’image** : mcr.microsoft.com/azure-blob-storage:latest
   * **Options de création de conteneur** :

   ```json
       {
         "Env":[
           "LOCAL_STORAGE_ACCOUNT_NAME=<your storage account name>",
           "LOCAL_STORAGE_ACCOUNT_KEY=<your storage account key>",
           "EVENTGRID_ENDPOINT=http://<event grid module name>:5888"
         ],
         "HostConfig":{
           "Binds":[
               "<storage mount>"
           ],
           "PortBindings":{
             "11002/tcp":[{"HostPort":"11002"}]
           }
         }
       }
   ```

   > [!IMPORTANT]
   > - Le module de stockage d’objets blob peut publier des événements à l’aide des protocoles HTTPS et HTTP. 
   > - Si vous avez activé l’authentification basée sur le client pour EventGrid, veillez à mettre à jour la valeur d’EVENTGRID_ENDPOINT pour autoriser HTTPS, comme suit : `EVENTGRID_ENDPOINT=https://<event grid module name>:4438`.
   > - Ajoutez aussi une autre variable d’environnement `AllowUnknownCertificateAuthority=true` au code JSON ci-dessus. Lorsque vous parlez à EventGrid via HTTPS, **AllowUnknownCertificateAuthority** permet au module de stockage d’approuver les certificats de serveur EventGrid auto-signés.

4. Mettez à jour JSON que vous avez copié avec les informations suivantes :

   - Remplacez `<your storage account name>` par un nom que vous pouvez mémoriser. Les noms de compte doivent être compris entre 3 et 24 caractères, et inclure des lettres minuscules et des chiffres. Pas d’espace.

   - Remplacez `<your storage account key>` par une clé en base64 de 64 octets. Vous pouvez générer une clé avec des outils tels que [GeneratePlus](https://generate.plus/en/base64?gp_base64_base[length]=64). Vous utilisez ces informations d’identification pour accéder au stockage d’objets blob à partir d’autres modules.

   - Remplacez `<event grid module name>` par le nom de votre module Event Grid.
   - Remplacez `<storage mount>` en fonction du système d’exploitation de votre conteneur.
     - Pour les conteneurs Linux, **my-volume:/blobroot**
     - Pour les conteneurs Windows,**my-volume:C:/BlobRoot**

5. Cliquez sur **Enregistrer**.
6. Cliquez sur **Suivant** pour passer à la section des itinéraires

    > [!NOTE]
    > Si vous utilisez une machine virtuelle Azure comme appareil de périphérie, ajoutez une règle de port entrant pour autoriser le trafic entrant sur les ports hôtes utilisés dans ce didacticiel : 4438, 5888, 8080 et 11002. Pour savoir comment ajouter la règle, voir [Guide pratique pour ouvrir des ports sur une machine virtuelle](../../virtual-machines/windows/nsg-quickstart-portal.md).

### <a name="setup-routes"></a>Configurer les itinéraires

Conservez les itinéraires par défaut, puis sélectionnez **Suivant** pour passer à la section de vérification

### <a name="review-deployment"></a>Vérifier le déploiement

1. La section de vérification vous montre le manifeste de déploiement JSON qui a été créé en fonction de vos sélections dans la section précédente. Vérifiez que vous voyez les quatre modules **$edgeAgent**, **$edgeHub**, **eventgridmodule**, **subscriber** and **azureblobstorageoniotedge** en cours de déploiement.
2. Passez en revue les informations de votre déploiement, puis sélectionnez **Envoyer**.

## <a name="verify-your-deployment"></a>Vérifier votre déploiement

1. Une fois le déploiement envoyé, la page IoT Edge du hub IoT s’affiche à nouveau.
2. Sélectionnez l’**appareil IoT Edge** que vous avez ciblé avec le déploiement pour accéder à ses informations détaillées.
3. Dans les détails de l’appareil, vérifiez que les modules eventgridmodule, subscriber et azureblobstorageoniotedge sont répertoriés comme **spécifiés dans le déploiement** et **signalés par l’appareil**.

   Il peut s’écouler quelques instants avant que le module ne démarre sur l’appareil et qu’il ne soit renvoyé à IoT Hub. Actualisez la page pour afficher un état mis à jour.

## <a name="publish-blobcreated-and-blobdeleted-events"></a>Publier des événements BlobCreated et BlobDeleted

1. Ce module crée automatiquement la rubrique **MicrosoftStorage**. Vérifiez qu’elle existe
    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage?api-version=2019-01-01-preview
    ```

    Exemple de sortie :

    ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/MicrosoftStorage",
            "name": "MicrosoftStorage",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/MicrosoftStorage/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
    ```

    > [!IMPORTANT]
    > - Pour le flux HTTPS, si l’authentification client est activée via la clé SAS, la clé SAS spécifiée précédemment doit être ajoutée en tant qu’en-tête. Par conséquent, la demande curl sera : `curl -k -H "Content-Type: application/json" -H "aeg-sas-key: <your SAS key>" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage?api-version=2019-01-01-preview`
    > - Pour le flux HTTPS, si l’authentification client est activée via un certificat, la demande curl sera : `curl -k -H "Content-Type: application/json" --cert <certificate file> --key <certificate private key file> -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage?api-version=2019-01-01-preview`

2. Les abonnés peuvent s’inscrire aux événements publiés dans une rubrique. Pour recevoir des événements, vous devez créer un abonnement Event Grid pour la rubrique **MicrosoftStorage**.
    1. Créez blobsubscription.json avec le contenu suivant. Pour plus d’informations sur la charge utile, consultez notre [documentation sur l’API](api.md)

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
       > La propriété **endpointType** spécifie que l’abonné est un **webhook**.  **endpointUrl** spécifie l’URL à laquelle l’abonné écoute les événements. Cette URL correspond à l’exemple de fonction Azure que vous avez déployé précédemment.

    2. Exécutez la commande suivante pour créer un abonnement pour la rubrique. Vérifiez que le code d’état HTTP est `200 OK`.

       ```sh
       curl -k -H "Content-Type: application/json" -X PUT -g -d @blobsubscription.json https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview
       ```

       > [!IMPORTANT]
       > - Pour le flux HTTPS, si l’authentification client est activée via la clé SAS, la clé SAS spécifiée précédemment doit être ajoutée en tant qu’en-tête. Par conséquent, la demande curl sera : `curl -k -H "Content-Type: application/json" -H "aeg-sas-key: <your SAS key>" -X PUT -g -d @blobsubscription.json https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview` 
       > - Pour le flux HTTPS, si l’authentification client est activée via un certificat, la demande curl sera : `curl -k -H "Content-Type: application/json" --cert <certificate file> --key <certificate private key file> -X PUT -g -d @blobsubscription.json https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview`

    3. Exécutez la commande suivante pour vérifier la bonne création de l’abonnement. Le code d’état HTTP 200 OK doit être retourné.

       ```sh
       curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview
       ```

       Exemple de sortie :

       ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription5",
          "properties": {
            "Topic": "MicrosoftStorage",
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "https://subscriber:4430"
              }
            }
          }
        }
       ```

       > [!IMPORTANT]
       > - Pour le flux HTTPS, si l’authentification client est activée via la clé SAS, la clé SAS spécifiée précédemment doit être ajoutée en tant qu’en-tête. Par conséquent, la demande curl sera : `curl -k -H "Content-Type: application/json" -H "aeg-sas-key: <your SAS key>" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview`
       > - Pour le flux HTTPS, si l’authentification client est activée via un certificat, la demande curl sera : `curl -k -H "Content-Type: application/json" --cert <certificate file> --key <certificate private key file> -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/MicrosoftStorage/eventSubscriptions/sampleSubscription5?api-version=2019-01-01-preview`

3. Téléchargez l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) et [Connectez-le à votre stockage local](../../iot-edge/how-to-store-data-blob.md#connect-to-your-local-storage-with-azure-storage-explorer)

## <a name="verify-event-delivery"></a>Vérifier la livraison de l’événement

### <a name="verify-blobcreated-event-delivery"></a>Vérifier la livraison des événements BlobCreated

1. Chargez les fichiers en tant qu’objets blob de blocs dans le stockage local à partir de Explorateur Stockage Azure. Le module publiera automatiquement les événements de création. 
2. Consultez les journaux d’abonnés pour l’événement de création. Suivez les étapes pour [vérifier la livraison d’événement](pub-sub-events-webhook-local.md#verify-event-delivery)

    Exemple de sortie :

    ```json
            Received Event:
            {
              "id": "d278f2aa-2558-41aa-816b-e6d8cc8fa140",
              "topic": "MicrosoftStorage",
              "subject": "/blobServices/default/containers/cont1/blobs/Team.jpg",
              "eventType": "Microsoft.Storage.BlobCreated",
              "eventTime": "2019-10-01T21:35:17.7219554Z",
              "dataVersion": "1.0",
              "metadataVersion": "1",
              "data": {
                "api": "PutBlob",
                "clientRequestId": "00000000-0000-0000-0000-000000000000",
                "requestId": "ef1c387b-4c3c-4ac0-8e04-ff73c859bfdc",
                "eTag": "0x8D746B740DA21FB",
                "url": "http://azureblobstorageoniotedge:11002/myaccount/cont1/Team.jpg",
                "contentType": "image/jpeg",
                "contentLength": 858129,
                "blobType": "BlockBlob"
              }
            }
    ```

### <a name="verify-blobdeleted-event-delivery"></a>Vérifier la livraison d’événement BlobDeleted

1. Supprimez les objets blob du stockage local à l’aide de l’Explorateur Stockage Azure, et le module publiera automatiquement les événements de suppression. 
2. Consultez les journaux d’abonnés pour l’événement de suppression. Suivez les étapes pour [vérifier la livraison d’événement](pub-sub-events-webhook-local.md#verify-event-delivery)

    Exemple de sortie :
    
    ```json
            Received Event:
            {
              "id": "ac669b6f-8b0a-41f3-a6be-812a3ce6ac6d",
              "topic": "MicrosoftStorage",
              "subject": "/blobServices/default/containers/cont1/blobs/Team.jpg",
              "eventType": "Microsoft.Storage.BlobDeleted",
              "eventTime": "2019-10-01T21:36:09.2562941Z",
              "dataVersion": "1.0",
              "metadataVersion": "1",
              "data": {
                "api": "DeleteBlob",
                "clientRequestId": "00000000-0000-0000-0000-000000000000",
                "requestId": "2996bbfb-c819-4d02-92b1-c468cc67d8c6",
                "eTag": "0x8D746B740DA21FB",
                "url": "http://azureblobstorageoniotedge:11002/myaccount/cont1/Team.jpg",
                "contentType": "image/jpeg",
                "contentLength": 858129,
                "blobType": "BlockBlob"
              }
            }
    ```

Félicitations ! Vous avez terminé le didacticiel. Les sections suivantes fournissent des détails sur les propriétés d’événement.

### <a name="event-properties"></a>Propriétés d’événement

Voici la liste des propriétés d’événement prises en charge, ainsi que leur type et leur description. 

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| topic | string | Chemin d’accès complet à la source de l’événement. Ce champ n’est pas modifiable. Event Grid fournit cette valeur. |
| subject | string | Chemin de l’objet de l’événement, défini par le serveur de publication. |
| eventType | string | Un des types d’événements inscrits pour cette source d’événement. |
| eventTime | string | L’heure à quelle l’événement est généré selon l’heure UTC du fournisseur. |
| id | string | Identificateur unique de l’événement. |
| data | object | Données d’événement de stockage Blob. |
| dataVersion | string | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| api | string | Opération qui a déclenché l’événement. Ce peut être l’une des valeurs suivantes : <ul><li>BlobCreated – Les valeurs autorisées sont : `PutBlob` et `PutBlockList`</li><li>BlobDeleted – Les valeurs autorisées sont `DeleteBlob`, `DeleteAfterUpload` et `AutoDelete`. <p>L’événement `DeleteAfterUpload` est généré lorsque l’objet blob est supprimé automatiquement, parce que la propriété deleteAfterUpload souhaitée a la valeur true. </p><p>L’événement `AutoDelete` est généré lorsque l’objet blob est supprimé automatiquement, parce que la valeur la propriété deleteAfterMinutes souhaitée a expiré.</p></li></ul>|
| clientRequestId | string | ID de requête fourni par le client pour l’opération d’API de stockage. Cet ID peut être utilisé pour mettre en corrélation les journaux de diagnostic Stockage Azure en utilisant le champ « client-request-id » dans les journaux, et peut être fourni dans des requêtes du client à l’aide de l’en-tête « x-ms-client-request-id ». Pour plus d’informations, voir [Format de journal](/rest/api/storageservices/storage-analytics-log-format). |
| requestId | string | ID de requête généré par le service pour l’opération de l’API de stockage. Peut être utilisé pour mettre en corrélation les journaux de diagnostic de stockage Azure en utilisant le champ « request-id-header » dans les journaux d’activité et est retourné lors de l’initialisation de l’appel d’API dans l’en-tête ’x-ms-request-id’. Consultez [Format de journal](/rest/api/storageservices/storage-analytics-log-format). |
| eTag | string | Valeur que vous pouvez utiliser pour effectuer des opérations de manière conditionnelle. |
| contentType | string | Type de contenu spécifié pour l’objet blob. |
| contentLength | entier | Taille de l’objet blob en octets. |
| blobType | string | Type d’objet blob. Les valeurs valides sont « BlockBlob » ou « PageBlob ». |
| url | string | Chemin de l’objet blob. <br>Si le client utilise une API REST Blob, la structure de l’URL est *\<storage-account-name\>.blob.core.windows.net/\<container-name\>/\<file-name\>* . <br>Si le client utilise une API REST Data Lake Storage, la structure de l’URL est *\<storage-account-name\>.dfs.core.windows.net/\<file-system-name\>/\<file-name\>* . |


## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants de la documentation relative à Stockage Blob :

- [Filtrer les événements de Stockage Blob](../../storage/blobs/storage-blob-event-overview.md#filtering-events)
- [Pratiques recommandées pour l’utilisation des événements de Stockage Blob](../../storage/blobs/storage-blob-event-overview.md#practices-for-consuming-events)

Dans ce didacticiel, vous avez publié des événements en créant ou supprimant des objets blob dans un Stockage Blob Azure. Consultez les autres didacticiels pour savoir comment transférer des événements vers le cloud (Azure Event Hub ou Azure IoT Hub) : 

- [Transférer des événements à Azure Event Grid](forward-events-event-grid-cloud.md)
- [Transférer des événements à Azure IoT Hub](forward-events-iothub.md)
