---
title: Guide sur les performances et le réglage de l’activité de copie
description: En savoir plus sur les facteurs clés ayant des répercussions sur les performances du déplacement de données dans Azure Data Factory lorsque vous utilisez l’activité de copie.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 4b9a6a4f-8cf5-4e0a-a06f-8133a2b7bc58
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 05/25/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 5910b94dba03f105197a94cf1ea1805f45249f3f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451345"
---
# <a name="copy-activity-performance-and-tuning-guide"></a>Guide sur les performances et le réglage de l’activité de copie

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-copy-activity-performance.md)
> * [Version 2 (version actuelle)](../copy-activity-performance.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Guide sur les performances et le réglage de l’activité de copie pour Data Factory](../copy-activity-performance.md).

L’activité de copie Azure Data Factory offre une solution de chargement de données de premier ordre, sécurisée, fiable et hautes performances. Elle vous permet de copier des dizaines de téraoctets de données tous les jours entre une grande variété de banques de données cloud et locales. Les performances de chargement des données très rapides constituent la clé pour vous permettre de vous concentrer sur le problème « big data » principal : créer des solutions d’analyse avancées et obtenir des informations approfondies à l’aide de toutes ces données.

Azure fournit un ensemble de solutions d’entrepôt de données et de stockage de données de niveau entreprise, et l’activité de copie offre une expérience de chargement des données hautement optimisée qui est facile à configurer et à mettre en œuvre. Avec une seule activité de copie, vous pouvez obtenir ce qui suit :

* Charger des données dans **Azure Synapse Analytics** à **1,2 Gbit/s**. Consultez [Charger 1 To dans Azure Synapse Analytics en moins de 15 minutes avec Azure Data Factory](data-factory-load-sql-data-warehouse.md) pour obtenir une procédure pas à pas avec un cas d’utilisation.
* Chargement de données dans le **stockage des objets blob Azure** à **1,0 Gbit/s**
* Chargement de données dans **Azure Data Lake Store** à **1,0 Gbit/s**

Cet article aborde les points suivants :

