---
title: Utiliser Azure Active Directory - Azure Database pour PostgreSQL - Serveur unique
description: Découvrez comment configurer Azure Active Directory (AAD) pour l’authentification avec Azure Database pour PostgreSQL - Serveur unique.
author: lfittl
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 07/23/2020
ms.openlocfilehash: 178c339f6f47569160a9a748794678c610f35734
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87171641"
---
# <a name="use-azure-active-directory-for-authentication-with-postgresql"></a>Utiliser Azure Active Directory pour l’authentification avec PostgreSQL

Cet article vous détaille les étapes de configuration de l’accès à Azure Active Directory avec Azure Database pour PostgreSQL et de la manière de se connecter à l’aide d’un jeton Azure AD.

## <a name="setting-the-azure-ad-admin-user"></a>Configuration de l’utilisateur Administrateur Azure AD

Seuls les utilisateurs administrateurs Azure AD peuvent créer/activer des utilisateurs pour l’authentification basée sur Azure AD. Nous vous déconseillons d’utiliser l’administrateur Azure AD pour les opérations de base de données courantes, car il dispose d’autorisations utilisateur élevées (par exemple, CREATEDB).

Pour définir l’administrateur Azure AD (vous pouvez utiliser un utilisateur ou un groupe), effectuez les étapes suivantes.

1. Dans le portail Azure, sélectionnez l’instance Azure Database pour PostgreSQL que vous souhaitez activer pour Azure AD.
2. Sous Paramètres, sélectionnez Administrateur Active Directory :

![configurer l’administrateur azure ad][2]

3. Sélectionnez un utilisateur Azure AD valide dans le locataire client pour qu’il devienne l’administrateur Azure AD.

> [!IMPORTANT]
> Lorsque vous définissez l’administrateur, un nouvel utilisateur est ajouté au serveur Azure Database pour PostgreSQL avec les autorisations d’administrateur complètes. L’utilisateur Administrateur Azure AD dans Azure Database pour PostgreSQL aura le rôle `azure_ad_admin`.

Un seul administrateur Azure AD peut être créé par serveur PostgreSQL et la sélection d’un autre administrateur remplacera l’administrateur Azure AD existant configuré pour le serveur. Vous pouvez spécifier un groupe Azure AD plutôt qu’un utilisateur individuel pour avoir plusieurs administrateurs. Notez que vous vous connecterez alors avec le nom du groupe à des fins d’administration.

## <a name="connecting-to-azure-database-for-postgresql-using-azure-ad"></a>Connexion à Azure Database pour PostgreSQL à l’aide d’Azure AD

Le diagramme de niveau élevé suivant résume le workflow de l’utilisation de l’authentification Azure AD avec Azure Database pour PostgreSQL :

![flux d’authentification][1]

Nous avons conçu l’intégration Azure AD pour qu’elle fonctionne avec des outils PostgreSQL communs, tels que psql, qui sont indépendants d’Azure AD et ne prennent en charge que la spécification du nom d’utilisateur et du mot de passe lors de la connexion à PostgreSQL. Nous transmettons le jeton Azure AD en tant que mot de passe, comme indiqué dans l’image ci-dessus.

Pour le moment, nous avons testé les clients suivants :

- psql CommandLine (utilisez la variable PGPASSWORD pour transmettre le jeton, voir ci-dessous)
- Azure Data Studio (utilisant l’extension PostgreSQL)
- Autres clients basés sur libpq (par exemple, frameworks d’application courants et ORM)

> [!NOTE]
> N’oubliez pas que l’utilisation du jeton Azure AD avec pgAdmin n’est pas prise en charge actuellement, à cause de la limitation de codage en dur de 256 caractères pour les mots de passe (que le jeton dépasse).

Voici les étapes nécessaires à l’authentification d’un utilisateur ou d’une application avec Azure AD :

### <a name="prerequisites"></a>Prérequis

Vous pouvez poursuivre dans Azure Cloud Shell, une machine virtuelle Azure ou sur votre ordinateur local. Assurez-vous que [l’interface Azure CLI est installée](/cli/azure/install-azure-cli).

### <a name="step-1-authenticate-with-azure-ad"></a>Étape 1 : S’authentifier avec Azure AD

Commencez par vous authentifier auprès d’Azure AD à l’aide de l’outil Azure CLI. Cette étape n’est pas obligatoire dans Azure Cloud Shell.

```
az login
```

