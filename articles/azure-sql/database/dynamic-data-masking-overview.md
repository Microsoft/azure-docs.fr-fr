---
title: Masquage dynamique des données
description: Le masquage des données dynamiques limite l’exposition des données sensibles en les masquant aux utilisateurs sans privilèges pour Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 08/04/2020
tags: azure-synpase
ms.openlocfilehash: f8d352dac98f953f7f6d8033d0d9e1376c4da313
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532242"
---
# <a name="dynamic-data-masking"></a>Masquage dynamique des données 
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics prennent en charge le masquage dynamique des données. Le masquage dynamique des données limite l’exposition des données sensibles en les masquant aux utilisateurs sans privilège. 

Le masquage dynamique des données permet d’empêcher les accès non autorisés à des données sensibles. Pour cela, les clients peuvent indiquer la quantité de données sensibles à exposer avec un impact minimal sur la couche Application. Il s’agit d’une fonctionnalité de sécurité basée sur des stratégies qui masque les données sensibles dans le jeu de résultats d’une requête sur des champs de base de données désignés (les données dans la base de données ne sont pas modifiées).

Par exemple, un représentant du centre d’appel peut identifier les appelants par plusieurs chiffres de leur numéro de carte de crédit, mais ces éléments de données ne doivent pas être divulgués entièrement au conseiller du service. Une règle de masquage peut être définie pour ne faire apparaître que les quatre derniers chiffres du numéro de carte de crédit dans l’ensemble de résultats de chaque requête. Autre exemple, un masque de données approprié peut être défini pour protéger les données personnelles, de telle sorte qu'un développeur puisse interroger les environnements de production à des fins de dépannage sans aller à l'encontre des réglementations de conformité.

## <a name="dynamic-data-masking-basics"></a>Principes de base du masquage des données dynamiques

Pour définir une stratégie de masquage de données dynamiques sur le portail Azure, sélectionnez le panneau **Dynamic Data Masking** sous **Sécurité** dans votre volet de configuration SQL Database. Cette fonctionnalité ne peut pas être définie à l'aide de SQL Managed Instance (utilisez PowerShell ou l'API REST). Pour plus d’informations, consultez [Dynamic Data Masking](/sql/relational-databases/security/dynamic-data-masking).

### <a name="dynamic-data-masking-permissions"></a>Autorisations du masquage des données dynamiques

