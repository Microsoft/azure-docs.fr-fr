---
title: 'Gestion des identités et des accès : bonnes pratiques de sécurité Azure | Microsoft Docs'
description: Cet article détaille un ensemble de meilleures pratiques en matière de gestion des identités et du contrôle d’accès à l’aide de fonctionnalités Azure intégrées.
services: security
documentationcenter: na
author: terrylanfear
manager: RKarlin
editor: TomSh
ms.assetid: 07d8e8a8-47e8-447c-9c06-3a88d2713bc1
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/28/2019
ms.author: terrylan
ms.openlocfilehash: bf593eab54233ab65435585485dd8e3940f5addb
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489817"
---
# <a name="azure-identity-management-and-access-control-security-best-practices"></a>Meilleures pratiques en matière de sécurité du contrôle d’accès et de la gestion des identités Azure

Dans cet article, nous étudions une collection de bonnes pratiques en matière de sécurité du contrôle d’accès et de la gestion des identités Azure. Ces meilleures pratiques sont issues de notre expérience avec [Azure AD](../../active-directory/fundamentals/active-directory-whatis.md), mais également de celle des clients, comme vous.

Pour chaque bonne pratique, nous détaillons les éléments suivants :

* Nature de la bonne pratique
* Raison pour laquelle activer cette bonne pratique
* Conséquence possible en cas de non-utilisation de la bonne pratique
* Alternatives possibles à la meilleure pratique
* Comment apprendre à utiliser la bonne pratique

Cet article repose sur un consensus, ainsi que sur les fonctionnalités et ensembles de fonctions de la plateforme Azure disponibles lors de l’écriture.

Cet article a pour objectif de fournir une feuille de route générale dans une démarche de sécurité renforcée après le déploiement, à l’aide de notre liste de contrôle « [Cinq étapes pour sécuriser votre infrastructure d’identité](steps-secure-identity.md) », qui vous guide tout au long de nos principaux services et fonctionnalités.

Les opinions et avis évoluent au fil du temps ; cet article sera régulièrement mis à jour de manière à tenir compte de ces changements.

Dans cet article, nous allons étudier les points suivants :

* Traiter l’identité en tant que périmètre de sécurité principal
* La centralisation de la gestion des identités
* Gérer les locataires connectés
* Activer l’authentification unique
* Activer l’accès conditionnel
* Planifier les améliorations de la sécurité de routine
* Activer la gestion des mots de passe
* Appliquer la vérification multifacteur pour les utilisateurs
* Utiliser le contrôle d’accès en fonction du rôle
* Exposition réduite des comptes privilégiés
* Contrôle des emplacements où se trouvent les ressources
* Utiliser Azure AD pour l’authentification du stockage

## <a name="treat-identity-as-the-primary-security-perimeter"></a>Traiter l’identité en tant que périmètre de sécurité principal

Beaucoup de gens considèrent l’identité comme le périmètre principal pour la sécurité. Il s’agit d’un changement par rapport à l’objectif traditionnel sur la sécurité réseau. Les périmètres de réseau continuent d’être plus poreux et la défense du périmètre ne peut plus être aussi efficace qu’avant l’explosion des appareils [BYOD](/mem/intune/fundamentals/byod-technology-decisions) et des applications cloud.

[Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md) est la solution Azure pour la gestion des identités et des accès. Azure AD est un service multilocataire basé sur le cloud pour la gestion des identités et des annuaires par Microsoft. Il associe des services d’annuaires principaux, la gestion de l’accès aux applications et la protection des identités dans une même solution.

Les sections suivantes répertorient les meilleures pratiques pour la sécurité des identités et des accès à l’aide d’Azure AD.

**Bonne pratique** : Centrez les contrôles et les détections de sécurité autour des identités d’utilisateur et de service.
**Détail** : Utilisez Azure AD pour colocaliser des contrôles et des identités.

## <a name="centralize-identity-management"></a>La centralisation de la gestion des identités

