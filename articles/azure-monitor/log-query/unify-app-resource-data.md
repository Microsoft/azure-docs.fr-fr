---
title: Unifier plusieurs ressources Application Insights Azure Monitor | Microsoft Docs
description: Cet article explique en détail comment utiliser une fonction dans les journaux d’activité Azure Monitor pour interroger plusieurs ressources Application Insights et visualiser ces données.
author: bwren
ms.author: bwren
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 02/02/2020
ms.openlocfilehash: 360578a36b92711c55b1fc65befa1b3df7927aad
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91330891"
---
# <a name="unify-multiple-azure-monitor-application-insights-resources"></a>Unifier plusieurs ressources Application Insights Azure Monitor 
Cet article décrit comment interroger et afficher toutes vos données du journal Application Insights à partir d’un emplacement unique, même quand elles se trouvent dans différents abonnements Azure, en remplacement de la dépréciation d’Application Insights Connector. Le nombre de ressources Application Insights que vous pouvez inclure dans une seule requête est limité à 100.

## <a name="recommended-approach-to-query-multiple-application-insights-resources"></a>Approche recommandée pour interroger plusieurs ressources Application Insights 
Lister plusieurs ressources Application Insights dans une requête peut être une opération fastidieuse et difficile à gérer. Au lieu de cela, vous pouvez utiliser une fonction pour séparer la logique de requête de l’étendue des applications.  

Cet exemple montre comment superviser plusieurs ressources Application Insights et visualiser le nombre de requêtes ayant échoué par nom d’application.

Créez une fonction en utilisant l'opérateur union avec la liste des applications, puis enregistrez la requête en tant que fonction dans votre espace de travail avec l'alias *applicationsScoping*. 

Vous pouvez à tout moment modifier les applications répertoriées sur le portail en accédant à l'Explorateur de requêtes de votre espace de travail, en sélectionnant la fonction à modifier et en enregistrant. Vous pouvez également utiliser la cmdlet PowerShell `SavedSearch`. 

>[!NOTE]
>Cette méthode ne peut pas être utilisée avec les alertes de journal, car la validation d’accès des ressources de règle d’alerte, notamment les espaces de travail et les applications, s’effectue au moment de la création de l’alerte. L’ajout de nouvelles ressources à la fonction après la création de l’alerte n’est pas pris en charge. Si vous préférez utiliser la fonction pour l’étendue des ressources dans les alertes de journal, vous devez modifier la règle d’alerte dans le portail ou à l’aide d’un modèle Resource Manager pour mettre à jour les ressources délimitées. Vous pouvez également inclure la liste des ressources dans la requête d’alerte de journal.

La commande `withsource= SourceApp` ajoute une colonne aux résultats, qui désigne l’application ayant envoyé le journal. L’opérateur parse est facultatif dans cet exemple ; il extrait le nom de l’application à partir de la propriété SourceApp. 

```
union withsource=SourceApp 
app('Contoso-app1').requests,  
app('Contoso-app2').requests, 
app('Contoso-app3').requests, 
app('Contoso-app4').requests, 
app('Contoso-app5').requests 
| parse SourceApp with * "('" applicationName "')" *  
```

Vous êtes maintenant prêt à utiliser la fonction applicationsScoping dans la requête interressource :  

```
applicationsScoping 
| where timestamp > ago(12h)
| where success == 'False'
| parse SourceApp with * '(' applicationName ')' * 
| summarize count() by applicationName, bin(timestamp, 1h) 
| render timechart
```

La requête utilise le schéma Application Insights, bien qu’elle soit exécutée dans l’espace de travail, car la fonction applicationsScoping retourne la structure de données Application Insights. L’alias de fonction retourne l’union des requêtes à partir de toutes les applications définies. Ensuite, la requête filtre les demandes ayant échoué et permet de visualiser les tendances par application.

![Exemple de résultats de requête croisée](media/unify-app-resource-data/app-insights-query-results.png)

>[!NOTE]
>[Les requêtes de ressources croisées](./cross-workspace-query.md) dans les alertes de journal sont uniquement prises en charge dans l’[API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules) actuelle. Si vous utilisez l’API Log Analytics Alerts héritée, vous devez [basculer sur l’API actuelle](../platform/alerts-log-api-switch.md). [Voir des exemples de modèles](../platform/alerts-log-create-templates.md).

## <a name="application-insights-and-log-analytics-workspace-schema-differences"></a>Différences entre les schémas d’espace de travail Log Analytics et Application Insights
Le tableau suivant montre les différences entre les schémas Log Analytics et Application Insights.  

| Propriétés d’espace de travail Log Analytics| Propriétés de ressources Application Insights|
|------------|------------| 
| AnonUserId | user_id|
| ApplicationId | appId|
| ApplicationName | appName|
| ApplicationTypeVersion | application_Version |
| AvailabilityCount | itemCount |
| AvailabilityDuration | duration |
| AvailabilityMessage | message |
| AvailabilityRunLocation | location |
| AvailabilityTestId | id |
| AvailabilityTestName | name |
| AvailabilityTimestamp | timestamp |
| Browser | client_browser |
| City | client_city |
| ClientIP | client_IP |
| Computer | cloud_RoleInstance | 
| Country | client_CountryOrRegion | 
| CustomEventCount | itemCount | 
| CustomEventDimensions | customDimensions |
| CustomEventName | name | 
| DeviceModel | client_Model | 
| DeviceType | client_Type | 
| ExceptionCount | itemCount | 
| ExceptionHandledAt | handledAt |
| ExceptionMessage | message | 
| ExceptionType | type |
| OperationID | operation_id |
| NomOpération | operation_Name | 
| Système d''exploitation | client_OS | 
| PageViewCount | itemCount |
| PageViewDuration | duration | 
| PageViewName | name | 
| ParentOperationID | operation_Id | 
| RequestCount | itemCount | 
| RequestDuration | duration | 
| RequestID | id | 
| RequestName | name | 
| RequestSuccess | success | 
| ResponseCode | resultCode | 
| Role | cloud_RoleName |
| RoleInstance | cloud_RoleInstance |
| SessionId | session_Id | 
| SourceSystem | operation_SyntheticSource |
| TelemetryTYpe | type |
| URL | url |
| UserAccountId | user_AccountId |

## <a name="next-steps"></a>Étapes suivantes

Utilisez [Recherche dans les journaux](./log-query-overview.md) pour afficher des informations détaillées sur vos applications Application Insights.

