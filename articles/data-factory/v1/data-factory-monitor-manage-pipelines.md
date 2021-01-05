---
title: Monitorage et gestion de pipelines avec le Portail Azure et PowerShell
description: Découvrez comment utiliser le portail Azure et Azure PowerShell pour surveiller et gérer les fabriques de données et les pipelines Azure que vous avez créés.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 04/30/2018
ms.openlocfilehash: 2a30c755bc19849ad3a821cbbc75b787a3b0bb98
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495852"
---
# <a name="monitor-and-manage-azure-data-factory-pipelines-by-using-the-azure-portal-and-powershell"></a>Surveiller et gérer les pipelines Azure Data Factory à l’aide du portail Azure et de PowerShell
> [!div class="op_single_selector"]
> * [Utilisation du portail Azure/d’Azure PowerShell](data-factory-monitor-manage-pipelines.md)
> * [Utilisation de l’application de surveillance et gestion](data-factory-monitor-manage-app.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Surveiller et gérer des pipelines Data Factory](../monitor-visually.md).

Cet article décrit comment surveiller, gérer et déboguer vos pipelines à l’aide du Portail Azure et de PowerShell.

> [!IMPORTANT]
> L’application de surveillance et gestion favorise la surveillance et la gestion de vos pipelines de données, ainsi que la résolution des problèmes. Pour en savoir plus sur l’utilisation de l’application, consultez [Surveiller et gérer les pipelines Azure Data Factory à l’aide de l’application de surveillance et gestion](data-factory-monitor-manage-app.md). 

> [!IMPORTANT]
> Azure Data Factory version 1 utilise maintenant la nouvelle [infrastructure d’alertes Azure Monitor](../../azure-monitor/platform/alerts-metric.md). L’ancienne infrastructure d’alertes est déconseillée. Par conséquent, vos alertes existantes configurées pour les fabriques de données version 1 ne fonctionnent plus. Vos alertes existantes pour les fabriques de données v1 ne sont pas migrées automatiquement. Vous devez recréer ces alertes sur la nouvelle infrastructure d’alertes. Connectez-vous au portail et sélectionnez **Surveiller** pour créer des alertes sur les métriques (par exemple les exécutions qui ont échoué ou réussi) pour vos fabriques de données version 1.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="understand-pipelines-and-activity-states"></a>Présentation des pipelines et des états d’activité
À l’aide du portail Azure, vous pouvez :

* Afficher votre fabrique de données sous forme de schéma.
* Afficher les activités à l’intérieur d’un pipeline.
* Afficher des jeux de données d’entrée et de sortie.

Cette section décrit également comment une tranche de jeu de données passe d’un état à un autre.   

### <a name="navigate-to-your-data-factory"></a>Accédez à votre fabrique de données
1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquer sur **Fabriques de données** dans le menu de gauche. Si vous ne voyez pas cette option, cliquez sur **Autres services >** , puis sur **Fabriques de données** dans la catégorie **INTELLIGENCE + ANALYSE**.

   ![Parcourir tout > Fabriques de données](./media/data-factory-monitor-manage-pipelines/browseall-data-factories.png)
3. Dans le panneau **Fabriques de données**, sélectionnez la fabrique de données qui vous intéresse.

    ![Sélectionner une fabrique de données](./media/data-factory-monitor-manage-pipelines/select-data-factory.png)

   Vous devez voir la page d’accueil de la fabrique de données apparaître.

   ![Panneau Data Factory](./media/data-factory-monitor-manage-pipelines/data-factory-blade.png)

#### <a name="diagram-view-of-your-data-factory"></a>Vue schématique de votre fabrique de données
La vue **schématique** d’une fabrique de données est un point unique de surveillance et de gestion de la fabrique de données et de ses ressources. Cliquez sur **Schématique** sur la page d’accueil de la fabrique de données ci-dessus pour afficher la vue **schématique** de votre fabrique de données.

