---
title: Planifier un déploiement de révision d’accès Azure Active Directory
description: Guide de planification pour un déploiement réussi de révisions d’accès
services: active-directory
documentationCenter: ''
author: BarbaraSelden
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: a2e670525840b801df874e3f8dcbdbadc0955a12
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746301"
---
# <a name="planning-azure-active-directory-access-reviews-deployment"></a>Planification d’un déploiement de révision d’accès Azure Active Directory

Les [révisions d’accès Azure Active Directory (Azure AD)](access-reviews-overview.md) aident votre organisation à sécuriser le réseau en gérant le [cycle de vie de l’accès aux ressources](identity-governance-overview.md). Grâce aux révisions d’accès, vous pouvez :

* Planifier des révisions régulières ou effectuer des révisions ad hoc pour voir qui a accès à des ressources spécifiques, telles que des applications et des groupes

* Suivre les révisions à des fins d’analyse, de conformité ou de stratégie

* Déléguer des révisions à des administrateurs, propriétaires d’entreprise ou utilisateurs finaux spécifiques qui peuvent attester eux-mêmes de la nécessité d’un accès continu

* Utiliser ces informations pour déterminer efficacement si les utilisateurs doivent continuer à avoir accès

* Automatiser les résultats des révisions, tels que la suppression de l’accès des utilisateurs aux ressources

  ![Diagramme montrant le flux des révisions d’accès.](./media/deploy-access-review/1-planning-review.png)

Les révisions d’accès sont une capacité d’[Azure AD Identity Governance](identity-governance-overview.md). Les autres fonctionnalités sont la [gestion des droits d’utilisation](entitlement-management-overview.md), [Privileged Identity Management](../privileged-identity-management/pim-configure.md) et les [Conditions d’utilisation](../conditional-access/terms-of-use.md). Ensemble, ils aident les organisations à répondre à ces quatre questions :

* Quels utilisateurs doivent pouvoir accéder à quelles ressources ?

* Que font les utilisateurs de cet accès ?

* Un contrôle organisationnel efficace est-il disponible pour gérer l’accès ?

* Des auditeurs peuvent-ils vérifier que les contrôles fonctionnent ?

Il est essentiel de planifier votre déploiement de révision d’accès pour vous assurer d’atteindre la stratégie de gouvernance souhaitée pour les utilisateurs de votre organisation.

### <a name="key-benefits"></a>Principaux avantages

Les principaux avantages de l’activation des révisions d’accès sont les suivants :

* **Contrôle de la collaboration**. Les révisions d’accès permettent aux organisations de gérer l’accès à toutes les ressources dont les utilisateurs ont besoin. Lorsque leurs utilisateurs partagent et collaborent, les organisations peuvent être assurées que les informations sont disponibles uniquement parmi les utilisateurs autorisés.

* **Gestion des risques**. Les révisions d’accès offrent aux organisations un moyen d’examiner l’accès aux données et aux applications, ce qui réduit le risque de fuite de données. Cela inclut des capacités permettant d’examiner régulièrement l’accès d’un partenaire externe aux ressources de l’entreprise. 

* **Respect de la gouvernance et de la conformité**. Grâce aux révisions d’accès, vous pouvez régir et recertifier le cycle de vie de l’accès aux groupes, applications et sites. Vous pouvez contrôler le suivi des révisions à des fins de conformité ou pour les applications sensibles aux risques propres à votre organisation. 

* **Réduction des coûts**. Les révisions d’accès sont créées dans le cloud et fonctionnent en mode natif avec des ressources cloud telles que des groupes, des applications et des packages d’accès. L’utilisation de révisions d’accès est moins coûteuse que la création de vos propres outils ou la mise à niveau de vos outils locaux.

### <a name="training-resources"></a>Ressources de formation

Les vidéos suivantes peuvent être utiles lorsque vous vous familiarisez avec les révisions d’accès :

