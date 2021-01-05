---
title: Gestion de l’accès aux applications à l’aide d’Azure AD
description: Décrit comment Azure Active Directory permet aux organisations de spécifier les applications auxquelles chaque utilisateur a accès.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 05/16/2017
ms.author: kenwith
ms.openlocfilehash: c73a5ccd9e16fb5efd9fefca6253701f942c3202
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92367786"
---
# <a name="managing-access-to-apps"></a>Gestion de l’accès aux applications

La gestion de l’accès, l’évaluation de l’utilisation et la création de rapports en continu demeurent un défi quand une application vient d’être intégrée au système d’identité de votre organisation. Dans de nombreux cas, les administrateurs informatiques ou le support technique doivent en permanence jouer un rôle actif dans la gestion de l’accès à vos applications. Parfois, l’affectation est effectuée par une équipe informatique générale ou rattachée à une division. Souvent, il revient au décideur d’entreprise de décider d’une affectation, que l’équipe informatique ne pourra mettre en œuvre qu’avec son approbation.  D’autres organisations investissent dans l’intégration à un système automatisé de gestion des identités et des accès, tel que le contrôle d’accès en fonction du rôle (RBAC) ou le contrôle d’accès en fonction de l’attribut (ABAC). L’intégration et le développement de règles ont tous deux tendance à être spécialisés et coûteux. Quelle que soit la méthode de gestion, l’analyse ou la création de rapports suppose un investissement distinct, coûteux et complexe.

## <a name="how-does-azure-active-directory-help"></a>En quoi Azure Active Directory peut-il vous aider ?

Azure AD prend en charge une gestion complète de l’accès pour les applications configurées, permettant aux organisations d’accomplir facilement les stratégies d’accès appropriées, allant de l’affectation automatique basée sur l’attribut (scénarios ABAC ou RBAC) à la gestion des administrateurs en passant par la délégation. Grâce à Azure AD, vous pouvez facilement accomplir des stratégies complexes, en combinant plusieurs modèles de gestion pour une application unique, et pouvez même réutiliser les règles de gestion entre les applications avec le même public.

Avec Azure AD, la création de rapports d’utilisation et d’affectation est entièrement intégrée, permettant aux administrateurs de créer facilement des rapports sur l’état des affectations, les erreurs d’affectation et même sur l’utilisation.

### <a name="assigning-users-and-groups-to-an-app"></a>Attribution d’utilisateurs et de groupes à une application

L’affectation d’applications Azure AD se concentre sur deux modes d’affectation principaux :

* **Affectation individuelle** : un administrateur informatique détenant des autorisations d’administrateur général de l’annuaire peut sélectionner différents comptes d’utilisateurs et leur octroyer un accès à l’application.

* **Affectation basée sur le groupe (nécessite Azure AD Premium P1 ou P2)**  : un administrateur informatique détenant des autorisations d’administrateur général de l’annuaire peut affecter un groupe à l’application. L’accès dont bénéficie un utilisateur dépend de son appartenance éventuelle au groupe au moment où il essaie d’accéder à l’application. En d’autres termes, un administrateur peut créer efficacement une règle d’affectation indiquant « tout membre actuel du groupe affecté a accès à l’application ». Avec cette option d’affectation, les administrateurs peuvent tirer parti des options de gestion de groupe Azure AD, notamment des [groupes dynamiques basés sur l’attribut](../fundamentals/active-directory-groups-create-azure-portal.md), des groupes de systèmes externes (par exemple, Active Directory local ou Workday) ou des groupes gérés par un administrateur ou en libre-service. Un même groupe peut être facilement affecté à plusieurs applications ; de la sorte, celles qui présentent une affinité d’affectation peuvent partager des règles d’affectation, réduisant ainsi la complexité globale de la gestion. Notez que des appartenances à des groupes imbriquées ne sont pas prises en charge pour l’affectation basée sur le groupe à des applications à ce stade.

Grâce à ces deux modes d’affectation, les administrateurs peuvent mettre en œuvre toute approche de gestion d’affectation souhaitable.

