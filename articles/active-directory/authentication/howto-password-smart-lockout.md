---
title: Prévention des attaques avec le verrouillage intelligent - Azure Active Directory
description: Découvrez comment le verrouillage intelligent d’Azure Active Directory permet de protéger votre organisation contre les attaques par force brute visant à deviner les mots de passe des utilisateurs.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/20/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: b8a57e77ea572f899bf540714e8ac9968988f028
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741726"
---
# <a name="protect-user-accounts-from-attacks-with-azure-active-directory-smart-lockout"></a>Protéger les comptes d’utilisateur contre les attaques avec le verrouillage intelligent Azure Active Directory

Le verrouillage intelligent empêche les personnes malveillantes de deviner vos mots de passe ou d’utiliser des méthodes de force brute pour rentrer dans vos systèmes. Le verrouillage intelligent peut reconnaître les connexions provenant d’utilisateurs validés et les traiter différemment de celles des attaquants et autres sources inconnues. Le verrouillage intelligent empêche les attaquants de pénétrer dans le système, tout en permettant à vos utilisateurs d’accéder à leurs comptes et de travailler.

## <a name="how-smart-lockout-works"></a>Fonctionnement du verrouillage intelligent

Par défaut, le verrouillage intelligent empêche les tentatives de connexion au compte pendant une minute après 10 tentatives infructueuses. Le compte se verrouille à nouveau après chaque échec de connexion consécutif. Le premier verrouillage dure une minute, les suivants durent plus longtemps. Afin de minimiser les moyens dont dispose un attaquant pour contourner ce comportement, nous ne divulguons pas le rythme auquel la période de verrouillage s’allonge au fil des tentatives de connexion infructueuses.

Le verrouillage intelligent suit les trois derniers hachages de mots de passe incorrects afin d'éviter d'incrémenter le compteur de verrouillages pour le même mot de passe. Si un utilisateur entre plusieurs fois le même mot de passe incorrect, le compte n'est pas verrouillé.

> [!NOTE]
> La fonctionnalité de suivi du hachage n’est pas disponible pour les clients sur lesquels l’authentification directe est activée dans la mesure où l’authentification est effectuée localement et pas dans le cloud.

Les déploiements fédérés utilisant AD FS 2016 et AD FS 2019 peuvent bénéficier d’avantages similaires à ceux fournis par l’utilisation du [verrouillage extranet et du verrouillage intelligent extranet AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection).

Le verrouillage intelligent est activé en permanence pour les clients Azure AD disposant des paramètres par défaut qui offrent la combinaison idéale de sécurité et de facilité d’utilisation. Pour personnaliser les paramètres de verrouillage intelligent en vue de répondre aux besoins de votre organisation, vos utilisateurs doivent disposer d’une licence Azure AD Premium P1 ou plus élevée.

L’utilisation du verrouillage intelligent ne garantit pas qu’un véritable utilisateur n’est jamais verrouillé. Lorsque le verrouillage intelligent verrouille un compte d'utilisateur, nous mettons tout en œuvre pour ne pas verrouiller le véritable utilisateur. Le service de verrouillage veille à ce que des personnes mal intentionnées n’aient pas accès au compte d’un véritable utilisateur. Les considérations suivantes s'appliquent :

* Chaque centre de données Azure AD assure indépendamment le suivi des verrouillages. Un utilisateur dispose d'un nombre de tentatives de (*seuil_limite x nombre_de_centres_de_données*) s'il atteint chaque centre de données.
* Le verrouillage intelligent utilise un emplacement familier par opposition à un emplacement inconnu pour faire la différence entre une personne mal intentionnée et un véritable utilisateur. Les emplacements familiers et inconnus disposent tous deux de compteurs de verrouillages distincts.

Le verrouillage intelligent peut être intégré aux déploiements hybrides à l’aide de la synchronisation du hachage de mot de passe ou de l’authentification directe, en vue d’empêcher les comptes Active Directory Domain Services (AD DS) locaux d’être verrouillés par les attaquants. En définissant les stratégies de verrouillage intelligent nécessaires dans Azure AD, vous pouvez bloquer les attaques avant même qu’elles atteignent l’instance locale d’AD DS.

Lorsque vous utilisez [l’authentification directe](../hybrid/how-to-connect-pta.md), les considérations suivantes s’appliquent :