* [chiffres de référence des performances](#performance-reference) pour les magasins de données source et récepteur pris en charge afin de vous aider à planifier votre projet.
* fonctionnalités qui peuvent améliorer le débit de la copie dans différents scénarios, notamment les [unités de déplacement de données cloud](#cloud-data-movement-units), la [copie parallèle](#parallel-copy) et la [copie intermédiaire](#staged-copy) ;
* [conseils de réglage des performances](#performance-tuning-steps) pour optimiser les performances et les facteurs clés qui peuvent affecter les performances de la copie.

> [!NOTE]
> Si vous n’êtes pas familiarisé avec l’activité de copie, voir [Déplacer des données à l’aide de l’activité de copie](data-factory-data-movement-activities.md) avant de lire cet article.
>

## <a name="performance-reference"></a>Performances de référence

À titre de référence, le tableau ci-dessous décrit la valeur de débit de copie en Mbits/s pour les paires de source et de récepteur données à partir de tests internes. À des fins de comparaison, il montre également comment les différents paramètres [d’unités de déplacement des données cloud](#cloud-data-movement-units) ou [d’évolutivité de la passerelle de gestion des données](data-factory-data-management-gateway-high-availability-scalability.md) (plusieurs nœuds de passerelle) peuvent améliorer les performances de copie.

![Matrice des performances](./media/data-factory-copy-activity-performance/CopyPerfRef.png)

>[!IMPORTANT]
>Dans Azure Data Factory version 1, le minimum d’unités de déplacement de données cloud pour la copie de cloud-cloud est de deux. Si ce minimum n’est pas spécifié, consultez les unités de déplacement de données par défaut utilisées dans les [unités de déplacement de données cloud](#cloud-data-movement-units).

**Points à noter :**
* Le débit est calculé à l’aide de la formule suivante : [taille des données lues à partir de la source]/[durée d’exécution de l’activité de copie].
* Les chiffres de référence des performances dans le tableau ont été mesurés à l’aide du jeu de données [TPC-H](http://www.tpc.org/tpch/) durant une seule exécution de l’activité de copie.
* Dans le cas de banques de données Azure, la source et le récepteur sont dans la même région Azure.
* Pour une copie hybride entre des magasins de données en local et dans le cloud, chaque nœud de passerelle était exécuté sur un ordinateur isolé du magasin de données local et présentant les caractéristiques ci-dessous. Lorsqu’une seule activité était exécutée sur la passerelle, l’opération de copie n’a utilisé qu’une petite partie du processeur ou de la mémoire de l’ordinateur de test, ou de la bande passante réseau. Pour en savoir plus, voir [Considérations relatives à la passerelle de gestion des données](#considerations-for-data-management-gateway).
    <table>
    <tr>
        <td>UC</td>
        <td>32 cœurs 2,20 GHz Intel Xeon E5-2660 v2</td>
    </tr>
    <tr>
        <td>Mémoire</td>
        <td>128 Go</td>
    </tr>
    <tr>
        <td>Réseau</td>
        <td>Interface Internet : 10 Gbits/s ; interface intranet : 40 Gbits/s</td>
    </tr>
    </table>


> [!TIP]
> Vous pouvez obtenir un débit plus élevé en exploitant davantage d’unités de déplacement des données (DMU) que le nombre maximum par défaut, à savoir 32, pour l’exécution d’une activité de copie de cloud à cloud. Par exemple, avec 100 unités DMU, vous pouvez copier des données depuis Azure Blob vers Azure Data Lake Store à **1,0 Go/s**. Pour en savoir plus sur cette fonctionnalité et le scénario pris en charge, consultez la section [Unités de déplacement de données cloud](#cloud-data-movement-units). Contactez le [support Azure](https://azure.microsoft.com/support/) pour demander plus de DMU.

## <a name="parallel-copy"></a>Copie en parallèle
Vous pouvez lire les données de la source ou écrire des données sur la destination **en parallèle dans une exécution de l’activité de copie**. Cette fonctionnalité améliore le débit d’une opération de copie et réduit le temps nécessaire pour déplacer des données.

Ce paramètre est différent de la propriété **concurrency** au niveau de la définition d’activité. La propriété **concurrency** détermine le nombre **d’exécutions d’activité de copie simultanées** pour traiter les données de différentes fenêtres d’activité (de 1h00 à 2h00, de 2h00 à 3h00, de 3h00 à 4h00, etc.). Cette fonctionnalité est utile lorsque vous effectuez une charge historique. La fonctionnalité de copie en parallèle s’applique à une **exécution d’activité unique**.

Examinons un exemple de scénario. Dans l’exemple suivant, plusieurs tranches dans le passé doivent être traitées. Data Factory exécute une instance de l’activité de copie (une exécution d’activité) pour chaque tranche :

* La tranche de données de la première fenêtre d’activité (de 1h00 à 2h00) == > exécution d’activité 1
* La tranche de données de la deuxième fenêtre d’activité (de 2h00 à 3h00) == > exécution d’activité 2
* La tranche de données de la deuxième fenêtre d’activité (de 3h00 à 4h00) == > exécution d’activité 3

Et ainsi de suite.

Dans cet exemple, lorsque la valeur **concurrency** est définie sur 2, **l’exécution d’activité 1** et **l’exécution d’activité 2** copient les données de deux fenêtres d’activité **de façon simultanée** pour améliorer les performances de déplacement de données. Toutefois, si plusieurs fichiers sont associés à l’exécution d’activité 1, le service de déplacement de données copie les fichiers de la source vers la destination, un fichier à la fois.

### <a name="cloud-data-movement-units"></a>Unités de déplacement de données cloud
Une **unité de déplacement de données cloud** est une mesure qui représente la puissance (combinaison de l’allocation de ressources de processeur, de mémoire et de réseau) d’une seule unité dans Data Factory. Une unité de déplacement de données cloud est applicable dans une opération de copie cloud-cloud, mais pas dans une copie hybride.

**Le minimum d’unités de déplacement de données cloud pour exécuter l’activité de copie est de deux.** Si aucun minimum n’est spécifié, le tableau suivant répertorie les unités de déplacement de données par défaut utilisées dans différents scénarios de copie :

| Scénario de copie | Unités de déplacement de données cloud par défaut déterminées par le service |
|:--- |:--- |
| Copie de données entre des magasins basés sur des fichiers | Entre 4 et 16 selon le nombre et la taille des fichiers. |
| Tous les autres scénarios de copie | 4 |

Pour remplacer cette valeur par défaut, spécifiez une valeur pour la propriété **cloudDataMovementUnits** comme suit. Les **valeurs autorisées** pour la propriété **cloudDataMovementUnits** sont les suivantes : 2, 4, 8, 16, 32. Le **nombre réel d’unités de déplacement de données cloud** que l’opération de copie utilise au moment de l’exécution est égal ou inférieur à la valeur configurée, en fonction de votre modèle de données. Pour plus d’informations sur le niveau de gain de performances que vous pouvez obtenir lorsque vous configurez plusieurs unités pour une source et un récepteur de copie spécifiques, voir [Performances de référence](#performance-reference).

```json
"activities":[
    {
        "name": "Sample copy activity",
        "description": "",
        "type": "Copy",
        "inputs": [{ "name": "InputDataset" }],
        "outputs": [{ "name": "OutputDataset" }],
        "typeProperties": {
            "source": {
                "type": "BlobSource",
            },
            "sink": {
                "type": "AzureDataLakeStoreSink"
            },
            "cloudDataMovementUnits": 32
        }
    }
]
```

> [!NOTE]
> Si vous avez besoin de plus d’unités de déplacement de données cloud pour un débit plus élevé, contactez le [support Azure](https://azure.microsoft.com/support/). Actuellement, un paramètre de 8 et plus fonctionne uniquement lorsque vous **copiez plusieurs fichiers à partir d’un stockage blob, d’une instance Data Lake Store, d’Amazon S3, d’un FTP cloud, d’un SFTP cloud vers un stockage blob, une instance Data Lake Store ou Azure SQL Database**.
>

### <a name="parallelcopies"></a>parallelCopies
Vous pouvez utiliser la propriété **parallelCopies** pour indiquer le parallélisme que vous voulez que l’activité de copie utilise. Vous pouvez penser cette propriété comme le nombre maximum de threads dans une activité de copie qui peut lire dans votre source ou écrire dans vos banques de données réceptrices en parallèle.

Pour chaque exécution d’activité de copie, Data Factory détermine le nombre de copies en parallèle à utiliser pour copier les données depuis la banque de données source et vers la banque de données de destination. Le nombre de copies en parallèle par défaut qu’il utilise dépend du type de source et de récepteur que vous utilisez.

| Source et récepteur | Nombre de copie en parallèle par défaut déterminé par le service |
| --- | --- |
| Copie de données entre les magasins basés sur fichier (stockage des objets blob ; Data Lake Store ; Amazon S3 ; un système de fichiers local ; un HDFS local) |Entre 1 et 32. Dépend de la taille des fichiers et du nombre d’unités de déplacement de données cloud utilisées pour copier des données entre les deux banques de données cloud, ou de la configuration physique de l’ordinateur de passerelle utilisé pour une copie hybride (pour copier des données vers ou à partir d’une banque de données locale). |
| Copier des données de **n’importe quelle banque de données source vers le stockage Table Azure** |4 |
| Toutes les autres paires de source et de récepteur |1 |

Généralement, le comportement par défaut doit offrir le meilleur débit. Or, pour contrôler la charge sur les ordinateurs qui hébergent vos banques de données ou pour optimiser les performances de copie, vous pouvez choisir de remplacer la valeur par défaut et spécifier une valeur pour la propriété **parallelCopies** . La valeur doit être comprise entre 1 et 32 (inclus). Au moment de l’exécution, pour des performances optimales, l’activité de copie utilise une valeur inférieure ou égale à la valeur que vous avez définie.

```json
"activities":[
    {
        "name": "Sample copy activity",
        "description": "",
        "type": "Copy",
        "inputs": [{ "name": "InputDataset" }],
        "outputs": [{ "name": "OutputDataset" }],
        "typeProperties": {
            "source": {
                "type": "BlobSource",
            },
            "sink": {
                "type": "AzureDataLakeStoreSink"
            },
            "parallelCopies": 8
        }
    }
]
```
Points à noter :

* Lorsque vous copiez des données entre des banques basées sur fichier, la valeur **parallelCopies** détermine le parallélisme au niveau du fichier. La segmentation dans un seul fichier se produit en arrière-plan de manière automatique et transparente. Elle est conçue pour utiliser la meilleure taille de segment appropriée pour un type de magasin de données source donné pour charger des données de manière parallèle et orthogonale à la valeur parallelCopies. Le nombre réel de copies en parallèle que le service de déplacement de données utilise pour l’opération de copie au moment de l’exécution ne peut pas être supérieur au nombre de fichiers dont vous disposez. Si le comportement de copie est défini sur **mergeFile**, l’activité de copie ne peut pas tirer parti du parallélisme au niveau du fichier.
* Lorsque vous spécifiez une valeur pour la propriété **parallelCopies** , songez à l’augmentation de la charge sur vos banques de données sources et réceptrices et sur la passerelle s’il s’agit d’une copie hybride. Cela se produit en particulier lorsque plusieurs activités ou exécutions simultanées des mêmes activités ont lieu en même temps dans la même banque de données. Si vous remarquez que la banque de données ou la passerelle est submergée par la charge, diminuez la valeur **parallelCopies** pour alléger la charge.
* Lorsque vous copiez des données à partir de banques qui ne sont pas basées sur fichier vers des banques basées sur fichier, le service de déplacement de données ignore la propriété **parallelCopies** . Même si le parallélisme est spécifié, il n’est pas appliqué dans ce cas.

> [!NOTE]
> Vous devez utiliser la passerelle de gestion des données 1.11 ou une version ultérieure pour utiliser la fonctionnalité **parallelCopies** lorsque vous effectuez une copie hybride.
>
>

Pour mieux utiliser ces deux propriétés et pour améliorer votre débit de déplacement de données, voir les exemples de cas d’utilisation. Vous n’avez pas besoin de configurer **parallelCopies** pour tirer parti du comportement par défaut. Si vous configurez **parallelCopies** et que la valeur est trop basse, plusieurs unités de déplacement de données cloud ne peuvent pas être pleinement utilisées.

### <a name="billing-impact"></a>Impact sur la facturation
Il est **important** de garder à l’esprit que vous êtes facturé selon la durée totale de l’opération de copie. Si un travail de copie prenait auparavant une heure avec une seule unité cloud et qu’il prend maintenant 15 minutes avec quatre unités cloud, le montant total de la facture reste identique. Par exemple, vous utilisez quatre unités cloud. La première unité cloud dépense 10 minutes, la deuxième, 10 minutes, la troisième, 5 minutes et la quatrième, 5 minutes, le tout dans une seule exécution d’activité de copie. Vous êtes facturé pour la durée (déplacement de données) de la copie totale, qui est de 10 + 10 + 5 + 5 = 30 minutes. L’utilisation de **parallelCopies** n’affecte pas la facturation.

## <a name="staged-copy"></a>copie intermédiaire
Lorsque vous copiez des données entre une banque de données source et une banque de données réceptrice, vous pouvez choisir d’utiliser le stockage Blob comme banque intermédiaire. La fonctionnalité intermédiaire est particulièrement utile dans les cas suivants :

1. **Vous voulez ingérer des données à partir de divers magasins de données dans Azure Synapse Analytics via PolyBase.** Azure Synapse Analytics utilise PolyBase comme un mécanisme à haut débit pour charger des données volumineuses dans Azure Synapse Analytics. Toutefois, la source de données doit se trouver dans le stockage Blob, et se conformer à des critères supplémentaires. Lorsque vous chargez des données à partir d’une banque de données autre que le stockage Blob, vous pouvez activer la copie de données via un stockage Blob intermédiaire. Dans ce cas, Data Factory effectue les transformations de données requises pour garantir la conformité vis-à-vis des exigences de PolyBase. Ensuite, il utilise PolyBase pour charger les données dans Azure Synapse Analytics. Pour plus d’informations, consultez [Utiliser PolyBase pour charger des données dans Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#use-polybase-to-load-data-into-azure-synapse-analytics). Consultez [Charger 1 To dans Azure Synapse Analytics en moins de 15 minutes avec Azure Data Factory](data-factory-load-sql-data-warehouse.md) pour obtenir une procédure pas à pas avec un cas d’utilisation.
2. **Il peut être assez long parfois d’effectuer des déplacements de données hybrides (c’est-à-dire, de copier entre une banque de données locale et une banque de données cloud) sur une connexion réseau lente**. Pour améliorer les performances, vous pouvez compresser les données locales afin de réduire le temps nécessaire pour déplacer des données vers la banque de données intermédiaire dans le cloud. Ensuite, vous pouvez décompresser les données dans la banque intermédiaire avant de les charger dans la banque de données de destination.
3. **Vous ne souhaitez pas ouvrir les ports autres que le port 80 et le port 443 dans votre pare-feu, en raison des stratégies informatiques d’entreprise**. Par exemple, lorsque vous copiez des données d’un magasin de données local vers un récepteur Azure SQL Database ou un récepteur Azure Synapse Analytics, vous devez activer les communications TCP sortantes sur le port 1433 pour le pare-feu Windows et votre pare-feu d’entreprise. Dans ce scénario, tirez parti de la passerelle pour commencer par copier les données dans une instance de stockage blob intermédiaire via HTTP ou HTTPS sur le port 443. Ensuite, chargez les données dans SQL Database ou Azure Synapse Analytics à partir du stockage d’objet blob intermédiaire. Dans ce flux, vous n’avez pas besoin d’activer le port 1433.

### <a name="how-staged-copy-works"></a>Fonctionnement de la copie intermédiaire
Lorsque vous activez la fonctionnalité intermédiaire, tout d’abord les données sont copiées à partir de la banque de données source vers la banque de données intermédiaire (indiquez la vôtre). Ensuite, les données sont copiées à partir de la banque de données intermédiaire dans la banque de données de réceptrice. Data Factory gère automatiquement le flux à deux étapes pour vous. Data Factory nettoie également les données temporaires du stockage intermédiaire une fois le déplacement de données terminé.

Dans le scénario de copie cloud (les banques de données sources et réceptrices de données sont dans le cloud), la passerelle n’est pas utilisée. Le service Data Factory effectue les opérations de copie.

![Copie intermédiaire : scénario cloud](media/data-factory-copy-activity-performance/staged-copy-cloud-scenario.png)

Dans le scénario de copie hybride, (la source existe en local et le récepteur dans le cloud), la passerelle déplace les données de la banque de données source vers une banque de données intermédiaire. Le service Data Factory déplace les données de la banque de données intermédiaire vers la banque de données réceptrice. La copie de données à partir d’une banque de données cloud vers une banque de données locale à l’aide d’une banque intermédiaire est également prise en charge avec un flux inversé.

![Copie intermédiaire : scénario hybride](media/data-factory-copy-activity-performance/staged-copy-hybrid-scenario.png)

Lorsque vous activez le déplacement de données à l’aide d’une banque de données intermédiaire, vous pouvez indiquer si vous souhaitez compresser les données avant de les déplacer de la banque de données source vers une banque de données intermédiaire, et les décompresser avant leur transfert d’une banque de données intermédiaire vers une banque de données réceptrice.

Actuellement, vous ne pouvez pas copier de données entre deux banques de données locales à l’aide d’une banque intermédiaire. Cette option sera bientôt disponible.

### <a name="configuration"></a>Configuration
Configurez le paramètre **enableStaging** sur l’activité de copie pour spécifier si vous souhaitez que les données soient placées dans un stockage blob intermédiaire avant d’être chargées dans une banque de données de destination. Lorsque vous définissez **enableStaging** sur TRUE, spécifiez les propriétés supplémentaires répertoriées dans le tableau suivant. Si vous n’en avez pas, vous devez créer un stockage Azure ou un service lié à la signature d’accès partagé du stockage pour le stockage intermédiaire.

| Propriété | Description | Valeur par défaut | Obligatoire |
| --- | --- | --- | --- |
| **enableStaging** |Indiquez si vous souhaitez copier les données via un magasin de données intermédiaire. |False |Non |
| **linkedServiceName** |Spécifiez le nom d’un service lié [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service) ou [AzureStorageSas](data-factory-azure-blob-connector.md#azure-storage-sas-linked-service) faisant référence à l’instance de stockage que vous utilisez comme banque de données intermédiaire. <br/><br/> Vous ne pouvez pas utiliser le stockage avec une signature d’accès partagé pour charger les données dans Azure Synapse Analytics via PolyBase. Vous pouvez l’utiliser dans tous les autres scénarios. |N/A |Oui, quand **enableStaging** est défini sur TRUE |
| **path** |Spécifiez le chemin du stockage Blob où vous souhaitez placer les données intermédiaires. Si vous ne renseignez pas le chemin d’accès, le service crée un conteneur pour stocker les données temporaires. <br/><br/> Ne spécifiez un chemin d’accès que si vous utilisez le stockage avec une signature d’accès partagé, ou si vous avez besoin de données temporaires dans un emplacement spécifique. |N/A |Non |
| **enableCompression** |Spécifie si les données doivent être compressées avant d’être copiées vers la destination. Ce paramètre réduit le volume de données transférées. |False |Non |

Voici un exemple de définition de l’activité de copie avec les propriétés qui sont décrites dans le tableau précédent :

```json
"activities":[
{
    "name": "Sample copy activity",
    "type": "Copy",
    "inputs": [{ "name": "OnpremisesSQLServerInput" }],
    "outputs": [{ "name": "AzureSQLDBOutput" }],
    "typeProperties": {
        "source": {
            "type": "SqlSource",
        },
        "sink": {
            "type": "SqlSink"
        },
        "enableStaging": true,
        "stagingSettings": {
            "linkedServiceName": "MyStagingBlob",
            "path": "stagingcontainer/path",
            "enableCompression": true
        }
    }
}
]
```

### <a name="billing-impact"></a>Impact sur la facturation
Vous êtes facturé en fonction de deux étapes : la durée de la copie et le type de copie.

* Lorsque vous utilisez la copie intermédiaire lors d’une copie dans le cloud (copie de données à partir d’une banque de données cloud vers une autre banque de données cloud), vous êtes facturé au prix de [somme de la durée de copie pour les étapes 1 et 2] x [prix unitaire de la copie dans le cloud].
* Lorsque vous utilisez la copie intermédiaire lors d’une copie hybride (copie de données à partir d’une banque de données locale vers une banque de données cloud), vous êtes facturé au prix de [durée de la copie hybride] x [prix unitaire de la copie hybride] + [durée de la copie cloud] x [prix unitaire de la copie cloud].

## <a name="performance-tuning-steps"></a>Procédure de réglage des performances
Nous vous recommandons d’effectuer cette procédure pour régler les performances de votre service Data Factory avec l’activité de copie :

1. **Établir une ligne de base**. Pendant la phase de développement, testez votre pipeline en utilisant l’activité de copie sur un échantillon de données représentatif. Vous pouvez utiliser le [modèle de découpage](data-factory-scheduling-and-execution.md) Data Factory pour limiter la quantité de données que vous utilisez.

   Collectez les temps d’exécution et les caractéristiques de performances à l’aide de **l’application de surveillance et gestion**. Choisissez **Surveiller et gérer** sur votre page d’accueil Data Factory. Dans l’arborescence, sélectionnez le **jeu de données de sortie**. Dans la liste des **fenêtres d’activité** , sélectionnez l’exécution de l’activité de copie. **fenêtres d’activité** répertorient la durée de l’activité de copie et la taille des données qui sont copiées. Le débit est répertorié dans **l’Explorateur de fenêtres d’activité**. Pour en savoir plus sur l’application, consultez [Surveiller et gérer les pipelines Azure Data Factory à l’aide de l’application de surveillance et gestion](data-factory-monitor-manage-app.md).

   ![Détails de l'exécution d'activité](./media/data-factory-copy-activity-performance/mmapp-activity-run-details.png)

   Plus loin dans cet article, vous pouvez comparer les performances et la configuration de votre scénario aux [performances de référence](#performance-reference) de l’activité de copie de nos tests.
2. **Diagnostiquer et optimiser les performances**. Si les performances que vous observez ne répondent pas à vos attentes, vous devez identifier les goulots d’étranglement. Ensuite, optimisez les performances pour supprimer ou réduire l’effet des goulots d’étranglement. Une description complète du diagnostic des performances dépasserait la portée de cet article, mais voici quelques considérations d’ordre général :

   * Fonctionnalités de performances :
     * [Copie en parallèle](#parallel-copy)
     * [Unités de déplacement de données cloud](#cloud-data-movement-units)
     * [Copie intermédiaire](#staged-copy)
     * [Évolutivité de la passerelle de gestion des données](data-factory-data-management-gateway-high-availability-scalability.md)
   * [Passerelle de gestion de données](#considerations-for-data-management-gateway)
   * [Source](#considerations-for-the-source)
   * [Section sink](#considerations-for-the-sink)
   * [Sérialisation et désérialisation](#considerations-for-serialization-and-deserialization)
   * [Compression](#considerations-for-compression)
   * [Mappage de colonnes](#considerations-for-column-mapping)
   * [Autres points à considérer](#other-considerations)
3. **Étendez la configuration à l’ensemble de votre jeu de données**. Lorsque vous êtes satisfait des résultats et des performances de l’exécution, vous pouvez étendre la définition et la période active du pipeline pour couvrir l’ensemble de votre jeu de données.

## <a name="considerations-for-data-management-gateway"></a>Considérations relatives à la passerelle de gestion des données
**Configuration de la passerelle** : nous vous recommandons d’utiliser un ordinateur dédié pour héberger la passerelle de gestion des données. Voir [Considérations relatives à l’utilisation de la passerelle de gestion des données](data-factory-data-management-gateway.md#considerations-for-using-gateway).

**Supervision de la passerelle et scale-up/out** : une seule passerelle logique comportant un ou plusieurs nœuds de passerelle peut traiter simultanément plusieurs exécutions de l’activité de copie. Vous pouvez afficher un instantané en temps quasi réel de l’utilisation des ressources (processeur, mémoire, réseau (entrant/sortant), etc.) sur un ordinateur de passerelle, et voir le nombre de travaux simultanés en cours d’exécution par rapport à la limite dans le portail Azure. Voir [Surveillance de la passerelle dans le portail](data-factory-data-management-gateway.md#monitor-gateway-in-the-portal). Si vous avez besoin de déplacer des données hybrides avec de nombreuses exécutions d’activité de copie simultanées ou avec un gros volume de données à copier, vous devriez envisager [d’effectuer un scale-up ou un scale-out de la passerelle](data-factory-data-management-gateway-high-availability-scalability.md#scale-considerations) afin de mieux utiliser vos ressources ou de configurer plus de ressources pour répondre à vos besoins de copie.

## <a name="considerations-for-the-source"></a>Considérations relatives à la source
### <a name="general"></a>Général
Vérifiez que la banque de données sous-jacente n’est pas submergée par d’autres charges de travail s’exécutant dessus ou s’y rapportant.

Pour les banques de données Microsoft, consultez les [rubriques relatives à la surveillance et au réglage](#performance-reference) spécifiques aux banques de données, qui peuvent vous aider à comprendre les caractéristiques de performances de celles-ci, à réduire les temps de réponse et à maximiser le débit.

Si vous copiez des données depuis le stockage d’objets blob vers Azure Synapse Analytics, envisagez d’utiliser **PolyBase** pour améliorer les performances. Pour plus d’informations, consultez [Utiliser PolyBase pour charger des données dans Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#use-polybase-to-load-data-into-azure-synapse-analytics). Consultez [Charger 1 To dans Azure Synapse Analytics en moins de 15 minutes avec Azure Data Factory](data-factory-load-sql-data-warehouse.md) pour obtenir une procédure pas à pas avec un cas d’utilisation.

### <a name="file-based-data-stores"></a>Magasins de données basés sur un fichier
*(Inclut le stockage des objets blob, Data Lake Store, Amazon S3, les systèmes de fichiers locaux et HDFS locaux)*

* **Taille et quantité moyennes de fichiers**: l’activité de copie transfère les données un fichier à la fois. Pour une même quantité de données à déplacer, le débit global est plus lent si les données se composent de beaucoup de petits fichiers plutôt que de quelques fichiers volumineux, en raison de la phase d’amorçage nécessaire pour chaque fichier. Par conséquent, vous devez autant que possible combiner plusieurs petits fichiers en fichiers plus volumineux pour augmenter le débit.
* **Format de fichier et compression** : pour d’autres méthodes permettant d’améliorer les performances, voir les sections [Considérations relatives à la sérialisation et à la désérialisation](#considerations-for-serialization-and-deserialization) et [Considérations relatives à la compression](#considerations-for-compression).
* Pour le scénario de **système de fichiers local** où l’utilisation d’une **passerelle de gestion des données** est requise, voir la section [Considérations relatives à la passerelle de gestion des données](#considerations-for-data-management-gateway).

### <a name="relational-data-stores"></a>Bases de données relationnelles
*(Inclut SQL Database, Azure Synapse Analytics, Amazon Redshift, les bases de données SQL Server et les bases de données Oracle, MySQL, DB2, Teradata, Sybase et PostgreSQL, etc.)*

* **Modèle de données** : Votre schéma de table a des répercussions sur le débit de copie. Une taille de ligne importante vous donne de meilleures performances qu’une petite taille de ligne pour copier la même quantité de données. Raison : la base de données peut récupérer plus efficacement moins de lots de données contenant moins de lignes.
* **Requête ou procédure stockée** : optimisez la logique de la requête ou de la procédure stockée que vous spécifiez dans la source d’activité de copie pour extraire les données plus efficacement.
* Pour des **bases de données relationnelles locales**, telles que SQL Server et Oracle, qui requièrent l’utilisation d’une **passerelle de gestion des données**, voir la section Considérations relatives à la passerelle de gestion des données.

## <a name="considerations-for-the-sink"></a>Considérations relatives au récepteur
### <a name="general"></a>Général
Vérifiez que la banque de données sous-jacente n’est pas submergée par d’autres charges de travail s’exécutant dessus ou s’y rapportant.

Pour les banques de données Microsoft, voir les [rubriques relatives à la surveillance et au réglage](#performance-reference) qui sont spécifiques aux banques de données. Ces rubriques peuvent vous aider à comprendre les caractéristiques des performances des banques de données et à déterminer comment réduire les temps de réponse et optimiser le débit.

Si vous copiez des données depuis le **stockage d’objets blob** vers **Azure Synapse Analytics**, envisagez d’utiliser **PolyBase** pour améliorer les performances. Pour plus d’informations, consultez [Utiliser PolyBase pour charger des données dans Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#use-polybase-to-load-data-into-azure-synapse-analytics). Consultez [Charger 1 To dans Azure Synapse Analytics en moins de 15 minutes avec Azure Data Factory](data-factory-load-sql-data-warehouse.md) pour obtenir une procédure pas à pas avec un cas d’utilisation.

### <a name="file-based-data-stores"></a>Magasins de données basés sur un fichier
*(Inclut le stockage des objets blob, Data Lake Store, Amazon S3, les systèmes de fichiers locaux et HDFS locaux)*

* **Comportement de copie**: si vous copiez des données d’une banque de données basée sur fichier différente, l’activité de copie propose trois options via la propriété **copyBehavior**. Elle conserve la hiérarchie, aplatit la hiérarchie ou fusionne les fichiers. La conservation ou l’aplanissement de la hiérarchie entraîne peu ou pas de surcharge de performances, mais la fusion de fichiers entraîne une augmentation de la surcharge des performances.
* **Format de fichier et compression** : voir les sections [Considérations relatives à la sérialisation et à la désérialisation](#considerations-for-serialization-and-deserialization) et [Considérations relatives à la compression](#considerations-for-compression) pour d’autres méthodes permettant d’améliorer les performances.
* **Stockage Blob** : actuellement, le Stockage Blob ne prend en charge que les objets blob de blocs pour un transfert de données et un débit optimisés.
* Pour les scénarios de **systèmes de fichiers locaux** nécessitant l’utilisation d’une **passerelle de gestion des données**, voir la section [Considérations relatives à la passerelle de gestion des données](#considerations-for-data-management-gateway).

### <a name="relational-data-stores"></a>Bases de données relationnelles
*(Inclut SQL Database, Azure Synapse Analytics, les bases de données SQL Server et les bases de données Oracle)*

* **Comportement de copie**: selon les propriétés que vous avez configurées pour **sqlSink**, l’activité de copie écrit des données dans la base de données de destination de différentes façons.
  * Par défaut, le service de déplacement de données utilise une API de copie en bloc pour insérer des données en mode Append, ce qui optimise les performances.
  * Si vous configurez une procédure stockée dans le récepteur, la base de données applique les données une ligne à la fois plutôt que de les charger en bloc. Les performances chutent considérablement. Si votre jeu de données est volumineux, le cas échéant, songez à utiliser la propriété **sqlWriterCleanupScript** .
  * Si vous configurez la propriété **sqlWriterCleanupScript** pour chaque exécution de l’activité de copie, le service déclenche le script, puis vous utilisez l’API de copie en bloc pour insérer les données. Par exemple, pour remplacer les données de la table entière par les dernières données, vous pouvez spécifier un script pour commencer par supprimer tous les enregistrements avant de charger en bloc les nouvelles données à partir de la source.
* **Modèle de données et taille de lot**:
  * Votre schéma de table a des répercussions sur le débit de copie. Pour copier une même quantité de données, une taille de ligne importante produit de meilleures performances qu’une petite taille, car la base de données peut valider plus efficacement moins de lots de données.
  * L’activité de copie insère des données dans une série de lots. Vous pouvez définir le nombre de lignes dans un lot à l’aide de la propriété **writeBatchSize** . Si vos données comportent des lignes de petite taille, vous pouvez définir la propriété **writeBatchSize** sur une valeur plus élevée pour réduire la surcharge de traitement par lots et augmenter le débit. Si la taille de ligne de vos données est importante, soyez prudent lorsque vous augmentez la valeur **writeBatchSize**. Une valeur élevée peut entraîner un échec de la copie en raison de la surcharge de la base de données.
* Pour des **bases de données relationnelles locales**, comme SQL Server et Oracle, qui requièrent l’utilisation d’une **passerelle de gestion des données**, voir la section [Considérations relatives à la passerelle de gestion des données](#considerations-for-data-management-gateway).

### <a name="nosql-stores"></a>Magasins NoSQL
*(Inclut le Stockage Table et Azure Cosmos DB)*

* Pour le **Stockage Table**:
  * **Partition** : l’écriture de données en partitions entrelacées dégrade considérablement les performances. Classez vos données sources par clé de partition afin qu’elles soient insérées efficacement partition après partition, ou ajustez la logique pour écrire les données dans une seule partition.
* Pour **Azure Cosmos DB** :
  * **Taille du lot** : la propriété **writeBatchSize** définit le nombre de requêtes parallèles adressées au service Azure Cosmos DB pour la création de documents. Vous pouvez vous attendre à de meilleures performances lorsque vous augmentez la valeur **writeBatchSize** , car davantage de demandes parallèles sont envoyées à Azure Cosmos DB. Toutefois, regardez la limitation lorsque vous écrivez sur Azure Cosmos DB (le message d’erreur est « Le taux de demandes est élevé »). Différents facteurs peuvent entraîner la limitation, notamment la taille des documents, le nombre de termes dans les documents et la stratégie d’indexation de la collection cible. Pour obtenir un débit de copie plus élevé, songez à utiliser une meilleure collection, par exemple, S3.

## <a name="considerations-for-serialization-and-deserialization"></a>Considérations relatives à la sérialisation et à la désérialisation
Une sérialisation et une désérialisation peuvent survenir quand le jeu de données d’entrée ou de sortie est un fichier. Consultez la page [Formats de fichier et de compression pris en charge](data-factory-supported-file-and-compression-formats.md) qui présente des détails sur les formats de fichier pris en charge par activité de copie.

**Comportement de copie**:

* Copie de fichiers entre banques de données basées sur fichier :
  * Lorsque les jeux de données d’entrée et de sortie ont les mêmes paramètres de format de fichier ou n’en ont aucun, le service de déplacement de données exécute une copie binaire sans sérialisation ou désérialisation. Vous voyez un débit plus élevé par rapport au scénario, dans lequel les paramètres de format de fichier source et récepteur sont différents.
  * Lorsque les jeux de données d’entrée et de sortie sont au format texte et que seul le type d’encodage est différent, le service de déplacement de données n’effectue que la conversion de l’encodage. Il n’effectue aucune sérialisation ni désérialisation, ce qui entraîne une surcharge de performances par rapport à la copie binaire.
  * Lorsque les jeux de données d’entrée et de sortie diffèrent par leur format de fichier ou leur configuration, par exemple au niveau des séparateurs, le service de déplacement de données désérialise les données sources en flux, les transforme, puis les sérialise au format de sortie que vous avez indiqué. Cette opération donne lieu à une surcharge des performances beaucoup plus importante par rapport à d’autres scénarios.
* Lorsque vous copiez des fichiers vers/à partir d’une banque de données qui n’est pas basée sur fichier (par exemple, d’une banque basée sur fichier vers une banque relationnelle), l’étape de sérialisation ou de désérialisation est requise. Cette étape génère une surcharge significative des performances.

**Format de fichier** : le format de fichier que vous choisissez peut affecter les performances de copie. Par exemple, Avro est un format binaire compact qui stocke des métadonnées avec des données. Il bénéficie d’une prise en charge étendue dans l’écosystème Hadoop pour le traitement et l’interrogation. En revanche, le format Avro étant plus coûteux en sérialisation et désérialisation, le débit de copie est inférieur par rapport au format texte. Choisissez votre format de fichier dans le flux de traitement de manière holistique. Commencez par la forme sous laquelle les données sont stockées dans les banques de données sources ou doivent être extraites de systèmes externes ; le meilleur format pour le stockage, le traitement analytique et les requêtes, et le format dans lequel les données doivent être exportées dans des mini-data warehouses pour les outils de rapport et de visualisation. Parfois, un format de fichier non optimal pour les performances en lecture et écriture peut constituer une option idéale pour le processus analytique général.

## <a name="considerations-for-compression"></a>Considérations relatives à la compression
Lorsque votre jeu de données d’entrée ou de sortie est un fichier, vous pouvez définir l’activité de copie pour qu’elle effectue une compression ou une décompression lors de l’écriture de données dans la destination. Lorsque vous choisissez la compression, vous trouvez un compromis entre les entrées/sorties (E/S) et le processeur. La compression des données a un coût supplémentaire en ressources de calcul. Toutefois, en retour, elle réduit les E/S réseau et le stockage. En fonction de vos données, vous pouvez constater une augmentation du débit global de copie.

**Codec** : l’activité de copie prend en charge les types de compression gzip, bzip2 et Deflate. Azure HDInsight peut utiliser les trois types de traitement. Chaque codec de compression présente des avantages. Par exemple, bzip2 a le plus faible débit de copie, mais vous obtenez les meilleures performances de requêtes Hive avec bzip2, car vous pouvez le fractionner pour traitement. Gzip est l’option la plus équilibrée, et la plus souvent utilisée. Choisissez le codec le plus approprié pour votre scénario de bout en bout.

**Niveau** : vous avec le choix entre deux options pour chaque codec de compression, la compression la plus rapide et la compression optimale. L’option de compression la plus rapide compresse les données aussi rapidement que possible, même si le fichier résultant de l’opération n’est pas compressé de façon optimale. L’option de compression optimale nécessite plus de temps, et produit une quantité de données minimale. Vous pouvez tester les deux options pour voir celle qui offre les meilleures performances globales dans votre cas.

**Élément à prendre en compte** : pour copier une grande quantité de données entre une banque locale et le cloud, utilisez le stockage Blob intermédiaire avec compression. L’utilisation du stockage intermédiaire est utile lorsque la bande passante de votre réseau d’entreprise et de vos services Azure est le facteur limitant et que vous souhaitez que le jeu de données d’entrée et le jeu de données de sortie soient sous forme non compressée. Plus spécifiquement, vous pouvez scinder une activité de copie unique en deux activités de copie. La première activité de copie copie à partir de la source vers un blob intermédiaire sous forme compressée. La deuxième activité de copie copie les données compressées de l’environnement intermédiaire, puis les décompresse tandis qu’elle écrit dans le récepteur.

## <a name="considerations-for-column-mapping"></a>Considérations relatives au mappage de colonnes
Vous pouvez définir la propriété **columnMappings** dans l’activité de copie pour mapper la totalité ou un sous-ensemble des colonnes d’entrée aux colonnes de sortie. Une fois que le service de déplacement de données a lu les données de la source, il doit effectuer le mappage de colonnes sur les données avant d’écrire celles-ci sur le récepteur. Ce traitement supplémentaire réduit le débit de copie.

Si votre banque de données source peut faire l’objet de requêtes, par exemple, s’il s’agit d’une banque de données relationnelle telle que SQL Database ou SQL Server ou s’il ne s’agit pas d’une banque NoSQL comme le Stockage Table ou Azure Cosmos DB, envisagez d’envoyer le filtrage de colonnes et la logique de réorganisation à la propriété **query** , au lieu d’utiliser le mappage de colonnes. Ainsi, la projection survient alors que le service de déplacement de données lit les données à partir de la banque de données source, la où elle est beaucoup plus efficace.

## <a name="other-considerations"></a>Autres considérations
Si la taille des données à copier est importante, vous pouvez ajuster votre logique métier pour partitionner davantage les données à l’aide du mécanisme de découpage dans Data Factory. Ensuite, planifiez l’activité de copie pour qu’elle s’exécute plus fréquemment pour réduire la taille des données pour chaque exécution d’activité de copie.

Faites attention au nombre de jeux de données et d’activités de copie nécessitant que Data Factory se connecte à la même banque de données en même temps. De nombreux travaux de copie simultanés peuvent limiter une banque de données, et entraîner une dégradation des performances, une multiplication des tentatives internes de travail de copie et, dans certains cas, des échecs d’exécution.

## <a name="sample-scenario-copy-from-a-sql-server-database-to-blob-storage"></a>Exemple de scénario : Copier d’une base de données SQL Server vers Stockage Blob
**Scénario** : Un pipeline est conçu pour copier des données d’une base de données SQL Server vers un Stockage Blob au format CSV. Pour accélérer la copie des travaux, les fichiers CSV doivent être compressés au format bzip2.

**Test et analyse** : le débit de l’activité de copie est inférieur à 2 Mbits/s, ce qui est beaucoup plus lent que le test d’évaluation des performances.

**Analyse des performances et réglage** : pour résoudre le problème de performances, nous allons tout d’abord examiner la manière dont les données sont traitées et déplacées.

1. **Lecture des données** : la passerelle ouvre la connexion à SQL Server et envoie la requête. SQL Server répond en envoyant le flux de données à la passerelle via l’intranet.
2. **Sérialiser et compresser les données** : la passerelle sérialise le flux de données au format CSV, et compresse les données dans un flux bzip2.
3. **Écriture des données** : la passerelle charge le flux bzip2 vers le Stockage Blob via Internet.

Comme vous pouvez le voir, les données sont traitées et déplacées de manière séquentielle en continu : SQL Server > LAN > Passerelle > WAN > Stockage Blob. **Les performances globales sont contrôlées par le débit minimum sur le pipeline**.

![Flux de données](./media/data-factory-copy-activity-performance/case-study-pic-1.png)

Un ou plusieurs des facteurs suivants peuvent entraîner un goulot d’étranglement des performances :

* **Source** : SQL Server offre lui-même un faible débit en raison des charges lourdes.
* **Passerelle de gestion des données**:
  * **LAN** : la passerelle est éloignée de l’ordinateur SQL Server et dispose d’une connexion à faible bande passante.
  * **Passerelle** : la passerelle a atteint ses limites de charge pour effectuer les opérations suivantes :
    * **Sérialisation** : la sérialisation du flux de données au format CSV présente un débit lent.
    * **Compression** : vous avez choisi un codec de compression lent (par exemple, bzip2, c’est-à-dire 2,8 Mbits/s avec Core i7).
  * **WAN** : la bande passante entre le réseau d’entreprise et vos services Azure est faible (par exemple, T1 = 1 544 Kbits/s ; T2 = 6 312 Kbits/s).
* **Récepteur** : le stockage Blob a un faible débit. (Ce scénario est peu probable car son contrat SLA garantit un minimum de 60 Mbits/s.)

Dans ce cas, la compression de données bzip2 pourrait ralentir l’ensemble du pipeline. Un basculement vers le codec de compression gzip peut résoudre ce goulot d’étranglement.

## <a name="sample-scenarios-use-parallel-copy"></a>Exemples de scénarios : Utiliser la copie en parallèle
**Scénario I :** copier 1000 fichiers de 1 Mo d’un système de fichiers local vers le Stockage Blob.

**Analyse des performances et réglage** : par exemple, si vous avez installé la passerelle sur un ordinateur à quatre cœurs, Data Factory utilise 16 copies en parallèle pour déplacer simultanément les fichiers du système de fichiers vers le Stockage Blob. Cette exécution parallèle doit aboutir à un débit élevé. Vous pouvez également spécifier explicitement le nombre de copies parallèles. Lorsque vous copiez plusieurs petits fichiers, les copies parallèles aident considérablement le débit en utilisant les ressources plus efficacement.

![Scénario 1](./media/data-factory-copy-activity-performance/scenario-1.png)

**Scénario II :** copier 20 blobs de 500 Mo chacun du Stockage Blob vers Data Lake Store Analytics, puis régler les performances.

**Analyse des performances et réglage** : dans ce scénario, Data Factory copie les données du Stockage Blob vers Data Lake Store en utilisant des unités de déplacement de données à copie unique (valeur **parallelCopies** définie sur 1) et à cloud unique. Le débit que vous constatez est proche de la description figurant dans la [section relative aux performances de référence](#performance-reference).

![Scénario 2](./media/data-factory-copy-activity-performance/scenario-2.png)

**Scénario III** : la taille des fichiers est supérieure à des dizaines de mégaoctets et le volume total est important.

**Analyse des performances et réglage** : l’augmentation de la valeur **parallelCopies** ne donne pas de meilleures performances de copie, en raison des limitations de ressource d’une unité de déplacement de données à cloud unique. Vous avez plutôt intérêt à spécifier plus d’unités de déplacement de données cloud pour obtenir davantage de ressources pour le déplacement de données. Ne spécifiez pas de valeur pour la propriété **parallelCopies** . Data Factory gère le parallélisme pour vous. Dans ce cas, si vous définissez **cloudDataMovementUnits** sur 4, un débit quatre fois supérieur se produit.

![Scénario 3](./media/data-factory-copy-activity-performance/scenario-3.png)

## <a name="reference"></a>Informations de référence
Voici des références relatives à la surveillance et au réglage des performances pour quelques banques de données prises en charge :

* Stockage Blob Azure : [Objectifs de performance et de scalabilité pour le stockage d’objets blob](../../storage/blobs/scalability-targets.md) et [Liste de contrôle des performances et de la scalabilité pour le stockage d’objets blob](../../storage/blobs/storage-performance-checklist.md).
* Stockage Table Azure : [Objectifs de performance et de scalabilité pour le stockage Table](../../storage/tables/scalability-targets.md) et [Liste de contrôle des performances et de la scalabilité pour le stockage Table](../../storage/tables/storage-performance-checklist.md).
* Azure SQL Database : vous pouvez [surveiller les performances](../../azure-sql/database/monitor-tune-overview.md) et vérifier le pourcentage de l’unité de transaction de base de données (DTU)
* Azure Synapse Analytics : sa capacité est mesurée en Data Warehouse Units (DWU) ; voir [Gestion de la puissance de calcul dans Azure Synapse Analytics (Vue d’ensemble)](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md)
* Azure Cosmos DB : [Niveaux de performances dans Azure Cosmos DB](../../cosmos-db/performance-levels.md)
* Serveur SQL Server local : [Surveillance et réglage des performances](/sql/relational-databases/performance/monitor-and-tune-for-performance)
* Serveur de fichiers local : [Réglage des performances des serveurs de fichiers](/previous-versions//dn567661(v=vs.85))