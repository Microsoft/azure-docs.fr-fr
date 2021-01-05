---
title: Résoudre les problèmes de connectivité - Azure Event Grid | Microsoft Docs
description: Cet article fournit des informations sur la résolution des problèmes de connectivité réseau avec Azure Event Grid.
author: batrived
ms.topic: article
ms.date: 06/21/2020
ms.author: batrived
ms.openlocfilehash: fa119784715b8c88ef3c9f2700b2cac1cc467234
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96339743"
---
# <a name="troubleshoot-connectivity-issues---azure-event-grid"></a>Résoudre les problèmes de connectivité - Azure Event Grid

Il existe plusieurs raisons pour lesquelles des applications clientes peuvent ne pas parvenir à se connecter à une rubrique/un domaine Event Grid. Les problèmes de connectivité que vous rencontrez peuvent être permanents ou transitoires. Si le problème se produit tout le temps (permanent), vous souhaiterez peut-être vérifier les paramètres de pare-feu de votre organisation, les paramètres de pare-feu IP, les balises du service, les points de terminaison privés, entre autres. Pour les problèmes temporaires, l’exécution de commandes pour vérifier les paquets supprimés et l’obtention de traces réseau peuvent vous aider à résoudre les problèmes.

Cet article fournit des conseils sur la résolution des problèmes de connectivité avec Azure Event Grid.

## <a name="troubleshoot-permanent-connectivity-issues"></a>Résoudre les problèmes de connectivité permanents

Si l’application n’est pas du tout en mesure de se connecter à Event Grid, suivez les étapes de cette section pour résoudre le problème.

### <a name="check-if-theres-a-service-outage"></a>Vérifier s’il y a une panne de service

