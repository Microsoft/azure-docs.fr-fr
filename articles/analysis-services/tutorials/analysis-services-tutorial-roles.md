---
title: 'Didacticiel : configurer les rôles Azure Analysis Services | Microsoft Docs'
description: Dans ce tutoriel, découvrez comment configurer des rôles d’utilisateur et d’administrateur Azure Analysis Services à l’aide du Portail Azure ou de SQL Server Management Studio.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: owend
ms.reviewer: owend
ms.openlocfilehash: aea2196671a136145671b977a6d925849b635b73
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92018693"
---
# <a name="tutorial-configure-server-administrator-and-user-roles"></a>Tutoriel : Configurer l’administrateur du serveur et les rôles d’utilisateur

 Dans ce tutoriel, vous utiliserez SQL Server Management Studio (SSMS) pour vous connecter à votre serveur dans Azure afin de configurer l’administrateur du serveur et les rôles de base de données model. Vous serez également initié au [Langage TMSL (Tabular Model Scripting Language)](/analysis-services/tabular-model-programming-compatibility-level-1200/tabular-model-programming-for-compatibility-level-1200). TMSL est un langage de script JSON pour les modèles tabulaires de niveau de compatibilité 1200 et supérieurs. Il peut être utilisé pour automatiser de nombreuses tâches de modélisation tabulaire. TMSL est souvent utilisé avec PowerShell, mais dans ce didacticiel, vous utilisez l’éditeur de requête XMLA dans SSMS. Durant ce tutoriel, vous effectuerez les tâches suivantes : 
  
> [!div class="checklist"]
> * Obtenir votre nom de serveur à partir du portail
> * Vous connecter à votre serveur à l’aide de SSMS
> * Ajouter un utilisateur ou un groupe au rôle d’administrateur du serveur 
> * Ajouter un utilisateur ou un groupe au rôle d’administrateur de la base de données model
> * Ajouter un nouveau rôle de base de données model et ajouter un utilisateur ou un groupe

Pour en savoir plus sur la sécurité utilisateur dans Azure Analysis Services, consultez [Authentification et autorisations utilisateur](../analysis-services-manage-users.md). 

## <a name="prerequisites"></a>Prérequis

