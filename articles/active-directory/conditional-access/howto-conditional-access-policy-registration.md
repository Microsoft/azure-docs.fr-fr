---
title: Accès conditionnel - Informations de sécurité combinées - Azure Active Directory
description: Créer une stratégie d’accès conditionnel personnalisée pour l’inscription des informations de sécurité
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 81f4407ee7721332a4143952d1720151bb70d8c9
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94837536"
---
# <a name="conditional-access-securing-security-info-registration"></a>Accès conditionnel : Sécurisation de l’inscription des informations de sécurité

La sécurisation de l’inscription des utilisateurs à Azure AD Multi-Factor Authentication et à la réinitialisation de mot de passe en libre-service est désormais possible avec les actions de l’utilisateur dans la stratégie d’accès conditionnel. Cette fonctionnalité en préversion est à la disposition des organisations qui ont activé l’[inscription combinée](../authentication/concept-registration-mfa-sspr-combined.md). Cette fonctionnalité peut être activée dans les organisations qui souhaitent utiliser des conditions telles qu’un emplacement réseau approuvé afin de limiter l’accès pour s’inscrire à Azure AD Multi-Factor Authentication et à la réinitialisation de mot de passe en libre-service (SSPR). Pour plus d’informations sur les conditions utilisables, consultez l’article [Accès conditionnel : Conditions](concept-conditional-access-conditions.md).

## <a name="create-a-policy-to-require-registration-from-a-trusted-location"></a>Créer une stratégie pour exiger l’inscription à partir d’un emplacement approuvé

La stratégie suivante s’applique à tous les utilisateurs sélectionnés, qui tentent de s’inscrire à l’aide de l’expérience d’inscription combinée et bloque l’accès, sauf si ces derniers se connectent à partir d’un emplacement réseau approuvé.

1. Dans le portail **Azure**, accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel**.
1. Sélectionnez **Nouvelle stratégie**.
1. Sous Nom, entrez un nom pour cette stratégie. Par exemple, **Inscription d’informations de sécurité combinée sur les réseaux approuvés**.
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**, puis sélectionnez les utilisateurs et les groupes auxquels vous souhaitez appliquer cette stratégie.

   > [!WARNING]
   > Les utilisateurs doivent être activés pour l’[inscription combinée](../authentication/howto-registration-mfa-sspr-combined.md).

1. Sous **Applications cloud ou actions**, sélectionnez **Actions utilisateur**, cochez **Inscrire des informations de sécurité**.
1. Sous **Conditions** > **Emplacements**.
   1. Configurez **Oui**.
   1. Incluez **N’importe quel emplacement**.
   1. Excluez **Tous les emplacements approuvés**.
   1. Sélectionnez **Terminé** dans le panneau Emplacements.
   1. Sélectionnez **Terminé** dans le panneau des conditions.
1. Sous **Conditions** > **Applications clientes (préversion)** , définissez **Configurer** sur **Oui**, puis sélectionnez **Terminé**.
1. Sous **Contrôles d’accès** > **Octroyer**.
   1. Sélectionnez **Bloquer l’accès**.
   1. Puis cliquez sur **Sélectionner**.
1. Définissez l’option **Appliquer la stratégie** sur **Activé**.
1. Ensuite, sélectionnez **Enregistrer**.

À l’étape 6 de cette stratégie, les organisations ont la possibilité d’effectuer des choix. La stratégie ci-dessus requiert l’inscription à partir d’un emplacement réseau approuvé. Les organisations peuvent choisir d’utiliser les conditions disponibles à la place des **Emplacements**. N’oubliez pas que cette stratégie est une stratégie de blocage, de sorte que tout ce qui est inclus est bloqué et tout ce qui ne correspond pas à l’inclusion est autorisé. 

Certains peuvent choisir d’utiliser l’état de l’appareil au lieu de l’emplacement à l’étape 6 ci-dessus :

6. Sous **Conditions** > **État de l'appareil (préversion)** .
   1. Configurez **Oui**.
   1. Incluez **Tous les états d'appareils**.
   1. Excluez **Appareil joint à une version Azure AD Hybride** et/ou **Appareil marqué comme conforme**.
   1. Sélectionnez **Terminé** dans le panneau Emplacements.
   1. Sélectionnez **Terminé** dans le panneau des conditions.

> [!WARNING]
> Si vous utilisez l’état de l’appareil comme condition dans votre stratégie, cela peut avoir un impact sur les utilisateurs invités dans l’annuaire. Le [Mode Rapport uniquement](concept-conditional-access-report-only.md) permet de déterminer l’impact des décisions stratégiques.
> Notez que le mode Rapport uniquement n’est pas applicable pour les stratégies d’autorité de certification avec l’étendue « Actions de l’utilisateur ».

## <a name="next-steps"></a>Étapes suivantes

[Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)

[Déterminer l'impact à l'aide du mode Rapport seul de l'Accès conditionnel](howto-conditional-access-insights-reporting.md)

[Simuler le comportement de connexion à l’aide de l’outil What If pour l’accès conditionnel](troubleshoot-conditional-access-what-if.md)
