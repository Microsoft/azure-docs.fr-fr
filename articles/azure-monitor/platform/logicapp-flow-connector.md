---
title: Utiliser Azure Monitor Logs avec Azure Logic Apps et Power Automate
description: Découvrez comment utiliser Azure Logic Apps et Power Automate pour automatiser rapidement les processus récurrents en utilisant le connecteur Azure Monitor.
ms.service: azure-monitor
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/13/2020
ms.openlocfilehash: d06501abe69ce9b06656cfa8949c42bb53a03983
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019036"
---
# <a name="azure-monitor-logs-connector-for-logic-apps-and-power-automate"></a>Connecteur Azure Monitor Logs pour Logic Apps et Power Automate
[Azure Logic Apps](../../logic-apps/index.yml) et [Power Automate](https://flow.microsoft.com) vous permettent de créer des workflows automatisés utilisant des centaines d’actions pour divers services. Le connecteur Azure Monitor Logs vous permet de générer des workflows qui récupèrent des données à partir d’un espace de travail Log Analytics ou d’une application Application Insights dans Azure Monitor. Cet article décrit les actions incluses dans le connecteur et montre pas à pas comment générer un workflow utilisant ces données.

Par exemple, vous pouvez créer une application logique pour utiliser les données de journal Azure Monitor contenues dans une notification par e-mail en provenance d’Office 365, créer un bogue dans Azure DevOps ou poster un message Slack.  Vous pouvez déclencher un flux de travail à l’aide d’un calendrier ou d’une action d’un service connecté, telle que la réception d’un e-mail ou d’un tweet. 

## <a name="connector-limits"></a>Limites des connecteurs
Le connecteur de journaux Azure Monitor a les limites suivantes :
* Taille maximale des données : 16 Mo
* Taille maximale de la réponse aux requêtes : 100 Mo
* Nombre maximal d’enregistrements : 500 000
* Délai d’expiration maximal des requêtes : 110 secondes.

En fonction de la taille de vos données et de la requête que vous utilisez, le connecteur peut atteindre ses limites et échouer. Vous pouvez contourner ce type de situation en ajustant la périodicité du déclencheur pour que le connecteur s’exécute plus fréquemment et interroge moins de données. Vous pouvez utiliser des requêtes qui agrègent vos données pour retourner moins d’enregistrements et de colonnes.

## <a name="actions"></a>Actions
Le tableau suivant décrit les actions incluses dans le connecteur Azure Monitor Logs. Les deux vous permettent d’exécuter une requête de journal sur un espace de travail Log Analytics ou une application Application Insights. La différence réside dans la façon dont les données sont retournées.

> [!NOTE]
> Le connecteur Azure Monitor Logs remplace le [connecteur Azure Log Analytics](/connectors/azureloganalytics/) et le [connecteur Azure Application Insights](/connectors/applicationinsights/). Ce connecteur propose les mêmes fonctionnalités que les autres et constitue la meilleure méthode pour exécuter une requête sur un espace de travail Log Analytics ou une application Application Insights.


| Action | Description |
|:---|:---|
| [Exécuter la requête et répertorier les résultats](/connectors/azuremonitorlogs/#run-query-and-list-results) | Retourne chaque ligne comme son propre objet. Utilisez cette action quand vous voulez utiliser chaque ligne séparément dans le reste du workflow. L’action est généralement suivie d’une [activité Foreach](../../logic-apps/logic-apps-control-flow-loops.md#foreach-loop). |
| [Exécuter une requête et visualiser les résultats](/connectors/azuremonitorlogs/#run-query-and-visualize-results) | Retourne toutes les lignes dans le jeu de résultats sous la forme d’un objet mis en forme unique. Utilisez cette action quand vous voulez utiliser l’ensemble du jeu de résultats dans le reste du workflow, par exemple pour envoyer les résultats dans un message.  |

## <a name="walkthroughs"></a>Procédures pas à pas
Les tutoriels suivants illustrent l’utilisation des connecteurs Azure Monitor dans Azure Logic Apps. Vous pouvez effectuer ces mêmes exemples avec Power Automate, la seule différence étant la façon dont le workflow initial est créé puis exécuté une fois celui-ci finalisé. La configuration du workflow et des actions est identique dans les deux cas. Pour commencer, consultez [Créer un flux à partir d’un modèle dans Power Automate](/power-automate/get-started-logic-template).


### <a name="create-a-logic-app"></a>Créer une application logique

Accédez à **Logic Apps** sur le portail Azure, puis cliquez sur **Ajouter**. Sélectionnez un **abonnement** , un **groupe de ressources** et une **région** où stocker la nouvelle application logique, puis attribuez-lui un nom unique. Vous pouvez activer le paramètre **Log Analytics** pour collecter des informations sur les événements et les données du runtime, comme décrit dans [Configurer les journaux Azure Monitor et collecter des données de diagnostic pour Azure Logic Apps](../../logic-apps/monitor-logic-apps-log-analytics.md). Ce paramètre n’est pas nécessaire pour utiliser le connecteur Azure Monitor Logs.

![Créer une application logique](media/logicapp-flow-connector/create-logic-app.png)


Cliquez sur **Vérifier + créer**, puis sur **Créer**. Une fois le déploiement terminé, cliquez sur **Accéder à la ressource** pour ouvrir le **concepteur Logic Apps**.

### <a name="create-a-trigger-for-the-logic-app"></a>Créer un déclencheur pour l’application logique
En dessous de **Démarrer avec un déclencheur courant**, sélectionnez **Récurrence**. Une application logique qui s’exécute automatiquement à intervalles réguliers est alors créée. Dans la zone **Fréquence** de l’action, sélectionnez **Jour** puis, dans la zone **Intervalle**, entrez **1** pour exécuter le workflow une fois par jour.

![Action de récurrence](media/logicapp-flow-connector/recurrence-action.png)

## <a name="walkthrough-mail-visualized-results"></a>Procédure pas à pas : Résultats visualisés par e-mail
Le tutoriel suivant vous montre comment créer une application logique qui envoie les résultats d’une requête de journal Azure Monitor par e-mail. 

### <a name="add-azure-monitor-logs-action"></a>Ajouter l’action Azure Monitor Logs
Cliquez sur **+ Nouvelle étape** pour ajouter une action qui s’exécute après l’action de récurrence. En dessous de **Choisir une action**, tapez **azure monitor**, puis sélectionnez **Azure Monitor Logs**.

![Action Azure Monitor Logs](media/logicapp-flow-connector/select-azure-monitor-connector.png)

Cliquez sur **Azure Log Analytics – Exécuter la requête et visualiser les résultats**.

![Capture d’écran de l’ajout d’une action à une étape dans le Concepteur d’application logique. Azure Monitor Logs est mis en surbrillance sous Choisir une action.](media/logicapp-flow-connector/select-query-action-visualize.png)


### <a name="add-azure-monitor-logs-action"></a>Ajouter l’action Azure Monitor Logs

Sélectionnez l’**abonnement** et le **groupe de ressources** pour votre espace de travail Log Analytics. Sélectionnez *Espace de travail Log Analytics* pour **Type de ressource**, puis sélectionnez le nom de l’espace de travail en dessous de **Nom de la ressource**.

Dans la zone **Requête**, ajoutez la requête de journal suivante.  

```Kusto
Event
| where EventLevelName == "Error" 
| where TimeGenerated > ago(1day)
| summarize TotalErrors=count() by Computer
| sort by Computer asc   
```

Sélectionnez *Définir dans la requête* pour **Période** et **Table HTML** pour **Type de graphique** .
   
![Capture d’écran des paramètres de la nouvelle action Azure Monitor Logs nommée Exécuter une requête et visualiser les résultats.](media/logicapp-flow-connector/run-query-visualize-action.png)

Le courrier est envoyé par le compte associé à la connexion active. Vous pouvez spécifier un autre compte en cliquant sur **Modifier la connexion**.

### <a name="add-email-action"></a>Action Ajouter un e-mail

Cliquez sur **+ Nouvelle étape**, puis sur **+ Ajouter une action**. En dessous de **Choisir une action**, tapez **outlook**, puis sélectionnez **Office 365 Outlook**.

![Sélectionner un connecteur Outlook](media/logicapp-flow-connector/select-outlook-connector.png)

Sélectionnez **Envoyer un e-mail (V2)** .

![Fenêtre de sélection d’Office 365 Outlook](media/logicapp-flow-connector/select-mail-action.png)

Cliquez n’importe où dans la zone **Corps** pour ouvrir une fenêtre **Contenu dynamique** contenant les valeurs des actions précédentes effectuées dans l’application logique. Sélectionnez **Afficher plus** puis **Corps**, qui correspond aux résultats de la requête de l’action Log Analytics.

![Sélectionner le corps](media/logicapp-flow-connector/select-body.png)

Spécifiez l’adresse e-mail d’un destinataire dans la fenêtre **À**, ainsi qu’un objet dans **Objet**. 

![Action de messagerie](media/logicapp-flow-connector/mail-action.png)


### <a name="save-and-test-your-logic-app"></a>Enregistrer et tester votre application logique
Cliquez sur **Enregistrer** puis sur **Exécuter** pour effectuer une série de tests sur l’application logique.

![Enregistrer et exécuter](media/logicapp-flow-connector/save-run.png)


Une fois que l’application logique a terminé, consultez l’e-mail du destinataire que vous avez spécifié.  Il doit avoir reçu un e-mail dont le corps est semblable à ceci :

![Exemple de message électronique](media/logicapp-flow-connector/sample-mail.png)



## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [requêtes de journal dans Azure Monitor](../log-query/log-query-overview.md).
- Découvrez plus en détail [Logic Apps](../../logic-apps/index.yml).
- En savoir plus sur [Power Automate](https://flow.microsoft.com).