![Vue schématique](./media/data-factory-monitor-manage-pipelines/diagram-view.png)

Vous pouvez faire un zoom avant, un zoom arrière, un zoom à 100 %, un zoom pour ajuster l’affichage à la taille de l’écran, figer l’affichage schématique et positionner automatiquement les pipelines et les jeux de données. Vous pouvez également afficher le lignage (c.-à-d. mise en surbrillance des éléments en amont et en aval des éléments sélectionnés).

### <a name="activities-inside-a-pipeline"></a>Activités à l'intérieur d'un pipeline
1. Cliquez avec le bouton droit sur le pipeline de votre choix, puis cliquez sur **Ouvrir le pipeline** pour faire apparaître toutes les activités dans le pipeline, ainsi que les jeux de données d’entrée et de sortie des activités. Cette fonctionnalité est utile quand votre pipeline comprend plusieurs activités et que vous souhaitez comprendre le lignage opérationnel d’un seul pipeline.

    ![Menu Ouvrir un pipeline](./media/data-factory-monitor-manage-pipelines/open-pipeline-menu.png)     
2. Dans l’exemple suivant, vous voyez une activité de copie dans le pipeline avec une entrée et une sortie. 

    ![Activités à l'intérieur d'un pipeline](./media/data-factory-monitor-manage-pipelines/activities-inside-pipeline.png)
3. Vous pouvez revenir à la page d’accueil de la fabrique de données en cliquant sur le lien **Data Factory** dans l’arborescence de navigation située dans le coin supérieur gauche.

    ![Revenir à la fabrique de données](./media/data-factory-monitor-manage-pipelines/navigate-back-to-data-factory.png)

### <a name="view-the-state-of-each-activity-inside-a-pipeline"></a>Afficher l’état de chaque activité à l’intérieur d’un pipeline
Vous pouvez afficher l’état actuel d’une activité en consultant l’état de l’un des jeux de données générés par l’activité.

En double-cliquant sur **OutputBlobTable** dans le **Diagramme**, vous pouvez voir toutes les tranches produites par les différentes exécutions de l’activité à l’intérieur d’un pipeline. Vous pouvez voir que l’activité de copie a été exécutée correctement au cours des huit dernières heures et qu’elle a produit les tranches dont l’état est **Prêt**.  

![État du pipeline](./media/data-factory-monitor-manage-pipelines/state-of-pipeline.png)

Voici la liste des différents états possibles pour les tranches d’un jeu de données de la fabrique de données :

<table>
<tr>
    <th align="left">State</th><th align="left">Sous-état</th><th align="left">Description</th>
</tr>
<tr>
    <td rowspan="8">En attente</td><td>ScheduleTime</td><td>L’heure n’est pas venue pour l’exécution de la tranche.</td>
</tr>
<tr>
<td>DatasetDependencies</td><td>Les dépendances en amont ne sont pas prêtes.</td>
</tr>
<tr>
<td>ComputeResources</td><td>Les ressources de calcul ne sont pas disponibles.</td>
</tr>
<tr>
<td>ConcurrencyLimit</td> <td>Toutes les instances d'activité sont occupées par l'exécution d'autres tranches.</td>
</tr>
<tr>
<td>ActivityResume</td><td>L’activité est mise en pause et ne peut pas exécuter les tranches jusqu’à sa reprise.</td>
</tr>
<tr>
<td>Recommencer</td><td>L’exécution de l’activité est retentée.</td>
</tr>
<tr>
<td>Validation</td><td>La validation n’a pas encore démarré.</td>
</tr>
<tr>
<td>ValidationRetry</td><td>Une nouvelle tentative de validation va avoir lieu.</td>
</tr>
<tr>
<tr>
<td rowspan="2">InProgress</td><td>Validation</td><td>La validation est en cours.</td>
</tr>
<td>-</td>
<td>La tranche est en cours de traitement.</td>
</tr>
<tr>
<td rowspan="4">Échec</td><td>TimedOut</td><td>L'exécution de l’activité a pris plus de temps que l’activité ne l’autorise.</td>
</tr>
<tr>
<td>Opération annulée</td><td>La tranche a été annulée par l’action de l’utilisateur.</td>
</tr>
<tr>
<td>Validation</td><td>La validation a échoué.</td>
</tr>
<tr>
<td>-</td><td>La validation et/ou la génération de la tranche a échoué.</td>
</tr>
<td>Ready</td><td>-</td><td>La tranche est prête à être consommée.</td>
</tr>
<tr>
<td>Ignoré</td><td>None</td><td>La tranche n’est pas en cours de traitement.</td>
</tr>
<tr>
<td>None</td><td>-</td><td>Tranche qui a été réinitialisée alors qu’elle existait avec un état différent.</td>
</tr>
</table>