La commande lance une fenêtre de navigateur sur la page d’authentification Azure AD. Pour ce faire, vous devez fournir votre ID d’utilisateur Azure AD et le mot de passe.

### <a name="step-2-retrieve-azure-ad-access-token"></a>Étape 2 : Récupérer un jeton d’accès Azure AD

Appelez l’outil Azure CLI pour obtenir un jeton d’accès pour l’utilisateur authentifié auprès d’Azure AD à l’étape 1 afin d’accéder à Azure Database pour PostgreSQL.

Exemple (pour le cloud public) :

```azurecli-interactive
az account get-access-token --resource https://ossrdbms-aad.database.windows.net
```

La valeur de la ressource ci-dessus doit être spécifiée exactement comme indiqué. Pour les autres clouds, la valeur de la ressource peut être recherchée à l’aide de ce qui suit :

```azurecli-interactive
az cloud show
```

Pour Azure CLI version 2.0.71 et les versions ultérieures, la commande peut être spécifiée dans la version plus pratique suivante pour tous les clouds :

```azurecli-interactive
az account get-access-token --resource-type oss-rdbms
```

Une fois l’authentification réussie, Azure AD retourne un jeton d’accès :

```json
{
  "accessToken": "TOKEN",
  "expiresOn": "...",
  "subscription": "...",
  "tenant": "...",
  "tokenType": "Bearer"
}
```

Le jeton est une chaîne de base 64 qui code toutes les informations relatives à l’utilisateur authentifié et qui est ciblée vers le service Azure Database pour PostgreSQL.

> [!NOTE]
> La validité du jeton d’accès est comprise entre 5 minutes et 60 minutes. Nous vous recommandons d’obtenir le jeton d’accès juste avant de lancer la connexion à Azure Database pour PostgreSQL.

### <a name="step-3-use-token-as-password-for-logging-in-with-postgresql"></a>Étape 3 : Utiliser un jeton comme mot de passe pour la connexion avec PostgreSQL

Lors de la connexion, vous devez utiliser le jeton d’accès comme mot de passe utilisateur PostgreSQL.

Lorsque vous utilisez le client de ligne de commande `psql`, le jeton d’accès doit être transmis via la variable d’environnement `PGPASSWORD`, car le jeton d’accès dépasse la longueur du mot de passe que `psql` peut accepter directement :

Exemple Windows :

```cmd
set PGPASSWORD=<copy/pasted TOKEN value from step 2>
```

```PowerShell
$env:PGPASSWORD='<copy/pasted TOKEN value from step 2>'
```

Exemple Linux/macOS :

```shell
export PGPASSWORD=<copy/pasted TOKEN value from step 2>
```

Vous pouvez désormais établir une connexion avec Azure Database pour PostgreSQL comme vous le feriez normalement :

```shell
psql "host=mydb.postgres... user=user@tenant.onmicrosoft.com@mydb dbname=postgres sslmode=require"
```

Considérations importantes à prendre en compte lors de la connexion :

