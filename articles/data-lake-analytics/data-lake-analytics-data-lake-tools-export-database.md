---
title: Exporter une base de données U-SQL - Azure Data Lake Tools pour Visual Studio
description: Découvrez comment utiliser Azure Data Lake Tools pour Visual Studio pour exporter une base de données U-SQL et l’importer automatiquement dans un compte local.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 11/27/2017
ms.openlocfilehash: 0a768990a33f0d60a3222bd3ad323edcc53c9ec0
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219086"
---
# <a name="export-a-u-sql-database"></a>Exporter une base de données U-SQL

Dans cet article, découvrez comment utiliser [Azure Data Lake Tools pour Visual Studio](https://aka.ms/adltoolsvs) pour exporter une base de données U-SQL sous la forme d’un seul script U-SQL et de ressources téléchargées. Vous pouvez importer la base de données exportée dans un compte local dans le même processus.

Les clients gèrent généralement plusieurs environnements de développement, de test et de production. Ces environnements sont hébergés à la fois sur un compte local sur l’ordinateur local des développeurs et sur un compte Azure Data Lake Analytics dans Azure. 

Lorsque vous développez et optimisez des requêtes U-SQL dans des environnements de développement et de test, les développeurs doivent souvent recréer leur travail dans une base de données de production. L’Assistant Exportation de base de données contribue à accélérer ce processus. À l’aide de l’Assistant, les développeurs peuvent cloner l’environnement de base de données existant et échantillonner des données vers d’autres comptes Data Lake Analytics.

## <a name="export-steps"></a>Opérations d’exportation

### <a name="step-1-export-the-database-in-server-explorer"></a>Étape 1: exporter la base de données dans l'Explorateur de serveurs

Tous les comptes Data Lake Analytics auxquels vous avez accès sont répertoriés dans l’Explorateur de serveurs. Pour exporter la base de données :

1. Dans l'Explorateur de serveurs, développez le compte qui contient la base de données que vous souhaitez exporter.
2. Cliquez avec le bouton droit sur la base de données et sélectionnez **Exporter**. 
   
    ![Explorateur de serveurs - Exporter une base de données](./media/data-lake-analytics-data-lake-tools-export-database/export-database.png)

     Si l'option de menu **Exporter** n'est pas disponible, vous devez [mettre à jour l'outil avec la dernière version](https://aka.ms/adltoolsvs).

### <a name="step-2-configure-the-objects-that-you-want-to-export"></a>Étape 2 : Configurer les objets à exporter

Si vous n'avez besoin que d'une petite partie d'une base de données volumineuse, vous pouvez configurer un sous-ensemble d'objets à exporter dans l'assistant Exportation. 

L'action d'exportation est finalisée en exécutant un travail U-SQL. Par conséquent, l'exportation à partir d'un compte Azure entraîne certains coûts.

![Assistant Exportation de base de données - Sélectionner les objets d'exportation](./media/data-lake-analytics-data-lake-tools-export-database/export-database-wizard.png)

### <a name="step-3-check-the-objects-list-and-other-configurations"></a>Étape 3 : Vérifier la liste des objets et d’autres configurations

Dans cette étape, vous pouvez vérifier les objets sélectionnés dans la zone de liste **Objet d'exportation**. S’il existe des erreurs, sélectionnez **Précédent** pour revenir en arrière et reconfigurer correctement les objets que vous voulez exporter.

Vous pouvez également configurer d'autres paramètres pour la cible d'exportation. Les descriptions de configuration sont répertoriées dans le tableau suivant :

|Configuration|Description|
|-------------|-----------|
|Nom de la destination|Ce nom indique où vous souhaitez enregistrer les ressources de base de données exportées, comme des assemblys, des fichiers supplémentaires et des exemples de données. Un dossier portant ce nom est créé sous le dossier racine de vos données locales.|
|Répertoire du projet|Ce chemin définit l'emplacement où vous voulez enregistrer le script U-SQL exporté. Toutes les définitions d'objet de base de données sont enregistrées à cet emplacement.|
|Schéma uniquement|Si vous sélectionnez cette option, seules les définitions de base de données et les ressources (comme les assemblys et les fichiers supplémentaires) sont exportées.|
|Schéma et données|Si vous sélectionnez cette option, les définitions de base de données, les ressources et les données sont exportées. Les N premières lignes des tables sont exportées.|
|Importer automatiquement dans la base de données locale|Si vous sélectionnez cette option, la base de données exportée est automatiquement importée dans votre base de données locale lorsque l'exportation est terminée.|

![Assistant Exportation de base de données - Exporter une liste d'objets et d'autres configurations](./media/data-lake-analytics-data-lake-tools-export-database/export-database-wizard-configuration.png)

### <a name="step-4-check-the-export-results"></a>Étape 4 : Vérifier les résultats de l’exportation

Lorsque l'exportation est terminée, vous pouvez afficher les résultats exportés dans la fenêtre du journal de l'assistant. L'exemple suivant montre comment rechercher des ressources de script et de base de données U-SQL exportées, notamment des assemblys, des fichiers supplémentaires et des exemples de données :

![Assistant Exportation de base de données - Exporter les résultats](./media/data-lake-analytics-data-lake-tools-export-database/export-database-wizard-completed.png)

## <a name="import-the-exported-database-to-a-local-account"></a>Importer la base de données exportée dans un compte local

Le moyen le plus pratique d'importer la base de données exportée consiste à cocher la case **Importer automatiquement dans la base de données locale** lors du processus d'exportation à l'étape 3. Si vous n'avez pas coché cette case, recherchez d'abord le script U-SQL exporté dans le journal d'exportation. Puis exécutez le script U-SQL localement pour importer la base de données dans votre compte local.

## <a name="import-the-exported-database-to-a-data-lake-analytics-account"></a>Importer la base de données exportée dans un compte Data Lake Analytics

Pour importer la base de données dans un autre compte Data Lake Analytics :

1. Chargez les ressources exportées, notamment les assemblys, les fichiers supplémentaires et les exemples de données dans le compte Azure Data Lake Store par défaut du compte Data Lake Analytics dans lequel vous voulez effectuer l’importation. Vous trouverez le dossier des ressources exportées dans le répertoire racine des données locales. Chargez la totalité du dossier à la racine du compte Data Lake Store par défaut.
2. Une fois le chargement terminé, envoyez le script U-SQL exporté au compte Data Lake Analytics dans lequel vous souhaitez importer la base de données.

## <a name="known-limitations"></a>Limitations connues

Actuellement, si vous sélectionnez l'option **Schéma et données** à l'étape 3, l’outil exécute un travail U-SQL pour exporter les données stockées dans des tables. Pour cette raison, le processus d'exportation de données peut être lent et entraîner des frais. 

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur les bases de données U-SQL](/u-sql/data-definition-language-ddl-statements) 
* [Tester et déboguer des travaux U-SQL à l’aide d’une exécution locale et du Kit de développement logiciel (SDK) Azure Data Lake U-SQL](data-lake-analytics-data-lake-tools-local-run.md)


