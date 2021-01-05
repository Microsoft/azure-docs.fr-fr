---
title: 'Tutoriel : Superviser les API publiées dans la Gestion des API Azure | Microsoft Docs'
description: Suivez les étapes de ce tutoriel pour apprendre à utiliser les métriques, les alertes, les journaux d’activité et les journaux de ressources afin de superviser vos API dans la Gestion des API Azure.
services: api-management
author: vladvino
ms.service: api-management
ms.custom: mvc
ms.topic: tutorial
ms.date: 10/14/2020
ms.author: apimpm
ms.openlocfilehash: 5e5c59d611cb7f4b5333b9919488e6fc083611cd
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779241"
---
# <a name="tutorial-monitor-published-apis"></a>Tutoriel : Surveiller les API publiées

Avec Azure Monitor, vous pouvez visualiser, interroger, router, archiver et exploiter les métriques ou les journaux provenant de votre service Gestion des API Azure.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Afficher les métriques de votre API 
> * Configurer une règle d’alerte 
> * Afficher les journaux d’activité
> * Activer et afficher les journaux de ressources

Vous pouvez également utiliser l’[analytique](howto-use-analytics.md) intégré de la Gestion des API pour superviser l’utilisation et les performances de vos API.

## <a name="prerequisites"></a>Prérequis

