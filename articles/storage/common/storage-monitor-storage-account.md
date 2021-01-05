---
title: Guide pratique pour superviser un compte Stockage Azure dans le portail Azure | Microsoft Docs
description: Découvrez comment surveiller un compte de stockage dans Azure en utilisant le portail Azure et Azure Storage Analytics.
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 01/09/2020
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.custom: monitoring
ms.openlocfilehash: bfecae4775655f732df8fd6cffea613cd39fb828
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92782006"
---
# <a name="monitor-a-storage-account-in-the-azure-portal"></a>Surveillance d'un compte de stockage dans le portail Azure

[Azure Storage Analytics](storage-analytics.md) fournit des métriques pour tous les services de stockage et des journaux d’activité pour les objets Blob, les files d’attente et les tables. Vous pouvez utiliser le [portail Azure](https://portal.azure.com) pour configurer les métriques et les journaux d’activité enregistrés pour votre compte, et configurer des graphiques qui fournissent des représentations visuelles de vos données de métrique. 

Nous vous recommandons de consulter [Azure Monitor pour le stockage](../../azure-monitor/insights/storage-insights-overview.md) (préversion). Il s’agit d’une fonctionnalité d’Azure Monitor qui fournit une analyse complète de vos comptes de Stockage Azure en présentant une vue unifiée des performances, de la capacité et de la disponibilité de vos services de Stockage Azure. Vous n’avez pas besoin d’activer ou de configurer quoi que ce soit, et vous pouvez afficher immédiatement ces métriques à partir des graphiques interactifs prédéfinis et d’autres visualisations incluses.

> [!NOTE]
> L’analyse des données de surveillance dans le portail Azure occasionne des frais. Pour plus d’informations, consultez [Storage Analytics](storage-analytics.md).
>
> Azure Files prend actuellement en charge les métriques de Storage Analytics, mais pas encore la journalisation.
>
> Les comptes de stockage d’objets blob de blocs de niveau de performance Premium ne prennent pas en charge les métriques d’analyse de stockage, mais ils prennent en charge la journalisation. Vous pouvez activer la journalisation par programmation via l’API REST ou la bibliothèque de client. Si vous souhaitez afficher les métriques pour des comptes de stockage d’objets blob de blocs de niveau de performance Premium, envisagez d’utiliser [Métriques de Stockage Azure dans Azure Monitor](../blobs/monitor-blob-storage.md).
>
> Pour obtenir un guide détaillé concernant l'utilisation de Storage Analytics et d'autres outils permettant d'analyser, de diagnostiquer et de résoudre les problèmes d'Azure Storage, consultez [Analyse, diagnostic et résolution des problèmes rencontrés sur Microsoft Azure Storage](storage-monitoring-diagnosing-troubleshooting.md).
>

## <a name="configure-monitoring-for-a-storage-account"></a>Configuration de la surveillance d'un compte de stockage

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Comptes de stockage** , puis le nom du compte de stockage pour ouvrir le tableau de bord du compte.
1. Sélectionnez **Diagnostics** dans la section **SURVEILLANCE** du panneau du menu.

    ![Capture d'écran mettant en évidence l'option Paramètres de diagnostic (classique) dans la section Surveillance (classique).](./media/storage-monitor-storage-account/storage-enable-metrics-00.png)

1. Sélectionnez le **type** de données de métrique pour chaque **service** à surveiller et la **stratégie de rétention** pour les données. Vous pouvez également désactiver la surveillance en définissant **l’état** sur **Off**.

    ![MonitoringOptions](./media/storage-monitor-storage-account/storage-enable-metrics-01.png)

   Pour définir la stratégie de rétention de données, déplacez le curseur **Rétention (en jours)** ou entrez le nombre de jours durant lesquels les données sont conservées (de 1 à 365 jours). La valeur par défaut pour les nouveaux comptes de stockage est de sept jours. Si vous ne souhaitez pas définir de stratégie de rétention, entrez 0. Dans ce cas, il vous appartient de supprimer ou non les données de surveillance.

   > [!WARNING]
   > Vous êtes facturé lorsque vous supprimez manuellement les données de métrique. Les données analytiques obsolètes (antérieures à votre stratégie de rétention des données) sont supprimées par le système sans frais. Nous vous recommandons de définir une stratégie de rétention en fonction de la durée de conservation que vous souhaitez appliquer aux données d’analyse de votre compte. Pour plus d’informations, consultez la page [Facturation sur les métriques de stockage](storage-analytics-metrics.md#billing-on-storage-metrics).
   >

1. Une fois la configuration de la surveillance terminée, sélectionnez **Enregistrer**.

Un ensemble de métriques par défaut s’affiche dans des graphiques sur le panneau du compte de stockage, ainsi que les panneaux des services individuels (blob, file d’attente, table et fichier). Une fois que vous avez activé les métriques d’un service, l’affichage des données dans les graphiques peut prendre jusqu’à une heure. Vous pouvez sélectionner **Modifier** sur n’importe quel graphique de métrique pour configurer les métriques qui sont affichées dans le graphique.

Vous pouvez désactiver la collecte de métriques et la journalisation en définissant **l’état** sur **Off**.

> [!NOTE]
> Le stockage Azure utilise un [stockage Table](storage-introduction.md#table-storage) pour stocker les métriques pour votre compte de stockage, et stocke les métriques dans des tables de votre compte. Pour plus d'informations, consultez [Stockage des métriques](storage-analytics-metrics.md#how-metrics-are-stored).
>

## <a name="customize-metrics-charts"></a>Personnaliser les graphiques de métrique

La procédure suivante permet de choisir les métriques de stockage à afficher dans un graphique de métrique.

1. Commencez par afficher un graphique de métrique de stockage dans le portail Azure. Vous trouverez les graphiques dans le **panneau du compte de stockage** et dans le panneau **Métriques** d’un service individuel (blob, file d’attente, table, fichier).

   Dans cet exemple, nous utilisons le graphique suivant qui apparaît dans le **panneau du compte de stockage** :

   ![Sélection de graphique dans le portail Azure](./media/storage-monitor-storage-account/stg-customize-chart-00.png)

1. Cliquez n’importe où dans le graphique pour le modifier.

1. Ensuite, sélectionnez la **Plage de temps** des métriques à afficher dans le graphique et le **service** (Blob, File d’attente, Table, Fichier) dont vous voulez afficher les métriques. Ici, nous avons choisi d’afficher les métriques de la semaine précédente pour le service Blob :

   ![Sélection de la période et du service dans le panneau Modifier le graphique](./media/storage-monitor-storage-account/storage-edit-metric-time-range.png)

1. Sélectionnez la **métrique** individuelle à afficher dans le graphique, puis cliquez sur **OK**.

   ![Sélection de métrique individuelle dans le panneau Modifier le graphique](./media/storage-monitor-storage-account/storage-edit-metric-selections.png)

Les paramètres de votre graphique n’ont aucune incidence sur la collecte, l’agrégation ou le stockage des données de surveillance dans le compte de stockage.

### <a name="metrics-availability-in-charts"></a>Disponibilité des métriques dans les graphiques

La liste des métriques disponibles change selon le service que vous avez choisi dans la liste déroulante et le type d’unité du graphique que vous modifiez. Par exemple, vous pouvez sélectionner les métriques de pourcentage comme *PercentNetworkError* et *PercentThrottlingError* uniquement si vous modifiez un graphique qui affiche les unités en pourcentage :

![Graphique de pourcentage d’erreur de demande dans le portail Azure](./media/storage-monitor-storage-account/stg-customize-chart-04.png)

### <a name="metrics-resolution"></a>Résolution des métriques

Les métriques que vous avez sélectionnées dans **Diagnostics** déterminent la résolution des métriques qui sont disponibles pour votre compte :

* La surveillance **Agréger** fournit des métriques telles que l’entrée/la sortie, la disponibilité, la latence et les pourcentages de réussite. Ces métriques sont agrégées à partir des services Blob, File d’attente, Table et Fichier.
* **Par API** offre une résolution plus pointue, avec les métriques disponibles pour les opérations de stockage individuelles, en plus des agrégats au niveau du service.

## <a name="configure-metrics-alerts"></a>Configurer des alertes de métriques

Vous pouvez créer des alertes pour vous avertir lorsque les seuils sont atteints pour les métriques des ressources de stockage.

1. Pour ouvrir le **panneau Règles d’alerte** , faites défiler jusqu’à la section **SURVEILLANCE** du **panneau Menu** et sélectionnez **Alertes (classiques)** .
2. Sélectionnez **Ajouter une alerte métrique (classique)** pour ouvrir le panneau **Ajouter une règle d’alerte**
3. Entrez un **Nom** et une **Description** pour votre nouvelle règle d’alerte.
4. Sélectionnez la **métrique** pour laquelle vous souhaitez ajouter une alerte, une **condition** d’alerte et un **seuil**. Le type d’unité de seuil change en fonction de la métrique choisie. Par exemple, « count » est le type d’unité pour *ContainerCount* , tandis que l’unité pour la métrique *PercentNetworkError* est un pourcentage.
5. Sélectionnez la **Période**. Les métriques qui atteignent ou dépassent le seuil durant la période déclenchent une alerte.
6. (Facultatif) Configurez des notifications **E-mail** et **Webhook**. Pour plus d’informations sur webhooks, consultez [Configurer un webhook sur une alerte de métrique Azure](../../azure-monitor/platform/alerts-webhooks.md). Si vous ne configurez pas de notifications e-mail ou webhook, les alertes s’affichent uniquement dans le portail Azure.

![Panneau « Ajouter une règle d’alerte » dans le portail Azure](./media/storage-monitor-storage-account/add-alert-rule.png)

## <a name="add-metrics-charts-to-the-portal-dashboard"></a>Ajouter des graphiques de métriques au tableau de bord du portail

Vous pouvez ajouter des graphiques de métriques Stockage Azure pour l’un de vos comptes de stockage au tableau de bord du portail.

1. Cliquez sur **Modifier le tableau de bord** lorsque le tableau de bord est affiché dans le [portail Azure](https://portal.azure.com).
1. Dans la **Galerie de vignettes** , sélectionnez **Recherche des vignettes par** > **Type**.
1. Sélectionnez **Type** > **Comptes de stockage**.
1. Dans **Ressources** , sélectionnez le compte de stockage dont vous souhaitez ajouter les métriques au tableau de bord.
1. Sélectionnez **Catégories** > **Surveillance**.
1. Glissez-déplacez la vignette du graphique sur votre tableau de bord pour la métrique à afficher. Répétez cette opération pour toutes les métriques à afficher sur le tableau de bord. Dans l’image suivante, le graphique « Objets Blob - Nombre total de requêtes » est mis en surbrillance comme exemple, mais tous les graphiques peuvent être placés sur votre tableau de bord.

   ![Galerie de vignettes dans le portail Azure](./media/storage-monitor-storage-account/storage-customize-dashboard.png)
1. Sélectionnez **Personnalisation terminée** en haut du tableau de bord lorsque vous avez terminé d’ajouter des graphiques.

Une fois que vous avez ajouté des graphiques à votre tableau de bord, vous pouvez les personnaliser davantage comme décrit dans Personnaliser les graphiques de métrique.

## <a name="configure-logging"></a>Configuration de la journalisation

Vous pouvez demander au Stockage Azure d’enregistrer les journaux de diagnostic pour les requêtes de lecture, d’écriture et de suppression pour les services blob, table et file d’attente. La stratégie de rétention des données que vous définissez s’applique également à ces journaux d’activité.

> [!NOTE]
> Azure Files prend actuellement en charge les métriques de Storage Analytics, mais pas encore la journalisation.
>

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Comptes de stockage** , puis le nom du compte de stockage pour ouvrir le panneau du compte de stockage.
1. Sélectionnez **Paramètres de diagnostic (classique)** dans la section **Surveillance (classique)** du panneau du menu.

    ![Élément de menu Diagnostics sous SURVEILLANCE dans le portail Azure.](./media/storage-monitor-storage-account/storage-enable-metrics-00.png)

1. Vérifiez que **l’état** est défini sur **On** , puis sélectionnez les **services** pour lesquels vous souhaitez activer la journalisation.

    ![Configurez la journalisation dans le portail Azure.](./media/storage-monitor-storage-account/enable-diagnostics.png)
1. Cliquez sur **Enregistrer**.

Les journaux de diagnostic sont enregistrés dans un conteneur d’objets blob nommé *$logs* dans votre compte de stockage. Vous pouvez afficher les données du journal à l’aide d’un explorateur de stockage, tel que l’[Explorateur Stockage Microsoft Azure](https://storageexplorer.com), ou de façon programmatique à l’aide d’une bibliothèque de client de stockage ou PowerShell.

Pour plus d’informations sur l’accès au conteneur $logs, consultez la page [Journalisation Storage Analytics](storage-analytics-logging.md).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [les métriques, la journalisation et la facturation](storage-analytics.md) pour Storage Analytics.