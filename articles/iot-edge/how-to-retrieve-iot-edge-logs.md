---
title: Récupérer les journaux IoT Edge – Azure IoT Edge
description: Le module IoT Edge permet de récupérer les journaux et de les charger dans Stockage Blob Azure.
author: v-tcassi
manager: philmea
ms.author: v-tcassi
ms.date: 11/12/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
ms.custom: devx-track-azurecli
services: iot-edge
ms.openlocfilehash: abd30c22aa2b4df20cdb795013768cd175cfef4c
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780737"
---
# <a name="retrieve-logs-from-iot-edge-deployments"></a>Récupérer les journaux des déploiements IoT Edge

Récupérez les journaux de vos déploiements IoT Edge sans avoir besoin d’un accès physique ou SSH à l’appareil à l’aide des méthodes directes incluses dans le module de l’agent IoT Edge. Les méthodes directes sont implémentées sur l’appareil, puis peuvent être appelées à partir du cloud. L’agent IoT Edge comprend des méthodes directes qui vous aident à surveiller et gérer vos appareils IoT Edge à distance. Les méthodes directes présentées dans cet article sont généralement disponibles avec la version 1.0.10.

Pour plus d’informations sur les méthodes directes, leur utilisation et la façon de les implémenter dans vos propres modules, consultez [Comprendre et appeler des méthodes directes à partir d’IoT Hub](../iot-hub/iot-hub-devguide-direct-methods.md).

Les noms de ces méthodes directes respectent la casse.

## <a name="recommended-logging-format"></a>Format de journalisation recommandé

Bien qu’il ne soit pas obligatoire, pour une meilleure compatibilité avec cette fonction, le format de journalisation recommandé est le suivant :

```
<{Log Level}> {Timestamp} {Message Text}
```

