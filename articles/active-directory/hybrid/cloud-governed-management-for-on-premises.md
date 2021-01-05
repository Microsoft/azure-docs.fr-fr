---
title: Gestion gouvernée par le cloud Azure AD pour les charges de travail locales - Azure
description: Cette rubrique aborde la gestion gouvernée par le cloud pour les charges de travail locales.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1b4d1041b9d330227fadf31f6afc1804174ea2ad
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96340847"
---
# <a name="how-azure-ad-delivers-cloud-governed-management-for-on-premises-workloads"></a>Comment Azure AD permet une gestion gouvernée par le cloud pour les charges de travail locales

Azure Active Directory (Azure AD) est une solution IDaaS (Identity as a Service) complète utilisée par des millions d’organisations, qui couvre tous les aspects de l’identité, de la gestion des accès et de la sécurité. Azure AD conserve les identités de plus d’un milliard d’utilisateurs. Ceux-ci peuvent se connecter et accéder de façon sécurisée aux types de ressources suivants :

* Ressources externes telles que Microsoft 365, le portail Azure et des milliers d’autres applications SaaS (Software-as-a-Service).
* Ressources internes telles que les applications situées sur le réseau d’entreprise et intranet d’une organisation, ainsi que les applications cloud développées par votre propre organisation

Les organisations peuvent utiliser Azure AD si elles sont « entièrement cloud », ou effectuer un déploiement « hybride » si elles disposent de charges de travail locales. Un déploiement hybride d’Azure AD peut faire partie de la stratégie d’une organisation souhaitant migrer ses ressources informatiques vers le cloud, ou souhaitant continuer à intégrer l’infrastructure locale existante aux nouveaux services cloud.

Depuis le début, les organisations « hybrides » ont vu Azure AD comme une extension de leur infrastructure locale. Avec ces déploiements, les points de contrôle sont l’administration de la gouvernance des identités locales, Windows Server Active Directory et d’autres systèmes d’annuaire internes. Les utilisateurs et les groupes sont synchronisés à partir de ces systèmes avec un annuaire cloud tel qu’Azure AD. Une fois que ces identités se trouvent dans le cloud, elles peuvent être mises à disposition dans Microsoft 365, Azure et d’autres applications.

![Cycle de vie des identités](media/cloud-governed-management-for-on-premises//image1.png)

Actuellement, les organisations transfèrent une plus grande partie de leur infrastructure informatique et de leurs applications vers le cloud, et un grand nombre d’entre elles souhaitent bénéficier de la sécurité améliorée et de la gestion simplifiée qui sont fournies par l’IDaaS. Découvrez comment les fonctionnalités IDaaS d’Azure AD peuvent accélérer votre transition vers une gestion gouvernée par le cloud, en fournissant des solutions et des fonctionnalités qui permettent d’adopter rapidement la gestion des identités dans Azure AD, tout en continuant de prendre en charge les applications existantes.

Ce document décrit la stratégie de Microsoft concernant l’IDaaS hybride et explique comment les organisations peuvent utiliser Azure AD pour leurs applications existantes.

## <a name="the-azure-ad-approach-to-cloud-governed-identity-management"></a>Approche Azure AD concernant la gestion des identités gouvernée par le cloud

Lorsque les organisations effectuent leur transition vers le cloud, elles veulent être sûres de garder le contrôle sur l’intégralité de leur environnement. Elles veulent également plus de sécurité et plus de visibilité sur les activités, ce qui est facilité par l’automatisation et les insights proactifs. La « **gestion gouvernée par le cloud** » désigne la façon dont les organisations gèrent et gouvernent les utilisateurs, les applications, les groupes et les appareils à partir du cloud.

En raison de la prolifération des applications SaaS et du rôle grandissant que jouent la collaboration et les identités externes, les organisations doivent pouvoir gérer efficacement à grande échelle. Le nouveau paysage des risques liés au cloud nécessite que les organisations soient plus réactives. Un acteur malveillant qui compromet un utilisateur cloud risque d’affecter aussi bien les applications cloud que les applications locales.

Les organisations hybrides doivent pouvoir déléguer et automatiser les tâches qui étaient auparavant effectuées manuellement par le service informatique. Pour automatiser les tâches, elles ont besoin d’API et de processus qui orchestrent le cycle de vie des différentes ressources liées aux identités (utilisateurs, groupes, applications, appareils). Elles pourront ainsi déléguer la gestion quotidienne de ces ressources à plusieurs personnes n’appartenant pas au service informatique. Azure AD répond à ces exigences via la gestion des comptes d’utilisateur et l’authentification native pour les utilisateurs, sans nécessiter d’infrastructure d’identité locale. Ne pas avoir à créer d’infrastructure locale peut être avantageux pour les organisations qui ont de nouvelles communautés d’utilisateurs (telles que des partenaires commerciaux) qui ne figurent pas dans leur annuaire local, mais dont l’accès est indispensable pour atteindre les objectifs de l’entreprise.

En outre, la gestion ne serait pas complète sans la gouvernance. Dans le monde actuel, la gouvernance est intégrée au système d’identité, il ne s’agit plus d’un module complémentaire. La gouvernance des identités offre aux organisations la possibilité de gérer les identités et d’accéder au cycle de vie des employés, des fournisseurs, des partenaires commerciaux, des services et des applications :

L’incorporation de la gouvernance des identités permet aux entreprises d’effectuer plus facilement leur transition vers la gestion gouvernée par le cloud. En outre, elle permet de réduire la taille du service informatique et facilite la gestion des utilisateurs invités. Enfin, comparé à ce dont peuvent bénéficier les utilisateurs dans une infrastructure locale, elle fournit des insights plus approfondis et une plus grande automatisation. Aujourd’hui, la gouvernance signifie que les organisations bénéficient de la transparence, de la visibilité et du contrôle nécessaires pour l’accès aux ressources de l’organisation. Avec Azure AD, les équipes chargées des opérations de sécurité et d’audit disposent d’une visibilité sur qui a (et qui doit avoir) accès aux ressources de l’organisation (et sur quels appareils), sur ce que ces utilisateurs font de cet accès et si l’organisation utilise un système de contrôle adapté permettant de supprimer ou de limiter l’accès, conformément aux stratégies de l’entreprise ou aux réglementations en vigueur.

Ce nouveau modèle de gestion est avantageux pour les organisations qui disposent à la fois d’applications SaaS et d’applications LOB, car il leur permet de gérer et de sécuriser plus facilement l’accès à ces applications. En intégrant les applications à Azure AD, les organisations pourront utiliser et gérer l’accès des identités cloud et locales de manière cohérente. La gestion du cycle de vie des applications devient plus automatisée, et Azure AD fournit des insights riches sur l’utilisation des applications, ce qui n’était pas facilement réalisable avec la gestion des identités locales. Grâce à Azure AD, aux groupes Microsoft 365 et aux fonctionnalités en libre-service de Teams, les organisations peuvent facilement créer des groupes pour la gestion de l’accès et la collaboration. En outre, elles peuvent ajouter et supprimer des utilisateurs en fonction des exigences relatives à la gestion de l’accès et à la collaboration.

Les fonctionnalités Azure AD à sélectionner pour la gestion gouvernée par le cloud varient en fonction des applications à utiliser, et de la façon dont ces applications sont intégrées à Azure AD. Les sections suivantes décrivent les approches à suivre pour les applications intégrées à Azure Directory, ainsi que les applications qui utilisent des protocoles de fédération (par exemple, SAML, OAuth ou OpenID Connect).

## <a name="cloud-governed-management-for-ad-integrated-applications"></a>Gestion gouvernée par le cloud pour les applications intégrées à Active Directory

Azure AD améliore la gestion des applications locales intégrées à Active Directory via l’accès à distance sécurisé et l’accès conditionnel. En outre, Azure AD fournit des fonctionnalités de gestion du cycle de vie des comptes et de gestion des informations d’identification pour les comptes AD existants de l’utilisateur, notamment :

* **Accès à distance sécurisé et accès conditionnel pour les applications locales**

Pour de nombreuses organisations, la gestion de l’accès aux applications de bureau distantes et aux applications web locales intégrées à Azure Directory, à partir du cloud, nécessite en premier lieu le déploiement du [proxy d’application](../manage-apps/application-proxy.md) devant ces applications afin de fournir un accès à distance sécurisé.

Après une authentification unique à Azure AD, les utilisateurs peuvent accéder aux applications cloud et locales par le biais d’une URL externe ou un portail d’applications interne. Par exemple, le proxy d’application fournit l’accès à distance et l’authentification unique aux applications Bureau à distance, SharePoint, Tableau, Qlik, ainsi qu’aux applications métier (LOB). En outre, les stratégies d’accès conditionnel peuvent inclure l’affichage de [conditions d’utilisation](../conditional-access/terms-of-use.md) et un processus permettant de [ vérifier que l’utilisateur a accepté ces conditions](../conditional-access/require-tou.md) avant de lui permettre d’accéder à une application.

![Architecture du proxy d’application](media/cloud-governed-management-for-on-premises/image2.png)

* **Gestion automatique du cycle de vie des comptes Active Directory**

La gouvernance des identités aide les organisations à trouver un équilibre entre *productivité* (vitesse à laquelle des personnes peuvent accéder aux ressources dont elles ont besoin) et *sécurité* (comment leur accès doit évoluer au fil du temps, par exemple à la suite de modifications de leur statut professionnel). La gestion du cycle de vie des identités constitue le fondement de la gouvernance des identités. Une gouvernance efficace à grande échelle implique la modernisation de l’infrastructure de gestion du cycle de vie des identités pour les applications.

Pour de nombreuses organisations, le cycle de vie des identités des employés est lié à la représentation de l’utilisateur concerné dans un système de gestion du capital humain. Pour les organisations qui utilisent Workday comme système de gestion du capital humain, Azure AD peut garantir que les comptes d’utilisateur dans AD seront [automatiquement provisionnés et déprovisionnés pour les employés dans Workday](../saas-apps/workday-inbound-tutorial.md). En procédant ainsi, vous améliorez la productivité des utilisateurs grâce à l’automatisation des comptes birthright et vous réduisez les risques en garantissant que l’accès aux applications sera automatiquement mis à jour lorsqu’un utilisateur change de rôle ou quitte l’organisation. Le [plan de déploiement](https://aka.ms/WorkdayDeploymentPlan) de l’attribution d’utilisateurs pilotée par Workday est un guide pas à pas qui décrit, dans un processus en cinq étapes, les bonnes pratiques nécessaires à l’implémentation de Workday dans la solution d’attribution d’utilisateurs Active Directory.

Azure AD Premium inclut également Microsoft Identity Manager, qui peut importer des enregistrements à partir d’autres systèmes locaux de gestion du capital humain, tels que SAP, Oracle eBusiness et Oracle PeopleSoft.

De plus en plus, la collaboration entreprise-entreprise nécessite que vous accordiez un accès aux personnes extérieures à votre organisation. La collaboration [Azure AD B2B](/azure/active-directory/b2b/) permet aux entreprises de partager de façon sécurisée leurs applications et leurs services avec des utilisateurs invités et des partenaires externes, tout en gardant le contrôle de leurs données.

Azure AD peut [créer automatiquement des comptes dans Active Directory pour les utilisateurs invités](../external-identities/hybrid-cloud-to-on-premises.md) en fonction des besoins. Ainsi, les invités peuvent accéder aux applications locales intégrées à Active Directory, sans avoir besoin d’un autre mot de passe. Les entreprises peuvent définir des [stratégies d’authentification multifacteur (MFA) pour les utilisateurs invités](../external-identities/conditional-access.md) afin que les vérifications MFA soient effectuées lors de l’authentification du proxy d’application. En outre, les [révisions d’accès](../governance/manage-guest-access-with-access-reviews.md) qui sont effectuées au sujet des utilisateurs cloud B2B s’appliquent également aux utilisateurs locaux. Par exemple, si l’utilisateur cloud est supprimé via vos stratégies de gestion du cycle de vie, l’utilisateur local est également supprimé.

**Gestion des informations d’identification pour les comptes Active Directory** La réinitialisation de mot de passe en libre-service d’Azure AD permet aux utilisateurs qui ont oublié leur mot de passe de se réauthentifier et de réinitialiser leur mot de passe. Dans ce cas, les mots de passe modifiés sont [écrits dans l’instance locale d’Active Directory](../authentication/concept-sspr-writeback.md). Le processus de réinitialisation du mot de passe peut également utiliser les stratégies de mot de passe de l’instance locale d’Active Directory : Lorsqu’un utilisateur réinitialise son mot de passe, celui-ci est vérifié afin de garantir qu’il répond à la stratégie Active Directory locale avant d’être validé dans l’annuaire. Le [plan de déploiement](../authentication/howto-sspr-deployment.md) de réinitialisation de mot de passe en libre-service décrit les bonnes pratiques relatives au déploiement d’une telle réinitialisation sur le Web et dans Windows.

![Architecture SSPR d’Azure AD](media/cloud-governed-management-for-on-premises/image3.png)

Enfin, pour les organisations qui autorisent les utilisateurs à modifier leurs mots de passe dans Active Directory, Active Directory peut être configuré pour utiliser la même stratégie de mot de passe que l’organisation dans Azure AD via la [fonctionnalité de protection de mot de passe Azure AD](../authentication/concept-password-ban-bad-on-premises.md) (actuellement en préversion publique).

Lorsqu’une entreprise s’apprête à déplacer une application intégrée à Active Directory vers le cloud en déplaçant le système d’exploitation qui l’héberge vers Azure, [Azure AD Domain Services](../../active-directory-domain-services/overview.md) fournit des services de domaine compatibles avec AD (comme la jonction de domaine, la stratégie de groupe ou l’authentification LDAP et Kerberos/NTLM). Azure Active Directory Domain Services s’intègre au locataire Azure AD existant de l’organisation, permettant ainsi aux utilisateurs de se connecter à l’aide de leurs informations d’identification d’entreprise. En outre, vous pouvez utiliser des comptes d’utilisateurs et des groupes existants pour sécuriser l’accès aux ressources, et garantir un « lift-and-shift » plus simple des ressources locales vers les services d’infrastructure Azure.

![Services de domaine Azure AD](media/cloud-governed-management-for-on-premises/image4.png)

## <a name="cloud-governed-management-for-on-premises-federation-based-applications"></a>Gestion gouvernée par le cloud pour les applications locales basées sur la fédération

Si l’organisation utilise déjà un fournisseur d’identité local, le déplacement des applications vers Azure AD permet un accès plus sécurisé et une gestion facilitée de la fédération. Azure AD permet de configurer un contrôle d'accès granulaire par application, incluant Azure AD Multi-Factor Authentication, à l'aide de la fonctionnalité d'accès conditionnel Azure AD. Azure AD prend en charge davantage de fonctionnalités, y compris les certificats de signature de jeton propres à l’application et les dates d’expiration de certificat configurables. Ces fonctionnalités, outils et conseils permettent aux organisations de mettre hors service leurs fournisseurs d’identité locaux. Pour vous donner un exemple, le service informatique de Microsoft a déplacé vers Azure AD 17 987 applications qui étaient situées sur son instance interne des services de fédération Active Directory (ADFS).

![Évolution d’AD Azure](media/cloud-governed-management-for-on-premises/image5.png)

Pour commencer la migration des applications fédérées vers Azure AD comme fournisseur d’identité, reportez-vous à la page https://aka.ms/migrateapps qui comprend des liens vers :

* Le livre blanc [Migrating Your Applications to Azure Active Directory](https://aka.ms/migrateapps/whitepaper) présente les avantages de la migration et explique comment la planifier en quatre phases clairement expliquées : la découverte, la classification, la migration et la gestion continue. Vous serez guidé pour savoir comment penser ce processus et diviser votre projet en éléments faciles à utiliser. Le document contient des liens vers des ressources importantes qui vous aideront tout au long du processus.

* Le guide de solution [Migrating Application Authentication from Active Directory Federation Services to Azure Active Directory](../manage-apps/migrate-adfs-apps-to-azure.md) explore plus en détail les mêmes quatre phases de planification et d’exécution d’un projet de migration d’application. Dans ce guide, vous allez voir comment appliquer ces phases au déplacement d’une application entre Active Directory Federation Services (AD FS) et Azure AD.

* Le [script de préparation à la migration des services de fédération Active Directory](https://aka.ms/migrateapps/adfstools) peut être exécuté sur des serveurs locaux AD FS existants pour déterminer l’état de préparation des applications en vue de leur migration vers Azure AD.

## <a name="ongoing-access-management-across-cloud-and-on-premises-applications"></a>Gestion continue de l’accès aux applications locales et aux applications cloud

Les organisations ont besoin d’un processus pour gérer un accès scalable. Les utilisateurs cumulent les droits d’accès et se retrouvent avec plus de droits que ce qui avait été initialement prévu pour eux. En outre, les entreprises doivent pouvoir procéder efficacement à une mise à l’échelle pour être en mesure de développer et d’appliquer des contrôles et une stratégie d’accès de manière continue.

En règle générale, le service informatique délègue aux décideurs d’entreprise la prise de décision en matière d’approbation d’accès. De plus, le service informatique peut impliquer les utilisateurs eux-mêmes. Par exemple, les utilisateurs accédant à des données client confidentielles dans l’application marketing d’une société en Europe doivent connaître les stratégies de l’entreprise. Il est également possible que les utilisateurs invités n’aient pas connaissance des exigences de traitement des données appliquées par l’organisation.

Les organisations peuvent automatiser le processus de cycle de vie des accès à l’aide de technologies comme les [groupes dynamiques](../enterprise-users/groups-dynamic-membership.md), couplées à l’attribution des utilisateurs dans les [applications SaaS](../saas-apps/tutorial-list.md), ou comme les [applications intégrées respectant le standard SCIM](../app-provisioning/use-scim-to-provision-users-and-groups.md). Les organisations peuvent également contrôler les [utilisateurs invités pouvant accéder aux applications locales](../external-identities/hybrid-cloud-to-on-premises.md). Ces droits d’accès peuvent ensuite être révisés régulièrement par le biais de [révisions d’accès Azure AD](../governance/access-reviews-overview.md) récurrentes.

## <a name="future-directions"></a>À l’avenir

Dans les environnements hybrides, la stratégie de Microsoft consiste à permettre les déploiements où le **cloud est le plan de contrôle de l’identité** et les répertoires locaux et autres systèmes d’identité, tels qu’Active Directory et autres applications locales, sont la cible de l’attribution des utilisateurs disposant d’un accès. Cette stratégie continuera de garantir les droits, identités et accès dans les applications et les charges de travail qui s’appuient sur eux. Le but est que les organisations puissent générer la productivité des utilisateurs finaux entièrement à partir du cloud.

![Architecture Azure AD](media/cloud-governed-management-for-on-premises/image6.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur ce processus, consultez les plans de déploiement Azure AD, situés ici : <https://aka.ms/deploymentplans>. Ces plans fournissent des instructions de bout en bout sur la façon de déployer certaines fonctionnalités Azure Active Directory (Azure AD). Chaque plan aborde les notions de base (valeur commerciale, planification, conception et procédures opérationnelles) nécessaires pour réussir le lancement des fonctionnalités Azure AD les plus courantes. Microsoft met constamment à jour les plans de déploiement en y ajoutant les bonnes pratiques rassemblées grâce aux commentaires qui nous sont envoyés dans le cadre de la publication des nouvelles fonctionnalités de gestion à partir du cloud dans Azure AD.