---
title: Cinq étapes pour l’intégration de toutes vos applications à Azure AD
description: Ce guide explique comment intégrer toutes vos applications dans Azure AD. À chaque étape, nous expliquons la valeur et fournissons des liens vers des ressources qui expliqueront les détails techniques.
services: active-directory
author: knicholasa
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 08/05/2020
ms.author: nichola
ms.openlocfilehash: 7fd9e504448d55b4a2ef8c10b4ba1176cb2e3402
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96172633"
---
# <a name="five-steps-for-integrating-all-your-apps-with-azure-ad"></a>Cinq étapes pour l’intégration de toutes vos applications à Azure AD

Azure Active Directory (Azure AD) est le service Microsoft basé sur le cloud qui gère les identités et les accès. Azure AD fournit des solutions d’authentification et d’autorisation sécurisées afin que les clients, les partenaires et les employés puissent accéder aux applications dont ils ont besoin. Avec Azure AD, [l’accès conditionnel](../conditional-access/overview.md), [l’authentification multifacteur](../authentication/concept-mfa-howitworks.md), [la connexion unique](../hybrid/how-to-connect-sso.md) et [le provisionnement automatique des utilisateurs](../app-provisioning/user-provisioning.md) facilitent et sécurisent la gestion des identités et des accès.

Si votre entreprise dispose d’un abonnement Microsoft 365, il est probable que vous [déjà utilisé](/office365/enterprise/about-office-365-identity) Azure AD. Toutefois, Azure AD peut être utilisé pour toutes vos applications, et en [centralisant la gestion de vos applications](../manage-apps/common-scenarios.md) vous pouvez utiliser les mêmes fonctionnalités, outils et stratégies de gestion des identités dans l’ensemble de votre portefeuille d’applications. Cela permet de fournir une solution unifiée qui améliore la sécurité, réduit les coûts, augmente la productivité et vous permet de garantir la conformité. Et vous obtiendrez un accès distant aux applications locales.

Ce guide explique comment intégrer toutes vos applications dans Azure AD. À chaque étape, nous expliquons la valeur et fournissons des liens vers des ressources qui expliqueront les détails techniques. Nous vous recommandons de suivre ces étapes dans l’ordre indiqué. Cependant, vous pouvez passer à n’importe quelle étape du processus pour commencer avec ce qui apporte le plus à votre entreprise.

D’autres ressources de cette rubrique, notamment des livres blancs détaillés sur les processus d’entreprise, se trouvent dans notre page [Ressources pour la migration d’applications vers Azure Active Directory](../manage-apps/migration-resources.md) .

## <a name="1-use-azure-ad-for-new-applications"></a>1. Utiliser Azure AD pour les nouvelles applications

Tout d’abord, concentrez-vous sur les applications nouvellement acquises. Lorsque votre entreprise commence à utiliser une nouvelle application, [ajoutez celle-ci à votre locataire Azure AD](../manage-apps/add-application-portal.md) immédiatement. Configurez une stratégie d’entreprise afin que l’ajout de nouvelles applications dans Azure AD soit une pratique courante dans votre organisation. Cela ne perturbe que très peu les processus métier existants et vous permet d’observer et de prouver la valeur que vous tirez de l’intégration des applications sans changer la façon dont les personnes font des affaires dans votre environnement actuel.

Azure Active Directory (Azure AD) dispose d’une galerie contenant des milliers d’applications pré-intégrées pour faciliter la prise en main. Vous pouvez [ajouter une application de galerie à votre organisation Azure AD](../manage-apps/add-application-portal.md) avec des [didacticiels pas à pas](../saas-apps/tutorial-list.md) pour l’intégration à des applications populaires comme :

- [ServiceNow](../saas-apps/servicenow-tutorial.md)
- [Workday](../saas-apps/workday-tutorial.md)
- [Salesforce](../saas-apps/salesforce-tutorial.md)
- [AWS](../saas-apps/amazon-web-service-tutorial.md)
- [Slack](../saas-apps/slack-tutorial.md)