- Un répertoire Azure Active Directory dans votre abonnement.
- Un [serveur Azure Analysis Services](../analysis-services-create-server.md) créé dans votre abonnement.
- Des autorisations d’[administrateur du serveur](../analysis-services-server-admins.md).
- [Ajoutez l’exemple de modèle adventureworks](../analysis-services-create-sample-model.md) à votre serveur.
- [Installez la dernière version de SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS).

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail](https://portal.azure.com/).

## <a name="get-server-name"></a>Obtenir le nom du serveur
Pour pouvoir vous connecter à votre serveur à partir de SSMS, vous avez besoin du nom du serveur. Vous pouvez obtenir le nom du serveur à partir du portail.

Dans **Portail Azure** > Serveur > **Présentation** > **Nom du serveur**, copiez le nom du serveur.
   
   ![Obtenir le nom du serveur dans Azure](./media/analysis-services-tutorial-roles/aas-copy-server-name.png)

## <a name="connect-in-ssms"></a>Se connecter dans SSMS

Pour les tâches restantes, vous utilisez SSMS pour vous connecter à votre serveur et le gérer.

1. Dans SSMS > **Explorateur d’objets**, cliquez sur **Se connecter** > **Analysis Services**.

    ![Se connecter](./media/analysis-services-tutorial-roles/aas-ssms-connect.png)

2. Dans la boîte de dialogue **Se connecter au serveur** sous **Nom du serveur**, collez le nom de serveur que vous avez copié à partir du portail. Dans **Authentification**, choisissez **Active Directory - Authentification universelle avec prise en charge de MFA**, puis entrez votre compte d’utilisateur et appuyez sur **Connexion**.
   
    ![Se connecter dans SSMS](./media/analysis-services-tutorial-roles/aas-connect-ssms-auth.png)

    > [!TIP]
    > Choisir Active Directory - Authentification universelle avec prise en charge de MFA est recommandé. Ce type de type d’authentification prend en charge l’[authentification multifacteur et non interactive](../../azure-sql/database/authentication-mfa-ssms-overview.md). 

3. Dans l’**Explorateur d’objets**, développez pour afficher les objets de serveur. Faites un clic droit pour afficher les propriétés de serveur.
   
    ![Explorateur d'objets dans SSMS](./media/analysis-services-tutorial-roles/aas-connect-ssms-objexp.png)

## <a name="add-a-user-account-to-the-server-administrator-role"></a>Ajouter un compte d’utilisateur au rôle d’administrateur du serveur

Dans cette tâche, vous ajoutez un compte d’utilisateur ou de groupe à partir d’Azure AD pour le rôle d’administrateur de serveur. Si vous spécifiez un groupe de sécurité, utilisez `obj:groupid@tenantid`.

1. Dans l’**Explorateur d’objets**, cliquez avec le bouton droit sur votre nom de serveur, puis cliquez sur **Propriétés**. 
2. Dans la fenêtre **Propriétés de Analysis Server**, cliquez sur **Sécurité** > **Ajouter**.
3. Dans la fenêtre **Sélectionner un utilisateur ou un groupe**, entrez un compte d’utilisateur ou de groupe dans Azure AD, puis cliquez sur **Ajouter**. 
   
     ![Ajouter un administrateur de serveur](./media/analysis-services-tutorial-roles/aas-add-server-admin.png)

4. Cliquez sur **OK** pour fermer les **Propriétés de Analysis Server**.

    > [!TIP]
    > Vous pouvez également ajouter des administrateurs de serveur à l’aide des **Administrateurs Analysis Service** dans le portail. 

## <a name="add-a-user-to-the-model-database-administrator-role"></a>Ajouter un utilisateur au rôle d’administrateur de la base de données model

Dans cette tâche, vous ajoutez un compte d’utilisateur ou de groupe au rôle d’administrateur des ventes sur Internet qui existe déjà dans le modèle. Ce rôle dispose des autorisations de contrôle total (administrateur) pour l’exemple de base de données model adventureworks. Cette tâche utilise la commande TMSL [CreateOrReplace](/analysis-services/tmsl/createorreplace-command-tmsl) dans un script créé pour vous.

1. Dans l’**Explorateur d’objets**, développez **Bases de données** > **adventureworks** > **Rôles**. 
2. Cliquez avec le bouton droit de la souris sur **Administrateur des ventes sur Internet**, puis cliquez sur **Générer un script du rôle en tant que** > **CRÉER OU REMPLACER PAR** > **Nouvelle fenêtre d’éditeur de requête**.

    ![Nouvelle fenêtre d’éditeur de requête](./media/analysis-services-tutorial-roles/aas-add-db-admin.png)

3. Dans **XMLAQuery**, modifiez la valeur de **"memberName":** en un compte d’utilisateur ou de groupe dans votre Azure AD. Le compte avec lequel vous êtes connecté est inclut par défaut ; cependant, vous n’avez pas besoin d’ajouter votre propre compte car vous êtes déjà un administrateur du serveur.

    ![Script TMSL dans la requête XMLA](./media/analysis-services-tutorial-roles/aas-add-db-admin-script.png)

4. Appuyez sur **F5** pour exécuter le script.


## <a name="add-a-new-model-database-role-and-add-a-user-or-group"></a>Ajouter un nouveau rôle de base de données model et ajouter un utilisateur ou un groupe

Dans cette tâche, vous utilisez la commande [Créer](/analysis-services/tmsl/create-command-tmsl) dans un script TMSL pour créer un nouveau rôle Ventes sur Internet global, spécifier les autorisations de *lecture* pour le rôle et ajouter un compte d’utilisateur ou de groupe à partir d’Azure AD.

1. Dans l’**Explorateur d’objets**, cliquez avec le bouton droit sur **adventureworks**, puis cliquez sur **Nouvelle requête** > **XMLA**. 
2. Copiez et collez le script TMSL suivant dans l’éditeur de requête :

    ```JSON
    {
    "create": {
      "parentObject": {
        "database": "adventureworks",
       },
       "role": {
         "name": "Internet Sales Global",
         "description": "All users can query model data",
         "modelPermission": "read",
         "members": [
           {
             "memberName": "globalsales@adventureworks.com",
             "identityProvider": "AzureAD"
           }
         ]
       }
      }
    }
    ```

3. Remplacez la valeur d’objet `"memberName": "globalsales@adventureworks.com"` par un compte d’utilisateur ou de groupe dans Azure AD.
4. Appuyez sur **F5** pour exécuter le script.

## <a name="verify-your-changes"></a>Vérifiez vos modifications.

1. Dans l’**Explorateur d’objets**, cliquez sur le nom de votre serveur, puis cliquez sur **Actualiser** ou appuyez sur **F5**.
2. Développez **Bases de données** > **adventureworks** > **Rôles**. Vérifiez que les nouvelles modifications de compte d’utilisateur et de rôle ajoutées dans les tâches précédentes s’affichent.   

    ![Vérifier dans l’Explorateur d’objets](./media/analysis-services-tutorial-roles/aas-connect-ssms-verify.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez les comptes d’utilisateur ou de groupe et les rôles. Pour ce faire, utilisez **Propriétés du rôle** > **Appartenance** pour supprimer des comptes d’utilisateur, ou cliquez sur un rôle avec le bouton droit de la souris, puis cliquez sur **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez appris comment vous connecter à votre serveur Azure Analysis Services et comment explorer les exemples de bases de données model adventureworks et les propriétés dans SSMS. Vous avez également appris à utiliser des scripts SSMS et TMSL pour ajouter des utilisateurs ou des groupes à des rôles existants et nouveaux. Maintenant que vous disposez d’autorisations d’utilisateur configurées pour votre serveur et votre exemple de base de données model, vous ainsi que les autres utilisateurs pouvez vous y connecter à l’aide d’applications clientes telles que Power BI. Pour en savoir plus, passez au didacticiel suivant. 

> [!div class="nextstepaction"]
> [Tutoriel : Se connecter avec Power BI Desktop](analysis-services-tutorial-pbid.md)