---
title: Interroger Azure Data Lake Analytics - Visual Studio
description: Découvrez comment installer Data Lake Tools pour Visual Studio et développer et tester des scripts U-SQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 08/30/2019
ms.openlocfilehash: d3812cb27d2d9ea7a49038f566f8b4cc5d779172
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95241690"
---
# <a name="develop-u-sql-scripts-by-using-data-lake-tools-for-visual-studio"></a>Développer des scripts U-SQL à l’aide des outils Data Lake pour Visual Studio

[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

Azure Data Lake et Stream Analytics Tools incluent des fonctionnalités en rapport avec deux services Azure : Azure Data Lake Analytics et Azure Stream Analytics. Pour plus d’informations sur les scénarios Azure Stream Analytics, consultez [Azure Stream Analytics Tools pour Visual Studio](../stream-analytics/stream-analytics-tools-for-visual-studio-install.md).

Cet article explique comment utiliser Visual Studio pour créer des comptes Azure Data Lake Analytics. Vous pouvez définir des tâches dans [U-SQL](data-lake-analytics-u-sql-get-started.md), et soumettre des tâches au service Data Lake Analytics. Pour plus d’informations sur Analytique Data Lake, consultez [Présentation d’Analytique Data Lake Azure](data-lake-analytics-overview.md).

> [!IMPORTANT]
> Nous vous recommandons de procéder à une mise à niveau vers Azure Data Lake Tools pour Visual Studio version 2.3.3000.4 ou ultérieure. Notez que les versions précédentes ne peuvent plus être téléchargées et que Microsoft les déconseille.
>
> 1. Veillez à utiliser une version d’Azure Data Lake Tools pour Visual Studio antérieure à 2.3.3000.4.
>
>    ![Vérifier la version de l’outil](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-about-data-lake.png)
>
> 1. Si votre version est antérieure à 2.3.3000.4, mettez à jour Azure Data Lake Tools pour Visual Studio par le biais du centre de téléchargement :
>    - [Pour Visual Studio 2017 et 2019](https://marketplace.visualstudio.com/items?itemName=ADLTools.AzureDataLakeandStreamAnalyticsTools)
>    - [Pour Visual Studio 2013 et 2015](https://www.microsoft.com/en-us/download/details.aspx?id=49504)

## <a name="prerequisites"></a>Conditions préalables requises

* **Visual Studio** : toutes les éditions sauf Express sont prises en charge.

  * Visual Studio 2019
  * Visual Studio 2017
  * Visual Studio 2015
  * Visual Studio 2013

* **Kit SDK Microsoft Azure pour .NET** version 2.7.1 ou ultérieure. Installez-le à l’aide de [Web Platform Installer](https://www.microsoft.com/web/downloads/platform.aspx).
* Un compte **Data Lake Analytics**. Pour créer un compte, consultez [Prise en main d’Azure Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-get-started-portal.md).

## <a name="install-azure-data-lake-tools-for-visual-studio"></a>Installer Azure Data Lake Tools pour Visual Studio

Ce didacticiel nécessite que Data Lake Tools pour Visual Studio soit installé. Pour plus d’informations, voir [Installation de Data Lake Tools pour Visual Studio](data-lake-analytics-data-lake-tools-install.md).

## <a name="connect-to-an-azure-data-lake-analytics-account"></a>Se connecter à un compte Azure Data Lake Analytics

1. Ouvrez Visual Studio.

1. Ouvrez **Explorateur Data Lake Analytics** en sélectionnant **Afficher** > **Explorateur Data Lake Analytics**.

1. Cliquez avec le bouton droit sur **Azure**, puis sélectionner **Se connecter à un abonnement Microsoft Azure**. Dans **Connectez-vous à votre compte**, suivez les instructions.

1. Dans l’**Explorateur de serveurs**, sélectionnez **Azure** > **Data Lake Analytics**. Une liste des comptes Data Lake Analytics s’affiche.

## <a name="write-your-first-u-sql-script"></a>Rédiger son premier script U-SQL

Le texte suivant est un script U-SQL simple. Il définit un petit jeu de données et l’écrit dans le Data Lake Store par défaut sous la forme d’un fichier appelé `/data.csv`.

```sql
USE DATABASE master;
USE SCHEMA dbo;
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
```

## <a name="submit-a-data-lake-analytics-job"></a>Envoyer le travail Analytique Data Lake

1. Dans Visual Studio, sélectionnez **Fichier** > **Nouveau** > **Projet**.

1. Sélectionnez le type **Projet U-SQL**, puis cliquez sur **Suivant**. Dans **Configurer votre nouveau projet**, sélectionnez **Créer**.

   Visual Studio crée une solution qui contient un fichier **Script.usql**.

1. Collez le script de [Rédiger son premier script U-SQL](#write-your-first-u-sql-script) dans la fenêtre **Script.usql**.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit de la souris sur **Script.usql**, puis sélectionnez **Envoyer le script**.

1. Dans **Envoyer le travail**, choisissez votre compte Data Lake Analytics, puis sélectionnez **Envoyer**.

   ![Soumettre un projet U-SQL Visual Studio](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-submit-job-vs2019.png)

Une fois que le travail a été envoyé, l’onglet **Vue du travail** s’affiche pour vous présenter la progression du travail.

* **Résumé du travail** montre le résumé du travail.
* **Graphique des travaux** permet de visualiser la progression du travail.
* **Opérations sur les métadonnées** indique toutes les actions qui ont été effectuées sur le catalogue U-SQL.
* **Données** indique toutes les entrées et sorties.
* **Historique de l’état** affiche la chronologie et les détails d’état.
* **Analyse des unités d’allocation** indique le nombre d’unités d’allocation utilisées dans le travail et explore des simulations de différentes stratégies d’allocation d’unités d’allocation.
* **Diagnostics** fournit une analyse avancée pour l’optimisation des performances et de l’exécution des travaux.

![Graphique des performances du travail U-SQL Visual Studio Data Lake Analytics](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-performance-graph.png)

Sélectionnez **Actualiser** pour afficher le dernier état du travail et actualiser l’écran.

## <a name="check-job-status"></a>Vérifier l’état du travail

1. Dans l’**Explorateur de serveurs**, sélectionnez **Azure** > **Data Lake Analytics**.

1. Développez le nom du compte Data Lake Analytics.

1. Double-cliquez sur **Travaux**.

1. Sélectionnez le travail que vous avez déjà envoyé.

## <a name="see-the-job-output"></a>Visualiser la sortie du travail

1. Dans l’**Explorateur de serveurs**, accédez au travail que vous avez envoyé.

1. Cliquez sur l'onglet **Données** .

1. Dans l’onglet **Sorties du travail**, sélectionnez le fichier `"/data.csv"`.

## <a name="next-steps"></a>Étapes suivantes

* [Run U-SQL scripts on your own workstation for testing and debugging (Exécuter des scripts U-SQL sur votre station de travail pour les tests et le débogage)](data-lake-analytics-data-lake-tools-local-run.md)
* [Débogage du code C# dans les travaux U-SQL à l’aide d’Azure Data Lake Tools pour Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
* [Utilisation d’Azure Data Lake Tools pour Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
