---
title: Aperçu des journaux et des métriques du pare-feu Azure
description: Vous pouvez surveiller le service Pare-feu Azure à l’aide des journaux d’activité de pare-feu. Vous pouvez également utiliser les journaux d’activité pour auditer les opérations sur les ressources de Pare-feu Azure.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 09/10/2020
ms.author: victorh
ms.openlocfilehash: 69890e2d846a63a70c1b7459b1df13ce5e891289
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659469"
---
# <a name="azure-firewall-logs-and-metrics"></a>Journaux et métriques du pare-feu Azure

Vous pouvez surveiller le service Pare-feu Azure à l’aide des journaux d’activité de pare-feu. Vous pouvez également utiliser les journaux d’activité pour auditer les opérations sur les ressources de Pare-feu Azure.

Vous pouvez accéder à certains de ces journaux d’activité via le portail. Les journaux d’activité peuvent être envoyés au service [Journaux d’activité Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md), au stockage et aux hubs d’événements, puis analysés dans les journaux d’activité Azure Monitor ou par différents outils comme Excel et Power BI.

Les métriques sont légères et peuvent prendre en charge des scénarios en temps quasi réel, ce qui les rend utiles pour les alertes et la détection rapide de problèmes.

## <a name="diagnostic-logs"></a>Journaux de diagnostic

 Les journaux de diagnostic suivants sont disponibles pour Pare-feu Azure :

* **Journal de règles d’application**

   Le journal de règles d’application est enregistré dans un compte de stockage, transmis en continu au service Event Hubs et/ou envoyé vers les journaux Azure Monitor uniquement si vous l’avez activé pour chaque Pare-feu Azure. Chaque nouvelle connexion qui correspond à l’une de vos règles d’application configurées entraîne un journal pour la connexion acceptée/refusée. Les données sont consignées au format JSON, comme indiqué dans l’exemple suivant :

   ```
   Category: application rule logs.
   Time: log timestamp.
   Properties: currently contains the full message.
   note: this field will be parsed to specific fields in the future, while maintaining backward compatibility with the existing properties field.
   ```

   ```json
   {
    "category": "AzureFirewallApplicationRule",
    "time": "2018-04-16T23:45:04.8295030Z",
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
    "operationName": "AzureFirewallApplicationRuleLog",
    "properties": {
        "msg": "HTTPS request from 10.1.0.5:55640 to mydestination.com:443. Action: Allow. Rule Collection: collection1000. Rule: rule1002"
    }
   }
   ```

* **Journal de règles de réseau**

   Le journal de règles de réseau est enregistré dans un compte de stockage, transmis en continu au service Event Hubs et/ou envoyé vers les journaux Azure Monitor uniquement si vous l’avez activé pour chaque Pare-feu Azure. Chaque nouvelle connexion qui correspond à l’une de vos règles de réseau configurées entraîne un journal pour la connexion acceptée/refusée. Les données sont consignées au format JSON, comme indiqué dans l’exemple suivant :

   ```
   Category: network rule logs.
   Time: log timestamp.
   Properties: currently contains the full message.
   note: this field will be parsed to specific fields in the future, while maintaining backward compatibility with the existing properties field.
   ```

   ```json
  {
    "category": "AzureFirewallNetworkRule",
    "time": "2018-06-14T23:44:11.0590400Z",
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
    "operationName": "AzureFirewallNetworkRuleLog",
    "properties": {
        "msg": "TCP request from 111.35.136.173:12518 to 13.78.143.217:2323. Action: Deny"
    }
   }

   ```

* **Journal du proxy DNS**

   Le journal du proxy DNS est enregistré dans un compte de stockage, transmis en continu au service Event Hubs et/ou envoyé vers les journaux Azure Monitor uniquement si vous l’avez activé pour chaque pare-feu Azure. Ce journal effectue le suivi des messages DNS envoyés à un serveur DNS configuré à l’aide du proxy DNS. Les données sont journalisées au format JSON, comme indiqué dans les exemples suivants :


   ```
   Category: DNS proxy logs.
   Time: log timestamp.
   Properties: currently contains the full message.
   note: this field will be parsed to specific fields in the future, while maintaining backward compatibility with the existing properties field.
   ```

   Réussite :
   ```json
   {
     "category": "AzureFirewallDnsProxy",
     "time": "2020-09-02T19:12:33.751Z",
     "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
     "operationName": "AzureFirewallDnsProxyLog",
     "properties": {
         "msg": "DNS Request: 11.5.0.7:48197 – 15676 AAA IN md-l1l1pg5lcmkq.blob.core.windows.net. udp 55 false 512 NOERROR - 0 2.000301956s"
     }
   }
   ```

   Échec :

   ```json
   {
     "category": "AzureFirewallDnsProxy",
     "time": "2020-09-02T19:12:33.751Z",
     "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
     "operationName": "AzureFirewallDnsProxyLog",
     "properties": {
         "msg": " Error: 2 time.windows.com.reddog.microsoft.com. A: read udp 10.0.1.5:49126->168.63.129.160:53: i/o timeout”
     }
   }
   ```

   Format du message :

   `[client’s IP address]:[client’s port] – [query ID] [type of the request] [class of the request] [name of the request] [protocol used] [request size in bytes] [EDNS0 DO (DNSSEC OK) bit set in the query] [EDNS0 buffer size advertised in the query] [response CODE] [response flags] [response size] [response duration]`

