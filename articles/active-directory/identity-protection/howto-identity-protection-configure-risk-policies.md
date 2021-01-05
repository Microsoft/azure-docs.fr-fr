---
title: Stratégies de risque – Azure Active Directory Identity Protection
description: Activer et configurer les stratégies en matière de risque dans Azure Active Directory Identity Protection
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 06/05/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 366d68be1a7f115980973015e363da6095876754
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997628"
---
# <a name="how-to-configure-and-enable-risk-policies"></a>Procédure : Configurer et activer des stratégies de risque

Comme nous l’avons appris dans l’article précédent, [Stratégies de protection des identités](concept-identity-protection-policies.md), nous avons deux stratégies de risque que nous pouvons activer dans notre annuaire. 

- Stratégie en matière de risque à la connexion
- Stratégie de risque d’utilisateur

![Page de vue d’ensemble de la sécurité permettant d’activer les stratégies de risque utilisateur et de connexion](./media/howto-identity-protection-configure-risk-policies/identity-protection-security-overview.png)

Ensemble, les deux stratégies automatisent la réponse aux détections de risques dans votre environnement et permettent aux utilisateurs de résoudre eux-mêmes des problèmes quand des risques sont détectés. 

> [!VIDEO https://www.youtube.com/embed/zEsbbik-BTE]

## <a name="prerequisites"></a>Prérequis 

Si votre organisation souhaite autoriser les utilisateurs à résoudre eux-mêmes les problèmes quand des risques sont détectés, les utilisateurs doivent être inscrits à la fois pour la réinitialisation du mot de passe en libre-service et Azure AD Multi-Factor Authentication. Nous vous recommandons [l’activation de l’expérience d’inscription d’informations de sécurité combinée](../authentication/howto-registration-mfa-sspr-combined.md) pour une expérience optimale. Le fait de permettre aux utilisateurs de résoudre eux-mêmes les problèmes les ramène à un état productif plus rapidement, sans nécessiter l’intervention de l’administrateur. Les administrateurs peuvent toujours voir ces événements et les examiner après coup. 

## <a name="choosing-acceptable-risk-levels"></a>Choix des niveaux de risque acceptables

Les organisations doivent déterminer le niveau de risque consenti lorsqu’elles mettent dans la balance l’expérience utilisateur et la posture de sécurité. 

La recommandation de Microsoft est de définir le seuil de stratégie de risque de l’utilisateur sur **Élevé** et la stratégie de risque de connexion sur **Moyen et supérieur**.

La sélection d’un niveau de risque **Élevé** réduit la fréquence de déclenchement d’une stratégie et minimise l’impact sur les utilisateurs. Cependant, cela a pour effet d’exclure les détections de risque **Faible** et **Moyen**. Par conséquent, il se peut qu’un cybercriminel soit en mesure d’exploiter une identité compromise. La sélection d’une seuil **Faible** introduit des interruptions utilisateur supplémentaires, mais renforce la sécurité.

## <a name="exclusions"></a>Exclusions

Toutes les stratégies permettent d’exclure des utilisateurs, tels que vos [comptes d’administrateur d’accès en urgence ou d’interruption](../roles/security-emergency-access.md). Les organisations peuvent déterminer qu’elles doivent exclure les autres comptes de stratégies spécifiques en fonction de la façon dont les comptes sont utilisés. Toutes les exclusions doivent être examinées régulièrement pour déterminer si elles sont toujours applicables.

Les [emplacements réseau](../conditional-access/location-condition.md) approuvés qui ont été configurés sont utilisés par Identity Protection dans certaines détections de risques afin de réduire les faux positifs.

## <a name="enable-policies"></a>Activer les stratégies

Pour activer les stratégies de risque utilisateur et de risque de connexion, procédez comme suit.

1. Accédez au [portail Azure](https://portal.azure.com).
1. Accédez à **Azure Active Directory** > **Sécurité** > **Identity Protection** > **Vue d’ensemble**.
1. Sélectionnez **Stratégie de risque utilisateur**.
   1. Sous **Affectations**
      1. **Utilisateurs** : choisissez **Tous les utilisateurs** ou **Sélectionner des personnes et des groupes** si vous limitez votre lancement.
         1. Si vous le souhaitez, vous pouvez exclure des utilisateurs de la stratégie.
      1. **Conditions** - **Risque utilisateur** : la recommandation de Microsoft consiste à définir cette option sur **Élevé**.
   1. Sous **Contrôles**
      1. **Accès** : la recommandation de Microsoft consiste à **Autoriser l’accès** et à **Exiger la modification du mot de passe**.
   1. **Appliquer la stratégie** - **Activé**
   1. **Enregistrer** : cette action a pour effet de renvoyer à la page **Vue d’ensemble**.
1. Sélectionnez **Stratégie de connexion à risque**.
   1. Sous **Affectations**
      1. **Utilisateurs** : choisissez **Tous les utilisateurs** ou **Sélectionner des personnes et des groupes** si vous limitez votre lancement.
         1. Si vous le souhaitez, vous pouvez exclure des utilisateurs de la stratégie.
      1. **Conditions** - **Risque connexion** : la recommandation de Microsoft consiste à définir cette option sur **Moyen ou plus**.
   1. Sous **Contrôles**
      1. **Accès** : la recommandation de Microsoft consiste à **Autoriser l’accès** et à **Exiger l’authentification multifacteur**.
   1. **Appliquer la stratégie** - **Activé**
   1. **Save**

## <a name="next-steps"></a>Étapes suivantes

- [Activer la stratégie d'inscription Azure AD Multi-Factor Authentication](howto-identity-protection-configure-mfa-policy.md)

- [Qu'est-ce qu'un risque ?](concept-identity-protection-risks.md)

- [Examiner les détections de risques](howto-identity-protection-investigate-risk.md)

- [Simuler des détections de risques](howto-identity-protection-simulate-risk.md)
