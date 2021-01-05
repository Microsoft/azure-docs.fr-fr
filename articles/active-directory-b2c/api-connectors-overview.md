---
title: À propos des connecteurs d’API dans Azure AD B2C
description: Utilisez les connecteurs d’API Azure Active Directory (Azure AD) pour personnaliser et étendre vos flux d’utilisateurs d’inscription à l’aide d’API Web.
services: active-directory-b2c
ms.service: active-directory
ms.subservice: B2C
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
ms.openlocfilehash: 94d6b0192b014396f8751e58f5620aec5c132203
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92503876"
---
# <a name="use-api-connectors-to-customize-and-extend-sign-up-user-flows"></a>Utiliser des connecteurs d’API pour personnaliser et étendre les flux d’utilisateurs d’inscription

> [!IMPORTANT]
> Les connecteurs d’API pour l’inscription sont une fonctionnalité en préversion publique d’Azure AD B2C. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="overview"></a>Vue d’ensemble 
En tant que développeur ou administrateur informatique, vous pouvez utiliser des connecteurs d’API pour intégrer vos flux d’utilisateur d’inscription avec des API Web pour personnaliser l’inscription. Par exemple, avec les connecteurs d’API, vous pouvez :

- **Valider des données d’entrée utilisateur** . Validez par rapport à des données utilisateur incorrectes ou non valides. Par exemple, vous pouvez valider les données fournies par l’utilisateur par rapport à des données existantes dans une banque de données externe ou une liste de valeurs autorisées. En cas d’invalidité, vous pouvez demander à un utilisateur de fournir des données valides ou empêcher l’utilisateur de continuer le processus d’inscription.
- **Intégrer à un flux de travail d’approbation personnalisé** . Connectez-vous à un système d’approbation personnalisé pour gérer et limiter la création de comptes.
- **Remplacer des attributs utilisateur** . Reformatez ou attribuez une valeur à un attribut collecté auprès de l’utilisateur. Par exemple, si un utilisateur entre le prénom entier en lettres minuscules ou majuscules, vous pouvez modifier sa mise en forme en utilisant une majuscule uniquement pour la première lettre. 
- **Exécuter une vérification d’identité** . Utilisez un service de vérification d’identité pour ajouter un niveau supplémentaire de sécurité aux décisions de création de comptes.
- **Exécuter une logique métier personnalisée** . Vous pouvez déclencher des événements en aval dans vos systèmes ce cloud pour envoyer des notifications Push, mettre à jour des bases de données d’entreprise, gérer des autorisations, auditer des bases de données et effectuer d’autres actions personnalisées.

Un connecteur d’API fournit à Azure Active Directory les informations nécessaires pour appeler une API en incluant l’URL de point de terminaison et l’authentification. Une fois que vous avez configuré un connecteur d’API, vous pouvez l’activer pour une étape spécifique dans un flux utilisateur. Quand un utilisateur atteint cette étape dans le flux d’inscription, le connecteur d’API est appelé et se matérialise en tant que requête HTTP POST à votre API, en envoyant des informations utilisateur (« revendications ») en tant que paires clé-valeur dans un corps JSON. La réponse d’API peut affecter l’exécution du flux utilisateur. Par exemple, la réponse d’API peut empêcher un utilisateur de s’inscrire, demander à l’utilisateur d’entre à nouveau des informations ou ajouter les attributs de l’utilisateur.

## <a name="where-you-can-enable-an-api-connector-in-a-user-flow"></a>Où activer un connecteur d’API dans un flux utilisateur

Il existe deux emplacements dans un flux utilisateur où vous pouvez activer un connecteur d’API :

- après la connexion avec un fournisseur d’identité
- avant la création de l’utilisateur

> [!IMPORTANT]
> Dans ces deux cas, les connecteurs d’API sont appelés lors de l’ **inscription** de l’utilisateur et non de la connexion.

### <a name="after-signing-in-with-an-identity-provider"></a>après la connexion avec un fournisseur d’identité

Un connecteur d’API à cette étape du processus d’inscription est appelé immédiatement après que l’utilisateur s’est authentifié auprès d’un fournisseur d’identité (comme Google, Facebook et Azure AD). Cette étape précède la **_page de collection d’attributs_** , qui est le formulaire présenté à l’utilisateur pour collecter des attributs utilisateur. Cette étape n’est pas appelée si un utilisateur s’inscrit auprès d’un compte local. Voici quelques exemples de scénarios de connecteur d’API que vous pouvez activer à cette étape :

- Utilisez l’adresse e-mail ou l’identité fédérée que l’utilisateur a fournie pour rechercher des revendications dans un système existant. Retournez ces revendications à partir du système existant, complétez la page de collection d’attributs, puis rendez-les disponibles pour retourner dans le jeton.
- Implémentez une liste d’autorisation ou de blocage en fonction de l’identité sociale.

### <a name="before-creating-the-user"></a>avant la création de l’utilisateur

Un connecteur d’API à cette étape du processus d’inscription est appelé après la page de collection d’attributs, si elle est inclue. Cette étape est toujours appelée avant qu’un compte d’utilisateur ne soit créé. Voici quelques exemples de scénarios que vous pouvez activer à cette étape au cours de l’inscription :

- Validez les données d’entrée utilisateur et demandez à un utilisateur de renvoyer des données.
- Bloquer l’inscription utilisateur en fonction des données entrées par l’utilisateur.
- Exécuter une vérification d’identité.
- Interrogez des systèmes externes pour obtenir des données existantes sur l’utilisateur afin de les retourner dans le jeton d’application ou de les stocker dans Azure AD.


## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [ajouter un connecteur d’API à un flux d’utilisateur](add-api-connector.md)
- Bien commencer avec nos [exemples](code-samples.md#api-connectors).
<!-- - Learn how to [add a custom approval system to self-service sign-up](add-approvals.md) -->