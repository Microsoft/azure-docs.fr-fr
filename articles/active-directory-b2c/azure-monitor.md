---
title: Superviser Azure AD B2C avec Azure Monitor
titleSuffix: Azure AD B2C
description: Découvrez comment journaliser des événements Azure AD B2C avec Azure Monitor en utilisant la gestion des ressources déléguées.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.author: mimart
ms.subservice: B2C
ms.date: 11/12/2020
ms.openlocfilehash: 6d40eab12c9726459543d0b69e27b73178eba99f
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96170614"
---
# <a name="monitor-azure-ad-b2c-with-azure-monitor"></a>Superviser Azure AD B2C avec Azure Monitor

Utilisez Azure Monitor pour router les journaux de connexion et d’[audit](view-audit-logs.md) d’Azure Active Directory B2C (Azure AD B2C) vers différentes solutions de supervision. Vous pouvez ensuite conserver les journaux pour les utiliser sur le long terme ou les intégrer à des outils SIEM (Security Information and Event Management) tiers pour obtenir davantage d’insights sur votre environnement.

Vous pouvez router des événements de journal vers :

* Un [compte de stockage](../storage/blobs/storage-blobs-introduction.md) Azure.
* Un [espace de travail Log Analytics](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace) (pour analyser les données, créer des tableaux de bord et générer des alertes sur des événements spécifiques).
* Un [hub d’événements](../event-hubs/event-hubs-about.md) Azure (pour les intégrer à vos instances Splunk et Sumo Logic).

![Azure Monitor](./media/azure-monitor/azure-monitor-flow.png)

Dans cet article, vous allez apprendre à transférer les journaux vers un espace de travail Azure Log Analytics. Vous pouvez ensuite créer un tableau de bord ou des alertes basées sur les activités des utilisateurs Azure AD B2C.

## <a name="deployment-overview"></a>Vue d’ensemble du déploiement

Azure AD B2C tire parti de la [supervision Azure Active Directory](../active-directory/reports-monitoring/overview-monitoring.md). Pour activer les *Paramètres de diagnostic* dans Azure Active Directory au sein de votre locataire Azure AD B2C, vous utilisez [Azure Lighthouse](../lighthouse/concepts/azure-delegated-resource-management.md) pour [déléguer une ressource](../lighthouse/concepts/azure-delegated-resource-management.md), ce qui permet à votre Azure AD B2C (le **Fournisseur de services**) de gérer une ressource Azure AD (le **Client**). Une fois que vous avez terminé la procédure de cet article, vous avez accès au groupe de ressources *azure-ad-b2c-monitor* qui contient l’[espace de travail Log Analytics](../azure-monitor/learn/quick-create-workspace.md) dans votre portail **Azure AD B2C**. Vous pouvez également transférer les journaux d’Azure AD B2C vers votre espace de travail Log Analytics.

Pendant ce déploiement, vous allez autoriser un utilisateur ou un groupe de votre annuaire Azure AD B2C à configurer l’instance de l’espace de travail Log Analytics au sein du locataire qui contient votre abonnement Azure. Pour créer l’autorisation, vous déployez un modèle [Azure Resource Manager](../azure-resource-manager/index.yml) sur votre locataire Azure AD contenant l’abonnement.

Le diagramme suivant illustre les composants que vous allez configurer dans vos locataires Azure AD et Azure AD B2C.

![Projection de groupe de ressources](./media/azure-monitor/resource-group-projection.png)

Pendant ce déploiement, vous allez configurer vos locataires Azure AD B2C et Azure AD dans lesquels l’espace de travail Log Analytics sera hébergé. Le compte utilisé pour exécuter le déploiement doit recevoir le rôle d'[Administrateur général](../active-directory/roles/permissions-reference.md#limit-use-of-global-administrator) dans ces deux locataires. Il est également important de vérifier que vous êtes connecté au répertoire correct lorsque vous effectuez chaque étape, comme décrit.

