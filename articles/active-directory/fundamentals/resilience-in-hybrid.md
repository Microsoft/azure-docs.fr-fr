---
title: Créer une authentification hybride plus résiliente dans Azure Active Directory
description: Guide destiné aux architectes et aux administrateurs informatiques sur la création d’une infrastructure hybride résiliente.
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
ms.openlocfilehash: 5c45b362bc37df71346fc3b635c8ae4a51f62cdc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95918987"
---
# <a name="build-resilience-in-your-hybrid-architecture"></a>Renforcer la résilience de votre architecture hybride

L’authentification hybride permet aux utilisateurs d’accéder aux ressources informatiques à l’aide de leurs identités locales. Une infrastructure hybride comprend des composants cloud et locaux.

* Les composants cloud incluent Azure AD, les ressources et services Azure, les applications informatiques de votre organisation et les applications SaaS.

* Les composants locaux incluent des applications locales, des ressources telles que des bases de données SQL et un fournisseur d’identité tel que Windows Server Active Directory. 

> [!IMPORTANT]
> Lorsque vous planifiez la résilience de votre infrastructure hybride, il est essentiel de réduire les dépendances et les points de défaillance uniques. 

Microsoft propose trois mécanismes d’authentification hybride. Les options sont répertoriées par ordre de résilience. Nous vous recommandons d’implémenter si possible la synchronisation de hachage de mot de passe.

* La [synchronisation de hachage de mot de passe](../hybrid/whatis-phs.md) (PHS) utilise Azure AD Connect pour synchroniser l’identité et le hachage du mot de passe à Azure AD, ce qui permet aux utilisateurs de se connecter aux ressources informatiques avec leur mot de passe local. La PHS présente des dépendances locales uniquement pour la synchronisation, et non pour l’authentification.

* L’[authentification directe](../hybrid/how-to-connect-pta.md) (PTA) redirige les utilisateurs vers Azure AD pour la connexion. Ensuite, le nom d’utilisateur et le mot de passe sont validés auprès d’Active Directory localement, par le biais d’un agent déployé dans le réseau d’entreprise. La PTA a une empreinte locale de ses agents PTA Azure AD qui résident sur des serveurs locaux.

* Les clients d’une [fédération](../hybrid/whatis-fed.md) déploient un service de fédération tel que AD FS, puis Azure AD valide l’assertion SAML produite par le service de fédération. La fédération présente la dépendance la plus élevée sur l’infrastructure locale et, par conséquent, davantage de points de défaillance. 

   
‎Vous utilisez peut-être une ou plusieurs de ces méthodes dans votre organisation. Pour plus d’informations, consultez [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure AD](../hybrid/choose-ad-authn.md). Cet article contient un arbre de décision qui peut vous aider à choisir votre méthodologie.

## <a name="password-hash-synchronization"></a>Synchronisation de hachage du mot de passe

L’option d’authentification hybride la plus simple et la plus résistante pour Azure AD est la [synchronisation de hachage de mot de passe](../hybrid/whatis-phs.md), qui ne dépend pas de l’infrastructure d’identité locale lors du traitement des demandes d’authentification. Une fois que les identités avec hachage de mot de passe sont synchronisées avec Azure AD, les utilisateurs peuvent s’authentifier sur des ressources cloud sans aucune dépendance vis-à-vis des composants d’identité locaux. 

![Diagramme d’architecture de PHS](./media/resilience-in-hybrid/admin-resilience-password-hash-sync.png)

Si vous choisissez cette option d’authentification, vous ne subirez pas de perturbations lorsque les composants d’identité locaux ne seront plus disponibles. Une interruption locale peut se produire pour de nombreuses raisons, notamment une défaillance matérielle, des pannes de courant, des catastrophes naturelles et des attaques de programmes malveillants. 

### <a name="how-do-i-implement-phs"></a>Comment implémenter la PHS ?

Pour implémenter la PHS consultez les ressources suivantes :

* [Implémenter la synchronisation de hachage de mot de passe à l’aide d’Azure AD Connect](../hybrid/how-to-connect-password-hash-synchronization.md)

* [Activer la synchronisation de hachage de mot de passe](../hybrid/how-to-connect-password-hash-synchronization.md)

Si vos exigences sont telles que vous ne pouvez pas utiliser la PHS, utilisez l’authentification directe.

## <a name="pass-through-authentication"></a>Authentification directe

