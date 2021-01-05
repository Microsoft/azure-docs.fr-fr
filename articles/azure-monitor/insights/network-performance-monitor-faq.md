---
title: FAQ - Solution Network Performance Monitor dans Azure | Microsoft Docs
description: Cet article présente les questions fréquemment posées sur Network Performance Monitor dans Azure Network Performance Monitor (NPM) vous aide à surveiller les performances de vos réseaux, presque en temps réel, afin de détecter et localiser les goulots d’étranglement affectant les performances réseau.
ms.subservice: logs
ms.topic: conceptual
author: vinynigam
ms.author: vinigam
ms.date: 10/12/2018
ms.openlocfilehash: dd6c6248fcdf30350daecaa4857a81447bbdb7a8
ms.sourcegitcommit: 58f12c358a1358aa363ec1792f97dae4ac96cc4b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93280318"
---
# <a name="network-performance-monitor-solution-faq"></a>FAQ relative à la solution Network Performance Monitor

![Symbole de Network Performance Monitor](media/network-performance-monitor-faq/npm-symbol.png)

Cet article présente les questions fréquemment posées sur Network Performance Monitor (NPM) dans Azure

[Network Performance Monitor](../../networking/network-monitoring-overview.md) est une solution [d’analyse réseau hybride](./network-performance-monitor-performance-monitor.md) basé sur le cloud qui vous permet d’analyser les performances du réseau entre différents points de votre infrastructure réseau. Vous pouvez également analyser la connectivité réseau avec le [service et les points de terminaison d’application](./network-performance-monitor-service-connectivity.md) ainsi que les [performances d’Azure ExpressRoute](./network-performance-monitor-expressroute.md). 

Network Performance Monitor détecte les problèmes réseau comme le blocage du trafic, les erreurs de routage et autres problèmes que les méthodes d’analyse réseau classiques ne sont pas capables de détecter. La solution génère des alertes et vous informe en cas de dépassement d’un seuil pour une liaison réseau. Il assure également la détection en temps opportun des problèmes de performances réseau et en localise la source en identifiant un segment ou un appareil réseau particuliers. 

Plus d’informations sur les différentes fonctionnalités de prise en charge par [Network Performance Monitor](../../networking/network-monitoring-overview.md) sont disponibles en ligne.

## <a name="set-up-and-configure-agents"></a>Installer et configurer des agents

### <a name="what-are-the-platform-requirements-for-the-nodes-to-be-used-for-monitoring-by-npm"></a>Quelle est la plateforme requise pour les nœuds à utiliser à des fins de surveillance par NPM ?
Vous trouverez ci-dessous la plateforme requise pour les différentes fonctionnalités NPM :

- Les fonctionnalités Analyseur de performances et Moniteur de connectivité de service de NPM prennent en charge les systèmes d’exploitation Windows Serveur et de postes de travail/client Windows. Les versions 2008 SP1 et ultérieures du système d’exploitation Windows Serveur sont prises en charge. Les versions de postes de travail/client Windows prises en charge sont Windows 10, Windows 8.1, Windows 8 et Windows 7. 
- La fonctionnalité Moniteur ExpressRoute de NPM prend uniquement en charge le système d'exploitation du serveur Windows (2008 SP1 ou version ultérieure).

### <a name="can-i-use-linux-machines-as-monitoring-nodes-in-npm"></a>Puis-je utiliser des machines Linux en tant que nœuds de surveillance dans NPM ?
La fonctionnalité de surveillance des réseaux utilisant des nœuds basés sur Linux est actuellement en préversion. Accédez à l’agent [ici](../../virtual-machines/extensions/oms-linux.md). Pour en savoir plus, adressez-vous au responsable de votre compte. Les agents Linux fournissent une fonctionnalité de surveillance réservée à la fonctionnalité Analyseur de performances de NPM et ne sont pas disponibles pour les fonctionnalités Analyseur de connectivité de service et Moniteur ExpressRoute

### <a name="what-are-the-size-requirements-of-the-nodes-to-be-used-for-monitoring-by-npm"></a>Quelle est la taille requise pour les nœuds à utiliser à des fins de surveillance par NPM ?
Pour exécuter la solution NPM sur des machines virtuelles à nœuds, ces derniers doivent présenter au moins 500 Mo de mémoire et un cœur. Vous n’êtes pas tenu d’utiliser des nœuds distincts pour exécuter NPM. La solution peut s’exécuter sur des nœuds présentant d'autres charges de travail. La solution peut arrêter le processus de surveillance s’il utilise plus de 5 % de l’UC.

