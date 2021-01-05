---
title: Activer l’inscription combinée des informations de sécurité – Azure Active Directory
description: Apprenez à simplifier l’expérience de l’utilisateur final grâce à l’inscription combinée à l’authentification multifacteur et à la réinitialisation de mot de passe en libre-service Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 12/04/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 29a577b50a561cb5b829e453c523e0bd18a70e1a
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741675"
---
# <a name="enable-combined-security-information-registration-in-azure-active-directory"></a>Activation de l’inscription combinée des informations de sécurité dans Azure Active Directory

Avant l'inscription combinée, les utilisateurs s'inscrivaient séparément aux méthodes d'authentification Azure AD Multi-Factor Authentication (Azure AD MFA) et Réinitialisation de mot de passe en libre-service (SSPR). Certains se demandaient pourquoi ils devaient s'inscrire aux deux fonctionnalités alors que des méthodes similaires étaient utilisées pour l'authentification Azure AD MFA et la réinitialisation SSPR. Désormais, grâce à l'inscription combinée, les utilisateurs n'ont plus besoin de s'inscrire deux fois pour bénéficier des avantages de l'authentification Azure AD MFA et de la réinitialisation SSPR.

> [!NOTE]
> À partir du 15 août 2020, l’inscription combinée sera automatiquement activée pour tous les nouveaux locataires Azure AD. À partir du 14 décembre 2020, l’inscription combinée sera automatiquement activée pour tous les locataires Azure AD ayant un utilisateur.

Pour vous assurer de bien comprendre la fonctionnalité et ses effets avant d’activer la nouvelle expérience, consultez l’article [Concepts de l’inscription combinée des informations de sécurité](concept-registration-mfa-sspr-combined.md).

![Amélioration de l’expérience d’inscription d’informations de sécurité combinée](media/howto-registration-mfa-sspr-combined/combined-security-info-more-required.png)

## <a name="enable-combined-registration"></a>Activer une inscription combinée

Pour activer l’inscription combinée, procédez comme suit :

1. Connectez-vous au portail Azure en tant qu’administrateur d’utilisateurs.
2. Accédez à **Azure Active Directory** > **Paramètres utilisateur** > **Gérer les paramètres de préversion de la fonctionnalité utilisateur**.
3. Sous **, les utilisateurs peuvent utiliser l’expérience combinée d’inscription des informations de sécurité**, choisissez d’autoriser un groupe d’utilisateurs **sélectionné** ou **tous** les utilisateurs.

   ![Activer l’expérience combinée d’inscription des informations de sécurité pour des utilisateurs](media/howto-registration-mfa-sspr-combined/enable-the-combined-security-info.png)

> [!NOTE]
> Une fois que vous activez l'inscription combinée, les utilisateurs qui s'inscrivent ou confirment leur numéro de téléphone ou une application mobile via la nouvelle expérience peuvent utiliser ceux-ci pour l'authentification Azure AD MFA et la réinitialisation SSPR, si ces méthodes sont activées dans les stratégies Azure AD Multi-Factor Authentication et SSPR.
>
> Si vous désactivez cette expérience par la suite, les utilisateurs qui accèdent à la précédente page d’inscription SSPR à l’adresse `https://aka.ms/ssprsetup` doivent effectuer une authentification multifacteur.

Si vous avez configuré la *Liste des attributions de sites aux zones* dans Internet Explorer, les sites suivants doivent être dans la même zone :

* *[https://login.microsoftonline.com](https://login.microsoftonline.com)*
* *[https://mysignins.microsoft.com](https://mysignins.microsoft.com)*
* *[https://account.activedirectory.windowsazure.com](https://account.activedirectory.windowsazure.com)*

## <a name="conditional-access-policies-for-combined-registration"></a>Stratégies d’accès conditionnel pour l’inscription combinée

Pour déterminer quand et comment les utilisateurs s'inscrivent à l'authentification Azure AD MFA et à la réinitialisation SSPR, vous pouvez utiliser les actions utilisateur de la stratégie d'accès conditionnel. Cette fonctionnalité peut être activée dans les organisations qui souhaitent que les utilisateurs s'inscrivent à l'authentification Azure AD MFA et à la réinitialisation SSPR à partir d'un emplacement central, tel qu'un emplacement réseau approuvé lors de l'intégration des nouveaux employés.

> [!NOTE]
> Cette stratégie s’applique uniquement lorsqu’un utilisateur accède à une page d’inscription combinée. Cette stratégie n’applique pas l’inscription MFA lorsqu’un utilisateur accède à d’autres applications.
>
> Vous pouvez créer une stratégie d’inscription MFA à l’aide de [Azure Identity Protection - Configurer la stratégie MFA](../identity-protection/howto-identity-protection-configure-mfa-policy.md).

Pour plus d’informations sur la création d’emplacements approuvés dans l’accès conditionnel, consultez [Qu’est-ce que la condition d’emplacement dans l’accès conditionnel Azure Active Directory ?](../conditional-access/location-condition.md#named-locations)

### <a name="create-a-policy-to-require-registration-from-a-trusted-location"></a>Créer une stratégie pour exiger l’inscription à partir d’un emplacement approuvé

Effectuez les étapes suivantes pour créer une stratégie qui s’applique à tous les utilisateurs sélectionnés qui tentent de s’inscrire à l’aide de l’expérience d’inscription combinée et qui leur bloque l’accès, sauf s’ils se connectent à partir d’un emplacement réseau approuvé :

1. Dans le portail **Azure**, accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel**.
1. Sélectionnez **+ Nouvelle stratégie**.
1. Donnez un nom à cette stratégie, par exemple *Inscription combinée des informations de sécurité sur les réseaux approuvés*.
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**. Choisissez les utilisateurs et les groupes auxquels vous souhaitez appliquer cette stratégie, puis sélectionnez **Terminé**.

   > [!WARNING]
   > Les utilisateurs doivent avoir activé l’inscription combinée.

1. Sous **Applications ou actions cloud**, sélectionnez **Actions utilisateurs**. Cochez **Inscrire les informations de sécurité**, puis sélectionnez **Terminé**.

    ![Création d’une stratégie d’accès conditionnel personnalisée pour contrôler l’inscription des informations de sécurité](media/howto-registration-mfa-sspr-combined/require-registration-from-trusted-location.png)

1. Sous **Conditions** > **Emplacements**, configurez les options suivantes :
   1. Configurez **Oui**.
   1. Incluez **N’importe quel emplacement**.
   1. Excluez **Tous les emplacements approuvés**.
1. Sélectionnez **Terminé** dans la fenêtre *Emplacements*, puis **Terminé** dans la fenêtre *Conditions*.
1. Sous **Contrôles d’accès** > **Octroi**, choisissez **Bloquer l’accès**, puis **Sélectionner**.
1. Définissez l’option **Appliquer la stratégie** sur **Activé**.
1. Pour finaliser la stratégie, sélectionnez **Créer**.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’aide, consultez [Résolution des problèmes liés à l’inscription combinée des informations de sécurité](howto-registration-mfa-sspr-combined-troubleshoot.md) ou [Présentation de la condition d’emplacement dans l’accès conditionnel Azure AD](../conditional-access/location-condition.md).

Une fois que l'inscription combinée est activée pour les utilisateurs, vous pouvez [activer la réinitialisation de mot de passe en libre-service](tutorial-enable-sspr.md) et [activer l'authentification Azure AD MFA](tutorial-enable-azure-mfa.md).

Si nécessaire, découvrez comment [obliger les utilisateurs à réinscrire les méthodes d’authentification](howto-mfa-userdevicesettings.md#manage-user-authentication-options).
