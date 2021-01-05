---
title: Utilisation de l’API REST d’alerte Log Analytics
description: L’API REST d’alerte Log Analytics vous permet de créer et de gérer des alertes dans Log Analytics.  Cet article fournit des détails sur l’API et plusieurs exemples pour effectuer différentes opérations.
ms.subservice: logs
ms.topic: conceptual
ms.date: 07/29/2018
ms.openlocfilehash: dce340db90c1528c46c1be0bc172751a04feaf31
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006402"
---
# <a name="create-and-manage-alert-rules-in-log-analytics-with-rest-api"></a>Créer et gérer des règles d’alerte dans Log Analytics avec l’API REST 

> [!IMPORTANT]
> Comme [annoncé](https://azure.microsoft.com/updates/switch-api-preference-log-alerts/), les espaces de travail Log Analytics créés après le *1er juin 2019* gèrent les règles d’alerte à l’aide de l’[API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules/) actuelle. Les clients sont encouragés à [basculer vers l’API actuelle](./alerts-log-api-switch.md) dans des espaces de travail plus anciens pour tirer parti des [avantages](./alerts-log-api-switch.md#benefits) scheduledQueryRules d’Azure Monitor. Cet article décrit la gestion des règles d’alerte à l’aide de l’API héritée.

L’API REST d’alerte Log Analytics vous permet de créer et de gérer des alertes dans Log Analytics.  Cet article fournit des détails sur l’API et plusieurs exemples pour effectuer différentes opérations.

L’API REST de recherche Log Analytics est un service RESTful qui est accessible par le biais de l’API REST Azure Resource Manager. Ce document présente des exemples montrant comment accéder à l’API à partir d’une ligne de commande PowerShell en utilisant [ARMClient](https://github.com/projectkudu/ARMClient), outil en ligne de commande open source qui simplifie l’appel de l’API Azure Resource Manager. L'utilisation d’ARMClient et de PowerShell est une des nombreuses options vous permettant d’accéder à l'API de recherche Log Analytics. Grâce à ces outils, vous pouvez utiliser l’API Azure Resource Manager RESTful pour effectuer des appels vers les espaces de travail Log Analytics et exécuter en leur sein des commandes de recherche. L'API produira pour vous des résultats de recherche au format JSON, qui vous permet d'utiliser ces résultats, par programme, de différentes manières.

## <a name="prerequisites"></a>Prérequis
Actuellement, les alertes peuvent être créées uniquement avec une recherche enregistrée dans Log Analytics.  Vous pouvez consulter l’ [API REST de recherche de journal](../log-query/log-query-overview.md) pour plus d’informations.

## <a name="schedules"></a>Planifications
Une recherche enregistrée peut avoir une ou plusieurs planifications. La planification définit la fréquence à laquelle la recherche est exécutée et l’intervalle de temps pendant lequel les critères sont identifiés.
Les propriétés des planifications sont décrites dans le tableau suivant.

| Propriété | Description |
|:--- |:--- |
| Intervalle |Fréquence à laquelle la recherche est exécutée. Exprimée en minutes. |
| QueryTimeSpan |Intervalle de temps pendant lequel les critères sont évalués. Doit être égal ou supérieur à Interval. Exprimée en minutes. |
| Version |Version de l’API utilisée.  Actuellement, cette propriété doit toujours être définie sur 1. |

Par exemple, considérez une requête d’événement avec Interval défini sur 15 minutes et QueryTimeSpan sur 30 minutes. Dans ce cas, la requête est exécutée toutes les 15 minutes et une alerte se déclenche si les critères sont satisfaits pendant un intervalle de 30 minutes.

### <a name="retrieving-schedules"></a>Récupération des planifications
Utilisez la méthode Get pour extraire toutes les planifications d’une recherche enregistrée.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules?api-version=2015-03-20
```

Utilisez la méthode Get avec un ID de planification pour extraire une planification particulière pour une recherche enregistrée.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20
```

Voici un exemple de réponse pour une planification.

```json
{
   "value": [{
      "id": "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/sampleRG/providers/Microsoft.OperationalInsights/workspaces/MyWorkspace/savedSearches/0f0f4853-17f8-4ed1-9a03-8e888b0d16ec/schedules/a17b53ef-bd70-4ca4-9ead-83b00f2024a8",
      "etag": "W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\"",
      "properties": {
         "Interval": 15,
         "QueryTimeSpan": 15,
         "Enabled": true,
      }
   }]
}
```

### <a name="creating-a-schedule"></a>Création d'une planification
Utilisez la méthode Put avec un ID de planification unique pour créer une planification.  Notez que deux planifications ne peuvent pas avoir le même ID, même si elles sont associées à d’autres recherches enregistrées.  Quand vous créez une planification dans la console Log Analytics, un GUID est créé pour l’ID de la planification.

> [!NOTE]
> Le nom de l’ensemble des recherches enregistrées, des planifications et des actions créées avec l’API Log Analytics doit être en minuscules.

```powershell
$scheduleJson = "{'properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Enabled':'true' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson
```

### <a name="editing-a-schedule"></a>Modification d’une planification
Utilisez la méthode Put avec un ID de planification existant pour la même recherche enregistrée afin de modifier cette planification. Dans l’exemple suivant, la planification est désactivée. Le corps de la demande doit inclure l’*ETag* de la planification.

```powershell
$scheduleJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A49.8074679Z'\""','properties': { 'Interval': 15, 'QueryTimeSpan':15, 'Enabled':'false' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/mynewschedule?api-version=2015-03-20 $scheduleJson
```

### <a name="deleting-schedules"></a>Suppression de planifications
Pour supprimer une planification, utilisez la méthode Delete avec un ID de planification.

```powershell
armclient delete /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}?api-version=2015-03-20
```

## <a name="actions"></a>Actions
Une planification peut avoir plusieurs actions. Une action peut définir un ou plusieurs processus à effectuer, tels que l’envoi d’un courrier électronique ou le démarrage d’un runbook, ou elle peut définir un seuil qui détermine si les résultats d’une recherche satisfont à certains critères.  Certaines actions définissent ces deux aspects afin que les processus soient exécutés quand le seuil est atteint.

Toutes les actions des planifications sont décrites dans le tableau suivant.  Différents types d’alertes ont différentes propriétés supplémentaires qui sont décrites ci-dessous.

| Propriété | Description |
|:--- |:--- |
| `Type` |Type de l’action.  Actuellement, les valeurs possibles sont Alert et Webhook. |
| `Name` |Nom d’affichage de l’alerte. |
| `Version` |Version de l’API utilisée.  Actuellement, cette propriété doit toujours être définie sur 1. |

### <a name="retrieving-actions"></a>Récupération des actions

Utilisez la méthode Get pour récupérer toutes les actions d’une planification.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search  ID}/schedules/{Schedule ID}/actions?api-version=2015-03-20
```

Utilisez la méthode Get avec l’ID d’action pour récupérer une action particulière d’une planification.

```powershell
armclient get /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/actions/{Action ID}?api-version=2015-03-20
```

### <a name="creating-or-editing-actions"></a>Création ou modification des actions
Utilisez la méthode Put avec un ID d’action propre à la planification pour créer une activité.  Quand vous créez une action dans la console Log Analytics, un GUID est défini pour l’ID de l’action.

> [!NOTE]
> Le nom de l’ensemble des recherches enregistrées, des planifications et des actions créées avec l’API Log Analytics doit être en minuscules.

Utilisez la méthode Put avec un ID d’action existant pour la même recherche enregistrée afin de modifier cette planification.  Le corps de la demande doit inclure l’ETag de la planification.

Le format de demande pour la création d’une action variant selon le type d’action, des exemples sont fournis dans les sections ci-dessous.

### <a name="deleting-actions"></a>Suppression des actions

Utilisez la méthode Delete avec l’ID d’action pour supprimer une action.

```powershell
armclient delete /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Subscription ID}/schedules/{Schedule ID}/Actions/{Action ID}?api-version=2015-03-20
```

### <a name="alert-actions"></a>Actions d’alerte
Une planification doit avoir une, et une seule, action d’alerte.  Les actions d’alerte ont une ou plusieurs des sections indiquées dans le tableau suivant.  Chacune est décrite plus en détail ci-dessous.

| Section | Description | Usage |
|:--- |:--- |:--- |
| Seuil |Critères d’exécution de l’action.| Obligatoire pour chaque alerte, avant ou après l’extension à Azure. |
| severity |Étiquette utilisée pour classer l’alerte quand elle est déclenchée.| Obligatoire pour chaque alerte, avant ou après l’extension à Azure. |
| Suppress |Option permettant d’arrêter les notifications d’alerte. | Section facultative pour chaque alerte, avant ou après l’extension à Azure. |
| Groupes d’actions |ID d’Azure ActionGroup, où les actions requises sont spécifiées. Par exemple, e-mails, SMS, appels vocaux, Webhooks, Runbooks Automation, connecteurs ITSM, etc.| Obligatoire une fois que les alertes sont étendues à Azure|
| Personnaliser les actions|Modifier la sortie standard pour les actions sélectionnées depuis ActionGroup| Facultatif pour chaque alerte, possibilité d’utilisation après l’extension des alertes à Azure. |

### <a name="thresholds"></a>Seuils
Une action d’alerte ne doit avoir qu’un seul seuil.  Quand les résultats d’une recherche enregistrée correspondent au seuil dans une action associée à cette recherche, tous les autres processus dans cette action sont exécutés.  En outre, une action peut ne contenir qu’un seuil et être ainsi utilisable avec d’autres types d’actions ne comportant pas de seuils.

Les propriétés des seuils sont décrites dans le tableau suivant.

| Propriété | Description |
|:--- |:--- |
| `Operator` |Opérateur de comparaison de seuil. <br> gt = supérieur à <br>  lt = inférieur à |
| `Value` |Valeur du seuil. |

Par exemple, considérez une requête d’événement avec Interval défini sur 15 minutes, QueryTimeSpan sur 30 minutes et Threshold sur une valeur supérieure à 10. Dans ce cas, la requête est exécutée toutes les 15 minutes et une alerte se déclenche si la requête renvoie 10 événements créés en l’espace de 30 minutes.

Voici un exemple de réponse pour une action comportant uniquement un seuil.  

```json
"etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "My threshold action",
   "Threshold": {
      "Operator": "gt",
      "Value": 10
   },
   "Version": 1
}
```

Utilisez la méthode Put avec un ID d’action unique pour créer une action de seuil pour une planification.  

```powershell
$thresholdJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson
```

Utilisez la méthode Put avec un ID d’action existant pour modifier une action de seuil pour une planification.  Le corps de la demande doit inclure l’ETag de l’action.

```powershell
$thresholdJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdJson
```

#### <a name="severity"></a>severity
Log Analytics vous permet de classer vos alertes en catégories, pour faciliter la gestion et le triage. La gravité d’alerte définie est : information, avertissement et critique. Elles sont mappées par rapport à l’échelle de gravité des alertes Azure de la façon suivante :

|Niveau de gravité Log Analytics  |Niveau de gravité des alertes Azure  |
|---------|---------|
|`critical` |Gravité 0|
|`warning` |Gravité 1|
|`informational` | Gravité 2|

Voici un exemple de réponse pour une action comportant uniquement un seuil et une gravité. 

```json
"etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "My threshold action",
   "Threshold": {
      "Operator": "gt",
      "Value": 10
   },
   "Severity": "critical",
   "Version": 1
}
```

Utilisez la méthode Put avec un ID d’action unique pour créer une action pour une planification avec une gravité.  

```powershell
$thresholdWithSevJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson
```

Utilisez la méthode Put avec un ID d’action existant pour modifier une action de gravité pour une planification.  Le corps de la demande doit inclure l’ETag de l’action.

```powershell
$thresholdWithSevJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/mythreshold?api-version=2015-03-20 $thresholdWithSevJson
```

#### <a name="suppress"></a>Suppress
Les alertes de requête basées sur Log Analytics sont déclenchées à chaque fois que le seuil est atteint ou dépassé. En fonction de la logique impliquée dans la requête, il peut s’ensuivre le déclenchement d’alertes pour une série d’intervalles et donc l’envoi de notifications en permanence. Pour empêcher un tel scénario, un utilisateur peut définir l’option Suppress et indiquer à Log Analytics d’attendre un délai spécifié avant que la notification ne soit déclenchée la seconde fois pour la règle d’alerte. Par conséquent, si l’option Suppress est définie sur 30 minutes, l’alerte est déclenchée la première fois et envoie les notifications configurées. Attendez 30 minutes avant que la notification de la règle d’alerte ne soit de nouveau utilisée. Pendant la période intérimaire, une règle d’alerte continue de s’exécuter : seule la notification est supprimée par Log Analytics pendant la période spécifiée, et ce, quel que soit le nombre de déclenchements de la règle d’alerte pendant cette période.

La propriété Suppress de la règle d’alerte Log Analytics est spécifiée à l’aide de la valeur *Throttling* et la durée de suppression l’est à l’aide de la valeur *DurationInMinutes*.

Voici un exemple de réponse pour une action comportant un seuil, une gravité et une propriété Suppress :

```json
"etag": "W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "My threshold action",
   "Threshold": {
      "Operator": "gt",
      "Value": 10
   },
   "Throttling": {
   "DurationInMinutes": 30
   },
   "Severity": "critical",
   "Version": 1
}
```

Utilisez la méthode Put avec un ID d’action unique pour créer une action pour une planification avec une gravité.  

```powershell
$AlertSuppressJson = "{'properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Throttling': { 'DurationInMinutes': 30 },'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myalert?api-version=2015-03-20 $AlertSuppressJson
```

Utilisez la méthode Put avec un ID d’action existant pour modifier une action de gravité pour une planification.  Le corps de la demande doit inclure l’ETag de l’action.

```powershell
$AlertSuppressJson = "{'etag': 'W/\"datetime'2016-02-25T20%3A54%3A20.1302566Z'\"','properties': { 'Name': 'My Threshold', 'Version':'1','Severity': 'critical', 'Type':'Alert', 'Throttling': { 'DurationInMinutes': 30 },'Threshold': { 'Operator': 'gt', 'Value': 10 } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{ResourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myalert?api-version=2015-03-20 $AlertSuppressJson
```

#### <a name="action-groups"></a>Groupes d’actions
Toutes les alertes dans Azure, utilisez le groupe d’actions en tant que mécanisme par défaut pour la gestion des actions. Avec le groupe d’actions, vous pouvez spécifier vos actions une seule fois, puis associer le groupe d’actions à plusieurs alertes sur Azure. Vous n’avez plus besoin de déclarer sans cesse les mêmes actions. Les groupes d’actions prennent en charge plusieurs actions, y compris, les e-mails, les SMS, les appels vocaux, les connexions ITSM, les Runbooks Automation, les URI Webhook et bien plus encore. 

Pour les utilisateurs qui ont étendu leurs alertes dans Azure, une planification doit maintenant avoir des informations sur le groupe d’actions transmises avec le seuil, afin de pouvoir créer une alerte. Les informations sur les e-mails, les URLS Webhook, les Runbooks Automation et sur d’autres actions doivent être définies dans un groupe d’actions avant de créer une alerte ; vous pouvez créer un [groupe d’actions à partir d’Azure Monitor](./action-groups.md) dans le portail ou utiliser [l’API Groupes d’action](/rest/api/monitor/actiongroups).

Pour ajouter l’association du groupe d’actions à une alerte, spécifiez l’ID Azure Resource Manager du groupe d’actions dans la définition de l’alerte. Un exemple est donné ci-dessous :

```json
"etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "test-alert",
   "Description": "I need to put a description here",
   "Threshold": {
      "Operator": "gt",
      "Value": 12
   },
   "AzNsNotification": {
      "GroupIds": [
         "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
      ]
   },
   "Severity": "critical",
   "Version": 1
}
```

Pour associer le groupe d’actions déjà existant d’une planification, utilisez la méthode Put avec un ID d’action unique.  Vous trouverez ci-dessous un exemple d’utilisation.

```powershell
$AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']} } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

