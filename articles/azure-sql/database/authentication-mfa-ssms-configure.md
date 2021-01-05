---
title: Configurer l’authentification multifacteur
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Découvrez l’utilisation de l’authentification multifacteur avec SSMS pour Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: has-adal-ref, sqldbrb=3
ms.devlang: ''
ms.topic: how-to
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 08/27/2019
ms.openlocfilehash: 4f90299daed46d06dad9ab37103e3b8f53763ed4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454374"
---
# <a name="configure-multi-factor-authentication-for-sql-server-management-studio-and-azure-ad"></a>Configurer l’authentification multifacteur pour SQL Server Management Studio et Azure AD
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Cet article montre comment utiliser l’authentification multifacteur (MFA) d’Azure Active Directory (Azure AD) avec SQL Server Management Studio (SSMS). Vous pouvez utiliser Azure AD MFA au moment de la connexion de SSMS ou SqlPackage.exe à [Azure SQL Database](sql-database-paas-overview.md), [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) et [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md). Pour une vue d’ensemble de l’authentification multifacteur, consultez [Authentification universelle avec SQL Database, SQL Managed Instance et Azure Synapse (prise en charge de SSMS pour MFA)](../database/authentication-mfa-ssms-overview.md).

> [!IMPORTANT]
> Dans cet article, les bases de données d’Azure SQL Database, d’Azure SQL Managed Instance et d’Azure Synapse sont collectivement désignées sous le nom de « bases de données », et le terme « serveur » fait référence au [serveur](logical-servers.md) qui héberge les bases de données d’Azure SQL Database et d’Azure Synapse.

## <a name="configuration-steps"></a>Configuration

1. **Configurer un annuaire Azure Active Directory** : pour plus d’informations, consultez [Administration de votre annuaire Azure AD](/previous-versions/azure/azure-services/hh967611(v=azure.100)), [Intégration de vos identités locales à Azure Active Directory](../../active-directory/hybrid/whatis-hybrid-identity.md), [Ajout de votre nom de domaine personnalisé à Azure AD](https://azure.microsoft.com/blog/20../../windows-azure-now-supports-federation-with-windows-server-active-directory/), [Microsoft Azure now supports federation with Windows Server Active Directory](https://azure.microsoft.com/blog/20../../windows-azure-now-supports-federation-with-windows-server-active-directory/) (Microsoft Azure prend désormais en charge la fédération avec Windows Server Active Directory) et [Gestion d’Azure AD à l’aide de Windows PowerShell](/previous-versions/azure/jj151815(v=azure.100)).
2. **Configurer l'authentification MFA** : pour obtenir des instructions pas à pas, consultez [Présentation d'Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md) et [Accès conditionnel (MFA) avec Azure SQL Database et Data Warehouse](conditional-access-configure.md). L’accès conditionnel complet requiert un annuaire Premium Azure Active Directory. L’authentification MFA limitée est disponible avec un annuaire Azure AD standard.)
3. **Configurer Azure AD Authentication** : pour obtenir des instructions pas à pas, consultez [Connexion à SQL Database, SQL Managed Instance ou Azure Synapse à l’aide de l’authentification Azure Active Directory](authentication-aad-overview.md).
4. **Télécharger SSMS** : sur l’ordinateur client, téléchargez la dernière version de SSMS à partir de la page [Télécharger SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).

## <a name="connecting-by-using-universal-authentication-with-ssms"></a>Connexion à l’aide de l’authentification universelle avec SSMS

Les étapes suivantes montrent comment se connecter à l’aide de la dernière version de SSMS.

[!INCLUDE[ssms-connect-azure-ad](../includes/ssms-connect-azure-ad.md)]

1. Pour vous connecter à l’aide de l’authentification universelle, accédez à la boîte de dialogue **Se connecter au serveur** dans SQL Server Management Studio (SSMS), puis sélectionnez **Active Directory – Authentification universelle avec MFA**. (Si vous voyez **Authentification universelle Active Directory**, vous n’êtes pas en train d’utiliser la dernière version de SSMS.)

   ![Capture d’écran de l’onglet Propriétés de connexion dans la boîte de dialogue Connexion au serveur dans SSMS. « MyDatabase » est sélectionné dans la liste déroulante Connexion à une base de données.](./media/authentication-mfa-ssms-configure/mfa-no-tenant-ssms.png)  
2. Renseignez la zone **Nom d’utilisateur** avec les informations d’identification Azure Active Directory, selon le format `user_name@domain.com`.

   ![Capture d’écran des paramètres de la boîte de dialogue Connexion au serveur pour Type de serveur, Nom du serveur, Authentification et Nom d’utilisateur.](./media/authentication-mfa-ssms-configure/1mfa-universal-connect-user.png)