### <a name="requiring-user-assignment-for-an-app"></a>Demande d’affectation d’utilisateurs pour une application

Avec certains types d’applications, vous avez la possibilité de [demander que des utilisateurs soient affectés à l’application](assign-user-or-group-access-portal.md#configure-an-application-to-require-user-assignment). En procédant ainsi, vous empêchez tous les utilisateurs de se connecter, à l’exception de ceux que vous affectez explicitement à l’application. Les types d’applications suivants prennent en charge cette option :

* Les applications configurées pour l’authentification unique (SSO) fédérée avec l’authentification basée sur SAML
* Les applications de proxy d’application qui utilisent la préauthentification Azure Active Directory
* Les applications créées sur la plateforme d’application Azure AD, utilisant l’authentification OAuth 2.0 / OpenID Connect après le consentement donné par un utilisateur ou un administrateur à cette application. Certaines applications d’entreprise offre un contrôle supplémentaire sur les personnes qui sont autorisées à se connecter.

Lorsque l’affectation d’utilisateurs n’est *pas obligatoire* , les utilisateurs non attribués ne voient pas l’application dans Mes applications, mais ils peuvent toujours se connecter à l’application elle-même (procédure également appelée « authentification initiée par le fournisseur de services »). Ils peuvent aussi utiliser l’ **URL d’accès utilisateur** dans la page **Propriétés** de l’application (action également appelée « authentification initiée par IDP »).

Pour certaines applications, l’option permettant de demander l’affectation d’utilisateurs n’est pas disponible dans les propriétés de l’application. Dans ce cas, vous pouvez utiliser PowerShell pour définir la propriété appRoleAssignmentRequired sur le principal du service.

### <a name="determining-the-user-experience-for-accessing-apps"></a>Détermination de l’expérience utilisateur pour l’accès aux applications

Azure AD offre [plusieurs moyens personnalisables qui permettent de déployer des applications](end-user-experiences.md) pour les utilisateurs finaux de votre organisation :

* Mes applications Azure AD
* Lanceur d'applications Microsoft 365
* Authentification directe pour les applications fédérées (service-pr)
* Liens ciblés vers des applications fédérées, avec mot de passe ou des applications existantes

Vous pouvez déterminer si les utilisateurs attribués à une application d'entreprise peuvent voir celle-ci dans le lanceur d'applications Microsoft 365.

## <a name="example-complex-application-assignment-with-azure-ad"></a>Exemple : Affectation d’application complexe avec Azure AD
Considérez une application comme Salesforce. Dans de nombreuses organisations, Salesforce est principalement utilisé par des équipes commerciales et marketing. Souvent, les membres de l’équipe marketing disposent d’un accès hautement privilégié à Salesforce, contrairement aux membres de l’équipe de ventes. Dans de nombreux cas, la majorité des travailleurs de l’information n’a qu’un accès limité à l’application. Les exceptions à ces règles compliquent les choses. C’est souvent la prérogative des équipes de direction marketing ou de ventes d’accorder un accès à un utilisateur ou de modifier son rôle indépendamment de ces règles génériques.

Avec Azure AD, les applications telles que Salesforce peuvent être préconfigurées pour l’authentification unique (SSO) et l’automatisation de l’approvisionnement. Dès que l’application est configurée, un administrateur peut créer et affecter des groupes appropriés une bonne fois pour toutes. Dans cet exemple, un administrateur peut exécuter les affectations suivantes :

* [groupes dynamiques](../fundamentals/active-directory-groups-create-azure-portal.md) peuvent être définis pour représenter automatiquement tous les membres des équipes de ventes et de marketing à l’aide d’attributs tels que le service ou le rôle :
  
  * Tous les membres de groupes marketing sont affectés au rôle « marketing » dans Salesforce.
  * Tous les membres de groupes de ventes sont affectés au rôle « ventes » dans Salesforce. Pour affiner les choses, plusieurs groupes représentant des équipes de ventes régionales affectées à différents rôles Salesforce peuvent être utilisés.

* Pour activer le mécanisme d’exception, un groupe libre-service peut être créé pour chaque rôle. Par exemple, le groupe « exception marketing dans Salesforce » peut être créé en tant que groupe libre-service. Le groupe peut être affecté au rôle marketing dans Salesforce, tandis que les membres de l’équipe de direction marketing peuvent être définis en tant que propriétaires. Les membres de l’équipe de direction marketing peuvent ajouter ou supprimer des utilisateurs, définir une stratégie de jonction ou même approuver ou refuser les demandes de jonction formulées par des utilisateurs spécifiques. Ce mécanisme repose sur une expérience de travailleur de l’information appropriée qui ne nécessite pas de formation spécialisée pour les propriétaires ou les membres.

Dans ce cas, tous les utilisateurs affectés sont automatiquement approvisionnés dans Salesforce ; quand ils sont ajoutés aux différents groupes, leur affectation de rôle est actualisée dans Salesforce. Les utilisateurs peuvent découvrir Salesforce et y accéder par le biais de Mes applications, de clients web Office, voire de la page de connexion à Salesforce de leur organisation. Les administrateurs peuvent facilement afficher l’état de l’utilisation et des affectations à l’aide de rapports d’Azure AD.

Les administrateurs peuvent utiliser l’ [accès conditionnel Azure AD](../conditional-access/concept-conditional-access-users-groups.md) pour définir des stratégies d’accès pour des rôles spécifiques. Ces stratégies peuvent indiquer si l’accès est autorisé en dehors de l’environnement de l’entreprise, et même inclure des exigences Multi-Factor Authentication ou liées aux appareils déterminant l’octroi de l’accès dans divers cas.

## <a name="access-to-microsoft-applications"></a>Accéder aux applications Microsoft

Les applications Microsoft (comme Exchange, SharePoint, Yammer, etc.) sont attribuées et gérées un peu différemment des applications SaaS tierces et des autres applications que vous intégrez à Azure AD pour l'authentification unique.

Il existe trois méthodes principales pour se connecter à une application publiée par Microsoft.

- Pour les applications de la suite Microsoft 365 ou d'autres suites payantes, les utilisateurs peuvent obtenir un accès par le biais de l' **attribution de licence** , soit directement sur leur compte d'utilisateur, soit par l'intermédiaire d'un groupe à l'aide de la fonctionnalité d'attribution de licence de groupe.
- Pour les applications publiées et fournies gratuitement par Microsoft ou un tiers, les utilisateurs peuvent obtenir un accès via le [consentement de l’utilisateur](configure-user-consent.md). Cela signifie qu’ils peuvent se connecter à l’application avec leur compte professionnel ou scolaire Azure AD et lui permettre d’accéder à certaines données de leur compte.
- Pour les applications publiées et fournies gratuitement par Microsoft ou par un tiers, les utilisateurs peuvent également obtenir un accès via le [consentement de l’administrateur](manage-consent-requests.md). Cela veut dire que l’administrateur a déterminé que l’application peut être utilisée par tous les membres de l’organisation. Dans ce cas, il se connecte à l’application avec un compte d’administrateur général et accorde l’accès à tous les membres de l’organisation.

Quelques applications combinent ces méthodes. Par exemple, certaines applications Microsoft font partie d'un abonnement Microsoft 365, mais exigent toujours le consentement.

Les utilisateurs peuvent accéder aux applications Microsoft 365 via leur portail Office 365. Vous pouvez également afficher ou masquer des applications Microsoft 365 dans Mes applications en [activant/désactivant la visibilité d'Office 365](hide-application-from-user-portal.md) dans les **Paramètres utilisateur** de votre annuaire. 

Comme avec les applications d’entreprise, vous pouvez [affecter des utilisateurs](assign-user-or-group-access-portal.md) à certaines applications Microsoft via le portail Azure ou, si l’option du portail n’est pas disponible, à l’aide de PowerShell.

## <a name="next-steps"></a>Étapes suivantes
* [Protection des applications avec un accès conditionnel](../conditional-access/concept-conditional-access-cloud-apps.md)
* [Gestion des groupes en libre service/accès aux applications en libre-service](../enterprise-users/groups-self-service-management.md)
