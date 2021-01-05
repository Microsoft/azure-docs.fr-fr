---
title: Migration du portail OMS vers Azure | Microsoft Docs
description: Le portail OMS devient caduque et toutes ses fonctionnalités migrent vers le portail Azure. Cet article fournit des détails sur cette transition.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/22/2019
ms.openlocfilehash: ba81e0bd74390d8b20803f8f9cf6e9f2ae9f0eab
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184516"
---
# <a name="oms-portal-moving-to-azure"></a>Migration du portail OMS vers Azure

> [!NOTE]
> Sauf indication contraire, cet article s’applique au cloud du secteur public et au cloud public Azure.

**Le portail Microsoft Operations Management Suite pour le cloud public Azure a été officiellement retiré. Le portail Microsoft Operations Management Suite pour le cloud Azure - Gouvernement des États-Unis a été officiellement retiré le 15 mai 2019.** Nous sommes heureux de migrer vers le portail Azure et pensons que la transition sera facile. Mais nous comprenons que les changements sont difficiles et peuvent entraîner des perturbations. Le reste de cet article passe en revue les scénarios clés et la feuille de route pour cette transition.

Le portail Azure est le hub de tous les services Azure qui propose une expérience de gestion complète avec des fonctionnalités telles que des tableaux de bord pour épingler les ressources, la recherche intelligente pour rechercher des ressources et le balisage pour gérer les ressources. Pour consolider et rationaliser le flux de travail de monitoring et de gestion, nous avons commencé à ajouter les fonctionnalités du portail OMS dans le portail Azure. Toutes les fonctionnalités du portail OMS font désormais partie du portail Azure. En fait, certaines des nouvelles fonctionnalités, comme Traffic Analytics, ne sont disponibles que sur le Portail Azure. Vous pouvez faire tout ce que vous faisiez dans le portail OMS avec le portail Azure, et bien plus encore. Si ce n’est déjà fait, vous devriez commencer à utiliser le portail Azure dès aujourd’hui !

## <a name="what-is-changing"></a>Qu’est-ce qui change ? 
Les modifications suivantes sont annoncées avec l’abandon du portail OMS. Chacun de ces changements est décrit en détail dans les sections ci-dessous.

