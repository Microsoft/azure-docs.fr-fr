---
title: Accès conditionnel basé sur les risques d’utilisateur – Azure Active Directory
description: Créer des stratégies d’accès conditionnel à l’aide du risque d’utilisateur Identity Protection
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 07/02/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: d5423a61c8febee72f32935f3dee4e9f9e868630
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89049075"
---
# <a name="conditional-access-user-risk-based-conditional-access"></a>Accès conditionnel : Accès conditionnel basé sur les risques d’utilisateur

Microsoft travaille avec des chercheurs, les forces de l’ordre, les différentes équipes de sécurité de Microsoft et d’autres sources approuvées pour rechercher les paires nom d’utilisateur/mot de passe divulguées. Les organisations disposant des licences Azure AD Premium P2 peuvent créer des stratégies d’accès conditionnel incorporant des [détections de risques des utilisateurs Azure AD Identity Protection](../identity-protection/concept-identity-protection-risks.md#user-risk). 

Il existe deux emplacements où cette stratégie peut être affectée. Les organisations doivent choisir l’une des options suivantes pour activer une stratégie d’accès conditionnel basé sur les risques utilisateur nécessitant une modification sécurisée du mot de passe.

## <a name="enable-with-conditional-access-policy"></a>Activer avec la stratégie d’accès conditionnel

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies. Pour plus d’informations, consultez [Définir des normes d’attribution de noms pour vos stratégies](./plan-conditional-access.md#set-naming-standards-for-your-policies).
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**.
   1. Sous **Inclure**, sélectionnez **Tous les utilisateurs**.
   1. Sous **Exclure**, sélectionnez **Utilisateurs et groupes**, puis choisissez les comptes d’accès d’urgence ou de secours de votre organisation. 
   1. Sélectionnez **Terminé**.
1. Sous **Applications ou actions cloud** > **Inclure**, sélectionnez **Toutes les applications cloud**.
1. Dans **Conditions** > **des risques utilisateur**, définissez **Configurer** sur **Oui**. Dans **Configurer les niveaux de risque des utilisateurs nécessaires à l’application de la stratégie**, sélectionnez **Élevé**, puis **Terminé**.
1. Dans **Accorder** **les contrôles d’accès** > , sélectionnez **Accorder l’accès**, **Demander la modification du mot de passe**, puis **Sélectionner**.
1. Confirmez vos paramètres et définissez **Activer la stratégie** sur **Activé**.
1. Sélectionnez **Créer** pour créer votre stratégie.

## <a name="enable-through-identity-protection"></a>Activer via Identity Protection

1. Connectez-vous au **portail Azure**.
1. Sélectionnez **Tous les services**, puis accédez à **Azure AD Identity Protection**.
1. Sélectionnez **Stratégie de risque utilisateur**.
1. Sous **Attributions**, sélectionnez **Utilisateurs**.
   1. Sous **Inclure**, sélectionnez **Tous les utilisateurs**.
   1. Sous **Exclure**, sélectionnez **Sélectionner les utilisateurs exclus**, choisissez les comptes d’accès d’urgence ou de secours de votre organisation, puis sélectionnez **Sélectionner**.
   1. Sélectionnez **Terminé**.
1. Sous **Conditions**, sélectionnez **Risque de l’utilisateur**, puis choisissez **Élevé**.
   1. Sélectionnez **Sélectionner**, puis **Terminé**.
1. Sous **Contrôles** > **Accès**, choisissez **Autoriser l’accès**, puis sélectionnez **Nécessite une modification du mot de passe**.
   1. Sélectionnez **Sélectionner**.
1. Définissez **Appliquer la stratégie** sur **Activé**.
1. Sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

[Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)

[Connexion à l’accès conditionnel basé sur les risques](howto-conditional-access-policy-risk.md)

[Déterminer l'impact à l'aide du mode Rapport seul de l'Accès conditionnel](howto-conditional-access-insights-reporting.md)

[Simuler le comportement de connexion à l’aide de l’outil What If pour l’accès conditionnel](troubleshoot-conditional-access-what-if.md)

[Qu’est-ce que Azure Active Directory Identity Protection ?](../identity-protection/overview-identity-protection.md)