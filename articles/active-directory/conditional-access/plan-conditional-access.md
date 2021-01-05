---
title: Planifier un déploiement d’accès conditionnel Azure Active Directory
description: Apprenez à concevoir des stratégies d’accès conditionnel et à les déployer efficacement au sein de votre organisation.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 10/16/2020
ms.author: baselden
author: BarbaraSelden
manager: daveba
ms.reviewer: joflore
ms.collection: M365-identity-device-management
ms.openlocfilehash: 20b51cc747d3a24b1437eda988397a2e999f6ab3
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94837479"
---
# <a name="plan-a-conditional-access-deployment"></a>Planifier un déploiement d’accès conditionnel

La planification de votre déploiement d’accès conditionnel joue un rôle capital dans la réussite de la stratégie d’accès de votre organisation pour les applications et les ressources.

Dans un monde où la mobilité et le cloud sont la priorité, vos utilisateurs accèdent, en tous lieux, aux ressources de votre organisation depuis un large éventail d’appareils et d’applications. Ainsi, s’attacher exclusivement au contrôle des personnes accédant à une ressource ne suffit plus. Vous devez également tenir compte de l’endroit où l’utilisateur se trouve, de l’appareil qui est utilisé, de la ressource à laquelle il accède, et bien plus encore. 

Les accès conditionnels Azure Active Directory (Azure AD) analysent les signaux, comme l’utilisateur, l’appareil et l’emplacement, afin d’automatiser les décisions et d’appliquer les stratégies d’accès propres à l’organisation pour les ressources. Vous pouvez utiliser des stratégies d’accès conditionnel pour appliquer des contrôles d’accès tels que Multi-Factor Authentication (MFA). Les stratégies d’accès conditionnel vous permettent d’inviter les utilisateurs à procéder à l’authentification MFA lorsque la sécurité l’exige, et à les laisser libres de naviguer à leur guise lorsqu’elle n’est pas nécessaire.

![Présentation de l’accès conditionnel](./media/plan-conditional-access/conditional-access-overview-how-it-works.png)

Microsoft fournit des stratégies conditionnelles standard, appelées [paramètres de sécurité par défaut](../fundamentals/concept-fundamentals-security-defaults.md), et qui garantissent un niveau de sécurité de base. Toutefois, votre organisation peut avoir besoin de plus de flexibilité que ce que propose la sécurité par défaut. Vous pouvez utiliser l’accès conditionnel pour personnaliser les paramètres de sécurité par défaut avec une plus grande granularité, et pour configurer de nouvelles stratégies qui répondent à vos besoins.

## <a name="learn"></a>Découvrir

Avant de commencer, vous devez bien comprendre le fonctionnement de l’[accès conditionnel](overview.md) et quand vous devez l’utiliser.

### <a name="benefits"></a>Avantages

Le déploiement de l’accès conditionnel présente les avantages suivants :

* Meilleure productivité. N’interrompt que les utilisateurs dotés d’une condition de connexion, telle que MFA, lorsqu’un ou plusieurs signaux le justifient. Les stratégies d’accès conditionnel vous permettent de contrôler le moment auquel les utilisateurs sont invités à utiliser l’authentification multifacteur, quand l’accès est bloqué et quand les utilisateurs doivent utiliser un appareil de confiance.

* Gérer le risque. Automatiser l’évaluation des risques avec des conditions de stratégie signifie que les connexions risquées sont immédiatement identifiées et corrigées ou bloquées. Le couplage de l’accès conditionnel à [Identity Protection](../identity-protection/overview-identity-protection.md), qui détecte les anomalies et les événements suspects, vous permet de cibler le moment où l’accès aux ressources est bloqué ou contrôlé. 

* Gouvernance et conformité des adresses. L’accès conditionnel vous permet d’auditer l’accès aux applications, de présenter les conditions d’utilisation pour le consentement et de restreindre l’accès en fonction des stratégies de conformité.

* Gestion des coûts. Le déplacement des stratégies d’accès sur Azure AD réduit le recours à des solutions locales ou personnalisées pour l’accès conditionnel, et leurs coûts d’infrastructure.

### <a name="license-requirements"></a>Conditions de licence :

Consultez [Conditions de licence de l’accès conditionnel](overview.md).