Vous pouvez afficher les détails relatifs à une tranche en cliquant sur une entrée de tranche dans le panneau **Tranches mises à jour récemment**.

![Détails de la tranche](./media/data-factory-monitor-manage-pipelines/slice-details.png)

Si la tranche a été exécutée plusieurs fois, plusieurs lignes s’affichent dans la liste **Exécutions d’activité** . Vous pouvez afficher des détails sur une exécution d’activité en cliquant sur l’entrée d’exécution dans la liste **Exécutions de l’activité**. Tous les fichiers journaux, ainsi que le message d’erreur associé, le cas échéant, s’affichent. Cette fonctionnalité est utile et pour cause. Vous visualisez et déboguez les journaux d’activité sans le souci de quitter votre fabrique de données.

![Détails de l'exécution d'activité](./media/data-factory-monitor-manage-pipelines/activity-run-details.png)

Si la tranche n’a pas l’état **Prêt**, vous pouvez voir les tranches en amont qui ne sont pas prêtes et qui empêchent l’exécution de la tranche actuelle dans la liste **Tranches en amont qui ne sont pas prêtes**. Cette fonctionnalité est utile lorsque votre tranche présente l’état **En attente** et que vous voulez connaître les dépendances en amont à l’origine de cette attente.

![Tranches en amont qui ne sont pas prêtes](./media/data-factory-monitor-manage-pipelines/upstream-slices-not-ready.png)

### <a name="dataset-state-diagram"></a>Schéma d’état de jeux de données
Quand vous avez déployé une fabrique de données et que la période d’activation des pipelines est valide, les tranches de données des jeux de données passent d’un état à un autre. Actuellement, l’état de la tranche suit le schéma d’état suivant :

