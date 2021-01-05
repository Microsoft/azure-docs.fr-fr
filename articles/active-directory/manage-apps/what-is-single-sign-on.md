---
title: Qu’est-ce que l’authentification unique Azure ?
description: Découvrez le fonctionnement de l’authentification unique (SSO) avec Azure Active Directory. Utilisez l’authentification unique afin d’éviter aux utilisateurs de mémoriser les mots de passe de toutes les applications. Utilisez également l’authentification unique pour simplifier l’administration de la gestion des comptes.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: overview
ms.date: 12/03/2019
ms.author: kenwith
ms.reviewer: arvindh, japere
ms.custom: contperf-fy21q1
ms.openlocfilehash: 4e27266688e39eadc8ae1f3d9afd84c42e1b524e
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031655"
---
# <a name="what-is-single-sign-on-sso"></a>Qu’est-ce que l’authentification unique ?

L’authentification unique signifie qu’un utilisateur n’a pas besoin de se connecter à chaque application qu’il utilise. Après que l’utilisateur s’est connecté, ses informations d’identification sont utilisées pour d’autres applications.

Si vous êtes un utilisateur final, vous ne vous souciez probablement pas beaucoup des détails de l’authentification unique. Vous souhaitez simplement utiliser les applications qui vous rendent productif sans avoir à saisir constamment votre mot de passe. Vous pouvez trouver vos applications à l’emplacement suivant : https://myapps.microsoft.com.
 
Si vous êtes un administrateur ou un professionnel de l’informatique, lisez ce qui suit pour en savoir plus sur de l’authentification unique et comment celle-ci est implémentée dans Azure.

## <a name="single-sign-on-basics"></a>Notions de base de l’authentification unique
L’authentification unique constitue un énorme bond en avant dans la manière dont les utilisateurs se connectent aux applications et les utilisent. Les systèmes d’authentification basés sur une authentification unique sont souvent considérés comme utilisant une « authentification moderne ». L’authentification moderne et l’authentification unique appartiennent à une catégorie de calcul appelée gestion des identités et des accès (IAM). Pour comprendre ce qui rend possible l’authentification unique, regardez cette vidéo.

Concepts de base de l’authentification : Principes de base | Azure Active Directory

> [!VIDEO https://www.youtube.com/embed/fbSVgC8nGz4]

## <a name="single-sign-on-with-web-applications"></a>Authentification unique avec des applications web
Les applications web sont incroyablement populaires. Les applications web sont hébergées par différentes entreprises et mises à disposition en tant que service. Voici quelques exemples populaires d’applications web : Microsoft 365, GitHub et Salesforce. Il existe des milliers d’autres applications. Les utilisateurs accèdent aux applications web à l’aide d’un navigateur web depuis leur ordinateur. L’authentification unique permet aux utilisateurs de naviguer entre les différentes applications web sans avoir à se connecter plusieurs fois.

Pour en savoir plus sur le fonctionnement de l’authentification unique avec les applications web, consultez ces deux vidéos.

Concepts de base de l’authentification : Applications web | Azure Active Directory

> [!VIDEO https://www.youtube.com/embed/tCNcG1lcCHY]

Concepts de base de l’authentification : Authentification unique web | Azure Active Directory

> [!VIDEO https://www.youtube.com/embed/51B-jSOBF8U]

## <a name="cloud-versus-on-premises-hosted-apps"></a>Applications hébergées dans le cloud ou en local
La façon d’implémenter l’authentification unique dépend de l’emplacement d’hébergement de l’application. L’hébergement est important en raison de la façon dont le trafic réseau est routé pour accéder à l’application. Si une application est hébergée et accessible sur votre réseau local (application locale), les utilisateurs n’ont pas besoin d’accéder à Internet pour l’utiliser. Si l’application est hébergée ailleurs (application hébergée dans le cloud), les utilisateurs devront accéder à Internet pour pouvoir utiliser l’application.

> [!TIP]
> Les applications hébergées dans le cloud sont également appelées applications SaaS (Software as a Service, logiciel en tant que service). 

L’authentification unique pour les applications hébergées dans le cloud est simple. Vous indiquez au fournisseur d’identité qu’il est utilisé pour l’application. Ensuite, vous configurez l’application pour qu’elle approuve le fournisseur d’identité. Pour apprendre comment utiliser Azure AD en tant que fournisseur d'identité pour une application, consultez la [Série de démarrages rapides sur la gestion des applications](add-application-portal.md).

> [!TIP]
> Les termes cloud et Internet sont souvent utilisés de manière interchangeable. La raison à cela est liée aux diagrammes de réseau. Les grands réseaux informatiques sont souvent représentés sous la forme de nuages (clouds) dans les diagrammes, car il n’est pas possible d’en dessiner chaque composant. Internet étant le réseau le plus connu et, il est facile d’utiliser les termes de manière interchangeable. Cependant, tout réseau informatique peut être considéré comme un cloud.

Vous pouvez également utiliser l’authentification unique pour les applications locales. La technologie d’application de l’authentification unique locale est appelée Proxy d’application. Pour en savoir plus à ce sujet, consultez les [Options d’authentification unique](sso-options.md).

## <a name="multiple-identity-providers"></a>Fournisseurs d'identité multiples
On appelle fédération le fait de configurer une authentification unique permettant de travailler entre plusieurs fournisseurs d'identité. Pour en savoir plus sur le fonctionnement de la fédération, regardez cette vidéo.

Concepts de base de l’authentification : Fédération | Azure Active Directory

> [!VIDEO https://www.youtube.com/embed/CjarTgjKcX8]


## <a name="next-steps"></a>Étapes suivantes
* [Série de guides de démarrage rapide sur la gestion des applications](view-applications-portal.md)
* [Options d’authentification unique](sso-options.md)
