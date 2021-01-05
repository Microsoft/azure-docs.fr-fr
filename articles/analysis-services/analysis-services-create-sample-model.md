---
title: Tutoriel - Ajouter un exemple de modèle - Azure Analysis Services | Microsoft Docs
description: Dans ce tutoriel, découvrez comment ajouter un exemple de modèle dans Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: f882a40940a5c7202e9cf1f5c8b8927f008f4a39
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92013608"
---
# <a name="tutorial-add-a-sample-model-from-the-portal"></a>Didacticiel : ajouter un exemple de modèle à partir du portail

Dans ce didacticiel, vous ajoutez un exemple de base de données de modèle tabulaire Adventure Works à votre serveur. L’exemple de modèle est une version complète de l’exemple de modèle de données Adventure Works Internet Sales (1200). Un exemple de modèle sert à tester la gestion des modèles, à se connecter à des outils et des applications clientes, ainsi qu’à interroger des données de modèle. Un serveur Analysis Services dans votre abonnementCe didacticiel utilise le [portail Azure](https://portal.azure.com) et [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) pour : 

> [!div class="checklist"]
> * Ajouter un exemple de modèle de données tabulaire complet à un serveur 
> * Se connecter au modèle avec SSMS

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

- Un serveur Azure Analysis Services. Pour en savoir plus, consultez [Créer un serveur - Portail](analysis-services-create-server.md).
- Des autorisations d’administrateur du serveur
- [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)


## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail](https://portal.azure.com/).

## <a name="add-a-sample-model"></a>Ajouter un exemple de modèle

1. Dans **Vue d’ensemble** du serveur, cliquez sur **Nouveau modèle**.

    ![Créer un exemple de modèle](./media/analysis-services-create-sample-model/aas-create-sample-new-model.png)

2. Dans **Nouveau modèle** > **Choisir une source de données**, vérifiez que l’option **Exemple de données** est sélectionnée, puis cliquez sur **Ajouter**.

    ![Sélectionner un nouveau modèle](./media/analysis-services-create-sample-model/aas-create-sample-data.png)

3. Dans **Vue d’ensemble**, vérifiez que l’exemple de modèle `adventureworks` est ajouté.

    ![Sélectionner un exemple de données](./media/analysis-services-create-sample-model/aas-create-sample-verify.png)


## <a name="clean-up-resources"></a>Nettoyer les ressources

Votre exemple de modèle utilise des ressources en mémoire cache. Si vous n’utilisez pas votre exemple de modèle à des fins de test, il est préférable de le supprimer de votre serveur.

Ces étapes décrivent comment supprimer un modèle d’un serveur à l’aide de SSMS.

1. Dans SSMS > **Explorateur d’objets**, cliquez sur **Se connecter** > **Analysis Services**.

2. Dans **Se connecter au serveur**, collez le nom du serveur, puis dans **Authentification**, choisissez **Active Directory - Authentification universelle avec prise en charge de MFA**, entrez votre nom d’utilisateur, puis cliquez sur **Se connecter**.

    ![Se connecter](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-signin.png)

3. Dans l’**Explorateur d’objets**, cliquez avec le bouton droit sur l’exemple de base de données `adventureworks`, puis cliquez sur **Supprimer**.

    ![Supprimer l’exemple de base de données](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-delete.png)

## <a name="next-steps"></a>Étapes suivantes 

Dans ce didacticiel, vous avez appris comment ajouter un exemple de modèle de base à votre serveur. Maintenant que vous disposez d’une base de données de modèle, vous pouvez la connecter à SQL Server Management Studio et ajouter des rôles d’utilisateur. Pour en savoir plus, passez au didacticiel suivant.

> [!div class="nextstepaction"]
> [Didacticiel : configurer les rôles d’utilisateur et d’administrateur de serveur](tutorials/analysis-services-tutorial-roles.md)