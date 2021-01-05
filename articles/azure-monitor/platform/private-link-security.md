---
title: Utiliser Azure Private Link pour connecter en toute sécurité des réseaux à Azure Monitor
description: Utiliser Azure Private Link pour connecter en toute sécurité des réseaux à Azure Monitor
author: noakup
ms.author: noakuper
ms.topic: conceptual
ms.date: 10/05/2020
ms.subservice: ''
ms.openlocfilehash: a85619b4947808ba1c13df3c1543102eea7273fd
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96853921"
---
# <a name="use-azure-private-link-to-securely-connect-networks-to-azure-monitor"></a>Utiliser Azure Private Link pour connecter en toute sécurité des réseaux à Azure Monitor

[Azure Private Link](../../private-link/private-link-overview.md) vous permet de lier en toute sécurité les services PaaS Azure à votre réseau virtuel à l’aide de points de terminaison privés. Pour de nombreux services, il vous suffit de configurer un point de terminaison par ressource. Toutefois, Azure Monitor est une constellation de différents services interconnectés qui fonctionnent ensemble pour surveiller vos charges de travail. Par conséquent, nous avons créé une ressource appelée Étendue de liaison privée Azure Monitor (AMPLS) qui vous permet de définir les limites de votre réseau de surveillance et de vous connecter à votre réseau virtuel. Cet article explique quand utiliser et comment configurer une étendue de liaison privée Azure Monitor.

## <a name="advantages"></a>Avantages

Grâce à Azure Private Link, vous pouvez :

- Vous connecter en privé à Azure Monitor sans ouvrir d’accès au réseau public
- Vérifier que vos données de surveillance sont accessibles uniquement par le biais de réseaux privés autorisés
- Empêcher l’exfiltration de données à partir de vos réseaux privés en définissant les ressources Azure Monitor spécifiques qui se connectent via votre point de terminaison privé
- Connecter en toute sécurité votre réseau local privé à Azure Monitor à l’aide d’ExpressRoute et d’Azure Private Link
- Conserver tout le trafic au sein du réseau principal Microsoft Azure

