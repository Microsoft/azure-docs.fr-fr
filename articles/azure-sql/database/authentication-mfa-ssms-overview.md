---
title: Utilisation de l’authentification multifacteur Azure Active Directory
description: Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics prennent en charge les connexions depuis SQL Server Management Studio (SSMS) à l'aide de l'authentification universelle Active Directory.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
ms.custom: seoapril2019, has-adal-ref, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 09/28/2020
tags: azure-synapse
ms.openlocfilehash: 9afad44bcf67478a81e75c17d0ff8ffc6d8c65aa
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94841126"
---
# <a name="using-multi-factor-azure-active-directory-authentication"></a>Utilisation de l’authentification multifacteur Azure Active Directory
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics prennent en charge les connexions à partir de [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) utilisant *Azure Active Directory - Authentification universelle avec MFA*. Cet article décrit les différences entre les diverses options d’authentification ainsi que les limitations associées à l’utilisation de l’authentification universelle dans Azure Active Directory (Azure AD) pour Azure SQL.

**Télécharger la dernière version de SSMS** : sur l’ordinateur client, téléchargez la dernière version de SSMS à partir de la page [Télécharger SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms). 

[!INCLUDE[ssms-connect-azure-ad](../includes/ssms-connect-azure-ad.md)]


Pour toutes les fonctionnalités abordées dans cet article, utilisez au minimum la version 17.2 (juillet 2017). La boîte de dialogue de connexion la plus récente doit ressembler à l’image suivante :

  ![Capture d’écran de la boîte de dialogue Connexion au serveur dans SQL Server Management Studio, montrant les paramètres pour le type de serveur, le nom du serveur et l’authentification.](./media/authentication-mfa-ssms-overview/1mfa-universal-connect.png)

## <a name="authentication-options"></a>Options d’authentification

Il existe deux modèles d’authentification non interactifs pour Azure ZD, qui peuvent être utilisés dans diverses applications (ADO.NET, JDBC, ODBC, etc.). Ces deux méthodes n’entraînent jamais l’affichage de boîtes de dialogue contextuelles :

- `Azure Active Directory - Password`
- `Azure Active Directory - Integrated`

La méthode interactive qui prend également en charge Azure AD Multi-Factor Authentication (MFA) est la suivante : 

- `Azure Active Directory - Universal with MFA`

L’authentification multifacteur (MFA) Azure AD contribue à sécuriser l’accès aux données et aux applications tout en répondant à la demande des utilisateurs souhaitant un processus de connexion simple. Il assure une authentification forte avec une gamme d’options de vérification simples (appel téléphonique, SMS, cartes à puce avec code PIN ou notification d’application mobile) pour permettre aux utilisateurs de choisir leur méthode préférée. L’authentification multifacteur (MFA) interactive avec Azure AD peut afficher une boîte de dialogue contextuelle de validation.

Pour une description d'Azure AD Multi-Factor Authentication, consultez la rubrique [Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md).
Pour les étapes de configuration, consultez [Configuration de l’authentification multifacteur pour les bases de données Azure SQL pour SQL Server Management Studio](authentication-mfa-ssms-configure.md).

### <a name="azure-ad-domain-name-or-tenant-id-parameter"></a>Nom de domaine Azure AD et paramètre d’ID de locataire

Depuis la [version 17 de SSMS](/sql/ssms/download-sql-server-management-studio-ssms), les utilisateurs qui sont importés dans l’instance active Azure AD à partir d’autres instances Azure Active Directory comme utilisateurs invités peuvent fournir le nom de domaine Azure AD ou l’ID de locataire au moment de se connecter. Les utilisateurs invités incluent les utilisateurs qui sont invités à partir d’autres annuaires Azure AD, de comptes Microsoft, tels qu’outlook.com, hotmail.com et live.com, ou d’autres comptes, comme gmail.com. Grâce à ces informations, l’authentification `Azure Active Directory - Universal with MFA` peut identifier l’autorité d’authentification appropriée. Cette option est également requise pour prendre en charge les comptes Microsoft (MSA) tels qu’outlook.com, hotmail.com, live.com, ou les comptes non-MSA. 

