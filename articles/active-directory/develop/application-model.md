---
title: Modèle d’application | Azure
titleSuffix: Microsoft identity platform
description: En savoir plus sur le processus d’inscription de votre application afin qu’elle puisse s’intégrer avec la Plateforme d’identités Microsoft (v2.0).
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/28/2020
ms.author: ryanwi
ms.reviewer: jmprieur, saeeda, sureshja, hirsin
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: 5aca96a9c3bc4e8f1061f677e316565b10014ac9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88117479"
---
# <a name="application-model"></a>Modèle d'application

Les applications peuvent connecter les utilisateurs elles-mêmes, ou déléguer la connexion à un fournisseur d’identité. Cette rubrique décrit les étapes nécessaires pour inscrire une application auprès de la Plateforme d’identités Microsoft.

## <a name="registering-an-application"></a>Inscription d’une application

Pour qu’un fournisseur d’identité sache qu’un utilisateur a accès à une application particulière, l’utilisateur et l’application doivent tous les deux être inscrits auprès du fournisseur d’identité. Lorsque vous inscrivez votre application auprès d’Azure AD, vous fournissez une configuration d’identité pour votre application, qui lui permet de s’intégrer avec la Plateforme d’identités Microsoft. L’inscription de l’application vous permet également d’effectuer les opérations suivantes :

* Personnaliser la boîte de dialogue de connexion avec la marque de votre application. C’est important, car il s’agit de la première expérience de l’utilisateur avec votre application.
* Décider si vous souhaitez laisser les utilisateurs se connecter uniquement s’ils appartiennent à votre organisation. Il s’agit d’une application monolocataire. Ou autoriser les utilisateurs à se connecter avec un compte professionnel ou scolaire. C’est alors une application multilocataire. Vous pouvez également autoriser les comptes Microsoft personnels, ou un compte social depuis LinkedIn, Google, etc.
* Demandez des autorisations d’étendue. Par exemple, vous pouvez demander l’étendue « user.read », qui accorde l’autorisation de lire le profil de l’utilisateur connecté.
* Préciser les étendues qui définissent l’accès à votre API web. En général, lorsqu’une application souhaite accéder à votre API, elle doit demander des autorisations pour les étendues que vous définissez.
* Partagez une clé secrète avec la Plateforme d’identités Microsoft qui prouve l’identité de l’application.  Cette action est pertinente dans le cas où l’application est une application cliente confidentielle. Une application cliente confidentielle est une application qui peut conserver les informations d’identification en toute sécurité. Un serveur back-end approuvé pour stocker les informations d’identification.

