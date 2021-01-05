---
title: Configuration du module proxy d’API – Azure IoT Edge | Microsoft Docs
description: Découvrez comment personnaliser le module proxy d’API pour les hiérarchies de passerelle IoT Edge.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/10/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
monikerRange: '>=iotedge-2020-11'
ms.openlocfilehash: 894398d63e326db3c6ee9de9bebc426a6e621600
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024668"
---
# <a name="configure-the-api-proxy-module-for-your-gateway-hierarchy-scenario-preview"></a>Configuration du module proxy d’API pour un scénario de hiérarchie de passerelle (préversion)

Le module proxy d’API offre aux appareils IoT Edge la possibilité d’envoyer des requêtes HTTP par le biais de passerelles au lieu d’établir des connexions directes aux services cloud. Cet article présente les options de configuration qui vous permettent de personnaliser le module en fonction des exigences de votre hiérarchie de passerelle.

>[!NOTE]
>Les conteneurs Linux doivent posséder la version 1.2 de IoT Edge, en préversion publique.

Dans certaines architectures réseau, les appareils IoT Edge situés derrière des passerelles ne possèdent pas d’accès direct au cloud. Les modules qui tentent de se connecter aux services cloud échouent. Le module proxy d’API prend en charge les appareils IoT Edge en aval dans cette configuration en réacheminant les connexions des modules à travers les couches d’une hiérarchie de passerelle. Il offre aux clients un moyen sécurisé de communiquer avec plusieurs services via le protocole HTTPS sans tunneling, mais en terminant les connexions au niveau de chaque couche. Il fonctionne comme un module proxy entre les appareils IoT Edge dans une hiérarchie de passerelle jusqu’à ce qu’ils atteignent l’appareil IoT Edge de la couche supérieure. Les services qui s’exécutent sur cet appareil gèrent alors ces demandes, et le module proxy d’API redirige tout le trafic HTTP des services locaux vers le cloud sur un seul port.

Le module proxy d’API offre de nombreux cas d’usage avec les hiérarchies de passerelle. Il permet notamment aux appareils de couche inférieure de tirer (pull) des images conteneur ou d’envoyer (push) des objets blob dans un système de stockage. C’est la configuration des règles du proxy qui définit le mode de routage des données. Cet article décrit plusieurs options de configuration courantes.

## <a name="deploy-the-proxy-module"></a>Déploiement du module proxy

Le module proxy d’API est disponible dans Microsoft Container Registry (MCR) : `mcr.microsoft.com/azureiotedge-api-proxy:latest`.

