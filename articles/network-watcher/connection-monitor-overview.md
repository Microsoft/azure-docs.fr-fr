---
title: Moniteur de connexion dans Azure | Microsoft Docs
description: Apprenez à utiliser la fonctionnalité Moniteur de connexion pour surveiller les communications réseau dans un environnement distribué.
services: network-watcher
documentationcenter: na
author: vinynigam
manager: agummadi
editor: ''
tags: azure-resource-manager
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: vinigam
ms.custom: mvc
ms.openlocfilehash: c5c3f8dedad47a819ce3648a0b81ffa3e65a1a1e
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95544885"
---
# <a name="network-connectivity-monitoring-with-connection-monitor"></a>Surveillance de la connectivité réseau à l’aide de Moniteur de connexion

La fonctionnalité Moniteur de connexion permet une vérification unifiée et de bout en bout de la connectivité dans Azure Network Watcher. Elle prend en charge les déploiements hybrides et cloud Azure. Network Watcher fournit des outils pour surveiller, diagnostiquer et consulter les métriques de connectivité de vos déploiements Azure.

Voici quelques cas d’usage de Moniteur de connexion :

- Votre machine virtuelle de serveur web front-end communique avec une machine virtuelle de serveur de base de données dans une application multiniveau. Vous souhaitez vérifier la connectivité réseau entre les deux machines virtuelles.
- Vous souhaitez que les machines virtuelles de la région USA Est puissent effectuer un test Ping ciblant les machines virtuelles de la région USA Centre, et vous souhaitez comparer les temps de réponse du réseau entre les régions.
- Vous disposez de plusieurs sites locaux à Seattle, Washington et Ashburn (Virginie). Vos sites de bureau se connectent à des URL Microsoft 365. Pour vos utilisateurs d’URL Microsoft 365, comparez les latences entre Seattle et Ashburn.
- Votre application hybride doit être connectée à un point de terminaison Stockage Azure. Votre site local et votre application Azure se connectent au même point de terminaison Stockage Azure. Vous souhaitez comparer les temps de réponse du site local avec ceux de l'application Azure.
- Vous souhaitez vérifier la connectivité entre vos installations locales et les machines virtuelles Azure qui hébergent votre application cloud.

