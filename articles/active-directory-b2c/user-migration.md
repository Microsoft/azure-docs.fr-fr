---
title: Approches de la migration des utilisateurs
titleSuffix: Azure AD B2C
description: Migrez les comptes d’utilisateur d’un autre fournisseur d’identité vers Azure AD B2C en utilisant les méthodes de prémigration ou de migration fluide.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 5d680da7a5f09a55bec5d791642b80fd76c6088e
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509612"
---
# <a name="migrate-users-to-azure-ad-b2c"></a>Migrer des utilisateurs vers Azure AD B2C

La migration depuis un autre fournisseur d’identité vers Azure Active Directory B2C (Azure AD B2C) peut également nécessiter la migration des comptes d’utilisateur existants. Deux méthodes de migration sont décrites ici : la *prémigration* et la *migration fluide*. Dans les deux approches, vous devez écrire une application ou un script qui utilise l’[API Microsoft Graph](manage-user-accounts-graph-api.md) pour créer des comptes d’utilisateur dans Azure AD B2C.

## <a name="pre-migration"></a>Prémigration

Dans le flux de prémigration, votre application de migration effectue les étapes suivantes pour chaque compte d’utilisateur :

1. Lire le compte d’utilisateur de l’ancien fournisseur d’identité, y compris ses informations d’identification actuelles (nom d’utilisateur et mot de passe).
1. Créer un compte correspondant dans votre annuaire Azure AD B2C avec les informations d’identification actuelles.

Utilisez le flux de prémigration dans l’une ou l’autre de ces deux situations :

- Vous avez accès aux informations d’identification en texte clair d’un utilisateur (son nom d’utilisateur et son mot de passe).
- Les informations d’identification sont chiffrées, mais vous pouvez les déchiffrer.

Pour plus d’informations sur la création programmatique de comptes d’utilisateur, consultez [Gérer les comptes d’utilisateurs Azure AD B2C avec Microsoft Graph](manage-user-accounts-graph-api.md).

## <a name="seamless-migration"></a>Migration fluide

Utilisez le flux de migration fluide si les mots de passe en texte en clair dans l’ancien fournisseur d’identité ne sont pas accessibles. Par exemple, dans les cas suivants :

- Le mot de passe est stocké dans un format chiffré unidirectionnel, comme avec une fonction de hachage.
- Le mot de passe est stocké par le fournisseur d’identité précédent de telle façon que vous ne pouvez pas y accéder. Par exemple, quand le fournisseur d’identité valide les informations d’identification en appelant un service web.

Le flux de migration fluide nécessite néanmoins toujours la prémigration des comptes d’utilisateur, mais il utilise ensuite une [stratégie personnalisée](custom-policy-get-started.md) pour interroger une [API REST](custom-policy-rest-api-intro.md) (que vous créez) pour définir le mot de passe de chaque utilisateur lors de la première connexion.

Le flux de migration fluide comporte ainsi deux phases : la *prémigration* et la *définition des informations d’identification*.

### <a name="phase-1-pre-migration"></a>Phase 1 : Prémigration

1. Votre application de migration lit les comptes d’utilisateur de l’ancien fournisseur d’identité.
1. L’application de migration crée les comptes d’utilisateur correspondants dans votre annuaire Azure AD B2C, mais *ne définit pas les mots de passe*.

### <a name="phase-2-set-credentials"></a>Phase 2 : Définition des informations d’identification

Une fois que la prémigration des comptes est terminée, votre stratégie et votre API REST personnalisées effectuent les opérations suivantes quand un utilisateur se connecte :

1. Lire le compte d’utilisateur Azure AD B2C correspondant à l’adresse e-mail entrée.
1. Vérifier si le compte est marqué pour migration en évaluant un attribut d’extension booléen.
    - Si l’attribut d’extension retourne `True`, appelez votre API REST pour vérifier le mot de passe par rapport au fournisseur d’identité précédent.
      - Si l’API REST détermine que le mot de passe est incorrect, retournez une erreur clairement formulée à l’utilisateur.
      - Si l’API REST détermine que le mot de passe est correct, écrivez le mot de passe dans le compte Azure AD B2C et passez la valeur de l’attribut d’extension booléen à `False`.
    - Si l’attribut d’extension booléen retourne `False`, poursuivez normalement le processus de connexion.

