---
title: 'Démarrage rapide : Créer un pool SQL dédié en utilisant le portail Azure'
description: Créez un pool SQL dédié à l’aide du portail Azure en suivant les étapes de ce guide.
services: synapse-analytics
author: julieMSFT
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql
ms.date: 04/15/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: 3537b126adf082c8b00a02d999789d8a32b3f51d
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462008"
---
# <a name="quickstart-create-a-dedicated-sql-pool-using-the-azure-portal"></a>Démarrage rapide : Créer un pool SQL dédié en utilisant le portail Azure

Azure Synapse Analytics propose différents moteurs d’analytique pour vous aider à ingérer, transformer, modéliser et analyser vos données. Un pool SQL dédié offre des fonctionnalités de calcul et de stockage basées sur T-SQL. Après avoir créé un pool SQL dédié dans votre espace de travail Synapse, vous pouvez charger, modéliser, traiter et distribuer les données pour obtenir plus rapidement des insights.

Dans ce guide de démarrage rapide, vous allez apprendre à créer un pool SQL dédié dans un espace de travail Synapse à l’aide du portail Azure.

Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit avant de commencer](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)
- [Espace de travail Synapse](./quickstart-create-workspace.md)

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/)

## <a name="navigate-to-the-synapse-workspace"></a>Accéder à l’espace de travail Synapse

1. Accédez à l’espace de travail Synapse dans lequel vous allez créer le pool SQL dédié en tapant le nom du service (ou directement le nom de la ressource) dans la barre de recherche.
![Barre de recherche du portail Azure contenant le texte « espaces de travail Synapse ».](media/quickstart-create-sql-pool/create-sql-pool-00a.png). 
1. Dans la liste des espaces de travail, saisissez le nom (ou une partie du nom) de l’espace de travail à ouvrir. Pour cet exemple, nous allons utiliser un espace de travail nommé **contosoanalytics**.
![Liste des espaces de travail Synapse filtrés pour afficher ceux contenant le nom contoso.](media/quickstart-create-sql-pool/create-sql-pool-00b.png)

## <a name="create-new-dedicated-sql-pool"></a>Créer un pool SQL dédié

1. Dans l’espace de travail Synapse où vous souhaitez créer le pool SQL dédié, sélectionnez **Nouveau pool SQL dédié** dans la barre supérieure.
![Vue d’ensemble de l’espace de travail Synapse avec, dans un encadré rouge, la commande permettant de créer un pool SQL dédié.](media/quickstart-create-sql-pool/create-sql-pool-portal-01.png)
2. Sous l’onglet **Informations de base**, entrez ce qui suit :

    | Paramètre | Valeur suggérée | Description |
    | :------ | :-------------- | :---------- |
    | **Nom du pool SQL dédié** | Nom valide | Nom du pool SQL dédié. |
    | **Niveau de performances** | DW100c | Plus petite taille définie pour réduire les coûts de ce guide de démarrage rapide |

  
    ![Flux de création d’un pool SQL dédié - Onglet Informations de base.](media/quickstart-create-sql-pool/create-sql-pool-portal-02.png)

    > [!IMPORTANT]
    > Notez que les noms que vous pouvez attribuer aux pools SQL dédiés sont soumis à des limitations spécifiques. Les noms ne peuvent pas contenir de caractères spéciaux, ne doivent pas dépasser 15 caractères, ne doivent pas contenir de mots réservés et doivent être uniques dans l’espace de travail.

3. Sélectionnez **Suivant : Paramètres supplémentaires**.
4. Sélectionnez **Aucune** pour provisionner le pool SQL dédié sans données. Laissez le classement par défaut tel quel.

    Si vous souhaitez restaurer votre pool SQL dédié à partir d’un point de restauration, sélectionnez **Point de restauration**. Pour plus d’informations sur la procédure de restauration, consultez [Guide pratique : Restaurer un pool SQL dédié existant](backuprestore/restore-sql-pool.md)

![Flux de création d’un pool SQL dédié - Onglet Paramètres supplémentaires.](media/quickstart-create-sql-pool/create-sql-pool-portal-03.png)

5. Sélectionnez **Revoir + créer**.
6. Vérifiez que les informations reflètent bien ce que vous avez entré. Sélectionnez **Create** (Créer).
![Flux de création d’un pool SQL dédié - Onglet Passer en revue les paramètres.](media/quickstart-create-sql-pool/create-sql-pool-portal-04.png)

7. À ce stade, le flux de provisionnement des ressources démarre.
 ![Capture d’écran montrant la page « Votre déploiement a été effectué ».](media/quickstart-create-sql-pool/create-sql-pool-portal-06.png)

8. Quand vous revenez à l’espace de travail au terme du provisionnement, une entrée apparaît pour le pool SQL dédié créé.
 ![Flux de création d’un pool SQL - Provisionnement des ressources.](media/quickstart-create-sql-pool/create-sql-pool-studio-27.png)


Une fois le pool SQL dédié créé, vous pouvez l’utiliser dans l’espace de travail pour charger des données, traiter des flux, lire dans le lac, etc.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Effectuez les étapes ci-dessous pour supprimer le pool SQL dédié de l’espace de travail.
> [!WARNING]
> Le fait de supprimer un pool SQL dédié supprime à la fois le moteur d’analytique et les données stockées dans la base de données du pool SQL dédié supprimé de l’espace de travail. Il n’est plus possible de se connecter au pool SQL dédié, et l’ensemble des requêtes, pipelines et notebooks qui lisent ou écrivent dans ce pool SQL dédié ne fonctionnent plus.

Si vous souhaitez supprimer le pool SQL dédié, effectuez les étapes suivantes :

1. Accédez au panneau des pools SQL du panneau de l’espace de travail.
1. Sélectionnez le pool SQL dédié à supprimer (dans ce cas, **contosowdw**)
1. Une fois le pool sélectionné, appuyez sur **Supprimer**
1. Confirmez la suppression, puis appuyez sur le bouton **Supprimer** ![Vue d’ensemble du pool SQL dédié - Mise en évidence de la confirmation de la suppression.](media/quickstart-create-sql-pool/create-sql-pool-portal-11.png)
1. À la fin du processus, le pool SQL dédié ne figure plus dans la liste des ressources de l’espace de travail.

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Démarrage rapide : Créer un pool Apache Spark serverless dans Synapse Studio à l’aide des outils web](quickstart-apache-spark-notebook.md).
- Consultez [Démarrage rapide : Création d’un pool Apache Spark serverless avec le portail Azure](quickstart-create-apache-spark-pool-portal.md).
