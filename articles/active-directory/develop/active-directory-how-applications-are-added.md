---
title: Comment et pourquoi les applications sont ajoutées à Azure AD
titleSuffix: Microsoft identity platform
description: Qu’implique l’ajout d’une application à Azure AD et comment en sommes-nous arrivés ici ?
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 12/01/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: lenalepa, sureshja
ms.openlocfilehash: 1f6fd0160988802e198ff9388cfeb3232b34b100
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861117"
---
# <a name="how-and-why-applications-are-added-to-azure-ad"></a>Comment et pourquoi les applications sont ajoutées à Azure AD

Il existe deux représentations des applications dans Azure AD :

* Les [objets d’application](app-objects-and-service-principals.md#application-object) : bien qu’il existe des [exceptions](#notes-and-exceptions), les objets d’application peuvent être considérés comme la définition d’une application.
* Les [principaux de service](app-objects-and-service-principals.md#service-principal-object) : ils peuvent être considérés comme une instance d’une application. En règle générale, les principaux de service référencent un objet d’application, et un objet d’application peut être référencé par plusieurs principaux de service sur plusieurs annuaires.

## <a name="what-are-application-objects-and-where-do-they-come-from"></a>À quoi correspondent les objets d’application et d’où viennent-ils ?

Vous pouvez gérer des [objets d’application](app-objects-and-service-principals.md#application-object) dans le portail Azure via [Inscriptions d’application](https://aka.ms/appregistrations). Les objets d’application décrivent l’application à Azure AD et peuvent être considérés comme la définition de l’application. Ils permettent au service de savoir comment émettre des jetons pour l’application, en fonction de ses paramètres. L’objet d’application existe uniquement dans son répertoire de base, même s’il s’agit d’une application mutualisée prenant en charge des principaux de service dans d’autres répertoires. L’objet d’application peut inclure les éléments suivants (ainsi que d’autres informations qui ne sont pas mentionnées ici) :

* Nom, logo et éditeur
* URI de redirection
* Secrets (clés symétriques et/ou asymétriques utilisées pour authentifier l’application)
* Dépendances d’API (OAuth)
* API/ressources/étendues publiées (OAuth)
* Rôles d'application (RBAC)
* Configuration et métadonnées de l’authentification unique
* Configuration et métadonnées du déploiement de l'utilisateur
* Configuration et métadonnées du proxy

Les objets d’application peuvent être créés via plusieurs méthodes, notamment :

* Via les inscriptions d’application dans le portail Azure
* Via la création d’une application à l’aide de Visual Studio et la configuration de celle-ci pour utiliser l’authentification Azure AD
* Lorsqu’un administrateur ajoute une application à partir de la galerie d’applications (ce qui crée également un principal de service)
* En utilisant l’API Microsoft Graph ou PowerShell pour créer une application
* Via beaucoup d’autres méthodes, notamment les différentes expériences de développement dans Azure et les expériences d’explorateur d’API dans tous les centres de développement

## <a name="what-are-service-principals-and-where-do-they-come-from"></a>À quoi correspondent les principaux de service et d’où proviennent-ils ?

Vous pouvez gérer les [principaux de service](app-objects-and-service-principals.md#service-principal-object) dans le portail Azure via les [ Applications d’entreprise](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps/menuId/). Ce sont les principaux de service qui gouvernent une application se connectant à Azure AD, et ils peuvent être considérés comme l’instance de l’application dans votre répertoire. Pour toute application donnée, le principal de service peut avoir au maximum un objet d’application (qui est inscrit dans un annuaire de base) et un ou plusieurs objets de principal de service représentant les instances de l’application dans tous les annuaires dans lesquels il agit. 

Le principal de service peut inclure :

* Une référence à un objet d’application via la propriété d’ID d’application
* Des enregistrements des attributions de rôle de l’application de l’utilisateur local et du groupe
* Des enregistrements des autorisations de l’utilisateur local et de l’administrateur accordées à l’application
  * Par exemple : autorisation pour l’application d’accéder à un e-mail d’utilisateur particulier
* Des enregistrements des stratégies locales, y compris une stratégie d’accès conditionnel
* Des enregistrements des paramètres locaux alternatifs pour une application
  * Revendication des règles de transformation
  * Mappages d'attributs (déploiement de l'utilisateur)
  * Rôles d’application spécifiques de l’annuaire (si l’application prend en charge les rôles personnalisés)
  * Logo ou nom spécifique de l’annuaire

À l’instar des objets d’application, les principaux de service peuvent être créés via plusieurs méthodes, notamment :

* Lorsque les utilisateurs se connectent à une application tierce intégrée à Azure AD
  * Lors de la connexion, les utilisateurs sont invités à autoriser l’application à accéder à leur profil et à effectuer d’autres actions. Dès que la première personne donne son consentement, le principal de service représentant l’application est ajouté à l’annuaire.
* Lorsque les utilisateurs se connectent aux services en ligne de Microsoft comme [Microsoft 365](https://products.office.com/)
  * Lorsque vous vous abonnez à Microsoft 365 ou commencez une version d’évaluation, un ou plusieurs principaux de service sont créés dans l’annuaire représentant les différents services qui sont utilisés pour transmettre toutes les fonctionnalités associées à Microsoft 365.
  * Certains services de Microsoft 365 tels que SharePoint créent des principaux de service sur une base continue, afin de sécuriser les communications entre les composants, y compris les flux de travail.
* Lorsqu’un administrateur ajoute une application à partir de la galerie d’applications (cette opération crée également un objet d’application sous-jacent)
* Lors de l’ajout d’une application pour utiliser le [Proxy d’application d’Azure AD](../manage-apps/application-proxy.md)
* Lors de la connexion d’une application pour l’authentification unique à l’aide de SAML ou de l’authentification unique (SSO) avec mot de passe
* Par programmation via l’API Microsoft Graph ou PowerShell

## <a name="how-are-application-objects-and-service-principals-related-to-each-other"></a>Quel est le lien entre les objets d’application et les principaux de service ?

Une application possède un objet d’application dans son annuaire de base qui est référencé par un ou plusieurs principaux de service dans chacun des annuaires où il opère (y compris l’annuaire de base de l’application).

![Illustre la relation entre les objets d’application et les principaux de service][apps_service_principals_directory]

Dans le schéma ci-dessus, Microsoft gère deux annuaires en interne (représentés à gauche), qu’il utilise pour publier des applications :

* Une pour Microsoft Apps (annuaire de services Microsoft)
* Un pour des applications tierces préintégrées (annuaire de la galerie d’applications)

Les fournisseurs/éditeurs d’applications qui s’intègrent à Azure AD doivent avoir un annuaire de publication (représenté à droite en tant que « Annuaire SaaS »).

Les applications que vous ajoutez vous-même (représentées en tant que **(Vos) applications** dans le schéma) incluent :

* Les applications que vous avez développées (intégrées à Azure AD)
* les applications que vous avez connectées à l'authentification unique ;
* Les applications que vous avez publiées à l’aide du proxy d’application Azure AD

### <a name="notes-and-exceptions"></a>Remarques et exceptions

* Tous les principaux de service ne pointent pas vers un objet d’application. Lorsqu’Azure AD a été créé, les services fournis aux applications étaient alors plus limités et le principal de service était suffisant pour établir une identité d’application. Le principal du service d'origine était plus proche en termes de forme du compte de service Windows Server Active Directory. Pour cette raison, il est toujours possible de créer des principaux de service via diverses méthodes, avec Azure AD PowerShell par exemple, sans d’abord créer un objet d’application. L’API Microsoft Graph requiert un objet d’application avant de pouvoir créer un principal de service.
* Actuellement, toutes les informations décrites ci-dessus sont exposées par programmation. Les éléments suivants sont uniquement disponibles dans l'interface utilisateur :
  * Revendication des règles de transformation
  * Mappages d'attributs (déploiement de l'utilisateur)
* Pour plus d’informations détaillées sur le principal de service et les objets d’application, consultez la documentation de référence sur l’API Microsoft Graph :
  * [Application](/graph/api/resources/application)
  * [Principal du service](/graph/api/resources/serviceprincipal?view=graph-rest-beta)

## <a name="why-do-applications-integrate-with-azure-ad"></a>Pourquoi les applications s’intègrent à Azure AD ?

Les applications sont ajoutées à Azure AD pour exploiter un ou plusieurs des services proposés par Azure AD, notamment :

* L’authentification et l’autorisation de l’application
* L’authentification et l’autorisation de l’utilisateur
* L’authentification unique à l’aide de la fédération ou du mot de passe
* La configuration et la synchronisation de l’utilisateur
* Le contrôle d’accès basé sur les rôles : utilisez le répertoire pour définir les rôles d’application, afin d’effectuer des vérifications d’autorisation basées sur les rôles dans une application
* Les services d’autorisation OAuth : utilisés par Microsoft 365 et d’autres applications Microsoft pour autoriser l’accès aux API/ressources
* La publication et le proxy d’applications : publiez une application sur Internet à partir d’un réseau privé
* Attributs d’extension de schéma d’annuaire : [Étendre le schéma du principal du service et des objets utilisateur](active-directory-schema-extensions.md) pour stocker des données supplémentaires dans Azure AD 

## <a name="who-has-permission-to-add-applications-to-my-azure-ad-instance"></a>Qui a l'autorisation d'ajouter des applications à mon instance Azure AD ?

Bien que certaines tâches puissent uniquement être effectuées par les administrateurs généraux (comme l’ajout d’applications depuis la galerie d’applications et la configuration d’une application pour utiliser le proxy d’application), par défaut tous les utilisateurs de votre annuaire ont des droits pour inscrire des objets d’application qu’ils sont en train de développer et décider des applications qu’ils partagent ou auxquelles ils donnent accès à leurs données organisationnelles via un consentement. Si une personne est le premier utilisateur de votre annuaire à se connecter à une application et à donner son consentement, un principal de service sera créé dans votre locataire ; sinon, les informations d’octroi du consentement seront stockées dans le principal de service existant.

Permettre aux utilisateurs d’inscrire des applications et de donner leur consentement peut, à première vue, sembler inquiétant, mais n’oubliez pas les points suivants :


* Les applications sont capables de tirer parti de Windows Server Active Directory pour l’authentification utilisateur depuis de nombreuses années sans que l’application ait besoin d’être inscrite ou enregistrée dans l’annuaire. Désormais, l’organisation disposera d’une visibilité améliorée sur le nombre précis d’applications qui utilisent l’annuaire, et pour quelles raisons.
* La délégation de ces responsabilités aux utilisateurs supprime le besoin d’avoir un processus de publication et d’inscription des applications piloté par un administrateur. Avec Active Directory Federation Services (ADFS), il était probable que l’administrateur doive ajouter une application en tant que partie de confiance pour le compte de ses développeurs. Les développeurs sont désormais libres.
* La connexion des utilisateurs à des applications à l’aide de leur compte d’organisation à des fins professionnelles est un point positif. Par la suite, s’ils quittent l’organisation, ils perdront automatiquement l’accès au compte qu’ils utilisaient pour cette application.
* Il est bon de disposer d'un enregistrement permettant de savoir avec quelle application les données ont été partagées. Les données sont plus que jamais transportables, et il est utile de disposer d’un enregistrement précisant qui a partagé quelles données, et à l’aide de quelles applications.
* Les propriétaires d’API qui utilisent Azure AD pour OAuth décident en détail des autorisations que les utilisateurs sont en mesure d’accorder aux applications et des autorisations nécessitant un administrateur pour les confirmer. Seuls les administrateurs peuvent donner leur consentement pour des étendues plus larges et des autorisations plus importantes. Le consentement de l’utilisateur se limite aux propres fonctionnalités et données de celui-ci.
* Quand un utilisateur ajoute ou autorise une application à accéder à ses données, l’événement peut être audité afin que vous puissiez afficher les rapports d’audit dans le portail Azure et ainsi déterminer de quelle façon une application a été ajoutée à l'annuaire.

Si vous souhaitez toujours empêcher les utilisateurs de votre annuaire d’inscrire des applications et de se connecter à des applications sans l’approbation d’un administrateur, vous avez la possibilité de modifier deux paramètres :

* Pour empêcher les utilisateurs de donner leur consentement pour leur propre compte :
  1. Dans le portail Azure, accédez à la section [Paramètres utilisateur](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/) sous Applications d’entreprise.
  2. Définissez le paramètre **Les utilisateurs peuvent autoriser les applications à accéder aux données de l’entreprise en leur nom** sur **Non**.
     
     > [!NOTE]
     > Si vous décidez de désactiver le consentement de l’utilisateur, un administrateur devra donner son consentement pour chaque nouvelle application qu’un utilisateur utilisera.

* Pour empêcher les utilisateurs d’inscrire leurs propres applications :
  1. Dans le portail Azure, accédez à la section [Paramètres utilisateur](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/UserSettings) sous Azure Active Directory
  2. Définissez le paramètre **Les utilisateurs peuvent inscrire des applications** sur **Non**.

> [!NOTE]
> Microsoft utilise la configuration par défaut, c’est-à-dire que les utilisateurs peuvent inscrire des applications et donner leur consentement à des applications pour leur propre compte.

<!--Image references-->
[apps_service_principals_directory]:../media/active-directory-how-applications-are-added/HowAppsAreAddedToAAD.jpg
