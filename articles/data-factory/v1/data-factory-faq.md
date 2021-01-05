---
title: Azure Data Factory - Forum aux Questions
description: Forum aux Questions sur Azure Data Factory.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 57f2beef3dc14a47480950bf9e3bdf35a1fd6ce8
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495818"
---
# <a name="azure-data-factory---frequently-asked-questions"></a>Azure Data Factory - Forum aux Questions
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Forum aux questions - Data Factory](../frequently-asked-questions.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="general-questions"></a>Questions générales
### <a name="what-is-azure-data-factory"></a>qu'est-ce qu'Azure Data Factory ?
Data Factory est un service d’intégration de données dans le cloud qui **automatise le déplacement et la transformation des données**. À la manière d’une usine, qui utilise des machines visant à transformer des matières premières en produits manufacturés, Data Factory orchestre des services existants qui collectent des données brutes et les transforment en informations prêtes à l’emploi.

Data Factory vous permet de créer des flux de travail pilotés par les données pour déplacer des données entre des magasins de données locaux et cloud et traiter/transformer des données avec des services de calcul comme Azure HDInsight et Azure Data Lake Analytics. Après avoir créé un pipeline qui exécute l’action dont vous avez besoin, vous pouvez planifier son exécution périodique (toutes les heures, tous les jours, toutes les semaines, etc.).   

Pour plus d’informations, consultez [Vue d’ensemble et concepts clés](data-factory-introduction.md).

### <a name="where-can-i-find-pricing-details-for-azure-data-factory"></a>où puis-je trouver des informations de tarification pour Azure Data Factory ?
Pour plus d’informations sur la tarification d’Azure Data Factory, consultez la [page de tarification de Data Factory][adf-pricing-details].  

### <a name="how-do-i-get-started-with-azure-data-factory"></a>comment prendre en main Azure Data Factory ?
* Pour obtenir une vue d'ensemble d'Azure Data Factory, consultez [Présentation d'Azure Data Factory](data-factory-introduction.md).
* Pour obtenir un tutoriel expliquant comment **copier/déplacer des données** au moyen de l’activité de copie, consultez [Copie de données depuis Stockage Blob Azure vers Azure SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
* Pour obtenir un didacticiel sur la façon de **transformer des données** au moyen de l’activité Hive de HDInsight. Consultez [Traiter des données en exécutant un script Hive sur un cluster Hadoop](data-factory-build-your-first-pipeline.md)

### <a name="what-is-the-data-factorys-region-availability"></a>Dans quelle région sera disponible Data Factory ?
Data Factory est disponible dans les régions **USA Ouest** et **Europe Nord**. Les services de calcul et de stockage utilisés par les fabriques de données peuvent être proposés dans d'autres régions. Consultez [Régions prises en charge](data-factory-introduction.md#supported-regions).

### <a name="what-are-the-limits-on-number-of-data-factoriespipelinesactivitiesdatasets"></a>quelles sont les limites du nombre de fabriques de données/pipelines/activités/jeux de données ?
Consultez la section **Limites d’Azure Data Factory** de l’article [Abonnement Azure et limites, quotas et contraintes du service](../../azure-resource-manager/management/azure-subscription-service-limits.md#data-factory-limits) .

### <a name="what-is-the-authoringdeveloper-experience-with-azure-data-factory-service"></a>Qu’est-il possible de faire avec le service Azure Data Factory en termes de création/développement ?
Vous pouvez créer des fabriques de données en utilisant un des outils/SDK suivants :

* **Visual Studio** : vous pouvez utiliser Visual Studio pour créer une fabrique de données Azure. Pour plus d’informations, consultez [Créer votre premier pipeline de données à l’aide de Visual Studio](data-factory-build-your-first-pipeline-using-vs.md) .
* **Azure PowerShell** : consultez [Créer et surveiller Azure Data Factory à l’aide d’Azure PowerShell](data-factory-build-your-first-pipeline-using-powershell.md) pour obtenir un didacticiel/une procédure pas à pas pour créer une fabrique de données à l’aide de PowerShell. Consultez les [Informations de référence sur les applets de commande Data Factory][adf-powershell-reference] dans la bibliothèque MSDN pour obtenir une documentation complète sur les applets de commande Data Factory.
* **Bibliothèque de classes .NET** Vous pouvez créer par programmation des fabriques de données à l'aide du Kit de développement logiciel (SDK) Data Factory .NET. Consultez [Créer, surveiller et gérer des fabriques de données à l'aide du Kit de développement logiciel (SDK) .NET](data-factory-create-data-factories-programmatically.md) pour découvrir comment créer une fabrique de données à l'aide du Kit de développement logiciel (SDK) .NET. Consultez les [Informations de référence sur la bibliothèque de classes Data Factory][msdn-class-library-reference] pour obtenir une documentation complète sur le Kit de développement logiciel (SDK) Data Factory .NET.
* **API REST** Vous pouvez également utiliser l'API REST exposée par le service Azure Data Factory pour créer et déployer des fabriques de données. Consultez les [Informations de référence sur l’API REST Data Factory][msdn-rest-api-reference] pour obtenir une documentation complète de l’API REST Data Factory.
* **Modèle Azure Resource Manager** Voir [Tutoriel : Concevoir votre première fabrique de données Azure à l’aide du modèle Azure Resource Manager](data-factory-build-your-first-pipeline-using-arm.md) pour plus de détails.

### <a name="can-i-rename-a-data-factory"></a>Puis-je renommer une fabrique de données ?
Non. Tout comme les autres ressources Azure, le nom d'une fabrique de données Azure ne peut pas être modifié.

### <a name="can-i-move-a-data-factory-from-one-azure-subscription-to-another"></a>Puis-je déplacer une fabrique de données d’un abonnement Azure à un autre ?
Oui. Utilisez le bouton **Déplacer** sur le panneau de votre fabrique de données comme indiqué dans le graphique ci-dessous :

![Déplacer la fabrique de données](media/data-factory-faq/move-data-factory.png)

### <a name="what-are-the-compute-environments-supported-by-data-factory"></a>Quels sont les environnements de calcul pris en charge par Data Factory ?
Le tableau suivant fournit une liste d’environnements de calcul pris en charge par Data Factory et les activités qui peuvent s’exécuter sur ces derniers.

| Environnement de calcul | activities |
| --- | --- |
| [Cluster HDInsight à la demande](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) ou [votre propre cluster HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) |[DotNet](data-factory-use-custom-activities.md), [Hive](data-factory-hive-activity.md), [Pig](data-factory-pig-activity.md), [MapReduce](data-factory-map-reduce.md), [Diffusion en continu Hadoop](data-factory-hadoop-streaming-activity.md) |
| [Azure Batch](data-factory-compute-linked-services.md#azure-batch-linked-service) |[DotNet](data-factory-use-custom-activities.md) |
| [Azure Machine Learning Studio (classique)](data-factory-compute-linked-services.md#azure-machine-learning-studio-classic-linked-service) |[Activités Studio (classique) : exécution par lot et ressource de mise à jour](data-factory-azure-ml-batch-execution-activity.md) |
| [Service Analytique Azure Data Lake](data-factory-compute-linked-services.md#azure-data-lake-analytics-linked-service) |[Langage U-SQL du service Analytique Data Lake](data-factory-usql-activity.md) |
| [Azure SQL](data-factory-compute-linked-services.md#azure-sql-linked-service), [Azure Synapse Analytics](data-factory-compute-linked-services.md#azure-synapse-analytics-linked-service), [SQL Server](data-factory-compute-linked-services.md#sql-server-linked-service) |[Procédure stockée](data-factory-stored-proc-activity.md) |

### <a name="how-does-azure-data-factory-compare-with-sql-server-integration-services-ssis"></a>Comparaison d’Azure Data Factory avec SQL Server Integration Services (SSIS) 
Consultez la présentation [Azure Data Factory et SSIS](https://www.sqlbits.com/Sessions/Event15/Azure_Data_Factory_vs_SSIS) faite par l’un de nos MVP (Most Valued Professional), Reza Rad. Certains des changements récents Data Factory peuvent ne pas figurer dans le jeu de diapositives. Nous ajoutons continuellement des fonctionnalités à Azure Data Factory. Nous ajoutons continuellement des fonctionnalités à Azure Data Factory. Nous intégrerons ces mises à jour dans la comparaison des technologies d’intégration de données de Microsoft un peu plus tard cette année.   

## <a name="activities---faq"></a>Activités - Forum Aux Questions
### <a name="what-are-the-different-types-of-activities-you-can-use-in-a-data-factory-pipeline"></a>Quels sont les différents types d’activités que vous pouvez utiliser dans un pipeline Data Factory ?
* [Activités de déplacement des données](data-factory-data-movement-activities.md) pour déplacer les données.
* [Activités de transformation des données](data-factory-data-transformation-activities.md) pour traiter/transformer les données.

### <a name="when-does-an-activity-run"></a>Quand une activité s'exécute-t-elle ?
Les paramètres de configuration de la **disponibilité** présents dans la table de données de sortie déterminent quand l'activité doit être exécutée. Si des jeux de données d’entrée sont spécifiés, l’activité vérifie si toutes les dépendances de données d’entrée sont satisfaites (par exemple, l’état **Prêt** ) avant de s’exécuter.

## <a name="copy-activity---faq"></a>Activité de copie - Forum Aux Questions
### <a name="is-it-better-to-have-a-pipeline-with-multiple-activities-or-a-separate-pipeline-for-each-activity"></a>Est-il préférable d'avoir un pipeline avec plusieurs activités ou un pipeline distinct pour chaque activité ?
Les pipelines sont censés regrouper des activités connexes. Vous pouvez conserver les activités dans un seul pipeline si les tables qui les relient ne sont pas utilisées par d’autres activités extérieures au pipeline. De cette façon, vous n'aurez pas besoin de relier les périodes actives du pipeline pour qu'elles s'accordent les unes avec les autres. En outre, l’intégrité des données dans les tables internes au pipeline est mieux préservée lors de la mise à jour du pipeline. La mise à jour d'un pipeline arrête toutes les activités du pipeline, les supprime et les crée de nouveau. En termes de création, il peut être plus facile de voir le flux de données au sein des activités connexes dans un seul fichier JSON pour le pipeline.

### <a name="what-are-the-supported-data-stores"></a>Quelles sont les banques de données prises en charge ?
L’activité de copie dans Data Factory permet de copier les données d’un magasin de données source vers un magasin de données récepteur. Data Factory prend en charge les magasins de données suivants. Les données de n’importe quelle source peuvent être écrites dans n’importe quel récepteur. Cliquez sur une banque de données pour découvrir comment copier des données depuis/vers cette banque.

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> Les banques de données signalées par un astérisque (*) peuvent être locales ou résider sur une instance Azure IaaS. Elles nécessitent que vous installiez une [passerelle de gestion des données](data-factory-data-management-gateway.md) sur un ordinateur local ou Azure IaaS.

### <a name="what-are-the-supported-file-formats"></a>Quels sont les formats de fichier pris en charge ?
[!INCLUDE [data-factory-file-format](../../../includes/data-factory-file-format.md)]

### <a name="where-is-the-copy-operation-performed"></a>Où est effectuée l’opération de copie ?
Consultez la section relative au [déplacement des données disponibles à l’échelle mondiale](data-factory-data-movement-activities.md#global) pour plus d’informations. En bref, lorsqu’il s’agit d’un magasin de données local, l’opération de copie est effectuée par la passerelle de gestion des données dans votre environnement local. Lorsque le déplacement des données se fait entre deux magasins cloud, l’opération de copie est effectuée dans la région la plus proche de l’emplacement du récepteur dans la même zone géographique.

## <a name="hdinsight-activity---faq"></a>Activité de HDInsight - Forum Aux Questions
### <a name="what-regions-are-supported-by-hdinsight"></a>Quelles sont les régions prises en charge par HDInsight ?
Consultez la page [Tarification relative à HDInsight][hdinsight-supported-regions] ou la section Disponibilité géographique dans l’article suivant.

### <a name="what-region-is-used-by-an-on-demand-hdinsight-cluster"></a>quelle est la région utilisée par un cluster HDInsight à la demande ?
Le cluster HDInsight à la demande est créé dans la même région que le stockage que vous avez spécifié pour être utilisé avec le cluster.    

### <a name="how-to-associate-additional-storage-accounts-to-your-hdinsight-cluster"></a>Comment associer des comptes de stockage supplémentaires à votre cluster HDInsight ?
Si vous utilisez votre propre cluster HDInsight (BYOC, Bring Your Own Cluster), consultez les rubriques suivantes :

* [Utilisation d’un cluster HDInsight avec des comptes de stockage et des metastores secondaires][hdinsight-alternate-storage]
* [Utilisation de comptes de stockage supplémentaires avec Hive HDInsight][hdinsight-alternate-storage-2]

Si vous utilisez un cluster à la demande créé par le service Data Factory, spécifiez les comptes de stockage supplémentaires pour le service lié HDInsight afin que le service Data Factory puisse les inscrire en votre nom. Dans la définition JSON pour le service lié à la demande, utilisez la propriété **additionalLinkedServiceNames** pour spécifier les comptes de stockage secondaires, comme indiqué dans l'extrait de code JSON suivant :

```JSON
{
    "name": "MyHDInsightOnDemandLinkedService",
    "properties":
    {
        "type": "HDInsightOnDemandLinkedService",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "LinkedService-SampleData",
            "additionalLinkedServiceNames": [ "otherLinkedServiceName1", "otherLinkedServiceName2" ]
        }
    }
}
```
Dans l'exemple ci-dessus, otherLinkedServiceName1 et otherLinkedServiceName2 représentent les services liés dont les définitions contiennent des informations d'identification nécessaires au cluster HDInsight pour accéder aux comptes de stockage secondaires.

## <a name="slices---faq"></a>Tranches - Forum Aux Questions
### <a name="why-are-my-input-slices-not-in-ready-state"></a>Pourquoi mes tranches d’entrée ne sont-elles pas à l’état Prêt ?
L’une des erreurs les plus courantes est de ne pas définir la propriété **external** sur **true** dans le jeu de données d’entrée quand les données d’entrée sont externes à la fabrique de données (c’est-à-dire qu’elles n’ont pas été générées par celle-ci).

Dans l’exemple suivant, vous devez uniquement définir **external** sur true pour **dataset1**.  

**DataFactory1** Pipeline 1 : dataset1 -&gt; activity1 -&gt; dataset2 -&gt; activity2 -&gt; dataset3 Pipeline 2 : dataset3-&gt; activity3 -&gt; dataset4

Si vous avez une autre fabrique de données avec un pipeline qui prend dataset4 (généré par le pipeline 2 dans DataFactory1), marquez dataset4 comme un jeu de données externe, car il a été généré par une autre fabrique de données (DataFactory1, et non DataFactory2).  

**DataFactory2**    
Pipeline 1 : dataset4->activity4->dataset5

Si la propriété « external » est définie correctement, vérifiez que les données d’entrée se trouvent bien dans l’emplacement spécifié dans la définition du jeu de données d’entrée.

### <a name="how-to-run-a-slice-at-another-time-than-midnight-when-the-slice-is-being-produced-daily"></a>Comment choisir une heure d’exécution autre que minuit pour une tranche qui est générée quotidiennement ?
Utilisez la propriété **offset** pour spécifier l’heure à laquelle la tranche doit être générée. Pour plus d’informations sur cette propriété, consultez la section [Disponibilité du jeu de données](data-factory-create-datasets.md#dataset-availability) . Voici un exemple rapide :

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```
Les tranches quotidiennes commencent à **6 h** au lieu de minuit, qui est l’heure par défaut.     

### <a name="how-can-i-rerun-a-slice"></a>Comment puis-je réexécuter une tranche ?
Vous pouvez réexécuter une tranche de l'une des manières suivantes :

* Utilisez l’application Surveiller et gérer pour réexécuter une fenêtre d’activité ou une tranche. Pour obtenir des instructions, consultez [Réexécuter les fenêtres d’activité sélectionnées](data-factory-monitor-manage-app.md#perform-batch-actions) .   
* Cliquez sur **Exécuter** dans la barre de commandes du panneau **TRANCHE DE DONNÉES** de la tranche, dans le portail Azure.
* Exécutez l’applet de commande **Set-AzRmDataFactorySliceStatus** en ayant affecté la valeur **En attente** à l’état de la tranche.   

    ```powershell
    Set-AzDataFactorySliceStatus -Status Waiting -ResourceGroupName $ResourceGroup -DataFactoryName $df -TableName $table -StartDateTime "02/26/2015 19:00:00" -EndDateTime "02/26/2015 20:00:00"
    ```
  Consultez [Set-AzDataFactorySliceStatus][set-azure-datafactory-slice-status] pour plus d’informations sur l’applet de commande.

### <a name="how-long-did-it-take-to-process-a-slice"></a>Combien de temps dure le traitement d'une tranche ?
Utilisez l’Explorateur de fenêtres d’activité dans l’application Surveiller et gérer pour connaître la durée du traitement d’une tranche de données. Pour plus d’informations, consultez [Explorateur de fenêtres d’activité](data-factory-monitor-manage-app.md#activity-window-explorer) .

Vous pouvez également faire ce qui suit dans le portail Azure :  

1. Cliquez sur la vignette **Jeux de données** dans le panneau **DATA FACTORY** de votre fabrique de données.
2. Cliquez sur le jeu de données spécifique dans le panneau **Jeux de données** .
3. Sélectionnez la tranche qui vous intéresse dans la liste **Tranches récentes** située dans le panneau **TABLE**.
4. Cliquez sur l'exécution dans la liste **Exécutions d’activité** située dans le panneau **TRANCHE DE DONNÉES**.
5. Cliquez sur la vignette **Propriétés** dans le panneau **DÉTAILS DE L’EXÉCUTION D’ACTIVITÉ**.
6. Une valeur doit apparaître dans le champ **DURÉE** . Il s’agit du temps nécessaire pour traiter la tranche.   

### <a name="how-to-stop-a-running-slice"></a>Comment arrêter une tranche en cours d'exécution ?
Si vous devez arrêter l’exécution du pipeline, vous pouvez utiliser l’applet de commande [Suspend-AzDataFactoryPipeline](/powershell/module/az.datafactory/suspend-azdatafactorypipeline) . Actuellement, l'interruption du pipeline n'arrête pas les exécutions de tranche en cours. Une fois que les exécutions en cours sont terminées, aucune tranche supplémentaire n'est récupérée.

Si vous voulez vraiment arrêter immédiatement toutes les exécutions, le seul moyen est de supprimer le pipeline et de le recréer. Si vous choisissez de supprimer le pipeline, il est INUTILE de supprimer les tables et les services liés qu'il utilise.

[create-factory-using-dotnet-sdk]: data-factory-create-data-factories-programmatically.md
[msdn-class-library-reference]: /dotnet/api/microsoft.azure.management.datafactories.models
[msdn-rest-api-reference]: /rest/api/datafactory/

[adf-powershell-reference]: /powershell/module/az.datafactory/
[azure-portal]: https://portal.azure.com
[set-azure-datafactory-slice-status]: /powershell/module/az.datafactory/set-Azdatafactoryslicestatus

[adf-pricing-details]: https://go.microsoft.com/fwlink/?LinkId=517777
[hdinsight-supported-regions]: https://azure.microsoft.com/pricing/details/hdinsight/
[hdinsight-alternate-storage]: https://social.technet.microsoft.com/wiki/contents/articles/23256.using-an-hdinsight-cluster-with-alternate-storage-accounts-and-metastores.aspx
[hdinsight-alternate-storage-2]: /archive/blogs/cindygross/use-additional-storage-accounts-with-hdinsight-hive