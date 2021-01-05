---
ms.openlocfilehash: c400856546142353a7294a03fce6bbff1c258cc0
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95555571"
---
Dans Azure Active Directory (Azure AD), le terme **approvisionnement de l’application** désigne la création automatique des identités et rôles des utilisateurs dans les applications cloud ([SaaS](https://azure.microsoft.com/overview/what-is-saas/)) auxquelles les utilisateurs ont besoin d’accéder. En plus de créer des identités utilisateur, l’approvisionnement automatique comprend la maintenance et la suppression d’identités utilisateur en cas de modification de l’état ou des rôles. Les scénarios courants incluent la configuration d’un utilisateur Azure AD dans des applications telles que [Dropbox](../articles/active-directory/saas-apps/dropboxforbusiness-provisioning-tutorial.md), [Salesforce](../articles/active-directory/saas-apps/salesforce-provisioning-tutorial.md), [ServiceNow](../articles/active-directory/saas-apps/servicenow-provisioning-tutorial.md), et bien plus encore.

![Schéma de présentation de l’approvisionnement](./media/active-directory-app-provisioning/provisioning-overview.png)

Cette fonctionnalité vous permet de :

- **Automatiser l’approvisionnement :** Créez automatiquement des comptes dans les systèmes adéquats pour les nouvelles personnes rejoignant votre équipe ou votre organisation.
- **Automatiser le déprovisionnement :** Désactivez automatiquement les comptes dans les systèmes adéquats lorsque des personnes quittent votre équipe ou votre organisation.
- **Synchroniser les données entre les systèmes :** Vérifiez que les identités dans vos applications et systèmes sont tenues à jour selon les modifications apportées au répertoire ou à votre système de gestion des ressources humaines.
- **Approvisionner des groupes :** Approvisionnez des groupes pour les applications qui les prennent en charge.
- **Régir l’accès :** Surveillez et auditez les personnes qui ont été approvisionnées dans vos applications.
- **Déployer en toute transparence dans les scénarios « brown field » :** Faites correspondre les identités existantes entre les systèmes et facilitez l’intégration, même lorsque les utilisateurs existent déjà dans le système cible.
- **Utiliser une personnalisation riche :** Tirez parti des mappages d’attributs personnalisables qui définissent les données utilisateurs qui doivent circuler entre le système source et le système cible.
- **Obtenir des alertes pour les événements critiques :** Le service d’approvisionnement fournit des alertes pour les événements critiques et permet une intégration de Log Analytics dans laquelle vous pouvez définir des alertes personnalisées pour répondre aux besoins de votre entreprise.

## <a name="benefits-of-automatic-provisioning"></a>Avantages de l’approvisionnement automatique

À mesure que le nombre d’applications utilisées dans les organisations modernes continue de croître, les administrateurs informatiques sont chargés de la gestion des accès à l’échelle. Des normes telles que SAML (Security Assertions Markup Language) ou OIDC (Open ID Connect) permettent aux administrateurs de configurer rapidement l’authentification unique (SSO), mais l’accès requiert également que les utilisateurs soient approvisionnés dans l’application. Pour de nombreux administrateurs, l’approvisionnement consiste à créer manuellement chaque compte d’utilisateur ou à télécharger des fichiers CSV chaque semaine, mais ces processus prennent du temps, sont coûteux et peuvent engendrer des erreurs. Des solutions telles que SAML juste-à-temps (JIT) ont été adoptées pour automatiser l’approvisionnement, mais les entreprises ont également besoin d’une solution pour déprovisionner des utilisateurs lorsqu’ils quittent l’organisation ou n’ont plus besoin d’accéder à certaines applications en fonction du changement de rôle.

Voici quelques-unes des motivations courantes de l’utilisation de l’approvisionnement automatique :

- Optimisation de l’efficacité et de la précision des processus d’approvisionnement.
- Économies sur les coûts liés à l’hébergement et à la gestion de scripts et de solutions d’approvisionnement personnalisés.
- Sécurisation de votre organisation en supprimant instantanément les identités des utilisateurs des applications SaaS lorsqu’ils quittent l’organisation.
- Importation facile de nombreux utilisateurs dans un système ou une application SaaS spécifique.
- Disposer d’un ensemble unique de stratégies pour déterminer qui est approvisionné et qui peut se connecter à une application.

L’approvisionnement d’utilisateurs d’Azure AD peut vous aider à relever ces défis. Pour en savoir plus sur la façon dont les clients utilisent l’approvisionnement d’utilisateurs Azure AD, vous pouvez lire l’[étude de cas ASOS](https://aka.ms/asoscasestudy). La vidéo ci-dessous offre une vue d’ensemble de l’approvisionnement d’utilisateurs dans Azure AD :

> [!VIDEO https://www.youtube.com/embed/_ZjARPpI6NI]

## <a name="what-applications-and-systems-can-i-use-with-azure-ad-automatic-user-provisioning"></a>Quels applications et systèmes puis-je utiliser avec l’approvisionnement d’utilisateurs automatique d’Azure AD ?

Azure AD offre une prise en charge préintégrée de plusieurs applications SaaS et systèmes de gestion des ressources humaines connus, ainsi qu’une prise en charge générique des applications qui implémentent des parties spécifiques du [standard SCIM 2.0](https://techcommunity.microsoft.com/t5/Identity-Standards-Blog/Provisioning-with-SCIM-getting-started/ba-p/880010).

* **Applications préintégrées (applications SaaS de la galerie)** . Vous trouverez toutes les applications pour lesquelles Azure AD prend en charge un connecteur d’approvisionnement préintégré dans la [liste des tutoriels d’applications pour l’approvisionnement d’utilisateurs](../articles/active-directory/saas-apps/tutorial-list.md). Les applications préintégrées répertoriées dans la galerie utilisent généralement des API de gestion des utilisateurs SCIM 2.0 pour l’approvisionnement. 

   ![Logo Salesforce](./media/active-directory-app-provisioning/gallery-app-logos.png)

   Si vous souhaitez demander une nouvelle application pour l’approvisionnement, vous pouvez [demander que votre application soit intégrée à notre galerie d’applications](../articles/active-directory/develop/v2-howto-app-gallery-listing.md). Pour une demande d’approvisionnement d’utilisateurs, nous avons besoin que l’application dispose d’un point de terminaison compatible SCIM. Veuillez demander au fournisseur de l’application de suivre la norme SCIM afin que nous puissions intégrer rapidement l’application à notre plateforme.

* **Applications prenant en charge SCIM 2.0**. Pour plus d’informations sur la connexion générique d’applications qui implémentent des API de gestion des utilisateurs SCIM 2.0, consultez [Créer un point de terminaison SCIM et configurer l’attribution des utilisateurs](../articles/active-directory/app-provisioning/use-scim-to-provision-users-and-groups.md).

## <a name="what-is-system-for-cross-domain-identity-management-scim"></a>Qu’est-ce que SCIM (System for Cross-Domain Identity Management) ?

Pour aider à automatiser l’approvisionnement et le déprovisionnement, les applications exposent les API propriétaires d’utilisateurs et de groupes. Cependant, quiconque a essayé de gérer des utilisateurs dans plus d’une application vous dira que chaque application essaie d’effectuer les mêmes actions simples, telles que créer ou mettre à jour des utilisateurs, ajouter des utilisateurs à des groupes ou déprovisionner des utilisateurs. Pourtant, toutes ces actions simples sont implémentées un peu différemment, en utilisant des chemins d’accès de point de terminaison différents, des méthodes différentes pour spécifier les informations utilisateur et un schéma différent pour représenter chaque élément d’information.

Pour relever ces défis, la spécification SCIM fournit un schéma utilisateur commun pour aider les utilisateurs à se déplacer dans, hors et autour des applications. SCIM devient de facto la norme pour l’approvisionnement et, lorsqu’elle est utilisée conjointement avec des normes de fédération comme SAML ou OpenID Connect, fournit aux administrateurs une solution de bout en bout basée sur des normes pour la gestion des accès.

Pour obtenir des instructions détaillées sur le développement d’un point de terminaison SCIM afin d’automatiser le provisionnement et le déprovisionnement d’utilisateurs et de groupes dans une application, consultez [Créer un point de terminaison SCIM et configurer l’attribution des utilisateurs](../articles/active-directory/app-provisioning/use-scim-to-provision-users-and-groups.md). Pour les applications pré-intégrées dans la galerie (Slack, Azure Databricks, Snowflake, etc.), vous pouvez ignorer la documentation du développeur et utiliser les tutoriels fournis [ici](../articles/active-directory/saas-apps/tutorial-list.md).

## <a name="manual-vs-automatic-provisioning"></a>Provisionnement manuel ou automatique

Les applications de la galerie Azure AD prennent en charge l’un de ces deux modes de provisionnement :

* Provisionnement **manuel** : utilisé lorsqu’il n’existe pas de connecteur de provisionnement Azure AD automatique pour l’application. Les comptes d’utilisateur doivent être créés manuellement, par exemple, directement dans le portail d’administration de l’application, ou en chargeant une feuille de calcul contenant les informations du compte d’utilisateur. Consultez la documentation fournie par l’application ou contactez le développeur de l’application pour déterminer les mécanismes disponibles.

* **Automatique** : dans le cadre d’un approvisionnement automatique, un connecteur d’approvisionnement Azure AD a été développé pour cette application. Vous devez suivre le tutoriel de configuration pour configurer le provisionnement de votre application. Vous trouverez les didacticiels spécifiques aux applications à la page [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](../articles/active-directory/saas-apps/tutorial-list.md).

Dans la galerie Azure AD, les applications qui prennent en charge le provisionnement automatique sont désignées par une icône de **provisionnement**. Passez à la nouvelle expérience de préversion de la galerie pour voir ces icônes (dans la bannière située en haut de la page **Ajouter une application**, sélectionnez le lien **Cliquez ici pour essayer la Galerie d’applications nouvelle et améliorée**).

![Icône de provisionnement dans la galerie d’applications](./media/active-directory-app-provisioning/browse-gallery.png)

Le mode de provisionnement pris en charge par une application est également visible sous l’onglet **Provisionnement** une fois que vous avez ajouté l’application à vos **Applications d’entreprise**.

## <a name="how-do-i-set-up-automatic-provisioning-to-an-application"></a>Comment configurer l’approvisionnement automatique pour une application ?

Pour les applications préintégrées figurant dans la galerie, des instructions pas à pas sont disponibles pour configurer l’approvisionnement automatique. Consultez la [liste des tutoriels pour les applications de galerie intégrées](../articles/active-directory/saas-apps/tutorial-list.md). La vidéo suivante montre comment configurer l’approvisionnement automatique d’utilisateurs pour SalesForce.

> [!VIDEO https://www.youtube.com/embed/pKzyts6kfrw]

Pour les autres applications qui prennent en charge SCIM 2.0, suivez les étapes décrites dans l’article [Créer un point de terminaison SCIM et configurer l’attribution des utilisateurs](../articles/active-directory/app-provisioning/use-scim-to-provision-users-and-groups.md).