Vous pouvez également déployer directement le module proxy d’API à partir de la Place de marché Azure : [Proxy d’API IoT Edge](https://azuremarketplace.microsoft.com/marketplace/apps/azure-iot.azureiotedge-api-proxy?tab=Overview).

## <a name="understand-the-proxy-module"></a>Présentation du module proxy

Le module proxy d’API utilise un proxy inverse nginx pour acheminer les données à travers les couches réseau. Un proxy est incorporé dans le module. Par conséquent, l’image de module doit prendre en charge la configuration du proxy. Par exemple, si le proxy écoute un certain port, ce dernier doit être ouvert sur le module.

Le proxy commence par un fichier de configuration par défaut incorporé dans le module. Vous pouvez transmettre une nouvelle configuration au module à partir du cloud à l’aide de son [jumeau de module](../iot-hub/iot-hub-devguide-module-twins.md). Il est également possible d’utiliser des variables d’environnement pour activer ou désactiver les paramètres de configuration au moment du déploiement.

Cet article se concentre d’abord sur le fichier de configuration par défaut et sur l’activation de ses paramètres au moyen des variables d'environnement. La personnalisation du fichier de configuration sera abordée à la fin.

### <a name="default-configuration"></a>Configuration par défaut

Le module proxy d’API est fourni avec une configuration par défaut qui prend en charge les scénarios courants et accepte la personnalisation. Il est possible de contrôler la configuration par défaut au moyen des variables d’environnement du module.

Actuellement, les variables d’environnement par défaut sont les suivantes :

| Variable d’environnement | Description |
| -------------------- | ----------- |
| `PROXY_CONFIG_ENV_VAR_LIST` | Liste toutes les variables à mettre à jour dans une liste séparée par des virgules. Cette étape empêche de modifier accidentellement les mauvais paramètres de configuration.
| `NGINX_DEFAULT_PORT` | Modifie le port écouté par le proxy nginx. Si vous mettez à jour cette variable d’environnement, veillez à ce que le port sélectionné soit également exposé dans le Dockerfile du module et déclaré comme liaison de port dans le manifeste de déploiement.<br><br>La valeur par défaut est 443.<br><br>En cas de déploiement à partir de la Place de marché Azure, le port par défaut devient 8000 pour empêcher les conflits avec le module edgeHub. Pour plus d’informations, consultez [Réduction des ports ouverts](#minimize-open-ports). |
| `DOCKER_REQUEST_ROUTE_ADDRESS` | Adresse à laquelle sont acheminées les demandes Docker. Modifiez cette variable sur l’appareil de la couche supérieure de sorte qu’elle pointe vers le module registre.<br><br>La valeur par défaut est le nom d’hôte parent. |
| `BLOB_UPLOAD_ROUTE_ADDRESS` | Adresse à laquelle sont acheminées les demandes du registre blob. Modifiez cette variable sur l’appareil de la couche supérieure de sorte qu’elle pointe vers le Stockage Blob.<br><br>La valeur par défaut est le nom d’hôte parent. |

## <a name="minimize-open-ports"></a>Réduction des ports ouverts

Pour réduire le nombre de ports ouverts, le module proxy d’API doit relayer tout le trafic HTTPS (port 443), y compris le trafic ciblant le module edgeHub. Il est configuré par défaut pour réacheminer tout le trafic edgeHub sur le port 443.

Pour configurer votre déploiement de façon à réduire les ports ouverts, procédez comme suit :

1. Mettez à jour les paramètres du module edgeHub pour qu’ils ne soient pas liés au port 443, afin d’éviter des conflits de liaison de port. Par défaut, ce module est lié aux ports 443, 5671 et 8883. Supprimez la liaison du port 443 et conservez les deux autres :

   ```json
   {
     "HostConfig": {
       "PortBindings": {
         "5671/tcp": [
           {
             "HostPort": "5671"
           }
         ],
         "8883/tcp": [
           {
             "HostPort": "8883"
           }
         ]
       }
     }
   }
   ```

1. Configurez le module proxy d’API de sorte qu’il soit lié au port 443.

   1. Donnez à la variable d’environnement **NGINX_DEFAULT_PORT** la valeur `443`.
   1. Mettez à jour les options de création de conteneur de façon à effectuer une liaison au port 443.

      ```json
      {
        "HostConfig": {
          "PortBindings": {
            "443/tcp": [
              {
                "HostPort": "443"
              }
            ]
          }
        }
      }
      ```

Si vous n’avez pas besoin de réduire les ports ouverts, vous pouvez laisser le module edgeHub utiliser le port 443 et configurer le module proxy d’API de sorte qu’il écoute un autre port. Par exemple, le module proxy d’API peut écouter le port 8000. Pour cela, donnez à la variable d’environnement **NGINX_DEFAULT_PORT** la valeur `8000` et créez une liaison pour le port 8000.

## <a name="enable-container-image-download"></a>Téléchargement des images conteneur

L’un des cas d’usage courants du module proxy d’API consiste à permettre aux appareils IoT Edge des couches inférieures de tirer (pull) des images conteneur. Ce scénario utilise le [module registre Docker](https://hub.docker.com/_/registry) pour récupérer les images conteneur à partir du cloud et les mettre en cache au niveau de la couche supérieure. Le proxy d’API relaie toutes les requêtes HTTPS pour télécharger à partir des couches inférieures une image conteneur, que le module registre devra traiter dans la couche supérieure.

Dans ce scénario, les appareils IoT Edge en aval doivent pointer vers le nom de domaine `$upstream` suivi du numéro de port du module proxy d’API, au lieu du registre de conteneurs d’une image. Par exemple : `$upstream:8000/azureiotedge-api-proxy:1.0`.

Ce cas d’usage est illustré dans le tutoriel [Création d’une hiérarchie d’appareils IoT Edge à l’aide de passerelles](tutorial-nested-iot-edge.md).

Configurez les modules suivants au niveau de la **couche supérieure** :

* Un module registre Docker
  * Configurez le module avec un nom facile à retenir (par exemple, *registre*), et exposez un port dans le module pour recevoir les demandes.
  * Configurez le module pour qu’il corresponde à votre registre de conteneurs.
* Un module proxy d’API
  * Configurez les variables d’environnement suivantes :

    | Nom | Valeur |
    | ---- | ----- |
    | `DOCKER_REQUEST_ROUTE_ADDRESS` | Nom du module registre et port ouvert. Par exemple : `registry:5000`. |
    | `NGINX_DEFAULT_PORT` | Port sur lequel le proxy nginx écoute les demandes des appareils en aval. Par exemple : `8000`. |

  * Configurez les options createOptions suivantes :

    ```json
    {
       "HostConfig": {
          "PortBindings": {
             "8000/tcp": [
                {
                   "HostPort": "8000"
                }
             ]
          }
       }
    }
    ```

Configurez le module suivant sur n’importe quelle **couche inférieure** pour ce scénario :

* Un module proxy d’API
  * Configurez les variables d’environnement suivantes :

    | Nom | Valeur |
    | ---- | ----- |
    | `NGINX_DEFAULT_PORT` | Port sur lequel le proxy nginx écoute les demandes des appareils en aval. Par exemple : `8000`. |

  * Configurez les options createOptions suivantes :

    ```json
    {
       "HostConfig": {
          "PortBindings": {
             "8000/tcp": [
                {
                   "HostPort": "8000"
                }
             ]
          }
       }
    }
    ```  

## <a name="enable-blob-upload"></a>Chargement d’objets blob

Il existe un autre cas d’usage courant du module proxy d’API : permettre aux appareils IoT Edge des couches inférieures de charger des objets blob. Sont ainsi activées les fonctionnalités de résolution des problèmes sur les appareils de couche inférieure, notamment le chargement des journaux de modules ou du Bundle de support.

Ce scénario utilise le module [Stockage Blob Azure sur IoT Edge](https://azuremarketplace.microsoft.com/marketplace/apps/azure-blob-storage.edge-azure-blob-storage) au niveau de la couche supérieure pour gérer la création et le chargement d’objets blob.

Configurez les modules suivants au niveau de la **couche supérieure** :

* Un module Stockage Blob Azure sur IoT Edge
* Un module proxy d’API
  * Configurez les variables d’environnement suivantes :

    | Nom | Valeur |
    | ---- | ----- |
    | `BLOB_UPLOAD_ROUTE_ADDRESS` | Nom du module Stockage Blob et port ouvert. Par exemple : `azureblobstorageoniotedge:1102`. |
    | `NGINX_DEFAULT_PORT` | Port sur lequel le proxy nginx écoute les demandes des appareils en aval. Par exemple : `8000`. |

  * Configurez les options createOptions suivantes :

    ```json
    {
       "HostConfig": {
          "PortBindings": {
             "8000/tcp": [
                {
                   "HostPort": "8000"
                }
             ]
          }
       }
    }
    ```

Configurez le module suivant sur n’importe quelle **couche inférieure** pour ce scénario :

* Un module proxy d’API
  * Configurez les variables d’environnement suivantes :

    | Nom | Valeur |
    | ---- | ----- |
    | `NGINX_DEFAULT_PORT` | Port sur lequel le proxy nginx écoute les demandes des appareils en aval. Par exemple : `8000`. |

  * Configurez les options createOptions suivantes :

    ```json
    {
       "HostConfig": {
          "PortBindings": {
             "8000/tcp": [
                {
                   "HostPort": "8000"
                }
             ]
          }
       }
    }
    ```

Pour charger le Bundle de support ou le fichier journal dans le module Stockage Blob situé dans la couche supérieure, procédez comme suit :

1. Créez un conteneur d’objets blob à l’aide de l’Explorateur Stockage Azure ou des API REST. Pour plus d’informations, consultez [Stockage de données en périphérie avec le Stockage Blob Azure sur IoT Edge](how-to-store-data-blob.md).
1. Demandez le chargement d’un journal ou d’un Bundle de support en suivant la procédure décrite dans [Récupération de journaux à partir de déploiements IoT Edge](how-to-retrieve-iot-edge-logs.md), mais utilisez le nom de domaine `$upstream` et le port de proxy ouvert à la place de l’adresse du module Stockage Blob. Par exemple :

   ```json
   {
      "schemaVersion": "1.0",
      "sasUrl": "https://$upstream:8000/myBlobStorageName/myContainerName?SAS_key",
      "since": "2d",
      "until": "1d",
      "edgeRuntimeOnly": false
   }
   ```

## <a name="edit-the-proxy-configuration"></a>Modification de la configuration du proxy

Un fichier de configuration par défaut est incorporé dans le module proxy d’API. Il est cependant possible de transmettre une nouvelle configuration à ce module via le cloud à l’aide du jumeau de module.

Lorsque vous écrivez votre propre configuration, vous pouvez toujours utiliser l’environnement pour ajuster les paramètres de chaque déploiement. Utilisez la syntaxe suivante :

* Utilisez `${MY_ENVIRONMENT_VARIABLE}` pour récupérer la valeur d’une variable d’environnement.
* Utilisez des instructions conditionnelles pour activer ou désactiver les paramètres en fonction de la valeur d’une variable d’environnement :

   ```conf
   #if_tag ${MY_ENVIRONMENT_VARIABLE}
       statement to execute if environment variable evaluates to 1
   #endif_tag ${MY_ENVIRONMENT_VARIABLE}

   #if_tag !${MY_ENVIRONMENT_VARIABLE}
       statement to execute if environment variable evaluates to 0
   #endif_tag !${MY_ENVIRONMENT_VARIABLE}
   ```

Lorsque le module proxy d’API analyse une configuration de proxy, il commence par remplacer toutes les variables d’environnement listées dans `PROXY_CONFIG_ENV_VAR_LIST` par les valeurs fournies à l’aide de la substitution. Ensuite, tout ce qui se trouve à l’intérieur de la paire `#if_tag` et `#endif_tag` est remplacé. Le module fournit alors la configuration analysée au proxy inverse nginx.

Pour mettre à jour la configuration du proxy de manière dynamique, procédez comme suit :

1. Écrivez votre fichier de configuration. Vous pouvez utiliser ce modèle par défaut comme référence : [nginx_default_config.conf](https://github.com/Azure/iotedge/blob/master/edge-modules/api-proxy-module/templates/nginx_default_config.conf).
1. Copiez le texte du fichier de configuration et convertissez-le en base64.
1. Collez le fichier de configuration encodé dans la valeur de la propriété `proxy_config` souhaitée du jumeau de module.

   ![Copier-coller du fichier de configuration encodé dans la valeur de la propriété proxy_config](./media/how-to-configure-api-proxy-module/change-config.png)

## <a name="next-steps"></a>Étapes suivantes

Utilisez le module proxy d’API pour [connecter un appareil IoT Edge en aval à une passerelle Azure IoT Edge](how-to-connect-downstream-iot-edge-device.md).