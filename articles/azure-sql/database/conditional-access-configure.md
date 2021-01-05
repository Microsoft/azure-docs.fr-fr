---
title: Accès conditionnel
description: Découvrez comment configurer l’accès conditionnel pour Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics.
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.topic: how-to
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.custom: sqldbrb=1
ms.date: 04/28/2020
tag: azure-synpase
ms.openlocfilehash: eae7d0a34d6bd76dae5998f05f9d64e0d40f7d9b
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321612"
---
# <a name="conditional-access-with-azure-sql-database-and-azure-synapse-analytics"></a>Accès conditionnel avec Azure SQL Database et Azure Synapse Analytics

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

[Azure SQL Database](sql-database-paas-overview.md), [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) et [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) prennent tous en charge l’accès conditionnel Microsoft.

Les étapes suivantes montrent comment configurer Azure SQL Database, Azure SQL Managed Instance ou Azure Synapse pour appliquer une stratégie d’accès conditionnel.  

## <a name="prerequisites"></a>Prérequis

- Vous devez configurer Azure SQL Database, Azure SQL Managed Instance ou le pool SQL dédié dans Azure Synapse pour prendre en charge l’authentification Azure Active Directory (Azure AD). Pour connaître la procédure spécifique, consultez [Configurer et gérer l’authentification Azure Active Directory avec SQL Database ou Azure Synapse](authentication-aad-configure.md).  
- Quand Multi-Factor Authentication est activé, vous devez vous connecter avec un outil pris en charge, tel que la dernière version de SQL Server Management Studio (SSMS). Pour plus d’informations, consultez [Configurer l’authentification multifacteur Azure SQL Database pour SQL Server Management Studio](authentication-mfa-ssms-configure.md).  

## <a name="configure-conditional-access"></a>Configurer un accès conditionnel

> [!NOTE]
> L’exemple ci-dessous utilise Azure SQL Database, mais vous devez sélectionner le produit dans lequel vous souhaitez configurer l’accès conditionnel.

1. Connectez-vous au portail Azure, sélectionnez **Azure Active Directory** , puis **Accès conditionnel**. Pour plus d’informations, consultez [Référence technique de l’accès conditionnel Azure Active Directory](../../active-directory/conditional-access/concept-conditional-access-conditions.md).  
   ![Panneau d’accès conditionnel](./media/conditional-access-configure/conditional-access-blade.png)

2. Dans le panneau des **stratégies d’accès conditionnel** , cliquez sur **Nouvelle stratégie** , fournissez un nom, puis cliquez sur **Configure rules** (Configurer les règles).  
3. Sous **Affectations** , sélectionnez **Utilisateurs et groupes** , cochez **Sélectionner des utilisateurs et des groupes** , puis sélectionnez l’utilisateur ou le groupe pour l’accès conditionnel. Cliquez sur **Sélectionner** , puis sur **Terminé** pour valider la sélection.  
   ![sélectionner des utilisateurs et des groupes](./media/conditional-access-configure/select-users-and-groups.png)  

4. Sélectionnez **Applications cloud** , puis cliquez sur **Sélectionner des applications**. Vous voyez toutes les applications disponibles pour l’accès conditionnel. Sélectionnez **Azure SQL Database** , cliquez en bas sur **Sélectionner** , puis sur **Terminé**.  
   ![sélectionner SQL Database](./media/conditional-access-configure/select-sql-database.png)  
   Si vous ne trouvez pas **Azure SQL Database** répertorié dans la troisième capture d’écran ci-dessous, effectuez les étapes suivantes :
   - Connectez-vous à votre base de données dans Azure SQL Database à l’aide de SSMS et d’un compte administrateur Azure AD.  
   - Exécutez `CREATE USER [user@yourtenant.com] FROM EXTERNAL PROVIDER`.  
   - Connectez-vous à Azure AD et vérifiez qu’Azure SQL Database, Azure SQL Managed Instance ou Azure Synapse sont répertoriés dans les applications de votre instance Azure AD.  

5. Sélectionnez **Contrôles d’accès** , **Accorder** , puis cochez la stratégie que vous souhaitez appliquer. Pour cet exemple, nous sélectionnons **Imposer l’authentification multifacteur**.  
   ![sélectionner Accorder l’accès](./media/conditional-access-configure/grant-access.png)  

## <a name="summary"></a>Résumé

L’application sélectionnée (Azure SQL Database) à l’aide d’Azure AD Premium applique maintenant la stratégie d’accès conditionnel sélectionnée **Imposer l’authentification multifacteur**.

Pour toute question sur Azure SQL Database et Azure Synapse concernant l’authentification multifacteur, contactez <MFAforSQLDB@microsoft.com>.  

## <a name="next-steps"></a>Étapes suivantes  

Pour un didacticiel, consultez [Sécuriser votre base de données dans SQL Database](secure-database-tutorial.md).