`{Log Level}` doit respecter le [format Niveau de gravité Syslog](https://wikipedia.org/wiki/Syslog#Severity_level) et `{Timestamp}` doit être formaté comme `yyyy-MM-dd hh:mm:ss.fff zzz`.

La [classe de journalisation dans IoT Edge](https://github.com/Azure/iotedge/blob/master/edge-util/src/Microsoft.Azure.Devices.Edge.Util/Logger.cs) sert d’implémentation canonique.

## <a name="retrieve-module-logs"></a>Récupérer les journaux de module

Utilisez la méthode directe **GetModuleLogs** pour récupérer les journaux d’un module IoT Edge.

Cette méthode accepte une charge utile JSON avec le schéma suivant :

```json
    {
       "schemaVersion": "1.0",
       "items": [
          {
             "id": "regex string",
             "filter": {
                "tail": "int",
                "since": "int",
                "until": "int",
                "loglevel": "int",
                "regex": "regex string"
             }
          }
       ],
       "encoding": "gzip/none",
       "contentType": "json/text" 
    }
```

| Name | Type | Description |
|-|-|-|
| schemaVersion | string | Paramètre à définir sur `1.0` |
| items | Tableau JSON | Tableau avec les tuples `id` et `filter`. |
| id | string | Expression régulière qui fournit le nom du module. Il peut correspondre à plusieurs modules sur un périphérique. Le format [Expressions régulières .NET](/dotnet/standard/base-types/regular-expressions) est attendu. |
| filter | Section JSON | Filtres de journal à appliquer aux modules correspondant à l’expression régulière `id` dans le tuple. |
| tail | entier | Nombre de lignes de journal dans le passé à récupérer à partir de la dernière. FACULTATIF. |
| since | entier | Retourne uniquement les journaux depuis cette heure, sous la forme d’une durée (1 j, 90 min, 2 jours 3 heures 2 minutes), d’un timestamp rfc3339 ou d’un timestamp UNIX.  Si `tail` et `since` sont spécifiés, les journaux sont d’abord récupérés à l’aide de la valeur `since`. Ensuite, la valeur `tail` est appliquée au résultat, et le résultat final est retourné. FACULTATIF. |
| until | entier | Retourne uniquement les journaux avant l’heure spécifiée, sous la forme d’un timestamp rfc3339, d’un timestamp UNIX ou d’une durée (1 j, 90 min, 2 jours 3 heures 2 minutes). FACULTATIF. |
| log level | entier | Filtre les lignes de journal inférieures ou égales au niveau de journal spécifié. Les lignes de journal doivent suivre le format de journalisation recommandé et utiliser la norme [Niveau de gravité Syslog](https://en.wikipedia.org/wiki/Syslog#Severity_level). FACULTATIF. |
| regex | string | Filtre les lignes de journal dont le contenu correspond à l’expression régulière spécifiée à l’aide du format [Expressions régulières .NET](/dotnet/standard/base-types/regular-expressions). FACULTATIF. |
| encodage | string | `gzip` ou `none`. La valeur par défaut est `none`. |
| contentType | string | `json` ou `text`. La valeur par défaut est `text`. |

> [!NOTE]
> Si le contenu des journaux dépasse la limite de taille de réponse des méthodes directes, qui est actuellement de 128 ko, la réponse retourne une erreur.

Une récupération réussie des journaux retourne un message **« état » : 200** suivi d’une charge utile contenant les journaux récupérés dans le module, filtrés par les paramètres que vous spécifiez dans votre demande.

Par exemple :

```azurecli
az iot hub invoke-module-method --method-name 'GetModuleLogs' -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
       "schemaVersion": "1.0",
       "items": [
          {
             "id": "edgeAgent",
             "filter": {
                "tail": 10
             }
          }
       ],
       "encoding": "none",
       "contentType": "text"
    }
'
```

Dans le portail Azure, appelez la méthode avec le nom de la méthode `GetModuleLogs` et la charge utile JSON suivante :

```json
    {
       "schemaVersion": "1.0",
       "items": [
          {
             "id": "edgeAgent",
             "filter": {
                "tail": 10
             }
          }
       ],
       "encoding": "none",
       "contentType": "text"
    }
```

![Appeler la méthode directe « GetModuleLogs » dans le portail Azure](./media/how-to-retrieve-iot-edge-logs/invoke-get-module-logs.png)

Vous pouvez également canaliser la sortie CLI vers des utilitaires Linux, comme [gzip](https://en.wikipedia.org/wiki/Gzip), pour traiter une réponse compressée. Par exemple :

```azurecli
az iot hub invoke-module-method \
  --method-name 'GetModuleLogs' \
  -n <hub name> \
  -d <device id> \
  -m '$edgeAgent' \
  --method-payload '{"contentType": "text","schemaVersion": "1.0","encoding": "gzip","items": [{"id": "edgeHub","filter": {"since": "2d","tail": 1000}}],}' \
  -o tsv --query 'payload[0].payloadBytes' \
  | base64 --decode \
  | gzip -d
```

## <a name="upload-module-logs"></a>Charger les journaux de module

Utilisez la méthode directe **UploadModuleLogs** pour envoyer les journaux demandés à un conteneur Stockage Blob Azure spécifié.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"

> [!NOTE]
> Pour charger des journaux à partir d’un appareil situé derrière un appareil de passerelle, vous devez configurer les [modules de proxy d’API et de stockage d’objets blob](how-to-configure-api-proxy-module.md) sur l’appareil de couche supérieure. Ces modules acheminent les journaux entre votre appareil de couche inférieure et votre stockage cloud via votre appareil de passerelle.

::: moniker-end

Cette méthode accepte une charge utile JSON similaire à **GetModuleLogs**, avec l’ajout de la clé « sasUrl » :

```json
    {
       "schemaVersion": "1.0",
       "sasUrl": "Full path to SAS URL",
       "items": [
          {
             "id": "regex string",
             "filter": {
                "tail": "int",
                "since": "int",
                "until": "int",
                "loglevel": "int",
                "regex": "regex string"
             }
          }
       ],
       "encoding": "gzip/none",
       "contentType": "json/text" 
    }
```

| Nom | Type | Description |
|-|-|-|
| sasURL | chaîne (URI) | [URL de signature d’accès partagé avec accès en écriture au conteneur Stockage Blob Azure](/archive/blogs/jpsanders/easily-create-a-sas-to-download-a-file-from-azure-storage-using-azure-storage-explorer). |

Une demande réussie de chargement des journaux retourne un message **« État » : 200** suivi d’une charge utile avec le schéma suivant :

```json
    {
        "status": "string",
        "message": "string",
        "correlationId": "GUID"
    }
```

| Nom | Type | Description |
|-|-|-|
| status | string | `NotStarted`, `Running`, `Completed`, `Failed` ou `Unknown`. |
| message | string | Message en cas d’erreur, chaîne vide dans le cas contraire. |
| correlationId | string   | ID à interroger pour obtenir l’état de la demande de chargement. |

Par exemple :

L’appel suivant charge les 100 dernières lignes de journal de tous les modules, au format JSON compressé :

```azurecli
az iot hub invoke-module-method --method-name UploadModuleLogs -n <hub name> -d <device id> -m \$edgeAgent --method-payload \
'
    {
        "schemaVersion": "1.0",
        "sasUrl": "<sasUrl>",
        "items": [
            {
                "id": ".*",
                "filter": {
                    "tail": 100
                }
            }
        ],
        "encoding": "gzip",
        "contentType": "json"
    }
'
```

L’appel suivant charge les 100 dernières lignes de journal d’edgeAgent et edgeHub avec les 1 000 dernières lignes de journal du module tempSensor dans un format de texte non compressé :

```azurecli
az iot hub invoke-module-method --method-name UploadModuleLogs -n <hub name> -d <device id> -m \$edgeAgent --method-payload \
'
    {
        "schemaVersion": "1.0",
        "sasUrl": "<sasUrl>",
        "items": [
            {
                "id": "edge",
                "filter": {
                    "tail": 100
                }
            },
            {
                "id": "tempSensor",
                "filter": {
                    "tail": 1000
                }
            }
        ],
        "encoding": "none",
        "contentType": "text"
    }
'
```

Dans le portail Azure, appelez la méthode avec le nom de la méthode `UploadModuleLogs` et la charge utile JSON suivante après avoir rempli la clé sasURL avec vos informations :

```json
    {
       "schemaVersion": "1.0",
       "sasUrl": "<sasUrl>",
       "items": [
          {
             "id": "edgeAgent",
             "filter": {
                "tail": 10
             }
          }
       ],
       "encoding": "none",
       "contentType": "text"
    }
```

![Appeler la méthode directe « UploadModuleLogs » dans le portail Azure](./media/how-to-retrieve-iot-edge-logs/invoke-upload-module-logs.png)

## <a name="upload-support-bundle-diagnostics"></a>Charger les diagnostics du pack de support

Utilisez la méthode directe **UploadSupportBundle** pour regrouper et charger un fichier zip des journaux du module IoT Edge dans un conteneur Stockage Blob Azure disponible. Cette méthode directe exécute la commande [`iotedge support-bundle`](./troubleshoot.md#gather-debug-information-with-support-bundle-command) sur votre appareil IoT Edge pour récupérer les journaux.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"

> [!NOTE]
> Pour charger des journaux à partir d’un appareil situé derrière un appareil de passerelle, vous devez configurer les [modules de proxy d’API et de stockage d’objets blob](how-to-configure-api-proxy-module.md) sur l’appareil de couche supérieure. Ces modules acheminent les journaux entre votre appareil de couche inférieure et votre stockage cloud via votre appareil de passerelle.

::: moniker-end

Cette méthode accepte une charge utile JSON avec le schéma suivant :

```json
    {
        "schemaVersion": "1.0",
        "sasUrl": "Full path to SAS url",
        "since": "2d",
        "until": "1d",
        "edgeRuntimeOnly": false
    }
```

| Nom | Type | Description |
|-|-|-|
| schemaVersion | string | Paramètre à définir sur `1.0` |
| sasURL | chaîne (URI) | [URL de signature d’accès partagé avec accès en écriture au conteneur Stockage Blob Azure](/archive/blogs/jpsanders/easily-create-a-sas-to-download-a-file-from-azure-storage-using-azure-storage-explorer). |
| since | entier | Retourne uniquement les journaux depuis cette heure, sous la forme d’une durée (1 j, 90 min, 2 jours 3 heures 2 minutes), d’un timestamp rfc3339 ou d’un timestamp UNIX. FACULTATIF. |
| until | entier | Retourne uniquement les journaux avant l’heure spécifiée, sous la forme d’un timestamp rfc3339, d’un timestamp UNIX ou d’une durée (1 j, 90 min, 2 jours 3 heures 2 minutes). FACULTATIF. |
| edgeRuntimeOnly | boolean | Si la valeur est true, retourne uniquement les journaux de l’agent Edge, du hub Edge et du démon de sécurité Edge. Valeur par défaut : false.  FACULTATIF. |

> [!IMPORTANT]
> Le pack de support IoT Edge peut contenir des informations d’identification personnelle.

Une demande réussie de chargement des journaux retourne un message **« État » : 200** suivi d’une charge utile avec le même schéma que la réponse **UploadModuleLogs** :

```json
    {
        "status": "string",
        "message": "string",
        "correlationId": "GUID"
    }
```

| Nom | Type | Description |
|-|-|-|
| status | string | `NotStarted`, `Running`, `Completed`, `Failed` ou `Unknown`. |
| message | string | Message en cas d’erreur, chaîne vide dans le cas contraire. |
| correlationId | string   | ID à interroger pour obtenir l’état de la demande de chargement. |

Par exemple :

```azurecli
az iot hub invoke-module-method --method-name 'UploadSupportBundle' -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
        "schemaVersion": "1.0",
        "sasUrl": "Full path to SAS url",
        "since": "2d",
        "until": "1d",
        "edgeRuntimeOnly": false
    }
'
```

Dans le portail Azure, appelez la méthode avec le nom de la méthode `UploadSupportBundle` et la charge utile JSON suivante après avoir rempli la clé sasURL avec vos informations :

```json
    {
        "schemaVersion": "1.0",
        "sasUrl": "Full path to SAS url",
        "since": "2d",
        "until": "1d",
        "edgeRuntimeOnly": false
    }
```

![Appeler la méthode directe « UploadSupportBundle » dans le portail Azure](./media/how-to-retrieve-iot-edge-logs/invoke-upload-support-bundle.png)

## <a name="get-upload-request-status"></a>Récupérer l’état de la demande de chargement

Utilisez la méthode directe **GetTaskStatus** pour interroger l’état d’une demande de chargement des journaux. La charge utile de la demande **GetTaskStatus** utilise le paramètre `correlationId` de la demande de chargement des journaux pour récupérer l’état de la tâche. Le `correlationId` est retourné en réponse à l’appel de la méthode directe **UploadModuleLogs**.

Cette méthode accepte une charge utile JSON avec le schéma suivant :

```json
    {
      "schemaVersion": "1.0",
      "correlationId": "<GUID>"
    }
```

Une demande réussie de chargement des journaux retourne un message **« État » : 200** suivi d’une charge utile avec le même schéma que la réponse **UploadModuleLogs** :

```json
    {
        "status": "string",
        "message": "string",
        "correlationId": "GUID"
    }
```

| Nom | Type | Description |
|-|-|-|
| status | string | `NotStarted`, `Running`, `Completed`, `Failed` ou `Unknown`. |
| message | string | Message en cas d’erreur, chaîne vide dans le cas contraire. |
| correlationId | string   | ID à interroger pour obtenir l’état de la demande de chargement. |

Par exemple :

```azurecli
az iot hub invoke-module-method --method-name 'GetTaskStatus' -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
      "schemaVersion": "1.0",
      "correlationId": "<GUID>"
    }
'
```

Dans le portail Azure, appelez la méthode avec le nom de la méthode `GetTaskStatus` et la charge utile JSON suivante après avoir rempli le GUID avec vos informations :

```json
    {
      "schemaVersion": "1.0",
      "correlationId": "<GUID>"
    }
```

![Appeler la méthode directe « GetTaskStatus » dans le portail Azure](./media/how-to-retrieve-iot-edge-logs/invoke-get-task-status.png)

## <a name="next-steps"></a>Étapes suivantes

[Propriétés des jumeaux de module de l’agent IoT Edge et du hub IoT Edge](module-edgeagent-edgehub.md)
