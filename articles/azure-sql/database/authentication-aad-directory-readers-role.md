---
title: Rôle Lecteurs de répertoire dans Azure Active Directory pour Azure SQL
description: En savoir plus sur le rôle du lecteur de répertoire dans Azure AD pour Azure SQL.
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse
ms.topic: conceptual
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 08/14/2020
ms.openlocfilehash: 2374b1fb7f355b336c713a8a3240eacc8b1f188c
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92675063"
---
# <a name="directory-readers-role-in-azure-active-directory-for-azure-sql"></a>Rôle Lecteurs de répertoire dans Azure Active Directory pour Azure SQL

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

> [!NOTE]
> La fonctionnalité de cet article est en **préversion publique** .

Azure Active Directory (Azure AD) a introduit [l’utilisation des groupes cloud pour gérer les attributions de rôles dans Azure Active Directory (préversion)](../../active-directory/roles/groups-concept.md). Cela permet d’affecter des rôles Azure AD à des groupes.

Lors de l’activation d’une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) pour Azure SQL Database, Azure SQL Managed Instance ou Azure Synapse Analytics, le rôle [**Lecteurs de répertoire**](../../active-directory/roles/permissions-reference.md#directory-readers) Azure AD doit être attribué à l’identité pour permettre l’accès en lecture à l’ [API Graph Azure AD](../../active-directory/develop/active-directory-graph-api.md). L’identité managée de SQL Database et Azure Synapse est appelée identité du serveur. L’identité managée de SQL Managed Instance est appelée « identité de l’instance gérée » et est automatiquement affectée lors de la création de l’instance. Pour plus d’informations sur l’affectation d’une identité de serveur à une instance SQL Database ou Azure Synapse, consultez [Permettre aux principaux de service de créer des utilisateurs Azure AD](authentication-aad-service-principal.md#enable-service-principals-to-create-azure-ad-users).

Le rôle **Lecteurs de répertoire** est nécessaire pour :

- Créer des connexions Azure AD pour SQL Managed Instance
- L’emprunt d’identité d’utilisateurs Azure AD dans Azure SQL
- Migrer les utilisateurs SQL Server qui utilisent l’authentification Windows vers SQL Managed Instance avec l’authentification Azure AD (à l’aide de la commande [ALTER USER (Transact-SQL)](/sql/t-sql/statements/alter-user-transact-sql?view=azuresqldb-mi-current#d-map-the-user-in-the-database-to-an-azure-ad-login-after-migration))
- Changer l’administrateur Azure AD pour SQL Managed Instance
- Autoriser les [principaux de service (applications)](authentication-aad-service-principal.md) à créer des utilisateurs Azure AD dans Azure SQL

## <a name="assigning-the-directory-readers-role"></a>Attribution du rôle Lecteurs de répertoire

Pour affecter le rôle [**Lecteurs de répertoire**](../../active-directory/roles/permissions-reference.md#directory-readers) à une identité, un utilisateur disposant des autorisations [Administrateur général](../../active-directory/roles/permissions-reference.md#global-administrator--company-administrator) ou [Administrateur de rôle privilégié](../../active-directory/roles/permissions-reference.md#privileged-role-administrator) est nécessaire. Les utilisateurs qui gèrent ou déploient souvent SQL Database, SQL Managed Instance ou Azure Synapse n’ont peut-être pas accès à ces rôles à privilèges élevés. Cela peut souvent causer des complications pour les utilisateurs qui créent des ressources Azure SQL non planifiées ou qui ont besoin de l’aide de membres à rôle hautement privilégié qui sont souvent inaccessibles dans les grandes organisations.

Pour SQL Managed Instance, le rôle **Lecteurs de répertoire** doit être affecté à l’identité de l’instance gérée avant de pouvoir [configurer un administrateur Azure AD pour l’instance gérée](authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance). 

L’attribution du rôle **Lecteurs de répertoire** à l’identité du serveur n’est pas nécessaire pour SQL Database ou Azure Synapse lors de la configuration d’un administrateur Azure AD pour le serveur logique. Cependant, pour permettre la création d’un objet Azure AD dans SQL Database ou Azure Synapse pour le compte d’une application Azure AD, le rôle **Lecteurs d’annuaires** est requis. Si le rôle n’est pas affecté à l’identité du serveur logique SQL, la création d’utilisateurs Azure AD dans Azure SQL échoue. Pour plus d’informations, consultez [Principal de service Azure Active Directory avec Azure SQL](authentication-aad-service-principal.md).

## <a name="granting-the-directory-readers-role-to-an-azure-ad-group"></a>Octroi du rôle Lecteurs de répertoire à un groupe Azure AD

Actuellement en **préversion publique** , vous pouvez désormais demander à un [administrateur général](../../active-directory/roles/permissions-reference.md#global-administrator--company-administrator) ou [administrateur de rôle privilégié](../../active-directory/roles/permissions-reference.md#privileged-role-administrator) de créer un groupe Azure AD et d’affecter l’autorisation [**Lecteurs de répertoire**](../../active-directory/roles/permissions-reference.md#directory-readers) au groupe. Cela permettra l’accès à l’API Graph Azure AD pour les membres de ce groupe. En outre, les utilisateurs Azure AD qui sont propriétaires de ce groupe sont autorisés à assigner de nouveaux membres pour ce groupe, y compris les identités des serveurs logiques SQL Azure.

Cette solution nécessite toujours un utilisateur doté de privilèges élevés (administrateur général ou administrateur de rôle privilégié) pour créer un groupe et affecter des utilisateurs en tant qu’activité ponctuelle, mais les propriétaires du groupe Azure AD peuvent ensuite attribuer des membres supplémentaires. Cela élimine le besoin d’impliquer un utilisateur à privilèges élevés pour configurer chaque base de données SQL Database, instance SQL Managed Instance ou serveur Azure Synapse dans son locataire Azure AD.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Attribuer le rôle Lecteurs de répertoire à un groupe Azure AD et gérer les attributions de rôles](authentication-aad-directory-readers-role-tutorial.md)