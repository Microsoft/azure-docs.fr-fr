---
title: Contrôles personnalisés avec accès conditionnel Azure AD
description: Découvrez comment fonctionnent les contrôles personnalisés dans l’accès conditionnel Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 08/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: inbarc
ms.collection: M365-identity-device-management
ms.openlocfilehash: e7585c91e42b2d3591532756c1ead9ea60b7035e
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94837581"
---
# <a name="custom-controls-preview"></a>Contrôles personnalisés (préversion)

Les contrôles personnalisés sont en préversion dans Azure Active Directory. Quand vous utilisez les contrôles personnalisés, les utilisateurs sont redirigés vers un service compatible pour satisfaire aux exigences d’authentification en dehors d’Azure Active Directory. Pour satisfaire à ce contrôle, le navigateur de l’utilisateur est redirigé vers le service externe, il effectue les opérations d’authentification nécessaires, puis il est redirigé vers Azure Active Directory. Azure Active Directory vérifie la réponse. Si l’utilisateur a été correctement authentifié ou vérifié, il continue dans le flux d’accès conditionnel.

> [!NOTE]
> Pour plus d’informations sur les changements que nous prévoyons d’apporter à la fonctionnalité Contrôle personnalisé, consultez l’article [Archive des nouveautés](../fundamentals/whats-new-archive.md#upcoming-changes-to-custom-controls) de février 2020.

## <a name="creating-custom-controls"></a>Création de contrôles personnalisés

Les contrôles personnalisés fonctionnent avec un ensemble limité de fournisseurs d’authentification approuvés. Pour créer un contrôle personnalisé, commencez par contacter le fournisseur que vous souhaitez utiliser. Les fournisseurs autres que Microsoft ont chacun leurs propres processus et exigences que vous devez suivre pour vous inscrire, vous abonner ou devenir membre du service, et pour demander à intégrer l’accès conditionnel. Le fournisseur vous fournit alors un bloc de données au format JSON. Ces données permettent au fournisseur et à l’accès conditionnel de fonctionner ensemble pour votre locataire, créent le contrôle et définissent de quelle façon l’accès conditionnel indique si vos utilisateurs ont réussi la vérification auprès du fournisseur.

Copiez les données JSON et collez-les dans la zone de texte correspondante. Ne modifiez pas les données JSON, à moins d'être parfaitement sûr de la modification que vous apportez. Une modification inappropriée peut rompre la liaison entre le fournisseur et Microsoft, et potentiellement verrouiller l’accès à votre compte et aux comptes de vos utilisateurs.

L’option permettant de créer un contrôle personnalisé se trouve dans la section **Gérer** de la page **Accès conditionnel**.

![Interface de contrôles personnalisés dans l’accès conditionnel](./media/controls/custom-controls-conditional-access.png)

Cliquez sur **Nouveau contrôle personnalisé** pour ouvrir un panneau contenant une zone de texte réservée aux données JSON de votre contrôle.  

![Nouveau contrôle personnalisé](./media/controls/new-custom-controls-conditional-access.png)

## <a name="deleting-custom-controls"></a>Suppression de contrôles personnalisés

Pour supprimer un contrôle personnalisé, vous devez d'abord vous assurer qu'il n'est pas utilisé dans une stratégie d'accès conditionnel. Quand vous avez terminé :

1. Accédez à la liste de contrôles personnalisés
1. Cliquez sur...  
1. Sélectionnez **Supprimer**.

## <a name="editing-custom-controls"></a>Modification des contrôles personnalisés

Pour modifier un contrôle personnalisé, vous devez supprimer le contrôle existant et créer un autre contrôle avec les nouvelles informations.

## <a name="known-limitations"></a>Limitations connues

Les contrôles personnalisés ne peuvent pas être utilisés avec l’automatisation d’Identity Protection qui nécessite Azure AD Multi-Factor Authentication, la réinitialisation de mot de passe en libre-service (SSPR) Azure AD, le respect des exigences de revendication de l’authentification multifacteur, pour l’élévation de rôles dans Privileged Identity Manager (PIM), dans le cadre de l’inscription d’appareils Intune ou lors de la jonction d’appareils à Azure AD.

## <a name="next-steps"></a>Étapes suivantes

- [Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)

- [Mode Rapport uniquement](concept-conditional-access-report-only.md)

- [Simuler le comportement de connexion à l’aide de l’outil What If pour l’accès conditionnel](troubleshoot-conditional-access-what-if.md)
