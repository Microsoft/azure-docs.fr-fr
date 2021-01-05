---
title: Atténuer les risques et débloquer des utilisateurs dans Azure AD Identity Protection
description: Découvrez les possibilités dont vous disposez pour fermer les détections de risques actives.
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 10/06/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6dd918aef85deefc23771413c3eb7b92f1189d39
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835796"
---
# <a name="remediate-risks-and-unblock-users"></a>Atténuer les risques et débloquer les utilisateurs

Une fois votre [investigation](howto-identity-protection-investigate-risk.md) terminée, vous pouvez prendre des mesures pour atténuer le risque ou débloquer les utilisateurs. Les organisations ont également la possibilité d’activer une correction automatisée en implémentant des [stratégies de gestion des risques](howto-identity-protection-configure-risk-policies.md). Les organisations doivent tenter de traiter toutes les détections de risque auxquelles elles sont confrontées dans un laps de temps qui leur convient. Microsoft recommande de clore les événements dès que possible, car le temps compte en matière de risques.

## <a name="remediation"></a>Correction

Toutes les détections de risque actifs sont prises en compte dans le calcul d’une valeur appelée niveau de risque utilisateur. Le niveau de risque utilisateur est un indicateur (faible, moyen, élevé) de la probabilité qu’un compte ait été compromis. En tant qu’administrateur, vous souhaitez fermer toutes les détections de risques afin que les utilisateurs affectés ne soient plus exposés.

Identity Protection peut marquer certaines détections de risques comme « Fermé (système) » quand les événements ne sont plus jugés à risque.

Pour corriger, les administrateurs disposent des options suivantes :

- Correction automatique avec une stratégie de gestion des risques
- Réinitialisation manuelle du mot de passe
- Ignorer le risque lié à l’utilisateur
- Fermeture manuelle de détections de risques spécifiques

### <a name="self-remediation-with-risk-policy"></a>Correction automatique avec une stratégie de gestion des risques

Si vous autorisez des utilisateurs à effectuer une correction automatique, avec Azure AD Multi-Factor Authentication (MFA) et la réinitialisation de mot de passe en libre-service (SSPR) dans vos stratégies de gestion des risques, ils peuvent se débloquer lorsqu'un risque est détecté. Ces détections sont alors considérées comme fermées. Les utilisateurs doivent s'être préalablement inscrits à Azure MFA et SSPR afin de pouvoir les utiliser lorsqu'un risque est détecté.

Si certaines détections peuvent ne pas augmenter de risque jusqu’au niveau où une correction automatique de l’utilisateur serait requise, les administrateurs doivent toujours évaluer ces détections. Les administrateurs peuvent juger que des mesures supplémentaires sont nécessaires, comme [bloquer l’accès à partir de certains emplacements](../conditional-access/howto-conditional-access-policy-location.md) ou abaisser le niveau de risque acceptable dans leurs stratégies.

### <a name="manual-password-reset"></a>Réinitialisation manuelle du mot de passe

Si exiger une réinitialisation du mot de passe à l’aide d’une stratégie de risque utilisateur n’est pas envisageable, les administrateurs peuvent fermer toutes les détections de risques pour un utilisateur en opérant une réinitialisation manuelle du mot de passe.

Pour réinitialiser un mot de passe pour leurs utilisateurs, les administrateurs disposent de deux options :

- **Générer un mot de passe temporaire** : la génération d’un mot de passe temporaire vous permet de rétablir immédiatement la sécurité d’une identité. Cette méthode nécessite de contacter les utilisateurs affectés, car ceux-ci ont besoin de connaître le mot de passe temporaire. Étant donné que le mot de passe est temporaire, l’utilisateur est invité à le modifier lors de sa prochaine connexion.

- **Obliger l’utilisateur à réinitialiser le mot de passe** : le fait de contraindre les utilisateurs à réinitialiser les mots de passe entraîne une récupération automatique qui ne nécessite aucun contact avec le support technique ou un administrateur. Cette méthode s'applique uniquement aux utilisateurs inscrits à Azure AD MFA et SSPR. Pour les utilisateurs non inscrits, cette option n’est pas disponible.

### <a name="dismiss-user-risk"></a>Ignorer le risque lié à l’utilisateur

Si une réinitialisation de mot de passe n’est pas envisageable pour vous, par exemple, parce que l’utilisateur a été supprimé, vous pouvez choisir d’ignorer les détections de risque utilisateur.

