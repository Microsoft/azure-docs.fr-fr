---
title: Versions de flux utilisateur héritées dans Azure Active Directory B2C | Microsoft Docs
description: Découvrez les versions héritées de flux utilisateur disponibles dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 7cdfd6cb8947fb3b56fcbfe92874c8b9b27a8db1
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97108125"
---
# <a name="legacy-user-flow-versions-in-azure-active-directory-b2c"></a>Versions de flux utilisateur héritées dans Azure Active Directory B2C

> [!IMPORTANT]
> Cet article décrit la méthode de contrôle de version héritée pour les flux utilisateurs, qui offre des versions V1 (prêtes pour la production), ainsi que V1.1 et V2 (préversion) des flux utilisateur. Des environnements autres que le cloud public Azure continuent d’utiliser cette méthode de contrôle de version héritée. Dans le cloud public Azure, cette méthode est remplacée par les [versions Nouvelle **Recommandée** et **Préversion**](user-flow-versions.md).
> 
Les flux utilisateur dans Azure Active Directory B2C (Azure AD B2C) vous aident à configurer des [stratégies](user-flow-overview.md) courantes qui décrivent entièrement les expériences liées à l’identité des clients. Ces expériences incluent l’inscription, la connexion, la réinitialisation du mot de passe et la modification du profil.

Dans le tableau ci-dessous, si un flux utilisateur n’est pas identifié comme **Recommandé**, il est considéré comme étant en *Préversion*. Veillez à n’utiliser que des flux d’utilisateur recommandés pour vos applications de production.

## <a name="v1"></a>V1

| Flux utilisateur | Recommandé | Description |
| --------- | ----------- | ----------- |
| Réinitialisation du mot de passe | Oui | Permet à un utilisateur de choisir un nouveau mot de passe après la vérification de son adresse e-mail. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>Paramètres de conformité de jeton</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |
| Modification de profil | Oui | Permet à un utilisateur de configurer ses attributs utilisateur. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li></ul> |
| Se connecter à l’aide de ROPC | Non | Permet à un utilisateur avec un compte local de se connecter directement à des applications natives (aucun navigateur n’est requis). À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li></ul> |
| Se connecter | Non | Permet à un utilisateur de se connecter à son compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>Bloquer la connexion</li><li>Forcer la réinitialisation du mot de passe</li><li>Maintenir la connexion</ul><br>Vous ne pouvez pas personnaliser l’interface utilisateur avec ce flux utilisateur. |
| Inscription | Non | Permet à un utilisateur de créer un compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |
| Inscription et connexion | Oui | Permet à un utilisateur de créer un compte ou de se connecter à son compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul>|

## <a name="v11"></a>V 1.1

| Flux utilisateur | Recommandé | Description |
| --------- | ----------- | ----------- |
| Réinitialisation de mot de passe v1.1 | Non | Permet à un utilisateur de choisir un nouveau mot de passe après la vérification de son adresse e-mail (nouvelle mise en page disponible). À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>Paramètres de conformité de jeton</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |

## <a name="v2"></a>V2

| Flux utilisateur | Recommandé | Description |
| --------- | ----------- | ----------- |
| Réinitialisation de mot de passe v2 | Non | Permet à un utilisateur de choisir un nouveau mot de passe après la vérification de son adresse e-mail. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>Paramètres de conformité de jeton</li><li>[Vérification de l’âge](basic-age-gating.md)</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |
| Modification du profil v2 | Oui | Permet à un utilisateur de configurer ses attributs utilisateur. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li></ul> |
| Connexion v2 | Non | Permet à un utilisateur de se connecter à son compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>[Vérification de l’âge](basic-age-gating.md)</li><li>Personnalisation de la page de connexion</li></ul> |
| Inscription v2 | Non | Permet à un utilisateur de créer un compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>[Vérification de l’âge](basic-age-gating.md)</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |
| Inscription et connexion v2 | Non | Permet à un utilisateur de créer un compte ou de se connecter à son compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Vérification de l’âge](basic-age-gating.md)</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |