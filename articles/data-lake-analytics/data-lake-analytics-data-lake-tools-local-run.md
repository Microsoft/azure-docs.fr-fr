---
title: Exécuter des scripts U-SQL Azure Data Lake sur votre ordinateur local
description: Découvrez comment utiliser Azure Data Lake Tools pour Visual Studio pour exécuter des travaux U-SQL sur votre ordinateur local.
author: liudan66
ms.author: liud
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 07/03/2018
ms.openlocfilehash: 5f9410cc91174420662bb5efc67c8904b5d5e647
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018985"
---
# <a name="run-u-sql-scripts-on-your-local-machine"></a>Exécuter des scripts U-SQL sur votre ordinateur local

Lorsque vous développez des scripts U-SQL, vous pouvez gagner du temps et de l’argent en les exécutant localement. Azure Data Lake Tools pour Visual Studio prend en charge l’exécution de scripts U-SQL sur votre ordinateur local. 

## <a name="basic-concepts-for-local-runs"></a>Concepts de base pour l’exécution locale

Le graphique suivant montre les composants pour l’exécution locale, ainsi que les composants correspondants pour l’exécution cloud.

|Composant|Exécution locale|Exécution cloud|
|---------|---------|---------|
|Stockage|Dossier racine de données local|Compte Azure Data Lake Store par défaut|
|Calcul|Moteur d’exécution locale U-SQL|Service Azure Data Lake Analytics|
|Environnement d’exécution|Répertoire de travail sur l’ordinateur local|Cluster Azure Data Lake Analytics|

Les sections qui suivent fournissent davantage d’informations sur les composants de l’exécution locale.

### <a name="local-data-root-folders"></a>Dossiers racine de données locaux

Un dossier racine de données local est un **magasin local** pour un compte de calcul local. Tous les dossiers du système de fichiers local présent sur votre ordinateur local peuvent être un dossier racine de données local. Il est équivalent au compte Azure Data Lake Store par défaut d’un compte Data Lake Analytics. Basculer vers un autre dossier racine de données revient à basculer vers un autre compte de magasin par défaut. 

Le dossier racine de données est utilisé pour :
- Stocker des métadonnées. Par exemple, des bases de données, des tables, des fonctions table et des assemblys.
- Rechercher les chemins d’entrée et de sortie qui sont définis comme des chemins relatifs dans les scripts U-SQL. Les chemins relatifs facilitent le déploiement de vos scripts U-SQL dans Azure.

### <a name="u-sql-local-run-engines"></a>Moteurs d’exécution locale U-SQL

Un moteur d’exécution locale U-SQL est un **compte de calcul local** pour les travaux U-SQL. Les utilisateurs peuvent exécuter des travaux U-SQL localement via Azure Data Lake Tools pour Visual Studio. Les exécutions locales sont également possibles via les interfaces de programmation et de ligne de commande du SDK U-SQL Azure Data Lake. En savoir plus sur le [SDK U-SQL Azure Data Lake](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/)

### <a name="working-directories"></a>Répertoires de travail

Lorsque vous exécutez un script U-SQL, un dossier de répertoire de travail est nécessaire pour mettre en cache les résultats de la compilation et les journaux d’activité d’exécution, et pour effectuer d’autres actions. Dans Azure Data Lake Tools pour Visual Studio, le répertoire de travail est celui du projet U-SQL. Il est situé sous `<U-SQL project root path>/bin/debug>`. Le répertoire de travail est nettoyé chaque fois qu’une nouvelle exécution est déclenchée.

## <a name="local-runs-in-microsoft-visual-studio"></a>Exécutions locales dans Microsoft Visual Studio

Azure Data Lake Tools pour Visual Studio comprend un moteur d’exécution locale intégré. Ce moteur est représenté comme un compte de calcul local. Pour exécuter un script U-SQL localement, sélectionnez le compte **Local-machine** ou **Local-project** dans le menu déroulant situé dans la marge de l’éditeur de script. Ensuite, sélectionnez **Envoyer**.

![Envoyer un script U-SQL vers un compte local](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-submit-script-to-local-account.png) 
 
## <a name="local-runs-with-a-local-machine-account"></a>Exécutions locales avec un compte Local-machine

Le compte **Local-machine** est un compte de calcul local partagé. Il comprend un seul dossier racine de données local qui joue le rôle de compte de magasin local. Par défaut, le dossier racine de données se trouve à l’emplacement **C:\Users\<username>\AppData\Local\USQLDataRoot**. Il est également configurable via **Outils** > **Data Lake** > **Options et paramètres**.

![Configurer un dossier racine de données local](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-configure-local-data-root.png)
  
Pour une exécution locale, vous avez besoin d’un projet U-SQL. Le répertoire de travail du projet U-SQL est utilisé pour le répertoire de travail de l’exécution locale U-SQL. Les résultats de la compilation, les journaux d’activité d’exécution et les autres fichiers liés à l’exécution du travail, sont générés et stockés dans le dossier du répertoire de travail pendant l’exécution locale. Chaque fois que vous réexécutez le script, tous les fichiers du répertoire de travail sont nettoyés et régénérés.

## <a name="local-runs-with-a-local-project-account"></a>Exécutions locales avec un compte Local-project

Un compte **Local-project** est un compte de calcul local isolé du projet, pour chaque projet comprenant un dossier racine de données local isolé. Chaque projet U-SQL actif qui s’ouvre dans l’Explorateur de solutions de Visual Studio est associé à un compte `(Local-project: <project name>)`. Les comptes sont répertoriés à la fois dans l’Explorateur de serveurs de Visual Studio et dans la marge de l’éditeur de script U-SQL.  

Le compte **Local-project** fournit un environnement de développement propre et isolé pour les développeurs. Un compte **Local-machine** comprend un dossier racine de données local partagé, qui stocke les métadonnées et les données d’entrée et de sortie pour tous les travaux locaux. Cependant, un compte **Local-project** crée un dossier racine de données local temporaire dans un répertoire de travail de projet U-SQL chaque fois qu’un script U-SQL est exécuté. Ce dossier racine de données temporaire est nettoyé après une regénération ou une réexécution. 

Un projet U-SQL gère l’environnement d’exécution locale isolé via une référence et une propriété de projet. Vous pouvez configurer les sources de données d’entrée pour les scripts U-SQL dans le projet, ainsi que dans les environnements de bases de données référencées.

### <a name="manage-the-input-data-source-for-a-local-project-account"></a>Gérer la source de données d’entrée pour le compte Local-project 

Un projet U-SQL crée un dossier racine de données local et définit des données pour un compte **Local-project**. Un dossier racine de données temporaire est nettoyé et recréé sous le répertoire de travail du projet U-SQL chaque fois qu’une exécution locale et une regénération se produisent. Toutes les sources de données qui sont configurées par le projet U-SQL sont copiées dans ce dossier racine de données local temporaire avant l’exécution locale des travaux. 

Vous pouvez configurer le dossier racine de vos sources de données. Cliquez avec le bouton droit sur **Projet U-SQL** > **Propriété** > **Source de données de test**. Lorsque vous exécutez un script U-SQL sur un compte **Local-project**, tous les fichiers et sous-dossiers du dossier **Source de données de test** sont copiés dans le dossier racine de données local temporaire. Les fichiers des sous-dossiers sont inclus. Après l’exécution locale d’un travail, les résultats de la sortie figurent également dans le dossier racine de données local temporaire du répertoire de travail du projet. Toutes ces sorties sont supprimées et nettoyées quand le projet est regénéré et nettoyé. 

![Configurer la source de données de test d’un projet](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-configure-project-test-data-source.png)

### <a name="manage-a-referenced-database-environment-for-a-local-project-account"></a>Gérer un environnement de base de données référencée pour un compte **Local-project** 

