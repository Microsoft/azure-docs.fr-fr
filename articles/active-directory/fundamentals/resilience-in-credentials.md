---
title: Renforcer la résilience grâce à la gestion des informations d’identification dans Azure Active Directory
description: Guide destiné aux architectes et aux administrateurs informatiques sur la création d’une stratégie résiliente d’informations d’identification.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 11/30/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: b5fd5cf419adb137df5c578e3b17b88749215be5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95918998"
---
# <a name="build-resilience-with-credential-management"></a>Renforcer la résilience grâce à la gestion des informations d’identification

Lorsqu’une information d’identification est présentée à Azure AD dans une demande de jeton, plusieurs dépendances doivent être disponibles pour la validation. Le premier facteur d’authentification repose sur l’authentification Azure AD et, dans certains cas, sur l’infrastructure locale. Pour plus d’informations sur les architectures d’authentification hybride, consultez [Renforcer la résilience de votre infrastructure hybride](resilience-in-hybrid.md). 

Si vous implémentez un second facteur, les dépendances du second facteur sont ajoutées aux dépendances de la première. Par exemple, si votre premier facteur s’effectue via l’authentification directe (PTA) et que votre second facteur est par SMS, vos dépendances sont les suivantes :

* Services d’authentification Azure AD

* Service Azure MFA

* Infrastructure locale

* Opérateur téléphonique

* Appareil de l’utilisateur (non représenté)

 
![Image des méthodes d’authentification et des dépendances](./media/resilience-in-credentials/admin-resilience-credentials.png)

Votre stratégie d’informations d’identification doit prendre en compte les dépendances de chaque type d’authentification et approvisionner des méthodes qui évitent un point de défaillance unique. 

Comme les méthodes d’authentification ont des dépendances différentes, il est judicieux d’autoriser les utilisateurs à s’inscrire à autant d’options de deuxième facteur que possible. Veillez à inclure des seconds facteurs avec des dépendances différentes dans la mesure du possible. Par exemple, les appels vocaux et les SMS en tant que seconds facteurs partagent les mêmes dépendances, de sorte que les avoir comme seules options ne réduit pas le risque.

La stratégie d’informations d’identification la plus résiliente consiste à utiliser l’authentification par mot de passe. Windows Hello Entreprise et les clés de sécurité FIDO 2.0 ont moins de dépendances que l’authentification forte avec deux facteurs distincts. L’application Microsoft Authenticator, Windows Hello Entreprise et les clés de sécurité FIDO 2.0 sont les plus sécurisées. 

En ce qui concerne les seconds facteurs, l’application Microsoft Authenticator ou d’autres applications d’authentification utilisant un mot de passe à usage unique et à durée définie (TOTP) ou des jetons matériels OATH ont le moins de dépendances, et sont donc plus résilientes.

## <a name="how-do-multiple-credentials-help-resilience"></a>Comment des informations d’identification multiples contribuent-elles à la résilience ?

L’approvisionnement de plusieurs types d’informations d’identification offre aux utilisateurs des options qui prennent en compte leurs préférences et leurs contraintes environnementales. Par conséquent, l’authentification interactive où les utilisateurs sont invités à l’authentification multifacteur est plus résistante aux dépendances spécifiques qui ne seront pas disponibles au moment de la demande. Vous pouvez [optimiser les invites de réauthentification pour l’authentification multifacteur](../authentication/concepts-azure-multi-factor-authentication-prompts-session-lifetime.md).

En plus de la résilience des utilisateurs individuels décrite ci-dessus, les entreprises doivent prévoir des mesures d’urgence en cas de perturbations à grande échelle, telles que les erreurs opérationnelles qui introduisent une erreur de configuration, une catastrophe naturelle ou une panne de ressources à l’échelle de l’entreprise pour un service de fédération local (surtout lorsqu’il est utilisé pour l’authentification multifacteur). 

## <a name="how-do-i-implement-resilient-credentials"></a>Comment implémenter des informations d’identification résilientes ?

* Déployez des [informations d’identification sans mot de passe](../authentication/howto-authentication-passwordless-deployment.md) telles que Windows Hello Entreprise, l’authentification par téléphone et les clés de sécurité FIDO2 pour réduire les dépendances.

* Déployez l’[application Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) en tant que second facteur.

* Activez [la synchronisation de hachage de mot de passe](../hybrid/whatis-phs.md) pour les comptes hybrides qui sont synchronisés à partir de Windows Server Active Directory. Cette option peut être activée en même temps que les services de fédération tels que AD FS et fournit une solution de repli en cas d’échec du service de fédération.

* [Analysez l’utilisation des méthodes d’authentification multifacteur](https://docs.microsoft.com/samples/azure-samples/azure-mfa-authentication-method-analysis/azure-mfa-authentication-method-analysis/) pour améliorer l’expérience des utilisateurs.

* [Implémentez une stratégie résiliente de contrôle d’accès](../authentication/concept-resilient-controls.md).

## <a name="next-steps"></a>Étapes suivantes
Ressources de résilience pour les administrateurs et les architectes
 
* [Renforcer la résilience grâce aux états des appareils](resilience-with-device-states.md)

* [Renforcer la résilience en utilisant Évaluation continue de l’accès (CAE)](resilience-with-continuous-access-evaluation.md)

* [Renforcer la résilience de l’authentification des utilisateurs externes](resilience-b2b-authentication.md)

* [Renforcer la résilience de votre authentification hybride](resilience-in-hybrid.md)

* [Renforcer la résilience de l’accès aux applications avec Proxy d’application](resilience-on-premises-access.md)

Ressources de résilience pour les développeurs

* [Renforcer la résilience de l’IAM dans vos applications](resilience-app-development-overview.md)

* [Renforcer la résilience de vos systèmes CIAM](resilience-b2c.md)