Utilisez la méthode Put avec un ID d’action existant pour modifier un groupe d’actions associé à une planification.  Le corps de la demande doit inclure l’ETag de l’action.

```powershell
$AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', 'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': { 'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'] } } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

#### <a name="customize-actions"></a>Personnaliser les actions
Par défaut, les actions suivent le modèle et le format standard des notifications. Mais l’utilisateur peut personnaliser certaines actions, même si elles sont contrôlées par les groupes d’actions. Actuellement, la personnalisation est possible pour les objets d’e-mail et les charges utiles de Webhook.

##### <a name="customize-e-mail-subject-for-action-group"></a>Personnaliser l’objet d’e-mail du groupe d’actions
Par défaut, l’objet de l’e-mail pour les alertes est : Notification d’alerte `<AlertName>` pour `<WorkspaceName>`. Mais cela peut être personnalisé, afin que vous puissiez ajouter des mots ou mots-clés spécifiques, utiles pour les règles de filtrage dans votre boîte de réception. Les informations sur l’en-tête d’e-mail personnalisé doivent être envoyées, ainsi que les informations sur ActionGroup, comme dans les exemples ci-dessous.

```json
"etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "test-alert",
   "Description": "I need to put a description here",
   "Threshold": {
      "Operator": "gt",
      "Value": 12
   },
   "AzNsNotification": {
      "GroupIds": [
         "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
      ],
      "CustomEmailSubject": "Azure Alert fired"
   },
   "Severity": "critical",
   "Version": 1
}
```

Pour associer le groupe d’actions déjà existant à une personnalisation d’une planification, utilisez la méthode Put avec un ID d’action unique.  Vous trouverez ci-dessous un exemple d’utilisation.

```powershell
$AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired'} } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

