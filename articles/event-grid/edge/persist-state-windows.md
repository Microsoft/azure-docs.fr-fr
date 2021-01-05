---
title: Conserver l’état sur Windows – Azure Event Grid IoT Edge | Microsoft Docs
description: Conserver l’état sur Windows
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: f38e23a3af1e2c81ee012a4f3c268cbff3fc1bee
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002815"
---
# <a name="persist-state-in-windows"></a>Conserver l’état sur Windows

Les rubriques et les abonnements créés dans le module Event Grid sont stockés dans le système de fichiers conteneur par défaut. Sans persistance, toutes les métadonnées créées seraient perdues en cas de redéploiement du module. Pour préserver les données tout au long des déploiements et des redémarrages, il est nécessaire de les rendre persistantes en dehors du système de fichiers conteneur. 

Par défaut, seules les métadonnées sont rendues persistantes et les événements sont toujours stockés en mémoire pour améliorer les performances. Suivez également la section Rendre les événements persistants pour activer la persistance des événements.

Cet article donne les étapes à suivre pour déployer le module Event Grid avec persistance dans les déploiements Windows.

> [!NOTE]
>Le module Event Grid s’exécute en tant qu’utilisateur à faibles privilèges **ContainerUser** sur Windows.

## <a name="persistence-via-volume-mount"></a>Persistance par montage de volume