Le masquage des données dynamiques peut être configuré par les rôles d’administrateur, d’administrateur de serveur ou [Gestionnaire de la sécurité SQL](../../role-based-access-control/built-in-roles.md#sql-security-manager) Azure SQL Database.

### <a name="dynamic-data-masking-policy"></a>Stratégie de masquage des données dynamiques

* **Utilisateurs SQL exclus du masquage** : ensemble d’utilisateurs SQL ou d’identités Azure AD qui obtiennent des données non masquées dans les résultats de requête SQL. Les utilisateurs possédant des privilèges administrateur sont toujours exclus du masquage et voient les données d’origine sans masque.
* **Règles de masquage** : ensemble de règles qui définissent les champs désignés à masquer et la fonction de masquage à utiliser. Les champs désignés peuvent être définis avec un nom de schéma de base de données, un nom de table et un nom de colonne.
* **Fonctions de masquage** : ensemble de méthodes qui contrôlent l'exposition des données dans différents scénarios.

| Fonction de masquage | Logique de masquage |
| --- | --- |
| **Par défaut** |**Masquage complet en fonction des types de données des champs désignés**<br/><br/>• Utilisez XXXX ou moins de X si la taille du champ est inférieure à 4 caractères pour les types de données String (nchar, ntext, nvarchar).<br/>• Utilisez la valeur zéro pour les types de données numériques (bigint, bit, decimal, int, money, numeric, smallint, smallmoney, tinyint, float, real).<br/>• Utilisez 01-01-1900 pour les types de données date/heure (date, datetime2, datetime, datetimeoffset, smalldatetime, time).<br/>• Pour la variante SQL, la valeur par défaut du type actuel est utilisé.<br/>• Pour XML, le document \<masked/> est utilisé.<br/>• Utilisez une valeur vide pour les types de données spéciaux (table timestamp, hierarchyid, GUID, binary, image, varbinary spatial). |
| **Carte de crédit** |**Méthode de masquage qui affiche les quatre derniers chiffres des champs de masquage désignés** et ajoute une chaîne constante comme préfixe sous la forme d’une carte de crédit.<br/><br/>XXXX-XXXX-XXXX-1234 |
| **E-mail** |**Méthode de masquage qui affiche la première lettre et remplace le domaine par XXX.com** avec en préfixe une chaîne constante sous la forme d’une adresse de messagerie.<br/><br/>aXX@XXXX.com |
| **Nombre aléatoire** |**Méthode de masquage qui génère un nombre aléatoire** selon les limites sélectionnées et les types de données réels. Si les limites désignées sont égales, la fonction de masquage est un nombre constant.<br/><br/>![Screenshot that shows the masking method for generating a random number.](./media/dynamic-data-masking-overview/1_DDM_Random_number.png) |
| **Texte personnalisé** |**Méthode de masquage qui affiche le premier et le dernier caractères** et ajoute une chaîne de remplissage personnalisée entre les deux. Si la chaîne d’origine est plus courte que le préfixe et le suffixe affichés, seule la chaîne de remplissage est utilisée. <br/>préfixe[remplissage]suffixe<br/><br/>![Volet de navigation](./media/dynamic-data-masking-overview/2_DDM_Custom_text.png) |

<a name="Anchor1"></a>

### <a name="recommended-fields-to-mask"></a>Champs à masquer recommandés

Le moteur de recommandations DDM signale certains champs de votre base de données en tant que champs potentiellement sensibles, qui peuvent être de bons candidats au masquage. Dans le volet Masquage des données dynamiques du portail, vous verrez les colonnes recommandées pour votre base de données. Il vous suffit de cliquer sur **Ajouter un masque** pour une ou plusieurs colonnes, puis de cliquer sur **Enregistrer** pour appliquer un masque à ces champs.

## <a name="set-up-dynamic-data-masking-for-your-database-using-powershell-cmdlets"></a>Configurer le masquage des données dynamiques pour votre base de données à l’aide des cmdlets PowerShell

### <a name="data-masking-policies"></a>Stratégies de masquage des données

- [Get-AzSqlDatabaseDataMaskingPolicy](/powershell/module/az.sql/Get-AzSqlDatabaseDataMaskingPolicy)
- [Set-AzSqlDatabaseDataMaskingPolicy](/powershell/module/az.sql/Set-AzSqlDatabaseDataMaskingPolicy)

### <a name="data-masking-rules"></a>Règles de masquage de données

- [Get-AzSqlDatabaseDataMaskingRule](/powershell/module/az.sql/Get-AzSqlDatabaseDataMaskingRule)
- [New-AzSqlDatabaseDataMaskingRule](/powershell/module/az.sql/New-AzSqlDatabaseDataMaskingRule)
- [Remove-AzSqlDatabaseDataMaskingRule](/powershell/module/az.sql/Remove-AzSqlDatabaseDataMaskingRule)
- [Set-AzSqlDatabaseDataMaskingRule](/powershell/module/az.sql/Set-AzSqlDatabaseDataMaskingRule)

## <a name="set-up-dynamic-data-masking-for-your-database-using-the-rest-api"></a>Configurer le masquage dynamique des données pour votre base de données à l’aide de l’API REST

Vous pouvez utiliser l’API REST pour gérer par programme les règles et stratégies de masquage des données. L’API REST publiée prend en charge les opérations suivantes :

### <a name="data-masking-policies"></a>Stratégies de masquage des données

- [Créer ou mettre à jour](/rest/api/sql/datamaskingpolicies/createorupdate) : Crée ou met à jour une stratégie de masquage des données de la base de données.
- [Obtenir](/rest/api/sql/datamaskingpolicies/get) : Fourni une stratégie de masquage des données de la base de données. 

### <a name="data-masking-rules"></a>Règles de masquage de données

- [Créer ou mettre à jour](/rest/api/sql/datamaskingrules/createorupdate) : Crée ou met à jour une règle de masquage des données de la base de données.
- [Liste par base de données](/rest/api/sql/datamaskingrules/listbydatabase) : Fournit une liste des règles de masquage des données de la base de données.