Une fois inscrite, l’application reçoit un identificateur unique qu’elle partage avec la Plateforme d’identités Microsoft lorsque celle-ci demande des jetons. Si l’application est une [application cliente confidentielle](developer-glossary.md#client-application), elle partage également le secret ou la clé publique, selon que des certificats ou des secrets ont été utilisés.

La Plateforme d’identités Microsoft représente des applications en utilisant un modèle qui remplit deux fonctions principales :

* Identifier l’application par les protocoles d’authentification qu’elle prend en charge
* Fournir tous les identificateurs, URL, secrets et informations connexes nécessaires à l’authentification

Plateforme d’identités Microsoft :

* contient toutes les données nécessaires lors de l’authentification, au moment de l’exécution ;
* contient toutes les données permettant de déterminer les ressources auxquelles une application peut devoir accéder et les circonstances dans lesquelles une requête donnée doit être satisfaite ;
* fournit l’infrastructure pour implémenter l’approvisionnement de l’application au sein du locataire de son développeur et de tout autre locataire Azure AD.
* Gérer le consentement de l’utilisateur au moment de la demande de jeton et faciliter l’approvisionnement dynamique des applications entre les locataires

Le **consentement** est le processus par lequel un propriétaire de ressources octroie à une application cliente l’accès à des ressources protégées, avec des autorisations spécifiques, en son nom. Plateforme d’identités Microsoft :

* Il permet aux utilisateurs et administrateurs d’accorder ou de refuser dynamiquement le consentement lorsque l’application souhaite accéder aux ressources en leur nom.
* Il permet aux administrateurs d’avoir la décision finale sur les applications qu’ils sont autorisées à fonctionner, les utilisateurs qui peuvent utiliser des applications spécifiques et la manière dont les ressources de répertoire sont consultées.

## <a name="multi-tenant-apps"></a>Applications multi-locataires

Dans la Plateforme d’identités Microsoft, un [objet d’application](developer-glossary.md#application-object) décrit une application. Au moment du déploiement, la Plateforme d’identités Microsoft utilise l’objet d’application en tant que blueprint pour créer un [principal du service](developer-glossary.md#service-principal-object). Ce dernier représente une instance concrète d’une application à l’intérieur d’un annuaire ou d’un locataire. Le principal de service définit les actions réelles d’une application dans un répertoire cible spécifique, les utilisateurs qui peuvent s’en servir, les ressources auxquelles elle a accès, etc. La plateforme d’identités Microsoft crée un principal de service à partir d’un objet d’application via le [consentement](developer-glossary.md#consent).

Le diagramme suivant illustre un flux de provisionnement simplifié piloté par consentement dans la plateforme d’identités Microsoft. Il montre deux locataires : *A* et *B*.

* Le *locataire A* possède l’application.
* Le *locataire B* instancie l’application via un principal de service.

![Flux d’approvisionnement simplifié piloté par consentement](./media/authentication-scenarios/simplified-provisioning-flow-consent-driven.svg)

Dans ce flux d’approvisionnement :

1. Un utilisateur du locataire B tente de se connecter avec l’application, le point de terminaison d’autorisation demande un jeton pour l’application.
1. Les informations d’identification de l’utilisateur sont acquises et vérifiées à des fins d’authentification.
1. L’utilisateur est invité à donner son consentement pour que l’application accède au locataire B.
1. La Plateforme d’identités Microsoft utilise l’objet d’application dans le locataire A en tant que blueprint pour créer un principal de service dans le locataire B.
1. L’utilisateur reçoit le jeton demandé.

Vous pouvez répéter ce processus pour d’autres locataires. Le locataire A conserve le blueprint pour l’application (objet d’application). Les utilisateurs et les administrateurs de tous les autres locataires où l’application a obtenu le consentement gardent le contrôle sur les actions de l’application par le biais de l’objet du principal de service correspondant dans chaque locataire. Pour plus d’informations, consultez [Objets d’application et de principal de service dans la plateforme d’identités Microsoft](app-objects-and-service-principals.md).

## <a name="next-steps"></a>Étapes suivantes

Pour d’autres rubriques traitant des concepts de base de l’authentification et de l’autorisation :

* Pour en savoir plus sur les concepts de base de l’authentification et de l’autorisation dans la Plateforme d’identités Microsoft, consultez [Authentification ou autorisation](authentication-vs-authorization.md).
* Pour savoir comment les jetons d’accès, les jetons d’actualisation et les jetons d’ID sont utilisés dans l’authentification et l’autorisation, consultez [Jetons de sécurité](security-tokens.md).
* Pour en savoir plus sur le flux de connexion des applications web, de bureau et mobiles dans la Plateforme d’identités Microsoft, consultez [Flux de connexion des applications](app-sign-in-flow.md).

Pour en savoir plus sur le modèle d’application :

* Pour plus d’informations sur les objets d’application et les principaux de service dans la Plateforme d’identités Microsoft, consultez [Comment et pourquoi les applications sont ajoutées à Azure AD](active-directory-how-applications-are-added.md).
* Pour plus d’informations sur les applications à locataire unique et les applications mutualisées, consultez [Location dans Azure Active Directory](single-and-multi-tenant-apps.md).
* Pour plus d’informations sur la façon dont Azure AD fournit également Azure Active Directory B2C pour permettre aux organisations de connecter des utilisateurs, généralement des clients, à l’aide d’identités sociales telles qu’un compte Google, consultez la [documentation d’Azure Active Directory B2C](../../active-directory-b2c/index.yml).