Pour stocker vos journaux d’activité, vous disposez de trois options :

* **Compte de stockage** : les comptes de stockage conviennent parfaitement aux journaux d’activité quand ils sont stockés pour une durée plus longue et consultés quand cela est nécessaire.
* **Hubs d’événements** : les hubs d’événements constituent une excellente solution pour l’intégration à d’autres outils SEIM (Security Information and Event Management) afin de recevoir des alertes sur vos ressources.
* **Journaux d’activité Azure Monitor** : Les journaux d’activité Azure Monitor conviennent parfaitement pour la supervision en temps réel générale de votre application ou la recherche de tendances.

## <a name="activity-logs"></a>Journaux d’activité

   Les entrées du journal d’activité sont recueillies par défaut et vous pouvez les afficher dans le Portail Azure.

   Vous pouvez utiliser les [journaux d’activité Azure](../azure-resource-manager/management/view-activity-logs.md) (anciennement journaux d’activité des opérations et journaux d’audit) pour afficher toutes les opérations soumises à votre abonnement Azure.

## <a name="metrics"></a>Mesures

Les métriques dans Azure Monitor sont des valeurs numériques décrivant certains aspects d’un système à un moment donné. Les métriques sont collectées toutes les minutes et sont utiles pour les alertes, car elles peuvent être échantillonnées fréquemment. Une alerte peut être déclenchée rapidement avec une logique relativement simple.

Les métriques suivantes sont disponibles pour le pare-feu Azure :

- **Nombre d’accès aux règles d’application** : nombre de fois où une règle d’application a été respectée.

    Nombre d’unité : nombre

- **Nombre d’accès aux réseau** : nombre de fois où une règle réseau a été respectée.

    Nombre d’unité : nombre

- **Données traitées** : somme des données qui traversent le pare-feu dans une fenêtre temporelle donnée.

    Unité : octets

- **Débit** : nombre de données traversant le pare-feu par seconde.

    Unité : bits par seconde

- **État d'intégrité du pare-feu** : indique l'intégrité du pare-feu en fonction de la disponibilité du port SNAT.

    Unité : pourcentage

   Cette métrique a deux dimensions :
  - État : les valeurs possibles sont *Sain*, *Détérioré*, *Non sain*.
  - Motif : indique la raison de l’état correspondant du pare-feu. 

     Si les ports SNAT sont utilisés à plus de 95 %, ils sont considérés comme épuisés et l'intégrité affichée est à 50 % avec : état=**Détérioré** et motif=**port SNAT**. Le pare-feu continue de traiter le trafic et les connexions existantes ne sont pas affectées. Toutefois, les nouvelles connexions peuvent ne pas être établies par intermittence.

     Si les ports SNAT sont utilisés à moins de 95 %, le pare-feu est considéré comme sain et l'intégrité est affichée à 100 %.

     Si aucune utilisation des ports SNAT n'est signalée, l'intégrité est affichée à 0 %. 

- **Utilisation des ports SNAT** : pourcentage de ports SNAT qui ont été utilisés par le pare-feu.

    Unité : pourcentage

   Lorsque vous ajoutez d’autres adresses IP publiques à votre pare-feu, davantage de ports SNAT sont disponibles, ce qui réduit l’utilisation des ports SNAT. De plus, lorsque le pare-feu est mis à l’échelle pour différentes raisons (par exemple, UC ou débit), des ports SNAT supplémentaires sont également rendus disponibles. De fait, un pourcentage donné de l’utilisation des ports SNAT peut baisser sans que vous ajoutiez d’adresses IP publiques, juste parce que le service est mis à l’échelle. Vous pouvez contrôler directement le nombre d’adresses IP publiques disponibles pour augmenter les ports disponibles sur votre pare-feu. Toutefois, vous ne pouvez pas contrôler directement la mise à l’échelle du pare-feu.


## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment superviser les métriques et les journaux de Pare-feu Azure, consultez le [Tutoriel : Surveiller les journaux d’activité de pare-feu Azure](./firewall-diagnostics.md).

- Pour en savoir plus sur les métriques dans Azure Monitor, consultez [Métriques dans Azure Monitor](../azure-monitor/platform/data-platform-metrics.md).