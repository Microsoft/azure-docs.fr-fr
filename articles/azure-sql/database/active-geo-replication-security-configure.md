---
title: Configurer la sécurité pour la récupération d’urgence
description: Découvrez les considérations de sécurité pour la configuration et la gestion de la sécurité après une restauration de base de données ou un basculement vers un serveur secondaire.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 12/18/2018
ms.openlocfilehash: 317b530fbaa34ca5689bb505126892e4eba06bd9
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92674794"
---
# <a name="configure-and-manage-azure-sql-database-security-for-geo-restore-or-failover"></a>Configurer et gérer la sécurité Azure SQL Database pour la géo-restauration ou le basculement
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article décrit les exigences d’authentification à respecter pour configurer et contrôler la [géoréplication active](active-geo-replication-overview.md) et les [groupes de basculement automatique](auto-failover-group-overview.md). Il indique également les étapes nécessaires pour configurer l’accès utilisateur à la base de données secondaire. Enfin, il explique comment activer l’accès à la base de données récupérée après [géorestauration](recovery-using-backups.md#geo-restore). Pour plus d’informations sur les options de récupération, consultez [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md).

## <a name="disaster-recovery-with-contained-users"></a>Récupération d’urgence avec des utilisateurs contenus

Contrairement aux utilisateurs classiques, qui doivent être mappés sur les connexions dans la base de données master, un utilisateur contenu est géré entièrement par la base de données elle-même. Cela a deux avantages. Dans le scénario de récupération d’urgence, les utilisateurs peuvent continuer de se connecter à la nouvelle base de données primaire ou à la base de données restaurée à l’aide de la géo-restauration sans configuration supplémentaire, car c’est la base de données qui gère les utilisateurs. Du point de vue de la connexion, cette configuration présente également des possibilités de mise à l’échelle et d’amélioration des performances. Pour plus d’informations, voir [Utilisateurs de base de données autonome - Rendre votre base de données portable](/sql/relational-databases/security/contained-database-users-making-your-database-portable).

L’inconvénient principal est que la gestion du processus de récupération d’urgence à grande échelle est plus difficile. Lorsque plusieurs de vos bases de données utilisent la même connexion, maintenir les informations d’identification avec des utilisateurs contenus dans plusieurs bases de données peut anéantir les avantages des utilisateurs contenus. Par exemple, la stratégie de rotation des mots de passe nécessite de faire les modifications de façon cohérente dans plusieurs bases de données plutôt que de modifier le mot de passe de l’identifiant de connexion une fois dans la base de données principale. Pour cette raison, si vous avez plusieurs bases de données qui utilisent le même nom d’utilisateur et le même mot de passe, l’utilisation d’utilisateurs contenus est déconseillée.

## <a name="how-to-configure-logins-and-users"></a>Configuration des identifiants de connexion et des utilisateurs

Si vous utilisez des identifiants de connexion et des utilisateurs (et non des utilisateurs contenus), vous devez prendre des mesures supplémentaires pour vous assurer que les mêmes identifiants de connexion existent dans la base de données primaire. Les sections suivantes décrivent les étapes impliquées et d’autres considérations relatives.

  >[!NOTE]
  > Il est également possible d’utiliser des ID de connexion d’Azure Active Directory (AAD) pour gérer vos bases de données. Pour plus d’informations, voir [ID de connexions et utilisateurs Azure SQL](./logins-create-manage.md).

### <a name="set-up-user-access-to-a-secondary-or-recovered-database"></a>Configurer l’accès utilisateur à une base de données secondaire ou restaurée

Pour que la base de données secondaire puisse être utilisable en tant que base de données en lecture seule et pour garantir un accès approprié à la nouvelle base de données primaire ou à la base de données restaurée à l’aide de la géo-restauration, la configuration de sécurité appropriée doit être mise en œuvre sur la base de données principale du serveur cible avant la restauration.

Les autorisations spécifiques de chaque étape sont décrites plus tard dans cette rubrique.

La préparation de l’accès utilisateur à une base de données secondaire de géoréplication doit être effectuée dans le cadre de la configuration de la géoréplication. La préparation de l’accès utilisateur aux bases de données géo-restaurées doit être effectuée à tout moment lorsque le serveur d’origine est en ligne (par exemple dans le cadre du test de récupération d’urgence).

> [!NOTE]
> Si vous effectuez un basculement ou une géorestauration vers un serveur sur lequel les identifiants de connexion ne sont pas configurés correctement, l’accès à ce serveur sera limité au compte d’administrateur du serveur.

La configuration des identifiants de connexion sur le serveur cible implique les trois étapes que voici :

#### <a name="1-determine-logins-with-access-to-the-primary-database"></a>1. Déterminez quelles connexions ont accès à la base de données primaire

La première étape du processus consiste à déterminer les noms de connexion qui doivent être dupliqués sur le serveur cible. Pour ce faire, vous devez disposer d’une paire d’instructions SELECT, l’une dans la base de données master logique sur le serveur source et l’autre dans la base de données principale elle-même.

Seuls l’administrateur du serveur ou un membre avec le rôle serveur **LoginManager** peut déterminer les connexions sur le serveur source avec l’instruction SELECT suivante.

```sql
SELECT [name], [sid]
FROM [sys].[sql_logins]
WHERE [type_desc] = 'SQL_Login'
```

Seul un membre du rôle db_owner, l’utilisateur dbo ou l’administrateur du serveur peuvent déterminer toutes les entités de base de données utilisateur dans la base de données principale.

```sql
SELECT [name], [sid]
FROM [sys].[database_principals]
WHERE [type_desc] = 'SQL_USER'
```

#### <a name="2-find-the-sid-for-the-logins-identified-in-step-1"></a>2. Recherchez le SID des connexions identifiées à l’étape 1

En comparant le résultat des requêtes issues de la section précédente et en faisant correspondre les SID, vous pouvez mapper la connexion serveur sur l’utilisateur de base de données. Les connexions dont l’utilisateur de base de données correspond au SID ont un accès utilisateur à cette base de données en tant que qu’utilisateur principal de base de données.

La requête suivante peut être utilisée pour voir toutes les entités utilisateur et leur SID dans une base de données. Seul un membre du rôle de base de données db_owner ou un administrateur serveur peut exécuter cette requête.

```sql
SELECT [name], [sid]
FROM [sys].[database_principals]
WHERE [type_desc] = 'SQL_USER'
```

> [!NOTE]
> Les utilisateurs de **INFORMATION_SCHEMA** et **sys** ont des SID *NULL* , et le SID **invité** est **0x00** . Le SID **dbo** peut commencer par *0x01060000000001648000000000048454* , si le créateur de la base de données est l’administrateur serveur et non un membre de **DbManager** .

#### <a name="3-create-the-logins-on-the-target-server"></a>3. Créez les connexions sur le serveur cible

La dernière étape consiste à accéder au(x) serveur(s) cible, et à générer les connexions avec les SID appropriés. La syntaxe de base est la suivante :

```sql
CREATE LOGIN [<login name>]
WITH PASSWORD = <login password>,
SID = <desired login SID>
```

> [!NOTE]
> Si vous souhaitez accorder un accès utilisateur à la base de données secondaire, mais pas au serveur principal, vous pouvez le faire en modifiant la connexion de l’utilisateur sur le serveur principal à l’aide de la syntaxe suivante.
>
> ```sql
> ALTER LOGIN <login name> DISABLE
> ```
>
> DISABLE ne modifie pas le mot de passe, pour que vous puissiez toujours l’activer si nécessaire.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la gestion de l’accès aux bases de données et des connexions, voir [Sécurité SQL Database : gérer la sécurité de l’accès aux bases de données et des connexions](logins-create-manage.md).
* Pour plus d’informations sur les utilisateurs de base de données autonome, consultez [Utilisateurs de base de données autonome - Rendre votre base de données portable](/sql/relational-databases/security/contained-database-users-making-your-database-portable).
* Pour plus d’informations sur la géoréplication active, voir la section [Géoréplication active](active-geo-replication-overview.md).
* Pour plus d’informations sur les groupes de basculement automatique, voir [Groupes de basculement automatique](auto-failover-group-overview.md).
* Pour plus d’informations sur l’utilisation de la géorestauration, voir [Géorestauration](recovery-using-backups.md#geo-restore).