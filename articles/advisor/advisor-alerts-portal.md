---
title: Créer des alertes Azure Advisor pour les nouvelles suggestions à l’aide du Portail Azure
description: Créer des alertes Azure Advisor pour les nouvelles recommandations
ms.topic: article
ms.date: 09/09/2019
ms.openlocfilehash: 837f12a12e532902ee76e345afa430f758f79c10
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90973662"
---
# <a name="create-azure-advisor-alerts-on-new-recommendations-using-the-azure-portal"></a>Créer des alertes Azure Advisor pour les nouvelles suggestions à l’aide du Portail Azure 

Cet article explique comment configurer une alerte pour les nouvelles suggestions à partir d’Azure Advisor à l’aide du Portail Azure. 

Chaque fois qu’Azure Advisor détecte une nouvelle recommandation pour l’une de vos ressources, un événement est stocké dans le [journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md). Vous pouvez configurer des alertes pour ces événements à partir d’Azure Advisor en utilisant une expérience de création d’alertes spécifique aux recommandations. Vous pouvez sélectionner un abonnement et éventuellement un groupe de ressources afin de spécifier les ressources pour lesquelles vous souhaitez recevoir des alertes. 

Vous pouvez également déterminer les types de recommandations à l’aide des propriétés suivantes :

* Category
* Niveau d’impact
* Type de recommandation

Vous pouvez également configurer l’action à entreprendre lorsqu’une alerte est déclenchée par :  

* La sélection d’un groupe d’actions existant
* La création d’un nouveau groupe d’actions

Pour en savoir plus sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions](../azure-monitor/platform/action-groups.md).

> [!NOTE] 
> Les alertes Advisor sont actuellement disponibles uniquement pour les recommandations relatives à la haute disponibilité, aux performances et aux coûts. Les recommandations de sécurité ne sont pas prises en charge. 

## <a name="create-alert-rule"></a>Créer une règle d’alerte
1. Dans le **portail**, sélectionnez **Azure Advisor**.

    ![Azure Advisor dans le portail](./media/advisor-alerts/create1.png)

2. Dans la section **Surveillance** du menu de gauche, sélectionnez **Alertes**. 

    ![Alertes dans Advisor](./media/advisor-alerts/create2.png)

3. Sélectionnez **Nouvelle alerte Advisor**.

    ![Nouvelle alerte Advisor](./media/advisor-alerts/create3.png)

4. Dans la section **Étendue**, sélectionnez l’abonnement et éventuellement le groupe de ressources au sujet desquels vous souhaitez être alerté. 

    ![Étendue d’alerte Advisor](./media/advisor-alerts/create4.png)

5. Dans la section **Condition**, sélectionnez la méthode que vous souhaitez utiliser pour configurer votre alerte. Si vous souhaitez être alerté pour toutes les recommandations d’une certaine catégorie et/ou d’un certain niveau d’impact, sélectionnez **Catégorie et niveau d’impact**. Si vous souhaitez être alerté pour toutes les recommandations d’un certain type, sélectionnez **Type de recommandation**.

    ![Condition d’alerte Azure Advisor](./media/advisor-alerts/create5.png)

6. Selon l’option Configuré par que vous sélectionnez, vous serez en mesure de spécifier les critères. Si vous souhaitez toutes les recommandations, laissez les champs restants vides. 

    ![Groupe d’actions d’alerte Advisor](./media/advisor-alerts/create6.png)

7. Dans la section relative aux **groupes d’actions**, sélectionnez **Ajouter existant** pour utiliser un groupe d’actions que vous avez déjà créé ou sélectionnez **Créer nouveau** pour configurer un nouveau [groupe d’actions](../azure-monitor/platform/action-groups.md). 

    ![Alerte Advisor - Ajouter existant](./media/advisor-alerts/create7.png)

