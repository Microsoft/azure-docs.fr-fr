---
title: Accéder aux métriques intégrées – Azure IoT Edge
description: Accès à distance aux métriques intégrées à partir des composants du runtime IoT Edge
author: v-tcassi
manager: philmea
ms.author: v-tcassi
ms.date: 10/05/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 7924b06b9056a53fa9861fcd0df516845662b34b
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92341564"
---
# <a name="access-built-in-metrics"></a>Accéder aux métriques intégrées

Les composants du runtime IoT Edge, le hub IoT Edge Hub et l’agent IoT Edge, produisent des métriques intégrées au [format d’exposition Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/). Accédez à ces métriques à distance pour analyser et comprendre l’intégrité d’un appareil IoT Edge.

À partir de la version 1.0.10, les métriques sont automatiquement exposées par défaut sur le **port 9600** des modules **edgeHub** et **edgeAgent** (`http://edgeHub:9600/metrics` et `http://edgeAgent:9600/metrics`). Elles ne sont pas mappées au port de l’hôte par défaut.

Accédez aux métriques à partir de l’hôte en exposant et en mappant le port des métriques à partir du paramètre `createOptions` du module. L’exemple ci-dessous mappe le port des métriques par défaut au port 9601 sur l’hôte :

```
{
  "ExposedPorts": {
    "9600/tcp": {}
  },
  "HostConfig": {
    "PortBindings": {
      "9600/tcp": [
        {
          "HostPort": "9601"
        }
      ]
    }
  }
}
```

Choisissez des numéros de port d’hôte différents et uniques si vous mappez les points de terminaison des métriques d’edgeHub et d’edgeAgent.

> [!NOTE]
> Si vous souhaitez désactiver les métriques, définissez la variable d’environnement `MetricsEnabled` sur `false` pour **edgeAgent**.

## <a name="available-metrics"></a>Métriques disponibles

Les métriques contiennent des balises permettant d’identifier la nature de la métrique collectée. Toutes les métriques contiennent les balises suivantes :

| Tag | Description |
|-|-|
| iothub | Hub avec lequel l’appareil communique |
| edge_device | ID de l’appareil actuel |
| instance_number | GUID représentant le runtime actuel. Au redémarrage, toutes les métriques sont réinitialisées. Ce GUID permet de rapprocher plus facilement les redémarrages |

