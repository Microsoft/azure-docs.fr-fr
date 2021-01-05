---
title: 'Rôles personnalisés : Migrations en lignes de SQL Server vers une instance gérée SQL'
titleSuffix: Azure Database Migration Service
description: Apprenez à utiliser les rôles personnalisés pour les migrations en ligne de SQL Server vers Azure SQL Managed Instance.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: conceptual
ms.date: 10/25/2019
ms.openlocfilehash: dad02735228bb639981bf3f053a74f29d1944e5a
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961479"
---
# <a name="custom-roles-for-sql-server-to-azure-sql-managed-instance-online-migrations"></a>Rôles personnalisés pour les migrations en ligne de SQL Server vers Azure SQL Managed Instance

Azure Database Migration Service utilise un ID d’application pour interagir avec les services Azure. L’ID d’application nécessite le rôle Contributeur au niveau de l’abonnement (ce que de nombreux services de sécurité d’entreprise n’autorisent pas) ou la création de rôles personnalisés qui accordent les autorisations spécifiques requises par Azure Database Migration Service. Étant donné qu’il y a une limite de 2 000 rôles personnalisés dans Azure Active Directory, vous pouvez combiner toutes les autorisations requises spécifiquement par ID d’application en un ou deux rôles personnalisés, puis accorder à l’ID d’application le rôle personnalisé sur des objets ou des groupes de ressources spécifiques (plutôt qu’au niveau de l’abonnement). Si le nombre de rôles personnalisés n’est pas un problème, vous pouvez fractionner les rôles personnalisés par type de ressource, afin de créer trois rôles personnalisés au total, comme décrit ci-dessous.

La section AssignableScopes de la chaîne JSON de définition de rôle vous permet de contrôler l’emplacement des autorisations dans l’interface utilisateur **Ajouter une attribution de rôle** dans le portail. Vous souhaiterez probablement définir le rôle au niveau du groupe de ressources, voire de la ressource pour éviter d’encombrer l’interface utilisateur avec des rôles supplémentaires. Notez que cela n’effectue pas d’attribution de rôle réelle.

## <a name="minimum-number-of-roles"></a>Nombre minimal de rôles

Nous vous recommandons actuellement de créer au moins deux rôles personnalisés pour l’ID d’application, un au niveau de la ressource et l’autre au niveau de l’abonnement.

> [!NOTE]
> La dernière exigence de rôle personnalisée pourrait finir par être supprimée, car un nouveau code SQL Managed Instance est déployé sur Azure.

**Rôle personnalisé pour l’ID d’application**. Ce rôle est requis pour la migration d’Azure Database Migration Service au niveau de la *ressource* ou du *groupe de ressources* (pour plus d’informations sur l’ID d’application, consultez l’article [Utiliser le portail pour créer une application et un principal du service Azure AD pouvant accéder aux ressources](../active-directory/develop/howto-create-service-principal-portal.md)).

```json
{
  "Name": "DMS Role - App ID",
  "IsCustom": true,
  "Description": "DMS App ID access to complete MI migrations",
  "Actions": [
        "Microsoft.Storage/storageAccounts/read",
        "Microsoft.Storage/storageAccounts/listKeys/action",
        "Microsoft.Storage/storageaccounts/blobservices/read",
        "Microsoft.Storage/storageaccounts/blobservices/write",
        "Microsoft.Sql/managedInstances/read",
        "Microsoft.Sql/managedInstances/write",
        "Microsoft.Sql/managedInstances/databases/read",
        "Microsoft.Sql/managedInstances/databases/write",
        "Microsoft.Sql/managedInstances/databases/delete",
        "Microsoft.Sql/managedInstances/metrics/read",
        "Microsoft.DataMigration/locations/*",
        "Microsoft.DataMigration/services/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>/ResourceGroups/<StorageAccount_rg_name>",
    "/subscriptions/<subscription_id>/ResourceGroups/<ManagedInstance_rg_name>",
    "/subscriptions/<subscription_id>/ResourceGroups/<DMS_rg_name>",
  ]
}
```

