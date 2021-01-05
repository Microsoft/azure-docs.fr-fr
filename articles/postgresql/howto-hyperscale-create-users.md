---
title: Créer des utilisateurs - Hyperscale (Citus) - Azure Database pour PostgreSQL
description: Cet article décrit comment vous pouvez créer des comptes d’utilisateurs pour interagir avec un serveur Azure Database pour PostgreSQL - Hyperscale (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 1/8/2019
ms.openlocfilehash: 3d23ee6119b625e11ce44bb9ad11ce4b3ee0280d
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91295734"
---
# <a name="create-users-in-azure-database-for-postgresql---hyperscale-citus"></a>Créer des utilisateurs dans Azure Database pour PostgreSQL - Hyperscale (Citus)

> [!NOTE]
> Le terme « utilisateurs » fait référence aux utilisateurs au sein d’un groupe de serveurs Hyperscale (Citus). Pour en savoir plus sur les utilisateurs de l’abonnement Azure et leurs privilèges, consultez l’article [Contrôle d’accès en fonction du rôle (Azure RBAC) Azure](../role-based-access-control/built-in-roles.md) ou lisez [comment personnaliser les rôles](../role-based-access-control/custom-roles.md).

## <a name="the-server-admin-account"></a>Compte d’administrateur de serveur

Le moteur PostgreSQL utilise des [rôles](https://www.postgresql.org/docs/current/sql-createrole.html) pour contrôler l’accès aux objets de base de données, et un groupe de serveurs Hyperscale (Citus) nouvellement créé est fourni avec plusieurs rôles prédéfinis :

* Les [rôles PostgreSQL par défaut](https://www.postgresql.org/docs/current/default-roles.html)
* `azure_pg_admin`
* `postgres`
* `citus`

Étant donné qu’Hyperscale (Citus) est un service PaaS managé, seul Microsoft peut se connecter avec le rôle de superutilisateur `postgres`. Pour un accès administratif limité, Hyperscale (Citus) fournit le rôle `citus`.

Autorisations pour le rôle `citus` :

* Lire toutes les variables de configuration, même les variables normalement visibles uniquement par les superutilisateurs.
* Lire tous les affichages pg\_stat\_\* et utiliser diverses extensions liées aux statistiques (même des affichages ou des extensions normalement visibles uniquement par les superutilisateurs).
* Exécuter des fonctions de supervision qui peuvent prendre des verrous ACCESS SHARE sur des tables, potentiellement pendant une longue période.
* [Créer des extensions PostgreSQL](concepts-hyperscale-extensions.md) (car le rôle est membre d’`azure_pg_admin`).

Notez que le rôle `citus` présente des restrictions :

* Impossibilité de créer des rôles
* Impossibilité de créer des bases de données

## <a name="how-to-create-additional-user-roles"></a>Comment créer des rôles d’utilisateur supplémentaires

Comme mentionné, le compte administrateur `citus` ne dispose pas des autorisations nécessaires pour créer des utilisateurs supplémentaires. Pour ajouter un utilisateur, utilisez le portail Azure.

1. Accédez à la page **Rôles** pour votre groupe de serveurs Hyperscale (Citus), puis cliquez sur **+ Ajouter** :

   :::image type="content" source="media/howto-hyperscale-create-users/1-role-page.png" alt-text="La page Rôles":::

2. Saisissez le nom de rôle et le mot de passe. Cliquez sur **Enregistrer**.

   :::image type="content" source="media/howto-hyperscale-create-users/2-add-user-fields.png" alt-text="La page Rôles":::

L’utilisateur est créé sur le nœud coordinateur du groupe de serveurs et est propagé à tous les nœuds Worker. Les rôles créés via le portail Azure comportent l’attribut `LOGIN`, ce qui signifie qu’il s’agit d’utilisateurs véritables qui peuvent se connecter à la base de données.

## <a name="how-to-modify-privileges-for-user-role"></a>Comment modifier des privilèges pour un rôle d’utilisateur

De nouveaux rôles d’utilisateur sont couramment utilisés pour fournir un accès à la base de données avec des privilèges restreints. Pour modifier des privilèges d’utilisateur, utilisez des commandes PostgreSQL standard, à l’aide d’un outil tel que PgAdmin ou psql. (Voir [Connexion avec psql](quickstart-create-hyperscale-portal.md#connect-to-the-database-using-psql) dans le démarrage rapide Hyperscale (Citus).)

Par exemple, pour autoriser `db_user` à lire `mytable`, accordez l’autorisation :

```sql
GRANT SELECT ON mytable TO db_user;
```

Hyperscale (Citus) propage les instructions GRANT sur une table unique à travers l’ensemble du cluster, en les appliquant à tous les nœuds Worker. Il propage également les allocations à l’ensemble du système (par exemple, pour toutes les tables d’un schéma) :

```sql
-- applies to the coordinator node and propagates to workers
GRANT SELECT ON ALL TABLES IN SCHEMA public TO db_user;
```

## <a name="how-to-delete-a-user-role-or-change-their-password"></a>Comment supprimer un rôle d’utilisateur ou modifier son mot de passe

Pour mettre à jour un utilisateur, accédez à la page **Rôles** pour votre groupe de serveurs Hyperscale (Citus), puis cliquez sur les points de suspension **...** à côté de l’utilisateur. L’ellipse ouvre un menu pour supprimer l’utilisateur ou réinitialiser son mot de passe.

   :::image type="content" source="media/howto-hyperscale-create-users/edit-role.png" alt-text="La page Rôles":::

Le rôle `citus` est privilégié et ne peut pas être supprimé.

## <a name="next-steps"></a>Étapes suivantes

Ouvrez le pare-feu pour les adresses IP des machines des nouveaux utilisateurs afin de leur permettre de se connecter : [Créer et gérer des règles de pare-feu Hyperscale (Citus) à l’aide du portail Azure](howto-hyperscale-manage-firewall-using-portal.md).

Pour plus d’informations sur la gestion des comptes d’utilisateur de base de données, consultez la documentation de PostgreSQL :

* [Rôles et privilèges de base de données](https://www.postgresql.org/docs/current/static/user-manag.html)
* [Syntaxe GRANT](https://www.postgresql.org/docs/current/static/sql-grant.html)
* [Privilèges](https://www.postgresql.org/docs/current/static/ddl-priv.html)