Tous les utilisateurs invités qui souhaitent être authentifiés à l’aide de l’authentification universelle doivent entrer leur nom de domaine ou ID de locataire Azure AD. Ce paramètre représente le nom de domaine ou l’ID de locataire Azure AD auquel le serveur logique Azure SQL est associé. Par exemple, si le serveur logique SQL est associé au domaine Azure AD `contosotest.onmicrosoft.com`, où l’utilisateur `joe@contosodev.onmicrosoft.com` est hébergé comme utilisateur importé du domaine Azure AD `contosodev.onmicrosoft.com`, le nom de domaine nécessaire à l’authentification de cet utilisateur est `contosotest.onmicrosoft.com`. Quand l’utilisateur est un utilisateur natif de l’instance Azure AD associée au serveur logique SQL et qu’il n’a pas de compte MSA, aucun nom de domaine ou ID de locataire n’est nécessaire. Pour entrer le paramètre (à partir de SSMS version 17.2) :


1. Ouvrez une connexion dans SSMS. Entrez votre nom de serveur, puis sélectionnez **Azure Active Directory - Authentification universelle avec MFA**. Ajoutez le **nom d’utilisateur** avec lequel vous souhaitez vous connecter.
1. Cochez la case **Options**, puis accédez à l’onglet **Propriétés de connexion**. Dans la boîte de dialogue **Connexion à une base de données**, complétez la boîte de dialogue pour votre base de données. Cochez la case **Nom du domaine AD ou ID de locataire** et fournissez l’autorité d’authentification, telle que le nom de domaine (**contosotest.onmicrosoft.com**) ou le GUID de l’ID de locataire. 

   ![Capture d’écran de l’onglet Propriétés de connexion mettant en évidence les paramètres pour « Connexion à une base de données » et « Nom du domaine AD ou ID de locataire ».](./media/authentication-mfa-ssms-overview/mfa-tenant-ssms.png)

Si vous exécutez SSMS 18.x ou une version ultérieure, le nom de domaine AD ou l’ID de locataire n’est plus nécessaire pour les utilisateurs invités, car la version 18.x ou ultérieure le reconnaît automatiquement.

   ![Capture d’écran de l’onglet Propriétés de connexion dans la boîte de dialogue Connexion au serveur dans SSMS. « MyDatabase » est sélectionné dans le champ Connexion à une base de données.](./media/authentication-mfa-ssms-overview/mfa-no-tenant-ssms.png)

### <a name="azure-ad-business-to-business-support"></a>Prise en charge d’Azure AD B2B

> [!IMPORTANT]
> La prise en charge des utilisateurs invités se connectant à Azure SQL Database, SQL Managed Instance et Azure Synapse sans nécessairement faire partie d’un groupe est actuellement en **préversion publique**. Pour plus d’informations, consultez [Créer des utilisateurs invités Azure AD et les définir comme administrateurs Azure AD](authentication-aad-guest-users.md).

Les utilisateurs Azure AD pris en charge pour les scénarios Azure AD B2B comme utilisateurs invités (voir [Qu’est-ce que la collaboration Azure AD B2B ?](../../active-directory/external-identities/what-is-b2b.md)) ne peuvent se connecter à SQL Database et Azure Synapse qu’en tant que membres d’un groupe créé dans l’instance Azure AD associée et mappés manuellement à l’aide de l’instruction Transact-SQL [CREATE USER (Transact-SQL)](/sql/t-sql/statements/create-user-transact-sql) dans une base de données spécifique. Par exemple, si `steve@gmail.com` est invité à Azure AD `contosotest` (avec le domaine Azure AD `contosotest.onmicrosoft.com`), un groupe Azure AD, tel que `usergroup`, doit être créé dans l’annuaire Azure AD qui contient le membre `steve@gmail.com`. Ensuite, l’administrateur Azure AD SQL ou le propriétaire de la base de données Azure AD doit créer ce groupe pour une base de données spécifique (par exemple, `MyDatabase`) en exécutant l’instruction Transact-SQL `CREATE USER [usergroup] FROM EXTERNAL PROVIDER`. 

Une fois l’utilisateur de base de données créé, l’utilisateur `steve@gmail.com` peut se connecter à `MyDatabase` en utilisant l’option d’authentification SSMS `Azure Active Directory – Universal with MFA`. Par défaut, `usergroup` dispose uniquement de l’autorisation de connexion. Tout autre accès aux données doit être [accordé](/sql/t-sql/statements/grant-transact-sql) dans la base de données par un utilisateur disposant de privilèges suffisants. 