+ Apprenez la [terminologie relative à Gestion des API Azure](api-management-terminology.md).
+ Suivez ce guide de démarrage rapide : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).
+ Effectuez également toutes les étapes du tutoriel suivant : [Importer et publier votre première API](import-and-publish.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="view-metrics-of-your-apis"></a>Afficher les métriques de vos API

Le service Gestion des API émet des [métriques](../azure-monitor/platform/data-platform-metrics.md) chaque minute, pour une visibilité en quasi temps réel de l’intégrité de vos API. Vous trouverez ci-après les deux métriques les plus fréquemment utilisées. Pour obtenir la liste de toutes les métriques disponibles, consultez [Métriques prises en charge](../azure-monitor/platform/metrics-supported.md#microsoftapimanagementservice).

* **Capacité** : vous aide à prendre des décisions concernant la mise à niveau/le passage à une version antérieure de vos services APIM. La métrique est émise chaque minute et reflète la capacité de la passerelle au moment de la création de rapports. Le calcul de cette métrique, dont la valeur est comprise entre 0 et 100, repose sur les ressources de la passerelle, telles que l’utilisation du processeur et de la mémoire.
* **Demandes** : vous aide à analyser le trafic d’API transitant par vos services Gestion des API. La métrique est émise par minute, et indique le nombre de demandes de passerelle avec des dimensions, y compris les codes de réponse, l’emplacement, le nom d’hôte et les erreurs. 

> [!IMPORTANT]
> Les métriques suivantes sont dépréciées à compter du mois de mai 2019, et seront supprimées en août 2023 : Nombre total de demandes de la passerelle, Demandes de la passerelle ayant abouti, Demandes de la passerelle non autorisées, Demandes de la passerelle ayant échoué, Autres demandes de la passerelle. Effectuez une migration vers la métrique Demandes, qui fournit des fonctionnalités équivalentes.

:::image type="content" source="media/api-management-howto-use-azure-monitor/apim-monitor-metrics.png" alt-text="Capture d’écran des métriques dans la vue d’ensemble de la Gestion des API":::

Pour accéder aux métriques :

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre instance APIM. Dans la page **Vue d’ensemble**, passez en revue les principales métriques de vos API.
1. Pour examiner en détail les métriques, sélectionnez **Métriques** dans le menu vers le bas de la page.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-metrics-blade.png" alt-text="Capture d’écran de l’élément Métriques dans le menu Supervision":::

1. Dans la liste déroulante, sélectionner les métriques qui vous intéressent. Par exemple, **Demandes**. 
1. Le graphique affiche le nombre total d’appels d’API.
1. Le graphique peut être filtré à l’aide des dimensions de la métrique **Requêtes**. Par exemple, sélectionnez **Ajouter un filtre**, sélectionnez **Catégorie de codes de réponse de back-end**, puis entrez 500 comme valeur. À présent, le graphique montre le nombre de requêtes qui ont échoué dans le back-end d’API.   

## <a name="set-up-an-alert-rule"></a>Configurer une règle d’alerte 

Vous pouvez recevoir des [alertes](../azure-monitor/platform/alerts-metric-overview.md) en fonction des métriques et des journaux d’activité. Azure Monitor vous permet de [configurer une alerte](../azure-monitor/platform/alerts-metric.md) pour effectuer les opérations suivantes lors de son déclenchement :

* Envoyer un e-mail de notification
* Appeler un webhook
* Appeler une application logique Azure

Pour configurer un exemple de règle d’alerte basée sur une métrique de demande :

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre instance APIM.
1. Sélectionnez **Alertes** dans la barre de menus vers le bas de la page.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/alert-menu-item.png" alt-text="Capture d’écran de l’option Alertes dans le menu Supervision":::

1. Sélectionnez **+ Nouvelle règle d’alerte**.
1. Dans la fenêtre **Créer une règle d’alerte**, cliquez sur **Sélectionner une condition**.
1. Dans la fenêtre **Configurer la logique du signal** :
    1. Dans **Type de signal**, sélectionnez **Métriques**.
    1. Dans **Nom du signal**, sélectionnez **Demandes**.
    1. Dans **Diviser par dimensions**, dans **Nom de la dimension**, sélectionnez **Catégorie de codes de réponse de passerelle**.
    1. Dans **Valeurs de dimension**, sélectionnez **4xx** pour les erreurs de client telles que les demandes non autorisées ou non valides.
    1. Dans **Logique d’alerte**, spécifiez le seuil au-delà duquel l’alerte doit être déclenchée, puis sélectionnez **Terminé**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/threshold.png" alt-text="Capture d’écran de la fenêtre Configurer la logique du signal":::

1. Sélectionnez un groupe d’actions existant ou créez-en un. Dans l’exemple suivant, un groupe d’actions est créé. Un e-mail de notification est envoyé à admin@contoso.com. 

    :::image type="content" source="media/api-management-howto-use-azure-monitor/action-details.png" alt-text="Capture d’écran des notifications pour le nouveau groupe d’actions":::

1. Entrez un nom et une description de la règle d’alerte, puis sélectionnez le niveau de gravité. 
1. Sélectionnez **Créer une règle d’alerte**.
1. À présent, testez la règle d’alerte en appelant l’API de conférence sans clé API. Par exemple :

    ```bash
    curl GET https://apim-hello-world.azure-api.net/conference/speakers HTTP/1.1 
    ```

    Une alerte est déclenchée en fonction de la période d’évaluation et un e-mail est envoyé à admin@contoso.com. 

    Les alertes s’affichent également dans la page **Alertes** pour l’instance de Gestion des API.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/portal-alerts.png" alt-text="Capture d’écran des alertes dans le portail":::

## <a name="activity-logs"></a>Journaux d’activité

Les journaux d’activité fournissent des informations sur les opérations qui ont été effectuées sur les services de Gestion des API. Avec les journaux d’activité, vous pouvez déterminer « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) sur vos services de Gestion des API.

> [!NOTE]
> Les journaux d’activité n’incluent pas les opérations de lecture (GET) ou les opérations effectuées dans le portail Azure ou utilisant les API de gestion d’origine.

Vous pouvez accéder aux journaux d’activité dans votre service de Gestion des API, ou accéder aux journaux d’activité de toutes vos ressources Azure dans Azure Monitor. 

:::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-activity-logs.png" alt-text="Capture d’écran du journal d’activité dans le portail":::

Pour afficher le journal d’activité :

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre instance APIM.

1. Sélectionnez **Journal d’activité**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-activity-logs-blade.png" alt-text="Capture d’écran de l’élément Journal d’activité dans le menu Supervision":::
1. Sélectionnez l’étendue de filtrage souhaitée, puis **Appliquer**.

## <a name="resource-logs"></a>Journaux d’activité de ressources

Les journaux de ressources offrent des informations détaillées sur les opérations et erreurs qui sont importantes pour l’audit et dans le cadre de dépannages. Les journaux de ressources diffèrent des journaux d’activité. Le journal d’activité fournit des informations sur les opérations qui ont été effectuées sur vos ressources Azure. Les journaux de ressources fournissent des informations détaillées sur les opérations effectuées par votre ressource.

Pour configurer les journaux de ressources :

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre instance APIM.
2. Sélectionnez **Paramètres de diagnostic**.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/api-management-diagnostic-logs-blade.png" alt-text="Capture d’écran de l’élément Paramètres de diagnostic dans le menu Supervision":::

1. Sélectionnez **+ Ajouter le paramètre de diagnostic**.
1. Sélectionnez les journaux ou métriques que vous souhaitez collecter.

   Vous pouvez archiver les journaux de ressources avec les métriques dans un compte de stockage, les envoyer en streaming à un hub d’événement ou les envoyer à un espace de travail Log Analytics. 

Pour plus d’informations, consultez [Créer des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations](../azure-monitor/platform/diagnostic-settings.md).

## <a name="view-diagnostic-data-in-azure-monitor"></a>Afficher les données de diagnostic dans Azure Monitor

Si vous activez la collecte du journal GatewayLogs ou des métriques dans un espace de travail Log Analytics, quelques minutes peuvent être nécessaires pour que les données s’affichent dans Azure Monitor. Pour visualiser les données :

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre instance APIM.
1. Sélectionnez **Journaux** dans le menu vers le bas de la page.

    :::image type="content" source="media/api-management-howto-use-azure-monitor/logs-menu-item.png" alt-text="Capture d’écran de l’élément Journaux dans le menu Supervision":::

Exécutez des requêtes pour afficher les données. Plusieurs [exemples de requêtes](../azure-monitor/log-query/example-queries.md) sont fournis, ou exécutez les vôtres. Par exemple, la requête suivante récupère les 24 heures de données les plus récentes à partir de la table GatewayLogs :

```kusto
ApiManagementGatewayLogs
| where TimeGenerated > ago(1d) 
```

Pour plus d’informations sur l’utilisation des journaux de ressources pour Gestion des API, consultez :

* [Bien démarrer avec Azure Monitor Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md) ou essayez l’[environnement de démonstration Log Analytics](https://portal.loganalytics.io/demo).

* [Vue d’ensemble des requêtes de journal dans Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

Le code JSON suivant montre un exemple d’entrée dans GatewayLogs pour une demande d’API réussie. Pour plus d’informations, consultez les [informations de référence sur le schéma](gateway-log-schema-reference.md). 

```json
{
    "Level": 4,
    "isRequestSuccess": true,
    "time": "2020-10-14T17:xx:xx.xx",
    "operationName": "Microsoft.ApiManagement/GatewayLogs",
    "category": "GatewayLogs",
    "durationMs": 152,
    "callerIpAddress": "xx.xx.xxx.xx",
    "correlationId": "3f06647e-xxxx-xxxx-xxxx-530eb9f15261",
    "location": "East US",
    "properties": {
        "method": "GET",
        "url": "https://apim-hello-world.azure-api.net/conference/speakers",
        "backendResponseCode": 200,
        "responseCode": 200,
        "responseSize": 41583,
        "cache": "none",
        "backendTime": 87,
        "requestSize": 526,
        "apiId": "demo-conference-api",
        "operationId": "GetSpeakers",
        "apimSubscriptionId": "master",
        "clientTime": 65,
        "clientProtocol": "HTTP/1.1",
        "backendProtocol": "HTTP/1.1",
        "apiRevision": "1",
        "clientTlsVersion": "1.2",
        "backendMethod": "GET",
        "backendUrl": "https://conferenceapi.azurewebsites.net/speakers"
    },
    "resourceId": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group>/PROVIDERS/MICROSOFT.APIMANAGEMENT/SERVICE/APIM-HELLO-WORLD"
}
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Afficher les métriques de votre API
> * Configurer une règle d’alerte 
> * Afficher les journaux d’activité
> * Activer et afficher les journaux de ressources


Passez au tutoriel suivant :

> [!div class="nextstepaction"]
> [Suivre des appels](api-management-howto-api-inspector.md)