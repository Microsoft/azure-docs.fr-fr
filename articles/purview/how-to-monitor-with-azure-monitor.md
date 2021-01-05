---
title: Comment surveiller Azure Purview
description: Découvrez comment configurer les métriques, les alertes et les paramètres de diagnostic Azure Purview à l’aide d’Azure Monitor.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 12/03/2020
ms.openlocfilehash: 2c21f84b9a10db504afb8ead67ae479518a0afba
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602965"
---
# <a name="azure-purview-metrics-in-azure-monitor"></a>Métriques d’Azure Purview dans Azure Monitor

Cet article explique comment configurer les métriques, les alertes et les paramètres de diagnostic pour Azure Purview à l’aide d’Azure Monitor.

## <a name="monitor-azure-purview"></a>Surveiller Azure Purview

Les administrateurs d’Azure Purview peuvent utiliser Azure Monitor pour suivre l’état opérationnel d’un compte Purview. Les métriques sont collectées pour fournir des points de données vous permettant de suivre les problèmes potentiels, de les résoudre et d’améliorer la fiabilité du compte Purview. Les métriques sont envoyées à Azure Monitor pour les événements survenant dans Azure Purview.

## <a name="aggregated-metrics"></a>Métriques agrégées

Pour un compte Purview, les métriques sont accessibles à partir du portail Azure. L’accès aux métriques est contrôlé par l’attribution de rôle du compte Purview. Les utilisateurs doivent appartenir au rôle « Lecteur d’analyse » dans Azure Purview pour voir les métriques. Pour en savoir plus sur les niveaux d’accès des rôles, consultez [Autorisations du rôle Lecteur d’analyse](../azure-monitor/platform/roles-permissions-security.md#built-in-monitoring-roles).

La personne qui a créé le compte Purview obtient automatiquement les autorisations nécessaires pour consulter les métriques. Si quelqu’un d’autre souhaite voir les métriques, ajoutez-le au rôle **Lecteur d’analyse**, en procédant comme suit :

### <a name="add-a-user-to-the-monitoring-reader-role"></a>Ajouter un utilisateur au rôle Lecteur d’analyse

Pour ajouter un utilisateur au rôle **Lecteur d’analyse**, le propriétaire du compte Purview ou le propriétaire de l’abonnement peut effectuer les étapes suivantes :

1. Accédez au [portail Azure](https://portal.azure.com) et recherchez le nom du compte Azure Purview.

2. Sélectionnez **Contrôle d’accès (IAM)** .

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/access-iam.png" alt-text="Capture d’écran montrant comment accéder à IAM.":::

3. Sélectionnez **Ajouter une attribution de rôle**.

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/add-role-assignment.png" alt-text="Capture d’écran montrant comment ajouter une attribution de rôle.":::

4. Sélectionnez le rôle **Lecteur d’analyse** et attribuez l’accès à **Utilisateur, groupe ou principal de service Azure AD**. Attribuez ensuite le compte AAD pour accéder aux métriques.  

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/add-monitoring-reader.png" alt-text="Capture d’écran montrant comment ajouter le rôle Lecteur d’analyse.":::

## <a name="metrics-visualization"></a>Visualisation des métriques

Les utilisateurs du rôle **Lecteur d’analyse** peuvent voir les métriques agrégées et les journaux de diagnostic envoyés à Azure Monitor. Les métriques sont répertoriées dans le portail Azure pour le compte Purview correspondant. Dans le portail Azure, sélectionnez la section Métriques pour afficher la liste de toutes les métriques disponibles.

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/purview-metrics.png" alt-text="Capture d’écran montrant la section des métriques Purview disponibles." lightbox="./media/how-to-monitor-with-azure-monitor/purview-metrics.png":::

Les utilisateurs d’Azure Purview peuvent également accéder à la page des métriques directement à partir du centre de gestion du compte Azure Purview. Sélectionnez Azure Monitor dans la page principale du centre de gestion Purview pour lancer Portail Azure.

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/launch-metrics-from-management.png" alt-text="Capture d’écran illustrant le lancement des métriques Purview à partir du centre de gestion." lightbox="./media/how-to-monitor-with-azure-monitor/launch-metrics-from-management.png":::

### <a name="available-metrics"></a>Métriques disponibles

Pour vous familiariser avec l’utilisation de la section Métriques dans le portail Azure, lisez au préalable les deux documents suivants : [Prise en main de Metrics Explorer](../azure-monitor/platform/metrics-getting-started.md) et [Fonctionnalités avancées de Metrics Explorer](../azure-monitor/platform/metrics-charts.md).

Le tableau suivant contient la liste des métriques disponibles à explorer dans le portail Azure :

| Nom de métrique | Espace de noms de métrique | Type d’agrégation | Description |
| ------------------- | ------------------- | ------------------- | ----------------- |
| Analyse annulée | Analyse automatisée | SUM <br> Nombre | Agréger les analyses de sources de données annulées sur une période donnée |
| Analyse effectuée | Analyse automatisée | SUM <br> Nombre | Agréger les analyses de sources de données terminées sur une période donnée |
| Analyse ayant échoué | Analyse automatisée | SUM <br> Nombre | Agréger les analyses de sources de données ayant échoué sur une période donnée |
| Durée de l’analyse | Analyse automatisée | Min <br> Max <br> SUM <br> Avg | Agréger le temps total pris par les analyses sur une période donnée |

## <a name="diagnostic-logs-to-azure-storage-account"></a>Des journaux de diagnostic au compte de stockage Azure

Les événements de télémétrie bruts sont émis vers Azure Monitor. Les événements peuvent être enregistrés sur le compte de stockage client de votre choix pour une analyse plus poussée. L’exportation des journaux s’effectue via les paramètres de diagnostic du compte Purview sur le portail Azure.

Suivez les étapes pour créer un paramètre de diagnostic pour votre compte Azure Purview.

1. Créez un nouveau paramètre de diagnostic pour collecter les journaux et les métriques de la plateforme en suivant cet article : [Créez des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations](../azure-monitor/platform/diagnostic-settings.md). Sélectionnez le compte de stockage Azure comme seule destination.

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/step-one-diagnostic-setting.png" alt-text="Capture d’écran montrant la création du journal de diagnostic." lightbox="./media/how-to-monitor-with-azure-monitor/step-one-diagnostic-setting.png":::

2. Consignez les événements dans un compte de stockage. Un compte de stockage dédié est recommandé pour l’archivage des journaux de diagnostic. Suivez cet article pour [créer un compte de stockage](../storage/common/storage-quickstart-create-account.md?tabs=azure-portal).

   :::image type="content" source="./media/how-to-monitor-with-azure-monitor/step-two-diagnostic-setting.png" alt-text="Capture d’écran montrant l’affectation du compte de stockage pour le journal de diagnostic." lightbox="./media/how-to-monitor-with-azure-monitor/step-two-diagnostic-setting.png":::

Attendez jusqu’à 15 minutes avant de commencer à recevoir des journaux dans le compte de stockage nouvellement créé. [Consultez la conservation des données et le schéma des journaux de ressources dans le compte de stockage Azure](../azure-monitor/platform/resource-logs-collect-storage.md). Une fois les journaux de diagnostic configurés, les événements sont transmis au compte de stockage.

### <a name="scanstatuslogevent"></a>ScanStatusLogEvent

L’événement suit le cycle de vie de l’analyse. Une opération d’analyse suit la progression à travers une séquence d’états, de Mise en file d’attente, à En cours d’exécution et enfin un état terminal Réussi, Échec ou Annulée. Un événement est enregistré pour chaque transition d’état et le schéma de l’événement aura les propriétés suivantes.

```JSON
{
  "time": "<The UTC time when the event occurred>",
  "properties": {
    "dataSourceName": "<Registered data source friendly name>",
    "dataSourceType": "<Registered data source type>",
    "scanName": "<Scan instance friendly name>",
    "assetsDiscovered": "<If the resultType is succeeded, count of assets discovered in scan run>",
    "assetsClassified": "<If the resultType is succeeded, count of assets classified in scan run>",
    "scanQueueTimeInSeconds": "<If the resultType is succeeded, total seconds the scan instance in queue>",
    "scanTotalRunTimeInSeconds": "<If the resultType is succeeded, total seconds the scan took to run>",
    "runType": "<How the scan is triggered>",
    "errorDetails": "<Scan failure error>",
    "scanResultId": "<Unique GUID for the scan instance>"
  },
  "resourceId": "<The azure resource identifier>",
  "category": "<The diagnostic log category>",
  "operationName": "<The operation that cause the event Possible values for ScanStatusLogEvent category are: 
                    |AdhocScanRun 
                    |TriggeredScanRun 
                    |StatusChangeNotification>",
  "resultType": "Queued – indicates a scan is queued. 
                 Running – indicates a scan entered a running state. 
                 Succeeded – indicates a scan completed successfully. 
                 Failed – indicates a scan failure event. 
                 Cancelled – indicates a scan was cancelled. ",
  "resultSignature": "<Not used for ScanStatusLogEvent category. >",
  "resultDescription": "<This will have an error message if the resultType is Failed. >",
  "durationMs": "<Not used for ScanStatusLogEvent category. >",
  "level": "<The log severity level. Possible values are:
            |Informational
            |Error >",
  "location": "<The location of the Azure Purview account>",
}
```

L’exemple de journal d’une instance d’événement est présenté dans la section ci-dessous.

```JSON
{
  "time": "2020-11-24T20:25:13.022860553Z",
  "properties": {
    "dataSourceName": "AzureDataExplorer-swD",
    "dataSourceType": "AzureDataExplorer",
    "scanName": "Scan-Kzw-shoebox-test",
    "assetsDiscovered": "0",
    "assetsClassified": "0",
    "scanQueueTimeInSeconds": "0",
    "scanTotalRunTimeInSeconds": "0",
    "runType": "Manual",
    "errorDetails": "empty_value",
    "scanResultId": "0dc51a72-4156-40e3-8539-b5728394561f"
  },
  "resourceId": "/SUBSCRIPTIONS/111111111111-111-4EB2/RESOURCEGROUPS/FOOBAR-TEST-RG/PROVIDERS/MICROSOFT.PURVIEW/ACCOUNTS/FOOBAR-HEY-TEST-NEW-MANIFEST-EUS",
  "category": "ScanStatusLogEvent",
  "operationName": "TriggeredScanRun",
  "resultType": "Delayed",
  "resultSignature": "empty_value",
  "resultDescription": "empty_value",
  "durationMs": 0,
  "level": "Informational",
  "location": "eastus",
}
```

## <a name="next-steps"></a>Étapes suivantes

[Afficher les insights relatifs aux ressources](asset-insights.md)