De plus, vous pouvez [intégrer des applications hors de la galerie](../manage-apps/view-applications-portal.md), notamment n’importe quelle application déjà présente dans votre organisation, ou n’importe quelle application tierce d’un fournisseur qui ne fait pas déjà partie de la galerie Azure AD. Vous pouvez également [ajouter votre application à la galerie](../develop/v2-howto-app-gallery-listing.md) si ce n’est pas le cas.

Enfin, vous pouvez également intégrer les applications que vous développez en interne. Ce sujet est abordé à l’étape 5 de ce guide.

## <a name="2-determine-existing-application-usage-and-prioritize-work"></a>2. Déterminer l’utilisation des applications existantes et hiérarchiser le travail

Découvrez ensuite les applications que les employés utilisent fréquemment, et hiérarchisez votre travail pour les intégrer à Azure AD.

Vous pouvez commencer par utiliser les [outils de découverte cloud](/cloud-app-security/tutorial-shadow-it) de Microsoft Cloud App Security pour détecter et gérer l’informatique « fantôme » dans votre réseau (autrement dit, les applications qui ne sont pas gérées par le service informatique). Vous pouvez [utiliser Microsoft Defender - Protection avancée contre les menaces](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection) pour simplifier et étendre le processus de découverte.

En outre, vous pouvez utiliser le [rapport d’activité des applications AD FS](../manage-apps/migrate-adfs-application-activity.md) dans le Portail Azure pour découvrir toutes les applications AD FS de votre organisation, le nombre d’utilisateurs uniques qui s’y sont connectés et la compatibilité pour les intégrer à Azure AD.

Après avoir découvert votre paysage existant, vous voudrez sans doute [créer un plan](../manage-apps/migration-resources.md) et hiérarchiser les applications les plus prioritaires à intégrer. Voici quelques exemples de questions que vous pouvez poser pour vous guider dans ce processus :

- Quelles sont les applications les plus utilisées ?
- Quelles sont les plus risquées ?
- Quelles sont les applications qui seront retirées à l’avenir, rendant inutile le déplacement ?
- Quelles applications doivent rester locales et ne peuvent pas être déplacées vers le cloud ?

Vous constaterez les avantages et les économies les plus importants lorsque toutes vos applications seront intégrées et que vous ne dépendrez plus de multiples solutions d’identité. Cependant, vous constaterez une gestion plus facile de votre identité et une sécurité accrue à mesure que vous progresserez vers cet objectif. Vous voulez utiliser ce temps pour établir des priorités dans votre travail et décider de ce qui convient le mieux à votre situation.

## <a name="3-integrate-apps-that-rely-on-other-identity-providers"></a>3. Intégrer les applications qui reposent sur d’autres fournisseurs d’identité

Au cours de votre processus de découverte, vous avez peut-être trouvé des applications qui ne sont pas suivies par le service informatique, ce qui rend vos données et vos ressources vulnérables. Vous pouvez également disposer d’applications qui utilisent des solutions d’identité alternatives, notamment les Services de fédération Active Directory (AD FS) ou d’autres fournisseurs d’identité. Réfléchissez à la manière dont vous pouvez consolider votre gestion des identités et des accès afin de réaliser des économies et d’accroître la sécurité. En réduisant le nombre de solutions d’identité, vous allez :

- Faire des économies en éliminant le besoin d’approvisionnement et d’authentification des utilisateurs locaux, ainsi que les frais de licence payés aux autres fournisseurs d’identité cloud pour le même service.
- Réduire la charge administrative et activer une sécurité plus étroite avec moins de redondances dans votre processus de gestion des identités et des accès.
- Permettre aux employés d’obtenir un accès sécurisé à l’authentification unique pour toutes les applications dont ils ont besoin via le [portail MyApps](../manage-apps/access-panel-collections.md).
- Améliorer l’intelligence des services liés à la [protection des identités](../identity-protection/overview-identity-protection.md) d’Azure AD, comme l’accès conditionnel, en augmentant la quantité de données que la fonctionnalité reçoit de l’utilisation de votre application, et étendre ses avantages aux applications nouvellement ajoutées.