## <a name="1-create-or-choose-resource-group"></a>1. Créer ou choisir un groupe de ressources

Tout d’abord, créez ou choisissez un groupe de ressources qui contient l’espace de travail Log Analytics de destination qui recevra les données d’Azure AD B2C. Spécifiez le nom du groupe de ressources lorsque vous déployez le modèle Azure Resource Manager.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre **locataire Azure AD**.
1. [Créez un groupe de ressources](../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups) ou utilisez un groupe existant. Cet exemple utilise un groupe de ressources nommé *azure-ad-b2c-monitor*.

## <a name="2-create-a-log-analytics-workspace"></a>2. Créer un espace de travail Log Analytics

Un **espace de travail Log Analytics** est un environnement unique pour les données de journal d’activité Azure Monitor. Vous utiliserez cet espace de travail Log Analytics pour collecter des données à partir des [journaux d’audit](view-audit-logs.md) Azure AD B2C, puis pour les visualiser avec des requêtes et des classeurs ou pour créer des alertes.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre **locataire Azure AD**.
1. [Créez un espace de travail Log Analytics](../azure-monitor/learn/quick-create-workspace.md). Cet exemple utilise un espace de travail Log Analytics nommé *AzureAdB2C*, dans un groupe de ressources nommé *azure-ad-b2c-monitor*.

## <a name="3-delegate-resource-management"></a>3. Gestion des ressources déléguées

Au cours de cette étape, vous choisissez votre locataire Azure AD B2C en tant que **fournisseur de services**. Vous pouvez définir les autorisations nécessaires pour attribuer les rôles intégrés Azure appropriés aux groupes de votre locataire Azure AD.

### <a name="31-get-your-azure-ad-b2c-tenant-id"></a>3.1 Obtenir votre ID de locataire Azure AD B2C

Commencez par obtenir l’**ID de locataire** de votre annuaire Azure AD B2C (également appelé ID d’annuaire).

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire **Azure AD B2C**.
1. Sélectionnez **Azure Active Directory**, puis **Vue d’ensemble**.
1. Enregistrez l’**ID de locataire**.

### <a name="32-select-a-security-group"></a>3.2 Sélectionner un groupe de sécurité

Sélectionnez maintenant un groupe ou utilisateur Azure AD B2C auquel vous voulez octroyer l’autorisation sur le groupe de ressources que vous avez créé précédemment dans l’annuaire contenant votre abonnement.  

Pour faciliter la gestion, nous vous recommandons d’utiliser des *groupes* d’utilisateurs Azure AD pour chaque rôle, ce qui vous permet d’ajouter ou de supprimer des utilisateurs dans le groupe au lieu d’attribuer directement des autorisations aux utilisateurs. Dans cette procédure pas à pas, nous allons ajouter un groupe de sécurité.