Utilisez la méthode Put avec un ID d’action existant pour modifier un groupe d’actions associé à une planification.  Le corps de la demande doit inclure l’ETag de l’action.

```powershell
$AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', 'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

##### <a name="customize-webhook-payload-for-action-group"></a>Personnaliser la charge utile du Webhook pour le groupe d’actions
Par défaut, le Webhook envoyé via le groupe d’actions pour l’analytique des journaux d’activité a une structure fixe. Néanmoins, vous pouvez personnaliser la charge utile JSON à l’aide de variables spécifiques prises en charge, pour répondre aux exigences du point de terminaison du Webhook. Pour plus d’informations, consultez [Actions webhook pour les règles d’alerte de journal](./alerts-log-webhook.md). 

Les informations sur le Webhook personnalisé doivent être envoyées avec les informations sur ActionGroup ; elles seront appliquées à tous les URI Webhook spécifiés dans le groupe d’actions, comme dans l’exemple ci-dessous.

```json
"etag": "W/\"datetime'2017-12-13T10%3A52%3A21.1697364Z'\"",
"properties": {
   "Type": "Alert",
   "Name": "test-alert",
   "Description": "I need to put a description here",
   "Threshold": {
      "Operator": "gt",
      "Value": 12
   },
   "AzNsNotification": {
      "GroupIds": [
         "/subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup"
      ],
   "CustomWebhookPayload": "{\"field1\":\"value1\",\"field2\":\"value2\"}",
   "CustomEmailSubject": "Azure Alert fired"
   },
   "Severity": "critical",
   "Version": 1
},
```

Pour associer le groupe d’actions déjà existant à une personnalisation d’une planification, utilisez la méthode Put avec un ID d’action unique.  Vous trouverez ci-dessous un exemple d’utilisation.

```powershell
$AzNsJson = "{'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup'], 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}'} } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