L’authentification directe a une dépendance vis-à-vis des agents d’authentification résidant en local sur les serveurs. Une connexion persistante (ou bus de service) est présente entre Azure AD et les agents PTA locaux. Le pare-feu, les serveurs hébergeant les agents d’authentification et Windows Server Active Directory (ou un autre fournisseur d’identité) sont tous des points de défaillance potentiels. 

![Diagramme d’architecture de PTA](./media/resilience-in-hybrid/admin-resilience-pass-through-authentication.png)

### <a name="how-do-i-implement-pta"></a>Comment implémenter la PTA ?

Pour implémenter l’authentification directe, consultez les ressources suivantes.

* [Fonctionnement de l’authentification directe](../hybrid/how-to-connect-pta-how-it-works.md)

* [Présentation approfondie de la sécurité de l’authentification directe](../hybrid/how-to-connect-pta-security-deep-dive.md)

* [Installer l’authentification directe Azure AD](../hybrid/how-to-connect-pta-quick-start.md)

* Si vous utilisez la PTA, définissez une [topologie à haute disponibilité](../hybrid/how-to-connect-pta-quick-start.md).

 ## <a name="federation"></a>Fédération

La fédération implique la création d’une relation de confiance entre Azure AD et le service de fédération, qui comprend l’échange de points de terminaison, de certificats de signature de jetons et d’autres métadonnées. Lorsqu’une demande est envoyée à Azure AD, elle lit la configuration et redirige l’utilisateur vers les points de terminaison configurés. À ce stade, l’utilisateur interagit avec le service de fédération, qui émet une assertion SAML validée par Azure AD. 

Le diagramme suivant illustre la topologie de services de fédération Active Directory (AD FS) d’entreprise, déploiement qui comprend des serveurs de fédération et de proxy d’application web redondants sur plusieurs centres de données locaux. Cette configuration s’appuie sur des composants de l’infrastructure réseau de l’entreprise tels que le DNS, l’équilibrage de la charge réseau avec des capacités d’affinité géographique, des pare-feu, etc. Tous les composants et connexions locaux sont susceptibles de tomber en panne. Pour plus d’informations, consultez la [documentation sur la planification de la capacité AD FS](https://docs.microsoft.com/windows-server/identity/ad-fs/design/planning-for-ad-fs-server-capacity).

> [!NOTE]
>  La fédération présente le plus grand nombre de dépendances locales et, par conséquent, le plus grand nombre de points de défaillance potentiels. Bien que ce diagramme montre AD FS, d’autres fournisseurs d’identité locaux sont soumis à des considérations de conception similaires pour obtenir une haute disponibilité, une scalabilité et un basculement.

![Diagramme d’architecture de la fédération](./media/resilience-in-hybrid/admin-resilience-federation.png)

 ### <a name="how-do-i-implement-federation"></a>Comment implémenter la fédération ?

Si vous implémentez une stratégie d’authentification fédérée ou si vous souhaitez la rendre plus résiliente, consultez les ressources suivantes.

* [Présentation de l’authentification fédérée](../hybrid/whatis-fed.md)

* [Fonctionnement de la fédération](../hybrid/how-to-connect-fed-whatis.md)

* [Liste de compatibilité de fédération Azure AD](../hybrid/how-to-connect-fed-compatibility.md)

* Suivez la [documentation sur la planification de la capacité AD FS](https://docs.microsoft.com/windows-server/identity/ad-fs/design/planning-for-ad-fs-server-capacity)

* [Déploiement d’AD FS dans Azure IaaS](https://docs.microsoft.com/windows-server/identity/ad-fs/deployment/how-to-connect-fed-azure-adfs)

* [Activer la PHS](../hybrid/tutorial-phs-backup.md) avec votre fédération

## <a name="next-steps"></a>Étapes suivantes
Ressources de résilience pour les administrateurs et les architectes
 
* [Renforcer la résilience grâce à la gestion des informations d’identification](resilience-in-credentials.md)

* [Renforcer la résilience grâce aux états des appareils](resilience-with-device-states.md)

* [Renforcer la résilience en utilisant Évaluation continue de l’accès (CAE)](resilience-with-continuous-access-evaluation.md)

* [Renforcer la résilience de l’authentification des utilisateurs externes](resilience-b2b-authentication.md)

* [Renforcer la résilience de l’accès aux applications avec Proxy d’application](resilience-on-premises-access.md)

Ressources de résilience pour les développeurs

* [Renforcer la résilience de l’IAM dans vos applications](resilience-app-development-overview.md)

* [Renforcer la résilience de vos systèmes CIAM](resilience-b2c.md)
