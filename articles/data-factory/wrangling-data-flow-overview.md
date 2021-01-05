---
title: Flux de données de wrangling dans Azure Data Factory
description: Vue d’ensemble des flux de données de wrangling dans Azure Data Factory
author: dcstwh
ms.author: weetok
ms.reviewer: gamal
ms.service: data-factory
ms.topic: conceptual
ms.date: 11/01/2019
ms.openlocfilehash: 607ed6167ec761a6bf380a2cc420eb72f808f0a3
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96494866"
---
# <a name="what-are-wrangling-data-flows"></a>Que sont les flux de données de wrangling ?

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]


Les organisations doivent procéder à la préparation et au wrangling des données pour une analyse précise des données complexes qui continuent à croître quotidiennement. La préparation des données est nécessaire pour que les organisations puissent utiliser les données dans différents processus d’entreprise et accélérer la rentabilité.

Les flux de données de wrangling dans Azure Data Factory vous permettent d’effectuer une préparation de données sans code à l’échelle du cloud de manière itérative. Les flux de données de wrangling s’intègrent à [Power Query Online](/power-query/) et rendent les fonctions Power Query M disponibles pour les utilisateurs de Data Factory.

Le flux de données de wrangling convertit le M généré par l’éditeur mashup Power Query Online en code Spark pour l’exécution à l’échelle du cloud.

Les flux de données de wrangling sont particulièrement utiles pour les ingénieurs de données ou les « intégrateurs de données citoyen ».

> [!NOTE]
> Le flux de wrangling data est actuellement disponible en préversion publique.

## <a name="use-cases"></a>Cas d'utilisation

### <a name="fast-interactive-data-exploration-and-preparation"></a>Exploration et préparation rapide des données interactives

Plusieurs ingénieurs de données et intégrateurs de données citoyen peuvent explorer et préparer de manière interactive les jeux de données à l’échelle du cloud. Avec l’augmentation du volume, de la variété et de la vélocité des données dans les lacs de données, les utilisateurs ont besoin d’un moyen efficace pour explorer et préparer les jeux de données. Par exemple, vous pourriez avoir à créer un jeu de données qui contient toutes les informations démographiques des nouveaux clients depuis 2017. Vous n’effectuez pas de mappage vers une cible connue. Vous explorez, effectuez le wrangling et préparez les jeux de données pour répondre à une exigence avant de les publier dans le lac. Les flux de données de wrangling sont souvent utilisés dans des scénarios analytiques moins formels. Les jeux de données préparés peuvent être utilisés pour exécuter des transformations et des opérations d’apprentissage automatique en aval.

### <a name="code-free-agile-data-preparation"></a>Préparation agile des données sans code

Les intégrateurs de données citoyen consacrent plus de 60 % de leur temps à la recherche et à la préparation des données. Ils cherchent à le faire sans code pour améliorer la productivité opérationnelle. Permettre aux intégrateurs de données citoyen d’enrichir, de mettre en forme et de publier des données à l’aide d’outils connus comme Power Query Online de façon évolutive améliore considérablement leur productivité. Un flux de données de wrangling dans Azure Data Factory permet à l’éditeur mashup Power Query Online de donner les moyens aux intégrateurs de données citoyen de corriger les erreurs rapidement, de normaliser les données et de produire des données de haute qualité pour appuyer les décisions commerciales.

### <a name="data-validation"></a>Validation des données

Analysez visuellement vos données sans code pour supprimer les valeurs hors norme et les anomalies, et les rendre aptes à une analyse rapide.

## <a name="supported-sources"></a>Sources prises en charge

| Connecteur | Format de données | Type d'authentification |
| -- | -- | --|
| [Stockage Blob Azure](connector-azure-blob-storage.md) | CSV, Parquet | Clé du compte |
| [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md) | CSV | Principal de service |
| [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) | CSV, Parquet | Clé de compte, Principal de service |
| [Azure SQL Database](connector-azure-sql-database.md) | - | Authentification SQL |
| [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md) | - | Authentification SQL |

## <a name="the-mashup-editor"></a>Editeur de mashup

Quand vous créez un flux de wrangling data, tous les jeux de données sources deviennent des requêtes de jeu de données et sont placés dans le dossier **ADFResource**. Par défaut, la requête utilisateur pointe vers la première requête de jeu de données. Toutes les transformations doivent être effectuées sur la requête utilisateur, car les modifications apportées aux requêtes de jeu de données ne sont pas prises en charge et ne sont pas rendues persistantes. Le renommage, l’ajout et la suppression de requêtes ne sont pas pris en charge.

![Wrangling](media/wrangling-data-flow/editor.png)

Actuellement, toutes les fonctions Power Query M ne sont pas prises en charge pour le rassemblement de données brutes à l’analyse, bien qu’elles soient disponibles pendant la création. Lors de la génération de vos flux de données de wrangling, le message d’erreur suivant s’affiche si une fonction n’est pas prise en charge :

`The wrangling data flow is invalid. Expression.Error: The transformation logic isn't supported. Please try a simpler expression`

Pour plus d’informations sur les transformations prises en charge, consultez [Fonctions de flux de wrangling data](wrangling-data-flow-functions.md).

Le flux de wrangling data prend uniquement en charge l’écriture dans un récepteur.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [créer un flux de données de wrangling](wrangling-data-flow-tutorial.md).