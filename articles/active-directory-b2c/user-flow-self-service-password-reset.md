---
title: Réinitialisation de mot de passe libre service dans Azure Active Directory B2C | Microsoft Docs
description: Explique comment configurer la réinitialisation de mot de passe en libre-service pour vos clients dans Azure Active Directory B2C
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 0c9edaf3356ea4c1a521a89f2ec60a4b6ba1a5ef
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87481493"
---
# <a name="set-up-self-service-password-reset-for-your-customers"></a>Configurer la réinitialisation de mot de passe en libre-service pour vos clients

La fonctionnalité de réinitialisation de mot de passe en libre-service permet à vos clients qui se sont inscrits pour obtenir des comptes locaux de réinitialiser eux-mêmes leurs mots de passe. Cela réduit considérablement la charge pesant sur votre personnel de support, surtout si votre application est utilisée régulièrement par des millions de clients. Pour l’instant, l’utilisation d’une adresse e-mail vérifiée est la seule méthode de récupération prise en charge.

> [!NOTE]
> Cet article s’applique à la réinitialisation de mot de passe libre-service utilisée dans le contexte d’un flux d’utilisateur de **connexion** standard, qui utilise **Local Account SignIn** comme fournisseur d’identité. Si vous avez besoin de flux d’utilisateur de réinitialisation de mot de passe entièrement personnalisables appelés à partir de votre application, consultez [cet article](user-flow-overview.md).
>
>

Par défaut, la réinitialisation de mot de passe en libre-service n’est pas activée pour votre répertoire. Pour l’activer, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur d’abonnements. Il s’agit du compte professionnel ou scolaire, ou du compte Microsoft que vous avez utilisé pour créer votre annuaire.
2. Ouvrez **Azure Active Directory** (dans la barre de navigation sur le côté gauche).
3. Faites défiler le panneau des options pour sélectionner **Réinitialisation de mot de passe**.
4. Définissez l’option **Réinitialisation du mot de passe en libre-service activée** sur **Tout**.
5. Cliquez sur **Enregistrer** dans la partie supérieure de la page. Vous avez terminé !

Pour tester, utilisez la fonctionnalité « Exécuter maintenant » sur un flux d’utilisateur de connexion (qui comporte des comptes locaux en tant que fournisseur d’identité). Dans la page de connexion au compte local (où vous entrez une adresse e-mail et un mot de passe, ou un nom d’utilisateur et un mot de passe), cliquez sur **Votre compte n’est pas accessible ?** pour vérifier l’expérience client.

> [!NOTE]
> Les pages de réinitialisation de mot de passe libre-service sont personnalisables à l’aide de la [fonctionnalité de personnalisation de la société](../active-directory/fundamentals/customize-branding.md).
>
>

