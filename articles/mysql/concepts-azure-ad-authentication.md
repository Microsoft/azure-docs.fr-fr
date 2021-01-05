---
title: Authentification Active Directory - Azure Database pour MySQL
description: En savoir plus sur les concepts d’Azure Active Directory pour l’authentification avec Azure Database pour MySQL
author: lfittl-msft
ms.author: lufittl
ms.service: mysql
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: 942789d5541e796247af2dbdf9371cce4701105e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87171890"
---
# <a name="use-azure-active-directory-for-authenticating-with-mysql"></a>Utiliser Azure Active Directory pour l’authentification avec MySQL

L’authentification Microsoft Azure Active Directory (Azure AD) est un mécanisme de connexion à Azure Database pour MySQL à l’aide d’identités définies dans Azure AD.
Avec l’authentification Azure AD, vous pouvez gérer les identités des utilisateurs de base de données et d’autres services Microsoft dans un emplacement centralisé, ce qui simplifie la gestion des autorisations.

Les avantages liés à l’utilisation d’Azure AD incluent ce qui suit :

- Authentification uniforme des utilisateurs dans les services Azure
- Gestion des stratégies de mot de passe et de la rotation de mot de passe dans un emplacement unique
- Formes d’authentification multiples prises en charge par Azure Active Directory, ce qui peut éviter d’avoir à stocker les mots de passe
- Les clients peuvent gérer les autorisations de base de données à l’aide des groupes (Azure AD) externes.
- L’authentification Azure AD utilise des utilisateurs de base de données MySQL pour authentifier les identités au niveau de la base de données.
- Prise en charge de l’authentification basée sur des jetons pour les applications se connectant à Azure Database pour MySQL

Pour configurer et utiliser l’authentification Azure Active Directory, procédez comme suit :

1. Créez et remplissez Azure Active Directory avec les identités des utilisateurs, si nécessaire.
2. Le cas échéant, associez ou modifiez l’annuaire Active Directory actuellement associé à votre abonnement Azure.
3. Créez un administrateur Azure AD pour le serveur Azure Database pour MySQL.
4. Créer des utilisateurs de base de données dans votre base de données mappés sur les identités Azure AD.
5. Connectez-vous à votre base de données en extrayant un jeton pour une identité Azure AD et en ouvrant une session.

> [!NOTE]
> Pour apprendre à créer et à remplir Azure AD, puis à configurer Azure AD avec Azure SQL Database pour MySQL, consultez [Configurer et se connecter avec Azure AD pour Azure Database pour MySQL](howto-configure-sign-in-azure-ad-authentication.md).

## <a name="architecture"></a>Architecture

Le diagramme général suivant résume le fonctionnement de l’authentification Azure AD avec Azure Database pour MySQL. Les flèches indiquent les voies de communication.

![flux d’authentification][1]

## <a name="administrator-structure"></a>Structure de l’administrateur

En cas d’utilisation de l’authentification Azure AD, il existe deux comptes administrateur pour le serveur MySQL ; l’administrateur de MySQL d’origine et l’administrateur Azure AD. Seul l’administrateur basé sur un compte Azure AD peut créer le premier utilisateur de la base de données autonome Azure AD dans une base de données utilisateur. La connexion d’administrateur Azure AD peut être un utilisateur Azure AD ou un groupe Azure AD. Lorsque l’administrateur est un compte de groupe, il peut être utilisé par n’importe quel membre du groupe, activant plusieurs administrateurs Azure AD pour le serveur MySQL. L’utilisation d’un compte de groupe en tant qu’administrateur facilite la gestion en vous permettant d’ajouter et de supprimer des membres du groupe dans Azure AD sans modifier les utilisateurs ou les autorisations du serveur MySQL. Seul un administrateur Azure AD (utilisateur ou groupe) peut être configuré à tout moment.

![structure admin][2]

## <a name="permissions"></a>Autorisations

Pour créer des utilisateurs qui peuvent s’authentifier avec Azure AD, vous devez être l'administrateur Azure AD désigné. Cet utilisateur est attribué en configurant le compte d’administrateur Azure AD pour un serveur Azure Database pour MySQL spécifique.

Pour créer un utilisateur de base de données Azure AD, vous devez vous connecter en tant qu’administrateur Azure AD. Cela est illustré dans [Configurer et se connecter avec Azure AD pour Azure Database pour MySQL](howto-configure-sign-in-azure-ad-authentication.md).