### <a name="to-use-npm-should-i-connect-my-nodes-as-direct-agent-or-through-system-center-operations-manager"></a>Pour utiliser NPM, dois-je connecter mes nœuds en tant qu’agents directs ou via System Center Operations Manager ?
Les fonctionnalités Analyseur de performances et Moniteur de connectivité de service prennent en charge les nœuds [connectés en tant qu’agents directs](../platform/agent-windows.md) et [connectés via Operations Manager](../platform/om-agents.md).

Pour la fonctionnalité Moniteur ExpressRoute, les nœuds Azure doivent être connectés en tant qu’agents directs uniquement. Les nœuds Azure connectés via Operations Manager ne sont pas pris en charge. Pour les nœuds locaux, les nœuds connectés en tant qu’agents directs et via Operations Manager sont pris en charge à des fins de surveillance d’un circuit ExpressRoute.

### <a name="which-protocol-among-tcp-and-icmp-should-be-chosen-for-monitoring"></a>Quel protocole choisir entre TCP et ICMP à des fins de surveillance ?
Si vous surveillez votre réseau à l’aide de nœuds basés sur le serveur Windows, nous vous recommandons d’utiliser TCP en tant que protocole de surveillance, car il offre une meilleure précision. 

ICMP est recommandé pour les nœuds basés sur le système d’exploitation des postes de travail/clients Windows. Cette plateforme n’autorise pas l’envoi de données TCP sur les sockets bruts, que NPM utilise pour découvrir la topologie réseau.

Vous trouverez plus d'informations sur les avantages liés à chaque protocole [ici](./network-performance-monitor-performance-monitor.md#choose-the-protocol).