* [Que sont les révisions d’accès dans Azure AD ?](https://youtu.be/kDRjQQ22Wkk) (en anglais)

* [Comment créer des révisions d’accès dans Azure AD](https://youtu.be/6KB3TZ8Wi40) (en anglais)

* [Comment activer les révisions d’accès dans Azure AD](https://youtu.be/X1SL2uubx9M) (en anglais)

* [Comment revoir l’accès à l’aide de Mon Accès](https://youtu.be/tIKdQhdHLXU) (en anglais)

### <a name="licenses"></a>Licences

Vous avez besoin d’une licence Azure AD Premium (P2) valide pour chaque personne qui créeront ou effectueront des révisions d’accès, à l’exception des administrateurs généraux ou des administrateurs d’utilisateurs. Pour plus d’informations, consultez [Exigences de licence pour les révisions d’accès](access-reviews-overview.md).

Vous pouvez également avoir besoin d’autres fonctionnalités d’Identity Governance, telles que la [gestion du cycle de vie des droits d’utilisation](entitlement-management-overview.md) ou Privileged Identity Management. Dans ce cas, vous pouvez également avoir besoin de licences connexes. Pour plus d’informations, consultez [Tarification Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="plan-the-access-reviews-deployment-project"></a>Planifier le projet de déploiement des révisions d’accès

Tenez compte des besoins de votre organisation pour déterminer la stratégie de déploiement des révisions d’accès dans votre environnement.

### <a name="engage-the-right-stakeholders"></a>Impliquer les parties prenantes appropriées

Lorsque des projets technologiques échouent, cela est généralement dû à des attentes qui ne correspondent pas à l’impact, aux résultats et aux responsabilités réels. Pour éviter un tel cas de figure, [prenez soin de faire appel aux bonnes parties prenantes](../fundamentals/active-directory-deployment-plans.md) et à clarifier les rôles du projet.

Pour les révisions d’accès, vous inclurez probablement des représentants des équipes suivantes au sein de votre organisation :

* **L’administration informatique** gère votre infrastructure informatique et administre vos investissements cloud et vos applications SaaS (Software as a service). Cette équipe effectuera les opérations suivantes :

   * Revoir l’accès privilégié à l’infrastructure et aux applications, notamment Microsoft 365 et Azure AD.

   * Planifier et exécuter des révisions d’accès sur des groupes utilisés pour gérer des listes d’exceptions ou des projets pilotes informatiques afin de maintenir à jour les listes d’accès.

   * Veiller à ce que l’accès programmatique (par script) aux ressources par le biais de principaux de service est régi et examiné.

* **Les équipes de développement** créent et entretiennent des applications pour votre organisation. Cette équipe effectuera les opérations suivantes :

   * Contrôler qui peut accéder et gérer les composants des ressources SaaS, PaaS et IaaS qui composent les solutions développées.

   * Gérer les groupes qui peuvent accéder aux applications et aux outils pour le développement d’applications internes.

   * Exiger des identités privilégiées ayant accès à des logiciels de production ou des solutions hébergées pour vos clients.

* **Les unités commerciales** gèrent des projets et possèdent des applications. Cette équipe effectuera les opérations suivantes : 

   * Examiner, et approuver ou refuser l’accès des utilisateurs internes et externes aux groupes et aux applications.

   * Planifier et effectuer des révisions pour attester de l’accès continu des employés et des identités externes telles que les partenaires commerciaux.

* **La gouvernance d’entreprise** garantit que l’organisation suit la stratégie interne et respecte les réglementations. Cette équipe effectuera les opérations suivantes :

   * Demander ou programmer de nouvelles révisions d’accès.

   * Évaluer les processus et les procédures de révision de l’accès, y compris la documentation et la tenue de registres pour en assurer la conformité.

   * Examiner les résultats des révisions antérieures pour les ressources les plus critiques.

> [!NOTE]
> Pour les révisions nécessitant des évaluations manuelles, veillez à planifier les réviseurs et cycles de révision adéquats qui répondent à vos besoins en matière de stratégie et de conformité. Si les cycles de révision sont trop fréquents ou qu’il y a trop peu de réviseurs, la qualité peut être perdue et trop ou trop peu de personnes peuvent y accéder. 

### <a name="plan-communications"></a>Planifier les communications

La communication est essentielle à la réussite de tout nouveau processus métier. Communiquez de manière proactive avec les utilisateurs sur la façon dont leur expérience va changer, à quel moment cela va se faire et comment il est possible d’obtenir de l’aide en cas de problème. 

#### <a name="communicate-changes-in-accountability"></a>Communiquer les changements en matière de responsabilité

Les révisions d’accès prennent en charge le transfert de responsabilité de la révision et l’action sur l’accès continu aux propriétaires d’entreprise. Le découplage des décisions d’accès de l’équipe informatique permet de prendre des décisions plus précises en matière d’accès. Il s’agit d’un changement culturel dans l’obligation de rendre des comptes et la responsabilité du propriétaire des ressources. Communiquez cette modification de manière proactive et assurez-vous que les propriétaires des ressources sont formés et en mesure d’utiliser les informations pour prendre de bonnes décisions.

Il est clair que l’équipe informatique souhaite garder le contrôle de toutes les décisions d’accès relatives à l’infrastructure et des attributions de rôles privilégiés. 

#### <a name="customize-email-communication"></a>Personnaliser la communication par e-mail

Lorsque vous planifiez une révision, vous désignez les utilisateurs qui effectuent cette révision. Ces réviseurs reçoivent ensuite une notification par e-mail concernant les nouvelles révisions qui leur sont attribuées, ainsi que des rappels avant l’expiration d’une de ces révisions.

Les administrateurs peuvent choisir d’envoyer cette notification soit à mi-chemin avant l’expiration de la révision, soit un jour avant son expiration. 

L’e-mail envoyé aux réviseurs peut être personnalisé de façon à inclure un message bref personnalisé qui les encourage à agir sur la révision. Nous vous recommandons d’utiliser le texte supplémentaire pour :

* Inclure un message personnel adressé aux réviseurs, afin qu’ils sachent qu’il est envoyé par votre service informatique ou de conformité.

* Inclure un lien hypertexte ou une référence à des informations internes sur les attentes de la révision et des documents de référence ou de formation supplémentaires.

* Inclure un lien vers des instructions sur la façon d’[effectuer une auto-révision de l’accès](review-your-access.md). 

  ![E-mail envoyé au réviseur](./media/deploy-access-review/2-plan-reviewer-email.png)

Lors de la sélection de l’option Démarrer la révision, les réviseurs sont dirigés vers le [portail myAccess](https://myapplications.microsoft.com/) pour les révisions d’accès aux groupes et aux applications. Le portail leur donne une vue d’ensemble de tous les utilisateurs qui ont accès à la ressource qu’ils révisent, ainsi que des recommandations système basées sur les dernières informations de connexion et d’accès.

### <a name="plan-a-pilot"></a>Prévoir un pilote

Nous encourageons les clients à tester initialement les révisions d’accès avec un petit groupe et à cibler des ressources non critiques. Cette phase pilote peut vous aider à ajuster les processus et les communications en fonction des besoins et à augmenter la capacité des utilisateurs et des réviseurs à répondre aux exigences de conformité et de sécurité.

Dans votre pilote, nous vous recommandons ce qui suit :

* Commencez par les révisions dont les résultats ne sont pas appliqués automatiquement, et vous pourrez en contrôler les implications.

* Assurez-vous que tous les utilisateurs disposent d’adresses e-mail valides figurant dans Azure AD et qu’ils reçoivent des communications par e-mail pour prendre les mesures appropriées. 

* Documentez tout accès supprimé dans le cadre du pilote au cas où vous devriez le restaurer rapidement.

* Analysez les journaux d’audit pour vous assurer que tous les événements sont correctement audités.

Pour plus d’informations, consultez les [meilleures pratiques pour un pilote](../fundamentals/active-directory-deployment-plans.md).

## <a name="introduction-to-access-reviews"></a>Présentation des révisions d’accès

Cette section présente les concepts de révision d’accès que vous devez connaître avant de planifier vos révisions.

### <a name="what-resource-types-can-be-reviewed"></a>Quels types de ressources peuvent être révisés ?

Une fois que vous avez intégré les ressources de votre organisation à Azure AD (telles que les utilisateurs, les applications et les groupes), elles peuvent être gérées et révisées. 

Les cibles classiques à réviser sont les suivantes :

* [Applications intégrées à Azure AD pour l’authentification unique](../manage-apps/what-is-application-management.md) (par exemple, application SaaS ou métier).

* [Appartenance](../fundamentals/active-directory-manage-groups.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context) à un groupe (synchronisée avec Azure AD ou créée dans Azure AD ou Microsoft 365, y compris Microsoft Teams).

* [Package d’accès](./entitlement-management-overview.md) qui regroupe les ressources (groupes, applications et sites) en un seul package pour gérer l’accès.

* [Rôles Azure AD et rôles Ressource Azure](../privileged-identity-management/pim-resource-roles-assign-roles.md) tels que définis dans Privileged Identity Management.

### <a name="who-will-create-and-manage-access-reviews"></a>Qui va créer et gérer les révisions d’accès ?

Le rôle administratif requis pour créer, gérer ou lire une révision d’accès dépend du type de ressource examiné. Le tableau suivant indique les rôles requis pour chaque type de ressource :

| Type de ressource| Création et gestion des révisions d’accès (créateurs)| Lecture des résultats de la révision d’accès |
| - | - | -|
| Groupe ou application| Administrateur général <p>Administrateur d'utilisateurs| Créateurs et Administrateur de la sécurité |
| Rôles privilégiés dans Azure AD| Administrateur général <p>Administrateur de rôle privilégié| Créateurs <p>Lecteur de sécurité<p>Security Administrator |
| Rôles privilégiés dans Azure (ressources)| Administrateur général<p>Administrateur d'utilisateurs<p>Propriétaire de la ressource| Créateurs |
| Package d’accès| Administrateur général<p>Créateur de package d’accès| Administrateur général uniquement |


Pour plus d’informations, consultez [Autorisations des rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md).

### <a name="who-will-review-the-access-to-the-resource"></a>Qui va réviser l’accès à la ressource ?

Le créateur de la révision d’accès détermine au moment de la création qui effectuera la révision. Ce paramètre ne peut pas être modifié une fois la révision démarrée. Les réviseurs sont représentés par trois personnages :

* Les propriétaires de la ressource, qui sont les propriétaires d’entreprise de la ressource.

* Un ensemble de délégués sélectionnés individuellement, tels que choisis par l’administrateur des révisions d’accès.

* Les utilisateurs finaux qui attestent eux-mêmes leurs besoins en matière d’accès continu.

Lors de la création d’une révision d’accès, les administrateurs peuvent choisir un ou plusieurs réviseurs. Tous les réviseurs peuvent démarrer et effectuer une révision, en choisissant des utilisateurs pour un accès continu à une ressource ou en les supprimant. 

### <a name="components-of-an-access-review"></a>Composants d’une révision d’accès

Avant d’implémenter vos révisions d’accès, vous devez planifier les types de révisions pertinents pour votre organisation. Pour ce faire, vous devez prendre des décisions commerciales sur ce que vous souhaitez examiner et les actions à entreprendre en fonction de ces révisions.

Pour créer une stratégie de révision d’accès, vous devez disposer des informations suivantes :

* Quelles sont les ressources à examiner ?

* Quel est l’utilisateur dont l’accès est examiné ?

* Quelle est la fréquence de la révision ?

* Qui effectuera la révision ?

   * Comment ces personnes seront-elles informées de la révision ?

   * Quels sont les délais à respecter pour la révision ?

* Quelles actions automatiques doivent être appliquées en fonction de la révision ?

   * Que se passe-t-il si le réviseur ne répond pas à temps ?

* Quelles actions manuelles seront exécutées en fonction de la révision ?

* Quelles communications doivent être envoyées en fonction des actions effectuées ?


**Exemple de plan de révision d’accès**

| Composant| Value |
| - | - |
| **Ressources à réviser**| Accès à Microsoft Dynamics |
| **Fréquence de révision**| Chaque mois |
| **Qui effectue la révision**| Responsables de programme du groupe d’entreprise Dynamics |
| **Notification**| E-mail envoyé 24 heures avant la révision à l’alias Dynamics-Pms<p>Inclure un message personnalisé encourageant les réviseurs pour obtenir leur adhésion |
| **Durée**| 48 heures à partir de la notification |
|**Actions automatiques**| Supprimer l’accès à tout compte qui n’a pas de connexion interactive dans les 90 jours, en supprimant l’utilisateur du groupe de sécurité dynamics-access. <p>*Exécuter des actions si la révision n’est pas effectuée dans les temps.* |
| **Actions manuelles**| Si vous le souhaitez, les réviseurs peuvent procéder à l’approbation des suppressions avant l’action automatisée. |
| **Communications**| Envoyer aux utilisateurs (membres) internes qui sont supprimés un e-mail expliquant qu’ils sont supprimés et comment récupérer l’accès. |


 

### <a name="automate-actions-based-on-access-reviews"></a>Automatiser les actions basées sur les révisions d’accès

Vous pouvez choisir d’automatiser la suppression des accès en activant l’option Appliquer automatiquement les résultats à la ressource.

  ![Planification des révisions d’accès](./media/deploy-access-review/3-automate-actions-settings.png)

Une fois la révision terminée, les utilisateurs qui n’ont pas été approuvés par le réviseur seront automatiquement supprimés de la ressource, ou conserveront un accès continu. Cela peut signifier la suppression de leur appartenance à un groupe ou de leur attribution d’application ou la révocation de leur droit d’élévation à un rôle privilégié.

Accepter les recommandations

Les recommandations sont présentées aux réviseurs dans le cadre de l’expérience de réviseur et indiquent la dernière connexion d’une personne au locataire ou le dernier accès à une application. Ces informations aident les réviseurs à prendre la décision appropriée en matière d’accès. La sélection de l’option Accepter les recommandations suivra les recommandations de la révision d’accès. À la fin d’une révision d’accès, le système appliquera automatiquement ces recommandations pour les utilisateurs pour lesquels les réviseurs n’ont pas répondu.

Les recommandations sont basées sur les critères de la révision d’accès. Par exemple, si vous configurez la révision pour supprimer tout accès sans connexion interactive pendant 90 jours, il est recommandé de supprimer tous les utilisateurs qui correspondent à ce critère. Microsoft travaille continuellement à l’amélioration des recommandations. 

### <a name="review-guest-user-access"></a>Réviser l’accès des utilisateurs invités

Utilisez les révisions d’accès pour examiner et nettoyer les identités des partenaires des organisations externes. La configuration d’une révision par partenaire peut répondre aux exigences de conformité.

Les identités externes peuvent être autorisées à accéder aux ressources de l’entreprise par le biais de l’une des actions suivantes :

* Ajoutée à un groupe 

* Invitée dans Teams 

* Affectée à une application d’entreprise ou à un package d’accès

* Affectée à un rôle privilégié dans Azure AD ou dans un abonnement Azure

Voir l’[exemple de script](https://github.com/microsoft/access-reviews-samples/tree/master/ExternalIdentityUse). Le script indique où les identités externes invitées dans le locataire sont utilisées. Vous pouvez voir l’appartenance à un groupe, les attributions de rôles et les attributions d’applications de l’utilisateur externe dans Azure AD. Le script n’affiche aucune attribution en dehors d’Azure AD, par exemple l’attribution directe des droits aux ressources SharePoint, sans l’utilisation de groupes.

Lorsque vous créez une révision d’accès pour des groupes ou des applications, vous pouvez choisir de laisser le réviseur se concentrer sur Toute personne ayant un accès, ou sur Utilisateurs invités uniquement. En sélectionnant Utilisateurs invités uniquement, les réviseurs reçoivent une liste ciblée d’identités externes d’Azure AD B2B qui ont accès à la ressource.

 ![Examiner les utilisateurs invités](./media/deploy-access-review/4-review-guest-users-admin-ui.png)

> [!IMPORTANT]
> Cela n’inclut PAS les membres externes qui ont un userType de membre. Cela n’inclut pas non plus les utilisateurs invités en dehors d’Azure AD B2B Collaboration, par exemple ceux qui ont accès au contenu partagé directement par le biais de SharePoint.

## <a name="plan-access-reviews-for-access-packages"></a>Planifier des révisions d’accès pour les packages d’accès

[Les packages d’accès](entitlement-management-overview.md) peuvent considérablement simplifier votre stratégie de gouvernance et de révision d’accès. Un package d’accès regroupe toutes les ressources avec l’accès dont un utilisateur a besoin pour travailler sur un projet ou accomplir sa tâche. Par exemple, vous pouvez créer un package d’accès qui comprend toutes les applications dont les développeurs de votre organisation ont besoin, ou toutes les applications auxquelles les utilisateurs externes doivent avoir accès. Un administrateur ou un gestionnaire délégué de package d’accès regroupe ensuite les ressources (groupes ou applications) et les rôles dont les utilisateurs ont besoin pour ces ressources.

Lors de [la création d’un package d’accès](entitlement-management-access-package-create.md), vous pouvez créer une ou plusieurs stratégies d’accès qui définissent les conditions auxquelles les utilisateurs peuvent demander un package d’accès, le processus d’approbation et la fréquence à laquelle une personne doit redemander l’accès. Les révisions d’accès sont configurées lors de la création ou de la modification d’une stratégie de package d’accès.

Ouvrez l’onglet Cycle de vie et faites défiler jusqu’à Révisions d’accès.

 ![Capture d’écran montrant l’option « Modifier la stratégie » sous l’onglet « Cycle de vie ».](./media/deploy-access-review/5-plan-access-packages-admin-ui.png)

## <a name="plan-access-reviews-for-groups"></a>Planifier des révisions d’accès pour les groupes

Outre les packages d’accès, la révision de l’appartenance à un groupe est la méthode la plus efficace pour régir l’accès. Nous vous recommandons d’attribuer l’accès aux ressources via [des groupes de sécurité ou des groupes Microsoft 365](../fundamentals/active-directory-manage-groups.md), et d’ajouter des utilisateurs à ces groupes pour qu’ils en obtiennent l’accès.

Un groupe peut se voir accorder l’accès à toutes les ressources appropriées. Vous pouvez attribuer au groupe un accès à des ressources individuelles ou à un package d’accès qui regroupe des applications et d’autres ressources. Grâce à cette méthode, vous pouvez réviser l’accès au groupe plutôt que l’accès d’un individu à chaque application. 

L’appartenance au groupe peut être révisée par : 

* Administrateurs

* Propriétaires de groupe

* Certains utilisateurs, capacité de révision déléguée lors de la création de la révision

* Membres du groupe, en attestant d’eux-mêmes

### <a name="group-ownership"></a>Propriété de groupe

Nous recommandons aux propriétaires de groupes de réviser leur appartenance, car ils sont les mieux placés pour savoir qui a besoin d’un accès. La propriété des groupes diffère selon le type de groupe :

Les groupes créés dans Microsoft 365 et Azure AD ont un ou plusieurs propriétaires bien définis. Dans la plupart des cas, ces propriétaires font de parfaits réviseurs pour leurs propres groupes, car ils savent qui doit y avoir accès. 

Par exemple, Microsoft Teams utilise les groupes Microsoft 365 comme modèle d’autorisation sous-jacent pour accorder aux utilisateurs l’accès aux ressources qui se trouvent dans SharePoint, Exchange, OneNote ou d’autres services Microsoft 365. Le créateur de l’équipe devient automatiquement propriétaire et doit être responsable d’attester l’appartenance de ce groupe. 

Les groupes créés manuellement dans le portail Azure AD ou par script via Microsoft Graph n’ont pas nécessairement de propriétaires définis. Nous vous recommandons de les définir par le biais du portail Azure AD dans la section « Propriétaires » du groupe ou par le biais de Graph.

Les groupes qui sont synchronisés à partir du service Active Directory local ne peuvent pas avoir de propriétaire dans Azure AD. Lors de la création d’une révision d’accès, vous devez sélectionner les personnes qui sont les plus aptes à décider de leur appartenance.

> [!NOTE]
> Nous vous recommandons de définir des stratégies d’entreprise qui définissent la façon dont les groupes sont créés afin de garantir la propriété et la responsabilité claires du groupe pour une révision régulière de ses membres. 

### <a name="review-membership-of-exclusion-groups-in-ca-policies"></a>Examiner l’appartenance des groupes d’exclusion dans les stratégies d’accès conditionnel 

Il peut arriver que les stratégies d’accès conditionnel conçues pour sécuriser votre réseau ne s’appliquent pas à tous les utilisateurs. Par exemple, une stratégie d’accès conditionnel qui autorise uniquement les utilisateurs à se connecter à partir du réseau de l’entreprise ne peut pas s’appliquer à l’équipe de vente, qui voyage beaucoup. Dans ce cas, les membres de l’équipe de vente seraient placés dans un groupe, et ce groupe serait exclu de la stratégie d’accès conditionnel. 

Révisez régulièrement l’appartenance à un tel groupe, car l’exclusion représente un risque potentiel si les mauvais membres sont exclus de l’exigence.

Vous pouvez [utiliser les révisions d’accès Azure AD pour gérer les utilisateurs exclus des stratégies d’accès conditionnel](conditional-access-exclusion.md).

### <a name="review-external-users-group-memberships"></a>Réviser les appartenances aux groupes des utilisateurs externes

Pour minimiser le travail manuel et les erreurs potentielles associées, envisagez d’utiliser des [groupes dynamiques](../enterprise-users/groups-create-rule.md) pour attribuer l’appartenance à un groupe en fonction des attributs d’un utilisateur. Vous pouvez créer un ou plusieurs groupes dynamiques pour les utilisateurs externes. Le commanditaire interne peut agir en tant que réviseur pour l’appartenance au groupe. 

Remarque : Les utilisateurs externes qui sont supprimés d’un groupe à la suite d’une révision d’accès ne sont pas supprimés du locataire. 

Ils peuvent être supprimés d’un locataire manuellement ou par le biais d’un script.

### <a name="review-access-to-on-premises-groups"></a>Réviser l’accès aux groupes locaux

Les révisions d’accès ne peuvent pas modifier l’appartenance au groupe des groupes que vous synchronisez à partir d’un emplacement local avec [Azure AD Connect](../hybrid/whatis-azure-ad-connect.md). Cela est dû au fait que la source d’autorité est locale.

Vous pouvez toujours utiliser les révisions d’accès pour planifier et gérer régulièrement des révisions de groupes locaux. Les réviseurs effectuent ensuite des actions dans le groupe local. Cette stratégie permet de garder les révisions d’accès comme outil pour toutes les révisions.

Vous pouvez utiliser les résultats d’une révision d’accès sur les groupes locaux et les traiter plus avant, de l’une des manières suivantes :

* Télécharger le rapport CSV de la révision d’accès et effectuer des actions manuelles.

* Utiliser Microsoft Graph pour accéder par programmation aux résultats et aux décisions dans les révisions d’accès terminées.

Par exemple, pour accéder aux résultats d’un groupe géré par Windows AD, utilisez cet [exemple de script PowerShell](https://github.com/microsoft/access-reviews-samples/tree/master/AzureADAccessReviewsOnPremises). Le script décrit les appels Graph requis et exporte les commandes Windows AD-PowerShell pour effectuer les modifications.

## <a name="plan-access-reviews-for-applications"></a>Planifier des révisions d’accès pour les applications 

Lorsque vous révisez l’accès à une application, vous révisez l’accès des employés et des identités externes aux informations et aux données contenues dans l’application. Choisissez de réviser une application lorsque vous avez besoin de savoir qui a accès à une application spécifique, plutôt qu’à un package d’accès ou à un groupe. 

Nous vous recommandons de planifier des révisions d’applications dans les scénarios suivants :

* Les utilisateurs bénéficient d’un accès direct à l’application (en dehors d’un groupe ou d’un package d’accès).

* L’application expose des informations critiques ou sensibles.

* L’application a des exigences de conformité spécifiques dont vous devez attester.

* Vous soupçonnez un accès inapproprié.

Afin de créer des révisions d’accès pour une application, définissez la propriété « Affectation d’utilisateurs requise ? » sur Oui. Si elle est définie sur Non, tous les utilisateurs de votre répertoire, y compris les identités externes, peuvent accéder à l’application et vous ne pouvez pas réviser l’accès à l’application. 

 ![Planifier les affectations d’applications](./media/deploy-access-review/6-plan-applications-assignment-required.png)

Vous devez ensuite [affecter les utilisateurs et les groupes](../manage-apps/assign-user-or-group-access-portal.md) dont vous voulez qu’ils aient accès. 

### <a name="reviewers-for-an-application"></a>Réviseurs pour une application

Les révisions d’accès peuvent porter sur les membres d’un groupe ou sur les utilisateurs qui ont été assignés à une application. Les applications dans Azure AD n’ont pas nécessairement de propriétaire, c’est pourquoi il n’est pas possible de sélectionner le propriétaire de l’application comme réviseur. Vous pouvez en outre étendre la portée d’une révision pour n’examiner que les utilisateurs invités affectés à l’application, plutôt que d’examiner tous les accès.

## <a name="plan-review-of-azure-ad-and-azure-resource-roles"></a>Planifier la révision des rôles Azure AD et Ressource Azure

[Privileged Identity Management (PIM)](../privileged-identity-management/pim-configure.md) simplifie la gestion des accès privilégiés aux ressources dans Azure AD. Cela permet de réduire considérablement la liste des rôles privilégiés, aussi bien dans [Azure AD](../roles/permissions-reference.md) que dans les [ressources Azure](../../role-based-access-control/built-in-roles.md), et d’accroître la sécurité globale du répertoire.

Les révisions d’accès permettent aux réviseurs d’attester si les utilisateurs doivent toujours appartenir à un rôle. Tout comme les révisions d’accès pour les packages d’accès, les révisions des rôles Azure AD et des ressources Azure sont intégrées dans l’expérience de l’utilisateur administrateur PIM. Nous vous recommandons de réviser régulièrement les attributions de rôle suivantes :

* Administrateur général

* Administrateur d'utilisateurs

* Administrateur d’authentification privilégié

* Administrateur de l’accès conditionnel

* Security Administrator

* Tous les rôles Administration de service Microsoft 365 et Dynamics

Les rôles sélectionnés ici incluent le rôle permanent et le rôle éligible. 

Dans la section Réviseurs, sélectionnez une ou plusieurs personnes pour vérifier tous les utilisateurs. Sinon, vous pouvez choisir de faire en sorte que les membres vérifient leur propre accès.

 ![Modifier la stratégie](./media/deploy-access-review/7-plan-azure-resources-reviewers-selection.png)

## <a name="deploy-access-reviews"></a>Déployer des révisions d’accès

Une fois que vous avez préparé une stratégie et un plan pour réviser l’accès aux ressources intégrées à Azure AD, déployez et gérez les révisions à l’aide des ressources ci-dessous.

### <a name="review-access-packages"></a>Réviser des packages d’accès

Pour réduire le risque d’un accès obsolète, les administrateurs peuvent activer les révisions périodiques des utilisateurs qui ont des affectations actives à un package d’accès. Suivez les instructions du lien ci-dessous :

| Articles sur les procédures| Description |
| - | - |
| [Créer des révisions d’accès](entitlement-management-access-reviews-create.md)| Activation des révisions du package d’accès. |
| [Effectuer des révisions d’accès](entitlement-management-access-reviews-review-access.md)| Réalisation de révisions d’accès pour d’autres utilisateurs affectés à un package d’accès. |
| [Réviser soi-même un package d’accès attribué](entitlement-management-access-reviews-self-review.md)| Auto-révision d’un package d’accès attribué. |


> [!NOTE]
> Les utilisateurs finaux qui se révisent eux-mêmes et déclarent ne plus avoir besoin d’un accès ne sont pas immédiatement supprimés du package d’accès. Ils sont supprimés du package d’accès à la fin de la révision ou si un administrateur arrête la révision.

### <a name="review-groups-and-apps"></a>Réviser des groupes et des applications

Les besoins d’accès aux groupes et aux applications des employés et des invités sont susceptibles d’évoluer dans le temps. Afin de réduire les risques associés aux attributions d’accès obsolètes, les administrateurs peuvent créer des révisions d’accès pour les membres de groupes ou pour l’accès aux applications. Suivez les instructions du lien ci-dessous :

| Articles sur les procédures| Description |
| - | - |
| [Créer des révisions d’accès](create-access-review.md)| Création d’une ou de plusieurs révisions d’accès pour les membres de groupes ou pour l’accès aux applications. |
| [Effectuer des révisions d’accès](perform-access-review.md)| Réalisation d’une révision d’accès pour les membres d’un groupe ou des utilisateurs ayant accès à une application. |
| [Réviser soi-même un accès](review-your-access.md)| Les membres révisent leur propre accès à un groupe ou une application. |
| [Effectuer une révision d’accès](complete-access-review.md)| Affichage d’une révision d’accès et application des résultats. |
| [Entreprendre des actions pour les groupes locaux](https://github.com/microsoft/access-reviews-samples/tree/master/AzureADAccessReviewsOnPremises)| Exemple de script PowerShell pour agir sur les révisions d’accès pour les groupes locaux. |


### <a name="review-azure-ad-roles"></a>Réviser les rôles Azure AD

Pour réduire les risques associés aux attributions de rôles obsolètes, vous devez régulièrement réviser l’accès aux rôles Azure AD privilégiés.

![Capture d’écran montrant la liste « Réviser l’appartenance » des rôles Azure AD.](./media/deploy-access-review/8-review-azure-ad-roles-picker.png)

Suivez les instructions des liens ci-dessous :

| Articles sur les procédures | Description |
| - | - |
 [Créer des révisions d’accès](../privileged-identity-management/pim-how-to-start-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Création de révisions d’accès pour les rôles Azure AD privilégiés dans PIM. |
| [Réviser soi-même un accès](../privileged-identity-management/pim-how-to-perform-security-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Si vous êtes affecté à un rôle d’administrateur, approbation ou refus de l’accès à votre rôle. |
| [Effectuer une révision d’accès](../privileged-identity-management/pim-how-to-complete-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Affichage d’une révision d’accès et application des résultats. |


### <a name="review-azure-resource-roles"></a>Réviser les rôles Ressource Azure

Pour réduire les risques associés aux attributions de rôles obsolètes, vous devez régulièrement réviser l’accès aux rôles Ressource Azure. 

![Réviser les rôles Azure AD](./media/deploy-access-review/9-review-azure-roles-picker.png)

Suivez les instructions des liens ci-dessous :

| Articles sur les procédures| Description |
| - | -|
| [Créer des révisions d’accès](../privileged-identity-management/pim-resource-roles-start-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Création de révisions d’accès pour les rôles Ressource Azure privilégiés dans PIM. |
| [Réviser soi-même un accès](../privileged-identity-management/pim-resource-roles-perform-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Si vous êtes affecté à un rôle d’administrateur, approbation ou refus de l’accès à votre rôle. |
| [Effectuer une révision d’accès](../privileged-identity-management/pim-resource-roles-complete-access-review.md?toc=%2fazure%2factive-directory%2fgovernance%2ftoc.json)| Affichage d’une révision d’accès et application des résultats. |


## <a name="use-the-access-reviews-api"></a>Utiliser l’API des révisions d’accès

Consultez les [méthodes de l’API Graph](/graph/api/resources/accessreviews-root?view=graph-rest-beta) et les [contrôles d’autorisation de rôle et de permission d’application](/graph/api/resources/accessreviews-root?view=graph-rest-beta) pour interagir avec les ressources révisables et les gérer. Les méthodes de révision d’accès dans l’API Microsoft Graph sont disponibles à la fois pour les applications et les utilisateurs. Lors de l’exécution de scripts dans le contexte de l’application, le compte utilisé pour exécuter l’API (le principal du service) doit obtenir l’autorisation « AccessReview.Read.All » pour interroger les informations de révision d’accès.

Les tâches les plus courantes à automatiser à l’aide de l’API Graph pour les révisions d’accès sont les suivantes :

* Créer et démarrer une révision d’accès

* Terminer manuellement une révision d’accès avant sa fin planifiée

* Répertorier toutes les révisions d’accès en cours d’exécution et leur état

* Consulter l’historique d’une série de révisions et les décisions et actions prises à chaque révision

* Collecter les décisions d’une révision d’accès

* Collecter les décisions des révisions terminées où le réviseur a pris une décision différente de celle recommandée par le système

Lorsque vous créez de nouvelles requêtes de l’API Graph pour l’automatisation, nous vous recommandons d’utiliser [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer). Vous pouvez générer et explorer vos requêtes Graph avant de les placer dans des scripts et du code. Cela peut vous aider à itérer rapidement votre requête afin d’obtenir exactement les résultats que vous recherchez, sans changer le code de votre script.

## <a name="monitor-access-reviews"></a>Surveiller les révisions d’accès

Les activités de révision d’accès sont enregistrées et disponibles dans les [journaux d’audit d’Azure AD](../reports-monitoring/concept-audit-logs.md). Vous pouvez filtrer les données d’audit par catégorie, type d’activité et plage de dates. Voici un exemple de requête :

| Category| Policy |
| - | - |
| Type d’activité| Créer une révision d’accès |
| | Mettre à jour une révision d’accès |
| | Révision d’accès terminée |
| | Supprimer une révision d’accès |
| | Approuver la décision |
| | Refuser la décision |
| | Réinitialiser la décision |
| | Appliquer la décision |
| Plage de dates| sept jours |


Pour les requêtes plus avancées et l’analyse des révisions d’accès, ainsi que pour le suivi des modifications et l’achèvement des révisions, nous vous recommandons d’exporter vos journaux d’audit Azure AD vers [Azure Log Analytics](../reports-monitoring/quickstart-azure-monitor-route-logs-to-storage-account.md) ou Azure Event Hubs. Lorsque vous les stockez dans Azure Log Analytics, vous pouvez utiliser le [puissant langage d’analytique](../reports-monitoring/howto-analyze-activity-logs-log-analytics.md) et créer vos propres tableaux de bord.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les technologies connexes ci-dessous.

* [Présentation de la gestion des droits d’utilisation Azure AD](entitlement-management-overview.md)

* [Qu’est-ce qu’Azure AD Privileged Identity Management ?](../privileged-identity-management/pim-configure.md)