Si des fonctionnalités supplémentaires sont nécessaires, il est possible que vous ayez également besoin des licences associées. Pour plus d’informations, consultez [Tarification Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

### <a name="prerequisites"></a>Prérequis

* Un locataire Azure AD actif avec une licence Azure AD Premium activée ou une licence d’évaluation activée. Si nécessaire, [créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Un compte doté des privilèges d’administrateur de l’accès conditionnel.

* Un utilisateur non-administrateur avec un mot de passe que vous connaissez, par exemple testuser. Si vous devez créer un utilisateur, consultez [Démarrage rapide : Ajouter de nouveaux utilisateurs à Azure Active Directory](../fundamentals/add-users-azure-active-directory.md).

* Un groupe dont l’utilisateur non-administrateur est membre. Si vous devez créer un groupe, consultez [Créer un groupe et ajouter des membres dans Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md).

### <a name="training-resources"></a>Ressources de formation

Les ressources suivantes peuvent être utiles lorsque vous vous familiarisez avec l’accès conditionnel :


#### <a name="videos"></a>Vidéos

* [Qu’est-ce que l’accès conditionnel ?](https://youtu.be/ffMAw2IVO7A)
* [Comment déployer l’accès conditionnel](https://youtu.be/c_izIRNJNuk)
* [Comment déployer des stratégies d’accès conditionnel pour les utilisateurs finaux](https://youtu.be/0_Fze7Zpyvc)
* [L’accès conditionnel et les contrôles d’appareil](https://youtu.be/NcONUf-jeS4)
* [Accès conditionnel avec Azure AD MFA](https://youtu.be/Tbc-SU97G-w)
* [Conditional Access in Enterprise Mobility + Security](https://youtu.be/A7IrxAH87wc) (Accès conditionnel dans Enterprise Mobility + Security)


#### <a name="online-courses-on-pluralsight"></a>Cours en ligne sur Pluralsight

* [Design Identity Management in Microsoft Azure](https://www.pluralsight.com/courses/microsoft-azure-identity-management-design) (Conception de la gestion des identités dans Microsoft Azure)
* [Design Authentication for Microsoft Azure](https://www.pluralsight.com/courses/microsoft-azure-authentication-design) (Conception de l’authentification pour Microsoft Azure)
* [Design Authorization for Microsoft Azure](https://www.pluralsight.com/courses/microsoft-azure-authorization-design) (Conception des autorisations pour Microsoft Azure)

## <a name="plan-the-deployment-project"></a>Planifier le projet de déploiement

Tenez compte des besoins de votre organisation lorsque vous déterminez la stratégie de ce déploiement dans votre environnement.

### <a name="engage-the-right-stakeholders"></a>Impliquer les parties prenantes appropriées

Lorsque des projets technologiques échouent, cela est généralement dû à des attentes qui ne correspondent pas à l’impact, aux résultats et aux responsabilités réels. Pour éviter un tel cas de figure, [prenez soin de faire appel aux bonnes parties prenantes](../fundamentals/active-directory-deployment-plans.md) et à clarifier les rôles du projet.

### <a name="plan-communications"></a>Planifier les communications

La communication est essentielle à la réussite de tout nouveau service. Communiquez de manière proactive avec vos utilisateurs sur ce qui va changer, à quel moment les changements seront appliqués et comment ils peuvent obtenir de l’aide en cas de problème.

### <a name="plan-a-pilot"></a>Prévoir un pilote

Lorsque de nouvelles stratégies sont prêtes pour votre environnement, déployez-les en phases dans l’environnement de production. Tout d’abord, appliquez une stratégie à un petit ensemble d’utilisateurs dans un environnement de test et vérifiez si la stratégie se comporte comme prévu. Consultez [Meilleures pratiques pour un pilote](../fundamentals/active-directory-deployment-plans.md).

> [!NOTE]
> Pour déployer de nouvelles stratégies qui ne sont pas spécifiques aux administrateurs, excluez tous les administrateurs. De cette façon, vous garantissez toujours l’accès des administrateurs à la stratégie, à laquelle ils apportent des modifications s’ils ne la révoquent pas en cas d’impact significatif. Validez toujours la stratégie avec des groupes d’utilisateurs plus petits avant de l’appliquer à tous les utilisateurs.

## <a name="understand-ca-policy-components"></a>Comprendre les composants de la stratégie d’accès conditionnel
Les stratégies d’accès conditionnel sont des instructions de type si-alors : si une affectation est remplie, alors appliquer ces contrôles d’accès.

Lors de la configuration de stratégies d’accès conditionnel, les conditions s’appellent des *affectations*. Les stratégies d’accès conditionnel vous permettent d’appliquer des contrôles d’accès aux applications de votre organisation, en fonction de certaines affectations.


Pour plus d’informations, consultez [Création d’une stratégie d’accès conditionnel](concept-conditional-access-policies.md).

![écran de création d’une stratégie](media/plan-conditional-access/create-policy.png)

Les [affectations](concept-conditional-access-policies.md#assignments) définissent les

* [utilisateurs et groupes](concept-conditional-access-users-groups.md) que la stratégie doit affecter ;

* [applications cloud ou actions](concept-conditional-access-cloud-apps.md) auxquelles la stratégie s’appliquera ; 

* [conditions](concept-conditional-access-conditions.md) en vertu desquelles la stratégie s’appliquera.

Les paramètres des [contrôles d’accès](concept-conditional-access-policies.md) déterminent la façon dont appliquer une stratégie :

* [Accorder ou bloquer l’accès](concept-conditional-access-grant.md) aux applications cloud.

* Les [contrôles de session](concept-conditional-access-session.md) permettent de limiter les expériences dans des applications cloud particulières.

### <a name="ask-the-right-questions-to-build-your-policies"></a>Poser les bonnes questions pour créer vos stratégies

Les stratégies répondent à des questions posées sur les personnes qui ont le droit d’accéder à vos ressources, et sur les ressources auxquelles elles peuvent accéder et dans quelles conditions elles doivent le faire. Les stratégies peuvent être conçues pour accorder l’accès, ou le bloquer. Assurez-vous de poser les bonnes questions par rapport aux objectifs poursuivis par votre stratégie. 

Documentez les réponses aux questions pour chaque stratégie avant de la créer. 

#### <a name="common-questions-about-assignments"></a>Questions courantes sur les affectations

[Utilisateurs et groupes](concept-conditional-access-users-groups.md)

* Quels utilisateurs et groupes seront inclus ou exclus de la stratégie ?

* Cette stratégie peut-elle inclure tous les utilisateurs, groupes d’utilisateurs particuliers, rôles d’annuaire ou utilisateurs externes ?

[Applications cloud ou actions](concept-conditional-access-cloud-apps.md)

* À quelle(s) application(s) la stratégie s’appliquera-t-elle ?

* Quelles actions de l’utilisateur seront soumises à cette stratégie ?

[Conditions](concept-conditional-access-conditions.md)

* Quelles plateformes d’appareils seront incluses ou exclues de la stratégie ?

* Quels sont les emplacements approuvés de l’organisation ?

* Quels emplacements seront inclus ou exclus de la stratégie ?

* Quels types d’applications clientes (navigateur, mobile, clients de bureau, applications avec méthodes d’authentification héritées) seront inclus ou exclus de la stratégie ?

* Comptez-vous des stratégies qui pourraient aboutir à exclure des appareils joints Azure AD ou des appareils Azure AD Hybride de stratégies ? 

* Si vous utilisez [Identity Protection](../identity-protection/concept-identity-protection-risks.md), voulez-vous incorporer la protection contre la connexion à risque ?

#### <a name="common-questions-about-access-controls"></a>Questions courantes sur les contrôles d’accès

[Octroyer ou bloquer](concept-conditional-access-grant.md) 

Voulez-vous accorder l’accès aux ressources en exigeant un ou plusieurs des éléments suivants ?

* Exiger une authentification multifacteur

* Exiger que l’appareil soit marqué comme conforme

* Exiger un appareil joint à Azure AD hybride

* Demander une application cliente approuvée

* Exiger une stratégie de protection des applications

[Contrôle de session](concept-conditional-access-session.md)

Voulez-vous appliquer les contrôles d’accès suivants sur les applications cloud ?

* Utiliser les autorisations appliquées par l’application

* Utiliser le contrôle d'application par accès conditionnel

* Appliquer la fréquence de connexion

* Utiliser les sessions de navigateur persistantes

### <a name="access-token-issuance"></a>Émission de jetons d’accès

Il est important de comprendre comment les jetons d’accès sont émis. 

![Diagramme d’émission du jeton d’accès](media/plan-conditional-access/CA-policy-token-issuance.png)

> [!NOTE]
> Si aucune affectation n’est exigée, et qu’aucune stratégie d’accès conditionnel n’est appliquée, le comportement par défaut consiste à émettre un jeton d’accès. 

Par exemple, imaginons une stratégie dans laquelle :

SI l’utilisateur est dans le groupe 1, ALORS forcer MFA sur l’accès à l’application 1.

Si un utilisateur qui n’est pas dans le groupe 1 tente d’accéder à l’application, aucune condition « If » n’est remplie et un jeton est émis. L’exclusion d’utilisateurs hors du groupe 1 nécessite une stratégie distincte qui permet de bloquer tous les autres utilisateurs.

## <a name="follow-best-practices"></a>Suivre les bonnes pratiques

L’infrastructure d’accès conditionnel vous offre une souplesse de configuration exceptionnelle. Toutefois, une grande flexibilité implique également que vous examiniez soigneusement chaque stratégie de configuration avant de la mettre en œuvre, afin d’éviter des résultats indésirables.

### <a name="apply-ca-policies-to-every-app"></a>Appliquer des stratégies d’accès conditionnel à chaque application

Les jetons d’accès sont émis par défaut si une condition de stratégie d’accès conditionnel ne déclenche pas de contrôle d’accès. Assurez-vous que chaque application compte au moins une stratégie d’accès conditionnel appliquée

> [!IMPORTANT]
> Soyez très prudent lors de l’utilisation des éléments Bloquer et Toutes les applications dans une stratégie unique. Les administrateurs du portail d’administration Azure peuvent se retrouver bloqués à l’extérieur du portail, et les exclusions ne pas être configurées pour des points de terminaison importants, tels que Microsoft Graph.

### <a name="minimize-the-number-of-ca-policies"></a>Réduire le nombre de stratégies d’accès conditionnel

Créer une stratégie pour chaque application n’est pas avantageux et débouche sur une administration compliqué. L’accès conditionnel n’appliquera que les 195 premières stratégies par utilisateur. Nous vous recommandons d’analyser vos applications et de les regrouper par applications partageant les mêmes exigences en ressources pour les mêmes utilisateurs. Par exemple, si toutes les applications Microsoft 365 ou de RH présentent les mêmes exigences pour les mêmes utilisateurs, créez une stratégie unique et incluez toutes les applications auxquelles elle s’applique. 

### <a name="set-up-emergency-access-accounts"></a>Configurer des comptes d’accès d’urgence

Si votre stratégie est mal configurée, elle peut verrouiller les organisations à l’extérieur du portail Azure. Vous pouvez pallier l’impact du verrouillage accidentel d’administrateurs en créant quelques [comptes d’accès d’urgence](../roles/security-emergency-access.md) dans votre organisation.

* Créez un compte d’utilisateur dédié à l’administration de stratégies, et qui est exclu de toutes vos stratégies.

* Scénario de secours des environnements hybrides :

  * Créez un groupe de sécurité local et synchronisez-le à Azure AD. Le groupe de sécurité doit contenir votre compte dédié à l’administration de stratégies. 

   * Faites EXEMPTER ce groupe de sécurité de toutes les stratégies d’accès conditionnel.

   * En cas de panne d’un service, ajoutez vos autres administrateurs au groupe local en fonction des besoins, et forcez une synchronisation. Leur exemption est ainsi animée sur les stratégies d’accès conditionnel.

### <a name="set-up-report-only-mode"></a>Configurer le mode Rapport seul

Il peut ne pas être aisé de prévoir le nombre et les noms des utilisateurs concernés par des initiatives de déploiement courantes, comme :

* le blocage de l’authentification héritée ;
* l’obligation d’utiliser MFA ;
* l’implémentation de stratégies de connexion à risque.

[Le mode rapport seul](concept-conditional-access-report-only.md) permet aux administrateurs d’évaluer l’impact des stratégies d’accès conditionnel avant de les activer dans leur environnement.

Apprenez à [Configurer le mode rapport seul sur une stratégie d’accès conditionnel](howto-conditional-access-insights-reporting.md).

### <a name="plan-for-disruption"></a>Planifier une interruption

Si vous vous appuyez sur un seul contrôle d’accès, comme MFA ou un emplacement réseau, pour sécuriser vos systèmes informatiques, vous êtes susceptible de rencontrer des problèmes d’accès si ce contrôle d’accès unique est indisponible ou mal configuré. Pour réduire le risque de verrouillage pendant des interruptions imprévues, [planifiez des stratégies](../authentication/concept-resilient-controls.md) à adopter pour votre organisation.

### <a name="set-naming-standards-for-your-policies"></a>Définir des normes de nommage pour vos stratégies

La convention de nommage vous permet de rechercher des stratégies et de comprendre à quoi elles servent sans les ouvrir dans le portail d’administration Azure. Nous vous recommandons de nommer votre stratégie pour faire ressortir :

* Un numéro de séquence

* L’application ou les applications cloud auxquelles elle s’applique

* La réponse

* L’objet auquel elle s’applique (qui)

* Quand elle s’applique (le cas échéant)

![Capture d’écran montrant les normes d’attribution de noms pour les stratégies.](media/plan-conditional-access/11.png)

**Exemple** Une stratégie qui exige MFA pour les utilisateurs Marketing accédant à l’application Dynamics CRP depuis les réseaux externes peut être formulée de la façon suivante :

![Convention de nommage](media/plan-conditional-access/naming-example.png)

Un nom descriptif vous aide à conserver une vue globale de votre implémentation de l’accès conditionnel. Le numéro de séquence est utile si vous devez faire référence à une stratégie dans une conversation. Par exemple, si vous parlez à un administrateur au téléphone, vous pouvez lui demander d’ouvrir la stratégie CA01 pour résoudre un problème.

#### <a name="naming-standards-for-emergency-access-controls"></a>Normes de nommage pour les contrôles d’accès d’urgence

En plus de vos stratégies actives, implémentez des stratégies désactivées qui agissent comme des [contrôles d’accès résilients secondaires dans les scénarios d’urgence ou de panne](../authentication/concept-resilient-controls.md). Votre norme de nommage pour les stratégies d’urgence doit inclure quelques éléments supplémentaires :
* ACTIVER EN CAS D’URGENCE au début, pour faire ressortir le nom au milieu des autres stratégies.

* Le nom d’interruption auquel elle doit s’appliquer.

* Un numéro de séquence de classement pour aider l’administrateur à savoir dans quel ordre les stratégies doivent être activées.

**Exemple**

Le nom suivant indique que cette stratégie est la première d’une série de quatre stratégies à activer en cas d’interruption de l’authentification multifacteur :

EM01 - ACTIVER EN CAS D’URGENCE : Interruption MFA [1/4] - Exchange SharePoint : Exiger la jonction Azure AD Hybride pour les utilisateurs VIP.

### <a name="exclude-countries-from-which-you-never-expect-a-sign-in"></a>Exclure les pays depuis lesquels vous n’espérez jamais aucune connexion.

Azure Active Directory vous permet de créer des [emplacements nommés](location-condition.md). Créez un emplacement nommé qui comprend tous les pays à partir desquels vous n’escomptez jamais qu’une connexion se produise. Créez ensuite une stratégie pour Toutes les applications qui bloquent la connexion à partir de cet emplacement nommé. **Veillez à exempter vos administrateurs de cette stratégie**.

### <a name="plan-your-policy-deployment"></a>Planifier votre déploiement de stratégies

Lorsque de nouvelles stratégies sont prêtes pour votre environnement, prenez soin de passer en revue chacune d’elle avant sa publication pour éviter des résultats indésirables.

## <a name="common-policies"></a>Stratégies courantes

Quand vous planifiez votre solution de stratégie d’accès conditionnel, déterminez si vous devez créer des stratégies pour obtenir les résultats suivants.

* [Comment exiger l’authentification MFA](#require-mfa)
* [Répondre aux comptes potentiellement compromis](#respond-to-potentially-compromised-accounts)
* [Exiger des appareils gérés](#require-managed-devices)
* [Exiger des applications clientes approuvées](#require-approved-client-apps)
* [Bloquer l’accès](#block-access)

### <a name="require-mfa"></a>Exiger une authentification multifacteur

Les cas d’utilisation courants pour lesquels exiger l’accès MFA :

* [Par les administrateurs](howto-conditional-access-policy-admin-mfa.md)

* [À des applications spécifiques](../authentication/tutorial-enable-azure-mfa.md)

* [Pour tous les utilisateurs](howto-conditional-access-policy-all-users-mfa.md)

* [À partir d’emplacements réseau non fiables](untrusted-networks.md)

* [ Pour la gestion Azure](howto-conditional-access-policy-azure-management.md)

### <a name="respond-to-potentially-compromised-accounts"></a>Répondre aux comptes potentiellement compromis

Avec des stratégies d’accès conditionnel, vous pouvez implémenter des réponses automatiques aux connexions par identités potentiellement compromises. La probabilité qu’un compte soit compromis est exprimée sous forme de niveaux de risque. Il existe deux niveaux de risque calculés par Identity Protection : la connexion à risque et l’utilisateur à risque. Les trois stratégies par défaut suivantes peuvent être activées.

* [Demander à tous les utilisateurs de s’inscrire pour l’authentification multifacteur](howto-conditional-access-policy-risk.md)

* [Exiger un changement de mot de passe pour les utilisateurs à risque élevé](howto-conditional-access-policy-risk.md)

* [Exiger l’authentification multifacteur pour les utilisateurs dont la connexion est à risque moyen ou élevé](howto-conditional-access-policy-risk.md)

### <a name="require-managed-devices"></a>Exiger des appareils gérés

L’augmentation du nombre d’appareils pris en charge pour accéder aux ressources cloud permet d’améliorer la productivité de vos utilisateurs. Il est probable que vous ne souhaitiez pas que des appareils dont le niveau de protection est inconnu puissent accéder à certaines ressources de votre environnement. Pour ces ressources, [exigez que les utilisateurs y accèdent uniquement au moyen d’un appareil géré](require-managed-devices.md).

### <a name="require-approved-client-apps"></a>Exiger des applications client approuvées

Les employés utilisent leurs appareils mobiles pour des tâches à la fois personnelles et professionnelles. Pour les scénarios BYOD, vous devez décider si vous voulez gérer l’appareil entièrement, ou seules les données qu’il contient. Si vous gérez uniquement les données et les accès, vous pouvez [exiger des applications cloud approuvées](app-based-conditional-access.md) qui peuvent protéger vos données d’entreprise. Par exemple, vous pouvez imposer l’accès à la messagerie électronique uniquement via Outlook Mobile, et non par le biais d’un programme de messagerie générique.

### <a name="block-access"></a>Bloquer l’accès

L’option permettant de [bloquer tous les accès](howto-conditional-access-policy-block-access.md) est puissante. Elle peut être utilisée, par exemple, lorsque vous migrez une application vers Azure AD, alors que vous n’êtes pas prêt à ce que tout le monde s’y connecte pour le moment. Bloquer l’accès : 

* Remplace toutes les autres affectations d’un utilisateur

* A la possibilité d’empêcher l’ensemble de votre organisation de se connecter à votre locataire

> [!IMPORTANT]
> Si vous créez une stratégie pour bloquer l’accès de tous les utilisateurs, veillez à exclure de la stratégie les comptes d’accès d’urgence, et éventuellement tous les administrateurs.

Voici d’autres scénarios courants dans lesquels vous pouvez bloquer l’accès à vos utilisateurs :

* [Bloquer à certains emplacements réseau](howto-conditional-access-policy-location.md) l’accès à vos applications cloud. Vous pouvez utiliser cette stratégie pour bloquer certains pays à partir desquels vous savez que le trafic ne se fera pas.

* Azure AD prend en charge l’authentification hérité. Toutefois, l’authentification héritée ne prend pas en charge MFA alors que de nombreux environnements l’exigent pour assurer la sécurité des identités. Dans ce cas, vous pouvez [empêcher des applications utilisant l’authentification héritée](block-legacy-authentication.md) d’accéder aux ressources de votre locataire.

## <a name="build-and-test-policies"></a>Générer et tester les stratégies

À chaque étape de votre déploiement, assurez-vous que les évaluations effectuées donnent les résultats attendus. 

Lorsque de nouvelles stratégies sont prêtes, déployez-les en phases dans l’environnement de production :

* Indiquez aux utilisateurs finaux les changements internes.

* Commencez par un petit ensemble d’utilisateurs et vérifiez que la stratégie se comporte comme prévu.

* Quand vous étendez une stratégie à davantage d’utilisateurs, continuez à exclure tous les administrateurs. De cette façon, au moins une personne a accès à la stratégie si un changement est nécessaire.

* N’appliquez une stratégie à tous les utilisateurs qu’après l’avoir testée minutieusement. Vérifiez que vous disposez au moins d’un compte administrateur auquel une stratégie ne s’applique pas.

### <a name="create-test-users"></a>Créer des utilisateurs de test

Créez un ensemble d’utilisateurs de test qui reflète les utilisateurs de votre environnement de production. La création d’utilisateurs de test vous permet de vérifier que vos stratégies fonctionnent comme prévu avant d’impacter les utilisateurs réels, et de risquer d’interrompre leur accès aux applications et aux ressources.

Certaines organisations ont des locataires de test dans ce but. Toutefois, il peut être difficile de recréer toutes les conditions et les applications dans un locataire de test pour tester intégralement le résultat d’une stratégie.

### <a name="create-a-test-plan"></a>Créer un plan de test

Le plan de test est important pour comparer les résultats attendus et les résultats réels. Vous devez toujours avoir un objectif avant de tester quelque chose. Le tableau suivant décrit des exemples de cas de test. Ajustez les scénarios et les résultats attendus en fonction de la configuration de vos stratégies d’accès conditionnel.

| Policy| Scénario| Résultat attendu |
| - | - | - |
| [Exiger l’authentification multifacteur en dehors du bureau](untrusted-networks.md)| L’utilisateur autorisé se connecte à l’application quand il est dans un emplacement approuvé / au bureau| L’utilisateur n’est pas invité à utiliser l’authentification multifacteur |
| [Exiger l’authentification multifacteur en dehors du bureau](untrusted-networks.md)| L’utilisateur autorisé se connecte à l’application quand il n’est pas dans un emplacement approuvé / au bureau| L’utilisateur est invité à utiliser l’authentification multifacteur et peut se connecter |
| [Exiger l’authentification multifacteur (pour les administrateurs)](../fundamentals/concept-fundamentals-security-defaults.md)| L’administrateur général se connecte à l’application| L’administrateur est invité à utiliser l’authentification multifacteur |
| [Connexions risquées](../identity-protection/howto-identity-protection-configure-risk-policies.md)| L’utilisateur se connecte à l’application à l’aide d’un navigateur non approuvé| L’administrateur est invité à utiliser l’authentification multifacteur |
| [Gestion des appareils](require-managed-devices.md)| L’utilisateur autorisé tente de se connecter à partir d’un appareil autorisé| Accès accordé |
| [Gestion des appareils](require-managed-devices.md)| L’utilisateur autorisé tente de se connecter à partir d’un appareil non autorisé| Accès bloqué |
| [Changement de mot de passe pour les utilisateurs à risque](../identity-protection/howto-identity-protection-configure-risk-policies.md)| L’utilisateur autorisé tente de se connecter avec des informations d’identification compromises (connexion à haut risque)| L’utilisateur est invité à changer le mot de passe ou l’accès est bloqué selon votre stratégie |


### <a name="configure-the-test-policy"></a>Configurer la stratégie de test

Dans le [portail Azure](https://portal.azure.com/), vous configurez des stratégies d’accès conditionnel sous Azure Active Directory > Sécurité > Accès conditionnel.

Si vous souhaitez en savoir plus sur la façon de créer des stratégies d’accès conditionnel, consultez cet exemple : [Stratégie d’accès conditionnel pour demander l’authentification MFA lorsqu’un utilisateur se connecte au portail Azure](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json). Ce guide de démarrage rapide vous permet de :

* Vous familiariser avec l’interface utilisateur

* Obtenir une première impression du fonctionnement de l’accès conditionnel

### <a name="enable-the-policy-in-report-only-mode"></a>Activer la stratégie en mode rapport seul

Pour évaluer l’impact de votre stratégie, commencez par activer la stratégie en [mode rapport seul](concept-conditional-access-report-only.md). Les stratégies de rapport seul sont évaluées lors de la connexion, mais les contrôles d’octroi et de session ne sont pas appliqués. Une fois que vous avez enregistré la stratégie en mode rapport seul, vous pouvez voir l’impact sur les connexions en temps réel dans les journaux de connexion. Dans les journaux de connexion, sélectionnez un événement et accédez à l’onglet Rapport seul pour afficher le résultat de chaque stratégie de rapport seul.


![mode rapport seul ](media/plan-conditional-access/report-only-mode.png)

En sélectionnant la stratégie, vous pouvez également savoir comment les affectations et les contrôles d’accès de cette stratégie ont été évalués à l’aide de l’écran des détails de la stratégie. Pour qu’une stratégie s’applique à une connexion, chaque affectation configurée doit être satisfaite. 

### <a name="understand-the-impact-of-your-policies-using-the-insights-and-reporting-workbook"></a>Comprendre l’impact de vos stratégies à l’aide du classeur Insights et rapports

Vous pouvez afficher l’impact agrégé de vos stratégies d’accès conditionnel dans le classeur Insights et rapports. Pour accéder au classeur, vous devez disposer d’un abonnement Azure Monitor et [envoyer en streaming vos journaux de connexion à un espace de travail Log Analytics](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md). 

### <a name="simulate-sign-ins-using-the-what-if-tool"></a>Simuler des connexions à l’aide de l’outil de simulation

Une autre façon de valider votre stratégie d’accès conditionnel consiste à utiliser l’[outil de simulation](troubleshoot-conditional-access-what-if.md) qui reproduit les stratégies pouvant s’appliquer à un utilisateur se connectant dans une situation hypothétique. Sélectionnez les attributs de connexion que vous souhaitez tester (par exemple, utilisateur, application, plateforme d’appareil et emplacement) et voyez quelles stratégies pourraient s’appliquer.

> [!NOTE] 
> Bien qu’une exécution simulée vous donne une bonne idée de l’impact d’une stratégie d’accès conditionnel, elle ne remplace pas une série de tests réels.

### <a name="test-your-policy"></a>Tester votre stratégie

Réalisez chaque test dans votre plan de test, avec des utilisateurs de test.

**Veillez à tester aussi les critères d’exclusion d’une stratégie**. Par exemple, vous pouvez exclure un utilisateur ou un groupe d’une stratégie qui exige l’authentification multifacteur. Testez si les utilisateurs exclus sont invités à utiliser l’authentification multifacteur, car l’association d’autres stratégies peut exiger l’authentification multifacteur pour ces utilisateurs.

### <a name="roll-back-policies"></a>Restaurer les stratégies

Si vous devez restaurer les stratégies que vous venez d’implémenter, utilisez une ou plusieurs options suivantes :

* **Désactiver la stratégie.** La désactivation d’une stratégie garantit qu’elle ne s’applique pas quand un utilisateur tente de se connecter. Vous pouvez toujours revenir en arrière et activer la stratégie quand vous voulez l’utiliser.

![image Activer la stratégie](media/plan-conditional-access/enable-policy.png)

* **Exclure un utilisateur ou un groupe d’une stratégie.** Si un utilisateur ne peut pas accéder à l’application, vous pouvez l’exclure de la stratégie.

![Exclure des utilisateurs et des groupes](media/plan-conditional-access/exclude-users-groups.png)

> [!NOTE]
>  Cette option doit être utilisée avec parcimonie, uniquement si l’utilisateur est approuvé. L’utilisateur doit être rajouté dans la stratégie ou le groupe dès que possible.

* **Supprimer la stratégie.** Si la stratégie n’est plus nécessaire, [supprimez](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json)-la.

## <a name="manage-access-to-cloud-apps"></a>Gérer l’accès aux applications cloud

Utilisez les options de gestion suivantes pour contrôler et gérer vos stratégies d’accès conditionnel :

![Capture d’écran montrant les options MANAGE pour les stratégies d’accès conditionnel, y compris les emplacements nommés, les contrôles personnalisés, les conditions d’utilisation, la connectivité VPN et les stratégies classiques sélectionnées.](media/plan-conditional-access/manage-access.png)


### <a name="named-locations"></a>Emplacements nommés

La condition d’emplacement d’une stratégie d’accès conditionnel vous permet de lier des paramètres de contrôle d’accès aux emplacements réseau de vos utilisateurs. Avec les [Emplacements nommés](location-condition.md), vous pouvez créer des regroupements logiques de plages d’adresses IP, ou de pays et de régions.

### <a name="custom-controls"></a>Contrôles personnalisés

Les [Contrôles personnalisés](controls.md) redirigent vos utilisateurs vers un service compatible pour satisfaire aux exigences d’authentification en dehors d’Azure AD. Pour satisfaire à ce contrôle, le navigateur de l’utilisateur est redirigé vers le service externe, il effectue les opérations d’authentification nécessaires, puis est redirigé vers Azure AD. Azure AD vérifie la réponse. Si l’utilisateur a été correctement authentifié ou vérifié, il continue dans le flux d’accès conditionnel.

### <a name="terms-of-use"></a>Conditions d’utilisation

Avant qu’ils n’accèdent à certaines applications cloud de votre environnement, vous pouvez obtenir le consentement des utilisateurs par l’acceptation de vos conditions d’utilisation. Suivez ce guide de démarrage rapide [pour créer des conditions d’utilisation](require-tou.md).

## <a name="troubleshoot-conditional-access"></a>Résoudre les problèmes d’accès conditionnel

Lorsqu’un utilisateur rencontre un problème avec une stratégie d’accès conditionnel, collectez les informations suivantes pour faciliter la résolution des problèmes.

* Nom d’utilisateur principal

* Nom d’affichage de l’utilisateur

* Nom du système d’exploitation

* Horodatage (approximation acceptée)

* Application cible

* Type d’application cliente (navigateur ou client)

* ID de corrélation (propre à la connexion)

Si l’utilisateur a reçu un message contenant un lien Plus de détails, il peut collecter la plupart de ces informations pour vous.

![message d’erreur Impossible d’accéder à l’application](media/plan-conditional-access/cant-get-to-app.png)

Dès que vous avez collecté les informations, consultez les ressources suivantes :

* [Problèmes de connexion liés à l’accès conditionnel](troubleshoot-conditional-access.md) – Comprenez les événements de connexion inattendus, relatifs à l’accès conditionnel, à l’aide des messages d’erreur et du journal des connexions Azure AD.

* [Utilisation de l’outil de simulation](troubleshoot-conditional-access-what-if.md) – Comprenez pourquoi une stratégie a été ou n’a pas été appliquée à un utilisateur dans une situation particulière, ou si une stratégie pourrait s’appliquer à un état connu.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur l’authentification multifacteur](../authentication/concept-mfa-howitworks.md)

[En savoir plus sur Identity Protection](../identity-protection/overview-identity-protection.md)

[Gérer des stratégies d’accès conditionnel avec API Graph de Microsoft](/graph/api/resources/conditionalaccesspolicy?view=graph-rest-beta.md)