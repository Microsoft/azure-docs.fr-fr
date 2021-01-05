---
title: Conserver l’état dans Linux – Azure Event Grid IoT Edge | Microsoft Docs
description: Conserver les métadonnées dans Linux
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 92333c2528303a6fa53fa30f47def33c33235d39
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171480"
---
# <a name="persist-state-in-linux"></a>Conserver l’état dans Linux

Les rubriques et les abonnements créés dans le module Event Grid sont stockés dans le système de fichiers conteneur par défaut. Sans persistance, toutes les métadonnées créées seraient perdues en cas de redéploiement du module. Pour préserver les données tout au long des déploiements et des redémarrages, il est nécessaire de les rendre persistantes en dehors du système de fichiers conteneur.

Par défaut, seules les métadonnées sont rendues persistantes et les événements sont toujours stockés en mémoire pour améliorer les performances. Suivez également la section Rendre les événements persistants pour activer la persistance des événements.

Cet article donne les étapes à suivre pour déployer le module Event Grid avec persistance dans les déploiements Linux.

> [!NOTE]
>Le module Event Grid s’exécute en tant qu’utilisateur à faibles privilèges avec l’UID `2000` et le nom `eventgriduser`.

## <a name="persistence-via-volume-mount"></a>Persistance par montage de volume

 Les [volumes Docker](https://docs.docker.com/storage/volumes/) permettent de conserver les données d’un déploiement à l’autre. Vous pouvez laisser Docker créer automatiquement un volume nommé dans le cadre du déploiement du module Event Grid. Cette option est la plus simple. Vous pouvez spécifier le nom du volume à créer dans la section **Binds** comme suit :

```json
  {
     "HostConfig": {
          "Binds": [
                 "<your-volume-name-here>:/app/metadataDb"
           ]
      }
   }
```

>[!IMPORTANT]
>Ne modifiez pas la deuxième partie de la valeur de liaison. Elle pointe vers un emplacement spécifique au sein du module. Dans le cas du module Event Grid sur Linux, il doit s’agir de **/app/metadataDb**.

Par exemple, la configuration suivante entraîne la création du volume **egmetadataDbVol** où les métadonnées sont conservées.

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
      "egmetadataDbVol:/app/metadataDb",
      "egdataDbVol:/app/eventsDb"
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

Au lieu de monter un volume, vous pouvez créer un répertoire sur le système hôte et le monter.

## <a name="persistence-via-host-directory-mount"></a>Persistance par montage du répertoire hôte

Au lieu d’un volume Docker, vous avez également la possibilité de monter un dossier hôte.

1. Commencez par créer un utilisateur portant le nom **eventgriduser** et ID **2000** sur l’ordinateur hôte en exécutant la commande suivante :

    ```sh
    sudo useradd -u 2000 eventgriduser
    ```
1. Créez un répertoire dans le système de fichiers hôte en exécutant la commande suivante.

   ```sh
   md <your-directory-name-here>
   ```

    Par exemple, l’exécution de la commande suivante crée un répertoire appelé **myhostdir**.

    ```sh
    md /myhostdir
    ```
1. Ensuite, faites de **eventgriduser** le propriétaire de ce dossier en exécutant la commande suivante.

   ```sh
   sudo chown eventgriduser:eventgriduser -hR <your-directory-name-here>
   ```

    Par exemple,

    ```sh
    sudo chown eventgriduser:eventgriduser -hR /myhostdir
    ```
1. Utilisez **Binds** pour monter le répertoire et redéployer le module Event Grid à partir du Portail Azure.

    ```json
    {
         "HostConfig": {
            "Binds": [
                "<your-directory-name-here>:/app/metadataDb",
                "<your-directory-name-here>:/app/eventsDb",
             ]
         }
    }
    ```

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
                  "/myhostdir:/app/metadataDb",
                  "/myhostdir2:/app/eventsDb"
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
    >Ne modifiez pas la deuxième partie de la valeur de liaison. Elle pointe vers un emplacement spécifique au sein du module. Dans le cas du module Event Grid sur Linux, il doit s’agir de **/app/metadataDb** et **/app/eventsDb**.


## <a name="persist-events"></a>Rendre les événements persistants

Pour activer la persistance des événements, vous devez d’abord activer la persistance des métadonnées via le montage d’un volume ou d’un répertoire hôte à l’aide des sections ci-dessus.

Points importants à noter concernant la persistance des événements :

* La persistance des événements est activée pour chaque abonnement aux événements et fait l’objet d’une option d’adhésion une fois qu’un volume ou un répertoire a été monté.
* La persistance des événements est configurée sur un abonnement aux événements au moment de la création et ne peut pas être modifiée une fois l’abonnement aux événements créé. Pour activer ou désactiver la persistance des événements, vous devez supprimer et recréer l’abonnement aux événements.
* La persistance des événements est presque toujours plus lente que les opérations en mémoire, mais la différence de vitesse dépend fortement des caractéristiques du lecteur. Le compromis entre la vitesse et la fiabilité est inhérent à tous les systèmes de messagerie, mais il ne devient généralement perceptible qu’à grande échelle.

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
