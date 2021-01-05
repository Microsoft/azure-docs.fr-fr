---
title: Authentification pour les solutions d’identité hybride Azure AD
titleSuffix: Active Directory
description: Ce guide s’adresse aux directeurs d’entreprise, directeurs des systèmes d’information, responsables de la sécurité des systèmes d’information, architectes en chef de la gestion des identités, architectes d’entreprise et autres décideurs dans les domaines de la sécurité et de l’informatique en charge de choisir une méthode d’authentification pour leur solution d’identité hybride Azure AD, dans les moyennes et grandes entreprises.
keywords: ''
author: martincoetzer
ms.author: martinco
ms.date: 10/30/2019
ms.topic: article
ms.service: security
ms.subservice: security-fundamentals
ms.workload: identity
ms.openlocfilehash: 15d62f40b50617fd1f6e543cb404a0d38361d3bd
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94836493"
---
# <a name="choose-the-right-authentication-method-for-your-azure-active-directory-hybrid-identity-solution"></a>Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure Active Directory

La première étape pour les organisations souhaitant migrer leurs applications vers le cloud est de choisir une méthode d’authentification appropriée. Cette décision ne doit pas être prise à la légère, et ce pour les raisons suivantes :

1. Il s’agit de la première décision que doit prendre une organisation souhaitant migrer vers le cloud.

2. La méthode d’authentification est un composant essentiel de la présence d’une organisation dans le cloud. Elle contrôle l’accès à l’ensemble des données et des ressources du cloud.

3. Elle constitue le fondement sur lequel reposent toutes les autres fonctionnalités avancées en matière de sécurité et d’expérience utilisateur dans Azure AD.

L’identité constitue le nouveau plan de contrôle en matière de sécurité informatique et, dès lors, l’authentification permet aux organisations de gérer les accès au cloud. Les organisations ont besoin d’un plan de contrôle d’identité qui renforce leur sécurité et protège leurs applications cloud contre les intrus.

> [!NOTE]
> Modifier votre méthode d’authentification implique une planification, des tests et de possibles temps d’arrêt. Le [déploiement intermédiaire](./how-to-connect-staged-rollout.md) constitue un excellent moyen de tester la migration des utilisateurs de l’authentification de la fédération à l’authentification cloud.

### <a name="out-of-scope"></a>Hors propos
Les organisations qui ne présentent aucune empreinte d’un annuaire local existant ne sont pas concernées par cet article. En général, ces entreprises créent uniquement des identités résidant dans le cloud, ce qui n’exige pas une solution d’identité hybride. Les identités résidant uniquement dans le cloud ne sont pas associées à des identités locales correspondantes.

## <a name="authentication-methods"></a>Méthodes d’authentification
En choisissant la solution d’identité hybride Azure AD comme nouveau plan de contrôle, l’authentification constitue la base de l’accès au cloud. Le choix de la méthode d’authentification est une première décision essentielle dans la configuration d’une solution d’identité hybride Azure AD. L’implémentation de la méthode d’authentification est configurée à l’aide d’Azure AD Connect, qui provisionne également les utilisateurs dans le cloud.

Pour choisir une méthode d’authentification, vous devez prendre en compte l’infrastructure existante, le temps nécessaire à l’implémentation, sa complexité et les coûts associés. Ces facteurs sont différents pour chaque organisation et peuvent varier au fil du temps.