![Schéma d'état](./media/data-factory-monitor-manage-pipelines/state-diagram.png)

Le flux de transition d’état de l’ensemble de données est le suivant dans la fabrique de données : En attente -> En cours/en cours (validation) -> Prêt/Échec.

Au départ, la tranche a l’état **En attente**, en attente des conditions requises à respecter avant l’exécution. Ensuite, l’exécution de l’activité commence, et la tranche passe à l’état **En cours**. L’exécution de l’activité peut réussir ou échouer. Selon le résultat de l’exécution, l’état de la tranche est **Prête** ou **Échec**.

Vous pouvez réinitialiser la tranche pour revenir de l’état **Prête** ou **Échec** à l’état **En attente**. Vous pouvez également définir l’état de la tranche sur **Ignorer** pour empêcher l’exécution de l’activité et ne pas traiter la tranche.

## <a name="pause-and-resume-pipelines"></a>Suspension et reprise des pipelines
Vous pouvez gérer vos pipelines à l’aide d’Azure PowerShell. Par exemple, vous pouvez suspendre et reprendre les pipelines en exécutant les applets de commande Azure PowerShell. 

> [!NOTE] 
> La vue schématique ne prend pas en charge la suspension et la reprise des pipelines. Si vous souhaitez utiliser une interface utilisateur, utilisez l’application de surveillance et gestion. Pour en savoir plus sur l’utilisation de l’application, consultez l’article [Surveiller et gérer les pipelines Azure Data Factory à l’aide de l’application de surveillance et gestion](data-factory-monitor-manage-app.md). 

Vous pouvez suspendre l’exécution des pipelines à l’aide de la cmdlet PowerShell **Suspend-AzDataFactoryPipeline**. Cette applet de commande est utile lorsque vous ne voulez pas exécuter vos pipelines jusqu'à ce qu’un problème est résolu. 

```powershell
Suspend-AzDataFactoryPipeline [-ResourceGroupName] <String> [-DataFactoryName] <String> [-Name] <String>
```
Par exemple :

```powershell
Suspend-AzDataFactoryPipeline -ResourceGroupName ADF -DataFactoryName productrecgamalbox1dev -Name PartitionProductsUsagePipeline
```

Une fois le problème résolu au niveau du pipeline, vous pouvez reprendre le pipeline suspendu en exécutant la commande PowerShell suivante :

```powershell
Resume-AzDataFactoryPipeline [-ResourceGroupName] <String> [-DataFactoryName] <String> [-Name] <String>
```
Par exemple :

```powershell
Resume-AzDataFactoryPipeline -ResourceGroupName ADF -DataFactoryName productrecgamalbox1dev -Name PartitionProductsUsagePipeline
```

## <a name="debug-pipelines"></a>Débogage de pipelines
Azure Data Factory offre des fonctionnalités exceptionnelles pour déboguer et résoudre les problèmes des pipelines via le portail Azure et Azure PowerShell.

> [!NOTE] 
> Il est beaucoup plus facile de résoudre les erreurs à l’aide de l’application de surveillance et gestion. Pour en savoir plus sur l’utilisation de l’application, consultez l’article [Surveiller et gérer les pipelines Azure Data Factory à l’aide de l’application de surveillance et gestion](data-factory-monitor-manage-app.md). 

### <a name="find-errors-in-a-pipeline"></a>Recherche d’erreurs dans un pipeline
En cas d’échec d’exécution de l’activité dans un pipeline, le jeu de données généré par celui-ci est alors en état d’erreur. Vous pouvez déboguer et corriger les erreurs dans Azure Data Factory à l’aide des méthodes suivantes.

#### <a name="use-the-azure-portal-to-debug-an-error"></a>Utiliser le portail Azure pour déboguer une erreur
1. Dans le panneau **Table**, cliquez sur la tranche qui pose problème, dont **l’état** est défini sur **Échec**.

   ![Panneau de table avec tranche problématique](./media/data-factory-monitor-manage-pipelines/table-blade-with-error.png)
2. Dans le panneau **Tranche de données**, cliquez sur l’exécution d’activité qui a échoué.

   ![Tranche de données avec une erreur](./media/data-factory-monitor-manage-pipelines/dataslice-with-error.png)
3. Dans le panneau **Détails sur l’exécution d’activité**, vous pouvez télécharger les fichiers associés au traitement HDInsight. Cliquez sur **Télécharger** pour que Status/stderr télécharge le fichier journal d’erreur qui contient les détails sur l’erreur.

   ![Panneau de détails sur l’exécution d’activité](./media/data-factory-monitor-manage-pipelines/activity-run-details-with-error.png)     

#### <a name="use-powershell-to-debug-an-error"></a>Utiliser PowerShell pour déboguer une erreur
1. Lancez **PowerShell**.
2. Exécutez la commande **Get-AzDataFactorySlice** pour voir les tranches et leur état. Une tranche dont l’état est **Échec** devrait apparaître.        

    ```powershell   
    Get-AzDataFactorySlice [-ResourceGroupName] <String> [-DataFactoryName] <String> [-DatasetName] <String> [-StartDateTime] <DateTime> [[-EndDateTime] <DateTime> ] [-Profile <AzureProfile> ] [ <CommonParameters>]
    ```   
   Par exemple :

    ```powershell   
    Get-AzDataFactorySlice -ResourceGroupName ADF -DataFactoryName LogProcessingFactory -DatasetName EnrichedGameEventsTable -StartDateTime 2014-05-04 20:00:00
    ```

   Remplacez **StartDateTime** par l’heure de début de votre pipeline. 
3. Exécutez maintenant la cmdlet **Get-AzDataFactoryRun** pour obtenir des détails sur l’exécution de l’activité de la tranche.

    ```powershell   
    Get-AzDataFactoryRun [-ResourceGroupName] <String> [-DataFactoryName] <String> [-DatasetName] <String> [-StartDateTime]
    <DateTime> [-Profile <AzureProfile> ] [ <CommonParameters>]
    ```

    Par exemple :

    ```powershell   
    Get-AzDataFactoryRun -ResourceGroupName ADF -DataFactoryName LogProcessingFactory -DatasetName EnrichedGameEventsTable -StartDateTime "5/5/2014 12:00:00 AM"
    ```

    La valeur de StartDateTime est l’heure de début de la tranche qui pose problème/l’erreur que vous avez notée à l’étape précédente. La valeur date-heure doit être entourée de guillemets doubles.
4. Vous devez voir la sortie avec les détails sur l’erreur qui est semblable à ce qui suit :

    ```   
    Id                      : 841b77c9-d56c-48d1-99a3-8c16c3e77d39
    ResourceGroupName       : ADF
    DataFactoryName         : LogProcessingFactory3
    DatasetName               : EnrichedGameEventsTable
    ProcessingStartTime     : 10/10/2014 3:04:52 AM
    ProcessingEndTime       : 10/10/2014 3:06:49 AM
    PercentComplete         : 0
    DataSliceStart          : 5/5/2014 12:00:00 AM
    DataSliceEnd            : 5/6/2014 12:00:00 AM
    Status                  : FailedExecution
    Timestamp               : 10/10/2014 3:04:52 AM
    RetryAttempt            : 0
    Properties              : {}
    ErrorMessage            : Pig script failed with exit code '5'. See wasb://        adfjobs@spestore.blob.core.windows.net/PigQuery
                                    Jobs/841b77c9-d56c-48d1-99a3-
                8c16c3e77d39/10_10_2014_03_04_53_277/Status/stderr' for
                more details.
    ActivityName            : PigEnrichLogs
    PipelineName            : EnrichGameLogsPipeline
    Type                    :
    ```
5. Vous pouvez exécuter la cmdlet **Save-AzDataFactoryLog** avec la valeur d’ID indiquée dans la sortie et télécharger les fichiers journaux en utilisant le paramètre **-DownloadLogsoption** pour la cmdlet.

    ```powershell
    Save-AzDataFactoryLog -ResourceGroupName "ADF" -DataFactoryName "LogProcessingFactory" -Id "841b77c9-d56c-48d1-99a3-8c16c3e77d39" -DownloadLogs -Output "C:\Test"
    ```

## <a name="rerun-failures-in-a-pipeline"></a>Réexécuter des échecs dans un pipeline

> [!IMPORTANT]
> Il est plus facile de résoudre les erreurs et de réexécuter les tranches ayant échoué à l’aide de l’application de surveillance et gestion. Pour en savoir plus sur l’utilisation de l’application, consultez [Surveiller et gérer les pipelines Azure Data Factory à l’aide de l’application de surveillance et gestion](data-factory-monitor-manage-app.md). 

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure
Après avoir débogué et résolu les problèmes dans un pipeline, vous pouvez réexécuter ceux-ci en accédant à la tranche qui pose problème, puis en cliquant sur le bouton **Exécuter** dans la barre de commandes.

![Réexécuter une tranche de données ayant échoué](./media/data-factory-monitor-manage-pipelines/rerun-slice.png)

En cas d’échec de validation de la tranche à cause d’une erreur de stratégie (p. ex. données indisponibles), vous pouvez résoudre le problème et relancer la validation en cliquant sur le bouton **Valider** de la barre de commandes.

![Corriger les erreurs et valider](./media/data-factory-monitor-manage-pipelines/fix-error-and-validate.png)

### <a name="use-azure-powershell"></a>Utilisation d'Azure PowerShell
Vous pouvez exécuter à nouveau des problèmes à l’aide de la cmdlet **Set-AzDataFactorySliceStatus**. Consultez la rubrique [Set-AzDataFactorySliceStatus](/powershell/module/az.datafactory/set-azdatafactoryslicestatus) pour en savoir plus sur la syntaxe et la cmdlet.

**Exemple :**

L’exemple suivant définit l’état de toutes les tranches de la table « DAWikiAggregatedData » sur « En attente » dans la fabrique de données Azure « WikiADF ».

« UpdateType » est défini sur « UpstreamInPipeline ». Concrètement, l’état « En attente » est défini pour chaque tranche de la table et toutes les tables (en amont) dépendantes. Sinon, la valeur « Individuel » pour ce paramètre est également possible.

```powershell
Set-AzDataFactorySliceStatus -ResourceGroupName ADF -DataFactoryName WikiADF -DatasetName DAWikiAggregatedData -Status Waiting -UpdateType UpstreamInPipeline -StartDateTime 2014-05-21T16:00:00 -EndDateTime 2014-05-21T20:00:00
```
## <a name="create-alerts-in-the-azure-portal"></a>Créer des alertes dans le portail Azure

1.  Connectez-vous au portail et sélectionnez **Surveiller -> Alertes** pour ouvrir la page des alertes.

    ![Ouvrez la page Alertes.](media/data-factory-monitor-manage-pipelines/v1alerts-image1.png)

2.  Sélectionnez **+ Nouvelle règle d’alerte** pour créer une nouvelle alerte.

    ![Créer une nouvelle alerte](media/data-factory-monitor-manage-pipelines/v1alerts-image2.png)

3.  Définissez la **condition de l’alerte**. (Veillez à sélectionner **Fabriques de données** dans le champ **Filtrer par type de ressource**.) Vous pouvez également spécifier les valeurs du champ **Dimensions**.

    ![Définir la condition de l’alerte - Sélectionner la cible](media/data-factory-monitor-manage-pipelines/v1alerts-image3.png)

    ![Définir la condition de l’alerte - Ajouter des critères d’alerte](media/data-factory-monitor-manage-pipelines/v1alerts-image4.png)

    ![Définir la condition de l’alerte - Ajouter une logique d’alerte](media/data-factory-monitor-manage-pipelines/v1alerts-image5.png)

4.  Définissez les **détails de l’alerte**.

    ![Définir les détails de l’alerte](media/data-factory-monitor-manage-pipelines/v1alerts-image6.png)

5.  Définissez le **groupe d’actions**.

    ![Définir le groupe d’actions - Créer un nouveau groupe d’actions](media/data-factory-monitor-manage-pipelines/v1alerts-image7.png)

    ![Définir le groupe d’actions - Définir les propriétés](media/data-factory-monitor-manage-pipelines/v1alerts-image8.png)

    ![Définir le groupe d’actions - Nouveau groupe d’actions créé](media/data-factory-monitor-manage-pipelines/v1alerts-image9.png)

## <a name="move-a-data-factory-to-a-different-resource-group-or-subscription"></a>Déplacer une fabrique de données vers un autre groupe de ressources ou abonnement
Vous pouvez déplacer une fabrique de données vers un autre groupe de ressources ou abonnement à l’aide du bouton de la barre de commandes **Déplacer** situé sur la page d’accueil de votre fabrique de données.

![Déplacer la fabrique de données](./media/data-factory-monitor-manage-pipelines/MoveDataFactory.png)

Vous pouvez également déplacer toutes les ressources associées (notamment les alertes associées à la fabrique de données) en même temps que la fabrique de données.

![Boîte de dialogue Déplacer des ressources](./media/data-factory-monitor-manage-pipelines/MoveResources.png)