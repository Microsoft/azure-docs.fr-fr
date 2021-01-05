---
title: Vue d’ensemble de l’authentification Azure Active Directory
description: Découvrez les différentes méthodes d’authentification et fonctionnalités de sécurité pour les connexions utilisateur avec Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: overview
ms.date: 07/13/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: sahenry, michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: f85b78480e7258780f8dd396431edeb968189e4c
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741250"
---
# <a name="what-is-azure-active-directory-authentication"></a>Qu’est-ce que l’authentification Azure Active Directory ?

L’une des principales fonctionnalités d’une plateforme d’identités consiste à vérifier, ou *authentifier*, les informations d’identification quand l’utilisateur se connecte à un appareil, à une application ou à un service. Dans Azure Active Directory (Azure AD), l’authentification implique plus que la simple vérification d’un nom d’utilisateur et d’un mot de passe. Pour améliorer la sécurité et réduire le recours à l’assistance du support technique, l’authentification Azure AD comprend les composants suivants :

* Réinitialisation de mot de passe libre-service
* Authentification multifacteur Azure AD
* Intégration hybride pour écrire les changements de mot de passe dans l’environnement local
* Intégration hybride afin d’appliquer des stratégies de protection par mot de passe pour un environnement local
* Authentification sans mot de passe

## <a name="improve-the-end-user-experience"></a>Améliorer l’expérience de l’utilisateur final

Azure AD aide à protéger l’identité de l’utilisateur et à simplifier son expérience de la connexion. Les fonctionnalités telles que la réinitialisation de mot de passe en libre-service permettent aux utilisateurs de mettre à jour ou de changer leurs mots de passe à l’aide d’un navigateur web à partir de l’appareil de leur choix. Cette fonctionnalité est particulièrement utile quand l’utilisateur a oublié son mot de passe ou que son compte est verrouillé. L’utilisateur peut se débloquer et continuer à travailler sans attendre qu’un agent du support technique ou un administrateur lui fournisse une assistance.

L’authentification multifacteur Azure AD permet aux utilisateurs de choisir une forme d’authentification supplémentaire lors de la connexion, par exemple un appel téléphonique ou une notification d’application mobile. L’utilisateur n’a plus besoin d’avoir une seule forme fixe d’authentification secondaire, telle qu’un jeton matériel. Si l’utilisateur n’a pas de forme d’authentification supplémentaire, il peut choisir une autre méthode et continuer à travailler.

![Méthodes d’authentification en cours d’utilisation sur l’écran de connexion](media/concept-authentication-methods/overview-login.png)

L’authentification sans mot de passe évite à l’utilisateur de créer et de mémoriser un mot de passe sécurisé. Les fonctionnalités, telles que les clés de sécurité FIDO2 ou Windows Hello Entreprise, permettent aux utilisateurs de se connecter à un appareil ou à une application sans mot de passe. Elles peuvent ainsi réduire la complexité de gérer les mots de passe dans différents environnements.

## <a name="self-service-password-reset"></a>Réinitialisation de mot de passe libre-service

La réinitialisation de mot de passe en libre-service permet aux utilisateurs de changer ou de réinitialiser leur mot de passe, sans intervention d’un administrateur ou d’un agent du support technique. Si le compte d’un utilisateur est verrouillé ou si ce dernier oublie son mot de passe, il peut suivre des invites afin de se débloquer et de reprendre son travail. Cette fonctionnalité réduit les appels au support technique et la perte de productivité quand l’utilisateur ne parvient pas à se connecter à son appareil ou à une application.

La réinitialisation de mot de passe en libre-service fonctionne dans les scénarios suivants :

* **Changement de mot de passe :** L’utilisateur connaît son mot de passe, mais il souhaite le changer.
* **Réinitialisation de mot de passe :** L’utilisateur ne peut pas se connecter, par exemple s’il a oublié son mot de passe, et souhaite réinitialiser ce dernier.
* **Déverrouillage de compte :** L’utilisateur ne peut pas se connecter car son compte est verrouillé et souhaite déverrouiller ce dernier.

Quand l’utilisateur met à jour ou réinitialise son mot de passe à l’aide de la réinitialisation de mot de passe en libre-service, ce mot de passe peut également être réécrit dans un environnement Active Directory local. Avec la réécriture du mot de passe, l’utilisateur peut immédiatement utiliser ses informations d’identification mises à jour avec les applications et les appareils locaux.

## <a name="azure-ad-multi-factor-authentication"></a>Authentification multifacteur Azure AD

L’authentification multifacteur est un processus dans lequel l’utilisateur est invité pendant le processus de connexion à suivre une forme d’identification supplémentaire, consistant par exemple à entrer un code sur son téléphone portable ou à scanner son empreinte digitale.