3. Si vous vous connectez en tant qu’utilisateur invité, vous n’avez plus besoin de renseigner les champs Nom de domaine AD ou ID de locataire pour les utilisateurs invités, car SSMS 18. x ou version ultérieure les renseigne automatiquement. Pour plus d’informations, consultez [Authentification universelle avec SQL Database, SQL Managed Instance et Azure Synapse (prise en charge de SSMS pour MFA)](../database/authentication-mfa-ssms-overview.md).

   ![Capture d’écran de l’onglet Propriétés de connexion dans la boîte de dialogue Connexion au serveur dans SSMS. « MyDatabase » est sélectionné dans la liste déroulante Connexion à une base de données.](./media/authentication-mfa-ssms-configure/mfa-no-tenant-ssms.png)

   En revanche, si vous vous connectez en tant qu’utilisateur invité à l’aide de SSMS 17.x ou version antérieure, vous devez cliquer sur **Options**, puis, dans la boîte de dialogue **Propriété de connexion**, renseigner les champs **Nom du domaine AD ou ID de locataire**.

   ![Capture d’écran de l’onglet Propriétés de connexion dans la boîte de dialogue Connexion au serveur dans SSMS. L’option Nom du domaine AD ou ID de locataire est renseignée.](./media/authentication-mfa-ssms-configure/mfa-tenant-ssms.png)

4. Sélectionnez **Options** et spécifiez la base de données dans la boîte de dialogue **Options**. Si l’utilisateur connecté est un utilisateur invité (c’est-à-dire joe@outlook.com), vous devez cocher la case et ajouter le nom de domaine AD ou l’ID de locataire actuel parmi les options. Voir [Authentification universelle avec SQL Database et Azure Synapse Analytics (prise en charge de SSMS pour l’authentification MFA)](../database/authentication-mfa-ssms-overview.md). Puis, cliquez sur **Se connecter**.  
5. Lorsque la boîte de dialogue **Connectez-vous à votre compte** s’affiche, spécifiez le compte et le mot de passe de votre identité Azure Active Directory. Aucun mot de passe n’est requis si un utilisateur fait partie d’un domaine fédéré avec Azure AD.

   ![Capture d’écran de la boîte de dialogue Connectez-vous à votre compte pour Azure SQL Database et Data Warehouse. Le compte et le mot de passe sont renseignés.](./media/authentication-mfa-ssms-configure/2mfa-sign-in.png)  

   > [!NOTE]
   > Pour l’authentification universelle avec un compte qui ne nécessite pas MFA, vous vous connectez à ce stade. Pour les utilisateurs nécessitant MFA, passez aux étapes suivantes :
   >  

6. Deux boîtes de dialogue de configuration de MFA peuvent s’afficher. Cette opération à exécuter une seule fois dépend de l’administrateur du paramètre de l’administrateur MFA et peut être par conséquent facultative. Pour un domaine avec MFA activée, cette étape est parfois prédéfinie (par exemple, le domaine exige que les utilisateurs utilisent une carte à puce et un code confidentiel).

   ![Capture d’écran de la boîte de dialogue Connectez-vous à votre compte pour Azure SQL Database et Data Warehouse, avec une invite à configurer une vérification de sécurité supplémentaire.](./media/authentication-mfa-ssms-configure/3mfa-setup.png)
  
7. La deuxième boîte de dialogue unique possible vous permet de sélectionner les détails de votre méthode d’authentification. Les options possibles sont configurées par votre administrateur.

   ![Capture d’écran de la boîte de dialogue Vérification de sécurité supplémentaire avec des options permettant de sélectionner et de configurer une méthode d’authentification.](./media/authentication-mfa-ssms-configure/4mfa-verify-1.png)  
8. Azure Active Directory vous envoie les informations de confirmation. Lorsque vous recevez le code de vérification, entrez-le dans la zone **Entrez le code de vérification**, puis cliquez sur **Connexion**.

   ![Capture d’écran de la boîte de dialogue Connectez-vous à votre compte pour Azure SQL Database et Data Warehouse, avec une invite à entrer un code de vérification.](./media/authentication-mfa-ssms-configure/5mfa-verify-2.png)  

Lorsque la vérification est terminée, SSMS se connecte normalement à condition que les informations d’identification soient valides et qu’un accès au pare-feu soit possible.

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble de l’authentification multifacteur, consultez [Authentification universelle avec SQL Database, SQL Managed Instance et Azure Synapse (prise en charge de SSMS pour MFA)](../database/authentication-mfa-ssms-overview.md).  
- Octroyer à d'autres utilisateurs l'accès à votre base de données : [Authentification et autorisation SQL Database : octroyer l'accès](logins-create-manage.md)  
- Assurez-vous que les autres utilisateurs peuvent se connecter via le pare-feu : [Configurer une règle de pare-feu au niveau du serveur à l’aide du portail Azure](./firewall-configure.md)  
- Quand vous utilisez l’authentification **Active Directory - Authentification universelle avec prise en charge de MFA**, le suivi ADAL est disponible à compter de [SSMS 17.3](/sql/ssms/download-sql-server-management-studio-ssms). Le suivi ADAL étant désactivé par défaut, vous pouvez l’activer comme suit : dans le menu **Outils**, choisissez **Options**, puis sous **Services Azure**, choisissez **Cloud Azure** et **Niveau de trace dans la fenêtre Sortie ADAL**, et activez **Sortie** dans le menu **Affichage**. Les suivis sont disponibles dans la fenêtre de sortie quand vous sélectionnez l’option **Azure Active Directory**.