### <a name="how-can-i-configure-a-node-to-support-monitoring-using-tcp-protocol"></a>Comment puis-je configurer un nœud pour prendre en charge la surveillance à l’aide du protocole TCP ?
Pour permettre à un nœud de prendre en charge la surveillance à l’aide du protocole TCP : 
* Vérifiez que la plateforme de nœud correspond à Windows Server (2008 SP1 ou version ultérieure).
* Exécutez le script Powershell [EnableRules.ps1](https://aka.ms/npmpowershellscript) sur le nœud. Pour plus d'informations, voir [Instructions](./network-performance-monitor.md#configure-log-analytics-agents-for-monitoring).


### <a name="how-can-i-change-the-tcp-port-being-used-by-npm-for-monitoring"></a>Comment puis-je modifier le port TCP utilisé par NPM à des fins de surveillance ?
Vous pouvez modifier le port TCP utilisé par NPM à des fins de surveillance en exécutant le script [EnableRules.ps1](https://aka.ms/npmpowershellscript). Vous devez entrer le numéro du port que vous souhaitez utiliser en tant que paramètre. Par exemple, pour activer TCP sur le port 8060, exécutez `EnableRules.ps1 8060`. Vérifiez que vous utilisez le même port TCP sur tous les nœuds utilisés à des fins de surveillance.

Le script configure uniquement le pare-feu Windows en local. En présence d'un pare-feu réseau ou de règles de groupe de sécurité réseau, vérifiez qu'ils autorisent le trafic destiné au port TCP utilisé par NPM.

### <a name="how-many-agents-should-i-use"></a>Combien d'agents dois-je utiliser ?
Vous devez utiliser au moins un agent pour chaque sous-réseau que vous souhaitez surveiller.

### <a name="what-is-the-maximum-number-of-agents-i-can-use-or-i-see-error--youve-reached-your-configuration-limit"></a>Quel est le nombre maximum d’agents utilisables avant qu’une erreur de type « ... vous avez atteint votre limite de configuration » s’affiche ?
NPM limite le nombre d'adresses IP à 5 000 par espace de travail. Si un nœud possède une adresse IPv4 et une adresse IPv6, deux adresses IP seront comptabilisées pour ce nœud. Par conséquent, cette limite de 5 000 adresses IP déterminera la limite supérieure du nombre d'agents. Vous pouvez supprimer les agents inactifs de l'onglet Nœuds sous NPM >> Configurer. NPM conserve aussi l’historique de toutes les adresses IP attribuées à la machine virtuelle qui héberge l’agent. Toutes ces adresses sont également comptabilisées en tant qu’adresses IP distinctes contribuant à la limite supérieure de 5 000 adresses IP. Afin de libérer des adresses IP pour votre espace de travail, vous pouvez utiliser la page Nœuds et supprimer les adresses IP inutilisées.

## <a name="monitoring"></a>Surveillance

### <a name="how-are-loss-and-latency-calculated"></a>Comment la perte et la latence sont-elles calculées ?
Les agents source envoient des requêtes TCP SYN (si TCP a été choisi en tant que protocole de surveillance) ou ICMP ECHO (si ICMP a été choisi en tant que protocole de surveillance) vers l'adresse IP de destination à intervalles réguliers pour vérifier que tous les chemins d’accès entre la combinaison d'adresses IP source-destination sont couverts. Le pourcentage de paquets reçus et la durée aller-retour des paquets sont mesurés pour calculer la perte et la latence de chaque chemin d’accès. Ces données sont agrégées sur l’intervalle d’interrogation et tous les chemins d’accès pour obtenir les valeurs agrégées de perte et de latence pour la combinaison d’adresses IP correspondant à l'intervalle d'interrogation donné.

### <a name="with-what-frequency-does-the-source-agent-send-packets-to-the-destination-for-monitoring"></a>À quelle fréquence l’agent source envoie-t-il des paquets vers la destination à des fins de surveillance ?
Pour les fonctionnalités Analyseur de performances et Moniteur ExpressRoute, la source envoie des paquets toutes les 5 secondes et enregistre les mesures du réseau. Ces données sont agrégées sur un intervalle d’interrogation de 3 minutes pour calculer les valeurs moyennes et maximales de perte et de latence. Pour la fonctionnalité Moniteur de connectivité de service, la fréquence d’envoi des paquets à des fins de mesure du réseau dépend de la fréquence entrée par l’utilisateur pour le test spécifique lors de la configuration du test.

### <a name="how-many-packets-are-sent-for-monitoring"></a>Combien de paquets sont-ils envoyés à des fins de surveillance ?
Le nombre de paquets envoyés par l’agent source vers la destination lors d'une interrogation est déterminé par notre algorithme propriétaire, qui peut être différent selon les topologies de réseau. Plus le nombre de chemins d’accès au réseau entre la combinaison d'adresses IP source-destination est élevé, plus le nombre de paquets envoyés est important. Le système s'assure que tous les chemins d’accès entre la combinaison d’adresses IP source-destination soient couverts.

### <a name="how-does-npm-discover-network-topology-between-source-and-destination"></a>Comment NPM détecte-t-il la topologie de réseau entre la source et la destination ?
NPM utilise un algorithme propriétaire basé sur la détermination d’itinéraire pour détecter tous les chemins d’accès et tronçons entre la source et la destination.

### <a name="does-npm-provide-routing-and-switching-level-info"></a>NPM fournit-il des informations de routage et de basculement ? 
Bien que NPM puisse détecter tous les itinéraires possibles entre l’agent source et la destination, il n'offre pas de visibilité sur l'itinéraire qu'empruntent les paquets envoyés par vos charges de travail spécifiques. La solution peut vous aider à identifier les chemins d’accès et tronçons sous-jacents ajoutant plus de latence que prévu.

### <a name="why-are-some-of-the-paths-unhealthy"></a>Pourquoi certains chemins sont-ils non sains ?
Il peut exister différents chemins d’accès réseau entre les adresses IP source et de destination, et chaque chemin d'accès peut présenter une valeur de perte et de latence différente. NPM marque ces chemins d’accès non sains (et les signale en rouge) car leurs valeurs de perte et/ou latence sont supérieures au seuil respectif défini dans la configuration de surveillance.

### <a name="what-does-a-hop-in-red-color-signify-in-the-network-topology-map"></a>Que signifie un tronçon en rouge sur la carte topologique du réseau ?
Si un tronçon est rouge, cela signifie qu’il fait partie d'au moins un chemin d'accès non sain. NPM se contente de marquer les chemins d’accès comme non sains, sans distinguer l'état d'intégrité de chacun. Pour identifier les tronçons problématiques, vous pouvez afficher la latence tronçon par tronçon et distinguer ceux qui ajoutent plus de latence que prévu.

### <a name="how-does-fault-localization-in-performance-monitor-work"></a>Comment la localisation des erreurs fonctionne-t-elle dans l’Analyseur de performances ?
NPM utilise un mécanisme probabiliste pour attribuer des probabilités d'erreur à chaque chemin d'accès réseau, segment réseau, tronçon réseau qui les composent selon le nombre de chemins d'accès non sains dont ils font partie. Si les segments et tronçons réseau font partie d'un nombre plus important de chemins d'accès non sains, la probabilité d'erreurs augmente. Cet algorithme fonctionne mieux en présence de nombreux nœuds avec des agents connectés les uns aux autres, car cela augmente les points de données pour calculer les probabilités d'erreurs.

### <a name="how-can-i-create-alerts-in-npm"></a>Comment puis-je créer des alertes dans NPM ?
Actuellement, la création d’alertes à partir de l’interface utilisateur NPM échoue en raison d’un problème connu. Veuillez [créer des alertes manuellement](../platform/alerts-log.md).

### <a name="what-are-the-default-log-analytics-queries-for-alerts"></a>Quelles sont les requêtes Log Analytics par défaut pour les alertes ?
Requête de l’analyseur de performances

```kusto
NetworkMonitoring
 | where (SubType == "SubNetwork" or SubType == "NetworkPath") 
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy") and RuleName == "<<your rule name>>"
```

Requête du moniteur de connectivité de service

```kusto
NetworkMonitoring
 | where (SubType == "EndpointHealth" or SubType == "EndpointPath")
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or ServiceResponseHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy") and TestName == "<<your test name>>"
```

Requêtes du moniteur ExpressRoute : Requête de circuits

```kusto
NetworkMonitoring
 | where (SubType == "ERCircuitTotalUtilization") and (UtilizationHealthState == "Unhealthy") and CircuitResourceId == "<<your circuit resource ID>>"
```

Peering privé

```kusto
NetworkMonitoring
 | where (SubType == "ExpressRoutePeering" or SubType == "ERVNetConnectionUtilization" or SubType == "ExpressRoutePath")   
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or UtilizationHealthState == "Unhealthy") and CircuitName == "<<your circuit name>>" and VirtualNetwork == "<<vnet name>>"
```

Peering Microsoft

```kusto
NetworkMonitoring
 | where (SubType == "ExpressRoutePeering" or SubType == "ERMSPeeringUtilization" or SubType == "ExpressRoutePath")
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or UtilizationHealthState == "Unhealthy") and CircuitName == ""<<your circuit name>>" and PeeringType == "MicrosoftPeering"
```

Requête courante

```kusto
NetworkMonitoring
 | where (SubType == "ExpressRoutePeering" or SubType == "ERVNetConnectionUtilization" or SubType == "ERMSPeeringUtilization" or SubType == "ExpressRoutePath")
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or UtilizationHealthState == "Unhealthy")
```

### <a name="can-npm-monitor-routers-and-servers-as-individual-devices"></a>NPM peut-il surveiller les routeurs et serveurs en tant qu’appareils individuels ?
NPM identifie uniquement l’adresse IP et le nom d’hôte des tronçons réseau sous-jacents (commutateurs, routeurs, serveurs, etc.) entre les adresses IP source et de destination. Il identifie également la latence entre ces tronçons identifiés. Il ne surveille pas individuellement ces tronçons sous-jacents.

### <a name="can-npm-be-used-to-monitor-network-connectivity-between-azure-and-aws"></a>NPM peut-il être utilisé pour surveiller la connectivité réseau entre Azure et AWS ?
Oui. Pour plus d'informations, voir l'article [Surveiller Azure, AWS et les réseaux locaux à l’aide de NPM](/archive/blogs/msoms/monitor-on-premises-cloud-iaas-and-hybrid-networks-using-oms-network-performance-monitor).

### <a name="is-the-expressroute-bandwidth-usage-incoming-or-outgoing"></a>L'utilisation de la bande passante ExpressRoute est-elle entrante ou sortante ?
L'utilisation de la bande passante correspond au total de bande passante entrante et sortante. Elle est exprimée en bits par seconde.

### <a name="can-we-get-incoming-and-outgoing-bandwidth-information-for-the-expressroute"></a>Est-il possible d'obtenir des informations de bande passante entrante et sortante pour ExpressRoute ?
Les valeurs entrantes et sortantes des bandes passantes principale et secondaire peuvent être capturées.

Pour des informations sur le peering MS, utilisez la requête mentionnée ci-dessous, dans Recherche dans les journaux

```kusto
NetworkMonitoring
 | where SubType == "ERMSPeeringUtilization"
 | project CircuitName,PeeringName,BitsInPerSecond,BitsOutPerSecond 
```

Pour des informations sur le peering privé, utilisez la requête mentionnée ci-dessous, dans Recherche dans les journaux

```kusto
NetworkMonitoring
 | where SubType == "ERVNetConnectionUtilization"
 | project CircuitName,PeeringName,BitsInPerSecond,BitsOutPerSecond
```

Pour des informations sur le niveau du circuit, utilisez la requête mentionnée ci-dessous, dans Recherche dans les journaux

```kusto
NetworkMonitoring
 | where SubType == "ERCircuitTotalUtilization"
 | project CircuitName, BitsInPerSecond, BitsOutPerSecond
```

### <a name="which-regions-are-supported-for-npms-performance-monitor"></a>Quelles régions sont-elles prises en charge pour la fonctionnalité Analyseur de performances de NPM ?
NPM peut surveiller la connectivité entre les réseaux dans n’importe quelle partie du monde, à partir d’un espace de travail hébergé dans une des [régions prises en charge](./network-performance-monitor.md#supported-regions)

### <a name="which-regions-are-supported-for-npms-service-connectivity-monitor"></a>Quelles régions sont-elles prises en charge pour la fonctionnalité Moniteur de connectivité de service de NPM ?
NPM peut surveiller la connectivité des services dans n’importe quelle partie du monde, à partir d’un espace de travail hébergé dans une des [régions prises en charge](./network-performance-monitor.md#supported-regions)

### <a name="which-regions-are-supported-for-npms-expressroute-monitor"></a>Quelles régions sont-elles prises en charge pour la fonctionnalité Moniteur ExpressRoute de NPM ?
NPM peut surveiller vos circuits ExpressRoute situés dans n’importe quelle région Azure. Pour intégrer NPM, vous devez disposer d’un espace de travail Log Analytics hébergé dans l’une des [régions prises en charge](../../expressroute/how-to-npm.md)

## <a name="troubleshoot"></a>Dépanner

### <a name="why-are-some-of-the-hops-marked-as-unidentified-in-the-network-topology-view"></a>Pourquoi certains tronçons sont-ils marqués comme non identifiés dans l'affichage de topologie de réseau ?
NPM utilise une version modifiée de détermination d’itinéraire pour détecter la topologie à partir de l’agent source vers la destination. Un tronçon non identifié indique que le tronçon réseau n'a pas répondu à la requête de détermination d’itinéraire de l’agent source. Si trois tronçons réseau consécutifs ne répondent pas à la détermination d’itinéraire de l’agent, la solution marque ces tronçons comme non identifiés et ne tente pas de détecter d’autres tronçons.

Un tronçon peut ne pas répondre à une détermination d’itinéraire dans un ou plusieurs des scénarios ci-dessous :

* Les routeurs ont été configurés pour ne pas révéler leur identité.
* Les périphériques réseau n'autorisent pas le trafic ICMP_TTL_EXCEEDED.
* Un pare-feu bloque la réponse ICMP_TTL_EXCEEDED à partir du périphérique réseau.

Quand l’un des points de terminaison se trouve dans Azure, traceroute affiche des sauts non identifiés, car l’infrastructure Azure ne révèle pas d’identité à traceroute. 

### <a name="i-get-alerts-for-unhealthy-tests-but-i-do-not-see-the-high-values-in-npms-loss-and-latency-graph-how-do-i-check-what-is-unhealthy"></a>Je reçois des alertes de tests non sains, mais aucune valeur élevée n'apparaît sur le graphique de perte et de latence de NPM. Comment vérifier ce qui n'est pas sain ?
NPM déclenche une alerte si une latence de bout en bout entre une source et une destination dépasse le seuil de tout chemin qui les sépare. Certains réseaux disposent de plusieurs chemins pour relier la même source et la même destination. NPM déclenche une alerte si un chemin n'est pas sain. La perte et la latence représentées sur les graphiques correspondent à la valeur moyenne de tous les chemins. Par conséquent, la valeur exacte d'un chemin individuel n'est pas nécessairement indiquée. Pour comprendre où le seuil a été dépassé, recherchez la colonne « SubType » dans l'alerte. Si le problème est dû à un chemin, la valeur de SubType est NetworkPath (pour les tests de Performance Monitor), EndpointPath (pour les tests de Service Connectivity Monitor) et ExpressRoutePath (pour les tests d'ExpressRoute Monitor). 

Exemple de requête pour déterminer si un chemin n'est pas sain :

```kusto
NetworkMonitoring
 | where ( SubType == "ExpressRoutePath")
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or UtilizationHealthState == "Unhealthy") and CircuitResourceID =="<your ER circuit ID>" and ConnectionResourceId == "<your ER connection resource id>"
 | project SubType, LossHealthState, LatencyHealthState, MedianLatency
```

### <a name="why-does-my-test-show-unhealthy-but-the-topology-does-not"></a>Pourquoi mon test apparaît-il comme non sain alors que ce n'est pas le cas de la topologie ? 
NPM surveille la perte de bout en bout, la latence et la topologie à différents intervalles. La perte et la latence sont mesurées toutes les 5 secondes et agrégées toutes les trois minutes (pour l’Analyseur de performances et le Moniteur ExpressRoute) alors que la topologie est calculée à l’aide de la détermination d’itinéraire toutes les 10 minutes. Par exemple, entre 3:44 et 4:04, la topologie peut être mise à jour trois fois (3:44, 3:54, 4:04), mais la perte et la latence sont mises à jour environ sept fois (3:44, 3:47, 3:50, 3:53, 3:56, 3:59, 4:02). La topologie générée à 3:54 sera affichée selon la perte et la latence calculées à 3:56, 3:59 et 4:02. Supposons la situation suivante : vous recevez une alerte indiquant que votre circuit ER était défectueux à 3:59. Vous ouvrez une session NPM et vous essayez de définir l’heure de la topologie à 3:59. NPM affichera la topologie générée à 3:54. Pour comprendre la dernière topologie connue de votre réseau, comparez les champs TimeProcessed (heure à laquelle la perte et la latence ont été calculées) et TracerouteCompletedTime (heure à laquelle la topologie a été calculée). 

### <a name="what-is-the-difference-between-the-fields-e2emedianlatency-and-avghoplatencylist-in-the-networkmonitoring-table"></a>Quelle est la différence entre les champs E2EMedianLatency et AvgHopLatencyList dans la table NetworkMonitoring ?
E2EMedianLatency est la latence mise à jour toutes les trois minutes après l’agrégation des résultats des tests de ping tcp, alors que AvgHopLatencyList est mis à jour toutes les 10 minutes en fonction de la détermination d’itinéraire. Pour comprendre l’heure exacte à laquelle E2EMedianLatency a été calculé, utilisez le champ TimeProcessed. Pour comprendre l’heure exacte à laquelle la détermination d’itinéraire a renseigné et mis à jour AvgHopLatencyList, utilisez le champ TracerouteCompletedTime.

### <a name="why-does-hop-by-hop-latency-numbers-differ-from-hoplatencyvalues"></a>Pourquoi les valeurs de latence tronçon par tronçon sont-elles différentes de HopLatencyValues ? 
Les valeurs HopLatencyValues s’entendent de la source jusqu’au point de terminaison.
Par exemple : Tronçons - A,B,C. AvgHopLatency - 10,15,20. Cela signifie que la latence de la source à A est 10, que la latence de la source à B est 15 et que la latence de la source à C est 20. L’interface utilisateur calculera la latence 5 pour le tronçon A-B dans la topologie.

### <a name="the-solution-shows-100-loss-but-there-is-connectivity-between-the-source-and-destination"></a>La solution affiche 100 % de pertes, mais il existe une connectivité entre la source et la destination.
Cela peut se produire si le pare-feu d’hôte ou le pare-feu intermédiaire (pare-feu réseau ou groupe de sécurité réseau Azure) bloque la communication entre l’agent source et la destination sur le port utilisé à des fins de surveillance par NPM (par défaut et sauf modification du client, ce port correspond à 8084).

* Pour vérifier que le pare-feu d’hôte ne bloque pas la communication sur le port requis, affichez l’état d’intégrité des nœuds source et de destination à partir de l’affichage suivant : Network Performance Monitor -> Configuration -> Nœuds. 
  S’ils s'avèrent non sains, consultez les instructions et prenez les mesures correctives qui s'imposent. Si les nœuds sont sains, passez à l’étape b. souhaité ci-dessous.
* Pour vérifier qu’un pare-feu réseau intermédiaire ou groupe de sécurité réseau Azure ne bloque pas la communication sur le port requis, utilisez l’utilitaire PsPing tiers comme suit :
  * L'utilitaire PsPing peut être téléchargé [ici](/sysinternals/downloads/psping). 
  * Exécutez la commande suivante à partir du nœud source.
    * psping -n 15 \<destination node IPAddress\>:portNumber. Par défaut, NPM utilise le port 8084. Si vous avez explicitement modifié cela en utilisant le script EnableRules.ps1, entrez le numéro de port personnalisé utilisé. Il s’agit d’une commande ping entre la machine Azure et le serveur local.
* Vérifiez si les tests ping aboutissent. Dans le cas contraire, cela indique qu’un pare-feu réseau intermédiaire ou groupe de sécurité réseau Azure bloque le trafic sur ce port.
* À présent, exécutez la commande entre le nœud de destination et l'adresse IP du nœud source.


### <a name="there-is-loss-from-node-a-to-b-but-not-from-node-b-to-a-why"></a>Il existe une perte entre le nœud A vers B, mais pas entre le nœud B vers A. Pourquoi ?
Les chemins d’accès réseau de A vers B peuvent être différents des chemins d’accès réseau de B vers A, des valeurs distinctes en termes de perte et latence peuvent être observées.

### <a name="why-are-all-my-expressroute-circuits-and-peering-connections-not-being-discovered"></a>Pourquoi mes circuits ExpressRoute et connexions de peering ne sont-ils pas détectés ?
NPM détecte désormais les circuits ExpressRoute et les connexions de peering dans tous les abonnements auxquels l’utilisateur a accès. Choisissez tous les abonnements dans lesquels vos ressources ExpressRoute sont reliées et activez la surveillance pour chaque ressource découverte. NPM recherche les objets de connexion lors de la découverte d’un peering privé. Par conséquent, vérifiez si un réseau virtuel est associé à votre peering. NPM ne détecte pas les circuits ni les Peerings qui se trouvent dans un locataire différent de l’espace de travail Log Analytics.

### <a name="the-er-monitor-capability-has-a-diagnostic-message-traffic-is-not-passing-through-any-circuit-what-does-that-mean"></a>La fonctionnalité Moniteur ER affiche un message de diagnostic « Le trafic n'est acheminé dans AUCUN circuit ». Qu'est-ce que cela signifie ?

Cela peut indiquer que la connexion entre les nœuds locaux et Azure est saine, mais que le trafic n'est pas acheminé via le circuit ExpressRoute configuré pour être surveillé par NPM. 

Cela peut se produire si :

* Le circuit ER est arrêté.
* Les filtres de routage sont configurés de manière à privilégier les autres itinéraires (connexion VPN ou autre circuit ExpressRoute, par exemple) sur le circuit ExpressRoute prévu. 
* Les nœuds locaux et Azure choisis pour surveiller le circuit ExpressRoute dans la configuration de surveillance n'ont pas de connectivité entre eux sur le circuit ExpressRoute prévu. Vérifiez que vous avez choisi des nœuds présentant une connectivité entre eux sur le circuit ExpressRoute que vous souhaitez surveiller.

### <a name="why-does-expressroute-monitor-report-my-circuitpeering-as-unhealthy-when-it-is-available-and-passing-data"></a>Pourquoi Moniteur ExpressRoute signale-t-il mon circuit/Peering comme non sain alors qu’il est disponible et transmet des données ?
Moniteur ExpressRoute compare les valeurs de performances du réseau (perte, latence et utilisation de la bande passante) signalées par les agents ou le service aux seuils définis lors de la configuration. Pour un circuit, si l’utilisation de la bande passante indiquée est supérieure au seuil défini dans la configuration, le circuit est marqué comme non sain. Pour les Peerings, si la perte, la latence ou l’utilisation de la bande passante indiquée est supérieure au seuil défini dans la configuration, le Peering est marqué comme non sain. NPM n’utilise pas de métriques ni toute autre forme de données pour déterminer l’état d’intégrité.

### <a name="why-does-expressroute-monitorbandwidth-utilisation-report-a-value-differrent-from-metrics-bits-inout"></a>Pourquoi l’utilisation de la bande passante de Moniteur ExpressRoute rapporte-t-elle une valeur différente des entrées/sorties de bits des métriques ?
Pour Moniteur ExpressRoute, l’utilisation de la bande passante correspond à la moyenne de la bande passante entrante et sortante au cours des 20 dernières minutes ; elle est exprimée en bits/s. Pour les métriques d’ExpressRoute, les entrées/sorties de bits sont des points de données par minute. En interne, le jeu de données utilisé pour les deux est le même, mais l’agrégation varie entre les métriques NPM et ER. Pour une surveillance granulaire, minute par minute et des alertes rapides, nous vous recommandons de définir des alertes directement sur les métriques ER

### <a name="while-configuring-monitoring-of-my-expressroute-circuit-the-azure-nodes-are-not-being-detected"></a>Lors de la configuration de la surveillance de mon circuit ExpressRoute, les nœuds Azure ne sont pas détectés.
Cela peut se produire si les nœuds Azure sont connectés via Operations Manager. La fonctionnalité Moniteur ExpressRoute prend uniquement en charge les nœuds Azure connectés en tant qu'agents directs.

### <a name="i-cannot-discover-by-expressroute-circuits-in-the-oms-portal"></a>Je ne parviens pas à détecter les circuits ExpressRoute dans le portail OMS.
Bien que NPM puisse être utilisé à partir du portail Azure et du portail OMS, la détection de circuits à l'aide de la fonctionnalité Moniteur ExpressRoute fonctionne uniquement via le portail Azure. Une fois les circuits détectés via le portail Azure, vous pouvez utiliser la fonctionnalité dans le portail de votre choix. 

### <a name="in-the-service-connectivity-monitor-capability-the-service-response-time-network-loss-as-well-as-latency-are-shown-as-na"></a>Pour ce qui est de la fonctionnalité Moniteur de connectivité de service, le temps de réponse du service, la perte de connexion réseau et la latence sont indiqués comme non applicables.
Cela peut se produire si une ou plusieurs des situations suivantes se vérifient :

* Le service est arrêté.
* Le nœud utilisé pour la vérification de la connectivité réseau du service est arrêté.
* La cible entrée dans la configuration de test est incorrecte.
* Le nœud n’a pas de connectivité réseau.

### <a name="in-the-service-connectivity-monitor-capability-a-valid-service-response-time-is-shown-but-network-loss-as-well-as-latency-are-shown-as-na"></a>Pour ce qui est de la fonctionnalité Moniteur de connectivité de service, un temps de réponse du service valide s'affiche, mais la perte de connexion réseau et la latence sont indiqués comme non applicables.
 Cela peut se produire si une ou plusieurs des situations suivantes se vérifient :

* Si le nœud utilisé pour la vérification de la connectivité réseau du service est un ordinateur client de Windows, le service cible bloque les requêtes ICMP ou un pare-feu réseau bloque les requêtes ICMP provenant du nœud d’origine.
* La case pour effectuer des mesures sur un réseau est vide dans la configuration de test.

### <a name="in-the-service-connectivity-monitor-capability-the-service-response-time-is-na-but-network-loss-as-well-as-latency-are-valid"></a>Pour ce qui est de la fonctionnalité Moniteur de connectivité de service est non applicable, mais la perte de connexion réseau et la latence sont valides.
Cela peut se produire si le service cible ne correspond pas à une application web, mais que le test est configuré en tant que test web. Modifiez la configuration de test et choisissez le type de test Réseau au lieu de Web.

## <a name="miscellaneous"></a>Divers

### <a name="is-there-a-performance-impact-on-the-node-being-used-for-monitoring"></a>Existe-t-il un impact en termes de performances sur le nœud utilisé à des fins de surveillance ?
Le processus NPM est configuré pour s’arrêter s'il utilise plus de 5 % des ressources d'UC de l'hôte. Vous pouvez ainsi continuer d'utiliser les nœuds pour leurs charges de travail habituelles, sans impact en termes de performances.

### <a name="does-npm-edit-firewall-rules-for-monitoring"></a>NPM modifie-t-il les règles de pare-feu pour la surveillance ?
NPM crée uniquement une règle de pare-feu Windows locale sur les nœuds sur lesquels le script Powershell EnableRules.ps1 est exécuté afin d'autoriser les agents à créer des connexions TCP entre eux sur le port spécifié. La solution ne modifie pas les règles de pare-feu réseau ou de groupe de sécurité réseau.

### <a name="how-can-i-check-the-health-of-the-nodes-being-used-for-monitoring"></a>Comment puis-je vérifier l’intégrité des nœuds utilisés à des fins de surveillance ?
Vous pouvez afficher l’état d’intégrité des nœuds utilisés pour la surveillance à partir de l’affichage suivant : Network Performance Monitor -> Configuration -> Nœuds. Si un nœud s'avère non sain, vous pouvez afficher les détails correspondants et prendre les mesures suggérées.

### <a name="can-npm-report-latency-numbers-in-microseconds"></a>NPM peut-il signaler les valeurs de latence en microsecondes ?
NPM arrondit les valeurs de latence dans l’interface utilisateur en millisecondes. Les mêmes données sont stockées avec une granularité plus élevée (pouvant aller jusqu'à quatre décimales).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d'informations sur Network Performance Monitor, voir [Solution Network Performance Monitor dans Azure](./network-performance-monitor.md).