L’utilisation d’un mot de passe uniquement ne protège pas complètement des attaques. Si le mot de passe est faible ou s’il a été exposé ailleurs, est-ce vraiment l’utilisateur qui se connecte avec le nom d’utilisateur et le mot de passe, ou s’agit-il d’un attaquant ? Quand vous exigez une deuxième forme d’authentification, la sécurité est accrue, car ce facteur supplémentaire n’est pas un élément facile à obtenir ou à dupliquer par un attaquant.

![Image conceptuelle des différentes formes d’authentification multifacteur](./media/concept-mfa-howitworks/methods.png)

L’authentification multifacteur Azure AD impose au minimum deux des méthodes d’authentification suivantes :

* Un élément que vous connaissez, généralement un mot de passe.
* Un élément que vous possédez, tel qu’un appareil de confiance qui n’est pas facilement dupliqué, comme un téléphone ou une clé matérielle.
* Un élément biométrique identifiant votre personne, tel qu’une empreinte digitale ou un scan du visage.

Les utilisateurs peuvent s’inscrire à la réinitialisation de mot de passe en libre-service et à l’authentification multifacteur Azure AD en une seule étape pour simplifier l’expérience d’intégration. Les administrateurs peuvent définir les formes d’authentification secondaire qui peuvent être utilisées. L’authentification multifacteur Azure AD peut également être nécessaire quand les utilisateurs effectuent une réinitialisation de mot de passe en libre-service pour sécuriser davantage ce processus.

## <a name="password-protection"></a>Protection par mot de passe

Par défaut, Azure AD bloque les mots de passe faibles tels que *Motdepasse1*. Une liste globale de mots de passe interdits est automatiquement mise à jour et appliquée, qui comprend des mots de passe faibles connus. Si l’utilisateur Azure AD tente de définir son mot de passe sur l’un de ces mots de passe faibles, il reçoit une notification lui demandant de choisir un mot de passe plus sécurisé.

Pour renforcer la sécurité, vous pouvez définir des stratégies de protection par mot de passe personnalisées. Ces stratégies peuvent utiliser des filtres pour bloquer toute variante d’un mot de passe contenant un nom comme *Contoso* ou un lieu comme *Londres*, par exemple.

Pour la sécurité hybride, vous pouvez intégrer la protection par mot de passe Azure AD à un environnement Active Directory local. Un composant installé dans l’environnement local reçoit la liste globale de mots de passe interdits et les stratégies de protection par mot de passe personnalisées d’Azure AD, et les contrôleurs de domaine les utilisent pour traiter les événements de changement de mot de passe. Cette approche hybride permet de s’assurer que, quel que soit où et comment l’utilisateur change ses informations d’identification, vous appliquez l’utilisation de mots de passe forts.

## <a name="passwordless-authentication"></a>Authentification sans mot de passe

L’objectif final de nombreux environnements est de supprimer l’utilisation de mots de passe dans le cadre des événements de connexion. Les fonctionnalités telles que la protection par mot de passe Azure ou l’authentification multifacteur Azure AD permettent d’améliorer la sécurité, mais un nom d’utilisateur et un mot de passe restent une forme d’authentification faible qui peut être exposée ou faire l’objet d’une attaque par force brute.

![Sécurité versus côté pratique avec le processus d’authentification sans mot de passe](./media/concept-authentication-passwordless/passwordless-convenience-security.png)

Quand vous vous connectez avec une méthode sans mot de passe, les informations d’identification sont fournies par le biais de l’utilisation de méthodes telles que la biométrie avec Windows Hello Entreprise ou une clé de sécurité FIDO2. Ces méthodes d’authentification ne peuvent pas être facilement dupliquées par un attaquant.

Azure AD permet d’effectuer une authentification en mode natif à l’aide de méthodes non basées sur un mot de passe afin de simplifier l’expérience de connexion des utilisateurs et de réduire le risque d’attaques.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer, consultez le [tutoriel sur la réinitialisation du mot de passe en libre-service][tutorial-sspr] et l’[authentification multifacteur Azure AD][tutorial-azure-mfa].

Pour en savoir plus sur les concepts de réinitialisation de mot de passe en libre-service, consultez [Fonctionnement de la réinitialisation de mot de passe en libre-service dans Azure AD][concept-sspr].

Pour en savoir plus sur les concepts de l’authentification multifacteur, consultez [Comment fonctionne l’authentification multifacteur Azure AD][concept-mfa].

<!-- INTERNAL LINKS -->
[tutorial-sspr]: tutorial-enable-sspr.md
[tutorial-azure-mfa]: tutorial-enable-azure-mfa.md
[concept-sspr]: concept-sspr-howitworks.md
[concept-mfa]: concept-mfa-howitworks.md
