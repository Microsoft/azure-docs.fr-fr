---
title: Analyse continue de votre pipeline de mise en production DevOps avec Azure Pipelines et Azure Application Insights | Microsoft Docs
description: Fournit des instructions permettant de configurer rapidement une analyse continue avec Application Insights
ms.topic: conceptual
ms.date: 05/01/2020
ms.openlocfilehash: fd7cd6a107ed45adb60167a57661b60be5dc8212
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86517125"
---
# <a name="add-continuous-monitoring-to-your-release-pipeline"></a>Ajouter l’analyse continue à votre pipeline de mise en production

Azure Pipelines s’intègre à Azure Application Insights pour permettre l’analyse continue de votre pipeline de mise en production DevOps tout au long du cycle de vie du développement logiciel. 

Avec l’analyse continue, les pipelines de mise en production peuvent incorporer des données d’analyse à partir de Application Insights et d’autres ressources Azure. Lorsque le pipeline de mise en production détecte une alerte Application Insights, le pipeline peut déterminer ou restaurer le déploiement jusqu’à ce que l’alerte soit résolue. Si toutes les vérifications réussissent, les déploiements passent automatiquement de la phase de test à la mise en production sans nécessiter d’intervention manuelle. 

## <a name="configure-continuous-monitoring"></a>Configurer l’analyse continue