8. Dans la section Détails de l’alerte, donnez un nom à votre alerte et spécifiez une brève description. Si vous souhaitez que votre alerte soit activée, laissez l’option **Activer la règle lors de sa création** définie sur **Oui**. Sélectionnez ensuite le groupe de ressources dans lequel vous souhaitez enregistrer votre alerte. Cela n’aura pas d’impact sur l’étendue de ciblage de la recommandation. 

    :::image type="content" source="./media/advisor-alerts/create8.png" alt-text="Capture d’écran de la section Détails de l’alerte.":::


## <a name="configure-recommendation-alerts-to-use-a-webhook"></a>Configurer les alertes de recommandation pour utiliser un webhook
Cette section montre comment configurer les alertes Azure Advisor pour envoyer des données de recommandation via des webhooks à vos systèmes existants. 

Vous pouvez configurer des alertes pour être averti lorsque vous avez une nouvelle recommandation Advisor sur l’une de vos ressources. Ces alertes peuvent vous avertir par e-mail ou SMS, mais elles peuvent également s’intégrer à vos systèmes existants via un webhook. 


### <a name="using-the-advisor-recommendation-alert-payload"></a>Utilisation de la charge utile d’alerte de recommandation Advisor
Si vous souhaitez intégrer des alertes Advisor à vos propres systèmes à l’aide d’un webhook, vous devez analyser la charge utile JSON qui est envoyée à partir de la notification. 

Quand vous configurez votre groupe d’actions pour cette alerte, vous sélectionnez si vous souhaitez utiliser le schéma d’alerte commun. Si vous sélectionnez le schéma d’alerte commun, votre charge utile ressemblera à : 

```json
{  
   "schemaId":"azureMonitorCommonAlertSchema",
   "data":{  
      "essentials":{  
         "alertId":"/subscriptions/<subid>/providers/Microsoft.AlertsManagement/alerts/<alerted>",
         "alertRule":"Webhhook-test",
         "severity":"Sev4",
         "signalType":"Activity Log",
         "monitorCondition":"Fired",
         "monitoringService":"Activity Log - Recommendation",
         "alertTargetIDs":[  
            "/subscriptions/<subid>/resourcegroups/<resource group name>/providers/microsoft.dbformariadb/servers/<resource name>"
         ],
         "originAlertId":"001d8b40-5d41-4310-afd7-d65c9d4428ed",
         "firedDateTime":"2019-07-17T23:00:57.3858656Z",
         "description":"A new recommendation is available.",
         "essentialsVersion":"1.0",
         "alertContextVersion":"1.0"
      },
      "alertContext":{  
         "channels":"Operation",
         "claims":"{\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress\":\"Microsoft.Advisor\"}",
         "caller":"Microsoft.Advisor",
         "correlationId":"8554b847-2a72-48ef-9776-600aca3c3aab",
         "eventSource":"Recommendation",
         "eventTimestamp":"2019-07-17T22:28:54.1566942+00:00",
         "httpRequest":"{\"clientIpAddress\":\"0.0.0.0\"}",
         "eventDataId":"001d8b40-5d41-4310-afd7-d65c9d4428ed",
         "level":"Informational",
         "operationName":"Microsoft.Advisor/recommendations/available/action",
         "properties":{  
            "recommendationSchemaVersion":"1.0",
            "recommendationCategory":"Performance",
            "recommendationImpact":"Medium",
            "recommendationName":"Increase the MariaDB server vCores",
            "recommendationResourceLink":"https://portal.azure.com/#blade/Microsoft_Azure_Expert/RecommendationListBlade/source/ActivityLog/recommendationTypeId/a5f888e3-8cf4-4491-b2ba-b120e14eb7ce/resourceId/%2Fsubscriptions%<subscription id>%2FresourceGroups%2<resource group name>%2Fproviders%2FMicrosoft.DBforMariaDB%2Fservers%2F<resource name>",
            "recommendationType":"a5f888e3-8cf4-4491-b2ba-b120e14eb7ce"
         },
         "status":"Active",
         "subStatus":"",
         "submissionTimestamp":"2019-07-17T22:28:54.1566942+00:00"
      }
   }
}
  ```

Si vous n’utilisez pas le schéma commun, votre charge utile ressemblera à : 

