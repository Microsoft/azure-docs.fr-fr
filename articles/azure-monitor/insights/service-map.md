---
title: Utiliser la solution Service Map dans Azure | Microsoft Docs
description: Service Map est une solution comprise dans Azure qui détecte automatiquement les composants d’application sur les systèmes Windows et Linux et mappe la communication entre les services. Cet article fournit des informations sur le déploiement de Service Map dans votre environnement et son utilisation dans divers scénarios.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/24/2019
ms.openlocfilehash: f7268f698dcc26dbe99b517c9dd4584be67c3a82
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91448455"
---
# <a name="using-service-map-solution-in-azure"></a>Utilisation de la solution Service Map dans Azure

La solution Service Map détecte automatiquement les composants d’application sur les systèmes Windows et Linux, et mappe la communication entre les services. Elle vous permet d’afficher vos serveurs comme vous les imaginez, en tant que systèmes interconnectés fournissant des services critiques. Elle affiche les connexions entre serveurs, les processus, la latence des connexions entrantes et sortantes, ainsi que les ports au sein de toute architecture TCP connectée, sans nécessiter de configuration autre que l’installation d’un agent.

Cet article décrit les détails de l’intégration et de l’utilisation de Service Map. Les conditions préalables de la solution sont les suivantes :

* Un espace de travail Log Analytics dans une [région prise en charge](vminsights-configure-workspace.md#supported-regions).

* L’[agent Log Analytics](vminsights-enable-overview.md#agents) installé sur l’ordinateur Windows ou le serveur Linux connecté à l’espace de travail avec lequel vous avez activé la solution.

* L’[agent Dependency](vminsights-enable-overview.md#agents) installé sur l’ordinateur Windows ou le serveur Linux.

>[!NOTE]
>Si vous avez déjà déployé Service Map, vous pouvez maintenant visualiser vos mappages dans Azure Monitor pour les machines virtuelles, incluant des fonctionnalités supplémentaires pour la supervision de l’intégrité et des performances de celles-ci. Pour en savoir plus, consultez [Présentation d’Azure Monitor pour machines virtuelles](./vminsights-overview.md). Pour en savoir plus sur les différences entre la solution Service Map et la fonctionnalité de carte Azure Monitor pour machines virtuelles, consultez les [questions fréquentes (FAQ)](../faq.md#azure-monitor-for-vms).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="enable-service-map"></a>Activer Service Map

1. Activez la solution Service Map depuis la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.ServiceMapOMS?tab=Overview) ou en procédant de la manière décrite dans [Ajouter des solutions de supervision à partir de la galerie Solutions](solutions.md).
1. [Installez l’agent de dépendances sur Windows](./vminsights-enable-hybrid.md#install-the-dependency-agent-on-windows) ou [installez l’agent de dépendances sur Linux](./vminsights-enable-hybrid.md#install-the-dependency-agent-on-linux) sur chaque ordinateur sur lequel vous souhaitez obtenir des données. L’agent de dépendances peut surveiller les connexions aux voisins immédiats ; par conséquent, vous n’avez pas forcément besoin d’un agent sur chaque ordinateur.

Accédez à Service Map dans le portail Azure à partir de votre espace de travail Log Analytics, puis sélectionnez l’option **Solutions** dans le volet gauche.<br><br> ![Sélectionnez l’option Solutions dans l’espace de travail](./media/service-map/select-solution-from-workspace.png).<br> Dans la liste des solutions, sélectionnez **ServiceMap(workspaceName)** et dans la page de présentation de la solution Service Map, cliquez sur la vignette de résumé Service Map.<br><br> ![Vignette de résumé Service Map](./media/service-map/service-map-summary-tile.png).

## <a name="use-cases-make-your-it-processes-dependency-aware"></a>Cas d’utilisation : Intégrer la dépendance dans vos processus informatiques

### <a name="discovery"></a>Découverte

Service Map crée automatiquement une carte de référence commune des dépendances entre vos serveurs, les processus et les services tiers. La solution découvre et mappe toutes les dépendances TCP, en identifiant les connexions inattendues, les systèmes tiers distants dont vous dépendez et les dépendances envers les zones sombres traditionnelles de votre réseau comme Active Directory. Service Map détecte les échecs de connexion réseau que vos systèmes gérés tentent d’établir, vous aidant ainsi à identifier d’éventuels problèmes de réseau, interruptions de service et autres configurations de serveur inappropriées.

### <a name="incident-management"></a>Gestion des incidents

Service Map vous montre comment les systèmes sont connectés et ont une incidence les uns sur les autres, ce qui vous évide de devoir isoler vous-même le problème. Outre l’identification des échecs de connexion, la solution aide à identifier les équilibreurs de charge mal configurés, des charges imprévues ou excessives sur des services critiques, ainsi que des clients non autorisés tels que des ordinateurs de développeurs qui communiquent avec les systèmes de production. En utilisant des workflows intégrés dans la solution Change Tracking, vous pouvez également voir si un événement de modification sur un ordinateur ou service principal explique la cause racine d’un incident.

### <a name="migration-assurance"></a>Garantie d’une migration réussie

La solution Service Map vous permet de planifier, d’accélérer et de valider efficacement les migrations vers Azure pour vous assurer que rien n’est oublié et vous prémunir contre toute panne surprise. Vous pouvez découvrir tous les systèmes interdépendants qui doivent migrer en même temps, évaluer la capacité et la configuration du système et déterminer si un système en cours d’exécution continue de servir les utilisateurs ou si une mise hors service peut être envisagée au lieu de la migration. Une fois la migration terminée, vous pouvez vérifier la charge et l’identité du client pour contrôler que les systèmes de test et les clients se connectent. Si vos définitions de pare-feu et de planification de sous-réseau rencontrent des problèmes, les connexions ayant échoué dans les mappages Service Map vous dirigent vers les systèmes nécessitant une connectivité.

### <a name="business-continuity"></a>Continuité de l’activité

Si vous utilisez Azure Site Recovery et que vous avez besoin d’aide pour définir la séquence de récupération pour votre environnement d’application, Service Map peut automatiquement vous indiquer comment les systèmes s’appuient les uns sur les autres pour s’assurer que votre plan de récupération est fiable. En choisissant un serveur ou un groupe critiques, et en affichant ses clients, vous pouvez identifier les systèmes frontaux à récupérer une fois le serveur restauré et disponible. Inversement, en examinant les dépendances back-end des serveurs critiques, vous pouvez identifier les systèmes à récupérer avant la restauration de vos systèmes principaux.

### <a name="patch-management"></a>Gestion des correctifs

Service Map améliore l’utilisation de l’évaluation des mises à jour système en vous montrant quels autres équipes et serveurs dépendent de votre service. Vous pouvez ainsi notifier ces derniers à l’avance avant d’interrompre vos systèmes pour la mise à jour corrective. Service Map améliore aussi la gestion des correctifs en vous montrant si vos services sont disponibles et connectés correctement une fois que la mise à jour corrective a été appliquée et qu’ils ont redémarré.

## <a name="mapping-overview"></a>Vue d’ensemble du mappage

Les agents Service Map rassemblent des informations sur tous les processus connectés via TCP sur le serveur où ils sont installés ainsi que des détails sur les connexions entrantes et sortantes pour chaque processus.

Dans la liste du volet gauche, vous pouvez sélectionner les ordinateurs ou les groupes disposant d’agents Service Map afin de visualiser leurs dépendances sur une plage de temps spécifiée. Les cartes de dépendance d’ordinateur ciblent un ordinateur spécifique, et montrent tous les ordinateurs qui sont des clients ou serveurs TCP directs de cet ordinateur.  Les cartes de groupe d’ordinateurs montrent des ensembles de serveurs et leurs dépendances.

![Vue d’ensemble de Service Map](media/service-map/service-map-overview.png)

Les ordinateurs peuvent être développés sur le mappage de manière à afficher les processus et équipes processus en cours d’exécution avec des connexions réseau actives pendant la période sélectionnée. Quand un ordinateur distant doté d’un agent Service Map est développé pour afficher des détails de processus, seuls les processus qui communiquent avec l’ordinateur ciblé sont affichés. Le nombre d’ordinateurs frontaux sans agent qui se connectent à l’ordinateur ciblé est indiqué à gauche des processus auxquels ils se connectent. Si l’ordinateur ciblé se connecte à un ordinateur principal dépourvu d’agent, ce serveur principal est inclus dans un groupe de ports du serveur, avec les autres connexions au même numéro de port.

Par défaut, les mappages Service Map affichent les 30 dernières minutes des informations de dépendance. Les contrôles de temps en haut à gauche permettent d’interroger des cartes pour obtenir des plages horaires historiques de jusqu’à une heure afin de voir l’aspect passé des dépendances (par exemple, pendant un incident ou avant une modification). Les données Service Map sont stockées pendant 30 jours dans les espaces de travail payants et pendant 7 jours dans les espaces de travail gratuits.

## <a name="status-badges-and-border-coloring"></a>Badges d’état et couleur de bordure

En bas de chaque serveur dans le mappage, une liste de badges état affiche des informations sur l’état du serveur. Les badges indiquent qu’il existe des informations pertinentes concernant le serveur provenant d’une des intégrations de solutions. En cliquant sur un badge, vous accédez directement aux détails de l’état dans le volet droit. Les badges d’état actuellement disponibles sont Alertes, Service Desk, Modifications, Sécurité et Mises à jour.

Selon la gravité des badges d’état, les bordures de nœud de machine peuvent être colorées en rouge (critique), en jaune (avertissement) ou en bleu (informations). La couleur est celle du badge d’état dont la gravité est la plus critique. Une bordure de couleur grise indique un nœud dépourvu d’indicateur d’état.

![Badges d’état](media/service-map/status-badges.png)

## <a name="process-groups"></a>Équipes processus

Les équipes processus combinent les processus associés à un produit ou service commun en une équipe processus.  Quand un nœud d’ordinateur est développé, il affiche les processus autonomes ainsi que les équipes processus.  Si des connexions entrantes et sortantes d’un processus dans une équipe processus échouent, la connexion est indiquée comme ayant échoué pour l’ensemble de l’équipe processus.

## <a name="machine-groups"></a>Groupes de machines

Les groupes de machines permettent de voir les cartes centrées autour d’un ensemble de serveurs, pas seulement d’un seul. Vous pouvez ainsi voir tous les membres d’un cluster d’applications ou de serveurs multiniveau dans une seule carte.

Les utilisateurs sélectionnent les serveurs appartenant à un groupe, puis choisissent un nom pour celui-ci.  Vous pouvez ensuite choisir d’afficher le groupe avec l’ensemble de ses processus et connexions, ou de l’afficher uniquement avec les processus et connexions directement liés aux autres membres du groupe.

![Groupe de machines](media/service-map/machine-group.png)

### <a name="creating-a-machine-group"></a>Création d’un groupe de machines

Pour créer un groupe, sélectionnez la ou les machines souhaitées dans la liste des machines, puis cliquez sur **Ajouter au groupe**.

![Créer un groupe](media/service-map/machine-groups-create.png)

Vous pouvez alors choisir **Créer un nouveau** et attribuer un nom au groupe.

![Nommer un groupe](media/service-map/machine-groups-name.png)

>[!NOTE]
>Les groupes de machines sont limités à 10 serveurs.

### <a name="viewing-a-group"></a>Affichage d’un groupe

Une fois que vous avez créé des groupes, vous pouvez les voir en cliquant sur l’onglet Groupes.

![Onglet Groupes](media/service-map/machine-groups-tab.png)

Sélectionnez ensuite le nom du groupe de machines pour afficher la carte de celui-ci.
![Groupe de machines](media/service-map/machine-group.png) Les machines appartenant au groupe sont entourées d’un contour blanc sur la carte.

Le développement du groupe de machines permet d’afficher la liste des ordinateurs qui le composent.

![Ordinateurs d’un groupe de machines](media/service-map/machine-groups-machines.png)

### <a name="filter-by-processes"></a>Filtrer par processus

Vous pouvez basculer la vue de carte entre l’affichage de tous les processus et connexions au sein du groupe, ou uniquement de ceux qui sont directement liés au groupe de machines.  La vue par défaut affiche tous les processus.  Vous pouvez modifier la vue en cliquant sur l’icône de filtre au-dessus de la carte.

![Groupe de filtres](media/service-map/machine-groups-filter.png)

Lorsque l’option **Tous les processus** est activée, la carte inclut l’ensemble des processus et connexions sur chaque machine du groupe.

![Tous les processus d’un groupe de machines](media/service-map/machine-groups-all.png)

Si vous modifiez l’affichage pour voir uniquement les **Processus connectés au groupe**, la carte est rétrécie de façon à n’afficher que les processus et connexions directement liés aux autres machines du groupe, créant ainsi un affichage simplifié.

![Processus filtrés d’un groupe de machines](media/service-map/machine-groups-filtered.png)
 
### <a name="adding-machines-to-a-group"></a>Ajout de machines à un groupe

Pour ajouter des machines à un groupe, cochez les cases en regard des machines souhaitées, puis cliquez sur **Ajouter au groupe**.  Ensuite, sélectionnez le groupe auquel vous souhaitez ajouter les machines.
 
### <a name="removing-machines-from-a-group"></a>Suppression de machines d’un groupe

Dans la liste des groupes, développez le nom du groupe pour afficher les machines constituant celui-ci.  Cliquez ensuite sur le menu représenté par des points de suspension en regard de la machine à supprimer, puis choisissez **Supprimer**.

![Supprimer une machine d’un groupe](media/service-map/machine-groups-remove.png)

### <a name="removing-or-renaming-a-group"></a>Suppression ou changement de nom d’un groupe

Cliquez sur le menu représenté par des points de suspension en regard du nom dans la liste de groupes.

![Menu de groupe de machines](media/service-map/machine-groups-menu.png)


## <a name="role-icons"></a>Icônes de rôle

Certains processus jouent des rôles particuliers sur les machines : serveurs web, serveurs d’applications, base de données, etc. Service Map annote les zones de processus et de machine avec des icônes de rôle pour identifier en un coup d’œil le rôle que joue un processus ou un serveur.

| Icône de rôle | Description |
|:--|:--|
| ![Serveur web](media/service-map/role-web-server.png) | Serveur web |
| ![App Server](media/service-map/role-application-server.png) | Serveur d’applications |
| ![Serveur de base de données](media/service-map/role-database.png) | Serveur de base de données |
| ![Serveur LDAP](media/service-map/role-ldap.png) | Serveur LDAP |
| ![Serveur SMB](media/service-map/role-smb.png) | Serveur SMB |

![Icônes de rôle](media/service-map/role-icons.png)


## <a name="failed-connections"></a>Connexions ayant échoué

Les connexions qui ont échoué sont affichées dans les cartes de type Service Map des processus et machines, avec une ligne rouge en pointillés indiquant qu’un système client ne parvient pas à atteindre un processus ou un port. Les connexions ayant échoué sont signalées par n’importe quel système ayant un agentService Map déployé si ce système est celui qui a tenté d’établir de telles connexions. La solution Service Map mesure ce processus en observant les sockets TCP qui ne parviennent pas à établir une connexion. Cet échec peut être dû à un pare-feu, à une configuration incorrecte du client ou du serveur, ou à l’indisponibilité d’un service distant.

![Capture d'écran d'une partie d'une solution Service Map dans laquelle une ligne rouge en pointillés est mise en surbrillance pour signaler que la connexion entre le processus backup.pl et le port 4475 a échoué.](media/service-map/failed-connections.png)

Le fait de comprendre les connexions ayant échoué peut faciliter le dépannage, la validation de la migration, l’analyse de la sécurité et la compréhension globale de l’architecture. Les échecs de connexion sont parfois sans incidence, mais ils dénotent souvent directement un problème tel qu’un environnement de basculement devenant subitement inaccessible ou deux couches Application incapables de communiquer après une migration vers le cloud.

## <a name="client-groups"></a>Groupes de clients

Les groupes de clients sont des zones sur la carte qui représentent des ordinateurs clients n’ayant pas d’agents de dépendance. Un groupe de clients représente les clients d’un processus ou d’une machine.

![Groupes de clients](media/service-map/client-groups.png)

Pour afficher les adresses IP des serveurs d’un groupe de clients, sélectionnez le groupe. Le contenu du groupe est affiché dans le volet **Propriétés du groupe de clients**.

![Propriétés du groupe de clients](media/service-map/client-group-properties.png)

## <a name="server-port-groups"></a>Groupes de ports du serveur

Les groupes de ports du serveur sont des zones qui représentent les ports de chaque serveur n’ayant pas d’agents de dépendance. La zone indique le port du serveur ainsi que le nombre de serveurs connectés à ce port. Développez la zone pour afficher chaque serveur et ses connexions. S’il n’y a qu’un serveur dans la zone, son nom ou son adresse IP s’affiche.

![Groupes de ports du serveur](media/service-map/server-port-groups.png)

## <a name="context-menu"></a>Menu contextuel

Un clic sur les points de suspension (...) en haut à droite d’un serveur permet d’afficher le menu contextuel de ce serveur.

![Capture d'écran représentant le menu contextuel d'un serveur dans Service Map. Le menu contient les options Charger la carte des serveurs et Afficher les self-links.](media/service-map/context-menu.png)

### <a name="load-server-map"></a>Charger la carte des serveurs

Un clic sur l’option **Charger la carte des serveurs** permet d’accéder à une nouvelle carte présentant le serveur sélectionné en tant que nouvelle machine ciblée.

### <a name="show-self-links"></a>Afficher les self-links

Un clic sur l’option **Afficher les self-links** permet de redessiner le nœud du serveur incluant tous les self-links qui représentent des connexions TCP commençant et se terminant sur des processus au sein du serveur. Si des self-links sont affichés, la commande de menu devient **Masquer les self-links**, de sorte que vous pouvez les désactiver.

## <a name="computer-summary"></a>Récapitulatif de la machine

Le volet **Récapitulatif d’une machine** contient une vue d’ensemble du système d’exploitation, du nombre de dépendances et des données provenant d’autres solutions pour un serveur. Ces données ont trait aux métriques de performance, aux tickets de Service Desk, au suivi des modifications, à la sécurité et aux mises à jour.

![Volet Récapitulatif d’une machine](media/service-map/machine-summary.png)

## <a name="computer-and-process-properties"></a>Propriétés des ordinateurs et processus

Lorsque vous naviguez dans une carte de type Service Map, vous pouvez sélectionner des machines et des processus afin d’obtenir un contexte supplémentaire concernant leurs propriétés. Les ordinateurs fournissent des informations sur le nom DNS, les adresses IPv4, la capacité en termes de processeur et de mémoire, le type de machine virtuelle, la version du système d’exploitation, l’heure du dernier redémarrage et les ID de leurs agents OMS et Service Map.

![Volet Propriétés d’une machine](media/service-map/machine-properties.png)

Vous pouvez collecter les détails du processus à partir des métadonnées du système d’exploitation relatives à l’exécution des processus, qui comprennent les nom et description du processus, les nom et domaine de l’utilisateur (sur Windows), les noms de la société et du produit, la version du produit, le répertoire de travail, la ligne de commande et l’heure de début du processus.

![Volet Propriétés du processus](media/service-map/process-properties.png)

Le volet **Récapitulatif du processus** fournit des informations supplémentaires sur la connectivité du processus, qui comprennent les ports liés, les connexions entrantes et sortantes ainsi que les connexions ayant échoué.

![Volet Récapitulatif du processus](media/service-map/process-summary.png)

## <a name="alerts-integration"></a>Intégration des alertes

Service Map intègre les alertes Azure pour afficher les alertes déclenchées pour le serveur sélectionné pendant la période sélectionnée. Le serveur affiche une icône s’il existe des alertes, et le volet **Alertes de la machine** répertorie les alertes.

![Volet Alertes d’une machine](media/service-map/machine-alerts.png)

Pour permettre à la solution Service Map d’afficher les alertes pertinentes, créez une règle d’alerte qui se déclenche pour un ordinateur spécifique. Pour créer des alertes appropriées :
- Incluez une clause de regroupement par ordinateur (par exemple, **by Computer interval 1minute**).
- Choisissez d’alerter des mesures de métriques.

## <a name="log-events-integration"></a>Intégration des événements du journal

Service Map intègre la recherche dans les journaux pour afficher un nombre de tous les événements du journal disponibles pour le serveur sélectionné au cours de la période sélectionnée. Vous pouvez cliquer sur n’importe quelle ligne dans la liste des événements pour passer à la recherche dans les journaux et consulter les événements du journal individuel.

![Volet Journaux d’événements d’une machine](media/service-map/log-events.png)

## <a name="service-desk-integration"></a>Intégration du service d’assistance

L’intégration de Service Map au connecteur de gestion des services informatiques est automatique quand les deux solutions sont activées et configurées dans votre espace de travail Log Analytics. L’intégration dans Service Map est libellée « Service Desk ». Pour plus d’informations, voir [Gérer de manière centralisée les éléments de travail ITSM à l’aide d’IT Service Management Connector (version préliminaire)](../platform/itsmc-overview.md).

Le volet **Service Desk d’une machine** affiche la liste de tous les événements d’IT Service Management survenus pour le serveur sélectionné pendant la période sélectionnée. Le serveur affiche une icône s’il existe des éléments en cours, et le volet Service Desk de la machine affiche la liste de ceux-ci.

![Volet Service Desk d’une machine](media/service-map/service-desk.png)

Pour ouvrir l’élément dans votre solution ITSM connectée, cliquez sur **Afficher l’élément de travail**.

Pour afficher les détails de l’élément dans la recherche, cliquez sur **Recherche dans les journaux**.
Les métriques de connexion sont écrites dans deux nouvelles table dans Log Analytics. 

## <a name="change-tracking-integration"></a>Intégration de Change Tracking

L’intégration de Service Map à Change Tracking est automatique quand les deux solutions sont activées et configurées dans votre espace de travail Log Analytics.

Le volet **Change Tracking d’une machine** affiche la liste de toutes les modifications, les plus récentes en tête, ainsi qu’un lien pour consulter des détails supplémentaires dans Recherche dans les journaux.

![Capture d'écran du volet Change Tracking d'une machine dans Service Map.](media/service-map/change-tracking.png)

L’illustration suivante est une vue détaillée d’un événement ModificationDeConfiguration que vous pourriez voir après avoir sélectionné **Afficher dans Log Analytics**.

![Événement ModificationDeConfiguration](media/service-map/configuration-change-event-01.png)

## <a name="performance-integration"></a>Intégration des performances

Le volet **Performances d’une machine** affiche des métriques de performance standard pour le serveur sélectionné. Ces métriques incluent l’utilisation du processeur, l’utilisation de la mémoire, les octets réseau envoyés et reçus, et la liste des principaux processus classée par octets réseau envoyés et reçus.

![Volet Performances d’une machine](media/service-map/machine-performance.png)

Pour afficher les données de performances, vous pouvez avoir besoin d’[activer les compteurs de performances Log Analytics appropriés](../platform/data-sources-performance-counters.md).  Les compteurs que vous souhaitez activer :

Windows :
- Processeur(*)\\ % de temps processeur
- Mémoire\\ % d’octets validés en cours d’utilisation
- Carte réseau(*)\\Octets envoyés/s
- Carte réseau(*)\\Octets reçus/s

Linux :
- Processeur(*)\\ % de temps processeur
- Mémoire(*)\\ % de mémoire utilisée
- Carte réseau(*)\\Octets envoyés/s
- Carte réseau(*)\\Octets reçus/s

Pour obtenir les données de performances réseau, vous devez également activer la solution Wire Data 2.0 dans votre espace de travail.
 
## <a name="security-integration"></a>Intégration de la sécurité

L’intégration de Service Map à Security and Audit est automatique quand les deux solutions sont activées et configurées dans votre espace de travail Log Analytics.

Le volet **Sécurité de la machine** présente les données de la solution Security and Audit pour le serveur sélectionné. Le volet affiche un récapitulatif de tous les problèmes de sécurité en attente pour le serveur pendant la période sélectionnée. Un clic sur un problème de sécurité permet d’accéder à Recherche dans les journaux pour plus d’informations concernant ce problème.

![Volet Sécurité d’une machine](media/service-map/machine-security.png)

## <a name="updates-integration"></a>Intégration des mises à jour

L’intégration de Service Map à Update Management est automatique quand les deux solutions sont activées et configurées dans votre espace de travail Log Analytics.

Le volet **Mises à jour de la machine** affiche les données de la solution Update Management pour le serveur sélectionné. Le volet affiche un récapitulatif de toutes les mises à jour manquantes pour le serveur pendant la période sélectionnée.

![Capture d'écran du volet Mises à jour de la machine dans Service Map.](media/service-map/machine-updates.png)

## <a name="log-analytics-records"></a>Enregistrements Log Analytics

Les données d’inventaire des ordinateurs et processus de la solution Service Map sont disponibles pour effectuer une [recherche](../log-query/log-query-overview.md) dans Log Analytics. Vous pouvez appliquer ces données à divers scénarios tels que la planification de la migration, l’analyse de la capacité, la détection et la résolution de problèmes de performances à la demande.

Un enregistrement par heure est généré pour chaque processus et ordinateur, en plus des enregistrements générés quand un processus ou ordinateur démarre ou est intégré à la solution Service Map. Les propriétés de ces enregistrements sont décrites dans les tableaux suivants. Les champs et les valeurs des événements ServiceMapComputer_CL sont mappés aux champs de la ressource Machine dans l’API Azure Resource Manager ServiceMap. Les champs et les valeurs des événements ServiceMapProcess_CL sont mappés aux champs de la ressource Processus dans l’API Azure Resource Manager ServiceMap. Le champ ResourceName_s correspond au champ de nom dans la ressource Azure Resource Manager correspondante. 

>[!NOTE]
>À mesure que les fonctionnalités la solution Service Map s’étofferont, il se peut que ces champs soient modifiés.

Il existe des propriétés générées en interne que vous pouvez utiliser pour identifier les ordinateurs et processus uniques :

- Ordinateur : Utilisez *ResourceId* ou *ResourceName_s* pour identifier de façon unique un ordinateur au sein d’un espace de travail Log Analytics.
- Processus : Utilisez *ResourceId* pour identifier de façon unique un processus au sein d’un espace de travail Log Analytics. *ResourceName_s* est unique dans le contexte de l’ordinateur sur lequel le processus est en cours d’exécution (MachineResourceName_s). 

Étant donné que plusieurs enregistrements peuvent exister pour un processus et un ordinateur donnés au cours d’une période spécifique, les requêtes peuvent renvoyer plusieurs enregistrements pour un même ordinateur ou processus. Pour inclure uniquement l’enregistrement le plus récent, ajoutez "| dedup ResourceId" à la requête.

### <a name="connections"></a>Connexions

Les métriques de connexion sont écrites dans une nouvelle table dans Log Analytics (VMConnection). Cette table fournit des informations sur les connexions relatives à une machine (entrantes et sortantes). Les métriques de connexion sont également exposées avec des API offrant le moyen d’obtenir une métrique spécifique dans une fenêtre de temps.  Les connexions TCP qui résultent de l’acceptation sur un socket d’écoute sont entrantes, alors que celles créées en se connectant à une adresse IP et à un port donnés sont sortantes. La direction d’une connexion est représentée par la propriété Direction, qui peut être définie avec la valeur **inbound** ou **outbound**. 

Les enregistrements inclus dans ces tables sont générés à partir des données rapportées par Dependency Agent. Chaque enregistrement représente une observation sur un intervalle de temps d’une minute. La propriété TimeGenerated indique le début de l’intervalle de temps. Chaque enregistrement contient des informations identifiant l’entité respective (à savoir la connexion ou le port), ainsi que des métriques associées à cette entité. Actuellement, seule l’activité réseau utilisant TCP sur IPv4 est rapportée.

À des fins de gestion des coûts et de la complexité, les enregistrements de connexion ne représentent pas des connexions réseau physiques individuelles. Plusieurs connexions réseau physiques sont groupées dans une connexion logique, qui est ensuite reflétée dans la table concernée.  Ainsi, les enregistrements de la table *VMConnection* représentent un regroupement logique et non les connexions physiques individuelles observées. Les connexions réseau physiques dont les attributs suivants présentent la même valeur au cours d’un intervalle d’une minute donné sont agrégées au sein d’un unique enregistrement logique dans *VMConnection*. 

| Propriété | Description |
|:--|:--|
| `Direction` |Direction de la connexion (valeur *inbound* ou *outbound*) |
| `Machine` |Nom de domaine complet (FQDN) de l’ordinateur |
| `Process` |Identité du processus ou de groupes de processus initialisant/acceptant la connexion |
| `SourceIp` |Adresse IP de la source |
| `DestinationIp` |Adresse IP de la destination |
| `DestinationPort` |Numéro de port de la destination |
| `Protocol` |Protocole utilisé pour la connexion.  La valeur est *tcp*. |

Pour prendre en compte l’impact du regroupement, les informations sur le nombre de connexions physiques groupées sont fournies dans les propriétés suivantes de l’enregistrement :

| Propriété | Description |
|:--|:--|
| `LinksEstablished` |Nombre de connexions réseau physiques qui ont été établies dans la fenêtre de temps de rapport |
| `LinksTerminated` |Nombre de connexions réseau physiques qui ont pris fin dans la fenêtre de temps de rapport |
| `LinksFailed` |Nombre de connexions réseau physiques qui ont échoué dans la fenêtre de temps de rapport. Actuellement, ces informations sont disponibles pour les connexions sortantes uniquement. |
| `LinksLive` |Nombre de connexions réseau physiques qui ont été ouvertes à la fin de la fenêtre de temps de rapport|

#### <a name="metrics"></a>Mesures

En plus des métriques concernant le nombre de connexions, des informations sur le volume de données envoyées et reçues sur une connexion logique ou un port réseau donné sont également incluses dans les propriétés suivantes de l’enregistrement :

| Propriété | Description |
|:--|:--|
| `BytesSent` |Nombre total d’octets qui ont été envoyés dans la fenêtre de temps de rapport |
| `BytesReceived` |Nombre total d’octets qui ont été reçus dans la fenêtre de temps de rapport |
| `Responses` |Nombre de réponses observées dans la fenêtre de temps de rapport. 
| `ResponseTimeMax` |Temps de réponse le plus long (en millisecondes) observé dans la fenêtre de temps de rapport.  En l’absence de valeur, la propriété est vide.|
| `ResponseTimeMin` |Temps de réponse le plus court (en millisecondes) observé dans la fenêtre de temps de rapport.  En l’absence de valeur, la propriété est vide.|
| `ResponseTimeSum` |Somme de tous les temps de réponse (en millisecondes) observés dans la fenêtre de temps de rapport.  En l’absence de valeur, la propriété est vide.|

Le troisième type de données rapportées est le temps de réponse : le temps passé par un appelant à attendre qu’une requête envoyée sur une connexion soit traitée par le point de terminaison distant et que ce dernier y réponde. Le temps de réponse rapporté est une estimation du temps de réponse réel du protocole d’application sous-jacent. Il est calculé à l’aide d’une méthode heuristique basée sur l’observation du flux de données entre la source et la destination d’une connexion réseau physique. Sur le plan conceptuel, il s’agit de la différence entre le moment auquel le dernier octet d’une requête quitte l’expéditeur et celui auquel le dernier octet de la réponse lui revient. Ces deux timestamps sont utilisés pour délimiter les événements de requête et de réponse sur une connexion physique donnée. La différence entre eux représente le temps de réponse d’une requête unique. 

Dans le cadre de la première version de cette fonctionnalité, notre algorithme est une approximation qui peut fonctionner de façon plus ou moins efficace selon le protocole d’application réel utilisé pour une connexion réseau donnée. Par exemple, l’approche actuelle fonctionne bien pour les protocoles de requête-réponse comme HTTP(S), mais ne fonctionne pas avec les protocoles unidirectionnels ou basés sur des files d’attente de messages.

Voici quelques points importants à prendre en compte :

1. Si un processus accepte des connexions sur la même adresse IP mais sur plusieurs interfaces réseau, un enregistrement distinct pour chaque interface est rapporté. 
2. Les enregistrements avec une adresse IP générique ne contiennent aucune activité. Ils sont inclus pour représenter le fait qu’un port de la machine est ouvert pour le trafic entrant.
3. Pour réduire le niveau de détail et le volume de données, les enregistrements avec IP générique sont omis en présence d’un enregistrement correspondant (pour le même processus, le même port et le même protocole) avec une adresse IP spécifique. Lorsqu’un enregistrement IP générique est omis, la propriété d’enregistrement IsWildcardBind avec l’adresse IP spécifique a la valeur « True » pour indiquer que le port est exposé sur toutes les interfaces de la machine à l’origine du rapport.
4. Pour les ports liés à une interface spécifique uniquement, IsWildcardBind a la valeur « False ».

#### <a name="naming-and-classification"></a>Affectation de noms et classification

Pour plus de commodité, l’adresse IP de l’extrémité distante d’une connexion est incluse dans la propriété RemoteIp. Pour les connexions entrantes, RemoteIp est identique à SourceIp, tandis que pour les connexions sortantes, elle est identique à DestinationIp. La propriété RemoteDnsCanonicalNames représente les noms canoniques DNS rapportés par la machine pour RemoteIp. Les propriétés RemoteDnsQuestions et RemoteClassification sont réservées en vue d’une utilisation ultérieure. 

#### <a name="geolocation"></a>Géolocalisation

*VMConnection* inclut également des informations de géolocalisation pour l’extrémité distante de chaque enregistrement de connexion dans les propriétés suivantes de l’enregistrement : 

| Propriété | Description |
|:--|:--|
| `RemoteCountry` |Nom du pays/de la région hébergeant RemoteIp.  Par exemple, *États-Unis* |
| `RemoteLatitude` |Latitude de géolocalisation.  Par exemple, *47,68*. |
| `RemoteLongitude` |Longitude de géolocalisation.  Par exemple, *-122,12*. |

#### <a name="malicious-ip"></a>Adresses IP malveillantes

Chaque propriété RemoteIp de la table *VMConnection* est comparée à un ensemble d’adresses IP associées à une activité malveillante connue. Si la propriété RemoteIp est identifiée comme malveillante, les propriétés suivantes de l’enregistrement sont renseignées (elles sont vides lorsque l’adresse IP n’est pas considérée comme malveillante) :

| Propriété | Description |
|:--|:--|
| `MaliciousIp` |Adresse RemoteIp |
| `IndicatorThreadType` |L’indicateur de menace détecté est l’une des valeurs suivantes :  *Botnet*, *C2*, *CryptoMining*, *Darknet*, *DDos*, *MaliciousUrl*, *Malware*, *Phishing*, *Proxy*, *PUA* ou *Watchlist*.   |
| `Description` |Description de la menace observée. |
| `TLPLevel` |Niveau de protocole TLP (Traffic Light Protocol) est réglé sur l’une des valeurs définies, *Blanc*, *Vert*, *Orange*, *Rouge*. |
| `Confidence` |Les valeurs sont comprises dans la fourchette *0 – 100*. |
| `Severity` |Les valeurs sont comprises dans la fourchette *0 – 5*,  dans laquelle *5* correspond à la gravité maximale et *0* à l’absence de gravité. La valeur par défaut est *3*.  |
| `FirstReportedDateTime` |La première fois que le fournisseur a déclaré l’indicateur. |
| `LastReportedDateTime` |La dernière fois que l’indicateur a été vu par Interflow. |
| `IsActive` |Indique les indicateurs sont désactivés avec la valeur *True* ou la valeur *False*. |
| `ReportReferenceLink` |Liens vers des rapports relatifs à un observable donné. |
| `AdditionalInformation` |Fournit des informations supplémentaires, s’il y a lieu, sur la menace observée. |

### <a name="servicemapcomputer_cl-records"></a>Enregistrements ServiceMapComputer_CL

Les enregistrements de type *ServiceMapComputer_CL* ont des données d’inventaire pour les serveurs incluant des agents Service Map. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|:--|:--|
| `Type` | *ServiceMapComputer_CL* |
| `SourceSystem` | *OpsManager* |
| `ResourceId` | Identificateur unique d’une machine au sein de l’espace de travail |
| `ResourceName_s` | Identificateur unique d’une machine au sein de l’espace de travail |
| `ComputerName_s` | Nom de domaine complet (FQDN) de l’ordinateur |
| `Ipv4Addresses_s` | Liste des adresses IPv4 du serveur |
| `Ipv6Addresses_s` | Liste des adresses IPv6 du serveur |
| `DnsNames_s` | Tableau de noms DNS |
| `OperatingSystemFamily_s` | Windows ou Linux |
| `OperatingSystemFullName_s` | Nom complet du système d’exploitation  |
| `Bitness_s` | Nombre de bits de la machine (32 bits ou 64 bits)  |
| `PhysicalMemory_d` | Mémoire physique en Mo |
| `Cpus_d` | Nombre de processeurs |
| `CpuSpeed_d` | Vitesse du processeur en MHz|
| `VirtualizationState_s` | *inconnu*, *physique*, *virtuel*, *hyperviseur* |
| `VirtualMachineType_s` | *hyper-v*, *vmware*, etc. |
| `VirtualMachineNativeMachineId_g` | ID de machine virtuelle assigné par son hyperviseur |
| `VirtualMachineName_s` | Nom de la machine virtuelle |
| `BootTime_t` | Temps de démarrage |

### <a name="servicemapprocess_cl-type-records"></a>Enregistrements de type ServiceMapProcess_CL

Les enregistrements de type *ServiceMapProcess_CL* ont des données d’inventaire pour les processus connectés à TCP sur des serveurs ayant des agents Service Map. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|:--|:--|
| `Type` | *ServiceMapProcess_CL* |
| `SourceSystem` | *OpsManager* |
| `ResourceId` | Identificateur unique d’un processus au sein de l’espace de travail |
| `ResourceName_s` | identificateur unique d’un processus au sein de la machine sur laquelle il s’exécute|
| `MachineResourceName_s` | Nom de ressource de la machine |
| `ExecutableName_s` | Nom de l’exécutable du processus |
| `StartTime_t` | Heure de début du pool de processus |
| `FirstPid_d` | Premier PID dans le pool de processus |
| `Description_s` | Description du processus |
| `CompanyName_s` | Nom de la société |
| `InternalName_s` | Nom interne |
| `ProductName_s` | Nom du produit |
| `ProductVersion_s` | Version du produit |
| `FileVersion_s` | Version du fichier |
| `CommandLine_s` | Ligne de commande |
| `ExecutablePath _s` | Chemin d’accès du fichier exécutable |
| `WorkingDirectory_s` | Le répertoire de travail |
| `UserName` | Compte sous lequel le processus s’exécute |
| `UserDomain` | Domaine sous lequel le processus s’exécute |

## <a name="sample-log-searches"></a>Exemples de recherches dans les journaux

### <a name="list-all-known-machines"></a>Liste de tous les ordinateurs connus

`ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId`

### <a name="list-the-physical-memory-capacity-of-all-managed-computers"></a>Répertorier la capacité de mémoire physique de tous les ordinateurs gérés

`ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId | project PhysicalMemory_d, ComputerName_s`

### <a name="list-computer-name-dns-ip-and-os"></a>Répertorier le nom de l’ordinateur, le DNS, l’adresse IP et le système d’exploitation.

`ServiceMapComputer_CL | summarize arg_max(TimeGenerated, *) by ResourceId | project ComputerName_s, OperatingSystemFullName_s, DnsNames_s, Ipv4Addresses_s`

### <a name="find-all-processes-with-sql-in-the-command-line"></a>Rechercher tous les processus contenant "sql" dans la ligne de commande

`ServiceMapProcess_CL | where CommandLine_s contains_cs "sql" | summarize arg_max(TimeGenerated, *) by ResourceId`

### <a name="find-a-machine-most-recent-record-by-resource-name"></a>Rechercher un ordinateur (enregistrement le plus récent) par nom de ressource

`search in (ServiceMapComputer_CL) "m-4b9c93f9-bc37-46df-b43c-899ba829e07b" | summarize arg_max(TimeGenerated, *) by ResourceId`

### <a name="find-a-machine-most-recent-record-by-ip-address"></a>Rechercher une machine (enregistrement le plus récent) par adresse IP

`search in (ServiceMapComputer_CL) "10.229.243.232" | summarize arg_max(TimeGenerated, *) by ResourceId`

### <a name="list-all-known-processes-on-a-specified-machine"></a>Répertorier tous les processus sur une machine spécifiée

`ServiceMapProcess_CL | where MachineResourceName_s == "m-559dbcd8-3130-454d-8d1d-f624e57961bc" | summarize arg_max(TimeGenerated, *) by ResourceId`

### <a name="list-all-computers-running-sql"></a>Répertorier tous les ordinateurs exécutant SQL

`ServiceMapComputer_CL | where ResourceName_s in ((search in (ServiceMapProcess_CL) "\*sql\*" | distinct MachineResourceName_s)) | distinct ComputerName_s`

### <a name="list-all-unique-product-versions-of-curl-in-my-datacenter"></a>Répertorier toutes les versions de produit uniques de CURL dans mon centre de données

`ServiceMapProcess_CL | where ExecutableName_s == "curl" | distinct ProductVersion_s`

### <a name="create-a-computer-group-of-all-computers-running-centos"></a>Créer un groupe de tous les ordinateurs exécutant CentOS

`ServiceMapComputer_CL | where OperatingSystemFullName_s contains_cs "CentOS" | distinct ComputerName_s`

### <a name="summarize-the-outbound-connections-from-a-group-of-machines"></a>Synthétiser les connexions sortantes d’un groupe de machines

```
// the machines of interest
let machines = datatable(m: string) ["m-82412a7a-6a32-45a9-a8d6-538354224a25"];
// map of ip to monitored machine in the environment
let ips=materialize(ServiceMapComputer_CL
| summarize ips=makeset(todynamic(Ipv4Addresses_s)) by MonitoredMachine=ResourceName_s
| mvexpand ips to typeof(string));
// all connections to/from the machines of interest
let out=materialize(VMConnection
| where Machine in (machines)
| summarize arg_max(TimeGenerated, *) by ConnectionId);
// connections to localhost augmented with RemoteMachine
let local=out
| where RemoteIp startswith "127."
| project ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine=Machine;
// connections not to localhost augmented with RemoteMachine
let remote=materialize(out
| where RemoteIp !startswith "127."
| join kind=leftouter (ips) on $left.RemoteIp == $right.ips
| summarize by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine=MonitoredMachine);
// the remote machines to/from which we have connections
let remoteMachines = remote | summarize by RemoteMachine;
// all augmented connections
(local)
| union (remote)
//Take all outbound records but only inbound records that come from either //unmonitored machines or monitored machines not in the set for which we are computing dependencies.
| where Direction == 'outbound' or (Direction == 'inbound' and RemoteMachine !in (machines))
| summarize by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol, RemoteIp, RemoteMachine
// identify the remote port
| extend RemotePort=iff(Direction == 'outbound', DestinationPort, 0)
// construct the join key we'll use to find a matching port
| extend JoinKey=strcat_delim(':', RemoteMachine, RemoteIp, RemotePort, Protocol)
// find a matching port
| join kind=leftouter (VMBoundPort 
| where Machine in (remoteMachines) 
| summarize arg_max(TimeGenerated, *) by PortId 
| extend JoinKey=strcat_delim(':', Machine, Ip, Port, Protocol)) on JoinKey
// aggregate the remote information
| summarize Remote=makeset(iff(isempty(RemoteMachine), todynamic('{}'), pack('Machine', RemoteMachine, 'Process', Process1, 'ProcessName', ProcessName1))) by ConnectionId, Direction, Machine, Process, ProcessName, SourceIp, DestinationIp, DestinationPort, Protocol
```

## <a name="rest-api"></a>API REST

L’ensemble des données relatives au serveur, au processus et aux dépendances dans Service Map est disponible via [l’API REST Carte de service](/rest/api/servicemap/).

## <a name="diagnostic-and-usage-data"></a>Données relatives aux diagnostics et à l’utilisation

Microsoft collecte automatiquement les données sur l’utilisation et les performances via votre utilisation du service Service Map. Microsoft utilise ces données pour fournir et améliorer la qualité, la sécurité et l’intégrité de la solution Service Map. Afin d’offrir des fonctionnalités de dépannage précises et efficaces, les données incluent des informations sur la configuration de votre logiciel, telles que le système d’exploitation et la version, l’adresse IP, le nom DNS et le nom de la station de travail. Microsoft ne collecte pas de nom, d’adresse ou d’autres coordonnées.

Pour plus d’informations sur l’utilisation et la collecte de données, voir la [Déclaration de confidentialité des Services en ligne de Microsoft](https://go.microsoft.com/fwlink/?LinkId=512132).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [recherches dans les journaux](../log-query/log-query-overview.md) dans Log Analytics pour récupérer les données collectées par la solution Service Map.

## <a name="troubleshooting"></a>Dépannage

Si vous rencontrez des problèmes d’installation ou d’exécution de Service Map, cette section vous aidera peut-être. Si vous n’arrivez toujours pas à les résoudre, contactez le Support Microsoft.

### <a name="dependency-agent-installation-problems"></a>Problèmes d’installation de l’agent de dépendances

#### <a name="installer-prompts-for-a-reboot"></a>Le programme d’installation invite l’utilisateur à redémarrer
L’agent de dépendances ne nécessite *généralement* pas de redémarrage à l’installation ou à la suppression. Toutefois, dans quelques rares cas, Windows Server nécessite un redémarrage pour poursuivre l’installation. Cela se produit quand une dépendance, généralement la bibliothèque redistribuable Microsoft Visual C++, exige un redémarrage en raison d’un fichier verrouillé.

#### <a name="message-unable-to-install-dependency-agent-visual-studio-runtime-libraries-failed-to-install-code--code_number-appears"></a>Le message « Impossible d’installer l’agent de dépendances : échec de l’installation des bibliothèques runtime de Visual Studio (code = [code_number]) » apparaît

L’agent de dépendances Microsoft repose sur les bibliothèques runtime de Microsoft Visual Studio. Vous recevrez un message si un problème est survenu lors de l’installation des bibliothèques. 

Les programmes d’installation des bibliothèques Runtime créent des journaux d’activité dans le dossier %LOCALAPPDATA%\temp. Le fichier se nomme `dd_vcredist_arch_yyyymmddhhmmss.log`, où *arch* correspond à `x86` ou `amd64` et *yyyymmddhhmmss* correspond à la date et l’heure (format 24 heures) de création du journal. Le journal fournit des détails sur le problème qui bloque l’installation.

Il peut être utile d’installer d’abord les [dernières bibliothèques runtime](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads).

Le tableau ci-dessous liste des numéros de code et des suggestions de résolutions.

| Code | Description | Résolution |
|:--|:--|:--|
| 0x17 | Le programme d’installation de la bibliothèque nécessite une mise à jour Windows qui n’a pas été installée. | Consultez le dernier journal du programme d’installation de la bibliothèque.<br><br>Si une référence à `Windows8.1-KB2999226-x64.msu` est suivie d’une ligne `Error 0x80240017: Failed to execute MSU package,`, vous n’avez pas les prérequis nécessaires à l’installation de KB2999226. Suivez les instructions de la section des prérequis dans l’article [Universal C Runtime sur Windows](https://support.microsoft.com/kb/2999226). Vous devrez peut-être exécuter Windows Update et redémarrer plusieurs fois afin d’installer les composants nécessaires.<br><br>Exécutez à nouveau le programme d’installation de l’agent de dépendances Microsoft. |

### <a name="post-installation-issues"></a>Problèmes après installation

#### <a name="server-doesnt-appear-in-service-map"></a>Le serveur n’apparaît pas dans Service Map

Si votre installation de l’agent de dépendances a réussi mais que vous ne voyez pas votre machine dans la solution Service Map :
* L’agent de dépendances est-il correctement installé ? Vous pouvez vous en assurer en vérifiant si le service est installé et en cours d’exécution.<br><br>
**Windows** : Recherchez le service nommé **Microsoft Dependency Agent**.
**Linux** : Recherchez **microsoft-dependency-agent** dans les processus en cours d’exécution.

* Êtes-vous sur le niveau gratuit de [Log Analytics](https://azure.microsoft.com/pricing/details/monitor/) ? Le plan Gratuit autorise jusqu’à cinq machines Service Map uniques. Toutes les machines suivantes n’apparaissent pas dans Service Map, même si les cinq précédentes n’envoient plus de données.

* Votre serveur envoie-t-il des données de journal et de performances aux journaux Azure Monitor ? Accédez à Azure Monitor\Logs et exécutez la requête suivante pour votre ordinateur : 

    ```kusto
    Usage | where Computer == "admdemo-appsvr" | summarize sum(Quantity), any(QuantityUnit) by DataType
    ```

Avez-vous reçu divers événements dans les résultats ? Les données sont-elles récentes ? Dans ce cas, votre agent Log Analytics fonctionne correctement et communique avec l’espace de travail. Si ce n’est pas le cas, vérifiez l’agent sur votre machine : [Résolution des problèmes de l’agent Log Analytics pour Windows](../platform/agent-windows-troubleshoot.md) ou [Résolution des problèmes de l’agent Log Analytics pour Linux](../platform/agent-linux-troubleshoot.md).

#### <a name="server-appears-in-service-map-but-has-no-processes"></a>Le serveur s’affiche dans Service Map, mais n’a aucun processus

Si vous voyez votre machine dans Service Map, mais qu’elle ne comporte aucune donnée de processus ou de connexion, cela indique que l’agent de dépendances est installé et en cours d’exécution, mais que le pilote du noyau ne s’est pas chargé. 

Vérifiez le `C:\Program Files\Microsoft Dependency Agent\logs\wrapper.log file` (Windows) ou `/var/opt/microsoft/dependency-agent/log/service.log file` (Linux). Les dernières lignes du fichier doivent indiquer la raison pour laquelle le noyau ne s’est pas chargé. Par exemple, le noyau n’est peut-être pas pris en charge sous Linux si vous l’avez mis à jour.

## <a name="suggestions"></a>Suggestions

Avez-vous des commentaires à nous transmettre à propos de Service Map ou de sa documentation ?  Si c’est le cas, sachez que notre page [User Voice](https://feedback.azure.com/forums/267889-log-analytics/category/184492-service-map) vous permet de nous suggérer des fonctionnalités ou de voter pour des suggestions en cours.