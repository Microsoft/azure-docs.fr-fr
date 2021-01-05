---
title: Surveiller les opérations, les événements et les compteurs pour un équilibreur de charge public
titleSuffix: Azure Load Balancer
description: Découvrez comment activer la journalisation pour Azure Load Balancer.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/05/2020
ms.author: allensu
ms.openlocfilehash: fcfd3da30ef9ace723b4204f5924591b1e2717f8
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97503163"
---
# <a name="azure-monitor-logs-for-azure-standard-load-balancer"></a>Journaux Azure Monitor pour Azure Standard Load Balancer

Vous pouvez utiliser différents types de journaux Azure Monitor pour gérer Azure Standard Load Balancer et résoudre les problèmes associés. Les journaux peuvent être envoyés en streaming à un hub d’événements ou à un espace de travail Log Analytics. Vous pouvez extraire tous les journaux du stockage Blob Azure et les afficher dans des outils comme Excel et Power BI. 

Les types de journaux sont les suivants :

* **Journaux d’activité :** Vous pouvez afficher toutes les activités envoyées à vos abonnements Azure, ainsi que leur état. Pour plus d’informations, consultez [Afficher les journaux d’activité pour surveiller les actions sur les ressources](../azure-resource-manager/management/view-activity-logs.md). Les journaux d’activité sont activés par défaut et peuvent être affichés dans le portail Azure. Ces journaux sont disponibles pour Azure Basic Load Balancer et Standard Load Balancer.
* **Métriques de Standard Load Balancer :** Vous pouvez utiliser ce journal pour interroger les métriques exportées sous forme de journaux pour Standard Load Balancer. Ces journaux sont disponibles uniquement pour Standard Load Balancer.

