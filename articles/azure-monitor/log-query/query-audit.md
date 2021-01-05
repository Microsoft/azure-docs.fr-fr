---
title: Requêtes d’audit dans les requêtes de journal Azure Monitor
description: Informations sur les journaux d’audit qui fournissent des données de télémétrie sur les requêtes de journal exécutées dans Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/03/2020
ms.openlocfilehash: 7ce0aea6bb257f0a52a843ce53cc904ed0a775dd
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95536198"
---
# <a name="audit-queries-in-azure-monitor-logs-preview"></a>Requêtes d’audit dans les journaux Azure Monitor (préversion)
Les journaux d’audit des requêtes de journaux fournissent des données de télémétrie sur les requêtes de journal exécutées dans Azure Monitor : par exemple, le moment où la requête a été exécutée, l’utilisateur qui l’a exécutée, l’outil utilisé, le texte de la requête et les statistiques de performances qui décrivent l’exécution de la requête.


## <a name="configure-query-auditing"></a>Configuration de l’audit des requêtes
L’audit des requêtes s’active au moyen d’un [paramètre de diagnostic](../platform/diagnostic-settings.md) de l’espace de travail Log Analytics. Elle vous permet d’envoyer des données d’audit à l’espace de travail actuel ou à tout autre espace de travail de votre abonnement, à Azure Event Hubs pour un envoi en dehors d’Azure ou au Stockage Azure à des fins d’archivage. 

### <a name="azure-portal"></a>Portail Azure
Accédez au paramètre de diagnostic d’un espace de travail Log Analytics sur le Portail Azure à l’un des emplacements suivants :