```json
{  
   "schemaId":"Microsoft.Insights/activityLogs",
   "data":{  
      "status":"Activated",
      "context":{  
         "activityLog":{  
            "channels":"Operation",
            "claims":"{\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress\":\"Microsoft.Advisor\"}",
            "caller":"Microsoft.Advisor",
            "correlationId":"3ea7320f-c002-4062-adb8-96d3bd92a5f4",
            "description":"A new recommendation is available.",
            "eventSource":"Recommendation",
            "eventTimestamp":"2019-07-17T20:36:39.3966926+00:00",
            "httpRequest":"{\"clientIpAddress\":\"0.0.0.0\"}",
            "eventDataId":"a12b8e59-0b1d-4003-bfdc-3d8152922e59",
            "level":"Informational",
            "operationName":"Microsoft.Advisor/recommendations/available/action",
            "properties":{  
               "recommendationSchemaVersion":"1.0",
               "recommendationCategory":"Performance",
               "recommendationImpact":"Medium",
               "recommendationName":"Increase the MariaDB server vCores",
               "recommendationResourceLink":"https://portal.azure.com/#blade/Microsoft_Azure_Expert/RecommendationListBlade/source/ActivityLog/recommendationTypeId/a5f888e3-8cf4-4491-b2ba-b120e14eb7ce/resourceId/%2Fsubscriptions%2F<subscription id>%2FresourceGroups%2F<resource group name>%2Fproviders%2FMicrosoft.DBforMariaDB%2Fservers%2F<resource name>",
               "recommendationType":"a5f888e3-8cf4-4491-b2ba-b120e14eb7ce"
            },
            "resourceId":"/subscriptions/<subscription id>/resourcegroups/<resource group name>/providers/microsoft.dbformariadb/servers/<resource name>",
            "resourceGroupName":"<resource group name>",
            "resourceProviderName":"MICROSOFT.DBFORMARIADB",
            "status":"Active",
            "subStatus":"",
            "subscriptionId":"<subscription id>",
            "submissionTimestamp":"2019-07-17T20:36:39.3966926+00:00",
            "resourceType":"MICROSOFT.DBFORMARIADB/SERVERS"
         }
      },
      "properties":{  
 
      }
   }
}
```

Dans l’un ou l’autre schéma, vous pouvez identifier les événements de recommandation Advisor en recherchant un **eventSource** égal à `Recommendation` et un **operationName** égal à `Microsoft.Advisor/recommendations/available/action`.

Voici quelques-uns des autres champs importants que vous pouvez utiliser : 

* *alertTargetIDs* (dans le schéma commun) ou *resourceId* (schéma hérité)
* *recommendationType*
* *recommendationName*
* *recommendationCategory*
* *recommendationImpact*
* *recommendationResourceLink*


## <a name="manage-your-alerts"></a>Gérez vos alertes 

À partir d’Azure Advisor, vous pouvez modifier, supprimer, désactiver ou activer vos alertes de recommandation. 

1. Dans le **portail**, sélectionnez **Azure Advisor**.

    :::image type="content" source="./media/advisor-alerts/create1.png" alt-text="Capture d’écran de la section Détails de l’alerte.":::

2. Dans la section **Surveillance** du menu de gauche, sélectionnez **Alertes**.

    :::image type="content" source="./media/advisor-alerts/create2.png" alt-text="Capture d’écran de la section Détails de l’alerte.":::

3. Pour modifier une alerte, cliquez sur le nom de l’alerte pour ouvrir l’alerte, puis modifiez les champs de votre choix.

4. Pour supprimer, activer ou désactiver une alerte, cliquez sur les points de suspension à la fin de la ligne, puis sélectionnez l’action que vous souhaitez entreprendre.
 

## <a name="next-steps"></a>Étapes suivantes
- Obtenir une [vue d’ensemble des alertes du journal d’activité](../azure-monitor/platform/alerts-overview.md) et découvrir comment recevoir des alertes.
- En savoir plus sur les [groupes d’actions](../azure-monitor/platform/action-groups.md).
