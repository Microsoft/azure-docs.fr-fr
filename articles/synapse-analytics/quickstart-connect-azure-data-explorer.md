---
title: 'Démarrage rapide : Connecter Azure Data Explorer à un espace de travail Azure Synapse Analytics'
description: Connectez un cluster Azure Data Explorer à un espace de travail Azure Synapse Analytics en utilisant Apache Spark pour Azure Synapse Analytics.
services: synapse-analytics
author: manojraheja
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: workspace
ms.date: 10/07/2020
ms.author: maraheja
ms.reviewer: jrasnick
ms.openlocfilehash: ee9d137973bfa4eeb28bc6526437e76e781f3199
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92172267"
---
# <a name="connect-to-azure-data-explorer-using-apache-spark-for-azure-synapse-analytics"></a>Se connecter à Azure Data Explorer à l’aide d’Apache Spark pour Azure Synapse Analytics

Cet article explique comment accéder à une base de données Azure Data Explorer à partir de Synapse Studio à l’aide d’Apache Spark pour Azure Synapse Analytics.

## <a name="prerequisites"></a>Prérequis

* [Créez un cluster et une base de données Azure Data Explorer](/azure/data-explorer/create-cluster-database-portal).
* Disposez d’un espace de travail Azure Synapse Analytics ou créez un nouvel espace de travail en suivant les étapes décrites dans [Démarrage rapide : Créer un espace de travail Azure Synapse](./quickstart-create-workspace.md).
* Disposez d’un pool Apache Spark ou créez un nouveau pool en suivant les étapes décrites dans [Démarrage rapide : Créer un pool Apache Spark à l’aide du portail Azure](./quickstart-create-apache-spark-pool-portal.md).
* [Créez une application Azure Active Directory (Azure AD) en provisionnant une application Azure AD](/azure/data-explorer/kusto/management/access-control/how-to-provision-aad-app).
* Accordez à votre application Azure AD l’accès à votre base de données en suivant les étapes décrites dans [Gérer les autorisations de base de données d’Azure Data Explorer](/azure/data-explorer/manage-database-permissions).

## <a name="go-to-synapse-studio"></a>Accéder à Synapse Studio

Dans un espace de travail Azure Synapse, sélectionnez **Lancer Synapse Studio** . Dans la page d’accueil de Synapse Studio, sélectionnez **Données** pour accéder à l’ **Explorateur d’objets de données** .

## <a name="connect-an-azure-data-explorer-database-to-an-azure-synapse-workspace"></a>Connecter une base de données Azure Data Explorer à un espace de travail Azure Synapse

La connexion d’une base de données Azure Data Explorer à un espace de travail s’effectue via un service lié. Avec un service lié Azure Data Explorer, vous pouvez parcourir et explorer des données, ainsi que lire et écrire à partir d’Apache Spark pour Azure Synapse. Vous pouvez également exécuter des travaux d’intégration dans un pipeline.

À partir de l’Explorateur d’objets de données, procédez comme suit pour connecter directement un cluster Azure Data Explorer :

1. Sélectionnez l’icône **+** à proximité de **Données** .
1. Sélectionnez **Se connecter** pour vous connecter à des données externes.
1. Sélectionnez **Azure Data Explorer (Kusto)** .
1. Sélectionnez **Continuer** .
1. Utilisez un nom convivial pour nommer le service lié. Le nom sera affiché dans l’Explorateur d’objets de données et les runtimes Azure Synapse l’utiliseront pour se connecter à la base de données.
1. Sélectionnez le cluster Azure Data Explorer dans votre abonnement ou entrez l’URI.
1. Entrez l’ **ID du principal de service** et la **clé du principal de service** . Vérifiez que ce principal de service dispose d’un accès en affichage à la base de données pour l’opération de lecture et d’un accès à l’ingéreur pour l’ingestion des données.
1. Entrez le nom de la base de données Azure Data Explorer.
1. Sélectionnez **Tester la connexion** pour vérifier que vous disposez des autorisations appropriées.
1. Sélectionnez **Create** (Créer).

    ![Capture d’écran montrant un nouveau service lié.](./media/quickstart-connect-azure-data-explorer/003-new-linked-service.png)

    > [!NOTE]
    > (Facultatif) L’option **Tester la connexion** ne valide pas l’accès en écriture. Assurez-vous que votre ID de principal de service dispose d’un accès en écriture à la base de données Azure Data Explorer.

1. Les clusters et bases de données Azure Data Explorer apparaissent sous l’onglet **Lié** dans la section **Azure Data Explorer** .

    ![Capture d’écran montrant la navigation pour sélectionner les clusters.](./media/quickstart-connect-azure-data-explorer/004-browse-clusters.png)

    > [!NOTE]
    > Dans la version actuelle, les objets de base de données sont renseignés en fonction des autorisations de votre compte Azure AD sur les bases de données Azure Data Explorer. Lorsque vous exécutez les travaux d’intégration ou les notebooks Apache Spark, les informations d’identification du service de liaison sont utilisées (par exemple, le principal de service).

## <a name="quickly-interact-with-code-generated-actions"></a>Interagir rapidement avec des actions générées par le code

Lorsque vous cliquez avec le bouton droit sur une base de données ou une table, une liste d’exemples de notebooks Spark s’affiche. Sélectionnez une option pour lire, écrire ou diffuser en continu des données vers Azure Data Explorer.

[![Capture d’écran montrant de nouveaux exemples de notebooks.](./media/quickstart-connect-azure-data-explorer/005-new-notebook.png)](./media/quickstart-connect-azure-data-explorer/005-new-notebook.png#lightbox)

Voici un exemple de lecture de données. Attachez le notebook à votre pool Spark et exécutez la cellule.

[![Capture d’écran montrant un nouveau notebook de lecture.](./media/quickstart-connect-azure-data-explorer/006-read-data.png)](./media/quickstart-connect-azure-data-explorer/006-read-data.png#lightbox)

   > [!NOTE]
   > La première exécution peut prendre plus de trois minutes pour lancer la session Spark. Les exécutions suivantes seront beaucoup plus rapides.

## <a name="limitations"></a>Limites

Le connecteur Azure Data Explorer n’est actuellement pas pris en charge avec les réseaux virtuels managés Azure Synapse.

## <a name="next-steps"></a>Étapes suivantes

* [Exemple de code avec options avancées](https://github.com/Azure/azure-kusto-spark/blob/master/samples/src/main/python/SynapseSample.py)
* [Connecteur Spark Azure Data Explorer (Kusto)](https://github.com/Azure/azure-kusto-spark)