Les [volumes Docker](https://docs.docker.com/storage/volumes/) permettent de conserver les données d’un déploiement à l’autre. Pour monter un volume, il faut le créer avec des commandes Docker, accorder les autorisations nécessaires pour que le conteneur puisse le lire et y écrire des données, puis déployer le module.

1. Créez un volume en exécutant la commande suivante :

    ```sh
    docker -H npipe:////./pipe/iotedge_moby_engine volume create <your-volume-name-here>
    ```

    Par exemple,

   ```sh
   docker -H npipe:////./pipe/iotedge_moby_engine volume create myeventgridvol
   ```
1. Récupérez le répertoire hôte auquel le volume est mappé en exécutant la commande ci-dessous :

    ```sh
    docker -H npipe:////./pipe/iotedge_moby_engine volume inspect <your-volume-name-here>
    ```

    Par exemple,

   ```sh
   docker -H npipe:////./pipe/iotedge_moby_engine volume inspect myeventgridvol
   ```

   Exemple de sortie :

   ```json
   [
          {
            "CreatedAt": "2019-07-30T21:20:59Z",
            "Driver": "local",
            "Labels": {},
            "Mountpoint": "C:\\ProgramData\\iotedge-moby\u000bolumes\\myeventgridvol\\_data",
            "Name": "myeventgridvol",
            "Options": {},
            "Scope": "local"
          }
   ]
   ```
1. Ajoutez le groupe **Utilisateurs** à la valeur pointée par **Mountpoint** :
    1. Lancez l’Explorateur de fichiers.
    1. Accédez au dossier pointé par **Mountpoint**.
    1. Cliquez avec le bouton droit, puis sélectionnez **Propriétés**.
    1. Sélectionnez **Sécurité**.
    1. Sous *Noms de groupes ou d’utilisateurs, sélectionnez **Modifier**.
    1. Sélectionnez **Ajouter**, entrez `Users`, sélectionnez **Vérifier les noms**, puis sélectionnez **OK**.
    1. Sous *Autorisations pour les utilisateurs*, sélectionnez **Modifier**, puis **OK**.
1. Utilisez **Binds** pour monter ce volume et redéployer le module Event Grid à partir du Portail Azure.

   Par exemple,

    ```json
        {
              "Env": [
                "inbound__serverAuth__tlsPolicy=strict",
                "inbound__serverAuth__serverCert__source=IoTEdge",
                "inbound__clientAuth__sasKeys__enabled=false",
                "inbound__clientAuth__clientCert__enabled=true",
                "inbound__clientAuth__clientCert__source=IoTEdge",
                "inbound__clientAuth__clientCert__allowUnknownCA=true",
                "outbound__clientAuth__clientCert__enabled=true",
                "outbound__clientAuth__clientCert__source=IoTEdge",
                "outbound__webhook__httpsOnly=true",
                "outbound__webhook__skipServerCertValidation=false",
                "outbound__webhook__allowUnknownCA=true"
              ],
              "HostConfig": {
                "Binds": [
                  "<your-volume-name-here>:C:\\app\\metadataDb"
                ],
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

   >[!IMPORTANT]
   >Ne modifiez pas la deuxième partie de la valeur de liaison. Elle pointe vers un emplacement spécifique dans le module. Dans le cas du module Event Grid sur Windows, il doit s’agir de **C:\\app\\metadataDb**.


    Par exemple,

    ```json
    {
        "Env": [
            "inbound__serverAuth__tlsPolicy=strict",
            "inbound__serverAuth__serverCert__source=IoTEdge",
            "inbound__clientAuth__sasKeys__enabled=false",
            "inbound__clientAuth__clientCert__enabled=true",
            "inbound__clientAuth__clientCert__source=IoTEdge",
            "inbound__clientAuth__clientCert__allowUnknownCA=true",
            "outbound__clientAuth__clientCert__enabled=true",
            "outbound__clientAuth__clientCert__source=IoTEdge",
            "outbound__webhook__httpsOnly=true",
            "outbound__webhook__skipServerCertValidation=false",
            "outbound__webhook__allowUnknownCA=true"
         ],
         "HostConfig": {
            "Binds": [
                "myeventgridvol:C:\\app\\metadataDb",
                "C:\\myhostdir2:C:\\app\\eventsDb"
             ],
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

## <a name="persistence-via-host-directory-mount"></a>Persistance par montage du répertoire hôte

Au lieu de monter un volume, vous pouvez créer un répertoire sur le système hôte et le monter.

1. Créez un répertoire dans le système de fichiers hôte en exécutant la commande suivante.

   ```sh
   mkdir <your-directory-name-here>
   ```

   Par exemple,

   ```sh
   mkdir C:\myhostdir
   ```
1. Utilisez **Binds** pour monter votre répertoire et redéployer le module Event Grid à partir du Portail Azure.

    ```json
    {
         "HostConfig": {
            "Binds": [
                "<your-directory-name-here>:C:\\app\\metadataDb"
             ]
         }
    }
    ```

    >[!IMPORTANT]
    >Ne modifiez pas la deuxième partie de la valeur de liaison. Elle pointe vers un emplacement spécifique dans le module. Dans le cas du module Event Grid sur Windows, il doit s’agir de **C:\\app\\metadataDb**.

    Par exemple,

    ```json
    {
        "Env": [
            "inbound__serverAuth__tlsPolicy=strict",
            "inbound__serverAuth__serverCert__source=IoTEdge",
            "inbound__clientAuth__sasKeys__enabled=false",
            "inbound__clientAuth__clientCert__enabled=true",
            "inbound__clientAuth__clientCert__source=IoTEdge",
            "inbound__clientAuth__clientCert__allowUnknownCA=true",
            "outbound__clientAuth__clientCert__enabled=true",
            "outbound__clientAuth__clientCert__source=IoTEdge",
            "outbound__webhook__httpsOnly=true",
            "outbound__webhook__skipServerCertValidation=false",
            "outbound__webhook__allowUnknownCA=true"
         ],
         "HostConfig": {
            "Binds": [
                "C:\\myhostdir:C:\\app\\metadataDb",
                "C:\\myhostdir2:C:\\app\\eventsDb"
             ],
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
## <a name="persist-events"></a>Rendre les événements persistants

Pour activer la persistance des événements, vous devez d’abord activer la persistance des événements via le montage d’un volume ou d’un répertoire hôte à l’aide des sections ci-dessus.

Points importants à noter concernant la persistance des événements :

* La persistance des événements est activée pour chaque abonnement aux événements et fait l’objet d’une option d’adhésion une fois qu’un volume ou un répertoire a été monté.
* La persistance des événements est configurée sur un abonnement aux événements au moment de la création et ne peut pas être modifiée une fois l’abonnement aux événements créé. Pour activer ou désactiver la persistance des événements, vous devez supprimer et recréer l’abonnement aux événements.
* La persistance des événements est presque toujours plus lente que les opérations en mémoire, mais la différence de vitesse dépend fortement des caractéristiques du lecteur. Le compromis entre la vitesse et la fiabilité est inhérent à tous les systèmes de messagerie, mais il ne devient perceptible qu’à grande échelle.

Pour activer la persistance des événements sur un abonnement aux événements, définissez `persistencePolicy` sur `true` :

 ```json
        {
          "properties": {
            "persistencePolicy": {
              "isPersisted": "true"
            },
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-webhook-url>"
              }
            }
          }
        }
 ```