> [!IMPORTANT]
> Actuellement, les journaux d’événements de sonde d’intégrité et d’alertes Load Balancer ne sont pas fonctionnels et sont listés parmi les [problèmes connus avec Azure Load Balancer](whats-new.md#known-issues). 

> [!IMPORTANT]
> Les journaux d’activité ne sont disponibles que pour les ressources déployées dans le modèle de déploiement Azure Resource Manager. Vous ne pouvez pas les utiliser pour les ressources utilisant le modèle de déploiement classique. Pour plus d’informations sur les modèles de déploiement, consultez [Présentation du déploiement Resource Manager et du déploiement classique](../azure-resource-manager/management/deployment-models.md).

## <a name="enable-logging"></a>Activation de la journalisation

La journalisation d’activité est automatiquement activée pour chaque ressource Resource Manager. Activez la journalisation des événements et des sondes d’intégrité pour commencer à collecter les données disponibles dans ces journaux. Utiliser les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Si vous ne disposez pas déjà d'un équilibreur de charge, [créez un équilibreur de charge](./quickstart-load-balancer-standard-public-portal.md) avant de continuer.
1. Dans le portail, sélectionnez **Groupes de ressources**.
2. Sélectionnez **\<resource-group-name>** l’emplacement de votre équilibreur de charge.
3. Sélectionnez votre équilibreur de charge.
4. Sélectionnez **Journal d’activité** > **Paramètres de diagnostic**.
5. Dans le volet **Paramètres de diagnostic**, sous **Paramètres de diagnostic**, sélectionnez **+ Ajouter un paramètre de diagnostic**.
6. Dans le volet de création de **Paramètres de diagnostic**, entrez **myLBDiagnostic** dans la zone **Nom**.
7. Trois options s’offrent à vous pour les **Paramètres de diagnostic**. Vous pouvez en choisir une, deux ou les trois, et configurer chacune d’elles selon vos besoins :

   * **Archive vers un compte de stockage**. Vous avez besoin d’un compte de stockage déjà créé pour ce processus. Pour créer un compte de stockage, consultez [Créer un compte de stockage](../storage/common/storage-account-create.md?tabs=azure-portal).
     1. Cochez la case **Archiver dans un compte de stockage**.
     2. Sélectionnez **Configurer** pour ouvrir le volet **Sélectionnez un compte de stockage**.
     3. Dans la liste déroulante **Abonnement**, sélectionnez l’abonnement dans lequel votre compte de stockage a été créé.
     4. Dans la liste déroulante **Compte de stockage**, sélectionnez le nom de votre compte de stockage.
     5. Sélectionnez **OK**.

   * **Transmettre à un Event Hub**. Vous avez besoin d’un hub d’événements déjà créé pour ce processus. Pour créer un hub d’événements, consultez [Démarrage rapide : Créer un hub d’événement à l’aide du portail Azure](../event-hubs/event-hubs-create.md).
     1. Cochez la case **Diffuser vers Event Hub**.
     2. Sélectionnez **Configurer** pour ouvrir le volet **Sélectionner un hub d’événements**.
     3. Dans la liste déroulante **Abonnement**, sélectionnez l’abonnement dans lequel votre hub d’événements a été créé.
     4. Dans la liste déroulante **Sélectionner l’espace de noms Event Hub**, sélectionnez l’espace de noms.
     5. Dans la liste déroulante **Sélectionner le nom de stratégie Event Hub**, sélectionnez le nom.
     6. Sélectionnez **OK**.

   * **Envoyer à Log Analytics**. Vous devez avoir un espace de travail Log Analytics déjà créé et configuré pour ce processus. Pour créer un espace de travail Log Analytics, consultez [Créer un espace de travail Log Analytics dans le portail Azure](../azure-monitor/learn/quick-create-workspace.md).
     1. Cochez la case **Envoyer à Log Analytics**.
     2. Dans la liste déroulante **Abonnement**, sélectionnez l’abonnement où figure votre espace de travail Log Analytics.
     3. Dans la liste déroulante **Espace de travail Log Analytics**, sélectionnez l’espace de travail.

8. Dans la section **MÉTRIQUE** du volet **Paramètres de diagnostic**, cochez la case **AllMetrics**.

9. Vérifiez que tout semble correct, puis sélectionnez **Enregistrer** en haut du volet de création **Paramètres de diagnostic**.

## <a name="view-and-analyze-the-activity-log"></a>Afficher et analyser le journal d’activité

Le journal d’activité est généré par défaut. Vous pouvez le configurer pour l’exporter à un niveau d’abonnement [en suivant les instructions de cet article](https://docs.microsoft.com/azure/azure-monitor/platform/activity-log). Pour en savoir plus sur ces journaux, lisez l’article [Afficher les journaux d’activité pour superviser les actions sur les ressources](../azure-resource-manager/management/view-activity-logs.md).

Vous pouvez afficher et analyser les données du journal d’activité en utilisant l’une ou l’autre des méthodes suivantes :

* **Outils Azure** : récupérez les informations du journal d’activité en utilisant Azure PowerShell, Azure CLI, l’API REST Azure ou le portail Azure. L’article [Opérations d’audit avec Resource Manager](../azure-resource-manager/management/view-activity-logs.md) fournit des instructions pas à pas pour chaque méthode.
* **Power BI :** si vous n’avez pas encore de compte [Power BI](https://powerbi.microsoft.com/pricing), vous pouvez l’essayer gratuitement. À l’aide de l’[intégration des journaux d’audit Azure pour Power BI](https://powerbi.microsoft.com/integrations/azure-audit-logs/), vous pouvez analyser vos données avec des tableaux de bord préconfigurés. Vous pouvez aussi personnaliser les vues selon vos besoins.

## <a name="view-and-analyze-metrics-as-logs"></a>Afficher et analyser les métriques sous forme de journaux
À l’aide de la fonctionnalité d’exportation dans Azure Monitor, vous pouvez exporter vos métriques Load Balancer. Ces métriques génèrent une entrée de journal pour chaque intervalle d’échantillonnage d’une minute.

L’exportation de métriques sous forme de journaux est activée au niveau de chaque ressource. Pour activer ces journaux :

1. Accédez au volet **Paramètres de diagnostic**.
1. Filtrez par groupe de ressources, puis sélectionnez l’instance Load Balancer pour laquelle vous souhaitez activer l’exportation de métriques. 
1. Une fois la page des paramètres de diagnostic de l’instance Load Balancer affichée, sélectionnez **AllMetrics** pour exporter les métriques éligibles sous forme de journaux.

Pour connaître les limitations relatives à l’exportation de métriques, consultez la section [Limitations](#limitations) de cet article.

Une fois l’option **AllMetrics** activée dans les paramètres de diagnostic de Standard Load Balancer, si vous utilisez un hub d’événements ou un espace de travail Log Analytics, ces journaux sont renseignés dans la table **AzureMonitor**. 

Si vous exportez vers le stockage, connectez-vous à votre compte de stockage et récupérez les entrées de journal JSON pour les journaux des événements et des sondes d’intégrité. Après avoir téléchargé les fichiers JSON, vous pouvez les convertir en CSV et les afficher dans Excel, Power BI ou tout autre outil de visualisation de données. 

> [!TIP]
> Si vous savez utiliser Visual Studio et les concepts de base de la modification des valeurs de constantes et variables en C#, vous pouvez utiliser les [outils de convertisseur de journaux](https://github.com/Azure-Samples/networking-dotnet-log-converter) disponibles dans GitHub.

## <a name="stream-to-an-event-hub"></a>Diffuser vers un hub d’événements
Quand les informations de diagnostic sont envoyées en streaming vers un hub d’événements, elles peuvent être utilisées pour une analyse centralisée des journaux dans un outil SIEM partenaire avec intégration à Azure Monitor. Pour plus d’informations, consultez [Transmettre en continu des données de surveillance Azure à un hub d’événements](../azure-monitor/platform/stream-monitoring-data-event-hubs.md#partner-tools-with-azure-monitor-integration).

## <a name="send-to-log-analytics"></a>Envoyer à Log Analytics
Vous pouvez envoyer des informations de diagnostic pour les ressources figurant dans Azure directement à un espace de travail Log Analytics. Dans cet espace de travail, vous pouvez exécuter des requêtes complexes sur les informations en vue d’un dépannage et d’une analyse. Pour plus d’informations, consultez [Collecter les journaux de ressources Azure dans un espace de travail Log Analytics dans Azure Monitor](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace).

## <a name="limitations"></a>Limites
La fonctionnalité d’exportation des métriques sous forme de journaux pour Azure Load Balancer présente les limitations suivantes :
* Les métriques sont actuellement affichées via des noms internes lorsqu’ils sont exportés sous forme de journaux. Vous pouvez trouver le mappage dans le tableau ci-dessous.
* La dimensionnalité des métriques n’est pas conservée. Par exemple, avec des métriques telles que **DipAvailability** (état de la sonde d’intégrité), vous ne pouvez pas fractionner ni afficher par adresse IP de back-end.
* Actuellement, les métriques pour les ports SNAT utilisés et les ports SNAT alloués ne sont pas disponibles pour l’exportation sous forme de journaux.

## <a name="next-steps"></a>Étapes suivantes
* [Examiner les métriques disponibles pour votre équilibreur de charge](https://docs.microsoft.com/azure/load-balancer/load-balancer-standard-diagnostics)
* [Créer et tester des requêtes en suivant les instructions Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/log-query-overview)