> [!NOTE]
> Pour SSMS 17.x, en utilisant `steve@gmail.com` comme utilisateur invité, vous devez cocher la case **Nom du domaine AD ou ID de locataire** et ajouter le nom de domaine AD `contosotest.onmicrosoft.com` dans la boîte de dialogue **Propriété de connexion**. L’option **Nom du domaine AD ou ID de locataire**  est prise en charge uniquement pour **Azure Active Directory - Authentification universelle avec MFA**. Sinon, la case à cocher est grisée.

## <a name="universal-authentication-limitations"></a>Limites de l'authentification universelle

- SSMS et SqlPackage.exe sont les seuls outils activés pour MFA par le biais de l’authentification universelle Active Directory.
- SSMS version 17.2 prend en charge l’accès simultané de plusieurs utilisateurs utilisant l’authentification universelle avec MFA. Pour SSMS versions 17.0 et 17.1, l’outil limite à un seul compte Azure Active Directory la connexion à une instance de SSMS avec l’authentification universelle. Pour vous connecter sous un autre compte Azure AD, vous devez utiliser une autre instance de SSMS. Cette restriction se limite à l’authentification universelle Active Directory ; vous pouvez vous connecter à un autre serveur en utilisant l’authentification `Azure Active Directory - Password`, l’authentification `Azure Active Directory - Integrated`, ou `SQL Server Authentication`.
- SSMS prend en charge l’authentification universelle Active Directory pour la visualisation de l’Explorateur d’objets, de l’Éditeur de requête et du magasin de requêtes.
- SSMS version 17.2 fournit une prise en charge de l’Assistant DacFx pour l’exportation, l’extraction et le déploiement de la base de données. Une fois qu’un utilisateur spécifique est authentifié via la boîte de dialogue d’authentification initiale à l’aide de l’authentification universelle, l’Assistant DacFx fonctionne de la même manière que pour toutes les autres méthodes d’authentification.
- Le Concepteur de tables SSMS ne prend pas en charge l’authentification universelle.
- Il n’existe aucune configuration logicielle supplémentaire nécessaire pour l’authentification universelle Active Directory, sauf que vous devez utiliser une version prise en charge de SSMS.  
- Pour obtenir la dernière version de la Bibliothèque d’authentification Active Directory (ADAL) pour l’authentification universelle, consultez le lien suivant : [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).  

## <a name="next-steps"></a>Étapes suivantes

- Pour les étapes de configuration, consultez [Configuration de l’authentification multifacteur pour les bases de données Azure SQL pour SQL Server Management Studio](authentication-mfa-ssms-configure.md).
- Octroyer à d'autres utilisateurs l'accès à votre base de données : [Authentification et autorisation SQL Database : octroyer l'accès](logins-create-manage.md)  
- Assurez-vous que les autres utilisateurs peuvent se connecter via le pare-feu : [Configurer une règle de pare-feu au niveau du serveur à l'aide du portail Azure](firewall-configure.md)  
- [Configurer et gérer l’authentification Azure Active Directory avec SQL Database ou Azure Synapse](authentication-aad-configure.md)
- [Créer des utilisateurs invités Azure AD et les définir comme administrateurs Azure AD](authentication-aad-guest-users.md) 
- [Microsoft SQL Server Data-Tier Application Framework (17.0.0 GA)](https://www.microsoft.com/download/details.aspx?id=55088)  
- [SQLPackage.exe](/sql/tools/sqlpackage)  
- [Importer un fichier BACPAC dans une nouvelle base de données](database-import.md)  
- [Exporter une base de données vers un fichier BACPAC](database-export.md)  
- Interface C# [Interface IUniversalAuthProvider](/dotnet/api/microsoft.sqlserver.dac.iuniversalauthprovider)  
- Quand vous utilisez **Azure Active Directory - Authentification universelle avec MFA**, le traçage ADAL est disponible à compter de [SSMS 17.3](/sql/ssms/download-sql-server-management-studio-ssms). Le suivi ADAL étant désactivé par défaut, vous pouvez l’activer comme suit : dans le menu **Outils**, choisissez **Options**, puis sous **Services Azure**, choisissez **Cloud Azure** et **Niveau de trace dans la fenêtre Sortie ADAL**, et activez **Sortie** dans le menu **Affichage**. Les suivis sont disponibles dans la fenêtre de sortie quand vous sélectionnez l’option **Azure Active Directory**.