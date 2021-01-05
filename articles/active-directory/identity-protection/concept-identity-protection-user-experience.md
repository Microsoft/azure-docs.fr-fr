---
title: Expériences utilisateur avec Azure AD Identity Protection
description: Expérience utilisateur d’Azure AD Identity Protection
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: conceptual
ms.date: 10/18/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4d4897b9187caab50be4db75bbc0af03e3d35aa4
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835983"
---
# <a name="user-experiences-with-azure-ad-identity-protection"></a>Expériences utilisateur avec Azure AD Identity Protection

Avec Azure Active Directory Identity Protection, vous pouvez :

* Exiger que les utilisateurs s'inscrivent à Azure AD Multi-Factor Authentication
* Automatiser la correction des connexions risquées et des utilisateurs compromis

Toutes les stratégies de protection des identités ont un impact sur l’expérience de connexion pour les utilisateurs. Le fait d'autoriser les utilisateurs à s'inscrire pour utiliser des outils tels qu'Azure AD MFA et la réinitialisation de mot de passe  en libre-service peut réduire l'impact. Ces outils, ainsi que les choix de stratégie appropriés, offrent aux utilisateurs une option de correction automatique lorsqu’ils en ont besoin.

## <a name="multi-factor-authentication-registration"></a>Inscription à l’authentification multifacteur

Grâce à l'activation de la stratégie Identity Protection exigeant une inscription à l'authentification multifacteur et ciblant tous vos utilisateurs, ceux-ci auront la possibilité d'utiliser Azure AD MFA pour effectuer une mise à niveau automatique à l'avenir. La configuration de cette stratégie offre à vos utilisateurs une période de 14 jours pendant laquelle ils peuvent choisir de s’inscrire et à la fin sont forcés de s’inscrire. L’expérience pour les utilisateurs est décrite ci-dessous. Pour plus d’informations, consultez la documentation de l’utilisateur final dans l’article [Vue d’ensemble de la vérification à deux facteurs et de votre compte professionnel ou scolaire](../user-help/multi-factor-authentication-end-user-first-time.md).

### <a name="registration-interrupt"></a>Interruption d’inscription

1. Lors de la connexion à une application intégrée Azure AD, l’utilisateur reçoit une notification sur la nécessité de configurer le compte pour l’authentification multifacteur. Cette stratégie est également déclenchée dans l’expérience Windows 10 prête à l’emploi pour les nouveaux utilisateurs disposant d’un nouvel appareil.
   
    ![Plus d’informations sont nécessaires](./media/concept-identity-protection-user-experience/identity-protection-experience-more-info-mfa.png)

1. Suivez les étapes guidées pour vous inscrire à Azure AD Multi-Factor Authentication et vous connecter.

## <a name="risky-sign-in-remediation"></a>Correction de connexion à risque

Lorsqu’un administrateur a configuré une stratégie pour les risques à la connexion, les utilisateurs affectés sont avertis quand ils tentent de se connecter et de déclencher le niveau de risque des stratégies. 

### <a name="risky-sign-in-self-remediation"></a>Correction automatique de connexion à risque

1. L’utilisateur est informé que quelque chose d’inhabituel a été détecté concernant sa connexion, par exemple en cas de connexion depuis un nouvel emplacement, un nouvel appareil ou une nouvelle application.
   
    ![Invite inhabituelle](./media/concept-identity-protection-user-experience/120.png)

1. L'utilisateur est tenu de prouver son identité en procédant à une authentification Azure AD MFA via l'une des méthodes auxquelles il s'est précédemment inscrit. 

### <a name="risky-sign-in-administrator-unblock"></a>Déblocage de l’administrateur de connexion risquée

Les administrateurs peuvent choisir de bloquer les utilisateurs lors de la connexion selon leur niveau de risque. Pour débloquer leur accès, les utilisateurs finaux doivent contacter leur équipe informatique ou ils peuvent essayer de se connecter depuis un emplacement ou un appareil connu. Il n’est possible de corriger automatiquement son compte en effectuant l’authentification multifacteur dans ce cas précis.

![Bloqué par une stratégie en matière de risque à la connexion](./media/concept-identity-protection-user-experience/200.png)

Le personnel informatique peut suivre les instructions de la section [Déblocage des utilisateurs](howto-identity-protection-remediate-unblock.md#unblocking-based-on-sign-in-risk) pour permettre aux utilisateurs de se reconnecter.

## <a name="risky-user-remediation"></a>Correction de l’utilisateur à risque

Lorsqu’une stratégie en matière de risque des utilisateurs a été configurée, les utilisateurs répondant à la probabilité de niveau de risque de compromission doivent passer par le flux de récupération de compte compromis avant de pouvoir se connecter. 

### <a name="risky-user-self-remediation"></a>Correction automatique de l’utilisateur à risque

1. L’utilisateur est informé que la sécurité de son compte est menacée en raison d’activités suspectes ou de la divulgation de ses informations d’identification.
   
    ![Correction](./media/concept-identity-protection-user-experience/101.png)

1. L'utilisateur est tenu de prouver son identité en procédant à une authentification Azure AD MFA via l'une des méthodes auxquelles il s'est précédemment inscrit. 
1. Enfin, l’utilisateur est obligé de changer son mot de passe par réinitialisation de mot de passe en libre-service, car il se peut que quelqu’un d’autre ait eu accès à son compte.

## <a name="risky-sign-in-administrator-unblock"></a>Déblocage de l’administrateur de connexion risquée

Les administrateurs peuvent choisir de bloquer les utilisateurs lors de la connexion selon leur niveau de risque. Pour être débloqués, les utilisateurs finaux doivent contacter leur service informatique. Dans ce cas, il n’est pas possible d’effectuer une correction automatique en effectuant l’authentification multifacteur et la réinitialisation du mot de passe en libre-service.

![Bloqué par la stratégie de risque utilisateur](./media/concept-identity-protection-user-experience/104.png)

Le personnel informatique peut suivre les instructions de la section [Déblocage des utilisateurs](howto-identity-protection-remediate-unblock.md#unblocking-based-on-user-risk) pour permettre aux utilisateurs de se reconnecter.

## <a name="see-also"></a>Voir aussi

- [Corriger les risques et débloquer les utilisateurs](howto-identity-protection-remediate-unblock.md)

- [Azure Active Directory Identity Protection](./overview-identity-protection.md)