Utilisez la méthode Put avec un ID d’action existant pour modifier un groupe d’actions associé à une planification.  Le corps de la demande doit inclure l’ETag de l’action.

```powershell
$AzNsJson = "{'etag': 'datetime'2017-12-13T10%3A52%3A21.1697364Z'\"', 'properties': { 'Name': 'test-alert', 'Version':'1', 'Type':'Alert', 'Threshold': { 'Operator': 'gt', 'Value': 12 },'Severity': 'critical', 'AzNsNotification': {'GroupIds': ['subscriptions/1234a45-123d-4321-12aa-123b12a5678/resourcegroups/my-resource-group/providers/microsoft.insights/actiongroups/test-actiongroup']}, 'CustomEmailSubject': 'Azure Alert fired','CustomWebhookPayload': '{\"field1\":\"value1\",\"field2\":\"value2\"}' } }"
armclient put /subscriptions/{Subscription ID}/resourceGroups/{Resource Group Name}/Microsoft.OperationalInsights/workspaces/{Workspace Name}/savedSearches/{Search ID}/schedules/{Schedule ID}/actions/myAzNsaction?api-version=2015-03-20 $AzNsJson
```

## <a name="next-steps"></a>Étapes suivantes

* Utilisez l’ [API REST pour effectuer des recherches de journaux](../log-query/log-query-overview.md) dans Log Analytics.
* En savoir plus sur les [alertes de journal dans Azure Monitor](./alerts-unified-log.md)
* Comment [créer, modifier ou gérer des règles d’alerte de journal dans Azure Monitor](./alerts-log.md)

