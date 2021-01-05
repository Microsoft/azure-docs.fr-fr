---
title: 'Azure AD Connect : authentification directe | Microsoft Docs'
description: Cet article présente l’authentification directe Azure Active Directory (Azure AD) et explique comment l’utiliser pour autoriser les connexions à Azure AD en validant les mots de passe des utilisateurs à partir d’Active Directory local.
services: active-directory
keywords: qu’est-ce que l’authentification directe Azure AD Connect, installation d’Active Directory, composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/21/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0b8f613cb7c75d9dd6af1fcf62f9d484398072c6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89279463"
---
# <a name="user-sign-in-with-azure-active-directory-pass-through-authentication"></a>Connexion de l’utilisateur avec l’authentification directe Azure Active Directory

## <a name="what-is-azure-active-directory-pass-through-authentication"></a>Qu’est-ce que l’authentification directe Azure Active Directory ?

L’authentification directe Azure Active Directory (Azure AD) permet à vos utilisateurs de se connecter aux applications locales et aux applications cloud à l’aide des mêmes mots de passe. Cette fonctionnalité offre aux utilisateurs une meilleure expérience : moins de mots de passe à mémoriser et réduction des coûts de support technique informatique, car les utilisateurs sont moins susceptibles d’oublier comment se connecter. Lorsque les utilisateurs se connectent à l’aide d’Azure AD, cette fonctionnalité valide les mots de passe utilisateurs directement à partir d’Active Directory local.

>[!VIDEO https://www.youtube.com/embed/PyeAC85Gm7w]

Cette fonctionnalité est une alternative à [Synchronisation du hachage de mot de passe Azure AD](how-to-connect-password-hash-synchronization.md), qui offre les mêmes fonctionnalités d’authentification sur le cloud aux organisations. Toutefois, certaines organisations souhaitant appliquer leurs stratégies de mot de passe et de sécurité Active Directory locales, peuvent choisir d’utiliser l’authentification directe à la place. Consultez [ce guide](./choose-ad-authn.md) pour voir une comparaison entre les différentes méthodes de connexion Azure AD, et savoir comment choisir la méthode de connexion appropriée pour votre organisation.

![Authentification directe Azure AD](./media/how-to-connect-pta/pta1.png)

Vous pouvez combiner l’authentification directe avec la fonctionnalité [Authentification unique transparente](how-to-connect-sso.md). De cette façon, lorsque vos utilisateurs accèdent à leurs ordinateurs d’entreprise au sein de votre réseau d’entreprise, ils n’ont pas besoin de saisir leurs mots de passe pour se connecter.

## <a name="key-benefits-of-using-azure-ad-pass-through-authentication"></a>Avantages clés de utilisation de l’authentification directe Azure AD

- *Une meilleure expérience utilisateur*
  - Les utilisateurs utilisent les mêmes mots de passe pour se connecter aux applications locales et sur le cloud.
  - Les utilisateurs passent moins de temps à communiquer avec le service d’assistance informatique pour résoudre les problèmes de mot de passe.
  - Les utilisateurs peuvent effectuer les tâches de [gestion de mot de passe en libre-service](../authentication/concept-sspr-howitworks.md) dans le cloud.
- *Facile à déployer et à gérer*
  - Des déploiements locaux ou une configuration réseau complexes ne sont pas nécessaires.
  - Seul un agent léger doit être installé localement.
  - Aucun frais de gestion. L’agent reçoit automatiquement des améliorations et correctifs de bogues.
- *Sécuriser*
  - Les mots de passe locaux ne sont jamais stockés dans le cloud sous quelque forme que ce soit.
  - Il protège vos comptes utilisateur en toute transparence avec les [stratégies d’accès conditionnel d’Azure AD](../conditional-access/overview.md), y compris l’authentification multifacteur (MFA), [en bloquant l’authentification héritée](../conditional-access/concept-conditional-access-conditions.md) et [en filtrant des attaques de mot de passe par recherche exhaustive](../authentication/howto-password-smart-lockout.md).
  - L’agent établit uniquement les connexions sortantes depuis votre réseau. Il n’est donc pas nécessaire d’installer l’agent dans un réseau de périmètre, également appelé DMZ.
  - La communication entre les agents et Azure AD est sécurisée à l’aide de l’authentification basée sur les certificats. Ces certificats sont renouvelés automatiquement tous les quelques mois par Azure AD.
- *Hautement disponible*
  - Il est possible d’installer des agents supplémentaires sur plusieurs serveurs locaux pour assurer la haute disponibilité des requêtes de connexion.

## <a name="feature-highlights"></a>Présentation des fonctionnalités

- Prend en charge la connexion de l’utilisateur dans toutes les applications basées sur un navigateur web et dans les applications clientes Microsoft Office qui utilisent [l’authentification moderne](https://aka.ms/modernauthga).
- Les noms d’utilisateur de connexion peuvent être soit un nom d’utilisateur local par défaut (`userPrincipalName`), soit un autre attribut configuré dans Azure AD Connect (appelé `Alternate ID`).
- Elle s’utilise très facilement avec les fonctionnalités d’[accès conditionnel](../conditional-access/overview.md) (telles que l’authentification multifacteur) pour la sécurisation de vos utilisateurs.
- Elle est intégrée à la [gestion des mots de passe libre-service](../authentication/concept-sspr-howitworks.md) sur le cloud, y compris la réécriture des mots de passe dans l’annuaire Active Directory local et la protection par mot de passe en interdisant l’emploi de mots de passe courants.
- Les environnements à plusieurs forêts sont pris en charge s’il existe des approbations de forêts entre les forêts AD et si le routage du suffixe de leurs noms est configuré correctement.
- Cette fonctionnalité est gratuite et il est inutile de disposer des éditions payantes d’Azure AD pour l’utiliser.
- Elle peut être activée via [Azure AD Connect](whatis-hybrid-identity.md).
- Elle utilise un agent local léger qui écoute et répond aux requêtes de validation du mot de passe.
- L’installation de plusieurs agents fournit une haute disponibilité des requêtes de connexion.
- Elle permet de [protéger](../authentication/howto-password-smart-lockout.md) vos comptes locaux contre les attaques de mot de passe par recherche exhaustive dans le cloud.

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide](how-to-connect-pta-quick-start.md) : commencez à utiliser l’authentification directe Azure AD.
- [Migrer à partir d’AD FS vers l’authentification directe](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx?raw=true) : guide détaillé de la migration d’AD FS (ou d’autres technologies de fédération) vers l’authentification directe.
- [Verrouillage intelligent](../authentication/howto-password-smart-lockout.md) : configurez la fonctionnalité Verrouillage intelligent sur votre locataire pour protéger les comptes d’utilisateur.
- [Limitations actuelles](how-to-connect-pta-current-limitations.md) : découvrez les scénarios pris en charge et ceux qui ne le sont pas.
- [Immersion technique](how-to-connect-pta-how-it-works.md) : découvrez comment fonctionne cette fonctionnalité.
- [Questions fréquentes (FAQ)](how-to-connect-pta-faq.md) : réponses aux questions fréquentes.
- [Résolution des problèmes](tshoot-connect-pass-through-authentication.md) : découvrez comment résoudre les problèmes courants susceptibles de se produire avec cette fonctionnalité.
- [Immersion dans la sécurité](how-to-connect-pta-security-deep-dive.md) : informations techniques approfondies supplémentaires sur la fonctionnalité.
- [Authentification unique fluide Azure AD](how-to-connect-sso.md) : explorez en détail cette fonctionnalité complémentaire.
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) : pour le dépôt de nouvelles demandes de fonctionnalités.