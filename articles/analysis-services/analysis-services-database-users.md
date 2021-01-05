---
title: Gérer les rôles et les utilisateurs dans Azure Analysis Services | Microsoft Docs
description: Découvrez comment gérer les rôles et les utilisateurs sur un serveur Analysis Services dans Azure.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions
ms.openlocfilehash: 56f98d41fc73cdd2be0923de66a5af09c875a050
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92013591"
---
# <a name="manage-database-roles-and-users"></a>Gérer les rôles et les utilisateurs de base de données

Au niveau de la base de données du modèle, tous les utilisateurs doivent appartenir à un rôle. Les rôles définissent les utilisateurs disposant d’autorisations spécifiques pour la base de données du modèle. Tout utilisateur ou groupe de sécurité ajouté à un rôle doit avoir un compte dans un client Azure AD dans le même abonnement que le serveur. 

Le mode de définition des rôles est différent selon l’outil utilisé, mais l’effet est le même.

Les autorisations des rôles incluent :
*  **Administrateur** : les utilisateurs disposent des autorisations complètes pour la base de données. Les rôles de base de données avec des autorisations d’administrateur sont différents des administrateurs de serveur.
*  **Processus** : les utilisateurs peuvent se connecter et effectuer des opérations de traitement sur la base de données et analyser les données des bases de données du modèle.
*  **Lecture** : les utilisateurs peuvent utiliser une application cliente pour se connecter et analyser les données des bases de données du modèle.

Lorsque vous créez un projet de modèle tabulaire, vous créez des rôles et ajoutez des utilisateurs ou des groupes à ces rôles à l’aide du Gestionnaire de rôles dans les projets Visual Studio avec Analysis Services. Lors du déploiement sur un serveur, utilisez SQL Server Management Studio (SSMS), [cmdlets PowerShell Analysis Services](/analysis-services/powershell/analysis-services-powershell-reference) ou [Tabular Model Scripting Language](/analysis-services/tmsl/tabular-model-scripting-language-tmsl-reference) (TMSL) pour ajouter ou supprimer des rôles et des membres utilisateur.

Lors de l'ajout d'**un groupe de sécurité**, utilisez `obj:groupid@tenantid`.

## <a name="to-add-or-manage-roles-and-users-in-visual-studio"></a>Pour ajouter ou gérer des rôles et des utilisateurs dans Visual Studio  
  
1.  Dans l’**Explorateur de modèle tabulaire**, cliquez avec le bouton droit sur **Rôles**.  
  
2.  Dans le **Gestionnaire de rôles**, cliquez sur **Nouveau**.  
  
3.  Entrez ensuite le nom du rôle.  
  
     Par défaut, le nom de rôle par défaut est numéroté de façon incrémentielle pour chaque nouveau rôle. Il est recommandé de saisir un nom qui identifie clairement le type de membre, par exemple, directeurs financiers ou spécialistes des ressources humaines.  
  
4.  Sélectionnez l’une des autorisations suivantes :  
  
    |Autorisation|Description|  
    |----------------|-----------------|  
    |**Aucun**|Les membres ne peuvent pas lire ou modifier le schéma de modèle et ne peuvent pas interroger les données.|  
    |**Lire**|Les membres peuvent interroger des données (selon les filtres de lignes) mais ne peuvent pas modifier le schéma de modèle.|  
    |**Lecture et traitement**|Les membres peuvent interroger des données (selon les filtres au niveau des lignes) et exécuter des processus et traiter toutes les opérations, mais ne peuvent pas modifier le schéma de modèle.|  
    |**Processus**|Les membres peuvent exécuter des processus et traiter toutes les opérations. Ils ne peuvent ni lire ou modifier le schéma de modèle, ni interroger les données.|  
    |**Administrateur**|Les membres peuvent modifier le schéma de modèle et interroger toutes les données.|   
  
5.  Si le rôle que vous créez a l’autorisation de Lecture ou de Lecture et processus, vous pouvez ajouter des filtres de lignes à l’aide d’une formule DAX. Cliquez sur l’onglet **Filtres de lignes**, sélectionnez une table, puis cliquez sur le champ **Filtre DAX**, puis tapez une formule DAX.
  
6.  Cliquez sur **Membres** > **Ajouter externe**.  
  
8.  Dans **Ajouter un membre externe**, entrez les utilisateurs ou groupes dans votre Azure AD client par adresse e-mail. Une fois que vous avez cliqué sur OK et fermé le Gestionnaire de rôles, les rôles et les membres du rôle s’affichent dans l’Explorateur de modèles tabulaires. 
 
     ![Rôles et les utilisateurs dans l’Explorateur de modèles tabulaires](./media/analysis-services-database-users/aas-roles-tmexplorer.png)

9. Déployez votre serveur Azure Analysis Services.


## <a name="to-add-or-manage-roles-and-users-in-ssms"></a>Pour ajouter ou gérer des rôles et des utilisateurs dans SSMS

Pour ajouter des rôles et des utilisateurs à une base de données du modèle déployée, vous devez être connecté au serveur en tant qu’administrateur de serveur ou déjà dans un rôle de base de données avec des autorisations d’administrateur.

1. Dans Object Exporer, cliquez avec le bouton droit sur **Rôles** > **Nouveau rôle**.

2. Dans **Créer un rôle**, entrez un nom de rôle et une description.