Pour plus d’informations, consultez [Principaux avantages d’Azure Private Link](../../private-link/private-link-overview.md#key-benefits).

## <a name="how-it-works"></a>Fonctionnement

Une étendue de liaison privée Azure Monitor est une ressource de regroupement qui permet de connecter un ou plusieurs points de terminaison privés (et par conséquent, les réseaux virtuels qui les contiennent) à une ou plusieurs ressources Azure Monitor. Les ressources incluent des espaces de travail Log Analytics et des composants Application Insights.

![Schéma de la topologie des ressources](./media/private-link-security/private-link-topology-1.png)

> [!NOTE]
> Une seule ressource Azure Monitor peut appartenir à plusieurs AMPLS, mais vous ne pouvez pas connecter un réseau virtuel unique à plusieurs AMPLS. 

## <a name="planning-based-on-your-network"></a>Planification en fonction de votre réseau

Avant de configurer vos ressources AMPLS, tenez compte de vos exigences en matière d’isolement réseau. Évaluez l’accès de vos réseaux virtuels à l’Internet public et les restrictions d’accès de chacune de vos ressources Azure Monitor (c’est-à-dire les composants Application Insights et les espaces de travail Log Analytics).

> [!NOTE]
> Les réseaux hub-and-spoke, ou toute autre topologie de réseaux appairés, peuvent configurer une liaison privée entre le réseau virtuel hub (principal) et les ressources Azure Monitor pertinentes, au lieu de configurer une liaison privée sur chaque réseau virtuel. Cela s’avère particulièrement utile si les ressources Azure Monitor utilisées par ces réseaux sont partagées. Toutefois, si vous souhaitez autoriser chaque réseau virtuel à accéder à un ensemble distinct de ressources d’analyse, créez une liaison privée vers un objet AMPLS dédié pour chaque réseau.

### <a name="evaluate-which-virtual-networks-should-connect-to-a-private-link"></a>Évaluer quels réseaux virtuels doivent se connecter à une liaison privée

Commencez par évaluer lesquels de vos réseaux virtuels ont un accès limité à Internet. Les réseaux virtuels disposant d’un accès gratuit à Internet peuvent ne pas avoir besoin d’une liaison privée pour accéder à vos ressources Azure Monitor. Les ressources de surveillance auxquelles vos réseaux virtuels se connectent peuvent limiter le trafic entrant et nécessiter une connexion par liaison privée (à des fins de requête ou d’ingestion de journal). Dans ce cas, même un réseau virtuel qui a accès à l’Internet public doit se connecter à ces ressources via une liaison privée et par le biais d’une AMPLS.

### <a name="evaluate-which-azure-monitor-resources-should-have-a-private-link"></a>Évaluer quelles ressources Azure Monitor doivent avoir une liaison privée

Passez en revue chacune de vos ressources Azure Monitor :

- La ressource doit-elle autoriser l’ingestion des journaux provenant de ressources situées sur des réseaux virtuels spécifiques uniquement ?
- La ressource doit-elle être interrogée uniquement par les clients situés sur des réseaux virtuels spécifiques ?

Si la réponse à l’une de ces questions est oui, définissez les restrictions comme expliqué dans [Configuration des espaces de travail Log Analytics](#configure-log-analytics) et [Configuration des composants Application Insights](#configure-application-insights) et associez ces ressources à une ou plusieurs AMPLS. Les réseaux virtuels qui doivent accéder à ces ressources de surveillance doivent disposer d’un point de terminaison privé qui se connecte à l’AMPLS appropriée.
N’oubliez pas : vous pouvez connecter les mêmes espaces de travail ou applications à plusieurs AMPLS pour que différents réseaux puissent les atteindre.

### <a name="group-together-monitoring-resources-by-network-accessibility"></a>Regrouper les ressources de surveillance par accessibilité réseau

Étant donné que chaque réseau virtuel ne peut se connecter qu’à une seule ressource AMPLS, vous devez regrouper les ressources de surveillance qui doivent être accessibles au même réseau. La façon la plus simple de gérer ce regroupement consiste à créer une AMPLS par réseau virtuel et à sélectionner les ressources à connecter à ce réseau. Toutefois, pour réduire les ressources et faciliter la gestion, vous pouvez réutiliser une AMPLS entre les réseaux.

Par exemple, si vos réseaux virtuels internes VNet1 et VNet2 doivent se connecter aux espaces de travail Workspace1 et Workspace2 ainsi qu’au composant Application Insights 3, associez les trois ressources à la même AMPLS. Si VNet3 ne doit accéder qu’à Workspace1, créez une autre ressource AMPLS, associez-y Workspace1 et connectez-y VNet3 comme indiqué dans les schémas suivants :

![Schéma de la topologie d’AMPLS A](./media/private-link-security/ampls-topology-a-1.png)

![Schéma de la topologie d’AMPLS B](./media/private-link-security/ampls-topology-b-1.png)

### <a name="consider-limits"></a>Tenir compte des limites

Il existe un certain nombre de limites à prendre en compte lorsque vous planifiez votre installation Azure Private Link :

* Un réseau virtuel ne peut se connecter qu’à un seul objet AMPLS. Cela signifie que l’objet AMPLS doit fournir l’accès à toutes les ressources Azure Monitor auxquelles le réseau virtuel doit avoir accès.
* Une ressource Azure Monitor (un espace de travail ou un composant Application Insights) peut être connectée à cinq objets AMPLS au maximum.
* Un objet AMPLS peut se connecter à 50 ressources Azure Monitor au maximum.
* Un objet AMPLS peut se connecter à 10 points de terminaison privés au maximum.

Dans la topologie ci-dessous :
* Chaque réseau virtuel se connecte uniquement à **1** objet AMPLS.
* AMPLS B est connecté aux points de terminaison privés de deux réseaux virtuels (VNet2 et VNet3) en utilisant 2 de ses 10 connexions (20 %) de ses points de terminaison privés possibles.
* AMPLS A se connecte à deux espaces de travail et à un composant Application Insights en utilisant 3 de ses 50 connexions (6 %) de ses connexions de ressources Azure Monitor possibles.
* Workspace2 se connecte à AMPLS A et AMPLS B : il utilise 2 de ses 5 connexions AMPLS possibles, soit 40 %.

![Diagramme des limites d’AMPLS](./media/private-link-security/ampls-limits.png)

> [!NOTE]
> Dans certaines topologies de réseau (principalement hub-and-spoke), vous pouvez atteindre rapidement la limite de 10 réseaux virtuels pour un même AMPLS. Dans ce cas, il est recommandé d’utiliser une connexion de liaison privée partagée plutôt que des connexions séparées. Créez un point de terminaison privé sur le réseau hub, liez-le à votre AMPLS, puis appairez les réseaux nécessaires au réseau hub.

![Point de terminaison privé hub-and-spoke](./media/private-link-security/hub-and-spoke-with-single-private-endpoint.png)

## <a name="example-connection"></a>Exemple de connexion

Commencez par créer une ressource d’étendue de liaison privée Azure Monitor.

1. Accédez à **Créer une ressource** dans le portail Azure et recherchez **Étendue de liaison privée Azure Monitor**.

   ![Rechercher l’étendue de liaison privée Azure Monitor](./media/private-link-security/ampls-find-1c.png)

2. Cliquez sur **Create** (Créer).
3. Choisissez un abonnement et un groupe de ressources.
4. Donnez un nom à l’AMPLS. Il est préférable d’utiliser un nom qui indique clairement à quelle fin et à quelle limite de sécurité l’étendue sera utilisée afin que personne n’arrête accidentellement les limites de sécurité réseau. Par exemple, « AppServerProdTelem ».
5. Cliquez sur **Revoir + créer**. 

   ![Créer une étendue de liaison privée Azure Monitor](./media/private-link-security/ampls-create-1d.png)

6. Laissez le processus de validation se terminer, puis cliquez sur **Créer**.

### <a name="connect-azure-monitor-resources"></a>Connecter des ressources Azure Monitor

Connectez des ressources Azure Monitor (espaces de travail Log Analytics et composants Application Insights) à votre AMPLS.

1. Dans votre étendue de liaison privée Azure Monitor, cliquez sur **Ressources Azure Monitor** dans le menu de gauche. Cliquez sur le bouton **Add** .
2. Ajoutez l’espace de travail ou le composant. Le fait de cliquer sur le bouton **Ajouter** affiche une boîte de dialogue dans laquelle vous pouvez sélectionner des ressources Azure Monitor. Vous pouvez parcourir vos abonnements et groupes de ressources, ou vous pouvez saisir leur nom pour les filtrer. Sélectionnez l’espace de travail ou le composant, puis cliquez sur **Appliquer** pour les ajouter à votre étendue.

    ![Capture d’écran de l’expérience utilisateur Sélectionner une étendue](./media/private-link-security/ampls-select-2.png)

> [!NOTE]
> Pour supprimer des ressources Azure Monitor, vous devez d’abord les déconnecter des objets AMPLS auxquels elles sont connectées. Vous ne pouvez pas supprimer des ressources connectées à un AMPLS.

### <a name="connect-to-a-private-endpoint"></a>Se connecter à un point de terminaison privé

Maintenant que vous disposez de ressources connectées à votre AMPLS, créez un point de terminaison privé pour connecter notre réseau. Vous pouvez effectuer cette tâche dans le [centre Azure Private Link sur le portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Network/PrivateLinkCenterBlade/privateendpoints) ou au sein même de votre étendue de liaison privée Azure Monitor, comme dans cet exemple.

1. Dans votre ressource d’étendue, cliquez sur **Connexions de point de terminaison privés** dans le menu gauche de la ressource. Cliquez sur **Point de terminaison privé** pour démarrer le processus de création du point de terminaison. Vous pouvez également approuver ici des connexions qui ont été démarrées dans le centre Azure Private Link en les sélectionnant et en cliquant sur **Approuver**.

    ![Capture d’écran de l’expérience utilisateur Connexions de point de terminaison privés](./media/private-link-security/ampls-select-private-endpoint-connect-3.png)

2. Choisissez l’abonnement, le groupe de ressources et le nom du point de terminaison, ainsi que la région dans laquelle il doit résider. La région doit être la même que celle du réseau virtuel auquel vous allez le connecter.

3. Cliquez sur **Suivant : Ressource**. 

4. Dans l’écran Ressource :

   a. Sélectionnez l’**abonnement** qui contient votre ressource d’étendue de liaison privée Azure Monitor. 

   b. Pour le **type de ressource**, choisissez **Microsoft.insights/privateLinkScopes**. 

   c. Dans la liste déroulante des **ressources**, choisissez l’étendue de liaison privée que vous avez créée précédemment. 

   d. Cliquez sur **Suivant : Configuration >** .
      ![Screenshot of select Create Private Endpoint](./media/private-link-security/ampls-select-private-endpoint-create-4.png)

5. Dans le volet configuration :

   a.    Choisissez le **réseau virtuel** et le **sous-réseau** que vous souhaitez connecter à vos ressources Azure Monitor. 
 
   b.    Choisissez **Oui** pour **Intégrer à une zone DNS privée**, et laissez-le créer automatiquement une nouvelle zone DNS privée. Les zones DNS réelles peuvent différer de ce que montre la capture d’écran ci-dessous. 
   > [!NOTE]
   > Si vous choisissez **Non** et préférez gérer les enregistrements DNS manuellement, commencez par configurer votre liaison privée, y compris ce point de terminaison privé et la configuration AMPLS. Ensuite, configurez votre DNS conformément aux instructions de la rubrique [Configuration DNS des points de terminaison privés Azure](../../private-link/private-endpoint-dns.md). Veillez à ne pas créer d’enregistrements vides en préparation de votre configuration de liaison privée. Les enregistrements DNS que vous créez peuvent remplacer les paramètres existants et affecter votre connectivité avec Azure Monitor.
 
   c.    Cliquez sur **Vérifier + créer**.
 
   d.    Laissez le processus de validation se terminer. 
 
   e.    Cliquez sur **Créer**. 

    ![Capture d’écran de la sélection Créer un point de terminaison privé2](./media/private-link-security/ampls-select-private-endpoint-create-5.png)

Vous avez maintenant créé un nouveau point de terminaison privé qui est connecté à cette étendue de liaison privée Azure Monitor.

## <a name="configure-log-analytics"></a>Configurer Log Analytics

Accédez au portail Azure. Dans votre ressource d’espace de travail Log Analytics se trouve un élément de menu **Isolement réseau** sur le côté gauche. Vous pouvez contrôler deux états différents à partir de ce menu.

![Isolement réseau LA](./media/private-link-security/ampls-log-analytics-lan-network-isolation-6.png)

### <a name="connected-azure-monitor-private-link-scopes"></a>Étendues de liaison privée Azure Monitor connectées
Toutes les étendues connectées à cet espace de travail apparaissent dans cet écran. La connexion aux étendues (AMPLS) autorise le trafic réseau à partir du réseau virtuel connecté à chaque AMPLS pour atteindre cet espace de travail. La création d’une connexion de cette manière a le même effet que si la connexion partait de l’étendue, comme c’est la cas dans la section [Connecter des ressources Azure Monitor](#connect-azure-monitor-resources). Pour ajouter une nouvelle connexion, connecterCliquez sur **Ajouter** et sélectionnez l’étendue de liaison privée Azure Monitor. Cliquez sur **Appliquer** pour la connecter. Notez qu’un espace de travail peut se connecter à 5 objets AMPLS, comme expliqué dans [Tenir compte des limites](#consider-limits). 

### <a name="access-from-outside-of-private-links-scopes"></a>Accès depuis l’extérieur des étendues de liaisons privées
Les paramètres dans la partie inférieure de cette page contrôlent l’accès à partir de réseaux publics, ce qui signifie que les réseaux ne sont pas connectés via les étendues répertoriées ci-dessus. Si vous définissez **Autoriser l’accès au réseau public pour l’ingestion** sur **Non**, les machines en dehors des étendues connectées ne peuvent pas charger de données dans cet espace de travail. Si vous définissez **Autoriser l’accès au réseau public pour les requêtes** sur **Non**, les machines en dehors des étendues ne peuvent pas accéder aux données de cet espace de travail, ce qui signifie qu’elles ne peuvent pas interroger ces données. Cela inclut les requêtes des classeurs, les tableaux de bord, les expériences client basées sur l’API, les informations sur le portail Azure et bien plus encore. Les expériences qui s’exécutent en dehors du portail Azure et qui interrogent des données Log Analytics doivent également être exécutées au sein du réseau virtuel connecté par liaison privée.

### <a name="exceptions"></a>Exceptions
La restriction de l’accès comme expliqué ci-dessus ne s’applique pas à Azure Resource Manager et présente donc les limitations suivantes :
* Accès aux données : si des requêtes de blocage/d’autorisation à partir de réseaux publics s’appliquent à la plupart des expériences Log Analytics, certaines expériences interrogent les données via Azure Resource Manager et ne peuvent donc pas interroger les données, sauf si les paramètres de liaison privée sont également appliqués au gestionnaire des ressources (fonctionnalité disponible prochainement). Cela comprend, par exemple, les solutions Azure Monitor, les classeurs et les Insights, ainsi que le connecteur LogicApp.
* Gestion de l’espace de travail : les modifications de paramètre et de configuration, (notamment l’activation ou la désactivation de ces paramètres d’accès), sont gérées par Azure Resource Manager. Limitez l’accès à la gestion de l’espace de travail à l’aide des rôles, autorisations, contrôles réseau et audits appropriés. Pour plus d’informations, consultez [Rôles, autorisations et sécurité Azure Monitor](roles-permissions-security.md).

> [!NOTE]
> Les journaux et les métriques chargés sur un espace de travail par le biais des [paramètres de diagnostic](diagnostic-settings.md) passent par un canal privé sécurisé de Microsoft et ne sont pas contrôlés par ces paramètres.

### <a name="log-analytics-solution-packs-download"></a>Télécharger des packs de solutions Log Analytics

Pour permettre à l’agent Log Analytics de télécharger des packs de solutions, ajoutez les noms de domaine complets appropriés à la liste d’autorisation de votre pare-feu. 


| Environnement cloud | Ressource de l'agent | Ports | Sens |
|:--|:--|:--|:--|
|Azure (public)     | scadvisorcontent.blob.core.windows.net         | 443 | Règle de trafic sortant
|Azure Government | usbn1oicore.blob.core.usgovcloudapi.net | 443 |  Règle de trafic sortant
|Azure China 21Vianet      | mceast2oicore.blob.core.chinacloudapi.cn| 443 | Règle de trafic sortant

## <a name="configure-application-insights"></a>Configurer Application Insights

Accédez au portail Azure. Dans votre ressource de composant Application Insights Azure Monitor se trouve un élément de menu **Isolement réseau** sur le côté gauche. Vous pouvez contrôler deux états différents à partir de ce menu.

![Isolement réseau AI](./media/private-link-security/ampls-application-insights-lan-network-isolation-6.png)

Tout d’abord, vous pouvez connecter cette ressource Application Insights aux étendues de liaison privée Azure Monitor auxquelles vous avez accès. Cliquez sur **Ajouter** et sélectionnez l’**étendue de liaison privée Azure Monitor**. Cliquez sur Appliquer pour la connecter. Toutes les étendues connectées s’affichent dans cet écran. Cette connexion permet au trafic des réseaux virtuels connectés d’atteindre ce composant. Elle a le même effet si la connexion partait de l’étendue, comme c’est la cas dans la section [Connecter des ressources Azure Monitor](#connect-azure-monitor-resources). 

Ensuite, vous pouvez contrôler la façon dont cette ressource peut être atteinte en dehors des étendues de liaison privée mentionnées précédemment. Si vous définissez **Autoriser l’accès au réseau public pour l’ingestion** sur **Non**, les machines ou les Kits de développement logiciel (SDK) en dehors des étendues connectées ne peuvent pas charger de données dans ce composant. Si vous définissez **Autoriser l’accès au réseau public pour les requêtes** sur **Non**, les machines en dehors des étendues ne peuvent pas accéder aux données de cette ressource Application Insights. Ces données incluent l’accès aux journaux APM, aux métriques et au flux de métriques en temps réel, ainsi que des expériences basées sur des classeurs, des tableaux de bord, des expériences client basées sur l’API de requête, des informations sur le portail Azure et bien plus encore. 

Notez que les expériences de consommation hors portail doivent également être exécutées dans le réseau virtuel connecté par liaison privée qui comprend les charges de travail surveillées. 

Vous devrez ajouter des ressources hébergeant les charges de travail surveillées à la liaison privée. Voici de la [documentation](../../app-service/networking/private-endpoint.md) sur la façon de procéder pour les services d’application.

Cette restriction d’accès ne s’applique qu’aux données de la ressource Application Insights. Les modifications de configuration, notamment l’activation ou la désactivation de ces paramètres d’accès, sont gérées par Azure Resource Manager. Au lieu de cela, limitez l’accès à Resource Manager à l’aide des rôles, autorisations, contrôles réseau et audits appropriés. Pour plus d’informations, consultez [Rôles, autorisations et sécurité Azure Monitor](roles-permissions-security.md).

> [!NOTE]
> Pour sécuriser entièrement un espace de travail basé sur Application Insights, vous devez verrouiller à la fois l’accès à la ressource Application Insights et à l’espace de travail Log Analytics sous-jacent.
>
> Les diagnostics au niveau du code (profileur/débogueur) nécessitent que vous fournissiez votre propre compte de stockage pour prendre en charge une liaison privée. Voici de la [documentation](../app/profiler-bring-your-own-storage.md) sur la manière de procéder.

## <a name="use-apis-and-command-line"></a>Utiliser des API et une ligne de commande

Vous pouvez automatiser le processus décrit précédemment en utilisant des modèles Azure Resource Manager et des interfaces de ligne de commande.

Pour créer et gérer des étendues de liaison privée, utilisez [az monitor private-link-scope](/cli/azure/monitor/private-link-scope?view=azure-cli-latest). À l’aide de cette commande, vous pouvez créer des étendues, associer des espaces de travail Log Analytics et des composants Application Insights et ajouter/supprimer/approuver des points de terminaison privés.

Pour gérer l’accès au réseau, utilisez les indicateurs `[--ingestion-access {Disabled, Enabled}]` et `[--query-access {Disabled, Enabled}]` sur les [espaces de travail Log Analytics ](/cli/azure/monitor/log-analytics/workspace?view=azure-cli-latest) ou les [composants Application Insights](/cli/azure/ext/application-insights/monitor/app-insights/component?view=azure-cli-latest).

## <a name="collect-custom-logs-over-private-link"></a>Collecter des journaux personnalisés sur une liaison privée

Des comptes de stockage sont utilisés dans le processus d’ingestion de journaux personnalisés. Par défaut, des comptes de stockage gérés par le service sont utilisés. Toutefois, pour ingérer des journaux personnalisés via des liaisons privées, vous devez utiliser vos propres comptes de stockage et les associer aux espaces de travail Log Analytics. Pour plus d’informations sur la configuration de ces comptes, utilisez la [ligne de commande](/cli/azure/monitor/log-analytics/workspace/linked-storage?view=azure-cli-latest).

Pour plus d’informations sur l’intégration de votre propre compte de stockage, consultez [Comptes de stockage appartenant au client pour l’ingestion des journaux](private-storage.md).

## <a name="restrictions-and-limitations"></a>Limitations et restrictions

### <a name="agents"></a>Agents

Les versions les plus récentes des agents Windows et Linux doivent être utilisées sur des réseaux privés pour permettre une ingestion sécurisée vers les espaces de travail Log Analytics. Les versions antérieures ne peuvent pas charger les données d’analyse dans un réseau privé.

**Agent Log Analytics pour Windows**

Utilisez la version 10.20.18038.0 ou ultérieure de l’agent Log Analytics.

**Agent Log Analytics pour Linux**

Utilisez la version 1.12.25 ou une version ultérieure de l’agent. Si vous ne pouvez pas, exécutez les commandes suivantes sur votre machine virtuelle.

```cmd
$ sudo /opt/microsoft/omsagent/bin/omsadmin.sh -X
$ sudo /opt/microsoft/omsagent/bin/omsadmin.sh -w <workspace id> -s <workspace key>
```

### <a name="azure-portal"></a>Portail Azure

Pour utiliser les expérience du portail Azure Monitor, comme Application Insights et Log Analytics, vous devez autoriser l’accès au portail Azure et aux extensions Azure Monitor sur les réseaux privés. Ajoutez les [étiquettes de service](../../firewall/service-tags.md) **AzureActiveDirectory**, **AzureResourceManager**, **AzureFrontDoor.FirstParty** et **AzureFrontdoor.Frontend** à votre groupe de sécurité réseau.

### <a name="programmatic-access"></a>Accès par programme

Pour utiliser l’API REST, [CLI](/cli/azure/monitor?view=azure-cli-latest) ou PowerShell avec Azure Monitor sur des réseaux privés, ajoutez les [balises de service](../../virtual-network/service-tags-overview.md) **AzureActiveDirectory** et **AzureResourceManager** à votre pare-feu.

L’ajout de ces balises vous permet d’effectuer des actions telles que l’interrogation des données du journal ainsi que la création et la gestion d’espaces de travail Log Analytics et de composants Application Insights.

### <a name="application-insights-sdk-downloads-from-a-content-delivery-network"></a>Téléchargement du Kit de développement logiciel (SDK) Application Insights à partir d’un réseau de distribution de contenu

Regroupez le code JavaScript dans votre script afin que le navigateur ne tente pas de télécharger le code à partir d’un réseau de distribution de contenu. Un exemple est disponible sur le site de [GitHub](https://github.com/microsoft/ApplicationInsights-JS#npm-setup-ignore-if-using-snippet-setup).

### <a name="browser-dns-settings"></a>Paramètres DNS du navigateur

Si vous vous connectez à vos ressources de Azure Monitor via une liaison privée, le trafic vers ces ressources doit traverser le point de terminaison privé configuré sur votre réseau. Pour activer le point de terminaison privé, mettez à jour vos paramètres DNS en procédant de la manière décrite dans [Se connecter à un point de terminaison privé](#connect-to-a-private-endpoint). Certains navigateurs utilisent leurs propres paramètres DNS à la place de ceux que vous définissez. Le navigateur peut tenter de se connecter à des points de terminaison publics d’Azure Monitor et ignorer entièrement la liaison privée. Vérifiez que les paramètres de votre navigateur ne remplacent pas ou ne mettent pas en cache d’anciens paramètres DNS. 

## <a name="next-steps"></a>Étapes suivantes

- Découvrir le [stockage privé](private-storage.md)