1. Dans [Azure DevOps](https://dev.azure.com), sélectionnez une organisation et un projet.
   
1. Dans le menu de gauche de la page projet, sélectionnez **Pipelines** > **Mises en production**. 
   
1. Faites glisser la flèche à côté de **Nouveau** et sélectionnez **Nouveau pipeline de mise en production**. Ou, si vous n’avez pas encore de pipeline, sélectionnez **Nouveau pipeline** dans la page qui s’affiche.
   
1. Dans le volet **Sélectionner un modèle**, recherchez et sélectionnez **Déploiement avec analyse continue Azure App Service**, puis sélectionnez **Appliquer**. 

   ![Nouveau pipeline de mise en production Azure Pipelines](media/continuous-monitoring/001.png)

1. Dans la zone **Étape 1**, sélectionnez le lien hypertexte pour **Afficher les tâches d’étape.**

   ![Afficher les tâches d’index](media/continuous-monitoring/002.png)

1. Dans le volet configuration de l’**Étape 1**, complétez les champs suivants : 

    | Paramètre        | Valeur |
   | ------------- |:-----|
   | **Nom de l’étape**      | Fournissez un nom d’étape ou laissez-le en tant que **Étape 1**. |
   | **Abonnement Azure** | Faites défiler et sélectionnez l’abonnement Azure lié que vous souhaitez utiliser.|
   | **Type d’application** | Faites défiler et sélectionnez votre type d’application. |
   | **Nom de l’App Service** | Entrez le nom de votre Azure App Service. |
   | **Nom du groupe de ressources pour Application Insights**    | Faites défiler et sélectionnez le groupe de ressources à utiliser. |
   | **Nom de la ressource Application Insights** | Faites défiler et sélectionnez la ressource Application Insights pour le groupe de ressources que vous avez sélectionné.

1. Pour enregistrer le pipeline avec les paramètres de règle d’ alerte par défaut, sélectionnez **Enregistrer** en haut à droite dans la fenêtre Azure DevOps. Entrez un commentaire descriptif, puis sélectionnez **OK**.

## <a name="modify-alert-rules"></a>Modifier les règles d’alerte

Prêt à l’emploi, le modèle de **déploiement avec analyse continue Azure App Service** comporte quatre règles d’alerte : **Disponibilité**, **Requêtes ayant échoué**, **Temps de réponse du serveur** et **Exceptions du serveur**. Vous pouvez ajouter d’autres règles ou modifier les paramètres de règle en fonction de vos besoins de niveau de service. 

Pour modifier les paramètres de règle d’alerte :

Dans le volet gauche de la page pipeline de mise en production, sélectionnez **Configurer des alertes Application Insights**.

Les quatre règles d’alerte par défaut sont créées à l’aide d’un script inline :

```bash
$subscription = az account show --query "id";$subscription.Trim("`"");$resource="/subscriptions/$subscription/resourcegroups/"+"$(Parameters.AppInsightsResourceGroupName)"+"/providers/microsoft.insights/components/" + "$(Parameters.ApplicationInsightsResourceName)";
az monitor metrics alert create -n 'Availability_$(Release.DefinitionName)' -g $(Parameters.AppInsightsResourceGroupName) --scopes $resource --condition 'avg availabilityResults/availabilityPercentage < 99' --description "created from Azure DevOps";
az monitor metrics alert create -n 'FailedRequests_$(Release.DefinitionName)' -g $(Parameters.AppInsightsResourceGroupName) --scopes $resource --condition 'count requests/failed > 5' --description "created from Azure DevOps";
az monitor metrics alert create -n 'ServerResponseTime_$(Release.DefinitionName)' -g $(Parameters.AppInsightsResourceGroupName) --scopes $resource --condition 'avg requests/duration > 5' --description "created from Azure DevOps";
az monitor metrics alert create -n 'ServerExceptions_$(Release.DefinitionName)' -g $(Parameters.AppInsightsResourceGroupName) --scopes $resource --condition 'count exceptions/server > 5' --description "created from Azure DevOps";
```

Vous pouvez modifier le script et ajouter d’autres règles d’alerte, modifier les conditions d’alerte ou supprimer des règles d’alerte qui ne sont pas pertinentes par rapport à votre déploiement.

## <a name="add-deployment-conditions"></a>Ajouter des conditions de déploiement

Lorsque vous ajoutez des jalons de déploiement à votre pipeline de mise en production, une alerte de dépassement des seuils que vous définissez empêche la promotion des mises en production indésirables. Une fois que vous avez résolu l’alerte, le déploiement peut se poursuivre automatiquement.

Pour ajouter des jalons de déploiement :

1. Dans la page pipeline principal, sous **Étapes**, sélectionnez le symbole **Conditions de pré-déploiement** ou **Conditions de post-déploiement**, en fonction de l’étape nécessitant une jalons d’analyse continue.
   
   ![Conditions de pré-déploiement](media/continuous-monitoring/004.png)
   
1. Dans le volet configuration des **Conditions de pré-déploiement**, définissez **Jalons** à **Activé**.
   
1. À côté de **Jalons de déploiement**, sélectionnez **Ajouter**.
   
1. Dans le menu déroulant, sélectionnez **Interroger les alertes Azure Monitor**. Cette option vous permet d’accéder aux alertes Azure Monitor et Application Insights.
   
   ![Interroger les alertes Azure Monitor](media/continuous-monitoring/005.png)
   
1. Sous **Options d’évaluation**, entrez les valeurs souhaitées pour les paramètres tels que **Le délai entre la réévaluation des jalons** et **Le délai d’attente après lequel les jalons échouent**. 

## <a name="view-release-logs"></a>Afficher les journaux des mises en production

Vous pouvez voir le comportement du jalon de déploiement et d’autres étapes de mise en production dans les journaux des mises en production. Pour ouvrir les journaux :

1. Sélectionnez **Mises en production** dans le menu de gauche de la page Pipeline. 
   
1. Sélectionnez n’importe quelle mise en production. 
   
1. Sous **Étapes**, sélectionnez une étape pour afficher un résumé de la mise en production. 
   
1. Pour afficher les journaux, sélectionnez **Afficher les journaux** dans le résumé de la mise en production, sélectionnez le lien hypertexte ayant **Réussi** ou **Échoué** à n’importe quelle étape, ou passez la souris sur une étape et sélectionnez **Journaux**. 
   
   ![Afficher les journaux des mises en production](media/continuous-monitoring/006.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Pipelines, consultez la [documentation Azure Pipelines](/azure/devops/pipelines).