L’authentification Azure AD n’est possible que si l’administrateur Azure AD a été créé pour Azure Database pour MySQL. Si l’administrateur Azure Active Directory a été supprimé du serveur, les utilisateurs Azure Active Directory existants créés précédemment ne peuvent plus se connecter à la base de données à l’aide de leurs informations d’identification Azure Active Directory.

## <a name="connecting-using-azure-ad-identities"></a>Connexion à l’aide des identités Azure AD

L’authentification Azure Active Directory prend en charge les méthodes suivantes de connexion à une base de données à l’aide d’identités Azure AD :

- Mot de passe Azure Active Directory
- Intégration d’Azure Active Directory
- Authentification universelle Azure Active Directory avec MFA
- Utilisation des certificats ou de clés secrètes client d’application Active Directory
- [Identité gérée](howto-connect-with-managed-identity.md)

Une fois authentifié auprès d’Active Directory, vous récupérez un jeton. Ce jeton est votre mot de passe de connexion.

Notez que les opérations de gestion, telles que l’ajout de nouveaux utilisateurs, sont uniquement prises en charge pour les rôles d’utilisateur Azure AD à ce stade.

> [!NOTE]
> Pour plus d’informations sur la façon de se connecter avec un jeton Active Directory, consultez [Configurer et se connecter avec Azure AD pour Azure Database pour MySQL](howto-configure-sign-in-azure-ad-authentication.md).

## <a name="additional-considerations"></a>Considérations supplémentaires

- L’authentification Azure Active Directory est disponible uniquement pour MySQL 5.7 et versions ultérieures.
- Un seul utilisateur administrateur Azure AD peut être configuré pour un serveur Azure Database pour MySQL à tout moment.
- Seul un administrateur d’Azure AD pour MySQL peut se connecter initialement au serveur Azure Database pour MySQL à l’aide d’un compte Azure Active Directory. L’administrateur Active Directory peut configurer les utilisateurs de base de données Azure AD suivants.
- Si un utilisateur est supprimé d’Azure AD, il ne pourra plus s’authentifier auprès d’Azure AD, et il ne sera donc plus possible d’acquérir un code d'accès pour cet utilisateur. Dans ce cas, bien que l'utilisateur correspondant se trouve toujours dans la base de données, il ne sera pas possible de se connecter au serveur avec cet utilisateur.
> [!NOTE]
> La connexion à l’aide de l’utilisateur Azure AD supprimé peut toujours être effectuée jusqu’à l’expiration du jeton (60 minutes maximum après l’émission du jeton).  Si vous supprimez également l’utilisateur de la base de données Azure Database pour MySQL, cet accès sera immédiatement révoqué.
- Si l’administrateur Azure AD est supprimé du serveur, ce dernier n’est plus associé à un locataire Azure AD et, par conséquent, toutes les connexions Azure AD sont désactivées pour le serveur. L’ajout d’un nouvel administrateur Azure AD à partir du même locataire réactivera les connexions Azure AD.
- Azure Database pour MySQL met en correspondance les jetons d’accès à l'utilisateur Azure Database pour MySQL à l’aide de l’ID d’utilisateur Azure AD unique de l’utilisateur au lieu d’utiliser le nom d’utilisateur. Cela signifie que si un utilisateur Azure AD est supprimé d’Azure AD et qu’un nouvel utilisateur est créé avec le même nom, Azure Database pour MySQL considère que le nouvel utilisateur est différent de l’utilisateur précédent. Par conséquent, si un utilisateur est supprimé d’Azure AD et qu’un nouvel utilisateur portant le même nom a été ajouté, le nouvel utilisateur ne sera pas en mesure de se connecter à l'utilisateur existant.

## <a name="next-steps"></a>Étapes suivantes

- Pour apprendre à créer et à remplir Azure AD, puis à configurer Azure AD avec Azure SQL Database pour MySQL, consultez [Configurer et se connecter avec Azure AD pour Azure Database pour MySQL](howto-configure-sign-in-azure-ad-authentication.md).
- Pour obtenir une vue d’ensemble des connexions et des utilisateurs de base de données Azure Database pour MySQL, consultez [Créer des utilisateurs dans Azure Database pour MySQL](howto-create-users.md).

<!--Image references-->

[1]: ./media/concepts-azure-ad-authentication/authentication-flow.png
[2]: ./media/concepts-azure-ad-authentication/admin-structure.png