Si une requête U-SQL utilise ou interroge avec des objets de base de données U-SQL, vous devez préparer les environnements de base de données localement avant d’exécuter ce script U-SQL localement. Pour le compte **Local-project**, les dépendances de base de données U-SQL peuvent être gérées par des références au projet U-SQL. Vous pouvez ajouter des références au projet de base de données U-SQL à votre projet U-SQL. Avant l’exécution des scripts U-SQL sur un compte **Local-project**, toutes les bases de données référencées sont déployées dans le dossier racine de données local temporaire. Pour chaque exécution, le dossier racine de données temporaire est alors nettoyé comme un nouvel environnement isolé.

Consultez cet article connexe :
* Découvrez comment gérer les définitions et les références de base de données U-SQL dans les [projets de base de données U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md).

## <a name="the-difference-between-local-machine-and-local-project-accounts"></a>Différences entre le compte **Local-machine** et le compte **Local-project**

Le compte **Local-machine** simule un compte Azure Data Lake Analytics sur les ordinateurs locaux des utilisateurs. Il s’utilise comme un compte Azure Data Lake Analytics. Un compte **Local-project** fournit un environnement de développement local convivial. Cet environnement permet aux utilisateurs de déployer des références de base de données et des données d’entrée avant d’exécuter localement les scripts. Un compte **Local-machine** fournit un environnement permanent partagé, accessible via tous les projets. Un compte **Local-project** fournit un environnement de développement isolé pour chaque projet. Il est actualisé pour chaque exécution. Un compte **Local-project** offre une expérience de développement plus rapide en appliquant rapidement les nouvelles modifications.

Le tableau suivant montre d’autres différences entre les comptes **Local-machine** et **Local-project** :

|Différence|Local-machine|Local-project|
|----------------|---------------|---------------|
|Accès local|Accessible par tous les projets.|Seul le projet correspondant peut accéder à ce compte.|
|Dossier racine de données local|Dossier local permanent. Configuré via **Outils** > **Data Lake** > **Options et paramètres**.|Un dossier temporaire créé pour chaque exécution locale sous le répertoire de travail du projet U-SQL. Le dossier est nettoyé en cas de regénération ou de réexécution.|
|Données d’entrée pour un script U-SQL|Chemin d’accès relatif sous le dossier racine de données local permanent.|Défini via **Propriété du projet U-SQL** > **Source des données de test**. Avant une exécution locale, tous les fichiers et les sous-dossiers sont copiés dans le dossier racine de données temporaire.|
|Données de sortie pour un script U-SQL|Chemin d’accès relatif sous le dossier racine de données local permanent.|Sortie du dossier racine de données temporaire. Les résultats sont nettoyés après une regénération ou une réexécution.|
|Déploiement de base de données référencée|Les bases de données référencées ne sont pas déployées automatiquement lors d’une exécution dans un compte **Local-machine**. Il en va de même pour l’envoi vers un compte Azure Data Lake Analytics.|Avant une exécution locale, les bases de données référencées sont déployées automatiquement dans le compte **Local-project**. Tous les environnements de base de données sont nettoyés et redéployés après une regénération ou une réexécution.|

## <a name="a-local-run-with-the-u-sql-sdk"></a>Exécution locale avec le SDK U-SQL

Vous pouvez exécuter des scripts U-SQL localement dans Visual Studio. Vous pouvez également utiliser le SDK U-SQL Azure Data Lake pour exécuter les scripts U-SQL localement, avec l’interface de ligne de commande et l’interface de programmation. Grâce à ces interfaces, vous pouvez automatiser les exécutions locales et les tests U-SQL.

En savoir plus sur le [SDK U-SQL Azure Data Lake](data-lake-analytics-u-sql-sdk.md)

## <a name="next-steps"></a>Étapes suivantes

- [Comment configurer un pipeline CI/CD pour Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md)
- [Guide pratique pour tester votre code Azure Data Lake Analytics](data-lake-analytics-cicd-test.md)