Dans un scénario [d’identité hybride](https://resources.office.com/ww-landing-M365E-EMS-IDAM-Hybrid-Identity-WhitePaper.html?), nous vous recommandons d’intégrer vos répertoires cloud et locaux. L’intégration permet à votre équipe informatique de gérer des comptes depuis un emplacement, quel que soit l’endroit où un compte est créé. L’intégration améliore également la productivité de vos utilisateurs en leur fournissant une identité commune pour accéder aux ressources cloud et locales.

**Bonne pratique** : Établir une instance Azure AD unique. La cohérence et une source d’autorité unique offrent plus de clarté et réduisent les risques de sécurité résultant d’erreurs humaines et de la complexité de la configuration.
**Détail** : Désignez un annuaire Azure AD unique comme source d’autorité pour les comptes professionnels.

**Bonne pratique** : Intégrez vos répertoires locaux à Azure AD.  
**Détail** : Utilisez [Azure AD Connect](../../active-directory/hybrid/whatis-hybrid-identity.md) pour synchroniser votre annuaire local avec votre annuaire cloud.

> [!Note]
> Certains [facteurs impactent les performances d’Azure AD Connect](../../active-directory/hybrid/plan-connect-performance-factors.md). Assurez-vous qu'Azure AD Connect a une capacité suffisante pour empêcher des systèmes peu performants de perturber la sécurité et la productivité. Les organisations complexes ou de grande taille (celles devant provisionner plus de 100 000 objets) devraient suivre les [recommandations](../../active-directory/hybrid/whatis-hybrid-identity.md) pour optimiser leur implémentation d’Azure AD Connect.

**Bonne pratique** : Ne synchronisez pas de comptes qui ont des privilèges élevés dans votre instance Active Directory existante vers Azure AD.
**Détail** : Ne modifiez pas la valeur par défaut [Configuration Azure AD Connect](../../active-directory/hybrid/how-to-connect-sync-configure-filtering.md) qui exclut ces comptes. Cette configuration réduit le risque de passage contradictoire dans le cloud de ressources locales (ceci pouvant être à l’origine d’un incident majeur).

**Bonne pratique** : Activez la synchronisation de hachage de mot de passe.  
**Détail** : La synchronisation de hachage de mot de passe est une fonctionnalité permettant de synchroniser des code de hachage des mots de passe utilisateur entre une instance Active Directory locale et une instance cloud Azure AD. Cette synchronisation contribue à empêcher la relecture d’informations d’identification ayant fait l’objet de fuites lors d’attaques précédentes.

Même si vous décidez d’utiliser la fédération avec Active Directory Federation Services (AD FS) ou d’autres fournisseurs d’identité, vous pouvez éventuellement configurer la synchronisation de hachage de mot de passe en tant que sauvegarde au cas où vos serveurs locaux connaîtraient une défaillance ou deviendraient temporairement non disponibles. Cette synchronisation permet aux utilisateurs de se connecter au service à l’aide du mot de passe qu’ils utilisent pour se connecter à leur instance Active Directory locale. Cela permet également à la protection d’identité de détecter les informations d’identification compromises en comparant des codes de hachage de mot de passe synchronisés avec des mots de passe connus pour être compromis, si un utilisateur a utilisé les mêmes adresse de messagerie et mot de passe sur d’autres services qui ne sont pas connectés à Azure AD.

Pour plus d’informations, consultez [Implémenter la synchronisation de hachage du mot de passe avec la synchronisation Azure AD Connect](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md).

**Bonne pratique** : Pour développer une nouvelle application, utilisez Azure AD pour l’authentification.
**Détail** : Utilisez les fonctionnalités appropriées pour prendre en charge l’authentification :

  - Azure AD pour les employés
  - [Azure AD B2B](../../active-directory/external-identities/index.yml) pour les utilisateurs invités et les partenaires externes
  - [Azure AD B2C](../../active-directory-b2c/index.yml) pour contrôler la façon dont les clients s’inscrivent, se connectent et gèrent leurs profils lorsqu’ils utilisent vos applications

Les organisations qui n’intègrent pas leur identité locale avec leur identité cloud peuvent avoir une charge plus importante dans la gestion des comptes. Cette surcharge augmente la probabilité d’erreurs et de failles de sécurité.

> [!Note]
> Vous devez choisir dans quels répertoires les comptes critiques résideront, et si la station de travail d’administrateur utilisée sera gérée par de nouveaux service cloud ou par des processus existants. L’utilisation de processus d’administration et d’approvisionnement d’identités existants peut réduire certains risques, mais également présenter un risque de compromission d’un compte local par un attaquant et de passage dans le cloud. Vous souhaiterez peut-être utiliser une stratégie différente pour différents rôles (par exemple, administrateur IT vs administrateur d’unité). Vous avez le choix entre deux options. La première option consiste à créer des comptes Azure AD qui ne sont pas synchronisées avec votre instance Active Directory locale. Reliez votre station de travail d’administrateur à Azure AD. L’administration et l’application de correctifs sont possibles avec Microsoft Intune. La seconde option consiste à utiliser des comptes d’administrateur existants en synchronisant avec votre instance Active Directory locale. Utilisez des stations de travail existantes dans votre domaine Active Directory pour l’administration et la sécurité.

## <a name="manage-connected-tenants"></a>Gérer les locataires connectés
Votre organisation de sécurité a besoin de visibilité pour évaluer les risques et déterminer si les stratégies de votre organisation et les exigences réglementaires sont respectées. Vous devez vous assurer que votre organisation de sécurité a une visibilité de tous les abonnements connectés à votre environnement de production et à votre réseau (via [Azure ExpressRoute](../../expressroute/expressroute-introduction.md) ou [VPN site à site](../../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)). Un [Administrateur général/Administrateur de société](../../active-directory/roles/permissions-reference.md#company-administrator-permissions) dans Azure AD peuvent élever leurs privilèges d’accès au rôle d’[Administrateur d’accès utilisateur](../../role-based-access-control/built-in-roles.md#user-access-administrator) et voir tous les abonnements et groupes managés connectés à votre environnement.

Voir [Élever les privilèges d’accès pour gérer tous les abonnements et groupes d’administration Azure](../../role-based-access-control/elevate-access-global-admin.md) pour vous assurer que vous et votre groupe de sécurité pouvez afficher tous les abonnements ou groupes d’administration connectés à votre environnement. Vous devez supprimer ces privilèges d’accès élevés une fois que vous avez évalué les risques.

## <a name="enable-single-sign-on"></a>Activer l’authentification unique

Dans un monde où mobilité et cloud occupent le premier plan, vous souhaitez activer l’authentification unique (SSO) sur tous les appareils, les applications et les services afin que vos utilisateurs soient être productifs où qu’ils soient et à tout moment. La gestion de plusieurs solutions d’identité pose un problème d’administration, non seulement pour l’informatique, mais également pour les utilisateurs qui auront à mémoriser plusieurs mots de passe.

En utilisant la même solution d’identité pour toutes vos applications et vos ressources, vous pouvez obtenir une authentification unique. Vos utilisateurs peuvent utiliser le même jeu d’informations d’identification pour s’authentifier et accéder aux ressources dont ils ont besoin, qu’elles soient situées en local ou dans le cloud.

**Bonne pratique** : Activez l’authentification unique.  
**Détail** : Azure AD [étend les versions locales d’Active Directory](../../active-directory/hybrid/whatis-hybrid-identity.md) sur le cloud. Les utilisateurs peuvent utiliser leur compte professionnel ou scolaire principal pour leurs appareils joints au domaine, les ressources de l’entreprise et toutes les applications web et SaaS dont ils ont besoin pour accomplir leur travail. Les utilisateurs n’ont plus besoin de gérer plusieurs combinaisons de nom d’utilisateur et mot de passe et l’accès aux applications peut être automatiquement mis en service (ou au contraire retiré) en fonction de leur appartenance aux groupes de l’entreprise et de leur statut en tant qu’employé. Vous pouvez en outre contrôler l’accès aux applications de la galerie ou aux applications en local que vous avez développées et publiées via le [proxy d’application Azure AD](../../active-directory/manage-apps/application-proxy.md).

L’authentification unique permet aux utilisateurs d’accéder à leurs [applications SaaS](../../active-directory/manage-apps/what-is-single-sign-on.md) avec leur compte professionnel ou scolaire dans Azure AD. Ceci s’applique non seulement aux applications SaaS de Microsoft, mais également à d’autres applications, telles que [Google Apps](../../active-directory/saas-apps/google-apps-tutorial.md) et [Salesforce](../../active-directory/saas-apps/salesforce-tutorial.md). Vous pouvez configurer votre application pour utiliser Azure AD comme fournisseur [d’identité SAML](../../active-directory/fundamentals/active-directory-whatis.md). Pour contrôler la sécurité, Azure AD n’émet pas de jetons permettant aux utilisateurs de se connecter à l’application avant que l’accès n’ait été octroyé par Azure AD. Les utilisateurs peuvent accorder un accès direct ou via un groupe dont ils sont membres.

Les organisations qui ne créent pas une identité commune pour établir l’authentification unique pour leurs utilisateurs et les applications sont davantage exposées à des scénarios où les utilisateurs disposent de plusieurs mots de passe. Ces scénarios augmentent les chances qu’ils réutilisent des mots de passe ou qu’ils choisissent des mots de passe faibles.

## <a name="turn-on-conditional-access"></a>Activer l’accès conditionnel

Les utilisateurs peuvent accéder aux ressources de votre organisation en utilisant différents appareils et applications, n’importe où. En tant qu’administrateur, vous souhaitez vous assurer que ces appareils répondent à vos normes de sécurité et de conformité. Contrôler les personnes autorisées à accéder à une ressource ne suffit plus.

Afin d’équilibrer la sécurité et la productivité, vous devez aussi tenir compte des moyens d’accéder à une ressource avant de pouvoir prendre une décision relative au contrôle d’accès. L’accès conditionnel Azure AD vous permet de satisfaire cette exigence. Avec l’accès conditionnel, vous pouvez prendre des décisions de contrôle d’accès automatisées basées sur les conditions d’accès à vos applications cloud.

**Bonne pratique** : Gérez et contrôlez l’accès aux ressources de l’entreprise.  
**Détail** : Configurez des [stratégies courantes d’accès conditionnel](../../active-directory/conditional-access/concept-conditional-access-policy-common.md) Azure AD en fonction du groupe, de l’emplacement et du niveau de confidentialité des applications SaaS et de celles connectées à Azure AD.

**Bonne pratique** : Bloquez les protocoles d’authentification hérités.
**Détail** : Les attaquants exploitent chaque jour les failles de protocoles plus anciens, en concernant les attaques par pulvérisations de mots de passe. Configurez l’accès conditionnel pour [bloquer les protocoles hérités](../../active-directory/conditional-access/howto-conditional-access-policy-block-legacy.md).

## <a name="plan-for-routine-security-improvements"></a>Planifier les améliorations de la sécurité de routine

La sécurité est en constante évolution, et il est important d’intégrer à votre infrastructure de gestion du cloud et des identités un moyen de montrer régulièrement la croissance et de découvrir de nouvelles façons de sécuriser votre environnement.

Identity Secure Score est un ensemble de contrôles de sécurité recommandés que Microsoft publie et qui vise à vous fournir un score numérique pour mesurer objectivement votre posture de sécurité et vous aider à planifier les futures améliorations de sécurité. Vous pouvez également consulter votre score par rapport à celui d’autres secteurs d’activité ainsi que vos propres tendances au fil du temps.

**Bonne pratique** : Planifiez des révisions et des améliorations de sécurité de routine basées sur les bonnes pratiques de votre secteur d’activité.
**Détail** : Utilisez la fonctionnalité Score d’identité sécurisée pour classer vos améliorations dans le temps.

## <a name="enable-password-management"></a>Activer la gestion des mots de passe

Si vous avez plusieurs locataires ou si vous voulez permettre aux utilisateurs de [réinitialiser leurs mots de passe](../../active-directory/user-help/active-directory-passwords-update-your-own-password.md), il est important d’utiliser des stratégies de sécurité appropriées afin d’éviter les abus.

**Bonne pratique** : Configurez la réinitialisation de mot de passe en libre-service pour vos utilisateurs.  
**Détail** : Utilisez la fonctionnalité de [réinitialisation de mot de passe en libre-service](../../active-directory-b2c/user-flow-self-service-password-reset.md) d’Azure AD.

**Bonne pratique** : Effectuez un monitoring de l’utilisation réelle de la réinitialisation de mot de passe en libre-service.  
**Détail** : Effectuez un monitoring des utilisateurs qui s’inscrivent avec le [Rapport d’activité d’inscription à la réinitialisation de mot de passe](../../active-directory/authentication/howto-sspr-reporting.md) Azure AD. La fonctionnalité de création de rapports fournie par Azure AD vous aide à répondre aux questions à l’aide de rapports prédéfinis. Si vous disposez d’une licence appropriée, vous pouvez également créer des requêtes personnalisées.

**Bonne pratique** : Étendez des stratégies de mot de passe basé sur le cloud à votre infrastructure locale.
**Détail** : Améliorez les stratégies de mot de passe de votre organisation en effectuant les mêmes vérifications pour les modifications de mots de passe en local, que celles que vous feriez pour les modifications de mots de passe basés sur le cloud. Installez la [protection de mot de passe Azure AD](../../active-directory/authentication/concept-password-ban-bad.md) pour les agents Windows Server Active Directory en local afin d’étendre de listes de mots de passe interdits à votre infrastructure existante. Les utilisateurs et les administrateurs qui modifient, définissent ou réinitialisent des mots de passe localement doivent se conformer à la même stratégie de mot de passe que les utilisateurs cloud uniquement.

## <a name="enforce-multi-factor-verification-for-users"></a>Appliquer la vérification multifacteur pour les utilisateurs

Nous vous recommandons d’exiger une vérification en deux étapes pour tous vos utilisateurs. Cela inclut les administrateurs et les autres membres de votre organisation (par exemple, les responsables financiers) dont la compromission de leur compte pourrait avoir un impact significatif si leur compte est compromis.

Il existe plusieurs options pour exiger une vérification en deux étapes. La meilleure option pour vous dépend de vos objectifs, de l’édition d’Azure AD utilisée et de votre programme de licence. Consultez [Comment exiger la vérification en deux étapes pour un utilisateur](../../active-directory/authentication/howto-mfa-userstates.md) pour déterminer la meilleure option pour vous. Consultez les pages de tarification [Azure AD](https://azure.microsoft.com/pricing/details/active-directory/) et [Azure AD Multi-Factor Authentication](https://azure.microsoft.com/pricing/details/multi-factor-authentication/) pour plus d’informations concernant les licences et la tarification.

Voici les options et les avantages de la vérification en deux étapes :

**Option 1** : Activez l’authentification multifacteur pour tous les utilisateurs et les méthodes de connexion avec l’**avantage** offert par les paramètres de sécurité par défaut d’Azure AD : Cette option vous permet d’intégrer aisément et rapidement Azure MFA pour tous les utilisateurs de votre environnement à une stratégie rigoureuse :

* Contester les comptes d’administration et les mécanismes d’ouverture de session d’administration
* Exiger une stimulation MFA via Microsoft Authenticator pour tous les utilisateurs
* Restreindre les protocoles d’authentification hérités.

Cette méthode est disponible pour tous les niveaux de licence, mais elle ne peut pas être combinée à des stratégies d’accès conditionnel existantes. Vous trouverez plus d’informations dans la section [Valeurs par défaut de la sécurité Azure AD](../../active-directory/fundamentals/concept-fundamentals-security-defaults.md)

**Option 2** : [Activez Multi-Factor Authentication en modifiant l’état de l’utilisateur](../../active-directory/authentication/howto-mfa-userstates.md).   
**Avantage** : C’est la méthode traditionnelle pour exiger une vérification en deux étapes. Elle fonctionne avec [Azure AD Multi-Factor Authentication dans le cloud et le serveur Azure Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md). Cette méthode nécessite que les utilisateurs effectuent la vérification en deux étapes chaque fois qu’ils se connectent, puis remplace les stratégies d’accès conditionnel.

Pour déterminer où Multi-Factor Authentication doit être activé, consultez [Quelle version d’Azure AD MFA est adaptée à mon organisation ?](../../active-directory/authentication/concept-mfa-howitworks.md).

**Option 3** : [Activez Multi-Factor Authentication avec stratégie d’accès conditionnel](../../active-directory/authentication/howto-mfa-getstarted.md).
**Avantage** : Cette option permet de demander une vérification en deux étapes sous certaines conditions à l’aide de [l’accès conditionnel](../../active-directory/conditional-access/concept-conditional-access-policy-common.md). Les conditions spécifiques peuvent être une connexion de l’utilisateur à partir d’emplacements différents, d’appareils non approuvés ou d’applications que vous considérez comme risquées. Le fait de définir des conditions spécifiques pour une vérification en deux étapes vous permet d’éviter de la demander continuellement à vos utilisateurs, ce qui peut être désagréable.

Il s’agit de la méthode la plus souple pour activer la vérification en deux étapes pour vos utilisateurs. Activer une stratégie d’accès conditionnel fonctionne uniquement pour Azure AD Multi-Factor Authentication dans le cloud, et c’est une fonctionnalité Premium d’Azure AD. Vous pouvez trouver plus d’informations sur cette méthode dans [Déployer Azure AD Multi-Factor Authentication basé sur le cloud](../../active-directory/authentication/howto-mfa-getstarted.md).

**Option 4** : Activez Multi-Factor Authentication avec des stratégies d’accès conditionnel en évaluant des [stratégies d’accès conditionnel en fonction des risques](../../active-directory/conditional-access/howto-conditional-access-policy-risk.md).   
**Avantage** : Cette option permet de :

* Détecter des vulnérabilités potentielles qui affectent les identités de votre organisation.
* Configurer des réponses automatiques aux actions suspectes détectées qui sont liées aux identités de votre organisation.
* Examiner les incidents suspects et prendre les mesures appropriées pour les résoudre.

Cette méthode utilise l’évaluation de risques d’Azure AD Identity Protection pour déterminer si la vérification en deux étapes est exigée en se basant sur les risques de l’utilisateur et de la connexion pour toutes les applications cloud. Cette méthode requiert une licence Azure Active Directory P2. Vous trouverez plus d’informations sur cette méthode dans [Azure Active Directory Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md).

> [!Note]
> Option 2 : l’activation de Multi-Factor Authentication en changeant l’état de l’utilisateur remplace les stratégies d’accès conditionnel. Étant donné que les options 3 et 4 utilisent des stratégies d’accès conditionnel, vous ne pouvez pas utiliser l’option 2 avec elles.

Les organisations qui n’ajoutent pas de couche supplémentaire de protection d’identité, comme la vérification en deux étapes, sont plus sensibles au vol d’informations d’identification. Le vol d’informations d’identification peut entraîner une compromission des données.

## <a name="use-role-based-access-control"></a>Utiliser le contrôle d’accès en fonction du rôle

Il est vital pour toute organisation qui utilise le cloud de pouvoir gérer les accès aux ressources situées dans cloud. Le [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../role-based-access-control/overview.md) permet de gérer les utilisateurs ayant accès aux ressources Azure, les modes d’utilisation des ressources par ces derniers et les zones auxquelles ils ont accès.

Désigner des groupes ou des rôles individuels avec des fonctions spécifiques dans Azure aide à éviter une confusion pouvant entraîner des erreurs humaines et d’automatisation qui sont à l’origine de risques de sécurité. Restreindre l’accès en fonction des principes du [besoin de connaître](https://en.wikipedia.org/wiki/Need_to_know) et du [privilège minimum](https://en.wikipedia.org/wiki/Principle_of_least_privilege) est impératif pour les organisations désireuses d’appliquer des stratégies de sécurité pour l’accès aux données.

Votre équipe de sécurité a besoin d’une visibilité sur vos ressources Azure afin d’évaluer le risque et d’y remédier. Si l’équipe de sécurité a des responsabilités opérationnelles, elle a besoin d’autorisations supplémentaires pour faire son travail.

Vous pouvez utiliser la [fonction de contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../role-based-access-control/overview.md) pour affecter des autorisations à des utilisateurs, groupes et applications dans une certaine étendue. L’étendue d’une attribution de rôle peut être une seule ressource, un groupe de ressources ou un abonnement.

**Bonne pratique** : Séparez les tâches au sein de votre équipe et accorder aux utilisateurs uniquement les accès nécessaires pour accomplir leur travail. Plutôt que de donner à tous des autorisations illimitées au sein de votre abonnement ou de vos ressources Azure, autorisez uniquement certaines actions sur une étendue donnée.
**Détail** : Vous pouvez utiliser des [rôles intégrés Azure](../../role-based-access-control/built-in-roles.md) dans Azure pour attribuer des privilèges aux utilisateurs.

> [!Note]
> Des autorisations spécifiques créent une complexité et une confusion inutiles, qui s’accumulent dans une configuration « héritée » qui est difficile à corriger sans craindre de perturber quelque chose. Évitez les autorisations spécifiques aux ressources. Au lieu de cela, utilisez des groupes d’administration pour des autorisations au niveau de l’entreprise et des groupes de ressources pour des autorisations au sein d’abonnements. Évitez les autorisations spécifiques à des utilisateurs. Au lieu de cela, attribuez l’accès à des groupes dans Azure AD.

**Bonne pratique** : Accordez l’accès à des équipes de sécurité avec des responsabilités Azure pour voir les ressources Azure afin de pouvoir évaluer les risques et y remédier.
**Détail** : Accordez à des équipes de sécurité le rôle [Lecteur sécurité](../../role-based-access-control/built-in-roles.md#security-reader) du RBAC Azure. Vous pouvez utiliser le groupe d’administration racine ou le groupe d’administration de segment, selon l’étendue des responsabilités :

* **Groupe d’administration racine** pour les équipes responsables de toutes les ressources d’entreprise
* **Groupe d’administration de segment** pour les équipes avec une portée limitée (généralement en raison de limites organisationnelles réglementaires ou autres)

**Bonne pratique** : Accordez les autorisations appropriées aux équipes de sécurité ayant des responsabilités opérationnelles directes.
**Détail** : Passez en revue les rôles intégrés Azure pour l’attribution de rôle appropriée. Si les rôles intégrés ne répondent pas aux besoins de votre organisation, vous pouvez créer des [rôles personnalisés Azure](../../role-based-access-control/custom-roles.md). Comme avec les rôles intégrés, vous pouvez affecter des rôles personnalisés à des utilisateurs, des groupes et des principaux de service dans l’étendue des abonnements, des groupes de ressources et des ressources.

**Bonnes pratiques** : Permettez aux rôles de sécurité qui en ont besoin d’accéder à Azure Security Center. Security Center permet aux équipes de sécurité d’identifier rapidement les risques et d’y remédier.
**Détail** : Ajoutez des équipes de sécurité qui en ont besoin au rôle [Administrateur de sécurité du RBAC Azure](../../role-based-access-control/built-in-roles.md#security-admin), afin qu’elles puissent afficher des états de sécurité, modifier des stratégies de sécurité, afficher des alertes et des suggestions ainsi qu’ignorer les alertes et les suggestions. Pour ce faire, vous pouvez utiliser le groupe d’administration racine ou le groupe d’administration de segment, selon l’étendue des responsabilités.

Les organisations qui n’appliquent aucun contrôle d’accès aux données en utilisant des fonctionnalités telles que le RBAC Azure risquent d’octroyer plus de privilèges que nécessaire à leurs utilisateurs. Le fait d’autoriser des utilisateurs à accéder à des types de données (par exemple, des données HBI), auxquelles ils ne devraient pas avoir accès, peut conduire à la compromission de celles-ci.

## <a name="lower-exposure-of-privileged-accounts"></a>Exposition réduite des comptes privilégiés

La sécurisation de l’accès privilégié est une première étape essentielle pour protéger les ressources d’entreprise. Limiter le nombre de personnes qui ont accès aux informations ou aux ressources sécurisées réduit le risque qu’un utilisateur malveillant accède à ces données ou qu’une ressource sensible soit accidentellement affectée par un utilisateur autorisé.

Les comptes privilégiés sont ceux qui administrent et gèrent des systèmes informatiques. Les pirates informatiques ciblent ces comptes pour accéder aux données et aux systèmes d’une organisation. Pour sécuriser l’accès privilégié, vous devez isoler les comptes et les systèmes contre les risques d’exposition à un utilisateur malveillant.

Nous vous recommandons de créer et de suivre une feuille de route pour sécuriser l’accès privilégié contre les cybercriminels. Pour plus d’informations sur la création d’un programme détaillé pour sécuriser les identités et les accès gérés ou signalés dans Azure AD, Microsoft Azure, Microsoft 365 et d’autres services cloud, passez en revue l’article [Sécurisation de l’accès privilégié pour les déploiements hybrides et cloud dans Azure AD](../../active-directory/roles/security-planning.md).

Les éléments suivants résument les meilleures pratiques indiquées dans [Sécurisation de l’accès privilégié pour les déploiements hybrides et cloud dans Azure AD](../../active-directory/roles/security-planning.md) :

**Bonne pratique** : Gérez, contrôlez et effectuez le monitoring de l’accès aux comptes privilégiés.   
**Détail** : Activez [Azure AD Privileged Identity Management](../../active-directory/roles/security-planning.md). Après avoir activé Privileged Identity Management, vous recevez des notifications par courrier électronique de changements de rôles d’accès privilégié. Ces notifications vous informent lorsque des utilisateurs supplémentaires sont ajoutés aux rôles disposant de privilèges élevés dans votre annuaire.

**Bonne pratique** : Vérifiez que tous les comptes administrateur critiques sont des comptes Azure AD managés.
**Détail** : Supprimez les comptes de consommateurs des rôles d’administrateur critiques (par exemple, les comptes Microsoft tels que hotmail.com, live.com et outlook.com).

**Bonne pratique** : Vérifiez que tous les rôles d’administrateur critiques ont un compte distinct pour les tâches administratives, afin d’éviter que l’hameçonnage et autres attaques compromettent des privilèges Administrateur.
**Détail** : Créez un compte administratif séparé qui a attribué les privilèges nécessaires pour effectuer les tâches administratives. Bloquez l’utilisation de ces comptes d’administration pour les outils de productivité quotidiens tels que la messagerie électronique Microsoft 365 ou la navigation web arbitraire.

**Bonne pratique** : Identifiez et catégorisez les comptes présentant des rôles très privilégiés.   
**Détail** : Après avoir activé Azure AD Privileged Identity Management, vous voyez les utilisateurs Administrateur général, Administrateur à rôle privilégié et d’autres rôles très privilégiés. Supprimez les comptes qui ne sont plus nécessaires dans ces rôles et classez les autres comptes qui sont affectés à des rôles d’administrateur :

* Affectés individuellement à des utilisateurs administratifs, et pouvant servir à des fins non administratives (par exemple, messagerie personnelle)
* Affectés individuellement à des utilisateurs administratifs et dédiés à des fins administratives uniquement
* Partagés entre plusieurs utilisateurs
* Pour les scénarios d’accès d’urgence
* Pour les scripts automatisés
* Pour les utilisateurs externes

**Bonne pratique** : Implémentez l’accès juste-à-temps pour réduire encore le temps d’exposition des privilèges et augmenter votre visibilité sur l’utilisation des comptes privilégiés.   
**Détail** : Grâce à Azure AD Privileged Identity Management, vous pouvez :

* Limiter les utilisateurs à l’utilisation de leur accès Juste à temps privilégiés.
* Attribuer des rôles pour une durée plus courte en sachant que les privilèges sont automatiquement révoqués.

**Bonne pratique** : Définissez au moins deux comptes d’accès d’urgence.   
**Détail** : Les comptes d’accès d’urgence aident les organisations à restreindre l’accès privilégié dans un environnement Azure Active Directory existant. Ces comptes sont hautement privilégiés et ne sont pas affectés à des individus spécifiques. Les comptes d’accès d’urgence sont limités aux scénarios où il est impossible d’utiliser des comptes d’administration normaux. Les organisations doivent limiter l’utilisation des comptes d’urgence au temps strictement nécessaire.

Évaluez les comptes qui sont affectés ou éligibles pour le rôle d’administrateur général. Si vous ne voyez pas de comptes cloud uniquement à l’aide du domaine `*.onmicrosoft.com` (conçu pour l’accès d’urgence), créez-les. Pour plus d’informations, consultez [Managing emergency access administrative accounts in Azure AD](../../active-directory/roles/security-emergency-access.md) (Gestion des comptes d’administration de l’accès d’urgence dans Azure AD).

**Bonne pratique** : Prévoyez un processus « brise-vitres « en cas d’urgence.
**Détail** : Suivez les étapes mentionnées dans [Sécurisation de l’accès privilégié pour les déploiements hybrides et cloud dans Azure AD](../../active-directory/roles/security-planning.md).

**Bonne pratique** : Exigez que tous les comptes administrateur critiques soient exempts de mots de passe (recommandé) ou exigez l'authentification multifacteur.
**Détail** : Utilisez l’[application Microsoft Authenticator](../../active-directory/authentication/howto-authentication-passwordless-phone.md) pour vous connecter à n’importe quel compte Azure AD sans utiliser de mot de passe. Comme [Windows Hello Entreprise](/windows/security/identity-protection/hello-for-business/hello-identity-verification), Microsoft Authenticator a recours à l’authentification par clé pour activer des informations d’identification utilisateur qui sont liées à un appareil et utilisent une authentification biométrique ou un code confidentiel.

Exigez Azure AD Multi-Factor Authentication lors de la connexion de tous les utilisateurs individuels auxquels sont affectés un ou plusieurs rôles d’administrateur Azure AD : Administrateur général, administrateur de rôle privilégié, administrateur Exchange Online et administrateur SharePoint Online. Activez [l’authentification multifacteur pour vos comptes administrateur](../../active-directory/authentication/howto-mfa-userstates.md) et vérifiez que les utilisateurs de comptes administrateur sont inscrits.

**Bonne pratique** : Pour les comptes administrateur critiques, ayez une station de travail d’administrateur sur laquelle les tâches de production ne sont pas autorisées (par exemple, la navigation et les e-mails). Ceci protège vos comptes administrateur à partir de vecteurs d’attaque qui utilisent la navigation et la messagerie et réduisent considérablement les risques d’incident majeur.
**Détail** : Utilisez une station de travail administrateur. Choisissez un niveau pour la station de travail :

- Des appareils de productivité hautement sécurisé offrent une sécurité avancée pour la navigation et d’autres tâches de productivité.
- Les [stations de travail d’accès privilégié (PAW)](/windows-server/identity/securing-privileged-access/privileged-access-workstations) fournissent un système d’exploitation dédié qui est protégé contre les attaques Internet et les vecteurs de menaces.

**Bonne pratique** : Déprovisionnez les comptes administrateur quand un employé quitte votre organisation.
**Détail** : Mettez en place un processus qui désactive ou supprime les comptes administrateur quand un employé quitte votre organisation.

**Bonne pratique** : Testez régulièrement les comptes administrateur à l’aide de techniques d’attaque actuelles.
**Détail** : Utilisez le simulateur d’attaques Microsoft 365 ou une offre tierce pour exécuter des scénarios d’attaque réalistes dans votre organisation. Cela peut vous aider à trouver des utilisateurs vulnérables avant qu’une attaque réelle se produise.

**Bonne pratique** : Prenez des mesures pour atténuer les techniques d’attaque les plus fréquentes.  
**Détail** : [Identifier les comptes Microsoft ayant des rôles d’administrateur à basculer vers des comptes professionnels ou scolaires](../../active-directory/roles/security-planning.md#identify-microsoft-accounts-in-administrative-roles-that-need-to-be-switched-to-work-or-school-accounts)  

[Vérifier les comptes d’utilisateur distincts et le transfert de messagerie pour les comptes administrateur général](../../active-directory/roles/security-planning.md)  

[Vérifier que les mots de passe des comptes administrateur ont été récemment modifiés](../../active-directory/roles/security-planning.md#ensure-the-passwords-of-administrative-accounts-have-recently-changed)  

[Activer la synchronisation de hachage de mot de passe](../../active-directory/roles/security-planning.md#turn-on-password-hash-synchronization)  

[Exiger l’authentification multifacteur pour les utilisateurs dans tous les rôles privilégiés, ainsi que pour les utilisateurs exposés](../../active-directory/roles/security-planning.md#require-multi-factor-authentication-for-users-in-privileged-roles-and-exposed-users)  

[Obtenir votre Niveau de sécurité Microsoft 365 (si vous utilisez Microsoft 365)](../../active-directory/roles/security-planning.md#obtain-your-microsoft-365-secure-score-if-using-microsoft-365)  

[Passer en revue l’aide Microsoft 365 en matière de sécurité (si vous utilisez Microsoft 365)](../../active-directory/roles/security-planning.md#review-the-microsoft-365-security-and-compliance-guidance-if-using-microsoft-365)  

[Configurer la supervision de l’activité Microsoft 365 (si vous utilisez Microsoft 365)](../../active-directory/roles/security-planning.md#configure-microsoft-365-activity-monitoring-if-using-microsoft-365)  

[Définir des propriétaires de plan de réponse d’incident/d’urgence](../../active-directory/roles/security-planning.md#establish-incidentemergency-response-plan-owners)  

[Sécuriser les comptes d’administration privilégiés locaux](../../active-directory/roles/security-planning.md#turn-on-password-hash-synchronization)

Si vous ne sécurisez l’accès privilégié, vous constaterez peut-être qu’un trop grand nombre d’utilisateurs disposent de rôles hautement privilégiés et sont plus vulnérables aux attaques. Les personnes malveillantes, comme les pirates informatiques, ciblent souvent des comptes administrateur et d’autres moyens d’accès privilégié pour accéder à des données sensibles et à des systèmes à l’aide de vols d’informations d’identification.

## <a name="control-locations-where-resources-are-created"></a>Contrôle des emplacements où les ressources sont créées

Le fait de permettre aux opérateurs de cloud d’effectuer des tâches tout en les empêchant de briser les conventions qui sont nécessaires à la gestion des ressources de votre organisation est très important. Les organisations qui veulent contrôler les emplacements où les ressources sont créées doivent coder en dur ces emplacements.

Vous pouvez utiliser [Azure Resource Manager](../../azure-resource-manager/management/overview.md) pour créer des stratégies de sécurité dotées de définitions décrivant les actions ou les ressources spécifiquement refusées. Vous affectez ces définitions de stratégies selon l’étendue souhaitée, au niveau de l’abonnement, du groupe de ressources ou d’une ressource individuelle.

> [!NOTE]
> Les stratégies de sécurité ne sont pas identiques au contrôle d’accès en fonction du rôle Azure (RBAC Azure). En fait, elles utilisent le RBAC Azure pour autoriser des utilisateurs à créer ces ressources.
>
>

Les organisations qui ne contrôlent pas la création des ressources sont plus sensibles aux utilisateurs susceptibles d’abuser du service en créant plus de ressources que nécessaire. Le renforcement du processus de création des ressources est une étape importante de sécurisation d’un scénario à plusieurs locataires.

## <a name="actively-monitor-for-suspicious-activities"></a>Surveillance active des activités suspectes

Un système de surveillance d’identité actif peut détecter rapidement un comportement suspect et déclencher une alerte pour un examen approfondi. Le tableau suivant répertorie les deux fonctionnalités Azure AD pouvant aider les organisations à surveiller leurs identités :

**Bonne pratique** : Définissez une méthode pour identifier :

- Les tentatives de connexion [sans être suivi](../../active-directory/reports-monitoring/howto-find-activity-reports.md).
- Les attaques par [force brute](../../active-directory/reports-monitoring/howto-find-activity-reports.md) contre un compte particulier.
- Les tentatives de connexion à partir de plusieurs emplacements.
- Les connexions depuis des [appareils infectés](../../active-directory/reports-monitoring/howto-find-activity-reports.md).
- Les adresses IP suspectes.

**Détail** : Utilisez les [rapports d’anomalies](../../active-directory/reports-monitoring/overview-reports.md) d’Azure AD Premium. Disposer de processus et de procédures permettant aux administrateurs informatiques d’exécuter ces rapports de manière quotidienne ou à la demande (généralement dans un scénario de réponse aux incidents).

**Bonne pratique** : Disposez d’un système de monitoring actif qui vous informe des risques et adapte le niveau de risque (élevé, moyen ou faible) à vos besoins métier.   
**Détail** : Utilisez [Azure AD Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md), qui marque les risques actuels sur son propre tableau de bord et envoie des notifications de synthèse quotidiennes par e-mail. Pour aider à protéger les identités de votre organisation, vous pouvez configurer des stratégies qui répondent automatiquement aux problèmes détectés lorsqu’un niveau de risque spécifié est atteint.

Les organisations qui ne surveillent pas activement leurs systèmes d’identité risquent de compromettre les informations d’identification des utilisateurs. Si elles n’ont pas connaissance des activités suspectes se déroulant avec ces informations d’identification, elles ne sont pas en mesure de limiter ce type de menace.

## <a name="use-azure-ad-for-storage-authentication"></a>Utiliser Azure AD pour l’authentification du stockage
Le [Stockage Azure](../../storage/common/storage-auth-aad.md) prend en charge l’authentification et l’autorisation avec Azure AD pour le stockage d’objets blob et le stockage File d’attente. Avec l’authentification Azure AD, vous pouvez utiliser le contrôle d’accès basé sur les rôles Azure pour accorder des autorisations spécifiques aux utilisateurs, aux groupes et aux applications jusqu’au niveau d’un conteneur d’objets blob individuel ou de la file d’attente.

Nous vous recommandons d’utiliser [Azure AD pour authentifier l’accès au stockage](https://azure.microsoft.com/blog/azure-storage-support-for-azure-ad-based-access-control-now-generally-available/).

## <a name="next-step"></a>Étape suivante

Consultez l’article [Bonnes pratiques et tendances Azure relatives à la sécurité](best-practices-and-patterns.md) pour découvrir d’autres bonnes pratiques en matière de sécurité à appliquer dans le cadre de la conception, du déploiement et de la gestion de vos solutions cloud avec Azure.