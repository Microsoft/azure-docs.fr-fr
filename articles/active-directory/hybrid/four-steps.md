---
title: Créer une base d’identité solide en quatre étapes – Azure AD
description: Cette rubrique décrit les quatre étapes que les clients d’identité hybride peuvent effectuer pour générer une base d’identité solide.
services: active-directory
author: martincoetzer
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/20/2019
ms.subservice: hybrid
ms.author: martinco
ms.collection: M365-identity-device-management
ms.openlocfilehash: 795f5ede382e561ee810e54e1f8897c5d806e8b8
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94412372"
---
# <a name="four-steps-to-a-strong-identity-foundation-with-azure-active-directory"></a>Une base d’identité solide en quatre étapes avec Azure Active Directory

La gestion de l’accès aux données et aux applications ne peut plus reposer sur les stratégies de limite de sécurité réseau traditionnelles telles que les réseaux de périmètre et les pare-feux en raison du déplacement rapide des applications dans le cloud. Les organisations doivent désormais faire confiance à leur solution d’identité pour contrôler qui a accès et à quelles applications et données de l’organisation. De plus en plus d’organisations autorisent leurs employés à utiliser leurs propres appareils au travail et depuis n’importe quel endroit où ils peuvent se connecter à Internet. Veiller à ce que ces appareils soient conformes et sécurisé est devenu un facteur important de la solution d’identité qu’une organisation décide d’implémenter. Dans l’environnement de travail numérique d’aujourd’hui, [l’identité est le principal plan de contrôle](https://www.microsoft.com/security/technology/identity-access-management?rtc=1) de toute organisation migrant vers le cloud.

En adoptant une solution d’identité hybride Azure Active Directory (Azure AD), les organisations peuvent accéder à des fonctionnalités premium qui libèrent la productivité grâce aux fonctionnalités d’automatisation, de délégation, de libre-service et d’authentification unique. Elle permet à vos employés d’accéder aux ressources d’entreprise là où ils doivent effectuer leur travail tout en permettant à votre équipe informatique de contrôler cet accès en s’assurant que les bonnes personnes disposent d’un accès approprié aux ressources appropriées pour garantir une productivité en toute sécurité.

Sur les connaissances que nous avons acquises, cette liste de contrôle des meilleures pratiques vous permettra de déployer rapidement les actions recommandées pour générer une base d’identité *solide* dans votre organisation :

* Se connecter facilement aux applications
* Définir automatiquement une identité pour chaque utilisateur
* Responsabiliser vos utilisateurs en toute sécurité
* Rendre vos informations opérationnelles

## <a name="step-1---connect-to-apps-easily"></a>Étape 1 - Se connecter facilement aux applications

En vous connectant à vos applications avec Azure AD, vous pouvez améliorer la productivité et la sécurité de l’utilisateur final en activant l’authentification unique (SSO) et en approvisionnant des utilisateurs. Grâce à la gestion centralisée de vos applications, dans Azure AD, vous pouvez réduire la charge administrative et obtenir un point de contrôle unique de vos stratégies de sécurité et de conformité.

Cette section décrit vos options de gestion de l’accès utilisateur aux applications, d’activation d’un accès à distance sécurisé aux applications internes et les avantages de la migration de vos applications vers Azure AD.

### <a name="make-apps-available-to-your-users-seamlessly"></a>Mettre des applications à disposition de vos utilisateurs en toute transparence

Azure AD permet aux administrateurs d’[ajouter des applications](../manage-apps/add-application-portal.md) à la galerie d’applications d’entreprise dans le [portail Azure](https://portal.azure.com/). L’ajout d’applications à la galerie d’applications d’entreprise vous permet de configurer plus facilement les applications pour qu’elles utilisent Azure AD comme fournisseur d’identité. Cela permet également de gérer l’accès utilisateur à l’application avec des stratégies d’accès conditionnel et de configurer l’authentification unique (SSO) sur les applications afin que les utilisateurs ne soient pas obligés d’entrer à chaque fois leur mot de passe et qu’ils soient automatiquement connectés aux applications locales et cloud.

Lorsque des applications sont ajoutées à la galerie Azure AD, les utilisateurs peuvent voir les applications qui leur sont attribuées, et rechercher et demander d’autres applications si nécessaire. Azure AD propose aux utilisateurs [plusieurs méthodes](../manage-apps/end-user-experiences.md) d’accès à leurs applications :

* Panneau d’accès/Mes applications
* Lanceur d’applications Microsoft 365
* Authentification directe pour les applications fédérées
* Liens d’authentification directe

Pour en savoir plus sur l’accès utilisateur aux applications, consultez **Étape 3 - Responsabiliser vos utilisateurs** dans cet article.

### <a name="migrate-apps-from-active-directory-federation-services-to-azure-ad"></a>Migrer des applications des services de fédération Active Directory vers Azure AD

La migration de la configuration de l’authentification unique entre les services de fédération Active Directory (AD FS) et Azure AD offre des fonctionnalités de sécurité supplémentaires, une facilité de gestion plus cohérente et la collaboration. Pour obtenir des résultats optimaux, nous vous recommandons de migrer vos applications d’AD FS vers Azure AD. La migration de l’authentification et l’autorisation de votre application vers Azure AD vous offrent les avantages suivants :

* Gestion des coûts
* Gestion des risques
* Optimisation de la productivité
* Respect de la conformité et de la gouvernance

Pour en savoir plus, consultez le livre blanc [Migration de vos applications vers Azure Active Directory](https://aka.ms/migrateapps/whitepaper).

### <a name="enable-secure-remote-access-to-apps"></a>Activer l’accès à distance sécurisé aux applications

Le [proxy d’application Azure AD](../manage-apps/what-is-application-proxy.md) fournit une solution simple permettant aux organisations de publier des applications locales sur le cloud pour les utilisateurs distants qui doivent accéder aux applications internes de manière sécurisée. Après une authentification unique à Azure AD, les utilisateurs peuvent accéder aux applications cloud et locales par le biais d’une URL externe ou d’un portail d’applications interne.

Le proxy d’application Azure AD offre les avantages suivants :

* L’extension d’Azure AD à des ressources locales
  * La protection et la sécurité à l’échelle du cloud
  * Des fonctionnalités comme l’accès conditionnel et l’authentification multifacteur, faciles à activer
* Aucun composant dans le réseau de périmètre tel que des solutions de VPN et de proxy inversé traditionnelles
* Aucune connexion entrante nécessaire
* Authentification unique (SSO) sur les appareils, ressources et applications dans le cloud et localement
* Permet aux utilisateurs finaux d’être productifs à tout moment et en tout lieu

### <a name="discover-shadow-it-with-microsoft-cloud-app-security"></a>Découvrir l’informatique fantôme grâce à Microsoft Cloud App Security

Dans les entreprises modernes, les services informatiques n’ont souvent pas connaissance de toutes les applications cloud utilisées par les utilisateurs pour effectuer leur travail. Lorsque l’on demande aux administrateurs informatiques quel nombre d’applications les employés utilisent, selon eux, ils répondent généralement 30 ou 40. En réalité, plus de 1 000 applications distinctes sont en moyenne utilisées par les employés de votre organisation. 80 % des employés utilisent des applications non approuvées que personne n’a révisées et qui peuvent ne pas être conformes à vos stratégies de sécurité et de conformité.

[Microsoft Cloud App Security](/cloud-app-security/what-is-cloud-app-security) (MCAS) peut vous aider à identifier les applications utiles et couramment utilisées par les utilisateurs que l’équipe informatique peut approuver et ajouter à la galerie des applications d’entreprise pour que les utilisateurs bénéficient de fonctionnalités comme l’authentification unique et l’accès conditionnel.

<em>« **Cloud App Security** nous permet de nous assurer que nos collaborateurs utilisent correctement nos applications cloud et SaaS, d’une manière conforme aux stratégies de sécurité fondamentales qui permettent de protéger Accenture. »</em> --- [John Blasi, Directeur général, Sécurité des informations, Accenture](https://customers.microsoft.com/story/accenture-professional-services-cloud-app-security)

Outre la détection de l’informatique fantôme, MCAS peut également déterminer le niveau de risque des applications, empêcher tout accès non autorisé aux données d’entreprise, les fuites de données possibles et autres risques de sécurité inhérents aux applications.

## <a name="step-2---establish-one-identity-for-every-user-automatically"></a>Étape 2 - Définir automatiquement une identité pour chaque utilisateur

Le regroupement des annuaires locaux et basés sur le cloud dans une solution d’identité hybride Azure AD vous permettra de réutiliser votre investissement Active Directory local en approvisionnant vos identités existantes dans le cloud. La solution synchronise les identités locales avec Azure AD alors que l’équipe informatique gère l’exécution locale d’Active Directory à l’aide des solutions de gouvernance existantes comme principale source approuvée d’identités. La solution d’identité hybride Azure AD de Microsoft regroupe des fonctionnalités, locales et cloud, de création d’une identité d’utilisateur commune pour l’authentification et l’autorisation d’accès à toutes les ressources, indépendamment de leur emplacement.

L’intégration de vos annuaires locaux à Azure AD améliore la productivité de vos utilisateurs et les empêche d’utiliser plusieurs comptes entre applications et services en leur fournissant une identité commune pour accéder aux ressources cloud et locales. L’utilisation de plusieurs comptes est une problématique pour les utilisateurs finaux et l’équipe informatique. Du point de vue de l’utilisateur final, disposer de plusieurs comptes implique de devoir mémoriser plusieurs mots de passe. Pour éviter ce problème, nombre d’utilisateurs réutilisent le même mot de passe pour chaque compte, ce qui n’est pas correct en termes de sécurité. Du point de vue de l’équipement informatique, la réutilisation entraîne généralement un plus grand nombre de réinitialisations des mots de passe, des coûts de support supérieurs et plus de réclamations d’utilisateur final.

Azure AD Connect est l’outil utilisé pour synchroniser vos identités locales avec Azure AD, qui peuvent ensuite être utilisées pour accéder aux applications cloud. Une fois que les identités se trouvent dans Azure AD, elles peuvent être approvisionnées pour des applications SaaS telles que Salesforce ou Concur.

Dans cette section, nous présentons les suggestions de fourniture de la haute disponibilité, de l’authentification moderne pour le cloud et de réduction de votre empreinte locale.

> [!NOTE]
> Si vous souhaitez en savoir plus sur Azure AD Connect, consultez [Qu’est-ce que la synchronisation d’Azure AD Connect ?](./how-to-connect-sync-whatis.md)

### <a name="set-up-a-staging-server-for-azure-ad-connect-and-keep-it-up-to-date"></a>Configurer un serveur intermédiaire pour Azure AD Connect et le maintenir à jour

Azure AD Connect joue un rôle clé dans le processus d’approvisionnement. Si le serveur de synchronisation est déconnecté pour une quelconque raison, les modifications apportées en local ne sont mises à jour dans le cloud et entraînent des problèmes d’accès pour les utilisateurs. Il est important de définir une stratégie de basculement qui permet aux administrateurs de reprendre rapidement la synchronisation lorsque le serveur de synchronisation est déconnecté.

Pour fournir la haute disponibilité en cas de déconnexion de votre serveur Azure AD Connect principal, il est recommandé de déployer un [serveur intermédiaire](./how-to-connect-sync-staging-server.md) distinct pour Azure AD Connect. Le déploiement d’un serveur permet à l’administrateur de « promouvoir » le serveur intermédiaire en production d’un simple changement de configuration. Disposer d’un serveur de secours configuré en mode intermédiaire vous permet également de tester et de déployer de nouvelles modifications de configuration et d’introduire un nouveau serveur si l’ancien est mis hors service.

> [!TIP]
> Azure AD Connect est régulièrement mis à jour. Par conséquent, il est vivement recommandé de maintenir le serveur intermédiaire à jour pour pouvoir bénéficier des améliorations de performances, des correctifs de bogues et des nouvelles fonctionnalités fournies par chaque nouvelle version.

### <a name="enable-cloud-authentication"></a>Activer l’authentification cloud

Les organisations disposant d’Active Directory en local doivent étendre leur annuaire à Azure AD à l’aide d’Azure AD Connect et configurer la méthode d’authentification appropriée. La [choix de la méthode d’authentification appropriée](./choose-ad-authn.md) pour votre organisation est la première étape dans votre processus de déplacement d’applications vers le cloud. Il s’agit d’un composant critique car il contrôle l’accès à l’ensemble des données et des ressources du cloud.

La méthode la plus simple et recommandée d’activation de l’authentification cloud pour les objets d’annuaire local dans Azure AD consiste à activer la [synchronisation de hachage du mot de passe](./how-to-connect-password-hash-synchronization.md). Certaines organisations peuvent également envisager d’activer l’[authentification directe](./how-to-connect-pta-quick-start.md).

Que vous choisissiez la synchronisation de hachage du mot de passe ou l’authentification directe, n’oubliez pas d’activer l’[authentification unique transparente](./how-to-connect-sso.md) pour permettre aux utilisateurs d’accéder aux applications cloud sans saisir constamment leurs nom d’utilisateur et mot de passe dans l’application lorsqu’ils utilisent des appareils Windows 7 et 8 sur votre réseau d’entreprise. Sans l’authentification unique, les utilisateurs doivent mémoriser des mots de passe spécifiques aux applications et se connecter à chaque application. L’équipe informatique doit également créer et mettre à jour les comptes d’utilisateur pour chaque application, comme Microsoft 365, Box et Salesforce. Les utilisateurs doivent mémoriser leurs mots de passe et passer du temps à se connecter à chaque application. La fourniture d’un mécanisme d’authentification unique standardisé à l’échelle de l’entreprise est essentielle pour une meilleure expérience utilisateur, réduire les risques, la possibilité de signaler et la gouvernance.

Pour les organisations qui utilisent déjà AD FS ou un autre fournisseur d’authentification local, le basculement vers Azure AD comme fournisseur d’identité peut réduire la complexité et améliorer la disponibilité. Sauf si vous rencontrez des cas d’usage spécifiques de fédération, nous vous recommandons de migrer de l’authentification fédérée vers la synchronisation de hachage du mot de passe ou l’authentification directe et l’authentification unique transparente ou l’authentification directe et l’authentification unique transparente pour tirer profit des avantages d’empreinte locale réduite et de la flexibilité du cloud avec de meilleures expériences utilisateurs. Pour plus d’informations, consultez [Migrer de la fédération à la synchronisation de hachage de mot de passe pour Azure Active Directory](./plan-migrate-adfs-password-hash-sync.md).

### <a name="enable-automatic-deprovisioning-of-accounts"></a>Activer le déprovisionnement automatique de comptes

L’activation de l’approvisionnement et du déprovisionnement automatisés sur vos applications est la meilleure stratégie de contrôle du cycle de vie des identités sur plusieurs systèmes. Azure AD prend en charge [l’approvisionnement et le déprovisionnement basés sur des stratégies automatisés](../app-provisioning/configure-automatic-user-provisioning-portal.md) de comptes d’utilisateur sur diverses applications SaaS populaires telles que ServiceNow et Salesforce, et d’autres qui implémentent le [protocole SCIM 2.0](../app-provisioning/use-scim-to-provision-users-and-groups.md). Contrairement aux solutions d’approvisionnement traditionnelles, qui nécessitent du code personnalisé ou le chargement manuel de fichiers CSV, le service d’approvisionnement est hébergé dans le cloud et inclut des connecteurs pré-intégrés pouvant être configurés et gérés à l’aide du portail Azure. Un avantage clé du déprovisionnement automatique est qu’il permet de sécuriser votre organisation en supprimant instantanément les identités des utilisateurs des applications SaaS lorsqu’ils quittent l’organisation.

Pour en savoir plus sur l’approvisionnement automatique de comptes d’utilisateur, consultez [Automatisation de l’approvisionnement et de l’annulation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="step-3---empower-your-users-securely"></a>Étape 3 - Responsabiliser vos utilisateurs en toute sécurité

Dans l’environnement de travail numérique d’aujourd’hui, il est important de trouver un équilibre entre sécurité et productivité. Cependant, les utilisateurs finaux contournent souvent les mesures de sécurité qui ralentissent leur productivité et leur accès aux applications cloud. Pour résoudre ce problème, Azure AD fournit des fonctionnalités en libre-service qui permettent aux utilisateurs de rester productifs tout en réduisant la charge administrative.

Cette section présente les suggestions d’élimination des frictions pour votre organisation en responsabilisant vos utilisateurs tout en restant vigilant.

### <a name="enable-self-service-password-reset-for-all-users"></a>Activer la réinitialisation du mot de passe en libre-service pour tous les utilisateurs

La [réinitialisation du mot de passe en libre-service](../authentication/tutorial-enable-sspr.md) (SSPR) d’Azure offre aux administrateurs informatiques un moyen simple pour permettre aux utilisateurs de réinitialiser et de déverrouiller leurs comptes ou leurs mots de passe, sans intervention de l’administrateur. Le système inclut des rapports détaillés de suivi d’accès au système, ainsi que des notifications pour vous prévenir de toute utilisation malveillante ou de tout abus.

Par défaut, Azure AD déverrouille les comptes quand il effectue une réinitialisation de mot de passe. Cependant, lorsque vous activez l’[intégration locale](../authentication/concept-sspr-howitworks.md#on-premises-integration) d’Azure AD Connect, vous pouvez également différencier ces deux opérations, permettant ainsi aux utilisateurs de déverrouiller leur compte sans devoir réinitialiser le mot de passe.

### <a name="ensure-all-users-are-registered-for-mfa-and-sspr"></a>S’assurer que tous les utilisateurs sont inscrits pour l’authentification multifacteur (MFA) et la réinitialisation de mot de passe en libre-service (SSPR)

Azure fournit des rapports qui peuvent être utilisés par vous et votre organisation pour s’assurer que les utilisateurs sont inscrits pour l’authentification multifacteur (MFA) et la réinitialisation de mot de passe en libre-service (SSPR) Les utilisateurs qui ne sont pas inscrits devront peut-être être formés sur le processus.

Le [rapport de connexions](../authentication/howto-mfa-reporting.md) d’authentification multifacteur inclut des informations sur l’utilisation de l’authentification multifacteur (MFA) et sur son fonctionnement dans votre organisation. Avoir accès à l’activité de connexion (et aux audits et détections des risques) pour Azure AD est essentiel pour la résolution des problèmes, l’analyse de l’utilisation et les investigations d’analyse.

De même, le [rapport de gestion des mots de passe en libre-service](../authentication/howto-sspr-reporting.md) peut être utilisé pour déterminer qui est (ou n’est pas) inscrit pour la réinitialisation de mot de passe en libre-service (SSPR).

### <a name="self-service-app-management"></a>Gestion des applications en libre-service

Pour permettre à vos utilisateurs de découvrir eux-mêmes des applications depuis leur panneau d’accès, vous devez activer l’option d’[accès aux applications en libre-service](../manage-apps/access-panel-manage-self-service-access.md) pour toutes les applications pour lesquelles vous souhaitez autoriser les utilisateurs à les découvrir eux-mêmes et en demander l’accès. L’accès aux applications en libre-service est un excellent moyen pour permettre aux utilisateurs de découvrir eux-mêmes des applications et éventuellement de permettre au groupe d’entreprise d’approuver l’accès à ces applications. Vous pouvez autoriser le groupe d’entreprise à gérer les informations d’identification affectées à ces utilisateurs dans le cadre d’une [authentification unique par mot de passe](../manage-apps/troubleshoot-password-based-sso.md#automatically-capture-sign-in-fields-for-an-app), directement depuis leurs panneaux d’accès.

### <a name="self-service-group-management"></a>Gestion des groupes en libre service

L’attribution d’utilisateurs à des applications est plus précise à l’aide de groupes, car ceux-ci offre une grande flexibilité et une possibilité de gestion à l’échelle :

* Basé sur des attributs utilisant l’appartenance à un groupe dynamique
* Délégation aux propriétaires d’application

Azure AD fournit la possibilité de gérer l’accès aux ressources à l’aide de groupes de sécurité et de groupes Microsoft 365. Ces groupes peuvent être gérés par un propriétaire du groupe qui peut approuver ou refuser des requêtes d’appartenance et déléguer le contrôle de l’appartenance au groupe. Également appelée [gestion de groupes en libre-service](../enterprise-users/groups-self-service-management.md), cette fonctionnalité permet de gagner du temps en permettant aux propriétaires de groupe qui n’ont pas un rôle d’administrateur de créer et gérer des groupes sans avoir à demander aux administrateurs de gérer leurs requêtes.

## <a name="step-4---operationalize-your-insights"></a>Étape 4 - Rendre vos informations opérationnelles

L’audit et la journalisation des événements liés à la sécurité et des alertes connexes sont des composants essentiels d’une stratégie efficace pour s’assurer que les utilisateurs restent productifs et que votre organisation est sécurisée. Les journaux et rapports de sécurité peuvent vous aider à répondre à des questions telles que :

* Utilisez-vous ce pour quoi que payez ?
* Des événements suspects ou malveillants se produisent-ils dans mon locataire ?
* Qui a été affecté pendant un incident de sécurité ?

Les journaux d’activité et les rapports de sécurité vous fournissent un enregistrement électronique des activités suspectes et vous aident à détecter les motifs pouvant indiquer une tentative d’intrusion ou une intrusion externe effective sur le réseau, ainsi que des attaques internes. Vous pouvez avoir recours aux audits pour surveiller les activités des utilisateurs, documenter la conformité aux exigences réglementaires, effectuer des analyses d’investigation, etc. Les alertes fournissent des notifications des événements de sécurité.

### <a name="assign-least-privileged-admin-roles-for-operations"></a>Attribuer des rôles administrateur moins privilégiés pour les opérations

Dans votre approche des opérations, prenez en compte deux niveaux d’administration. Dans le premier niveau, les tâches fastidieuses d’administration sont confiées à votre ou vos administrateurs généraux. Toujours utiliser le rôle Administrateur général peut convenir aux petites entreprises. Mais pour les grandes organisations dans lesquelles le personnel du support technique et les administrateurs sont responsables de tâches spécifiques, l’attribution du rôle d’administrateur général peut poser un risque de sécurité car il permet à ces personnes de gérer des tâches au-dessus et au-delà de ce qu’ils sont capables de faire.

Dans ce cas, vous devez envisager le niveau d’administration suivant. À l’aide d’Azure AD, vous pouvez désigner des utilisateurs finaux comme des « administrateurs limités » qui peuvent gérer des tâches dans des rôles moins privilégiés. Par exemple, vous pouvez attribuer à votre personnel du support technique le rôle [Lecteur Sécurité](../roles/permissions-reference.md#security-reader) pour lui permettre de gérer des fonctionnalités liées à la sécurité avec un accès en lecture seule. Il peut également être judicieux d’attribuer le rôle [Administrateur d’authentification](../roles/permissions-reference.md#authentication-administrator) à des personnes pour leur permettre de réinitialiser des informations d’identification autres qu’un mot de passe, ou de consulter et de configurer Azure Service Health.

Pour en savoir plus, consultez [Autorisations des rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md).

### <a name="monitor-hybrid-components-azure-ad-connect-sync-ad-fs-using-azure-ad-connect-health"></a>Surveiller des composants hybrides (synchronisation d’Azure AD Connect, AD FS) avec Azure AD Connect Health

Azure AD Connect et AD FS sont des composants critiques qui peuvent éventuellement interrompre la gestion du cycle de vie et l’authentification, et ainsi entraîner des pannes. Par conséquent, vous devez déployer Azure AD Connect Health pour surveiller et générer des rapports de ces composants.

Pour en savoir plus, lisez [Surveiller AD FS avec Azure AD Connect Health](./how-to-connect-health-adfs.md).

### <a name="use-azure-monitor-to-collect-data-logs-for-analytics"></a>Utiliser Azure Monitor pour collecter des journaux de données pour analyse

[Azure Monitor](../../azure-monitor/overview.md) est un portail de surveillance unifié pour tous les journaux d’Azure AD qui fournit des informations détaillées, des analyses avancées et le Machine Learning intelligent. Avec Azure Monitor, vous pouvez utiliser des métriques et des journaux dans le portail et via des API pour obtenir plus de visibilité sur l’état et les performances de vos ressources. Il offre une expérience transparente sur le portail tout en permettant l’intégration d’un large éventail d’intégrations de produits via des API et des options d’exportation données prenant en charge des systèmes SIEM tiers traditionnels. Azure Monitor vous permet également de configurer des règles d’alerte afin d’être informé ou d’appliquer des mesures automatisées sur les problèmes affectant vos ressources.

![Azure Monitor](./media/four-steps/image1.png)

### <a name="create-custom-dashboards-for-your-leadership-and-your-day-to-day"></a>Créer des tableaux de bord personnalisés pour votre équipe de direction et votre travail quotidien

Les organisations n’ayant pas de solution SIEM peuvent télécharger le [Pack de contenu Power BI](../reports-monitoring/howto-use-azure-monitor-workbooks.md) pour Azure AD. Le pack de contenu Power BI contient des rapports prédéfinis conçus pour vous aider à comprendre comment vos utilisateurs adoptent et utilisent les fonctionnalités Azure AD, vous permettant ainsi d’obtenir des informations sur toutes les activités au sein de votre annuaire. Vous pouvez également créer votre propre [tableau de bord personnalisé](/power-bi/service-dashboards) et le partager avec votre équipe de direction pour rapporter les activités quotidiennes. Les tableaux de bord sont un excellent moyen pour surveiller votre activité et voir vos métriques les plus importantes d’un clin d’œil. Les visualisations sur un tableau de bord peuvent provenir d’un ou plusieurs jeux de données sous-jacents et d’un ou plusieurs rapports. Un tableau de bord combine des données locales et cloud, fournissant ainsi une vue consolidée, quel que soit l’endroit où les données résident.

![Tableau de bord personnalisé Power BI](./media/four-steps/image2.png)

### <a name="understand-your-support-call-drivers"></a>Comprendre vos pilotes d’appel au support

Lorsque vous implémentez une solution d’identité hybride comme indiqué dans cet article, vous remarquerez finalement une réduction de vos appels au support. Les problèmes courants tels que les mots de passe oubliés et les verrouillages de compte sont atténués grâce à l’implémentation de la réinitialisation de mot de passe libre-service d’Azure, alors que l’activation de l’accès aux applications en libre-service permet aux utilisateurs de découvrir eux-mêmes et de demander l’accès aux applications sans devoir passer par le service informatique.

Si vous ne constatez une réduction des appels au support, nous vous recommandons d’analyser vos pilotes d’appel au support pour vérifier si la réinitialisation de mot de passe en libre-service (SSPR) ou l’accès aux applications en libre-service a été correctement configuré ou si de nouveaux problèmes pouvant être systématiquement résolus se posent.

*« Pour notre transformation numérique, nous avions besoin d’un fournisseur d’identité et de gestion des accès fiable pour faciliter l’intégration transparente déjà présente entre nous, nos partenaires et fournisseurs de services cloud, pour obtenir un écosystème efficace. Azure AD a été la meilleure option en nous fournissant les fonctionnalités et la visibilité nécessaires qui nous a permis de détecter et de répondre aux risques. »* --- [Yazan Almasri, Directeur de la sécurité des informations globales, Aramex](https://customers.microsoft.com/story/aramex-azure-active-directory-travel-transportation-united-arab-emirates-en)

### <a name="monitor-your-usage-of-apps-to-drive-insights"></a>Surveiller votre utilisation des applications pour obtenir des informations

Outre la découverte de l’informatique fantôme, la surveillance de l’utilisation des applications à l’échelle de votre organisation à l’aide de [Microsoft Cloud App Security](/cloud-app-security/what-is-cloud-app-security) peut aider le processus de migration de votre organisation à tirer pleinement parti des applications cloud. Elle peut vous permettre de garder le contrôle sur vos ressources grâce à une meilleure visibilité de l’activité et de renforcer la protection des données critiques entre les applications cloud. La surveillance de l’utilisation des applications dans votre organisation à l’aide de MCAS peut vous aider à répondre aux questions suivantes :

* Quelles applications non approuvées les employés utilisent-ils pour stocker des données ?
* Où et quand des données sensibles sont stockées dans le cloud ?
* Qui a accès à des données sensibles dans le cloud ?

*« Grâce à Cloud App Security, nous pouvons identifier rapidement les anomalies et prendre des mesures. »* --- [Eric LePenske, Cadre senior, Sécurité des informations, Accenture](https://customers.microsoft.com/story/accenture-professional-services-cloud-app-security)

## <a name="summary"></a>Résumé

L’implémentation d’une solution d’identité hybride implique de nombreux aspects, mais cette liste de contrôle en quatre étapes vous aidera à créer rapidement une infrastructure d’identité qui permettra aux utilisateurs d’être plus productifs et sécurisés.

* Se connecter facilement aux applications
* Définir automatiquement une identité pour chaque utilisateur
* Responsabiliser vos utilisateurs en toute sécurité
* Rendre vos informations opérationnelles

Nous espérons que ce document sera de feuille de route utile au développement d’une base d’identité solide pour votre organisation.

## <a name="identity-checklist"></a>Liste de contrôle d’identité

Nous vous recommandons d’imprimer la liste de contrôle suivante pour référence dans le processus d’adoption d’une base d’identité plus solide dans votre organisation.

### <a name="today"></a>Aujourd’hui

|Vous avez terminé ?|Élément|
|:-|:-|
||Piloter la réinitialisation de mot de passe en libre-service (SSPR) pour un groupe|
||Surveiller des composants hybrides avec Azure AD Connect Health|
||Attribuer des rôles administrateur moins privilégiés pour les opérations|
||Découvrir l’informatique fantôme grâce à Microsoft Cloud App Security|
||Utiliser Azure Monitor pour collecter des journaux de données pour analyse|

### <a name="next-two-weeks"></a>Deux prochaines semaines

|Vous avez terminé ?|Élément|
|:-|:-|
||Mettre une application à disposition de vos utilisateurs|
||Piloter l’approvisionnement Azure AD pour l’application SaaS de votre choix|
||Configurer un serveur intermédiaire pour Azure AD Connect et le maintenir à jour|
||Démarrer la migration d’applications d’AD FS vers Azure AD|
||Créer des tableaux de bord personnalisés pour votre équipe de direction et votre travail quotidien|

### <a name="next-month"></a>Mois prochain

|Vous avez terminé ?|Élément|
|:-|:-|
||Surveiller votre utilisation des applications pour obtenir des informations|
||Piloter l’accès à distance sécurisé aux applications|
||S’assurer que tous les utilisateurs sont inscrits pour l’authentification multifacteur (MFA) et la réinitialisation de mot de passe en libre-service (SSPR)|
||Activer l’authentification cloud|

### <a name="next-three-months"></a>Trois prochains mois

|Vous avez terminé ?|Élément|
|:-|:-|
||Activer la gestion des applications en libre-service|
||Activer la gestion des groupes en libre-service|
||Surveiller votre utilisation des applications pour obtenir des informations|
||Comprendre vos pilotes d’appel au support|

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment vous pouvez augmenter votre sécurité à l’aide des fonctionnalités d’Azure Active Directory et de cette liste de contrôle en cinq étapes, [Cinq étapes pour sécuriser votre infrastructure d’identité](../../security/fundamentals/steps-secure-identity.md).

Découvrez comment les fonctionnalités d’identité dans Azure AD peuvent vous aider à accélérer votre migration vers une gestion gouvernée par le cloud en fournissant les solutions et fonctionnalités qui permettent aux organisations d’adopter rapidement et de migrer davantage leur gestion des identités entre des systèmes locaux traditionnels et Azure AD, [Comment Azure AD offre une gestion gouvernée par le cloud pour les charges de travail locales](./cloud-governed-management-for-on-premises.md).