Dans le format d’exposition Prometheus, il existe quatre types de métriques principales : compteur, jauge, histogramme et résumé. Pour plus d’informations sur les différents types de métriques, consultez la [documentation relative aux types de métriques de Prometheus](https://prometheus.io/docs/concepts/metric_types/).

Les quantiles fournis pour les métriques Histogramme et Résumé intégrées sont 0,1, 0,5, 0,9 et 0,99.

Le module **edgeHub** produit les métriques suivantes :

| Nom | Dimensions | Description |
|-|-|-|
| `edgehub_gettwin_total` | `source` (source de l’opération)<br> `id` (ID de module) | Type : compteur<br> Nombre total d’appels GetTwin |
| `edgehub_messages_received_total` | `route_output` (sortie du message envoyé)<br> `id` | Type : compteur<br> Nombre total de messages reçus des clients |
| `edgehub_messages_sent_total` | `from` (source du message)<br> `to` (destination du message)<br>`from_route_output`<br> `to_route_input` (entrée de destination du message)<br> `priority` (priorité du message à la destination) | Type : compteur<br> Nombre total de messages envoyés aux clients ou en amont<br> `to_route_input` est vide lorsque `to` est $upstream |
| `edgehub_reported_properties_total` | `target`(cible de mise à jour)<br> `id` | Type : compteur<br> Nombre total d’appels de mises à jour de propriétés rapportées |
| `edgehub_message_size_bytes` | `id`<br> | Type : résumé<br> Taille des messages des clients<br> Les valeurs peuvent être rapportées comme `NaN` si aucune nouvelle métrique n’est rapportée pendant un certain laps de temps (actuellement 10 minutes) ; pour le type `summary`, les compteurs `_count` et `_sum` correspondants seront émis. |
| `edgehub_gettwin_duration_seconds` | `source` <br> `id` | Type : résumé<br> Temps nécessaire pour obtenir des opérations de jumeau |
| `edgehub_message_send_duration_seconds` | `from`<br> `to`<br> `from_route_output`<br> `to_route_input` | Type : résumé<br> Temps nécessaire pour envoyer un message |
| `edgehub_message_process_duration_seconds` | `from` <br> `to` <br> `priority` | Type : résumé<br> Temps nécessaire pour traiter un message de la file d’attente |
| `edgehub_reported_properties_update_duration_seconds` | `target`<br> `id` | Type : résumé<br> Temps nécessaire pour mettre à jour les propriétés rapportées |
| `edgehub_direct_method_duration_seconds` | `from` (appelant)<br> `to` (destinataire) | Type : résumé<br> Temps nécessaire pour résoudre un message direct |
| `edgehub_direct_methods_total` | `from`<br> `to` | Type : compteur<br> Nombre total de messages directs envoyés |
| `edgehub_queue_length` | `endpoint` (source du message)<br> `priority` (priorité de file d’attente) | Type : jauge<br> Longueur actuelle de la file d’attente d’edgeHub pour une priorité donnée |
| `edgehub_messages_dropped_total` | `reason` (no_route, ttl_expiry)<br> `from` <br> `from_route_output` | Type : compteur<br> Nombre total de messages supprimés du fait de « reason » |
| `edgehub_messages_unack_total` | `reason` (storage_failure)<br> `from`<br> `from_route_output` | Type : compteur<br> Nombre total de messages sans accusé de réception en raison d’un échec de stockage |
| `edgehub_offline_count_total` | `id` | Type : compteur<br> Nombre total de fois où edgeHub a été mis hors connexion |
| `edgehub_offline_duration_seconds`| `id` | Type : résumé<br> Durée pendant laquelle edgeHub était hors connexion |
| `edgehub_operation_retry_total` | `id`<br> `operation` (nom de l’opération) | Type : compteur<br> Nombre total de fois où les opérations edgeHub ont fait l’objet d’une nouvelle tentative |
| `edgehub_client_connect_failed_total` | `id` <br> `reason` (non authentifié)<br> | Type : compteur<br> Nombre total de fois où les clients n’ont pas réussi à se connecter à edgeHub |

Le module **edgeAgent** produit les métriques suivantes :

| Nom | Dimensions | Description |
|-|-|-|
| `edgeAgent_total_time_running_correctly_seconds` | `module_name` | Type : jauge<br> Durée pendant laquelle le module était spécifié dans le déploiement et en état de fonctionnement |
| `edgeAgent_total_time_expected_running_seconds` | `module_name` | Type : jauge<br> Durée pendant laquelle le module était spécifié dans le déploiement |
| `edgeAgent_module_start_total` | `module_name`, `module_version` | Type : compteur<br> Nombre de fois où edgeAgent a demandé à Docker de démarrer le module |
| `edgeAgent_module_stop_total` | `module_name`, `module_version` | Type : compteur<br> Nombre de fois où edgeAgent a demandé à Docker d’arrêter le module |
| `edgeAgent_command_latency_seconds` | `command` | Type : jauge<br> Temps qu’il a fallu à Docker pour exécuter la commande donnée. Les commandes possibles sont les suivantes : create, update, remove, start, stop, restart |
| `edgeAgent_iothub_syncs_total` | | Type : compteur<br> Nombre de fois où edgeAgent a tenté de synchroniser son jumeau avec IoTHub, avec ou sans succès. Ce nombre comprend l’agent demandant un jumeau et le hub informant d’une mise à jour de jumeau |
| `edgeAgent_unsuccessful_iothub_syncs_total` | | Type : compteur<br> Nombre de fois où edgeAgent n’a pas réussi à synchroniser son jumeau avec IoTHub |
| `edgeAgent_deployment_time_seconds` | | Type : compteur<br> Temps qu’il a fallu pour terminer un nouveau déploiement après avoir reçu une modification |
| `edgeagent_direct_method_invocations_count` | `method_name` | Type : compteur<br> Nombre de fois où une méthode directe edgeAgent intégrée est appelée, par exemple Ping ou Restart. |
| `edgeAgent_host_uptime_seconds` | | Type : jauge<br> Durée depuis laquelle l’hôte est allumé |
| `edgeAgent_iotedged_uptime_seconds` | | Type : jauge<br> Durée depuis laquelle iotedged est en cours d’exécution |
| `edgeAgent_available_disk_space_bytes` | `disk_name`, `disk_filesystem`, `disk_filetype` | Type : jauge<br> Quantité d’espace restant sur le disque |
| `edgeAgent_total_disk_space_bytes` | `disk_name`, `disk_filesystem`, `disk_filetype`| Type : jauge<br> Taille du disque |
| `edgeAgent_used_memory_bytes` | `module_name` | Type : jauge<br> Quantité de RAM utilisée par tous les processus |
| `edgeAgent_total_memory_bytes` | `module_name` | Type : jauge<br> RAM disponible |
| `edgeAgent_used_cpu_percent` | `module_name` | Type : histogramme<br> Pourcentage d’UC utilisé par tous les processus |
| `edgeAgent_created_pids_total` | `module_name` | Type : jauge<br> Nombre de processus ou de threads créés par le conteneur |
| `edgeAgent_total_network_in_bytes` | `module_name` | Type : jauge<br> Nombre d’octets reçus du réseau |
| `edgeAgent_total_network_out_bytes` | `module_name` | Type : jauge<br> Nombre d’octets envoyés au réseau |
| `edgeAgent_total_disk_read_bytes` | `module_name` | Type : jauge<br> Nombre d’octets lus à partir du disque |
| `edgeAgent_total_disk_write_bytes` | `module_name` | Type : jauge<br> Nombre d’octets écrits sur le disque |
| `edgeAgent_metadata` | `edge_agent_version`, `experimental_features`, `host_information` | Type : jauge<br> Métadonnées générales relatives à l’appareil. La valeur est toujours 0, les informations sont encodées dans les balises. Notez que `experimental_features` et `host_information` sont des objets JSON. `host_information` ressemble à ```{"OperatingSystemType": "linux", "Architecture": "x86_64", "Version": "1.0.10~dev20200803.4", "Provisioning": {"Type": "dps.tpm", "DynamicReprovisioning": false, "AlwaysReprovisionOnStartup": true}, "ServerVersion": "19.03.6", "KernelVersion": "5.0.0-25-generic", "OperatingSystem": "Ubuntu 18.04.4 LTS", "NumCpus": 6, "Virtualized": "yes"}```. Notez que `ServerVersion` désigne la version de Docker et `Version` désigne la version du démon de sécurité IoT Edge. |

## <a name="next-steps"></a>Étapes suivantes

* [Présentation du runtime Azure IoT Edge et de son architecture](iot-edge-runtime.md)
* [Propriétés des jumeaux de module de l’agent IoT Edge et du hub IoT Edge](module-edgeagent-edgehub.md)
