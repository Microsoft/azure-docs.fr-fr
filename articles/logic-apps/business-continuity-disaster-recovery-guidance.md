---
title: Continuité d’activité et reprise d’activité
description: Concevez votre stratégie de protection des données, récupérez rapidement après des événements perturbants, restaurez les ressources requises par les fonctions métier critiques et assurez la continuité d’activité pour Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 03/31/2020
ms.openlocfilehash: 0a36cb468ebcb77c0614bffd0afc392df3655c20
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89658207"
---
# <a name="business-continuity-and-disaster-recovery-for-azure-logic-apps"></a>Continuité d’activité et reprise d’activité pour Azure Logic Apps

Pour réduire l’impact et les effets des événements imprévisibles sur votre entreprise et vos clients, assurez-vous de disposer d’une solution de [*reprise d’activité* (DR)](https://en.wikipedia.org/wiki/Disaster_recovery) afin de pouvoir protéger les données, restaurer rapidement les ressources qui prennent en charge les fonctions métier critiques et maintenir l’exécution des opérations afin de préserver la [*continuité d’activité* (BC)](https://en.wikipedia.org/wiki/Business_continuity_planning). Par exemple, les interruptions peuvent inclure des pannes, des pertes de l’infrastructure sous-jacente ou des composants tels que les ressources de stockage, réseau ou de calcul, des défaillances d’application irrécupérables ou même une perte complète du centre de ressources. En disposant d’une solution de continuité d’activité et de reprise d’activité (BCDR), votre entreprise ou organisation peut faire face plus rapidement aux interruptions, planifiées ou non, et réduire les temps d’arrêt affectant vos clients.

Cet article fournit des conseils et des stratégies BCDR que vous pouvez appliquer lorsque vous créez des flux de travail automatisés à l’aide [d’Azure Logic Apps](../logic-apps/logic-apps-overview.md). Les flux de travail d’application logique vous aident à intégrer et orchestrer plus facilement les données entre les applications, les services cloud et les systèmes locaux en réduisant la quantité de code que vous devez écrire. Quand vous planifiez d’utiliser une solution BCDR, veillez à ne pas prendre en compte que vos applications logiques, mais également ces ressources Azure que vous utilisez avec vos applications logiques :

* Les [connexions](../connectors/apis-list.md) que vous créez à partir d’applications logiques vers d’autres applications, services et systèmes. Pour plus d’informations, consultez [Connexions aux ressources](#resource-connections), plus loin dans cette rubrique.

* Les [passerelles de données locales](../logic-apps/logic-apps-gateway-connection.md) qui sont des ressources Azure que vous créez et utilisez dans vos applications logiques pour accéder aux données dans des systèmes locaux. Chaque ressource de passerelle représente une [installation de passerelle de données](../logic-apps/logic-apps-gateway-install.md) distincte sur un ordinateur local. Pour plus d’informations, consultez [Passerelles de données locales](#on-premises-data-gateways), plus loin dans cette rubrique.

* Les [comptes d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) où vous définissez et stockez les artefacts que les applications logiques utilisent pour les scénarios [d’intégration d’entreprise B2B](../logic-apps/logic-apps-enterprise-integration-overview.md). Vous pouvez, par exemple [configurer la reprise d’activité entre des régions pour des comptes d’intégration](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md).

* Les [environnements de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) où vous créez des applications logiques qui s’exécutent dans une instance de runtime Logic Apps isolée au sein d’un réseau virtuel Azure. Ces applications logiques peuvent ensuite accéder aux ressources qui sont protégées derrière un pare-feu dans ce réseau virtuel.

<a name="primary-secondary-locations"></a>

## <a name="primary-and-secondary-locations"></a>Emplacements principal et secondaire

Chaque application logique doit spécifier l’emplacement que vous souhaitez utiliser pour le déploiement. Cet emplacement est soit une région publique dans un Azure multilocataire mondial, comme « USA Ouest », soit un environnement de service d’intégration (ISE) que vous avez précédemment créé et déployé dans un réseau virtuel Azure. L’exécution d’applications logiques dans un environnement ISE est identique à l’exécution d’applications logiques dans une région Azure mondiale, ce qui signifie que votre stratégie de reprise d’activité peut s’appliquer aux deux scénarios. Toutefois, les environnements ISE ont d’autres considérations, comme la configuration de l’accès aux ressources disponibles uniquement pour les environnements ISE.

> [!NOTE]
> Si votre application logique fonctionne également avec des artefacts B2B, comme des partenaires commerciaux, des contrats, des schémas, des mappages et des certificats, qui sont stockés dans un compte d’intégration, ce dernier et vos applications logiques doivent spécifier le même emplacement.

Cette stratégie de reprise d’activité est axée sur la configuration de votre application logique principale afin qu’elle [*bascule*](https://en.wikipedia.org/wiki/Failover) vers une application logique de secours ou de sauvegarde dans un autre emplacement où Azure Logic Apps est également disponible. Ainsi, si l’application logique principale subit des pertes, des interruptions ou des défaillances, l’application logique secondaire peut prendre le relais. Cette stratégie exige que votre application logique secondaire et les ressources dépendantes soient déjà déployées et prêtes dans l’emplacement secondaire.

Si vous suivez les bonnes pratiques DevOps, vous utilisez déjà des [modèles Azure Resource Manager](../azure-resource-manager/management/overview.md) pour définir et déployer vos applications logiques et leurs ressources dépendantes. Les modèles Resource Manager vous permettent d’utiliser une seule définition de déploiement, puis d’utiliser les fichiers de paramètres pour fournir les valeurs de configuration à utiliser pour chaque destination de déploiement. Cela signifie que vous pouvez déployer la même application logique dans différents environnements, par exemple, de développement, de test et de production. Vous pouvez également déployer la même application logique, qui prend en charge les stratégies de reprise d’activité utilisant des [régions appairées](../best-practices-availability-paired-regions.md), dans plusieurs régions Azure ou environnements ISE.

Pour la stratégie de basculement, vos applications logiques et emplacements doivent respecter les conditions suivantes :

* L’instance de l’application logique secondaire a accès aux mêmes applications, services et systèmes que l’instance de l’application logique principale.

* Les deux instances de l’application logique ont le même type d’hôte. Donc, soit les deux instances sont déployées dans des régions dans un Azure multilocataire mondial, soit les deux instances sont déployées dans des environnements ISE, ce qui permet à vos applications logiques d’accéder directement aux ressources d’un réseau virtuel Azure. Pour connaître les meilleures pratiques et obtenir plus d’informations sur les régions appairées, consultez [Continuité d’activité et reprise d’activité (BCDR) : régions couplées Azure](../best-practices-availability-paired-regions.md).

  Par exemple, les emplacements principal et secondaire doivent être des environnements ISE lorsque l’application logique principale s’exécute dans un environnement ISE et utilise des [connecteurs avec version ISE](../connectors/apis-list.md#ise-connectors), des actions HTTP pour appeler des ressources dans le réseau virtuel Azure, ou les deux. Dans ce scénario, votre application logique secondaire doit également avoir une configuration similaire à l’application logique principale dans l’emplacement secondaire.

  > [!NOTE]
  > Pour des scénarios plus avancés, vous pouvez combiner un Azure multilocataire et un environnement ISE en tant qu’emplacements. Toutefois, veillez à prendre en compte et à comprendre les [ différences entre l’exécution des applications logiques dans un environnement ISE et l’exécution dans un Azure multilocataire](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#difference).

* Si vous utilisez des environnements ISE, [assurez-vous qu’ils ont subi un scale-out ou qu’ils disposent d’une capacité suffisante](../logic-apps/ise-manage-integration-service-environment.md#add-capacity) pour gérer la charge.

#### <a name="example-multi-tenant-azure"></a>Exemple : Azure multilocataire

Cet exemple montre des instances d’application logique principale et secondaire, qui sont déployées dans des régions distinctes d’Azure multilocataire mondial pour ce scénario. Un seul [modèle Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) définit les deux instances d’application logique et les ressources dépendantes requises par ces applications logiques. Des fichiers de paramètres distincts spécifient les valeurs de configuration à utiliser pour chaque emplacement de déploiement :

![Instances d’application logique principale et secondaire dans des emplacements distincts](./media/business-continuity-disaster-recovery-guidance/primary-secondary-locations.png)

#### <a name="example-integration-service-environment"></a>Exemple : Environnement de service d’intégration (ISE)

Cet exemple montre les instances d’application logique principale et secondaire précédentes, mais déployées dans des environnements ISE distincts. Un seul modèle Resource Manager définit les deux instances d’application logique, les ressources dépendantes requises par ces applications logiques et les environnements ISE en tant qu’emplacements de déploiement. Des fichiers de paramètres distincts définissent les valeurs de configuration à utiliser pour le déploiement dans chaque emplacement :

![Applications logiques principale et secondaire dans différents emplacements](./media/business-continuity-disaster-recovery-guidance/primary-secondary-locations-ise.png)

<a name="resource-connections"></a>

## <a name="connections-to-resources"></a>Connexions aux ressources

Azure Logic Apps fournit des [déclencheurs et des actions intégrés, ainsi que des centaines de connecteurs managés](../connectors/apis-list.md) que votre application logique peut utiliser pour fonctionner avec d’autres applications, services, systèmes et autres ressources, comme les comptes de stockage Azure, les bases de données SQL Server, les comptes e-mail professionnels ou scolaires, etc. Si votre application logique a besoin d’accéder à ces ressources, vous créez des connexions qui authentifient l’accès à ces ressources. Chaque connexion est une ressource Azure distincte qui existe dans un emplacement spécifique et qui ne peut pas être utilisée par des ressources situées dans d’autres emplacements.

Pour votre stratégie de reprise d’activité, prenez en compte les emplacements où des ressources dépendantes existent par rapport à vos instances d’application logique :

* Votre instance principale et les ressources dépendantes existent dans des emplacements différents. Dans ce cas, votre instance secondaire peut se connecter aux mêmes ressources dépendantes ou points de terminaison. Toutefois, vous devez créer des connexions spécifiques pour votre instance secondaire. De cette façon, si votre emplacement principal devient indisponible, les connexions de l’emplacement secondaire ne sont pas affectées.

  Supposons, par exemple, que votre application logique principale se connecte à un service externe tel que Salesforce. En règle générale, la disponibilité et l’emplacement du service externe sont indépendants de la disponibilité de votre application logique. Dans ce cas, votre instance secondaire peut se connecter au même service, mais doit avoir sa propre connexion.

* Votre instance principale et les ressources dépendantes existent dans le même emplacement. Dans ce cas, les ressources dépendantes doivent avoir des sauvegardes ou des versions répliquées dans un autre emplacement afin que votre instance secondaire puisse toujours accéder à ces ressources.

  Supposons, par exemple, que votre application logique principale se connecte à un service qui se trouve dans le même emplacement ou la même région, par exemple, Azure SQL Database. Si toute cette région n’est plus disponible, le service Azure SQL Database de cette région est aussi probablement indisponible. Dans ce cas, il faudra que votre instance secondaire utilise une base de données répliquée ou de sauvegarde avec une connexion distincte à cette base de données.

<a name="on-premises-data-gateways"></a>

## <a name="on-premises-data-gateways"></a>Passerelles de données locales

Si votre application logique s’exécute dans un Azure multilocataire et a besoin d’accéder à des ressources locales telles que des bases de données SQL Server, vous devez installer la [passerelle de données locale](../logic-apps/logic-apps-gateway-install.md) sur un ordinateur local. Vous pouvez ensuite créer une ressource de passerelle de données dans le portail Azure afin que votre application logique puisse utiliser la passerelle lorsque vous créez une connexion à la ressource.

La ressource de passerelle de données est associée à un emplacement ou à une région Azure, tout comme la ressource de votre application logique. Dans votre stratégie de reprise d’activité, assurez-vous que la passerelle de données reste disponible pour que votre application logique puisse l’utiliser. Vous pouvez [activer la haute disponibilité pour votre passerelle](../logic-apps/logic-apps-gateway-install.md#high-availability) lorsque vous avez plusieurs installations de passerelle.

> [!NOTE]
> Si votre application logique s’exécute dans un environnement de service d’intégration (ISE) et utilise uniquement des connecteurs avec version ISE pour des sources de données locales, vous n’avez pas besoin de la passerelle de données, car les connecteurs ISE fournissent un accès direct à la ressource locale.
>
> Si aucun connecteur avec version ISE n’est disponible pour la ressource locale requise, votre application logique peut toujours créer la connexion à l’aide d’un connecteur non-ISE, qui s’exécute dans Azure multilocataire mondial, et non dans votre environnement ISE. Toutefois, cette connexion requiert la passerelle de données locale.

<a name="roles"></a>

## <a name="active-active-versus-active-passive-roles"></a>Rôles actif/actif et actif/passif

Vous pouvez configurer vos emplacements principal et secondaire de manière à ce que les instances de votre application logique dans ces emplacements puissent jouer ces rôles :

| Rôle principal/secondaire | Description |
|------------------------|-------------|
| *Actif/actif* | Les instances d’application logique principale et secondaire dans les deux emplacements gèrent activement les demandes en suivant l’un de ces modèles : <p><p>- *Équilibrage de charge* : Vous pouvez faire en sorte que les deux instances écoutent un point de terminaison et équilibrent la charge du trafic vers chaque instance selon les besoins. <p>- *Consommateurs concurrents* : Vous pouvez faire en sorte que les deux instances agissent comme des consommateurs concurrents afin qu’elles soient en concurrence pour les messages provenant d’une file d’attente. Si une instance échoue, l’autre instance prend la charge de travail. |
| *Actif/passif* | L’instance d’application logique principale gère activement l’intégralité de la charge de travail, tandis que l’instance secondaire est passive (désactivée ou inactive). L’instance secondaire attend un signal indiquant que l’instance principale n’est pas disponible ou ne fonctionne pas en raison d’une interruption ou d’un échec, et prend la charge de travail en tant qu’instance active. |
| Combinaison | Certaines applications logiques jouent un rôle actif/actif, tandis que d’autres applications logiques jouent un rôle actif/passif. |
|||

<a name="active-active-examples"></a>

### <a name="active-active-examples"></a>Exemples actif/actif

Ces exemples illustrent la configuration actif/actif dans laquelle les deux instances de l’application logique gèrent activement les requêtes ou les messages. Un autre système ou service distribue les demandes ou les messages entre les instances, par exemple, l’une de ces options :

* Un équilibreur de charge « physique », comme du matériel qui achemine le trafic

* Un équilibreur de charge « logiciel », comme [Azure Load Balancer](../load-balancer/load-balancer-overview.md) ou [Gestion des API Azure](../api-management/api-management-key-concepts.md). Avec Gestion des API, vous pouvez spécifier des stratégies qui déterminent comment équilibrer la charge du trafic entrant. Vous pouvez également utiliser un service qui prend en charge le suivi de l’état, par exemple, [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md).

  Bien que cet exemple illustre principalement Azure Load Balancer, vous pouvez utiliser l’option qui répond le mieux aux besoins de votre scénario :

  ![Configuration « actif/actif » qui utilise un équilibrage de charge ou un service avec état](./media/business-continuity-disaster-recovery-guidance/active-active-setup-load-balancer.png)

* Chaque instance d’application logique agit comme un consommateur et les deux instances sont en concurrence pour les messages provenant d’une file d’attente :

  ![Configuration « actif/actif » qui utilise des « consommateurs concurrents »](./media/business-continuity-disaster-recovery-guidance/active-active-competing-consumers-pattern.png)

<a name="active-passive-examples"></a>

### <a name="active-passive-examples"></a>Exemples actif/passif

Cet exemple montre la configuration actif/passif dans laquelle l’instance d’application logique principale est active à un emplacement, tandis que l’instance secondaire reste inactive à un autre emplacement. Si l’instance principale subit une interruption ou une défaillance, vous pouvez faire en sorte qu’un opérateur exécute un script qui active l’instance secondaire pour qu’elle prenne en charge la charge de travail.

![Configuration « actif/passif » qui utilise des « consommateurs concurrents »](./media/business-continuity-disaster-recovery-guidance/active-passive-setup.png)

<a name="active-active-active-passive-examples"></a>

### <a name="combination-with-active-active-and-active-passive"></a>Combinaison de configuration actif/actif et actif/passif

Cet exemple montre une configuration combinée dans laquelle l’emplacement principal contient les deux instances d’application logique actives, tandis que l’emplacement secondaire contient des instances d’application logique actives/passives. Si l’emplacement principal subit une interruption ou une défaillance, l’application logique active dans l’emplacement secondaire, qui se charge déjà d’une charge de travail partielle, peut se charger de la charge de travail dans son intégralité.

* Dans l’emplacement principal, une application logique active écoute les messages dans une file d’attente Azure Service Bus, tandis qu’une autre application logique active vérifie les e-mails à l’aide d’un déclencheur d’interrogation Office 365 Outlook.

* Dans l’emplacement secondaire, une application logique active fonctionne avec l’application logique dans l’emplacement principal en écoutant les messages dans la même file d’attente Service Bus et en entrant en concurrence pour ceux-ci. Pendant ce temps, une application logique inactive passive attend, en mode veille, pour vérifier les e-mails lorsque l’emplacement principal devient indisponible, mais elle est *désactivée* pour éviter de relire les mêmes e-mails.

![Combinaison « actif/passif » et « actif/passif » qui utilise des déclencheurs de périodicité](./media/business-continuity-disaster-recovery-guidance/combo-active-active-active-passive-setup.png)

<a name="state-history"></a>

## <a name="logic-app-state-and-history"></a>État et historique de l’application logique

Lorsque votre application logique est déclenchée et commence à s’exécuter, l’état de l’application est stocké à l’emplacement où l’application a démarré et n’est pas transférable vers un autre emplacement. En cas de défaillance ou d’interruption, toutes les instances de flux de travail en cours sont abandonnées. Lorsque vous avez configuré un emplacement principal et un emplacement secondaire, les nouvelles instances de flux de travail commencent à s’exécuter à l’emplacement secondaire.

<a name="reduce-abandoned-in-progress"></a>

### <a name="reduce-abandoned-in-progress-instances"></a>Réduire les instances en cours abandonnées

Pour réduire le nombre d’instances de flux de travail en cours abandonnées, vous pouvez choisir parmi différents modèles de messages que vous pouvez implémenter, par exemple :

* [Modèle de bordereau de routage fixe](/biztalk/esb-toolkit/message-routing-patterns#routing-slip)

  Ce modèle de message d’entreprise fractionne un processus métier en étapes plus petites. Pour chaque étape, vous configurez une application logique qui gère la charge de travail de cette étape. Pour communiquer entre elles, vos applications logiques utilisent un protocole de messagerie asynchrone, comme des files d’attente ou des rubriques Azure Service Bus. Lorsque vous divisez un processus en étapes plus petites, vous réduisez le nombre de processus métier qui peuvent être bloqués sur une instance d’application logique qui a échoué. Pour plus d’informations générales sur ce modèle, consultez [Modèles d’intégration d’entreprise - Bordereau de routage](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RoutingTable.html).

  Cet exemple montre un modèle de bordereau de routage où chaque application logique représente une étape et utilise une file d’attente Service Bus pour communiquer avec l’application logique suivante dans le processus.

  ![Fractionner un processus métier en étapes représentées par des applications logiques, qui communiquent entre elles à l’aide de files d’attente Azure Service Bus](./media/business-continuity-disaster-recovery-guidance/fixed-routing-slip-pattern.png)

  Si les deux instances d’application logique principale et secondaire suivent le même modèle de bordereau de routage dans leurs emplacements, vous pouvez implémenter le [modèle de consommateurs concurrents](/azure/architecture/patterns/competing-consumers) en configurant des [rôles actifs/actifs](#roles) pour ces instances.

* [Modèle de gestionnaire de processus (répartiteur)](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html)

* [Modèle Afficher-verrouiller (Peek-lock) sans délai d’expiration](https://social.technet.microsoft.com/wiki/contents/articles/50022.azure-service-bus-how-to-peek-lock-a-message-from-queue-using-azure-logic-apps.aspx)

<a name="access-trigger-runs-history"></a>

### <a name="access-to-trigger-and-runs-history"></a>Accéder à l’historique du déclencheur et des exécutions

Pour obtenir plus d’informations sur les exécutions passées de flux de travail de votre application logique, vous pouvez examiner l’historique du déclencheur et des exécutions de l’application. L’historique d’exécution d’une application logique est stocké dans l’emplacement ou la région où l’application logique a été exécutée, ce qui signifie que vous ne pouvez pas migrer cet historique vers un autre emplacement. Si votre instance principale bascule vers une instance secondaire, vous ne pouvez accéder à l’historique du déclencheur et des exécutions de chaque instance que dans les emplacements respectifs où ces instances ont été exécutées. Toutefois, vous pouvez obtenir des informations indépendantes de l’emplacement sur l’historique de votre application logique en configurant vos applications logiques pour qu’elles envoient des événements de diagnostic à un espace de travail Azure Log Analytics. Vous pouvez ensuite passer en revue l’intégrité et l’historique des applications logiques qui s’exécutent à plusieurs emplacements.

<a name="trigger-types-guidance"></a>

## <a name="trigger-type-guidance"></a>Aide sur le type de déclencheur

Le type de déclencheur que vous utilisez dans vos applications logiques détermine les options de configuration des applications logiques entre les différents emplacements dans votre stratégie de reprise d’activité. Voici les types de déclencheurs disponibles que vous pouvez utiliser dans des applications logiques :

* [Déclencheur de périodicité](#recurrence-trigger)
* [Déclencheur d’interrogation](#polling-trigger)
* [Déclencheur de requête](#request-trigger)
* [Déclencheur Webhook](#webhook-trigger)

<a name="recurrence-trigger"></a>

### <a name="recurrence-trigger"></a>Déclencheur recurrence

Le déclencheur de périodicité **Recurrence** est indépendant de tout service ou point de terminaison spécifique, et se déclenche uniquement en fonction d’une planification spécifiée et d’aucun autre critère, par exemple :

* Une fréquence et un intervalle fixes, par exemple toutes les 10 minutes
* Une planification plus avancée, comme le dernier lundi de chaque mois à 17h

Lorsque votre application logique démarre avec un déclencheur Recurrence, vous devez configurer vos instances d’application logique principale et secondaire avec les [rôles actifs/passifs](#roles). Pour réduire *l’objectif de délai de récupération* (RTO), qui fait référence à la durée cible de restauration d’un processus métier après une interruption ou un incident, vous pouvez configurer vos instances d’application logique avec une combinaison de [rôles actifs/passifs](#roles) et de [rôles passifs/actifs](#roles). Dans cette configuration, vous fractionnez la planification entre les différents emplacements.

Supposons, par exemple, que vous ayez une application logique qui doit s’exécuter toutes les 10 minutes. Vous pouvez configurer vos applications logiques et emplacements de sorte que si l’emplacement principal devient indisponible, l’emplacement secondaire peut se charger du travail :

![Combinaison « actif/passif » et « passif/actif » qui utilise des déclencheurs de périodicité](./media/business-continuity-disaster-recovery-guidance/combo-active-passive-passive-active-setup.png)

* Dans l’emplacement principal, configurez des [rôles actifs/passifs](#roles) pour ces applications logiques :

  * Pour l’application logique *active* activée, définissez le déclencheur de périodicité afin qu’il démarre sur l’heure et se répète toutes les 20 minutes, par exemple, 9h, 9h20, et ainsi de suite.

  * Pour l’application logique *passive* désactivée, définissez le déclencheur de périodicité sur le même horaire mais avec un démarrage 10 minutes après l’heure et une répétition toutes les 20 minutes, par exemple, 9h10, 9h30, et ainsi de suite.

* Dans l’emplacement secondaire, configurez [passif/actif](#roles) pour ces applications logiques :

  * Pour l’application logique *passive* désactivée, définissez le déclencheur de périodicité sur le même horaire que l’application logique active dans l’emplacement principal, c’est-à-dire un démarrage sur l’heure et une répétition toutes les 20 minutes, par exemple, 9h, 9h20, et ainsi de suite.

  * Pour l’application logique *active* activée, définissez le déclencheur de périodicité sur le même horaire que l’application logique passive dans l’emplacement principal, c’est-à-dire un démarrage 10 minutes après l’heure et une répétition toutes les 20 minutes, par exemple, 9h10, 9h30, et ainsi de suite.

Si un événement perturbateur se produit dans l’emplacement principal, activez l’application logique passive à l’emplacement secondaire. Ainsi, si la détection de la défaillance prend du temps, cette configuration limite le nombre de récurrences manquées durant ce délai.

<a name="polling-trigger"></a>

### <a name="polling-trigger"></a>Déclencheur d’interrogation

Pour vérifier régulièrement si de nouvelles données pour le traitement sont disponibles à partir d’un service ou d’un point de terminaison spécifique, votre application logique peut utiliser un déclencheur *d’interrogation* qui appelle de façon répétée le service ou le point de terminaison en fonction d’une planification de périodicité fixe. Les données que le service ou le point de terminaison fournit peuvent avoir l’un des types suivants :

* Données statiques, qui décrivent les données qui sont toujours disponibles pour la lecture
* Données volatiles, qui décrivent les données qui ne sont plus disponibles après la lecture

Pour éviter de lire les mêmes données de façon répétée, votre application logique doit mémoriser les données qui ont été lues précédemment en conservant l’état côté client ou côté serveur, service ou système.

* Les applications logiques qui fonctionnent avec l’état côté client utilisent des déclencheurs qui peuvent conserver l’état.

  Par exemple, un déclencheur qui lit un nouveau message à partir d’une boîte de réception de courrier électronique nécessite que le déclencheur mémorise le dernier message lu. De cette façon, le déclencheur ne démarre l’application logique que lorsque le message non lu suivant arrive.

* Les applications logiques qui fonctionnent avec l’état côté serveur, service ou système utilisent des paramètres ou valeurs de propriété qui se trouvent côté serveur, service ou système.

  Par exemple, un déclencheur basé sur requête qui lit une ligne d’une base de données nécessite que la ligne ait une colonne `isRead` définie sur `FALSE`. Chaque fois que le déclencheur lit une ligne, l’application logique met à jour cette ligne en modifiant la colonne `isRead` de `FALSE` à `TRUE`.

  Cette approche côté serveur fonctionne de la même façon pour les files d’attente ou les rubriques Service Bus qui disposent d’une sémantique de mise en file d’attente où un déclencheur peut lire et verrouiller un message pendant que l’application logique traite le message. À la fin du traitement par l’application logique, le déclencheur supprime le message de la file d’attente ou de la rubrique.

Du point de vue de la reprise d’activité, quand vous configurez les instances principale et secondaire de votre application logique, assurez-vous de prendre en compte ces comportements selon que votre application logique effectue le suivi de l’état côté client ou côté serveur :

* Pour une application logique qui fonctionne avec l’état côté client, assurez-vous que votre application logique ne lit pas le même message plusieurs fois. Un seul emplacement peut avoir une application logique active à tout moment donné. Vérifiez que l’instance d’application logique à l’emplacement secondaire est inactive ou désactivée jusqu’au basculement de l’instance principale vers l’emplacement secondaire.

  Par exemple, le déclencheur Office 365 Outlook conserve l’état côté client et effectue le suivi de l’horodatage du dernier e-mail lu afin d’éviter de lire un doublon.

* Pour une application logique qui fonctionne avec l’état côté serveur, vous pouvez configurer vos instances d’application logique pour qu’elles jouent des [rôles actifs/actifs](#roles) où elles fonctionnent comme des consommateurs concurrents ou des [rôles actifs/passifs](#roles) où l’instance de remplacement attend que l’instance principale bascule vers l’emplacement secondaire.

  Par exemple, la lecture à partir d’une file d’attente de messages, comme une file d’attente Azure Service Bus, utilise l’état côté serveur, car le service de mise en file d’attente maintient des verrous sur les messages pour empêcher d’autres clients de lire les mêmes messages.

  > [!NOTE]
  > Si votre application logique doit lire les messages dans un ordre spécifique, par exemple, à partir d’une file d’attente Service Bus, vous pouvez utiliser le modèle de consommateurs concurrents, mais uniquement en association avec des sessions Service Bus, ce qui est également appelé [*modèle de convoi séquentiel*](/azure/architecture/patterns/sequential-convoy). Dans le cas contraire, vous devez configurer vos instances d’application logique avec des rôles actifs/passifs.

<a name="request-trigger"></a>

### <a name="request-trigger"></a>Déclencheur de requête

Le déclencheur de requête **Request** fait en sorte que votre application logique puisse être appelée à partir d’autres applications, services et systèmes, et est généralement utilisé pour fournir les fonctionnalités suivantes :

* Une API REST directe pour votre application logique que les autres peuvent appeler

  Par exemple, utilisez le déclencheur de requête pour démarrer votre application logique afin que d’autres applications logiques puissent appeler le déclencheur à l’aide de l’action **Appeler le flux de travail - Logic Apps**.

* Un [webhook](#webhook-trigger) ou un mécanisme de rappel pour votre application logique

* Une façon dont vous pouvez exécuter manuellement des opérations ou routines utilisateur pour appeler votre application logique, par exemple, à l’aide d’un script PowerShell qui effectue une tâche spécifique

Du point de vue de la reprise d’activité, le déclencheur de requête est un récepteur passif, car l’application logique n’effectue aucun travail et attend qu’un autre service ou système appelle le déclencheur de manière explicite. En tant que point de terminaison passif, vous pouvez configurer vos instances principale et secondaire de l’une des manières suivantes :

* [Actif/actif](#roles) : Les deux instances gèrent activement les requêtes ou les appels. L’appelant ou le routeur équilibre ou distribue le trafic entre ces instances.

* [Actif/passif](#roles) : Seule l’instance principale est active et gère l’ensemble du travail, tandis que l’instance secondaire attend que l’instance principale subisse une interruption ou une défaillance. L’appelant ou le routeur détermine quand appeler l’instance secondaire.

En tant qu’architecture recommandée, vous pouvez utiliser Gestion des API Azure comme proxy pour les applications logiques qui utilisent des déclencheurs de requête. Gestion des API offre une [résilience intégrée entre les régions et la capacité à acheminer le trafic entre plusieurs points de terminaison](../api-management/api-management-howto-deploy-multi-region.md).

<a name="webhook-trigger"></a>

### <a name="webhook-trigger"></a>Déclencheur de webhook

Un déclencheur *webhook* permet à votre application logique de s’abonner à un service en passant une *URL de rappel* à ce service. Votre application logique peut ensuite écouter et attendre qu’un événement spécifique se produise au niveau de ce point de terminaison de service. Lorsque l’événement se produit, le service appelle le déclencheur webhook à l’aide de l’URL de rappel, qui exécute ensuite l’application logique. Lorsqu’il est activé, le déclencheur webhook s’abonne au service. Lorsqu’il est désactivé, le déclencheur se désabonne du service.

Du point de vue de la reprise d’activité, configurez des instances principale et secondaire utilisant des déclencheurs webhook pour jouer des rôles actifs/passifs, car une seule instance doit recevoir des événements ou des messages du point de terminaison abonné.

## <a name="assess-primary-instance-health"></a>Évaluer l’intégrité de l’instance principale

Pour que votre stratégie de reprise d’activité fonctionne, votre solution nécessite des moyens d’effectuer les tâches suivantes :

* [Vérifier la disponibilité de l’instance principale](#check-primary-availability)
* [Surveiller l’intégrité de l’instance principale](#monitor-primary-health)
* [Activer l’instance secondaire](#activate-secondary)

Cette section décrit une solution que vous pouvez utiliser en mode autonome ou en tant que base de votre propre conception. Voici une vue d’ensemble visuelle de haut niveau pour cette solution :

![Créer une application logique de surveillance qui surveille une application logique de contrôle d’intégrité dans l’emplacement principal](./media/business-continuity-disaster-recovery-guidance/check-location-health-watchdog.png)

<a name="check-primary-availability"></a>

### <a name="check-primary-instance-availability"></a>Vérifier la disponibilité de l’instance principale

Pour déterminer si l’instance principale est disponible, en cours d’exécution et en mesure de fonctionner, vous pouvez créer une application logique de « contrôle d’intégrité » qui se trouve dans le même emplacement que l’instance principale. Vous pouvez ensuite appeler cette application de contrôle d’intégrité à partir d’un autre emplacement. Si l’application de contrôle d’intégrité répond correctement, l’infrastructure sous-jacente du service Azure Logic Apps de cette région est disponible et opérationnelle. Si l’application de contrôle d’intégrité ne répond pas, vous pouvez supposer que l’emplacement n’est plus sain.

Pour cette tâche, créez une application logique de contrôle d’intégrité de base qui effectue les tâches suivantes :

1. Reçoit un appel de l’application de surveillance à l’aide du déclencheur de requête.

1. Répond à l’aide d’un état indiquant si l’application logique vérifiée fonctionne toujours à l’aide de l’action de réponse.

   > [!IMPORTANT]
   > L’application logique de contrôle d’intégrité doit utiliser une action de réponse pour que l’application réponde de façon synchrone, et non asynchrone.

1. Pour déterminer plus précisément si l’emplacement principal est sain, vous pouvez éventuellement prendre en compte l’intégrité de tous les autres services qui interagissent avec l’application logique cible à cet emplacement. Développez simplement l’application logique de contrôle d’intégrité pour évaluer aussi l’intégrité de ces autres services.

<a name="monitor-primary-health"></a>

### <a name="create-a-watchdog-logic-app"></a>Créer une application logique de surveillance

Pour surveiller l’intégrité de l’instance principale et appeler l’application logique de contrôle d’intégrité, créez une application logique de « surveillance » dans un *autre emplacement*. Par exemple, vous pouvez configurer l’application logique de surveillance de sorte que si l’appel de l’application logique de contrôle d’intégrité échoue, l’application logique de surveillance peut envoyer une alerte à votre équipe des opérations pour qu’elle examine l’échec et la raison pour laquelle l’instance principale ne répond pas.

> [!IMPORTANT]
> Assurez-vous que votre application logique de surveillance se trouve dans un *emplacement qui diffère de l’emplacement principal*. Si le service Logic Apps dans l’emplacement principal rencontre des problèmes, votre application logique de surveillance peut ne pas s’exécuter.

Pour cette tâche, dans l’emplacement secondaire, créez une application logique de surveillance qui effectue les tâches suivantes :

1. Exécution basée sur une périodicité fixe ou planifiée à l’aide du déclencheur de périodicité.

   Vous pouvez définir la périodicité sur une valeur qui se trouve sous le niveau de tolérance pour votre objectif de délai de récupération (RTO).

1. Appelez l’application logique de contrôle d’intégrité dans l’emplacement principal à l’aide de l’action HTTP, par exemple :

<a name="activate-secondary"></a>

### <a name="activate-your-secondary-instance"></a>Activer votre instance secondaire

Pour activer automatiquement l’instance secondaire, vous pouvez créer une application logique qui appelle l’API de gestion, par exemple [le connecteur Azure Resource Manager](/connectors/arm/) pour activer les applications logiques appropriées dans l’emplacement secondaire. Vous pouvez développer votre application de surveillance pour appeler cette application logique d’activation après un nombre spécifique d’échecs.

<a name="collect-diagnostic-data"></a>

## <a name="collect-diagnostic-data"></a>Collecter des données de diagnostic

Vous pouvez configurer la journalisation des exécutions de votre application logique et envoyer les données de diagnostic obtenues à des services comme Stockage Azure, Azure Event Hubs et Azure Log Analytics pour une gestion et un traitement supplémentaires.

* Si vous souhaitez utiliser ces données avec Azure Log Analytics, vous pouvez rendre les données disponibles pour les emplacements principal et secondaire en configurant les **paramètres de diagnostic** de votre application logique et en envoyant les données à plusieurs espaces de travail Log Analytics. Pour plus d'informations, consultez [Configurer les journaux d'activité Azure Monitor et collecter des données de diagnostic pour Azure Logic Apps](../logic-apps/monitor-logic-apps-log-analytics.md).

* Si vous souhaitez envoyer les données à Stockage Azure ou Azure Event Hubs, vous pouvez rendre les données disponibles pour les emplacements principal et secondaire en configurant la géoredondance. Pour plus d’informations, voir les articles suivants :<p>

  * [Reprise d’activité et basculement de compte dans Stockage Blob Azure](../storage/common/storage-disaster-recovery-guidance.md)
  * [Azure Event Hubs - Géorécupération d’urgence](../event-hubs/event-hubs-geo-dr.md)

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble de la résilience pour Azure](/azure/architecture/framework/resiliency/overview)
* [Liste de vérification de la résilience pour des services Azure spécifiques](/azure/architecture/checklist/resiliency-per-service)
* [Gestion des données pour la résilience dans Azure](/azure/architecture/framework/resiliency/data-management)
* [Sauvegarde et reprise d’activité pour les applications Azure](/azure/architecture/framework/resiliency/backup-and-recovery)
* [Récupérer après une interruption de service à l’échelle de la région](/azure/architecture/resiliency/recovery-loss-azure-region)
* [Contrats de niveau de service (SLA) Microsoft pour les services Azure](https://azure.microsoft.com/support/legal/sla/)