Nous avons publié des conseils dans le cadre de la gestion du processus métier d’intégration des applications dans Azure AD, notamment une [affiche](https://aka.ms/AppOnePager) et une [présentation](https://aka.ms/AppGuideline) que vous pouvez utiliser pour sensibiliser et intéresser les entreprises et les propriétaires d’applications. Vous pouvez modifier ces exemples avec votre personnalisation et les publier dans votre organisation par le biais du portail d’entreprise, du bulletin d’informations ou d’un autre support au cours de l’exécution de ce processus.

Un bon point de départ consiste à évaluer votre utilisation des Services de fédération Active Directory (AD FS). De nombreuses organisations utilisent ADFS pour l’authentification avec des applications SaaS, des applications métier personnalisées et des applications Microsoft 365 et Azure AD :

![Le schéma montre les applications locales, les applications métier, les applications SaaS et, via Azure AD, les applications Office 365 dont les connexions dans Active Directory et AD FS sont représentées en pointillés.](\media\five-steps-to-full-application-integration-with-azure-ad\adfs-integration-1.png)

Vous pouvez mettre à niveau cette configuration en [remplaçant ADFS par Azure AD comme centre](../manage-apps/migrate-adfs-apps-to-azure.md) de votre solution de gestion des identités. Cela permet d’ouvrir une session pour chaque application à laquelle vos employés veulent accéder, et permet aux employés de trouver facilement les applications métier dont ils ont besoin via le portail [MyApps](../user-help/my-apps-portal-end-user-access.md), en plus des autres avantages mentionnés ci-dessus.

![Le schéma montre les applications locales via Active Directory et AD FS, les applications métier, les applications SaaS et Office 365 dont les connexions dans Azure Active Directory sont représentées en pointillés.](\media\five-steps-to-full-application-integration-with-azure-ad\adfs-integration-2.png)

Azure AD étant devenu le fournisseur d’identité central, vous avez la possibilité de basculer entièrement depuis ADFS plutôt que d’utiliser une solution fédérée. Les applications qui utilisaient auparavant ADFS pour l’authentification peuvent maintenant utiliser uniquement Azure AD.

![Le schéma montre les applications locales, les applications métier, les applications SaaS et Office 365 dont les connexions dans Azure Active Directory sont représentées en pointillés. Active Directory et AD FS ne sont pas présents.](\media\five-steps-to-full-application-integration-with-azure-ad\adfs-integration-3.png)

Vous pouvez également migrer vers Azure AD, les applications qui utilisent un autre fournisseur d’identité basé sur le cloud. Votre organisation peut être dotée de plusieurs solutions IAM (Identity Access Management). La migration vers une infrastructure Azure AD vous donne les moyens de réduire les dépendances sur les licences IAM (localement ou dans le cloud) et les coûts d’infrastructure. Dans les cas où vous avez déjà payé l’utilisation d’Azure AD dans le cadre de licences M365, il n’y a aucune raison de subir le coût supplémentaire d’une autre solution IAM.

## <a name="4-integrate-on-premises-applications"></a>4. Intégrer des applications locales

Traditionnellement, les applications étaient sécurisées en n’autorisant l’accès que lorsqu’elles étaient connectées au réseau de l’entreprise. Cependant, dans un monde de plus en plus connecté, nous souhaitons autoriser l’accès aux applications pour les clients, les partenaires et/ou les employés, quel que soit l’endroit où ils se trouvent dans le monde. [Proxy d’application Azure AD](../manage-apps/what-is-application-proxy.md) (AppProxy) est une fonctionnalité d’Azure AD qui connecte vos applications locales existantes à Azure AD et vous évite d’avoir à gérer des serveurs de périphérie ou d’autres infrastructures supplémentaires pour les connexions.

![Le fonctionnement du service Proxy d’application est illustré dans un schéma. Un utilisateur accède à « https://sales.contoso.com  » et la demande est redirigée via « https://sales-contoso.msappproxy.net  » dans Azure Active Directory vers l’adresse locale « http://sales  »](./media/five-steps-to-full-application-integration-with-azure-ad\app-proxy.png)

Vous pouvez vous reporter au [Didacticiel : Ajoutez une application locale pour l’accès distant via Proxy d’application dans Azure Active Directory](../manage-apps/application-proxy-add-on-premises-application.md) afin d’activer Proxy d’application et d’ajouter une application locale à votre locataire Azure AD.

En outre, vous pouvez intégrer des contrôleurs de livraison d’application, tels que F5 BIG-IP APM, ou un accès privé Zscaler. En les intégrant à Azure AD, vous bénéficiez de l’authentification moderne et de la gestion des identités Azure AD en plus de la gestion du trafic et des fonctionnalités de sécurité du produit partenaire. Nous appelons cette solution [Secure Hybrid Access](../manage-apps/secure-hybrid-access.md). Si vous utilisez actuellement l’un des services suivants, nous vous proposerons des didacticiels qui vous guideront dans la procédure à suivre pour les intégrer à Azure AD.

- [Akamai Enterprise Application Access (EAA)](../saas-apps/akamai-tutorial.md)
- [Citrix Application Deliver Controller (ADC)](../saas-apps/citrix-netscaler-tutorial.md) (anciennement Citrix Netscaler)
- [F5 Big-IP APM](../saas-apps/headerf5-tutorial.md)
- [Zscaler Private Access (ZPA)](../saas-apps/zscalerprivateaccess-tutorial.md)

## <a name="5-integrate-apps-your-developers-build"></a>5. Intégrer des applications créées par vos équipes de développeurs

En ce qui concerne les applications produites au sein de votre entreprise, vos développeurs peuvent utiliser la [Plateforme d’identité Microsoft](../develop/index.yml) afin d’implémenter l’authentification et l’autorisation. Les applications intégrées à la plateforme sont [inscrites auprès d’Azure AD](../develop/quickstart-register-app.md) et gérées comme n’importe quelle autre application de votre portefeuille.

Les développeurs peuvent utiliser la plateforme pour les applications à usage interne et les applications orientées client, mais les avantages conférés par la plateforme ne s’arrêtent pas là. [Les bibliothèques d’authentification Microsoft (MSAL)](../develop/msal-overview.md), qui font partie de la plateforme, permettent aux développeurs de mettre en place des expériences modernes, telles que l’authentification multifacteur et l’utilisation de clés de sécurité, pour accéder à leurs applications sans avoir à les implémenter eux-mêmes. En outre, les applications intégrées à la plate-forme d’identité de Microsoft peuvent accéder à [Microsoft Graph](../develop/microsoft-graph-intro.md) - un point de terminaison d’API unifié fournissant les données Microsoft 365 qui décrivent les modèles de productivité, d’identité et de sécurité dans une organisation. Les développeurs peuvent utiliser ces informations pour implémenter des fonctionnalités qui augmentent la productivité de vos utilisateurs. Par exemple, en identifiant les personnes avec lesquelles l’utilisateur a récemment interagi et en les mettant en avant dans l’interface utilisateur de l’application.

Nous avons une [série de vidéos](https://www.youtube.com/watch?v=zjezqZPPOfc&amp;list=PLLasX02E8BPBxGouWlJV-u-XZWOc2RkiX) qui offre une introduction complète à la plateforme, ainsi que [de nombreux exemples de code](../develop/sample-v2-code.md) dans les langages et les plateformes pris en charge.

## <a name="next-steps"></a>Étapes suivantes

- [Ressources pour la migration d’applications vers Azure Active Directory](../manage-apps/migration-resources.md)