>[!VIDEO https://www.youtube.com/embed/YtW2cmVqSEw]

Azure AD prend en charge les méthodes d’authentification suivantes pour les solutions d’identité hybride.

### <a name="cloud-authentication"></a>Authentification cloud
Quand vous choisissez cette méthode d’authentification, Azure AD gère le processus de connexion des utilisateurs. Si vous l’associez à une authentification unique (SSO) fluide, les utilisateurs peuvent se connectent aux applications cloud sans avoir à retaper leurs informations d’identification. L’authentification cloud propose deux options :

**Synchronisation de hachage de mot de passe Azure AD**. Il s’agit du moyen le plus simple d’activer l’authentification pour les objets d’annuaire locaux dans Azure AD. Elle permet aux utilisateurs d’utiliser les mêmes nom d’utilisateur et mot de passe qu’ils utilisent localement sans avoir à déployer une infrastructure supplémentaire. Certaines fonctionnalités Premium d’Azure AD, comme Identity Protection et [Azure Active Directory Domain Services](../../active-directory-domain-services/tutorial-create-instance.md), nécessitent la synchronisation de hachage du mot de passe, quelle que soit la méthode d’authentification choisie.

> [!NOTE]
> Les mots de passe ne sont jamais stockés en texte clair ni chiffrés avec un algorithme réversible dans Azure AD. Pour plus d’informations sur le processus réel de la synchronisation de hachage du mot de passe, consultez [Implémenter la synchronisation de hachage du mot de passe avec la synchronisation Azure AD Connect](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md).

**Authentification directe Azure AD**. Fournit une validation de mot de passe simple pour les services d’authentification Azure AD à l’aide d’un agent logiciel qui s’exécute sur un ou plusieurs serveurs locaux. Les serveurs valident les utilisateurs directement avec votre Active Directory local, ce qui garantit que la validation du mot de passe ne se produit pas dans le cloud.

Cette méthode d’authentification convient pour les entreprises qui, pour des raisons de sécurité, requièrent l’application immédiate d’heures d’ouverture de session, de stratégies de mot de passe et d’états des comptes d’utilisateur locaux. Pour plus d’informations sur le processus réel de l’authentification directe, consultez [Connexion de l’utilisateur avec l’authentification directe Azure AD](../../active-directory/hybrid/how-to-connect-pta.md).

### <a name="federated-authentication"></a>Authentification fédérée
Quand vous choisissez cette méthode d’authentification, Azure AD délègue le processus d’authentification à un système d’authentification approuvée distinct, par exemple les services de fédération Active Directory (AD FS), pour valider le mot de passe de l’utilisateur.

Le système d’authentification peut fournir des conditions d’authentification supplémentaires, comme l’authentification par carte à puce ou une authentification multifacteur tierce. Pour plus d’informations, consultez [Déploiement des services de fédération Active Directory (AD FS)](/windows-server/identity/ad-fs/deployment/windows-server-2012-r2-ad-fs-deployment-guide).

L’arbre de décision présenté dans la section suivante vous aide à déterminer la méthode d’authentification adaptée à vos besoins. Vous pouvez ainsi décider si vous devez déployer une authentification cloud ou une authentification fédérée pour votre solution d’identité hybride Azure AD.

## <a name="decision-tree"></a>Arbre de décision

![Authentification AD Azure : arbre de décision](./media/choose-ad-authn/azure-ad-authn-image1.png)

Détails relatifs aux questions de décision :

1. Azure AD peut gérer la connexion des utilisateurs sans s’appuyer sur des composants locaux pour vérifier les mots de passe.
2. Azure AD peut transférer la connexion des utilisateurs à un fournisseur d’authentification approuvé tel qu’AD FS de Microsoft.
3. Si vous devez appliquer des stratégies de sécurité Active Directory au niveau utilisateur, telles que l’expiration de compte, la désactivation de compte, l’expiration de mot de passe, le verrouillage de compte et les heures de connexion à chaque connexion d’utilisateur, Azure AD requiert certains composants locaux.
4. Fonctionnalités de connexion non prises en charge en mode natif par Azure AD :
   * Connexion à l’aide de cartes à puce ou de certificats.
   * Connexion à l’aide d’un serveur MFA local.
   * Connexion à l’aide d’une solution d’authentification tierce.
   * Solution d’authentification locale multisite.
5. Quelle que soit la méthode de connexion choisie, pour générer le rapport *Utilisateurs avec des informations d’identification divulguées*, Azure AD Identity Protection nécessite une synchronisation du hachage de mot de passe. Les organisations peuvent basculer vers une synchronisation du hachage de mot de passe si leur méthode de connexion principale échoue alors qu’elle a été configurée avant l’événement d’échec.

> [!NOTE]
> Azure AD Identity Protection nécessite des licences [Azure AD Premium P2](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="detailed-considerations"></a>Considérations détaillées

### <a name="cloud-authentication-password-hash-synchronization"></a>Authentification cloud : Synchronisation de hachage de mot de passe

* **Effort**. La synchronisation de hachage du mot de passe nécessite le moins d’effort en matière de déploiement, de maintenance et d’infrastructure.  Ce niveau d’effort s’applique généralement aux organisations dont les utilisateurs se connectent uniquement à Microsoft 365, à des applications SaaS et à d’autres ressources Azure AD basées sur Active Directory. Une fois activée, la synchronisation de hachage du mot de passe fait partie du processus de synchronisation Azure AD Connect et s’exécute toutes les deux minutes.

* **Expérience utilisateur**. Pour améliorer l’expérience de connexion des utilisateurs, déployez l’authentification unique transparente avec synchronisation de hachage du mot de passe. L’authentification unique transparente élimine les invites inutiles quand les utilisateurs sont connectés.

* **Scénarios avancés**. Les organisations peuvent choisir d’utiliser les insights des identités avec les rapports Azure AD Identity Protection avec Azure AD Premium P2, par exemple le rapport sur les informations d’identification divulguées. Windows Hello Entreprise a des [exigences spécifiques quand vous utilisez la synchronisation de hachage du mot de passe](/windows/access-protection/hello-for-business/hello-identity-verification). [Azure Active Directory Domain Services](../../active-directory-domain-services/tutorial-create-instance.md) nécessite la synchronisation de hachage de mot de passe pour fournir aux utilisateurs leurs informations d’identification dans le domaine managé.

    Les organisations qui exigent une authentification multifacteur avec synchronisation du hachage de mot de passe doivent utiliser Azure AD Multi-Factor Authentication ou les [contrôles personnalisés d'accès conditionnel](../../active-directory/conditional-access/controls.md#custom-controls-preview). Elles ne peuvent pas utiliser des méthodes d’authentification multifacteur tierces ou locales qui reposent sur la fédération.

> [!NOTE]
> L’accès conditionnel Azure AD requiert des licences [Azure AD Premium P1](https://azure.microsoft.com/pricing/details/active-directory/).

* **Continuité des activités**. La synchronisation de hachage du mot de passe avec authentification cloud est un service cloud hautement disponible qui s’adapte à tous les centres de données Microsoft. Pour vous assurer que la synchronisation de hachage du mot de passe ne baisse pas pendant de longues périodes, déployez un second serveur Azure AD Connect en mode préproduction dans une configuration de secours.

* **Considérations**. À l’heure actuelle, la synchronisation de hachage du mot de passe n’applique pas immédiatement les changements aux états des comptes locaux. Cela signifie qu’un utilisateur a accès aux applications cloud jusqu’à ce que l’état du compte utilisateur soit synchronisé avec Azure AD. Pour contourner cette limitation, les organisations peuvent exécuter un nouveau cycle de synchronisation après les mises à jour en bloc effectuées par les administrateurs sur les états des comptes locaux, par exemple la désactivation de comptes.

> [!NOTE]
> Les états indiquant un mot de passe expiré et un compte verrouillé ne sont pas synchronisés avec Azure AD à l’aide d’Azure AD Connect. Lorsque vous modifiez le mot de passe d’un utilisateur et configurez l’indicateur *L’utilisateur doit changer de mot de passe à la prochaine ouverture de session*, le hachage de mot de passe n’est pas synchronisé avec Azure AD et Azure AD Connect, tant que l’utilisateur n’aura pas modifié son mot de passe.

Pour obtenir les étapes de déploiement, consultez [Implémentation de la synchronisation de hachage du mot de passe](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md).

### <a name="cloud-authentication-pass-through-authentication"></a>Authentification cloud : Authentification directe  

* **Effort**. L’authentification directe nécessite l’installation d’un ou plusieurs agents légers (trois sont recommandés) sur des serveurs existants. Ces agents doivent avoir accès à vos services AD DS (Active Directory Domain Services) locaux et à vos contrôleurs de domaine AD locaux. Ils doivent disposer d’un accès sortant à Internet et à vos contrôleurs de domaine. Le déploiement des agents dans un réseau de périmètre n’est donc pas pris en charge.

    L’authentification directe nécessite un accès réseau sans contrainte aux contrôleurs de domaine. Tout le trafic réseau est chiffré et limité aux demandes d’authentification. Pour plus d’informations sur ce processus, consultez [l’immersion dans la sécurité](../../active-directory/hybrid/how-to-connect-pta-security-deep-dive.md) sur l’authentification directe.

* **Expérience utilisateur**. Pour améliorer l’expérience de connexion des utilisateurs, déployez l’authentification unique transparente avec l’authentification directe. L’authentification unique transparente élimine les invites inutiles quand les utilisateurs sont connectés.

* **Scénarios avancés**. L’authentification directe applique la stratégie de compte local au moment de la connexion. Par exemple, l’accès est refusé lorsque l’état d’un compte d’utilisateur local indique que le compte est désactivé, verrouillé, que le [mot de passe a expiré](../../active-directory/hybrid/how-to-connect-pta-faq.md#what-happens-if-my-users-password-has-expired-and-they-try-to-sign-in-by-using-pass-through-authentication) ou que les heures de connexion associées au compte ne correspondent pas aux heures d’ouverture de session autorisées de l’utilisateur.

    Les organisations qui exigent une authentification multifacteur avec authentification directe doivent utiliser Azure AD Multi-Factor Authentication (MFA) ou les [contrôles personnalisés d'accès conditionnel](../../active-directory/conditional-access/controls.md#custom-controls-preview). Ces organisations ne peuvent pas utiliser des méthodes d’authentification multifacteur tierces ou locales qui reposent sur la fédération. Les fonctionnalités avancées nécessitent le déploiement de la synchronisation de hachage du mot de passe (que vous choisissiez l’authentification directe ou non). C’est le cas notamment du rapport sur la fuite des informations d’identification généré par Identity Protection.

* **Continuité des activités**. Nous vous recommandons de déployer deux agents d’authentification directe supplémentaires. Ces agents complètent le premier agent sur le serveur Azure AD Connect. Ce déploiement supplémentaire garantit la haute disponibilité des demandes d’authentification. Quand trois agents sont déployés et que l’un d’eux est hors service pour maintenance, l’échec d’un agent n’a aucune incidence.

    Outre l’authentification directe, le déploiement de la synchronisation de hachage du mot de passe offre un autre avantage. Il sert de méthode d’authentification de secours lorsque la méthode d’authentification principale n’est plus disponible.

* **Considérations**. Vous pouvez utiliser la synchronisation de hachage de mot de passe comme une méthode d’authentification de secours pour l’authentification directe, et les agents ne peuvent pas valider les informations d’identification d’un utilisateur en raison d’un incident local. Le basculement vers la synchronisation de hachage du mot de passe ne se produit pas automatiquement et vous devez utiliser Azure AD Connect pour basculer manuellement la méthode de connexion.

    Pour découvrir d’autres considérations sur l’authentification directe, notamment la prise en charge d’ID de connexion de substitution, consultez le [forum aux questions](../../active-directory/hybrid/how-to-connect-pta-faq.md).

Pour obtenir les étapes de déploiement à suivre, consultez [Implémentation de l’authentification directe](../../active-directory/hybrid/how-to-connect-pta.md).

### <a name="federated-authentication"></a>Authentification fédérée

* **Effort**. L’utilisation d’un système d’authentification fédérée s’appuie sur un système externe de confiance pour authentifier les utilisateurs. Certaines entreprises souhaitent rentabiliser leur investissement et réutiliser leur système fédéré existant avec leur solution d’identité hybride Azure AD. La maintenance et la gestion du système fédéré ne relèvent pas d’Azure AD. Il appartient à l’organisation d’utiliser le système fédéré pour vérifier qu’il est déployé de manière sécurisée et qu’il peut gérer la charge de l’authentification.

* **Expérience utilisateur**. L’expérience utilisateur de l’authentification fédérée dépend de l’implémentation de fonctionnalités, de la topologie et de la configuration de la batterie de serveurs de fédération. Certaines organisations ont besoin de cette souplesse pour configurer l’accès à la batterie de serveurs de fédération en fonction de leurs exigences en matière de sécurité. Par exemple, il est possible de configurer en interne des utilisateurs connectés et des appareils capables de connecter automatiquement les utilisateurs. Aucune information d’identification n’est donc demandée aux utilisateurs. Cette configuration fonctionne car ils sont déjà connectés à leur appareil. Si nécessaire, certaines fonctionnalités de sécurité avancées compliquent le processus de connexion des utilisateurs.

* **Scénarios avancés**. Une solution d’authentification fédérée est requise lorsque les clients ont une exigence d’authentification non prise en charge par Azure AD en mode natif. Affichez des informations détaillées pour vous aider à [choisir l’option de connexion adaptée](/archive/blogs/samueld/choosing-the-right-sign-in-option-to-connect-to-azure-ad-office-365). Examinez les exigences courantes suivantes :

  * Authentification nécessitant des cartes à puce ou des certificats.
  * Les serveurs MFA locaux ou fournisseurs d’authentification multifacteur tiers nécessitent un fournisseur d’identité fédéré.
  * Authentification à l’aide de solutions d’authentification tierces. Consultez la [liste de compatibilité de fédération Azure AD](../../active-directory/hybrid/how-to-connect-fed-compatibility.md).
  * Connexion nécessitant un sAMAccountName, par exemple DOMAINE\nomutilisateur, et non avec un nom d’utilisateur principal (UPN) comme user@domain.com.

* **Continuité des activités**. Les systèmes fédérés nécessitent généralement un groupe de serveurs à charge équilibrée, également appelé « batterie de serveurs ». Cette batterie est configurée dans une topologie de réseau interne et de réseau de périmètre pour garantir la haute disponibilité des demandes d’authentification.

    Déployez une synchronisation de hachage du mot de passe conjointement avec l’authentification fédérée comme méthode d’authentification de secours quand la méthode d’authentification principale n’est plus disponible, par exemple en cas d’indisponibilité des serveurs locaux. Certaines grandes entreprises exigent une solution de fédération pour prendre en charge plusieurs points d’entrée Internet configurés avec géo-DNS pour les demandes d’authentification à faible latence.

* **Considérations**. Les systèmes fédérés nécessitent généralement un investissement plus important dans l’infrastructure locale. La plupart des organisations choisissent cette option si elles disposent déjà d’une solution de fédération locale et qu’elles sont contraintes d’utiliser un fournisseur d’identité unique pour des raisons commerciales. La fédération est plus difficile à utiliser et à dépanner que les solutions d’authentification cloud.

Avec un domaine non routable qui ne peut pas être vérifié dans Azure AD, l’implémentation d’une connexion d’ID utilisateur nécessite une configuration supplémentaire. Cette exigence est connue sous le nom de « prise en charge des ID de connexion de substitution ». Pour connaître les limitations et les exigences, consultez [Configuration d’un ID de connexion de substitution](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id). Si vous choisissez d’utiliser un fournisseur d’authentification multifacteur tiers avec la fédération, vérifiez qu’il prend en charge WS-Trust pour autoriser les appareils à rejoindre Azure AD.

Pour accéder aux étapes de déploiement, consultez [Déploiement des serveurs de fédération](/windows-server/identity/ad-fs/deployment/deploying-federation-servers).

> [!NOTE]
> Quand vous déployez votre solution d’identité hybride Azure AD, vous devez veiller à implémenter l’une des topologies prises en charge d’Azure AD Connect. Pour découvrir les configurations prises en charge et celles qui ne le sont pas, consultez[Topologies pour Azure AD Connect](../../active-directory/hybrid/plan-connect-topologies.md).

## <a name="architecture-diagrams"></a>Diagrammes d’architecture

Les diagrammes suivants présentent les composants architecturaux de haut niveau nécessaires pour chaque méthode d’authentification que vous pouvez utiliser avec votre solution d’identité hybride Azure AD. Ils vous offrent une vue d’ensemble pour vous aider à comparer les différences entre les solutions.

* Simplicité d’une solution de synchronisation de hachage du mot de passe :

    ![Identité hybride Azure AD avec synchronisation de hachage de mot de passe](./media/choose-ad-authn/azure-ad-authn-image2.png)

* Configuration requise de l’agent d’authentification directe, à l’aide de deux agents pour la redondance :

    ![Identité hybride Azure AD avec authentification directe](./media/choose-ad-authn/azure-ad-authn-image3.png)

* Composants obligatoires pour la fédération dans le réseau interne et le réseau de périmètre de votre organisation :

    ![Identité hybride Azure AD avec authentification fédérée](./media/choose-ad-authn/azure-ad-authn-image4.png)

## <a name="comparing-methods"></a>Comparaison des méthodes

|Considération|Synchronisation de hachage du mot de passe + authentification unique transparente|Authentification directe + authentification unique transparente|Fédération avec AD FS|
|:-----|:-----|:-----|:-----|
|Où l’authentification se produit-elle ?|Dans le cloud|Dans le cloud, après un échange de vérification de mot de passe sécurisé avec l’agent d’authentification local|Local|
|Quelles sont les exigences pour le serveur local au-delà du système de provisionnement : Azure AD Connect ?|None|Un serveur par agent d’authentification supplémentaire|Deux ou plusieurs serveurs AD FS<br><br>Deux ou plusieurs serveurs WAP dans le réseau de périmètre/DMZ|
|Quelles sont les exigences Internet et réseau locales au-delà du système de provisionnement ?|None|[Accès Internet sortant](../../active-directory/hybrid/how-to-connect-pta-quick-start.md) à partir des serveurs exécutant des agents d’authentification|[Accès Internet entrant](/windows-server/identity/ad-fs/overview/ad-fs-requirements) pour les serveurs WAP dans le périmètre<br><br>Accès réseau entrant aux serveurs AD FS à partir des serveurs WAP dans le périmètre<br><br>Équilibrage de charge réseau|
|Y a-t-il une exigence de certificat TLS/SSL ?|Non|Non|Oui|
|Existe-t-il une solution de supervision de l’intégrité ?|Non requis|État de l’agent fourni par le [Centre d’administration Azure Active Directory](../../active-directory/hybrid/tshoot-connect-pass-through-authentication.md)|[Azure AD Connect Health](../../active-directory/hybrid/how-to-connect-health-adfs.md)|
|Les utilisateurs obtiennent-ils une authentification unique auprès des ressources cloud à partir d’appareils joints au domaine au sein du réseau d’entreprise ?|Oui avec [authentification unique fluide](../../active-directory/hybrid/how-to-connect-sso.md)|Oui avec [authentification unique fluide](../../active-directory/hybrid/how-to-connect-sso.md)|Oui|
|Quels types de connexion sont pris en charge ?|UserPrincipalName + mot de passe<br><br>Authentification Windows intégrée avec [authentification unique transparente](../../active-directory/hybrid/how-to-connect-sso.md)<br><br>[ID de connexion de substitution](../../active-directory/hybrid/how-to-connect-install-custom.md)|UserPrincipalName + mot de passe<br><br>Authentification Windows intégrée avec [authentification unique transparente](../../active-directory/hybrid/how-to-connect-sso.md)<br><br>[ID de connexion de substitution](../../active-directory/hybrid/how-to-connect-pta-faq.md)|UserPrincipalName + mot de passe<br><br>sAMAccountName + mot de passe<br><br>Authentification Windows intégrée<br><br>[Authentification par certificat et carte à puce](/windows-server/identity/ad-fs/operations/configure-user-certificate-authentication)<br><br>[ID de connexion de substitution](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)|
|Windows Hello Entreprise est-il pris en charge ?|[Modèle de confiance de clé](/windows/security/identity-protection/hello-for-business/hello-identity-verification)|[Modèle de confiance de clé](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br>*Nécessite le niveau fonctionnel de domaine Windows Server 2016*|[Modèle de confiance de clé](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Modèle de certificat de confiance ](/windows/security/identity-protection/hello-for-business/hello-key-trust-adfs)|
|Quelles sont les options d’authentification multifacteur ?|[Azure AD MFA](/azure/multi-factor-authentication/)<br><br>[Contrôles personnalisés avec accès conditionnel*](../../active-directory/conditional-access/controls.md)|[Azure AD MFA](/azure/multi-factor-authentication/)<br><br>[Contrôles personnalisés avec accès conditionnel*](../../active-directory/conditional-access/controls.md)|[Azure AD MFA](/azure/multi-factor-authentication/)<br><br>[Serveur Azure MFA](../../active-directory/authentication/howto-mfaserver-deploy.md)<br><br>[MFA tiers](/windows-server/identity/ad-fs/operations/configure-additional-authentication-methods-for-ad-fs)<br><br>[Contrôles personnalisés avec accès conditionnel*](../../active-directory/conditional-access/controls.md)|
|Quels sont les états de compte d’utilisateur pris en charge ?|Comptes désactivés<br>(délai pouvant atteindre 30 minutes)|Comptes désactivés<br><br>Compte verrouillé<br><br>Compte expiré<br><br>Mot de passe expiré<br><br>Heures de connexion|Comptes désactivés<br><br>Compte verrouillé<br><br>Compte expiré<br><br>Mot de passe expiré<br><br>Heures de connexion|
|Quelles sont les options d’accès conditionnel ?|[Accès conditionnel Azure AD, avec Azure AD Premium](../../active-directory/conditional-access/overview.md)|[Accès conditionnel Azure AD, avec Azure AD Premium](../../active-directory/conditional-access/overview.md)|[Accès conditionnel Azure AD, avec Azure AD Premium](../../active-directory/conditional-access/overview.md)<br><br>[Règles de revendication AD FS](https://adfshelp.microsoft.com/AadTrustClaims/ClaimsGenerator)|
|Le blocage des protocoles hérités est-il pris en charge ?|[Oui](../../active-directory/conditional-access/overview.md)|[Oui](../../active-directory/conditional-access/overview.md)|[Oui](/windows-server/identity/ad-fs/operations/access-control-policies-w2k12)|
|Pouvez-vous personnaliser le logo, l’image et la description sur les pages de connexion ?|[Oui, avec Azure AD Premium](../../active-directory/fundamentals/customize-branding.md)|[Oui, avec Azure AD Premium](../../active-directory/fundamentals/customize-branding.md)|[Oui](../../active-directory/hybrid/how-to-connect-fed-management.md)|
|Quels sont les scénarios avancés pris en charge ?|[Verrouillage de mot de passe intelligent](../../active-directory/authentication/howto-password-smart-lockout.md)<br><br>[Rapports des informations d’identification divulguées, avec Azure AD Premium P2](../identity-protection/overview-identity-protection.md)|[Verrouillage de mot de passe intelligent](../../active-directory/authentication/howto-password-smart-lockout.md)|Système d’authentification multisite à faible latence<br><br>[Verrouillage extranet AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection)<br><br>[Intégration aux systèmes d’identité tiers](../../active-directory/hybrid/how-to-connect-fed-compatibility.md)|

> [!NOTE]
> Contrôles personnalisés dans Azure AD. L’accès conditionnel ne prend actuellement pas en charge l’inscription d’appareil.

## <a name="recommendations"></a>Recommandations
Votre système d’identité garantit à vos utilisateurs l’accès aux applications cloud et aux applications métier que vous migrez et rendez accessibles dans le cloud. Pour assurer la productivité des utilisateurs autorisés et protéger les données sensibles de votre organisation contre les intrus, k’authentification contrôle l’accès aux applications.

Utilisez ou activez la synchronisation de hachage du mot de passe, quelle que soit la méthode d’authentification choisie, et ce pour les raisons suivantes :

1. **Haute disponibilité et récupération d’urgence**. L’authentification directe et la fédération s’appuient sur une infrastructure locale. Pour l’authentification directe, l’empreinte locale inclut le matériel de serveur et la mise en réseau requis par les agents d’authentification directe. Pour la fédération, l’empreinte locale est encore plus importante. Elle nécessite des serveurs dans votre réseau de périmètre pour rediriger par l’intermédiaire d’un proxy les demandes d’authentification et les serveurs de fédération interne.

    Pour éviter des points de défaillance uniques, déployez des serveurs redondants. Cette méthode garantit le traitement des requêtes d’authentification en cas d’échec d’un composant. Du point de vue de l’authentification directe comme de la fédération, il est aussi attendu que les contrôleurs de domaine répondent aux demandes d’authentification, qui peuvent également échouer. L’intégrité de nombreux composants passe par des opérations de maintenance. Si celles-ci ne sont pas planifiées et implémentées correctement, le risque de panne est plus élevé. La synchronisation de hachage du mot de passe permet d’éviter les pannes, car le service d’authentification cloud Azure AD de Microsoft est mis à l’échelle globalement et toujours disponible.

2. **Survie à une panne locale**.  Les conséquences d’une panne locale à la suite d’une cyberattaque ou d’un sinistre peuvent être considérables, de l’atteinte à l’image de marque à la paralysie de l’organisation si celle-ci ne parvient pas à gérer l’attaque. Récemment, de nombreuses organisations ont été victimes d’attaques menées par des programmes malveillants, notamment des ransomwares (rançongiciels), qui ont provoqué la mise hors service de leurs serveurs locaux. Lorsque Microsoft aide les clients à gérer ces types d’attaques, deux catégories d’organisations se dégagent :

   * Les organisations qui utilisaient précédemment la synchronisation de hachage de mot de passe en plus de l’authentification fédérée ou directe ont modifié leur méthode d’authentification principale pour utiliser la synchronisation de hachage de mot de passe. Ils étaient de nouveau en ligne après quelques heures. En accédant aux e-mails par le biais de Microsoft 365, elles ont pu résoudre les problèmes et accéder à d’autres charges de travail sur le cloud.

   * Les organisations n’ayant pas auparavant activé la synchronisation de hachage du mot de passe ont dû faire appel à des systèmes de messagerie externes non fiables pour la communication et la résolution des problèmes. Dans ce cas, il leur a fallu des semaines pour restaurer leur infrastructure d’identité locale, avant que les utilisateurs ne puissent se connecter à nouveau aux applications basées sur le Cloud.

3. **Protection des identités**. Azure AD Identity Protection avec Azure AD Premium P2 est un des meilleurs moyens de protéger les utilisateurs dans le cloud. Microsoft analyse en permanence Internet à la recherche de listes d’utilisateurs et de mots de passe vendues et mises à disposition sur le « dark web » par des utilisateurs malveillants. Azure AD peut utiliser ces informations pour vérifier si les noms d’utilisateur et les mots de passe de votre organisation sont compromis. Dès lors, il est primordial d’activer la synchronisation de hachage du mot de passe, et ce quelle que soit la méthode d’authentification vous utilisez (fédérée ou directe). Les informations d’identification divulguées sont présentées sous forme de rapport. Utilisez ces informations pour bloquer ou forcer les utilisateurs à modifier leurs mots de passe lorsqu’ils tentent de se connecter avec des mots de passe divulgués.

## <a name="conclusion"></a>Conclusion

Cet article présente les différentes options d’authentification que les organisations peuvent configurer et déployer pour prendre en charge l’accès aux applications cloud. Face à des exigences professionnelles, sécuritaires et techniques variées, les organisations ont le choix entre la synchronisation de hachage du mot de passe, l’authentification directe et la fédération.

Examinez chaque méthode d’authentification. L’effort visant à déployer la solution et l’expérience utilisateur du processus de connexion répondent-ils aux besoins de votre entreprise ? Déterminez également si votre organisation a besoin des scénarios avancés et des fonctionnalités de continuité d’activité de chaque méthode d’authentification. Enfin, vous devez évaluer les considérations relatives à chaque méthode d’authentification pour voir si l’une d’elles empêche l’implémentation de votre choix.

## <a name="next-steps"></a>Étapes suivantes

Aujourd’hui, les menaces sont présentes 24 heures sur 24 et proviennent de partout. L’implémentation d’une méthode d’authentification appropriée permet d’atténuer les risques en matière de sécurité et de protéger vos identités.

[Démarrez](../fundamentals/active-directory-whatis.md) avec Azure AD et déployez la solution d’authentification adaptée à votre organisation.

Si vous envisagez de passer de l’authentification fédérée à l’authentification cloud, découvrez plus en détail [comment changer la méthode de connexion](../../active-directory/hybrid/plan-connect-user-signin.md). Pour vous aider à planifier et à implémenter la migration, utilisez [ces plans de déploiement de projet](../fundamentals/active-directory-deployment-plans.md) ou envisagez d’utiliser la nouvelle fonctionnalité de [Déploiement intermédiaire](../../active-directory/hybrid/how-to-connect-staged-rollout.md) pour migrer les utilisateurs fédérés vers à l’aide de l’authentification cloud dans une approche intermédiaire.