- Dans le menu **Azure Monitor** , sélectionnez **Paramètres de diagnostic**, puis recherchez et sélectionnez l’espace de travail.

    [![Paramètres de diagnostic – Azure Monitor](media/log-query-audit/diagnostic-setting-monitor.png) ](media/log-query-audit/diagnostic-setting-monitor.png#lightbox) 

- Dans le menu **Espaces de travail Log Analytics**, sélectionnez l’espace de travail, puis sélectionnez **Paramètres de diagnostic**.

    [![Paramètres de diagnostic – Espace de travail Log Analytics](media/log-query-audit/diagnostic-setting-workspace.png) ](media/log-query-audit/diagnostic-setting-workspace.png#lightbox) 

### <a name="resource-manager-template"></a>Modèle Resource Manager
Pour voir un exemple de modèle Resource Manager, consultez [Paramètre de diagnostic de l’espace de travail Log Analytics](../samples/resource-manager-diagnostic-settings.md#diagnostic-setting-for-log-analytics-workspace).

## <a name="audit-data"></a>Données d’audit
Un enregistrement d’audit est créé chaque fois qu’une requête s’exécute. Si vous envoyez les données à un espace de travail Log Analytics, elles sont stockées dans une table nommée *LAQueryLogs*. Le tableau suivant décrit les propriétés de chacun des enregistrements des données d’audit.

| Champ | Description |
|:---|:---|
| TimeGenerated         | Heure UTC à laquelle la requête a été envoyée. |
| CorrelationId         | ID unique d’identification de la requête. Il peut être utilisé dans les scénarios de résolution des problèmes pour demander de l’aide à Microsoft. |
| AADObjectId           | ID Azure Active Directory du compte d’utilisateur qui a lancé la requête.  |
| AADTenantId           | ID du locataire du compte d’utilisateur qui a lancé la requête.  |
| AADEmail              | Adresse e-mail du locataire du compte d’utilisateur qui a lancé la requête.  |
| AADClientId           | ID et nom résolu de l’application utilisée pour lancer la requête. |
| RequestClientApp      | Nom résolu de l’application utilisée pour lancer la requête. |
| QueryTimeRangeStart   | Début de l’intervalle de temps sélectionné pour la requête. Il existe des cas où ce champ n’est pas rempli, par exemple lorsque la requête est lancée à partir de Log Analytics et que l’intervalle de temps est spécifié à l’intérieur de la requête plutôt que dans le sélecteur d’heure. |
| QueryTimeRangeEnd     | Fin de l’intervalle de temps sélectionné pour la requête. Il existe des cas où ce champ n’est pas rempli, par exemple lorsque la requête est lancée à partir de Log Analytics et que l’intervalle de temps est spécifié à l’intérieur de la requête plutôt que dans le sélecteur d’heure.  |
| QueryText             | Texte de la requête qui a été exécutée. |
| RequestTarget         | URL de l’API utilisée pour envoyer la requête.  |
| RequestContext        | Liste des ressources sur lesquelles la requête doit s’exécuter. Ce champ peut contenir trois tableaux de chaînes maximum : espaces de travail, applications et ressources. Les ressources ciblant un abonnement ou un groupe de ressources apparaissent sous l’intitulé *ressources*. La cible impliquée par RequestTarget est incluse.<br>L’ID de ressource de chaque ressource sera inclus s’il peut être résolu. Il est possible qu’il ne puisse pas être résolu si une erreur est retournée lors de l’accès à la ressource. Dans ce cas, le texte spécifique à partir de la requête sera utilisé.<br>Si la requête utilise un nom ambigu, tel qu’un nom d’espace de travail existant dans plusieurs abonnements, ce nom ambigu sera utilisé. |
| RequestContextFilters | Ensemble de filtres spécifiés dans le cadre de l’appel de requête. Ce champ peut contenir trois tableaux de chaînes maximum :<br>- ResourceTypes : type de ressource permettant de limiter l’étendue de la requête<br>- Workspaces : liste des espaces de travail auxquels la requête sera limitée<br>- WorkspaceRegions : liste des régions d’espaces de travail permettant de limiter la requête |
| ResponseCode          | Code de réponse HTTP renvoyé lors de l’envoi de la requête. |
| ResponseDurationMs    | Heure à laquelle la réponse doit être retournée.  |
| ResponseRowCount     | Nombre total de lignes renvoyées par la requête. |
| StatsCPUTimeMs       | Temps de calcul total utilisé pour le calcul, l’analyse et l’extraction de données. Ce champ n’est renseigné que si la requête retourne le code de statut 200. |
| StatsDataProcessedKB | Volume de données consultées pour traiter la requête. Influencé par la taille de la table cible, l’intervalle de temps utilisé, les filtres appliqués et le nombre de colonnes référencées. Ce champ n’est renseigné que si la requête retourne le code de statut 200. |
| StatsDataProcessedStart | Heure des données les plus anciennes consultées pour traiter la requête. Ce champ est influencé par l’intervalle de temps explicite de la requête et par les filtres appliqués. Il peut être plus élevé que l’intervalle de temps explicite en raison du partitionnement des données. Ce champ n’est renseigné que si la requête retourne le code de statut 200. |
| StatsDataProcessedEnd  |Heure des données les plus récentes consultées pour traiter la requête. Ce champ est influencé par l’intervalle de temps explicite de la requête et par les filtres appliqués. Il peut être plus élevé que l’intervalle de temps explicite en raison du partitionnement des données. Ce champ n’est renseigné que si la requête retourne le code de statut 200. |
| StatsWorkspaceCount | Nombre d’espaces de travail auxquels la requête accède. Ce champ n’est renseigné que si la requête retourne le code de statut 200. |
| StatsRegionCount | Nombre de régions auxquels la requête accède. Ce champ n’est renseigné que si la requête retourne le code de statut 200. |

## <a name="considerations"></a>Considérations

- Les requêtes sont journalisées uniquement lorsqu’elles sont exécutées dans un contexte utilisateur. Aucune requête de service à service dans Azure n’est journalisée. Les deux principaux ensembles de requêtes que cette exclusion englobe sont les calculs de facturation et les exécutions d’alertes automatisées. Dans le cas des alertes, seule la requête d’alerte planifiée elle-même n’est pas journalisée. L’exécution initiale de l’alerte dans l’écran de création d’alerte est effectuée dans un contexte utilisateur et disponible à des fins d’audit. 
- Les statistiques de performances ne sont pas disponibles pour les requêtes provenant du proxy Azure Data Explorer. L’intégralité des autres données pour ces requêtes seront quand même remplies.
- L’indicateur *h* sur les chaînes, qui [obfusque les littéraux de chaîne](/azure/data-explorer/kusto/query/scalar-data-types/string#obfuscated-string-literals), n’aura pas d’effet sur les journaux d’audit de requête. Les requêtes sont capturées exactement comme elles sont envoyées, sans que la chaîne soit obfusquée. Vous devez vous assurer que seuls les utilisateurs disposant de droits de conformité pour voir ces données peuvent le faire au moyen des différents modes Kubernetes RBAC ou Azure RBAC disponibles dans les espaces de travail Log Analytics.
- Pour les requêtes qui comportent des données provenant de plusieurs espaces de travail, la requête sera uniquement capturée dans les espaces de travail auxquels l’utilisateur a accès.

## <a name="costs"></a>Coûts  
Il n’y a aucun coût lié à l’extension Diagnostics Azure, mais vous pouvez être facturé pour les données ingérées. Consultez les [tarifs Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) appliqués pour la destination dans laquelle vous collectez les données.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [paramètres de diagnostic](../platform/diagnostic-settings.md).
- En savoir plus sur [l’optimisation des requêtes de journal](query-optimization.md).