Vérifiez s’il y a une panne du service Azure Event Grid sur le [Site d’état du service Azure](https://azure.microsoft.com/status/).

### <a name="check-if-the-ports-required-to-communicate-with-event-grid-arent-blocked-by-organizations-firewall"></a>Vérifier que les ports requis pour communiquer avec Event Grid ne sont pas bloqués par le pare-feu de l’organisation

Vérifiez que les ports utilisés pour communiquer avec Azure Event Grid ne sont pas bloqués sur le pare-feu de votre organisation. Consultez le tableau suivant pour savoir quels ports de sortie vous devez ouvrir dans le but de communiquer avec Azure Event Grid.

| Protocol | Ports |
| -------- | ----- |
| HTTPS    | 443   |

Voici un exemple de commande qui permet de vérifier si le port 443 est bloqué.

```powershell
.\psping.exe -n 25 -i 1 -q {sampletopicname}.{region}-{suffix}.eventgrid.azure.net:443 -nobanner
```

Sur Linux :

```shell
telnet {sampletopicname}.{region}-{suffix}.eventgrid.azure.net 443
```

### <a name="verify-that-ip-addresses-are-allowed-in-your-corporate-firewall"></a>Vérifier que les adresses IP sont autorisées dans votre pare-feu d’entreprise

Lorsque vous utilisez Azure, vous devez parfois autoriser des plages d’adresses IP ou des URL spécifiques dans votre pare-feu ou proxy d’entreprise pour accéder à tous les services Azure que vous utilisez ou essayez d’utiliser. Vérifiez que le trafic est autorisé sur les adresses IP utilisées par Event Grid. Pour les adresses IP utilisées par Azure Event Grid : consultez [Plages d’adresses IP Azure et balises de service - Cloud public](https://www.microsoft.com/download/details.aspx?id=56519) et [Balise de service - AzureEventGrid](network-security.md#service-tags).

Le document [Plages d’adresses IP Azure et balises de service](https://www.microsoft.com/download/details.aspx?id=56519) répertorie également les adresses IP **par région**. Vous pouvez autoriser des plages d’adresses pour la **région de la rubrique** et la **région jumelée** dans votre pare-feu ou votre proxy d’entreprise. Pour la région jumelée d’une région, consultez [Continuité des activités et récupération d’urgence (BCDR) : régions jumelées d’Azure](../best-practices-availability-paired-regions.md). 

> [!NOTE]
> De nouvelles adresses IP peuvent être ajoutées à la balise de service AzureEventGrid, bien qu’il ne s’agisse pas d’une habitude. Il est donc préférable d’effectuer une vérification hebdomadaire sur les balises de service.

### <a name="verify-that-azureeventgrid-service-tag-is-allowed-in-your-network-security-groups"></a>Vérifier que la balise de service AzureEventGrid est autorisée dans vos groupes de sécurité réseau

Si votre application s’exécute à l’intérieur d’un sous-réseau et en présence d’un groupe de sécurité réseau associé, vérifiez si l’un des services sortants Internet est autorisé ou si la balise de service AzureEventGrid est autorisée. Voir [Étiquettes de service](../virtual-network/service-tags-overview.md)

### <a name="check-the-ip-firewall-settings-for-your-topicdomain"></a>Vérifier les paramètres de pare-feu IP pour votre Rubrique/Domaine

Vérifiez que l’adresse IP publique de la machine sur laquelle l’application est en cours d’exécution n’est pas bloquée par le pare-feu IP de la rubrique/du domaine EventGrid.

Par défaut, les rubriques/domaines Event Grid sont accessibles sur Internet tant que la requête s’accompagne d’une authentification et d’une autorisation valides. Avec le pare-feu IP, vous pouvez les limiter à un ensemble d’adresses IPv4 ou de plages d’adresses IPv4 dans la notation [CIDR (Classless InterDomain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Les règles de pare-feu IP sont appliquées au niveau de la rubrique/du domaine Event Grid. Par conséquent, les règles s’appliquent à toutes les connexions de clients utilisant un protocole pris en charge. Toute tentative de connexion à partir d’une adresse IP qui ne correspond pas à une règle IP autorisée dans la rubrique/le domaine Event Grid est rejetée comme étant non autorisée. La réponse ne mentionne pas la règle IP.

Pour plus d'informations, consultez [Configurer des règles de pare-feu IP pour une rubrique/un domaine Azure Event Grid](configure-firewall.md).

#### <a name="find-the-ip-addresses-blocked-by-ip-firewall"></a>Rechercher les adresses IP bloquées par le pare-feu IP

Activez les journaux de diagnostic pour la rubrique/le domaine Event Grid [Activer les journaux de diagnostic](enable-diagnostic-logs-topic.md#enable-diagnostic-logs-for-a-custom-topic). Vous verrez l’adresse IP de la connexion qui est refusée.

```json
{
  "time": "2019-11-01T00:17:13.4389048Z",
  "resourceId": "/SUBSCRIPTIONS/SAMPLE-SUBSCTIPTION-ID/RESOURCEGROUPS/SAMPLE-RESOURCEGROUP-NAME/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/SAMPLE-TOPIC-NAME",
  "category": "PublishFailures",
  "operationName": "Post",
  "message": "inputEventsCount=null, requestUri=https://SAMPLE-TOPIC-NAME.region-suffix.eventgrid.azure.net/api/events, publisherInfo=PublisherInfo(category=User, inputSchema=EventGridEvent, armResourceId=/SUBSCRIPTIONS/SAMPLE-SUBSCTIPTION-ID/RESOURCEGROUPS/SAMPLE-RESOURCEGROUP-NAME/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/SAMPLE-TOPIC-NAME), httpStatusCode=Forbidden, errorType=ClientIPRejected, errorMessage=Publishing to SAMPLE-TOPIC-NAME.{region}-{suffix}.EVENTGRID.AZURE.NET by client {clientIp} is rejected due to IpAddress filtering rules."
}
```

### <a name="check-if-the-eventgrid-topicdomain-can-be-accessed-using-only-a-private-endpoint"></a>Vérifier si la rubrique/le domaine EventGrid est accessible uniquement à l’aide d’un point de terminaison privé

Si la rubrique/le domaine Event Grid est configuré pour être accessible uniquement par le biais d’un point de terminaison privé, vérifiez que l’application cliente accède à la rubrique/le domaine sur le point de terminaison privé. Pour ce faire, vérifiez si l’application cliente s’exécute à l’intérieur d’un sous-réseau et qu’il existe un point de terminaison privé pour la rubrique/le domaine Event Grid dans ce sous-réseau.

[Le service Azure Private Link](../private-link/private-link-overview.md) vous permet d’accéder à Azure Event Grid sur un **point de terminaison privé** de votre réseau virtuel. Un point de terminaison privé est une interface réseau qui vous permet de vous connecter de façon privée et sécurisée à un service basé sur Azure Private Link. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ou d’adresses IP publiques. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez vous connecter à une instance d’une ressource Azure, ce qui vous donne le plus haut niveau de granularité en matière de contrôle d’accès.

Pour plus d’informations, consultez [Configurer des points de terminaison privés](configure-private-endpoints.md).

## <a name="troubleshoot-transient-connectivity-issues"></a>Résoudre les problèmes de connectivité temporaires

Si vous rencontrez des problèmes de connectivité intermittents, consultez les sections suivantes pour obtenir des conseils de dépannage.

### <a name="run-the-command-to-check-dropped-packets"></a>Exécutez la commande pour vérifier les paquets ignorés

Si vous constatez des problèmes de connectivité intermittents, exécutez la commande suivante pour détecter les paquets supprimés. Cette commande essaiera d’établir 25 connexions TCP différentes toutes les 1 seconde avec le service. Ensuite, vous pouvez vérifier le nombre d’entre elles ayant réussi ou échoué, ainsi que la latence de connexion TCP. Vous pouvez télécharger l’outil `psping` à partir d’[ici](/sysinternals/downloads/psping).

```shell
.\psping.exe -n 25 -i 1 -q {sampletopicname}.{region}-{suffix}.eventgrid.azure.net:443 -nobanner
```

Vous pouvez utiliser des commandes équivalentes dans d’autres outils, par exemple `tcpping` [tcpping.exe](https://www.elifulkerson.com/projects/tcping.php).

Si les étapes précédentes n’ont pas résolu le problème, obtenez une trace réseau et analysez-la à l’aide d’un outil tel que [Wireshark](https://www.wireshark.org/). Contactez le [support Microsoft](https://support.microsoft.com/) si nécessaire.

### <a name="service-upgradesrestarts"></a>Mises à niveau/redémarrages du service

Des problèmes de connectivité temporaires peuvent se produire en raison de mises à niveau et redémarrages du service principal. Lorsqu’ils se produisent, vous pouvez observer les symptômes suivants :

- Il peut y avoir une chute des messages/demandes entrants.
- Le fichier journal peut contenir des messages d’erreur.
- Les applications peuvent être déconnectées du service pendant quelques secondes.
- Les demandes peuvent être momentanément limitées.

Le fait d’intercepter ces erreurs temporaires, d’effectuer une sauvegarde et une nouvelle tentative d’appel permet de garantir la résilience de votre code à ces problèmes temporaires.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’une aide supplémentaire, publiez votre problème sur le forum [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-eventgrid) ou ouvrez un [ticket de support](https://azure.microsoft.com/support/options/).