Lorsque vous cliquez sur **Ignorer le risque lié à l’utilisateur**, tous les événements sont fermés et l’utilisateur concerné n’est plus à risque. Toutefois, étant donné que cette méthode n’a pas d’incidence sur le mot de passe existant, elle ne rétablit pas la sécurité de l’identité associée. 

### <a name="close-individual-risk-detections-manually"></a>Fermeture manuelle de détections de risques spécifiques

Vous pouvez fermer manuellement des détections de risques spécifiques. La fermeture manuelle de détections de risques vous permet de réduire le niveau de risque utilisateur. En règle générale, les détections de risques sont fermées manuellement en réponse à une investigation associée, par exemple, lorsqu’une discussion avec un utilisateur révèle qu’une détection de risque active n’est plus requise. 
 
Lorsque vous fermez des détections de risques manuellement, vous pouvez choisir d’exécuter l’une des actions ci-après pour modifier l’état d’une détection de risque :

- Confirmer que l’utilisateur est compromis
- Ignorer le risque lié à l’utilisateur
- Confirmer que la connexion est sécurisée
- Confirmer que la connexion est compromise

## <a name="unblocking-users"></a>Déblocage des utilisateurs

Un administrateur peut choisir de bloquer une connexion en fonction de sa stratégie de gestion de risque ou d’investigations. Un blocage peut se produire en fonction d’une connexion ou d’un risque utilisateur.

### <a name="unblocking-based-on-user-risk"></a>Déblocage basé sur le risque utilisateur

Pour débloquer un compte bloqué en raison d’un risque utilisateur, les administrateurs disposent des options suivantes :

1. **Réinitialiser le mot de passe réinitialisé** : vous pouvez réinitialiser le mot de passe de l’utilisateur.
1. **Ignorer le risque lié à l’utilisateur** : la stratégie de gestion du risque utilisateur bloque un utilisateur si le niveau de risque utilisateur configuré pour bloquer l’accès a été atteint. Vous pouvez réduire le niveau de risque d’un utilisateur en ignorant le risque utilisateur ou en fermant manuellement des détections de risques signalées.
1. **Exclure l’utilisateur de la stratégie** : si vous pensez que la configuration actuelle de votre stratégie d’authentification occasionne des problèmes pour certains utilisateurs, vous pouvez les en exclure. Pour plus d’informations, consultez la section Exclusions de l’article [Guide pratique : Configurer et activer des stratégies de risque](howto-identity-protection-configure-risk-policies.md#exclusions).
1. **Désactiver la stratégie** : si vous pensez que votre configuration de la stratégie est à l’origine des problèmes pour tous vos utilisateurs, vous pouvez désactiver la stratégie. Pour plus d’informations, consultez l’article [Guide pratique : Configurer et activer des stratégies de risque](howto-identity-protection-configure-risk-policies.md).

### <a name="unblocking-based-on-sign-in-risk"></a>Déblocage basé sur le risque de connexion

Pour débloquer un compte en fonction du risque de connexion, les administrateurs disposent des options suivantes :

1. **Connexion à partir d’un emplacement ou d’un appareil connu** : les connexions suspectes bloquées sont généralement des tentatives de connexion effectuées à partir d’un emplacement ou d’un appareil inconnu. Vos utilisateurs peuvent déterminer rapidement s’il s’agit bien de la raison du blocage en essayant de se connecter à partir d’un appareil ou d’un emplacement connu.
1. **Exclure l’utilisateur de la stratégie** : si vous pensez que la configuration actuelle de votre stratégie d’authentification occasionne des problèmes pour certains utilisateurs, vous pouvez les en exclure. Pour plus d’informations, consultez la section Exclusions de l’article [Guide pratique : Configurer et activer des stratégies de risque](howto-identity-protection-configure-risk-policies.md#exclusions).
1. **Désactiver la stratégie** : si vous pensez que votre configuration de la stratégie est à l’origine des problèmes pour tous vos utilisateurs, vous pouvez désactiver la stratégie. Pour plus d’informations, consultez l’article [Guide pratique : Configurer et activer des stratégies de risque](howto-identity-protection-configure-risk-policies.md).

## <a name="powershell-preview"></a>PowerShell en préversion

Le module Microsoft Graph PowerShell SDK Preview permet aux organisations de gérer les risques à l'aide de PowerShell. Les modules disponibles en préversion et les exemples de code se trouvent dans le [référentiel GitHub Azure AD](https://github.com/AzureAD/IdentityProtectionTools).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir une vue d’ensemble du service Azure AD Identity Protection, consultez l’article de [présentation d’Azure AD Identity Protection](overview-identity-protection.md).