**Rôle personnalisé pour l’ID d’application - abonnement**. Ce rôle est requis pour la migration d’Azure Database Migration Service au niveau de *l’abonnement*.

```json
{
  "Name": "DMS Role - App ID - Sub",
  "IsCustom": true,
  "Description": "DMS App ID access at subscription level to complete MI migrations",
  "Actions": [
        "Microsoft.Sql/locations/managedDatabaseRestoreAzureAsyncOperation/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>"
  ]
}
```

Le JSON ci-dessus doit être stocké dans trois fichiers texte. Vous pouvez utiliser les applets de commande AZ PowerShell, AzureRM ou Azure CLI pour créer les rôles à l’aide de **New-AzureRmRoleDefinition (AzureRM)** ou **New-AzRoleDefinition (AZ)** .

Pour plus d’informations, consultez l’article [Rôles personnalisés Azure](../role-based-access-control/custom-roles.md).

Après avoir créé ces rôles personnalisés, vous devez ajouter des attributions de rôles aux utilisateurs et aux ID d’application pour les ressources ou groupes de ressources appropriés :

* Le rôle « Rôle DMS - ID d’application » doit être accordé à l’ID d’application qui sera utilisé pour les migrations, ainsi qu’au niveau du compte de stockage, de l’instance Azure Database Migration Service et SQL Managed Instance.
* Le rôle « Rôle DMS - ID d’application » doit être accordé à l’ID d’application au niveau de l’abonnement (l’octroi au niveau de la ressource ou du groupe de ressources échoue). Cette exigence est temporaire jusqu’à ce qu’une mise à jour du code soit déployée.

## <a name="expanded-number-of-roles"></a>Nombre de rôles développé

Si le nombre de rôles personnalisés dans votre Azure Active Directory n’est pas un problème, nous vous recommandons de créer un total de trois rôles. Vous avez toujours besoin du rôle « Rôle DMS - ID d’application - Secondaire », mais le rôle « Rôle DMS - ID d’application » ci-dessus est fractionné par type de ressource en deux rôles différents.

**Rôle personnalisé pour l’ID d’application pour SQL Managed Instance**

```json
{
  "Name": "DMS Role - App ID - SQL MI",
  "IsCustom": true,
  "Description": "DMS App ID access to complete MI migrations",
  "Actions": [
        "Microsoft.Sql/managedInstances/read",
        "Microsoft.Sql/managedInstances/write",
        "Microsoft.Sql/managedInstances/databases/read",
        "Microsoft.Sql/managedInstances/databases/write",
        "Microsoft.Sql/managedInstances/databases/delete",
        "Microsoft.Sql/managedInstances/metrics/read"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>/resourceGroups/<ManagedInstance_rg_name>"
  ]
}
```

**Rôle personnalisé pour l’ID d’application pour le stockage**

```json
{
  "Name": "DMS Role - App ID - Storage",
  "IsCustom": true,
  "Description": "DMS App ID storage access to complete MI migrations",
  "Actions": [
"Microsoft.Storage/storageAccounts/read",
        "Microsoft.Storage/storageAccounts/listKeys/action",
        "Microsoft.Storage/storageaccounts/blobservices/read",
        "Microsoft.Storage/storageaccounts/blobservices/write"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>/resourceGroups/<StorageAccount_rg_name>"
  ]
}
```

## <a name="role-assignment"></a>Attribution de rôle

Pour affecter un rôle à des utilisateurs/ID d’application, ouvrez le portail Azure, et procédez comme suit :

1. Accédez au groupe de ressources ou à la ressource (à l’exception du rôle qui doit être accordé sur l’abonnement), accédez à **Contrôle d'accès**, puis faites défiler la liste pour rechercher les rôles personnalisés que vous venez de créer.

2. Sélectionnez le rôle approprié, sélectionnez l’ID de l’application, puis enregistrez les modifications.

  Votre ou vos ID d’application s’affichent désormais sous l’onglet **Attributions de rôles**.

## <a name="next-steps"></a>Étapes suivantes

* Lisez l’aide à la migration du [Guide de migration de bases de données](https://datamigration.microsoft.com/) de Microsoft pour votre scénario.