Pour voir un exemple de stratégie personnalisée et d’API REST, consultez l’[exemple de migration fluide d’utilisateurs](https://aka.ms/b2c-account-seamless-migration) sur GitHub.

![Diagramme de flux de l’approche de la migration fluide pour la migration d’utilisateurs](./media/user-migration/diagram-01-seamless-migration.png)<br />*Schéma : Flux de migration fluide*

## <a name="best-practices"></a>Meilleures pratiques

### <a name="security"></a>Sécurité

L’approche de la migration fluide utilise votre propre API REST personnalisée pour valider les informations d’identification d’un utilisateur par rapport au fournisseur d’identité précédent.

**Vous devez protéger votre API REST contre les attaques par force brute.** Un attaquant peut envoyer plusieurs mots de passe en espérant finalement deviner les informations d’identification d’un utilisateur. Pour aider à contrer de telles attaques, cessez de répondre aux demandes effectuées auprès de votre API REST quand le nombre de tentatives de connexion dépasse un certain seuil. Sécurisez aussi la communication entre Azure AD B2C et votre API REST. Pour savoir comment sécuriser vos API RESTful pour la production, consultez [API RESTful sécurisée](secure-rest-api.md).

### <a name="user-attributes"></a>Attributs utilisateur

Toutes les informations du fournisseur d’identité précédent ne doivent pas être migrées vers votre annuaire Azure AD B2C. Identifiez l’ensemble approprié d’attributs utilisateur à stocker dans Azure AD B2C avant d’effectuer la migration.

- **STOCKER** dans Azure AD B2C
  - Nom d’utilisateur, mot de passe, adresses e-mail, numéros de téléphone, numéros d’appartenance/identificateurs.
  - Marqueurs de consentement pour la politique de confidentialité et contrats de licence utilisateur final.
- **NE PAS STOCKER** dans Azure AD B2C
  - Données sensibles comme des numéros de carte de crédit, des numéros de sécurité sociale, des dossiers médicaux ou d’autres données réglementées par des organismes de conformité nationaux ou sectoriels.
  - Préférences marketing ou de communication, comportements des utilisateurs et insights.

### <a name="directory-clean-up"></a>Nettoyage de l’annuaire

Avant de commencer le processus de migration, vous pouvez en profiter pour nettoyer votre annuaire.

- Identifiez l’ensemble des attributs utilisateur à stocker dans Azure AD B2C et migrez seulement ce dont vous avez besoin. Si nécessaire, vous pouvez créer des [attributs personnalisés](user-flow-custom-attributes.md) pour stocker davantage de données sur un utilisateur.
- Si vous effectuez une migration à partir d’un environnement avec plusieurs sources d’authentification (par exemple chaque application a son propre annuaire d’utilisateurs), migrez vers un compte unifié dans Azure AD B2C.
- Si plusieurs applications ont des noms d’utilisateur différents, vous pouvez les stocker dans un compte d’utilisateur Azure AD B2C en utilisant la collection d’identités. En ce qui concerne le mot de passe, laissez l’utilisateur en choisir un et définissez-le dans l’annuaire. Par exemple, avec la migration fluide, seul le mot de passe choisi doit être stocké dans le compte Azure AD B2C.
- Supprimez les comptes d’utilisateur inutilisés avant la migration ou ne migrez pas les comptes obsolètes.

### <a name="password-policy"></a>Stratégie de mot de passe

Si les comptes que vous migrez ont un mot de passe moins fort que ce qui est prescrit par les [règles de mot de passe fort](../active-directory/authentication/concept-sspr-policy.md) appliquées par Azure AD B2C, vous pouvez désactiver l’exigence d’un mot de passe fort. Pour plus d’informations, consultez [Propriété de la stratégie de mot de passe](manage-user-accounts-graph-api.md#password-policy-property).

## <a name="next-steps"></a>Étapes suivantes

Le dépôt [azure-ad-b2c/user-migration](https://github.com/azure-ad-b2c/user-migration) sur GitHub contient un exemple de stratégie personnalisée de migration fluide et un exemple de code d’API REST :

[Exemple de stratégie et d’API REST personnalisées pour la migration fluide d’utilisateurs](https://aka.ms/b2c-account-seamless-migration)
