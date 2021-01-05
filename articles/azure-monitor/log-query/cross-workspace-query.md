---
title: Interroger des ressources avec Azure Monitor | Microsoft Docs
description: Cet article explique comment interroger les ressources à partir de plusieurs espaces de travail et d’une application App Insights incluse dans votre abonnement.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/01/2020
ms.openlocfilehash: 8eb163c95fb1426ebae8956d50f6d8f6aec6fd7f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013647"
---
# <a name="perform-log-query-in-azure-monitor-that-span-across-workspaces-and-apps"></a>Exécuter une requête de journal dans Azure Monitor qui s’étend sur plusieurs espaces de travail et applications

Les journaux Azure Monitor prennent en charge des requêtes sur plusieurs espaces de travail Log Analytics et l’application Application Insights dans le même groupe de ressources, un autre groupe de ressources ou un autre abonnement. Cela vous donne une vue de vos données à l’échelle du système.

Il existe deux méthodes pour interroger des données stockées dans plusieurs espaces de travail et applications :
1. Explicitement en spécifiant les détails de l’espace de travail et de l’application. Cette technique est détaillée dans cet article.
2. Implicitement à l’aide de [requêtes du contexte des ressources](../platform/design-logs-deployment.md#access-mode). Lorsque vous interrogez dans le contexte d’une ressource, d’un groupe de ressources ou d’un abonnement spécifique, les données pertinentes sont extraites de tous les espaces de travail contenant des données pour ces ressources. Les données Application Insights stockées dans des applications ne sont pas extraites.

> [!IMPORTANT]
> Si vous utilisez une [ressource Application Insights basée sur un espace de travail](../app/create-workspace-resource.md), la télémétrie est stockée dans un espace de travail Log Analytics avec toutes les autres données de journal. Utilisez l’expression log () pour écrire une requête incluant une application dans plusieurs espaces de travail. En présence de plusieurs applications dans le même espace de travail, vous n’avez pas besoin de requête entre espaces de travail.


## <a name="cross-resource-query-limits"></a>Limites de requête inter-ressources 

* Le nombre de ressources Application Insights et d’espaces de travail Log Analytics que vous pouvez inclure dans une seule requête est limité à 100.
* Les requêtes inter-ressources ne sont pas prises en charge dans le Concepteur de vue. Vous pouvez créer une requête dans Log Analytics et l’épingler au tableau de bord Azure pour [visualiser une requête de journal](../learn/tutorial-logs-dashboards.md). 
* Les requêtes de ressources croisées dans les alertes de journal sont uniquement prises en charge dans l’[API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules) actuelle. Si vous utilisez l’API Log Analytics Alerts héritée, vous devez [basculer sur l’API actuelle](../platform/alerts-log-api-switch.md).


## <a name="querying-across-log-analytics-workspaces-and-from-application-insights"></a>Interrogation de plusieurs espaces de travail Log Analytics à partir d’Application Insights
Pour référencer un autre espace de travail dans votre requête, utilisez l’identificateur [*workspace*](./workspace-expression.md). Pour une application Application Insights, utilisez l’identificateur [*app*](./app-expression.md).  

### <a name="identifying-workspace-resources"></a>Identification des ressources d’espace de travail
Les exemples suivants illustrent des requêtes exécutées sur des espaces de travail Log Analytics dans le but de retourner les décomptes synthétiques des journaux d’activité à partir de la table Update d’un espace de travail nommé *contosoretail-it*. 

Il est possible d’identifier un espace de travail de plusieurs manières :

* Nom de la ressource : il s’agit d’un nom convivial de l’espace de travail, parfois appelé *nom du composant*. 

    `workspace("contosoretail-it").Update | count`

* Nom qualifié : il s’agit du « nom complet » de l’espace de travail, composé du nom de l’abonnement, du groupe de ressources et du nom du composant au format suivant : *nomAbonnement/groupeRessources/nomComposant*. 

    `workspace('contoso/contosoretail/contosoretail-it').Update | count`

    >[!NOTE]
    >Étant donné que les noms des abonnements Azure ne sont pas uniques, cet identificateur peut être ambigu. 
    >

* ID de l’espace de travail : il s’agit de l’identificateur unique et immuable affecté à chaque espace de travail, représenté sous la forme d’un identificateur global unique (GUID).

    `workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update | count`

* ID de ressource Azure : c’est l’identité unique de l’espace de travail, définie par Azure. Vous utilisez l’ID de ressource quand le nom de la ressource est ambigu.  Pour les espaces de travail, le format est : */subscriptions/idAbonnement/resourcegroups/groupeRessources/providers/microsoft.OperationalInsights/workspaces/nomComposant*.  

    Par exemple :
    ``` 
    workspace("/subscriptions/e427519-5645-8x4e-1v67-3b84b59a1985/resourcegroups/ContosoAzureHQ/providers/Microsoft.OperationalInsights/workspaces/contosoretail-it").Update | count
    ```

### <a name="identifying-an-application"></a>Identification d’une application
L’exemple suivant retourne un décompte synthétique des demandes adressées à une application nommée *fabrikamapp* dans Application Insights. 

Vous pouvez identifier une application dans Application Insights avec l’expression *app(Identifier)* .  L’argument *Identifier* spécifie l’application à l’aide de l’un éléments suivants :

* Nom de la ressource : il s’agit d’un nom convivial de l’application, parfois appelé *nom du composant*.  

    `app("fabrikamapp")`

    >[!NOTE]
    >L’identification d’une application par son nom est supposée être unique pour tous les abonnements accessibles. Si plusieurs applications portent le nom spécifié, la requête échoue en raison de l’ambiguïté. Dans ce cas, vous devez utiliser l’un des autres identificateurs.

* Nom qualifié : il s’agit du « nom complet » de l’application, composé du nom de l’abonnement, du groupe de ressources et du nom du composant au format suivant : *nomAbonnement/groupeRessources/nomComposant*. 

    `app("AI-Prototype/Fabrikam/fabrikamapp").requests | count`

     >[!NOTE]
    >Étant donné que les noms des abonnements Azure ne sont pas uniques, cet identificateur peut être ambigu. 
    >

* ID : GUID de l’application.

    `app("b459b4f6-912x-46d5-9cb1-b43069212ab4").requests | count`

* ID de ressource Azure : identité unique de l’application, définie par Azure. Vous utilisez l’ID de ressource quand le nom de la ressource est ambigu. Le format est le suivant : */subscriptions/ID_abonnement/resourcegroups/Groupe_Ressources/providers/microsoft. OperationalInsights/components/Nom_Composant*.  

    Par exemple :
    ```
    app("/subscriptions/b459b4f6-912x-46d5-9cb1-b43069212ab4/resourcegroups/Fabrikam/providers/microsoft.insights/components/fabrikamapp").requests | count
    ```

### <a name="performing-a-query-across-multiple-resources"></a>Exécution d’une requête sur plusieurs ressources
Vous pouvez interroger plusieurs ressources à partir d’une de vos instances de ressource. Il peut s’agir d’un mélange d’espaces de travail et d’applications.
    
Exemple de requête sur deux espaces de travail :    

```
union Update, workspace("contosoretail-it").Update, workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update
| where TimeGenerated >= ago(1h)
| where UpdateState == "Needed"
| summarize dcount(Computer) by Classification
```

## <a name="using-cross-resource-query-for-multiple-resources"></a>Utilisation d’une requête multiressource sur plusieurs ressources
Quand vous utilisez une requête inter-ressources pour mettre en corrélation des données provenant de plusieurs espaces de travail Log Analytics et ressources Application Insights, la requête peut devenir complexe et difficile à gérer. Vous devez tirer parti des [fonctions des requêtes de journal d’Azure Monitor](functions.md) pour séparer la logique de la requête de l’étendue des ressources de la requête, simplifiant ainsi la structure de la requête. L’exemple suivant montre comment superviser plusieurs ressources Application Insights et visualiser le nombre de demandes ayant échoué par nom d’application. 

Créez une requête semblable à la suivante qui référence l’étendue des ressources Application Insights. La commande `withsource= SourceApp` ajoute une colonne qui désigne le nom de l’application ayant envoyé le journal. [Enregistrez la requête en tant que fonction](functions.md#create-a-function) avec l’alias _applicationsScoping_.

```Kusto
// crossResource function that scopes my Application Insights resources
union withsource= SourceApp
app('Contoso-app1').requests, 
app('Contoso-app2').requests,
app('Contoso-app3').requests,
app('Contoso-app4').requests,
app('Contoso-app5').requests
```



Vous pouvez à présent [utiliser cette fonction](./functions.md#use-a-function) dans une requête multiressource comme suit. L’alias de fonction _applicationsScoping_ retourne l’union de la table requests à partir de toutes les applications définies. Ensuite, la requête filtre les demandes ayant échoué et permet de visualiser les tendances par application. L’opérateur _parse_ est facultatif dans cet exemple. Il extrait le nom de l’application de la propriété _SourceApp_.

```Kusto
applicationsScoping 
| where timestamp > ago(12h)
| where success == 'False'
| parse SourceApp with * '(' applicationName ')' * 
| summarize count() by applicationName, bin(timestamp, 1h) 
| render timechart
```

>[!NOTE]
> Cette méthode ne peut pas être utilisée avec les alertes de journal, car la validation d’accès des ressources de règle d’alerte, notamment les espaces de travail et les applications, s’effectue au moment de la création de l’alerte. L’ajout de nouvelles ressources à la fonction après la création de l’alerte n’est pas pris en charge. Si vous préférez utiliser la fonction pour l’étendue des ressources dans les alertes de journal, vous devez modifier la règle d’alerte dans le portail ou à l’aide d’un modèle Resource Manager pour mettre à jour les ressources délimitées. Vous pouvez également inclure la liste des ressources dans la requête d’alerte de journal.


![Graphique temporel](media/cross-workspace-query/chart.png)

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble des requêtes de journal et de la manière dont les données de journal d’Azure Monitor sont structurées, voir [Analyser les données de journal dans Azure Monitor](log-query-overview.md).