Moniteur de connexion allie le meilleur des deux fonctionnalités suivantes : la fonctionnalité [Moniteur de connexion (Classique)](./network-watcher-monitoring-overview.md#monitor-communication-between-a-virtual-machine-and-an-endpoint) de Network Watcher et la fonctionnalité [Moniteur de connectivité de service](../azure-monitor/insights/network-performance-monitor-service-connectivity.md), [Surveillance ExpressRoute](../expressroute/how-to-npm.md) et [Analyse des performances](../azure-monitor/insights/network-performance-monitor-performance-monitor.md) de Network Performance Monitor (NPM).

Moniteur de connexion présente notamment les avantages suivants :

* Expérience unifiée et intuitive pour les besoins de supervision des clouds Azure et hybride
* Surveillance de la connectivité entre les régions et entre les espaces de travail
* Fréquences de sondage supérieures et meilleure visibilité des performances du réseau
* Alertes plus rapides pour vos déploiements hybrides
* Prise en charge des vérifications de la connectivité reposant sur HTTP, TCP et ICMP 
* Prise en charge des métriques et de Log Analytics pour les initialisations (tearDown) de test Azure et non Azure

![Diagramme montrant comment le Moniteur de connexion interagit avec les machines virtuelles Azure, les hôtes non Azure, les points de terminaison et les emplacements de stockage de données](./media/connection-monitor-2-preview/hero-graphic.png)

Pour commencer à utiliser Moniteur de connexion à des fins de surveillance, procédez comme suit : 

1. Installez des agents de surveillance.
1. Activez Network Watcher sur votre abonnement.
1. Créez un moniteur de connexion.
1. Configurez l'analyse des données et les alertes.
1. Diagnostiquez les problèmes liés à votre réseau.

Pour plus d'informations, lisez les sections suivantes.

## <a name="install-monitoring-agents"></a>Installer des agents de supervision

Le Moniteur de connexion s'appuie sur des exécutables légers pour effectuer les vérifications de la connectivité. Il prend en charge les vérifications de la connectivité à partir des environnements Azure et des environnements locaux. L'exécutable à utiliser varie selon que votre machine virtuelle est hébergée sur Azure ou en local.

### <a name="agents-for-azure-virtual-machines"></a>Agents pour les machines virtuelles Azure

Pour que le Moniteur de connexion reconnaisse vos machines virtuelles Azure comme sources de surveillance, installez l'extension de machine virtuelle Network Watcher Agent sur celles-ci. Cette extension est également connue sous le nom d'*extension Network Watcher*. Les machines virtuelles Azure ont besoin de cette extension pour déclencher une surveillance de bout en bout et d'autres fonctionnalités avancées. 

Vous pouvez installer l'extension Network Watcher au moment de la [création d'une machine virtuelle](./connection-monitor.md#create-the-first-vm). Vous pouvez également procéder à l'installation, à la configuration et à la résolution des problèmes liés à l'extension Network Watcher séparément pour [Linux](../virtual-machines/extensions/network-watcher-linux.md) et [Windows](../virtual-machines/extensions/network-watcher-windows.md).

Les règles d'un groupe de sécurité réseau (NSG) ou d'un pare-feu peuvent empêcher la communication entre la source et la destination. Le Moniteur de connexion détecte ce problème et le signale sous la forme d'un message de diagnostic dans la topologie. Pour activer la vérification de la connectivité, vérifiez que les règles NSG et les règles de pare-feu autorisent les paquets sur TCP ou ICMP entre la source et la destination.

### <a name="agents-for-on-premises-machines"></a>Agents pour machines locales

Pour que le Moniteur de connexion reconnaisse vos machines locales en tant que sources de surveillance, installez l'agent Log Analytics sur les machines. Puis activez la solution Network Performance Monitor. Ces agents sont liés aux espaces de travail Log Analytics. Par conséquent, vous devez configurer l'ID de l'espace de travail et la clé primaire pour permettre aux agents d'entamer la surveillance.

Pour installer l'agent Log Analytics pour machines Windows, consultez [Extension de machine virtuelle Azure Monitor pour Windows](../virtual-machines/extensions/oms-windows.md).

Si le chemin inclut des pare-feu ou des appliances virtuelles réseau (NVA), assurez-vous que la destination est accessible.

## <a name="enable-network-watcher-on-your-subscription"></a>Activer Network Watcher dans votre abonnement

Tous les abonnements disposant d'un réseau virtuel sont activés avec Network Watcher. Lorsque vous créez un réseau virtuel sur votre abonnement, Network Watcher est automatiquement activé dans la région et sur l'abonnement correspondant à ce réseau virtuel. Cette activation automatique n'affecte pas vos ressources et n'entraîne pas de frais. Vérifiez que Network Watcher n’est pas explicitement désactivé dans votre abonnement. 

Pour plus d’informations, consultez [Activer Network Watcher](./network-watcher-create.md).

## <a name="create-a-connection-monitor"></a>Créer un moniteur de connexion 

Le Moniteur de connexion surveille la communication à intervalles réguliers. Il vous informe des changements en matière d'accessibilité et de latence. Vous pouvez également vérifier la topologie actuelle et historique du réseau entre les agents sources et les points de terminaison de destination.

Les sources peuvent être des machines virtuelles Azure ou des machines locales sur lesquelles un agent de surveillance est installé. Les points de terminaison de destination peuvent être des URL Microsoft 365, des URL Dynamics 365, des URL personnalisées, des ID de ressources de machine virtuelle Azure, des adresses IPv4 ou IPv6, un FQDN ou un nom de domaine.

### <a name="access-connection-monitor"></a>Accéder à Moniteur de connexion

1. Sur la page d'accueil du portail Azure, accédez à **Network Watcher**.
1. Sur la gauche, dans la section **Surveillance**, sélectionnez **Moniteur de connexion**.
1. Tous les moniteurs de connexion créés dans la fonctionnalité Moniteur de connexion sont répertoriés. Pour afficher les moniteurs de connexion créés dans l'expérience utilisateur classique du Moniteur de connexion, accédez à l'onglet **Moniteur de connexion**.
    
  :::image type="content" source="./media/connection-monitor-2-preview/cm-resource-view.png" alt-text="Capture d’écran illustrant des moniteurs de connexion créés dans Moniteur de connexion" lightbox="./media/connection-monitor-2-preview/cm-resource-view.png":::

### <a name="create-a-connection-monitor"></a>Créer un moniteur de connexion

Dans les moniteurs de connexion que vous créez à l’aide de la fonctionnalité Moniteur de connexion, vous pouvez ajouter des machines locales et des machines virtuelles Azure en tant que sources. Ces moniteurs de connexion peuvent également surveiller la connectivité aux points de terminaison. Les points de terminaison peuvent se trouver sur Azure ou sur toute autre URL ou adresse IP.

Le Moniteur de connexion inclut les entités suivantes :

* **Ressource de moniteur de connexion** : ressource Azure spécifique à la région. Toutes les entités ci-dessous sont des propriétés d'une ressource de moniteur de connexion.
* **Point de terminaison** : source ou destination qui participe aux vérifications de la connectivité. Les machines virtuelles Azure, les agents locaux, les URL et les adresses IP sont des exemples de points de terminaison.
* **Configuration de test** : configuration spécifique à un protocole dans le cadre d'un test. En fonction du protocole que vous avez choisi, vous pouvez définir le port, les seuils, la fréquence de test et d'autres paramètres.
* **Groupe de tests** : groupe contenant les points de terminaison sources, les points de terminaison de destination et les configurations de test. Un moniteur de connexion peut contenir plusieurs groupes de tests.
* **Test** : combinaison d'un point de terminaison source, d'un point de terminaison de destination et d'une configuration de test. Un test correspond au niveau le plus granulaire auquel les données de surveillance sont disponibles. Les données de surveillance incluent le pourcentage de vérifications qui ont échoué et la durée des boucles.

 ![Diagramme illustrant un moniteur de connexion, avec définition de la relation entre les groupes de tests et les tests](./media/connection-monitor-2-preview/cm-tg-2.png)

Vous pouvez créer un moniteur de connexion à l’aide du [portail Azure](./connection-monitor-create-using-portal.md) ou d’[ARMClient](./connection-monitor-create-using-template.md).

Toutes les sources, destinations et configurations de test que vous ajoutez à un groupe de tests sont réparties en tests individuels. Voici un exemple de répartition des sources et des destinations :

* Groupe de tests : Groupe de tests 1
* Sources : 3 (A, B, C)
* Destinations : 2 (D, E)
* Configurations de test : 2 (Config 1, Config 2)
* Nombre total de tests créés : 12

| Numéro du test | Source | Destination | Configuration de test |
| --- | --- | --- | --- |
| 1 | Un | D | Config 1 |
| 2 | Un | D | Config 2 |
| 3 | Un | E | Config 1 |
| 4 | Un | E | Config 2 |
| 5 | B | D | Config 1 |
| 6 | B | D | Config 2 |
| 7 | B | E | Config 1 |
| 8 | B | E | Config 2 |
| 9 | C | D | Config 1 |
| 10 | C | D | Config 2 |
| 11 | C | E | Config 1 |
| 12 | C | E | Config 2 |

### <a name="scale-limits"></a>Limites de mise à l’échelle

Les moniteurs de connexion présentent les limites suivantes en termes de mise à l'échelle :

* Nombre maximum de moniteurs de connexion par abonnement et par région : 100
* Nombre maximum de groupes de tests par moniteur de connexion : 20
* Nombre maximum de sources et de destinations par moniteur de connexion : 100
* Nombre maximum de configurations de test par moniteur de connexion :

## <a name="analyze-monitoring-data-and-set-alerts"></a>Analyser les données de surveillance et définir des alertes

Après la création d'un moniteur de connexion, les sources vérifient la connectivité aux destinations en fonction de votre configuration de test.

### <a name="checks-in-a-test"></a>Vérifications dans un test

En fonction du protocole que vous avez choisi dans la configuration de test, Moniteur de connexion exécute une série de vérifications pour la paire source-destination. Les vérifications sont exécutées en fonction de la fréquence de test que vous avez choisie.

Si vous utilisez HTTP, le service calcule le nombre de réponses HTTP qui ont renvoyé un code de réponse valide. Les codes de réponse valides peuvent être définis à l’aide de PowerShell et de l’interface CLI. Le résultat détermine le pourcentage de vérifications qui ont échoué. Pour calculer la durée des boucles, le service mesure le délai entre un appel HTTP et la réponse.

Si vous utilisez TCP ou ICMP, le service calcule le pourcentage de perte de paquets pour déterminer le pourcentage de vérifications qui ont échoué. Pour calculer la durée des boucles, le service mesure le temps nécessaire à la réception de l'accusé de réception (ACK) pour les paquets qui ont été envoyés. Si vous avez activé les données traceroute pour vos tests réseau, vous pouvez voir les pertes et la latence tronçon par tronçon de votre réseau local.

### <a name="states-of-a-test"></a>États d’un test

Sur la base des données renvoyées par les vérifications, les tests peuvent présenter les états suivants :

* **Réussite** : les valeurs réelles du pourcentage de vérifications qui ont échoué et de la durée des boucles se situent dans les seuils spécifiés.
* **Échec** : les valeurs réelles du pourcentage de vérifications qui ont échoué et de la durée des boucles ont dépassé les seuils spécifiés. Si aucun seuil n'est spécifié, un test atteint l'état Échec lorsque le pourcentage de vérifications qui ont échoué est de 100.
* **Avertissement** : 
     * si le seuil est spécifié et que Moniteur de connexion constate un pourcentage d’échec de vérification supérieur à 80 % du seuil, le test est marqué en tant qu’avertissement.
     * En l’absence de seuils spécifiés, Moniteur de connexion attribue automatiquement un seuil. Lorsque ce seuil est dépassé, l'état du test passe à Avertissement. Pour la durée de l’aller-retour dans les tests TCP ou ICMP, le seuil est de 750 msec. Pour le pourcentage de vérifications ayant échoué, le seuil est de 10 %. 
* **indéterminé**  : aucune donnée dans l’espace de travail Log Analytics.  Vérifiez les mesures. 
* **Pas en cours d’exécution**  : désactivé par désactivation du groupe de tests.  

### <a name="data-collection-analysis-and-alerts"></a>Collecte de données, analyse et alertes

Les données collectées par Moniteur de connexion sont stockées dans l’espace de travail Log Analytics. Vous avez configuré cet espace de travail lors de la création du Moniteur de connexion. 

Les données de supervision sont également disponibles dans les métriques d’Azure Monitor. Vous pouvez utiliser Log Analytics pour conserver vos données de surveillance aussi longtemps que vous le souhaitez. Par défaut, Azure Monitor ne stocke les données que pendant 30 jours. 

Vous pouvez [définir des alertes basées sur des métriques pour les données](https://azure.microsoft.com/blog/monitor-at-scale-in-azure-monitor-with-multi-resource-metric-alerts/).

#### <a name="monitoring-dashboards"></a>Tableaux de bord de surveillance

Les tableaux de bord de surveillance présentent la liste des moniteurs de connexion auxquels vous pouvez accéder pour vos abonnements, régions, horodatages, sources et types de destination.

Lorsque vous accédez à Moniteur de connexion à partir de Network Watcher, vous pouvez afficher les données par :

* **Moniteur de connexion** : liste de tous les moniteurs de connexion créés pour vos abonnements, régions, horodatages, sources et types de destination. Il s'agit de la vue par défaut.
* **Groupes de tests** : liste de tous les groupes de tests créés pour vos abonnements, régions, horodatages, sources et types de destination. Ces groupes de tests ne sont pas filtrés par moniteurs de connexion.
* **Test** : liste de tous les tests qui s'exécutent pour vos abonnements, régions, horodatages, sources et types de destination. Ces tests ne sont pas filtrés par moniteurs de connexion ou groupes de tests.

Sur l'illustration suivante, les trois vues de données sont signalées par la flèche 1.

Sur le tableau de bord, vous pouvez développer chaque moniteur de connexion pour afficher ses groupes de tests. Vous pouvez ensuite développer chaque groupe de tests pour afficher les tests qui y sont exécutés. 

Vous pouvez filtrer une liste en fonction des critères suivants :

* **Filtres de niveau supérieur** : liste de recherche par texte, type d’entité (Moniteur de connexion, groupe de test ou test), horodateur et étendue. L’étendue comprend les abonnements, les régions, les sources et les types de destinations. Voir la zone 1 dans l’image suivante.
* **Filtres basés sur l'état** : filtres basés sur l'état du moniteur de connexion, du groupe de tests ou du test. Voir la zone 2 dans l'image suivante.
* **Filtre basé sur des alertes** : filtre par alertes déclenchées sur la ressource du moniteur de connexion. Voir la zone 3 dans l’image suivante.

  :::image type="content" source="./media/connection-monitor-2-preview/cm-view.png" alt-text="Capture d’écran montrant comment filtrer les vues des moniteurs de connexion, des groupes de tests et des tests dans Moniteur de connexion" lightbox="./media/connection-monitor-2-preview/cm-view.png":::
    
Par exemple, pour afficher dans Moniteur de connexion tous les tests dont l’adresse IP source correspond à 10.192.64.56 :
1. Remplacez la vue par **Test**.
1. Dans le champ de recherche, entrez *10.192.64.56*
1. Dans **Étendue** dans le filtre de niveau supérieur, sélectionnez **Sources**.

Pour n’afficher dans Moniteur de connexion que les tests qui ont échoué et dont l’adresse IP source correspond à 10.192.64.56 :
1. Remplacez la vue par **Test**.
1. Pour le filtre basé sur l'état, sélectionnez **Échec**.
1. Dans le champ de recherche, entrez *10.192.64.56*
1. Dans **Étendue** dans le filtre de niveau supérieur, sélectionnez **Sources**.

Pour n’afficher dans Moniteur de connexion que les tests qui ont échoué et dont la destination est outlook.office365.com :
1. Remplacez la vue par **Test**.
1. Pour le filtre basé sur l'état, sélectionnez **Échec**.
1. Dans le champ de recherche, entrez *office.live.com*.
1. Dans **Étendue** dans le filtre de niveau supérieur, sélectionnez **Destinations**.
  
  :::image type="content" source="./media/connection-monitor-2-preview/tests-view.png" alt-text="Capture d'écran montrant une vue filtrée pour n'afficher que les tests qui ont échoué et dont la destination est Outlook.Office365.com" lightbox="./media/connection-monitor-2-preview/tests-view.png":::

Pour connaître la raison de l’échec d’un Moniteur de connexion, d’un groupe de test ou d’un test, cliquez sur la colonne intitulée Raison.  Celle-ci indique quel seuil (% de vérifications ayant échoué ou RTT) a été violé et les messages de diagnostics associés
  
  :::image type="content" source="./media/connection-monitor-2-preview/cm-reason-of-failure.png" alt-text="Capture d’écran montrant la raison de l’échec d’un moniteur de connexion, d’un test ou d’un groupe de test" lightbox="./media/connection-monitor-2-preview/cm-reason-of-failure.png":::
    
Pour afficher les tendances relatives à la durée des boucles et le pourcentage de vérifications qui ont échoué pour un moniteur de connexion :
1. Sélectionnez le moniteur de connexion à examiner.

    :::image type="content" source="./media/connection-monitor-2-preview/cm-drill-landing.png" alt-text="Capture d'écran illustrant les métriques d'un moniteur de connexion, affichées par groupe de tests" lightbox="./media/connection-monitor-2-preview/cm-drill-landing.png":::

1. Les sections suivantes s’affichent  
    1. Essentiels : propriétés spécifiques de la ressource du Moniteur de connexion sélectionné. 
    1. Résumé : 
        1. courbes de tendance agrégées pour RTT et pourcentage de vérifications ayant échoué pour tous les tests dans le moniteur de connexion. Vous pouvez définir une heure spécifique pour afficher les détails.
        1. Top 5 des groupes de test, des sources et des destinations en fonction du RTT ou du pourcentage de vérifications ayant échoué. 
    1. Onglets pour Groupes de test, Sources, Destinations et Configurations de test : répertorie les groupes de test, les sources ou les destinations dans le Moniteur de connexion. Valeurs de % de tests de vérification ayant échoué, de RTT d’agrégat et de vérifications ayant échoué.  Vous pouvez également remonter dans le temps pour afficher les données. 
    1. Problèmes : problèmes au niveau tronçon pour chaque test dans le Moniteur de connexion. 

    :::image type="content" source="./media/connection-monitor-2-preview/cm-drill-landing-2.png" alt-text="Capture d’écran illustrant les métriques d’un moniteur de connexion, affichées par groupe de tests partie 2" lightbox="./media/connection-monitor-2-preview/cm-drill-landing-2.png":::

1. Vous pouvez :
    * Cliquer sur Afficher tous les tests pour afficher tous les tests dans le Moniteur de connexion
    * Cliquez sur Afficher tous les groupes de test, les configurations de test, les sources et les destinations pour afficher les détails spécifiques de chacun. 
    * Choisissez un groupe de test, une configuration de test, une source ou une destination pour afficher tous les tests de l’entité.

1. À partir de l’affichage de tous les tests, vous pouvez :
    * Sélectionnez des tests, puis cliquez sur Comparer.
    
    :::image type="content" source="./media/connection-monitor-2-preview/cm-compare-test.png" alt-text="Capture d’écran montrant la comparaison de 2 tests" lightbox="./media/connection-monitor-2-preview/cm-compare-test.png":::
    
    * Utiliser un cluster pour étendre des ressources composées telles qu’un réseau virtuel et des sous-réseaux à leurs ressources enfants
    * Affichez la topologie des tests en cliquant sur Topologie.

Pour afficher les tendances relatives à la durée des boucles et le pourcentage de vérifications qui ont échoué pour un groupe de tests :
1. Sélectionnez le groupe de tests à examiner. 
1. Vous obtiendrez un affichage similaire au Moniteur de connexion : essentiels, résumé, table pour les groupes de test, sources, destinations et configurations de test. Parcourez-les comme vous le feriez pour un moniteur de connexion

Pour afficher les tendances relatives à la durée des boucles et le pourcentage de vérifications qui ont échoué pour un test :
1. Choisissez le test à examiner. Vous verrez la topologie de réseau et les graphiques de tendance de bout en bout pour le % de vérifications ayant échoué et la durée aller-retour. Pour afficher les problèmes identifiés, dans la topologie, sélectionnez un tronçon. (Ces tronçons sont des ressources Azure.) Cette fonctionnalité n’est actuellement pas disponible pour les réseaux locaux.

  :::image type="content" source="./media/connection-monitor-2-preview/cm-test-topology.png" alt-text="Capture d’écran montrant l’affichage de la topologie d’un test" lightbox="./media/connection-monitor-2-preview/cm-test-topology.png":::

#### <a name="log-queries-in-log-analytics"></a>Consigner les requêtes dans Log Analytics

Utilisez Log Analytics pour créer des vues personnalisées de vos données de supervision. Toutes les données affichées par l'interface utilisateur proviennent de Log Analytics. Vous pouvez analyser les données de manière interactive dans le référentiel. Mettez en corrélation les données provenant d'Agent Health ou d'autres solutions basées sur Log Analytics. Exportez les données vers Excel ou Power BI, ou créez un lien partageable.

#### <a name="metrics-in-azure-monitor"></a>Mesures dans Azure Monitor

Dans les moniteurs de connexion créés avant le lancement de la fonctionnalité Moniteur de connexion, les quatre métriques suivantes sont disponibles : % Probes Failed, AverageRoundtripMs, ChecksFailedPercent (Préversion) et RoundTripTimeMs (Préversion). Dans les moniteurs de connexion créés à partir de la fonctionnalité Moniteur de connexion, les données sont uniquement disponibles pour les métriques portant l’étiquette *(Préversion)* .

  :::image type="content" source="./media/connection-monitor-2-preview/monitor-metrics.png" alt-text="Capture d’écran illustrant les métriques dans Moniteur de connexion" lightbox="./media/connection-monitor-2-preview/monitor-metrics.png":::

Lorsque vous utilisez des métriques, définissez le type de ressource sur Microsoft.Network/networkWatchers/connectionMonitors

| Métrique | Nom complet | Unité | Type d’agrégation | Description | Dimensions |
| --- | --- | --- | --- | --- | --- |
| ProbesFailedPercent | % de sondes ayant échoué | Pourcentage | Average | Pourcentage de sondes de surveillance de connectivité ayant échoué | Aucune dimension |
| AverageRoundtripMs | Avg. Durée aller-retour (ms) | Millisecondes | Average | Durée moyenne des boucles réseau pour les sondes de surveillance de la connectivité envoyées entre la source et la destination. |             Aucune dimension |
| ChecksFailedPercent (préversion) | Pourcentage d’échecs de vérification (préversion) | Pourcentage | Average | Pourcentage de vérifications ayant échoué pour un test. | ConnectionMonitorResourceId <br>SourceAddress <br>SourceName <br>SourceResourceId <br>SourceType <br>Protocol <br>DestinationAddress <br>DestinationName <br>DestinationResourceId <br>DestinationType <br>DestinationPort <br>TestGroupName <br>TestConfigurationName <br>Région |
| RoundTripTimeMs (préversion) | Durée d’aller-retour (ms) (préversion) | Millisecondes | Average | Durée des boucles pour les vérifications envoyées entre la source et la destination. Cette valeur ne fait pas l'objet d'une moyenne. | ConnectionMonitorResourceId <br>SourceAddress <br>SourceName <br>SourceResourceId <br>SourceType <br>Protocol <br>DestinationAddress <br>DestinationName <br>DestinationResourceId <br>DestinationType <br>DestinationPort <br>TestGroupName <br>TestConfigurationName <br>Région |

#### <a name="metric-based-alerts-for-connection-monitor"></a>Alertes basées sur des métriques pour le Moniteur de connexion

Vous pouvez créer des alertes de métriques sur des moniteurs de connexion à l’aide des méthodes ci-dessous 

1. Dans Moniteur de connexion, lors de la création du moniteur de connexion [à l’aide du portail Azure](connection-monitor-preview-create-using-portal.md#) 
1. Dans Moniteur de connexion, à l’aide de « Configurer des alertes » dans le tableau de bord 
1. À partir d’Azure Monitor, pour créer une alerte dans Azure Monitor : 
    1. Choisissez la ressource de moniteur de connexion que vous avez créée dans Moniteur de connexion.
    1. Assurez-vous que **Métrique** apparaît comme type de signal pour le moniteur de connexion.
    1. Dans **Ajouter une condition**, pour le **Nom du signal**, sélectionnez **ChecksFailedPercent (préversion)** ou **RoundTripTimeMs (préversion)** .
    1. Pour le **Type de signal**, choisissez **Métriques**. Par exemple, sélectionnez **ChecksFailedPercent (préversion)** .
    1. Toutes les dimensions de la métrique sont répertoriées. Choisissez le nom et la valeur de dimension. Par exemple, sélectionnez **Adresse source**, puis entrez l'adresse IP d'une source dans votre moniteur de connexion.
    1. Dans **Logique d'alerte**, renseignez les informations suivantes :
        * **Type de condition** : **Statique**.
        * **Condition** et **Seuil**.
        * **Granularité d'agrégation et fréquence d'évaluation** : Moniteur de connexion met à jour les données toutes les minutes.
    1. Dans **Actions**, choisissez votre groupe d'actions.
    1. Indiquez les détails de l'alerte.
    1. Créez la règle d’alerte

  :::image type="content" source="./media/connection-monitor-2-preview/mdm-alerts.jpg" alt-text="Capture d’écran montrant la zone Créer une règle dans Azure Monitor. L’adresse source et le nom du point de terminaison source sont mis en surbrillance" lightbox="./media/connection-monitor-2-preview/mdm-alerts.jpg":::

## <a name="diagnose-issues-in-your-network"></a>Diagnostiquer les problèmes de votre réseau

Moniteur de connexion vous aide à diagnostiquer les problèmes liés à votre moniteur de connexion et à votre réseau. Les problèmes liés à votre réseau hybride sont détectés par les agents Log Analytics que vous avez précédemment installés. Les problèmes liés à Azure sont détectés par l'extension Network Watcher. 

Vous pouvez afficher les problèmes liés au réseau Azure dans la topologie du réseau.

Pour les réseaux dont les sources sont des machines virtuelles locales, les problèmes suivants peuvent être détectés :

* La requête a expiré.
* Point de terminaison non résolu par le DNS : temporaire ou persistant. URL non valide.
* Hôte introuvable.
* Impossibilité pour la source de se connecter à la destination. Cible inaccessible via ICMP.
* Problèmes liés au certificat : 
    * Certificat client requis pour authentifier l'agent. 
    * La liste de réadressage des certificats n'est pas accessible. 
    * Le nom d'hôte du point de terminaison ne correspond pas au sujet ou à l'autre nom du sujet du certificat. 
    * Le certificat racine ne figure pas dans le magasin des Autorités de certification de confiance de l'ordinateur local de la source. 
    * Le certificat SSL est arrivé à expiration, n'est pas valide, a été révoqué ou n'est pas compatible.

Pour les réseaux dont les sources sont des machines virtuelles Azure, les problèmes suivants peuvent être détectés :

* Problèmes liés à l'agent :
    * Agent arrêté.
    * Échec de la résolution DNS.
    * Aucune application ou aucun écouteur n'écoute sur le port de destination.
    * Impossible d'ouvrir le socket.
* Problèmes liés à l'état de la machine virtuelle : 
    * Démarrage en cours
    * En cours d’arrêt
    * Arrêté
    * Libération
    * Libéré
    * Redémarrage
    * Non alloué
* L'entrée de table ARP est manquante.
* Le trafic a été bloqué en raison de problèmes liés au pare-feu local ou aux règles NSG.
* Problèmes liés à la passerelle de réseau virtuel : 
    * Itinéraires manquants.
    * Le tunnel entre deux passerelles est déconnecté ou manquant.
    * Le tunnel n'a pas trouvé la deuxième passerelle.
    * Aucune information de peering n'a été trouvée.
* L'itinéraire était manquant dans Microsoft Edge.
* Le trafic a été interrompu à cause d'itinéraires système ou de règles UDR.
* Le protocole BGP n'est pas activé sur la connexion à la passerelle.
* La sonde DIP est en panne sur l'équilibreur de charge.

## <a name="next-steps"></a>Étapes suivantes
    
   * Découvrez [comment créer une instance Moniteur de connexion à l’aide du portail Azure](./connection-monitor-create-using-portal.md)  
   * Découvrez [comment créer une instance Moniteur de connexion à l’aide d’ARMClient](./connection-monitor-create-using-template.md)