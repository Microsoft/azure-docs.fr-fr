---
title: Guide sur les performances et la scalabilité de l’activité de copie
description: Découvrez les facteurs clés ayant des répercussions sur les performances du déplacement de données dans Azure Data Factory lorsque vous utilisez l’activité de copie.
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/15/2020
ms.openlocfilehash: a6e89883ec0e1e493bad9572876af86f4a0d3853
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91324429"
---
# <a name="copy-activity-performance-and-scalability-guide"></a>Guide sur les performances et la scalabilité de l’activité de copie

> [!div class="op_single_selector" title1="Sélectionnez la version Azure Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-copy-activity-performance.md)
> * [Version actuelle](copy-activity-performance.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Parfois, vous souhaitez effectuer une migration de données à grande échelle d’un lac de données ou un entrepôt de données d’entreprise (EDW) vers Azure. Dans d’autres cas, vous souhaiterez ingérer dans Azure de grandes quantités de données provenant de différentes sources pour l’analytique du Big Data. Dans chaque cas, il est essentiel d’obtenir des performances et une scalabilité optimales.

Azure Data Factory fournit un mécanisme permettant d’ingérer des données. Azure Data Factory présente les avantages suivants :

* Gère de grandes quantités de données
* Est très performant
* Est économique

Ces avantages font d’Azure Data Factory une solution idéale pour les ingénieurs de données qui souhaitent créer des pipelines d’ingestion de données évolutifs et très performants.

Après avoir lu cet article, vous serez en mesure de répondre aux questions suivantes :

* Quel niveau de performance et de scalabilité puis-je obtenir à l’aide de l’activité de copie ADF pour les scénarios de migration de données et d’ingestion de données ?
* Quelles sont les étapes à suivre pour régler les performances de l’activité de copie ADF ?
* Quels sont les leviers d’optimisation des performances ADF que je peux utiliser pour optimiser les performances d’une exécution d’activité de copie unique ?
* Quels sont les autres facteurs hors d’ADF à prendre en compte lors de l’optimisation des performances de copie ?

> [!NOTE]
> Si vous n’êtes pas familiarisé avec l’activité de copie, consultez la [vue d’ensemble de l’activité de copie](copy-activity-overview.md) avant de lire cet article.

## <a name="copy-performance-and-scalability-achievable-using-adf"></a>Scalabilité et performances de l’activité de copie réalisables avec ADF

Azure Data Factory offre une architecture serverless qui permet un parallélisme à différents niveaux.

Cette architecture vous permet de développer des pipelines qui optimisent le débit des déplacements de données pour votre environnement. Ces pipelines utilisent pleinement les ressources suivantes :

* Bande passante réseau
* Opérations d’entrée/sortie par seconde (IOPS) sur le stockage et bande passante

Cette utilisation complète signifie que vous pouvez estimer le débit global en mesurant le débit minimal disponible avec les ressources suivantes :

* Magasin de données source
* Banque de données de destination
* Bande passante réseau entre les magasins de données source et de destination

Le tableau ci-dessous calcule la durée de la copie. La durée dépend de la taille des données et de la limite de bande passante pour votre environnement.

&nbsp;

| Taille des données / <br/> bandwidth | 50 Mbits/s    | 100 Mbits/s  | 500 Mbits/s  | 1 Gbit/s   | 5 Gbit/s   | 10 Gbits/s  | 50 Gbit/s   |
| --------------------------- | ---------- | --------- | --------- | -------- | -------- | -------- | --------- |
| **1 Go**                    | 2,7 min    | 1,4 min   | 0,3 min   | 0,1 min  | 0,03 min | 0,01 min | 0,0 min   |
| **10 Go**                   | 27,3 min   | 13,7 min  | 2,7 min   | 1,3 min  | 0,3 min  | 0,1 min  | 0,03 min  |
| **100 Go**                  | 4,6 heures    | 2,3 heures   | 0,5 heure   | 0,2 heures  | 0,05 heure | 0,02 heure | 0,0 heure   |
| **1 To**                    | 46,6 heures   | 23,3 heures  | 4,7 heures   | 2,3 heures  | 0,5 heure  | 0,2 heures  | 0,05 heure  |
| **10 To**                   | 19,4 jours  | 9,7 jours  | 1,9 jours  | 0,9 jour | 0,2 jour | 0,1 jour | 0,02 jour |
| **100 To**                  | 194,2 jours | 97,1 jours | 19,4 jours | 9,7 jours | 1,9 jours | 1 jour    | 0,2 jour  |
| **1 Po**                    | 64,7 mois    | 32,4 mois   | 6,5 mois    | 3,2 mois   | 0,6 mois   | 0,3 mois   | 0,06 mois   |
| **10 Po**                   | 647,3 mois   | 323,6 mois  | 64,7 mois   | 31.6 mois  | 6,5 mois   | 3,2 mois   | 0,6 mois    |
| | |  | | |  | | |

La copie ADF est scalable à différents niveaux :

![Mise à l’échelle de la copie ADF](media/copy-activity-performance/adf-copy-scalability.png)

* Le flux de contrôle ADF peut démarrer plusieurs activités de copie en parallèle, par exemple [Pour chaque boucle](control-flow-for-each-activity.md).

* Une seule activité de copie peut tirer parti de plusieurs ressources de calcul évolutives.
  * Quand vous utilisez le runtime d’intégration Azure, vous pouvez spécifier [jusqu’à 256 unités d’intégration de données (DIU)](#data-integration-units) pour chaque activité de copie de manière serverless.
  * Lorsque vous utilisez le runtime d’intégration auto-hébergé, vous pouvez adopter l’une des approches suivantes :
    * Effectuer manuellement un scale-up de la machine.
    * Effectuer un scale-out sur plusieurs machines ([jusqu’à quatre nœuds](create-self-hosted-integration-runtime.md#high-availability-and-scalability)), et une seule activité de copie partitionne son ensemble de fichiers sur tous les nœuds.

* Une activité de copie unique lit et écrit dans le magasin de données à l’aide de plusieurs threads [en parallèle](#parallel-copy).

## <a name="performance-tuning-steps"></a>Procédure de réglage des performances

Procédez comme suit pour régler les performances de votre service Azure Data Factory avec l’activité de copie :

1. **Choisir un jeu de données de test et établir une base de référence.**

    Pendant le développement, testez votre pipeline en utilisant l’activité de copie sur un échantillon de données représentatif. Le jeu de données que vous choisissez doit représenter vos modèles de données typiques avec les attributs suivants :

    * Structure de dossiers
    * Modèle de fichier
    * Schéma de données

    Votre jeu de données doit également être suffisamment important pour évaluer les performances de copie. Une taille appropriée prend au moins dix minutes pour que l’activité de copie se termine. Collectez les détails de l’exécution et les caractéristiques des performances suivant la [surveillance de l’activité de copie](copy-activity-monitoring.md).

2. **Comment optimiser les performances d’une seule activité de copie** :

    Nous vous recommandons d’optimiser les performances en utilisant une seule activité de copie.

    * **Si l’activité de copie est exécutée sur un runtime d’intégration _Azure_ :**

        Commencez avec les valeurs par défaut des paramètres [DIU (unités d’intégration de données)](#data-integration-units) et de [copie en parallèle](#parallel-copy).

    * **Si l’activité de copie est exécutée sur un runtime d’intégration _auto-hébergé_ :**

        Nous vous recommandons d’utiliser un ordinateur dédié pour héberger le runtime d’intégration. L’ordinateur doit être séparé du serveur qui héberge le magasin de données. Commencez avec les valeurs par défaut du paramètre de [copie parallèle](#parallel-copy) en utilisant un seul nœud pour l’IR auto-hébergé.

    Effectuez une série de tests de performances. Prenez note des performances atteintes. Incluez les valeurs réelles utilisées, telles que les DIU et les copies en parallèles. Reportez-vous à [Supervision de l’activité de copie](copy-activity-monitoring.md) pour savoir comment collecter les résultats des tests et les paramètres de performances utilisés. Pour identifier et résoudre le goulot d’étranglement, découvrez comment [résoudre les problèmes liés aux performances de l’activité de copie](copy-activity-performance-troubleshooting.md).

    Renouvelez l’opération pour exécuter des tests de performances supplémentaires en suivant les instructions de résolution des problèmes et de réglage. Une fois que les exécutions d’activité de copie unique n’obtiennent pas de meilleur débit, déterminez s’il faut optimiser le débit agrégé en exécutant plusieurs copies simultanément. Cette option est abordée dans le point numéroté suivant.

3. **Comment optimiser le débit agrégé en exécutant plusieurs copies simultanément :**

    Vous avez désormais optimisé les performances d’une seule activité de copie. Si vous n’avez pas encore atteint les limites supérieures de débit de votre environnement, vous pouvez exécuter plusieurs activités de copie en parallèle. Vous pouvez les exécuter en parallèle à l’aide des constructions de flux de contrôle d’Azure Data Factory. L’une de ces constructions est la [boucle For Each](control-flow-for-each-activity.md). Pour plus d’informations, consultez les articles suivants relatifs aux modèles de solution :

    * [Copie de fichiers à partir de plusieurs conteneurs](solution-template-copy-files-multiple-containers.md)
    * [Migrer des données d’Amazon S3 vers ADLS Gen2](solution-template-migration-s3-azure.md)
    * [Copie en bloc avec une table de contrôles](solution-template-bulk-copy-with-control-table.md)

4. **Étendez la configuration à l’ensemble de votre jeu de données.**

    Lorsque vous êtes satisfait des résultats et des performances de l’exécution, vous pouvez étendre la définition et le pipeline pour couvrir l’ensemble de votre jeu de données.

## <a name="troubleshoot-copy-activity-performance"></a>Résoudre les problèmes de performances de l’activité de copie

Suivez la [procédure de réglage des performances](#performance-tuning-steps) pour planifier et effectuer un test de performances pour votre scénario. Et découvrez comment résoudre les problèmes de performances de chaque exécution d’activité de copie dans Azure Data Factory dans [Résoudre les problèmes de performances de l’activité de copie](copy-activity-performance-troubleshooting.md).

## <a name="copy-performance-optimization-features"></a>Fonctionnalités d’optimisation des performances de copie

Azure Data Factory fournit les fonctionnalités d’optimisation des performances suivantes :

* [Unités d’intégration de données](#data-integration-units)
* [Extensibilité du runtime d’intégration auto-hébergé](#self-hosted-integration-runtime-scalability)
* [Copie en parallèle](#parallel-copy)
* [Copie intermédiaire](#staged-copy)

### <a name="data-integration-units"></a>Unités d’intégration de données

Une unité d’intégration de données (DIU) est une mesure qui représente la puissance d’une seule unité dans Azure Data Factory. La puissance est une combinaison de l’allocation des ressources du processeur, de la mémoire et du réseau. Une DIU s’applique uniquement au [runtime d’intégration Azure](concepts-integration-runtime.md#azure-integration-runtime). Une DIU ne s’applique pas au [runtime d’intégration auto-hébergé](concepts-integration-runtime.md#self-hosted-integration-runtime). [En savoir plus ici](copy-activity-performance-features.md#data-integration-units).

### <a name="self-hosted-integration-runtime-scalability"></a>Extensibilité du runtime d’intégration auto-hébergé

Vous souhaiterez peut-être héberger une charge de travail simultanée croissante. Vous pouvez également chercher à obtenir des performances supérieures dans le cadre de votre charge de travail actuelle. Vous pouvez améliorer l’échelle du traitement en procédant de l’une des manières suivantes :

* Vous pouvez effectuer un _scale-up_ du runtime d’intégration auto-hébergé en augmentant le nombre de [travaux simultanés](create-self-hosted-integration-runtime.md#scale-up) pouvant s’exécuter sur un nœud.  
Le scale-up fonctionne uniquement si le processeur et la mémoire du nœud ne sont pas pleinement utilisés.
* Vous pouvez effectuer un _scale-out_ du runtime d’intégration auto-hébergé en ajoutant plus de nœuds (machines).

Pour plus d'informations, consultez les pages suivantes :

* [Fonctionnalités d’optimisation des performances de l’activité de copie : Extensibilité du runtime d’intégration auto-hébergé](copy-activity-performance-features.md#self-hosted-integration-runtime-scalability)
* [Créer et configurer un runtime d’intégration auto-hébergé : Considérations d’échelle](create-self-hosted-integration-runtime.md#scale-considerations)

### <a name="parallel-copy"></a>Copie en parallèle

Vous pouvez définir la propriété `parallelCopies` pour indiquer le parallélisme que vous souhaitez que l’activité de copie utilise. Considérez cette propriété comme le nombre maximal de threads dans l’activité de copie. Les threads fonctionnent en parallèle. Les threads lisent à partir de votre source ou écrivent dans vos magasins de données récepteur. [En savoir plus](copy-activity-performance-features.md#parallel-copy)

### <a name="staged-copy"></a>copie intermédiaire

Une opération de copie de données peut envoyer les données _directement_ au magasin de données récepteur. Vous pouvez également choisir d’utiliser Stockage Blob comme magasin _temporaire de processus de site_. [Plus d’informations](copy-activity-performance-features.md#staged-copy)

## <a name="next-steps"></a>Étapes suivantes

Consultez les autres articles relatifs à l’activité de copie :

* [Vue d’ensemble des activités de copie](copy-activity-overview.md)
* [Résoudre les problèmes de performances de l’activité de copie](copy-activity-performance-troubleshooting.md)
* [Fonctionnalités d’optimisation des performances de l’activité de copie](copy-activity-performance-features.md)
* [Utiliser Azure Data Factory pour migrer des données de Data Lake ou Data Warehouse vers Azure](data-migration-guidance-overview.md)
* [Migrer des données d’Amazon S3 vers le stockage Azure](data-migration-guidance-s3-azure-storage.md)