* Le seuil de verrouillage d’AD DS est **inférieur** au seuil de verrouillage de compte Active Directory. Définissez les valeurs de sorte que le seuil de verrouillage de compte Active Directory soit au moins deux ou trois fois supérieur au seuil de verrouillage d’AD DS.
* La durée de verrouillage d’Azure AD doit être plus longue que la durée après laquelle réinitialiser le compteur de verrouillage de compte AD DS. La durée d’Azure AD est définie en secondes, tandis que la durée d’AD est définie en minutes.

Par exemple, si vous souhaitez que votre compteur Azure AD soit supérieur à celui d’AD DS, Azure AD doit être défini sur 120 secondes (2 minutes) tandis que votre AD local doit être défini sur 1 minute (60 secondes).

> [!IMPORTANT]
> Actuellement, un administrateur ne peut pas déverrouiller les comptes cloud des utilisateurs s’ils ont été verrouillés à l’aide de la fonctionnalité Verrouillage intelligent. L’administrateur doit attendre que la durée de verrouillage expire. En revanche, l’utilisateur peut procéder à un déverrouillage en utilisant la réinitialisation de mot de passe en libre-service (SSPR) à partir d’un appareil ou d’un emplacement de confiance.

## <a name="verify-on-premises-account-lockout-policy"></a>Vérifier la stratégie de verrouillage d’un compte local

Pour vérifier votre stratégie de verrouillage de compte AD DS locale, effectuez les étapes suivantes à partir d’un système joint à un domaine avec des privilèges d’administrateur :

1. Ouvrez l’outil de gestion de stratégie de groupe.
2. Modifiez la stratégie de groupe qui comprend la stratégie de verrouillage de compte de votre organisation, par exemple, la **stratégie de domaine par défaut**.
3. Accédez à **Configuration de l’ordinateur** > **Stratégies** > **Paramètres Windows** > **Paramètres de sécurité** > **Stratégies de compte** > **Stratégie de verrouillage de compte**.
4. Vérifiez vos valeurs de **Seuil de verrouillage de compte** et **Réinitialiser le compteur de verrouillage de compte après**.

![Modifier la stratégie de verrouillage des comptes locaux Active Directory](./media/howto-password-smart-lockout/active-directory-on-premises-account-lockout-policy.png)

## <a name="manage-azure-ad-smart-lockout-values"></a>Gérer les valeurs du verrouillage intelligent Azure AD

En fonction des exigences de votre organisation, vous pouvez personnaliser les valeurs du verrouillage intelligent Azure AD. Pour personnaliser les paramètres de verrouillage intelligent en vue de répondre aux besoins de votre organisation, vos utilisateurs doivent disposer d’une licence Azure AD Premium P1 ou plus élevée.

Pour vérifier ou modifier les valeurs de verrouillage intelligent de votre organisation, procédez aux étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez et sélectionnez *Azure Active Directory*, puis sélectionnez **Sécurité** > **Méthodes d’authentification** > **Protection par mot de passe**.
1. Définissez le **Seuil de verrouillage**, c’est-à-dire le nombre d’échecs de connexions autorisé avant qu’un compte ne soit verrouillé.

    La valeur par défaut est de 10.

1. Définissez la **Durée du verrouillage en secondes** sur la durée en secondes souhaitée de chaque verrouillage.

    La valeur par défaut est 60 secondes (une minute).

> [!NOTE]
> Si la première connexion après verrouillage échoue également, le compte est de nouveau verrouillé. Si un compte est verrouillé à plusieurs reprises, la durée de verrouillage augmente.

![Personnaliser la stratégie de verrouillage intelligent Azure AD dans le portail Azure](./media/howto-password-smart-lockout/azure-active-directory-custom-smart-lockout-policy.png)

## <a name="how-to-determine-if-the-smart-lockout-feature-is-working-or-not"></a>Vérifier le bon fonctionnement de la fonctionnalité Verrouillage intelligent

Lorsque le seuil de verrouillage intelligent est déclenché, le message suivant s'affiche en cas de verrouillage du compte :

*Votre compte est verrouillé de façon temporaire afin d'éviter toute utilisation non autorisée. Réessayez plus tard. Si le problème persiste, contactez votre administrateur.*

## <a name="next-steps"></a>Étapes suivantes

Pour personnaliser davantage l’expérience, vous pouvez [configurer des mots de passe interdits personnalisés pour la protection par mot de passe Azure AD](tutorial-configure-custom-password-protection.md).

Pour aider les utilisateurs à réinitialiser ou à modifier leur mot de passe à partir d’un navigateur web, vous pouvez [configurer la réinitialisation de mot de passe en libre-service Azure AD](tutorial-enable-sspr.md).