> [!IMPORTANT]
> Pour que vous puissiez ajouter des autorisations pour un groupe Azure AD, le **type de groupe** doit être défini sur **Sécurité**. Cette option est sélectionnée lors de la création du groupe. Pour plus d’informations, consultez [Créer un groupe de base et ajouter des membres avec Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

1. Avec **Azure Active Directory** toujours sélectionné dans votre répertoire **Azure AD B2C**, sélectionnez **Groupes**, puis sélectionnez un groupe. Si vous n’avez pas de groupe existant, créez un groupe **Sécurité**, puis ajoutez des membres. Pour plus d’informations, suivez la procédure figurant dans [Créer un groupe de base et ajouter des membres avec Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). 
1. Sélectionnez **Vue d’ensemble** et enregistrez l’**ID d’objet** du groupe.

### <a name="33-create-an-azure-resource-manager-template"></a>3.3 Créer un modèle Azure Resource Manager

Ensuite, vous allez créer un modèle Azure Resource Manager qui accorde l’accès Azure AD B2C au groupe de ressources Azure AD que vous avez créé précédemment (par exemple, *azure-ad-b2c-monitor*). Déployez le modèle à partir de l’exemple GitHub à l’aide du bouton **Déployer sur Azure**, qui ouvre le portail Azure et vous permet de configurer et de déployer le modèle directement dans le portail. Pour cette procédure, assurez-vous que vous êtes connecté à votre locataire Azure AD (et non au locataire Azure AD B2C).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire **Azure AD**.
3. Le bouton **Déployer sur Azure** permet d’ouvrir le portail Azure et de déployer le modèle directement dans le portail. Pour plus d’informations, consultez [Créer un modèle Azure Resource Manager](../lighthouse/how-to/onboard-customer.md#create-an-azure-resource-manager-template).

   [![Déployer sur Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Lighthouse-samples%2Fmaster%2Ftemplates%2Frg-delegated-resource-management%2FrgDelegatedResourceManagement.json)

5. Sur la page **Déploiement personnalisé**, entrez les informations suivantes :

   | Champ   | Définition |
   |---------|------------|
   | Abonnement |  Sélectionnez le répertoire qui contient l’abonnement Azure dans lequel le groupe de ressources *azure-ad-b2c-monitor* a été créé. |
   | Région| Sélectionnez la région où la ressource sera déployée.  | 
   | Nom de l’offre MSP| Nom décrivant cette définition. Par exemple, *Supervision d’Azure AD B2C*.  |
   | Description de l’offre MSP| Courte description de votre offre. Par exemple, *Active Azure Monitor dans Azure AD B2C*.|
   | Géré par l’ID de locataire| **ID de locataire** de votre locataire Azure AD B2C (également appelé ID d’annuaire). |
   |Autorisations|Spécifiez un tableau JSON d’objets qui incluent Azure AD `principalId`, `principalIdDisplayName` et Azure `roleDefinitionId`. Ce `principalId` est l’**ID d’objet** du groupe ou de l’utilisateur B2C qui aura accès aux ressources de cet abonnement Azure. Pour cette procédure pas à pas, spécifiez l’ID d’objet du groupe que vous avez enregistré précédemment. Pour `roleDefinitionId`, utilisez la valeur du [rôle intégré](../role-based-access-control/built-in-roles.md) du *rôle Contributeur*, `b24988ac-6180-42a0-ab88-20f7382dd24c`.|
   | Nom du groupe de ressources | Nom du groupe de ressources que vous avez créé précédemment dans votre locataire Azure AD. Par exemple, *azure-ad-b2c-monitor*. |

   L’exemple suivant illustre un tableau d’autorisations avec un groupe de sécurité.

   ```json
   [
       {
           "principalId": "<Replace with group's OBJECT ID>",
           "principalIdDisplayName": "Azure AD B2C tenant administrators",
           "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"
       }
   ]
   ```

Une fois le modèle déployé, quelques minutes peuvent être nécessaires (généralement pas plus de cinq) pour effectuer la projection des ressources. Vous pouvez vérifier le déploiement dans votre locataire Azure AD et obtenir les détails de la projection de ressources. Pour plus d’informations, consultez [Voir et gérer les fournisseurs de services](../lighthouse/how-to/view-manage-service-providers.md).

## <a name="4-select-your-subscription"></a>4. Sélectionnez votre abonnement

Une fois que vous avez déployé le modèle et patienté quelques minutes pour que la projection des ressources s’effectue, procédez comme suit pour associer votre abonnement à votre annuaire Azure AD B2C.

1. Déconnectez-vous du portail Azure si vous êtes actuellement connecté (cela permet l’actualisation des informations d’identification de votre session à l’étape suivante).
2. Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte d’administrateur **Azure AD B2C**. Ce compte doit être membre du groupe de sécurité que vous avez spécifié à l’étape [Gestion des ressources déléguées](#3-delegate-resource-management) .
3. Sélectionnez l’icône **Répertoire + abonnement** dans la barre d’outils du portail.
4. Sélectionnez l’annuaire Azure AD qui contient l’abonnement Azure et le groupe de ressources *azure-ad-b2c-monitor* que vous avez créé.

    ![Changer d’annuaire](./media/azure-monitor/azure-monitor-portal-03-select-subscription.png)

1. Vérifiez que vous avez sélectionné l’annuaire et l’abonnement appropriés. Dans cet exemple, tous les annuaires et tous les abonnements sont sélectionnés.

    ![Option Tous les annuaires sélectionnée dans le filtre Annuaire et abonnement](./media/azure-monitor/azure-monitor-portal-04-subscriptions-selected.png)

## <a name="5-configure-diagnostic-settings"></a>5. Configurer les paramètres de diagnostic

Les paramètres de diagnostic définissent où les journaux et les métriques d’une ressource doivent être envoyés. Les destinations possibles sont les suivantes :

- [Compte Azure Storage](../azure-monitor/platform/resource-logs.md#send-to-azure-storage)
- Solutions [Event Hub](../azure-monitor/platform/resource-logs.md#send-to-azure-event-hubs)
- [Espace de travail Log Analytics](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace)

Dans cet exemple, nous utilisons l’espace de travail Log Analytics pour créer un tableau de bord.

### <a name="51-create-diagnostic-settings"></a>5.1 Créer des paramètres de diagnostic

Vous êtes maintenant prêt à [créer des paramètres de diagnostic](../active-directory/reports-monitoring/overview-monitoring.md) dans le portail Azure.

Pour configurer les paramètres de supervision des journaux d’activité Azure AD B2C :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) avec votre compte d’administrateur Azure AD B2C. Ce compte doit être membre du groupe de sécurité que vous avez spécifié à l’étape [Sélectionnez un groupe de sécurité](#32-select-a-security-group).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.
1. Sélectionnez **Azure Active Directory**.
1. Sous **Supervision**, sélectionnez **Paramètres de diagnostic**.
1. Si des paramètres sont définis pour la ressource, la liste des paramètres configurés s’affiche. Sélectionnez **Ajouter un paramètre de diagnostic** pour ajouter un paramètre ou **Modifier** pour modifier un paramètre existant. Chaque paramètre ne peut pas avoir plus d’un type de destination.

    ![Volet Paramètres de diagnostic dans le portail Azure](./media/azure-monitor/azure-monitor-portal-05-diagnostic-settings-pane-enabled.png)

1. Si votre paramètre n’a pas de nom, nommez-le.
1. Cochez la case correspondant à chaque destination à laquelle envoyer les journaux. Sélectionnez **Configurer** pour spécifier les paramètres **comme décrit dans le tableau suivant**.
1. Sélectionnez **Envoyer vers Log Analytics**, puis sélectionnez le **Nom de l’espace de travail** que vous avez créé précédemment (`AzureAdB2C`).
1. Sélectionnez **AuditLogs** et **SignInLogs**.
1. Sélectionnez **Enregistrer**.

> [!NOTE]
> Il peut falloir jusqu’à 15 minutes après l’émission d’un événement pour qu’il [s’affiche dans un espace de travail Log Analytics](../azure-monitor/platform/data-ingestion-time.md). En outre, apprenez-en davantage sur les [latences de création de rapports Active Directory](../active-directory/reports-monitoring/reference-reports-latencies.md), qui peuvent avoir un impact sur l’obsolescence des données et jouer un rôle important dans la création de rapports.

Si vous voyez le message d’erreur « Pour configurer les paramètres de diagnostic de manière à utiliser Azure Monitor pour votre annuaire Azure AD B2C, vous devez configurer la gestion des ressources déléguées », vérifiez que vous vous connectez avec un utilisateur membre du [groupe de sécurité](#32-select-a-security-group) et [sélectionnez votre abonnement](#4-select-your-subscription).

## <a name="6-visualize-your-data"></a>6. Visualiser vos données

Vous pouvez maintenant configurer votre espace de travail Log Analytics pour visualiser vos données et configurer des alertes. Ces configurations peuvent être effectuées à la fois dans votre locataire Azure AD et dans votre locataire Azure AD B2C.

### <a name="61-create-a-query"></a>6.1 Créer une requête

Les requêtes de journal vous aident à tirer pleinement parti de la valeur des données collectées dans les journaux Azure Monitor. Un puissant langage de requête vous permet de joindre des données provenant de plusieurs tables, d’agréger des jeux de données volumineux et d’effectuer des opérations complexes avec un minimum de code. Vous pouvez répondre à pratiquement n’importe quelle question et effectuer n’importe quelle analyse tant que les données de soutien ont été collectées et que vous comprenez comment construire la bonne requête. Pour plus d’informations, voir [Bien démarrer avec les requêtes de journal dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md).

1. Dans l’**espace de travail Log Analytics**, sélectionnez **Journaux**.
1. Dans l’éditeur de requête, collez la requête [Langage de requête Kusto](/azure/data-explorer/kusto/query/) suivante. Cette requête affiche l’utilisation de la stratégie par opération au cours des x derniers jours. La durée par défaut est définie sur 90 jours (90j). Notez que la requête est axée uniquement sur l’opération dans laquelle un jeton/code est émis par la stratégie.

    ```kusto
    AuditLogs
    | where TimeGenerated  > ago(90d)
    | where OperationName contains "issue"
    | extend  UserId=extractjson("$.[0].id",tostring(TargetResources))
    | extend Policy=extractjson("$.[1].value",tostring(AdditionalDetails))
    | summarize SignInCount = count() by Policy, OperationName
    | order by SignInCount desc  nulls last
    ```

1. Sélectionnez **Exécuter**. Les résultats de la requête sont affichés en bas de l’écran.
1. Pour enregistrer votre requête en vue d’une utilisation ultérieure, sélectionnez **Enregistrer**.

   ![Éditeur de journaux Log Analytics](./media/azure-monitor/query-policy-usage.png)

1. Complétez les détails suivants :

    - **Nom** : entrez le nom de votre requête.
    - **Enregistrer sous** : sélectionnez `query`.
    - **Catégorie** : sélectionnez `Log`.

1. Sélectionnez **Enregistrer**.

Vous pouvez également modifier votre requête pour visualiser les données à l’aide de l’opérateur [render](/azure/data-explorer/kusto/query/renderoperator?pivots=azuremonitor).

```kusto
AuditLogs
| where TimeGenerated  > ago(90d)
| where OperationName contains "issue"
| extend  UserId=extractjson("$.[0].id",tostring(TargetResources))
| extend Policy=extractjson("$.[1].value",tostring(AdditionalDetails))
| summarize SignInCount = count() by Policy
| order by SignInCount desc  nulls last
| render  piechart
```

![Graphique à secteurs de l’éditeur de journaux Log Analytics](./media/azure-monitor/query-policy-usage-pie.png)

Pour obtenir d’autres exemples, consultez le [référentiel GitHub SIEM](https://aka.ms/b2csiem) Azure AD B2C.

### <a name="62-create-a-workbook"></a>6.2 Créer un classeur

Les classeurs fournissent un canevas flexible pour l’analyse des données et la création de rapports visuels enrichis au sein du portail Azure. Ils vous permettent d’exploiter plusieurs sources de données à travers l’écosystème Azure et de les combiner dans des expériences interactives unifiées. Pour plus d’informations, consultez [Classeurs Azure Monitor](../azure-monitor/platform/workbooks-overview.md).

Suivez les instructions ci-dessous pour créer un classeur à l’aide d’un modèle de la galerie JSON. Ce classeur fournit un tableau de bord **Insights utilisateur** et **Authentification** pour le locataire Azure AD B2C.

1. Dans l’**espace de travail Log Analytics**, sélectionnez **Classeurs**.
1. Dans la barre d’outils, sélectionnez l’option **+ Nouveau** pour créer un classeur.
1. Sur la page **Nouveau classeur**, sélectionnez **Éditeur avancé** à l’aide de l’option **</>** de la barre d’outils.

     ![Modèle de la galerie](./media/azure-monitor/wrkb-adv-editor.png)

1. Sélectionnez **Modèle de la galerie**.
1. Remplacez le code JSON dans le **Modèle de la galerie** par le contenu du [classeur de base Azure AD B2C](https://raw.githubusercontent.com/azure-ad-b2c/siem/master/workbooks/dashboard.json) :
1. Appliquez le modèle à l’aide du bouton **Appliquer**.
1. Sélectionnez le bouton **Fin de l’édition** dans la barre d’outils pour terminer les modifications du classeur.
1. Enfin, enregistrez le classeur à l’aide du bouton **Enregistrer** à partir de la barre d’outils.
1. Fournissez un **Titre**, tel que *Tableau de bord Azure AD B2C*.
1. Sélectionnez **Enregistrer**.

    ![Enregistrer le classeur](./media/azure-monitor/wrkb-title.png)

Le classeur affiche des rapports sous la forme d’un tableau de bord.

![Premier tableau de bord du classeur](./media/azure-monitor/wkrb-dashboard-1.png)

![Deuxième tableau de bord du classeur](./media/azure-monitor/wrkb-dashboard-2.png)

![Troisième tableau de bord du classeur](./media/azure-monitor/wrkb-dashboard-3.png)


## <a name="create-alerts"></a>Créez des alertes

Les alertes sont créées par des règles d’alerte dans Azure Monitor et peuvent exécuter automatiquement des requêtes enregistrées ou des recherches personnalisées dans les journaux à intervalles réguliers. Vous pouvez créer des alertes sur la base de métriques de performances spécifiques ou quand certains événements sont créés, quand un événement fait défaut ou quand un certain nombre d’événements sont créés dans une fenêtre de temps donnée. Par exemple, les alertes peuvent permettre de vous avertir lorsque le nombre moyen de connexions dépasse un certain seuil. Pour plus d’informations, consultez [Créer des alertes](../azure-monitor/learn/tutorial-response.md).


Suivez les instructions ci-dessous pour créer une alerte Azure, qui enverra une [notification par courrier électronique](../azure-monitor/platform/action-groups.md#configure-notifications) lors de chaque baisse de 25 % du **Nombre total de demandes** par rapport à la période précédente. L’alerte s’exécute toutes les 5 minutes et recherche la baisse dans la fenêtre des 24 dernières heures. Les alertes sont créées à l’aide du langage de requête Kusto.


1. Dans l’**espace de travail Log Analytics**, sélectionnez **Journaux**. 
1. Créez une **requête Kusto** à l’aide de la requête ci-dessous.

    ```kusto
    let start = ago(24h);
    let end = now();
    let threshold = -25; //25% decrease in total requests.
    AuditLogs
    | serialize TimeGenerated, CorrelationId, Result
    | make-series TotalRequests=dcount(CorrelationId) on TimeGenerated in range(start, end, 1h)
    | mvexpand TimeGenerated, TotalRequests
    | where TotalRequests > 0
    | serialize TotalRequests, TimeGenerated, TimeGeneratedFormatted=format_datetime(todatetime(TimeGenerated), 'yyyy-M-dd [hh:mm:ss tt]')
    | project   TimeGeneratedFormatted, TotalRequests, PercentageChange= ((toreal(TotalRequests) - toreal(prev(TotalRequests,1)))/toreal(prev(TotalRequests,1)))*100
    | order by TimeGeneratedFormatted
    | where PercentageChange <= threshold   //Trigger's alert rule if matched.
    ```

1. Sélectionnez **Exécuter** pour tester la requête. Les résultats doivent s’afficher en cas de baisse d’au moins 25 % du nombre total de demandes au cours des 24 dernières heures.
1. Pour créer une règle d’alerte basée sur la requête ci-dessus, utilisez l’option **+ Nouvelle règle d’alerte** disponible dans la barre d’outils.
1. Sur la page **Créer une règle d’alerte**, sélectionnez **Nom de la condition**. 
1. Sur la page **Configurer la logique du signal**, définissez les valeurs suivantes, puis utilisez le bouton **Terminé** pour enregistrer les modifications.
    * Logique d'alerte : Définissez **Nombre de résultats** sur **Supérieur à** **0**.
    * Évaluation basée sur : Sélectionnez **1440** pour Période (en minutes) et **5** pour Fréquence (en minutes) 

    ![Créer une condition de règle d’alerte](./media/azure-monitor/alert-create-rule-condition.png)

Une fois l’alerte créée, accédez à l’**espace de travail Log Analytics** et sélectionnez **Alertes**. Cette page affiche toutes les alertes déclenchées pendant la période définie par l’option **Plage temporelle**.  

### <a name="configure-action-groups"></a>Configurer des groupes actions

Les alertes Azure Monitor et Service Health utilisent des groupes d’actions pour avertir les utilisateurs qu’une alerte a été déclenchée. Vous pouvez y inclure l’envoi d'un appel vocal, d'un SMS ou d'un e-mail ; ou le déclenchement de différents types d'actions automatisées. Consultez l’aide dans [Créer et gérer des groupes d’actions dans le portail Azure](../azure-monitor/platform/action-groups.md)

Voici un exemple d’e-mail de notification d’alerte. 

   ![E-mail de notification](./media/azure-monitor/alert-email-notification.png)

## <a name="multiple-tenants"></a>Locataires multiples

Pour intégrer plusieurs journaux de locataire Azure AD B2C au même espace de travail Log Analytics (ou compte de stockage Azure ou Event Hub), vous avez besoin de déploiements distincts avec des valeurs **Nom d’offre MSP** différentes. Vérifiez que votre espace de travail Log Analytics se trouve dans le même groupe de ressources que celui que vous avez configuré dans [Créer ou choisir un groupe de ressources](#1-create-or-choose-resource-group).

Si vous travaillez avec plusieurs espaces de travail Log Analytics, utilisez [Requête inter-espaces de travail](../azure-monitor/log-query/cross-workspace-query.md) pour créer des requêtes qui fonctionnent sur plusieurs espaces de travail. Par exemple, la requête suivante effectue une jointure de deux journaux d’audit de différents locataires en fonction de la même catégorie (par exemple, Authentification) :

```kusto
workspace("AD-B2C-TENANT1").AuditLogs
| join  workspace("AD-B2C-TENANT2").AuditLogs
  on $left.Category== $right.Category
```

## <a name="change-the-data-retention-period"></a>Changer la période de rétention des données

Les journaux Azure Monitor sont conçus pour la mise à l’échelle et la prise en charge de la collecte, de l’indexation et du stockage de quantités importantes de données quotidiennes provenant de toute source de votre entreprise ou déployées dans Azure. Par défaut, les journaux sont conservés pendant 30 jours, mais la durée de rétention peut être augmentée jusqu’à deux ans. Apprenez à [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](../azure-monitor/platform/manage-cost-storage.md). Après avoir sélectionné le niveau tarifaire, vous pouvez [Modifier la période de conservation des données](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’exemples, consultez la [Galerie SIEM](https://aka.ms/b2csiem) Azure AD B2C. 

* Pour plus d’informations sur l’ajout et la configuration de paramètres de diagnostic dans Azure Monitor, consultez le [Tutoriel : Collecter et analyser des journaux de ressources à partir d’une ressource Azure](../azure-monitor/insights/monitor-azure-resource.md).

* Pour plus d’informations sur le streaming des journaux Azure AD vers un hub d’événements, consultez le [Tutoriel : Envoyer les journaux Azure Active Directory vers un hub d’événements Azure](../active-directory/reports-monitoring/tutorial-azure-monitor-stream-logs-to-event-hub.md).