- Vous pouvez créer des [espaces de travail uniquement](#new-workspaces) dans le portail Azure.
- La nouvelle expérience de gestion des alertes [remplace la solution Alert Management](#changes-to-alerts).
- [La gestion de l’accès utilisateur](#user-access-and-role-migration) s’effectue désormais dans le portail Azure avec le contrôle d’accès en fonction du rôle Azure.
- [Application Insights Connector n’est plus nécessaire](#application-insights-connector-and-solution), dans la mesure où une même fonctionnalité est activée avec des requêtes portant sur plusieurs espaces de travail.
- [L’application mobile OMS](#oms-mobile-app) est dépréciée. 
- La [solution NSG est remplacée](#azure-network-security-group-analytics) par une fonctionnalité améliorée disponible via la solution Traffic Analytics.
- Les nouvelles connexions de System Center Operations Manager à Log Analytics exigent des [packs d’administration à jour](#system-center-operations-manager).
- Pour plus d’informations sur les changements apportés à [Update Management](../../automation/update-management/overview.md), consultez [Migrer vos déploiements de mises à jour OMS vers Azure](../../automation/migrate-oms-update-deployments.md).


## <a name="what-should-i-do-now"></a>Que dois-je savoir ?
Même si vous pouvez continuer à utiliser la plupart des fonctionnalités sans effectuer de migration, vous devez accomplir les tâches suivantes :

- Vous devez [migrer vos autorisations utilisateur](#user-access-and-role-migration) vers le portail Azure.
- Pour plus d’informations sur la transition de la solution Update Management, consultez [Migrer vos déploiements de mises à jour OMS vers Azure](../../automation/migrate-oms-update-deployments.md).

Référez-vous à [Questions courantes sur la transition du portail OMS vers le portail Azure pour les utilisateurs Log Analytics](../overview.md) pour plus d’informations sur la transition vers le portail Azure. 

## <a name="user-access-and-role-migration"></a>Accès utilisateur et migration du rôle
La gestion des accès du portail Azure est plus complète et plus puissante que la gestion des accès du portail OMS. Pour plus d’informations sur la gestion des accès dans Log Analytics, voir [Conception de votre espace de travail de journaux Azure](design-logs-deployment.md).

> [!NOTE]
> Les versions précédentes de cet article indiquaient que les autorisations seraient automatiquement converties depuis le portail OMS vers le portail Azure. Cette conversion automatique n’étant plus planifiée, vous devez effectuer la conversion vous-même.

Si vous avez déjà un accès approprié dans le portail Azure, vous n’avez pas besoin de changer quoi que ce soit. Dans les rares cas où vous pouvez ne pas avoir un accès approprié, votre administrateur doit vous attribuer des autorisations.

- Vous disposez d’autorisations utilisateur en lecture seule dans le portail OMS, mais d’aucune autorisation dans le portail Azure. 
- Vous disposez d’autorisations contributeur dans le portail OMS, mais uniquement d’un accès en lecture dans le portail Azure.
 
Dans ces deux cas, votre administrateur doit vous assigner manuellement le rôle approprié à partir du tableau suivant. Nous vous recommandons d’assigner ce rôle au niveau de l’abonnement ou du groupe de ressources.  Nous fournirons sous peu davantage de conseils pour ces deux cas.

| Autorisation dans le portail OMS | Rôle Azure |
|:---|:---|
| Lecture seule | Lecteur Log Analytics |
| Contributeur | Contributeur Log Analytics |
| Administrateur | Propriétaire | 
 

## <a name="new-workspaces"></a>Nouveaux espaces de travail
Vous ne pouvez plus créer d’espaces de travail à l’aide du portail OMS. Pour créer un espace de travail sur le Portail Azure, voir [Créer un espace de travail Log Analytics sur le Portail Azure](../learn/quick-create-workspace.md).

## <a name="changes-to-alerts"></a>Changements dans les alertes

### <a name="alert-extension"></a>Extension d’alerte  

Les alertes ont été [étendue dans le portail Azure](./alerts-unified-log.md) Les alertes existantes continueront d’être répertoriées dans le portail Microsoft Operations Management Suite, mais ne vous pourrez les gérer que dans le portail Azure. Si vous accédez aux alertes par programme à l’aide de l’API REST Log Analytics Alert ou du modèle Log Analytics Alert Resource, vous devrez utiliser des groupes d’actions à la place des actions dans vos appels d’API, modèles Azure Resource Manager et commandes PowerShell.

### <a name="alert-management-solution"></a>solution de gestion des alertes
Contrairement à ce qui avait été annoncé précédemment, la [solution de gestion des alertes](alert-management-solution.md) restera disponible et entièrement prise en charge dans le portail Azure. Vous pouvez continuer d'installer la solution depuis Place de marché Azure.

Bien que la solution de gestion des alertes reste disponible, nous vous encourageons à utiliser [l'interface d'alerte unifiée d'Azure Monitor](alerts-overview.md) pour visualiser et gérer toutes les alertes dans Azure. Cette nouvelle expérience regroupe nativement les alertes à partir de plusieurs sources dans Azure, notamment les alertes de journal de Log Analytics. Si vous utilisez l’interface d’alerte unifiée d’Azure Monitor, la solution de gestion des alertes est uniquement requise pour permettre l’intégration des alertes entre System Center Operations Manager et Azure. Dans l'interface d'alerte unifiée d'Azure Monitor, vous pouvez voir les distributions de vos alertes, tirer parti du regroupement automatique des alertes associées via des groupes intelligents et afficher les alertes entre plusieurs abonnements lors de l’application de filtres riches. Cette nouvelle expérience permettra de bénéficier des nouvelles avancées en matière de gestion des alertes. 

Les données collectées par la solution Alert Management (enregistrements avec un type d’alerte) se trouvent toujours dans Log Analytics tant que la solution est installée pour l’espace de travail. 

## <a name="oms-mobile-app"></a>Application mobile OMS
L’application mobile OMS deviendra caduque, ainsi que le portail OMS. Au lieu de l’application mobile OMS, pour accéder aux informations sur votre infrastructure informatique, tableaux de bord et requêtes enregistrées, vous pouvez accéder au portail Azure directement à partir de votre navigateur sur votre appareil mobile. Pour obtenir des alertes, vous devez configurer des [groupes d’actions Azure](action-groups.md) pour recevoir des notifications sous la forme d’un appel vocal ou d’un SMS

## <a name="application-insights-connector-and-solution"></a>Application Insights Connector et solution
[Application Insights Connector](app-insights-connector.md) permet d’inclure des données Application Insights dans un espace de travail Log Analytics. Cette duplication des données était nécessaire pour activer la visibilité sur les données d’infrastructure et d’application. Avec la prise en charge de la conservation des données étendue jusqu’en mars 2019 et la possibilité d’effectuer des [requêtes couvrant plusieurs ressources](../log-query/cross-workspace-query.md), en plus de pouvoir [voir plusieurs ressources Azure Monitor Application Insights](../log-query/unify-app-resource-data.md), il n’est pas nécessaire de dupliquer les données de vos ressources Application Insights et de les envoyer vers Log Analytics. En outre, le connecteur envoie à Log Analytics un sous-ensemble des propriétés des applications, et les requêtes couvrant plusieurs ressources vous offrent plus de flexibilité.  

Dès lors, Application Insights Connector a été déconseillé et supprimé de Place de marché Azure, de même que du portail Microsoft Operations Management Suite le 30 mars 2019. Les connexions existantes continueront de fonctionner jusqu'au 30 juin 2019. Une fois le portail OMS déconseillé, il ne sera pas possible de configurer et de supprimer les connexions existantes du portail. Cela sera pris en charge par l'API REST qui sera disponible en janvier 2019, et une notification sera publiée sur les [mises à jour Azure](https://azure.microsoft.com/updates/). 

## <a name="azure-network-security-group-analytics"></a>Azure Network Security Group Analytics
La [solution Azure Network Security Group Analytics](../insights/azure-networking-analytics.md#azure-network-security-group-analytics-solution-in-azure-monitor) sera remplacée par [Traffic Analytics](https://azure.microsoft.com/blog/traffic-analytics-in-preview/) récemment lancé qui offre une visibilité sur l’activité des utilisateurs et des applications sur les réseaux cloud. Traffic Analytics vous permet d’auditer l’activité réseau de votre organisation, de sécuriser les applications et les données, d’optimiser les performances de la charge de travail et de rester en conformité. 

Cette solution analyse les journaux de flux NSG et fournit des insights sur les éléments suivants.

- Les flux de trafic sur vos réseaux entre Azure et Internet, les régions de cloud public, les réseaux virtuels et les sous-réseaux.
- Les applications et les protocoles sur votre réseau, sans renifleur ni appliance de collection de flux dédiée.
- Les principaux consommateurs, les applications bavardes, les conversations de machine virtuelle dans le cloud, les zones réactives du trafic.
- Les sources et destinations du trafic entre les réseaux virtuels, les relations entre les services métier critiques et les applications.
- La sécurité, comme le trafic malveillant, les ports ouverts sur Internet, les applications ou machines virtuelles tentant d’accéder à Internet.
- L’utilisation de la capacité, qui vous aide à éliminer les problèmes de sur-provisionnement ou de sous-utilisation.

Vous pouvez continuer à utiliser les Paramètres de diagnostics pour envoyer les journaux d’activité NSG à Log Analytics, afin que vos recherches enregistrées, alertes, tableaux de bord existants continuent de fonctionner. Les clients qui ont déjà installé la solution peuvent continuer à l’utiliser jusqu’à nouvel ordre. À partir du 5 septembre, la solution Network Security Group Analytics sera supprimée de la place de marché et mise à disposition via la communauté sous forme de [Modèle de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Operationalinsights).

## <a name="system-center-operations-manager"></a>System Center Operations Manager
Si le [groupe d’administration Operations Manager est déjà connecté à Log Analytics](om-agents.md), il continuera de fonctionner sans modification. Pour les nouvelles connexions en revanche, suivez les instructions du [Pack d’administration Microsoft System Center Operations Manager pour configurer Operations Management Suite](https://techcommunity.microsoft.com/t5/system-center-blog/bg-p/SystemCenterBlog).

## <a name="next-steps"></a>Étapes suivantes
- Consultez [Questions courantes sur la transition du portail OMS vers le portail Azure pour les utilisateurs Log Analytics](../overview.md) pour obtenir des conseils sur la transition depuis le portail OMS vers le portail Azure.