3. Sélectionnez une autorisation.

   |Autorisation|Description|  
   |----------------|-----------------|  
   |**Contrôle total (administrateur)**|Les membres peuvent modifier le schéma de modèle, le processus et interroger toutes les données.| 
   |**Base de données de processus**|Les membres peuvent exécuter des processus et traiter toutes les opérations. Impossible de modifier le schéma de modèle et d'interroger les données.|  
   |**Lire**|Les membres peuvent interroger des données (selon les filtres de lignes) mais ne peuvent pas modifier le schéma de modèle.|  
  
4. Cliquez sur **Appartenance**, puis entrez un utilisateur ou un groupe dans votre client Azure AD par adresse e-mail.

     ![Ajouter un utilisateur](./media/analysis-services-database-users/aas-roles-adduser-ssms.png)

5. Si le rôle que vous créez a l’autorisation de Lecture, vous pouvez ajouter des filtres de lignes à l’aide d’une formule DAX. Cliquez sur **Filtres de lignes**, sélectionnez une table, puis tapez une formule DAX dans le champ **Filtre DAX**. 

## <a name="to-add-roles-and-users-by-using-a-tmsl-script"></a>Pour ajouter des rôles et des utilisateurs à l’aide d’un script TMSL

Vous pouvez exécuter un script TMSL dans la fenêtre XMLA dans SSMS ou à l’aide de PowerShell. Utilisez la commande [CreateOrReplace](/analysis-services/tmsl/createorreplace-command-tmsl) et l’objet [Rôles](/analysis-services/tmsl/roles-object-tmsl).

**Exemple de script TMSL**

Dans cet exemple, un utilisateur externe B2B et un groupe sont ajoutés au rôle d’analyste avec des autorisations de Lecture pour la base de données SalesBI. L’utilisateur externe et le groupe doivent être dans le même client Azure AD.

```
{
  "createOrReplace": {
    "object": {
      "database": "SalesBI",
      "role": "Analyst"
    },
    "role": {
      "name": "Users",
      "description": "All allowed users to query the model",
      "modelPermission": "read",
      "members": [
        {
          "memberName": "user1@contoso.com",
          "identityProvider": "AzureAD"
        },
        {
          "memberName": "group1@adventureworks.com",
          "identityProvider": "AzureAD"
        }
      ]
    }
  }
}
```

## <a name="to-add-roles-and-users-by-using-powershell"></a>Pour ajouter des rôles et des utilisateurs à l’aide de Powershell

Le module [SqlServer](/analysis-services/powershell/analysis-services-powershell-reference) fournit des applets de commande de gestion de bases de données spécifiques à chaque tâche, ainsi que l’applet de commande Invoke-ASCmd à usage général, qui accepte un script ou une requête utilisant le langage de script de modèle tabulaire (TMSL). Les applets de commande suivantes sont utilisées pour la gestion des utilisateurs et des rôles de bases de données.
  
|Applet de commande|Description|
|------------|-----------------| 
|[Add-RoleMember](/powershell/module/sqlserver/Add-RoleMember)|Ajoute un membre à un rôle de base de données.| 
|[Remove-RoleMember](/powershell/module/sqlserver/remove-rolemember)|Supprime un membre d’un rôle de base de données.|   
|[Invoke-ASCmd](/powershell/module/sqlserver/invoke-ascmd)|Exécute un script TMSL.|

## <a name="row-filters"></a>Filtres de lignes  

Les filtres de lignes définissent les lignes d’une table qui peuvent être interrogées par les membres d’un rôle donné. Les filtres de lignes sont définis pour chaque table dans un modèle à l'aide de formules DAX.  
  
Les filtres de lignes peuvent être définis uniquement pour les rôles avec des autorisations de Lecture et de Lecture et processus. Par défaut, si un filtre de lignes n’est pas défini pour une table en particulier, les membres peuvent interroger toutes les lignes de la table, sauf si le filtrage croisé s’applique à partir d’une autre table.
  
 Les filtres de lignes nécessitent une formule DAX, qui doit correspondre à une valeur TRUE/FALSE, pour définir les lignes qui peuvent être interrogées par les membres de ce rôle en particulier. Les lignes non incluses dans la formule DAX ne peuvent pas être interrogées. Par exemple, la table Clients avec l’expression de filtres de la ligne suivante, *=Customers [Country] = "USA"* , les membres du rôle Ventes peuvent voir uniquement les clients aux États-Unis.  
  
Les filtres de lignes s’appliquent aux lignes spécifiées et aux lignes connexes. Lorsqu'une table contient plusieurs relations, les filtres appliquent la sécurité de la relation qui est active. Les filtres de lignes sont croisés avec d’autres filtres de lignes définis pour les tables associées, par exemple :  
  
|Table de charge de travail|Expression DAX|  
|-----------|--------------------|  
|Région|=Region[Country]="USA"|  
|ProductCategory|=ProductCategory[Name]="Bicycles"|  
|Transactions|=Transactions[Year]=2016|  
  
 L’effet net est que les membres peuvent interroger les lignes de données pour lesquelles le client réside aux États-Unis, la catégorie de produits est bicyclettes et l’année est 2016. Les utilisateurs ne peuvent pas interroger les transactions en dehors des États-Unis, qui ne sont pas des bicyclettes ou les transactions hors de 2016, sauf si ils sont membres d’un autre rôle qui accorde ces autorisations.
  
 Vous pouvez utiliser le filtre, *= FALSE()* , pour refuser l’accès à toutes les lignes pour une table entière.

## <a name="next-steps"></a>Étapes suivantes

  [Gérer les administrateurs de serveur](analysis-services-server-admins.md)   
  [Gérer Azure Analysis Services avec PowerShell](analysis-services-powershell.md)  
  [Langage TMSL (Tabular Model Scripting Language)](/analysis-services/tmsl/tabular-model-scripting-language-tmsl-reference)