* `user@tenant.onmicrosoft.com` est le nom de l’utilisateur ou du groupe Azure AD auquel vous essayez de vous connecter
* Ajoutez toujours le nom du serveur après le nom de groupe/d’utilisateur Azure AD (par exemple, `@mydb`)
* Veillez à utiliser exactement la façon dont le nom d’utilisateur ou de groupe Azure AD est épelé
* Les noms d’utilisateurs et de groupes Azure AD respectent la casse
* Quand vous vous connectez en tant que groupe, utilisez uniquement le nom du groupe (par exemple, `GroupName@mydb`)
* Si le nom contient des espaces, utilisez `\` avant chaque espace pour le placer dans une séquence d’échappement

Vous êtes maintenant authentifié auprès de votre serveur PostgreSQL à l’aide de l’authentification Azure AD.

## <a name="creating-azure-ad-users-in-azure-database-for-postgresql"></a>Création d’utilisateurs Azure AD dans Azure Database pour PostgreSQL

Pour ajouter un utilisateur Azure AD à votre base de données Azure Database pour PostgreSQL, procédez comme suit après la connexion (voir la section suivante sur la connexion) :

1. Tout d’abord, assurez-vous que l’utilisateur Azure AD `<user>@yourtenant.onmicrosoft.com` est un utilisateur valide dans le locataire Azure AD.
2. Connectez-vous à votre instance Azure Database pour PostgreSQL en tant qu’utilisateur Administrateur Azure AD.
3. Créez le rôle `<user>@yourtenant.onmicrosoft.com` dans Azure Database pour PostgreSQL.
4. Faites de `<user>@yourtenant.onmicrosoft.com` un membre du rôle azure_ad_user. Ce rôle doit être donné uniquement aux utilisateurs Azure AD.

**Exemple :**

```sql
CREATE ROLE "user1@yourtenant.onmicrosoft.com" WITH LOGIN IN ROLE azure_ad_user;
```

> [!NOTE]
> L’authentification d’un utilisateur par le biais d’Azure AD ne donne pas à l’utilisateur des autorisations d’accès aux objets dans la base de données Azure Database pour PostgreSQL. Vous devez accorder manuellement les autorisations requises à l’utilisateur.

## <a name="creating-azure-ad-groups-in-azure-database-for-postgresql"></a>Création de groupes Azure AD dans Azure Database pour PostgreSQL

Pour permettre à un groupe Azure AD d’accéder à votre base de données, utilisez le même mécanisme que pour les utilisateurs, mais spécifiez à la place le nom du groupe :

**Exemple :**

```sql
CREATE ROLE "Prod DB Readonly" WITH LOGIN IN ROLE azure_ad_user;
```

Lors de la connexion, les membres du groupe utilisent leurs jetons d’accès personnels, mais se connectent avec le nom du groupe spécifié comme nom d’utilisateur.

## <a name="token-validation"></a>Validation du jeton

L’authentification Azure AD dans Azure Database pour PostgreSQL garantit que l’utilisateur existe dans le serveur PostgreSQL et vérifie la validité du jeton en validant le contenu du jeton. Les étapes de validation de jeton suivantes sont exécutées :

- Le jeton est signé par Azure AD et n’a pas été falsifié
- Le jeton a été émis par Azure AD pour le locataire associé au serveur
- Le jeton n’a pas expiré
- Le jeton est destiné à la ressource Azure Database pour PostgreSQL (et non pour une autre ressource Azure)

## <a name="migrating-existing-postgresql-users-to-azure-ad-based-authentication"></a>Migration d’utilisateurs PostgreSQL existants vers l’authentification basée sur Azure AD

Vous pouvez activer l’authentification Azure AD pour des utilisateurs existants. Il existe deux cas à prendre en compte :

### <a name="case-1-postgresql-username-matches-the-azure-ad-user-principal-name"></a>Cas n° 1 : Le nom d’utilisateur PostgreSQL correspond au nom d’utilisateur principal Azure AD

Dans le cas improbable où vos utilisateurs existants correspondent déjà aux noms d’utilisateur Azure AD, vous pouvez leur accorder le rôle `azure_ad_user` afin de les activer pour l’authentification Azure AD :

```sql
GRANT azure_ad_user TO "existinguser@yourtenant.onmicrosoft.com";
```

Ils peuvent désormais se connecter avec les informations d’identification Azure AD au lieu d’utiliser leur mot de passe utilisateur PostgreSQL précédemment configuré.

### <a name="case-2-postgresql-username-is-different-than-the-azure-ad-user-principal-name"></a>Cas n° 2 : Le nom d’utilisateur PostgreSQL est différent du nom d’utilisateur principal Azure AD

Si un utilisateur PostgreSQL n’existe pas dans Azure AD ou a un nom d’utilisateur différent, vous pouvez utiliser les groupes Azure AD pour vous authentifier en tant que cet utilisateur PostgreSQL. Vous pouvez migrer des utilisateurs Azure Database pour PostgreSQL existants vers Azure AD en créant un groupe Azure AD avec un nom qui correspond à l’utilisateur PostgreSQL, puis en accordant le rôle azure_ad_user à l’utilisateur PostgreSQL existant :

```sql
GRANT azure_ad_user TO "DBReadUser";
```

Cela suppose que vous avez créé un groupe « DBReadUser » dans votre Azure AD. Les utilisateurs appartenant à ce groupe peuvent désormais se connecter à la base de données en tant que cet utilisateur.

## <a name="next-steps"></a>Étapes suivantes

* Passez en revue les concepts généraux pour [Authentification Azure Active Directory avec Azure Database pour PostgreSQL - Serveur unique](concepts-aad-authentication.md)

<!--Image references-->

[1]: ./media/concepts-aad-authentication/authentication-flow.png
[2]: ./media/concepts-aad-authentication/set-aad-admin.png
