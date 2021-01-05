---
title: Bonnes pratiques du développement sécurisé sur Microsoft Azure
description: Il s’agit des bonnes pratiques permettant de vous aider à développer du code plus sécurisé et de déployer une application plus sécurisée dans le cloud.
author: TerryLanfear
manager: barbkess
ms.author: terrylan
ms.date: 06/11/2019
ms.topic: article
ms.service: security
ms.subservice: security-develop
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: baaa311620f5c10948aa3494002ce359cc5dab28
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94517170"
---
# <a name="secure-development-best-practices-on-azure"></a>Bonnes pratiques du développement sécurisé sur Azure
Cette série d’articles présente les activités et contrôles de sécurité à prendre en compte lorsque vous développez des applications pour le cloud. Les phases du Microsoft Security Development Lifecycle et les questions et concepts de sécurité à prendre en compte lors de chaque phase du cycle de vie y sont abordées. L’objectif est de vous aider à définir les activités et services Azure que vous pouvez utiliser lors de chaque phase du cycle de vie pour concevoir, développer et déployer une application plus sécurisée.

Les suggestions présentées dans les articles proviennent de notre expérience en matière de sécurité Azure et des expériences de nos clients. Vous pouvez utiliser ces articles pour vous référer aux éléments à prendre en compte au cours de chaque phase spécifique de votre projet de développement. Toutefois, nous vous suggérons également de lire entièrement tous les articles au moins une fois. La lecture de tous les articles présente les concepts que vous avez omis dans les premières phases de votre projet. L’implémentation de ces concepts avant la publication de votre produit peut vous permettre de créer des logiciels sécurisés, de répondre aux exigences de conformité de sécurité et de réduire les coûts de développement.

Ces articles sont conçus comme une ressource qui aidera les concepteurs, les développeurs et les testeurs de logiciels, à tous les niveaux, à créer et déployer des applications Azure sécurisées.

## <a name="overview"></a>Vue d’ensemble

La sécurité est un des aspects les plus importants d’une application, et n’est pas une chose si simple. Heureusement, Azure propose de nombreux services qui peuvent vous aider à sécuriser votre application dans le cloud. Ces articles concernent les activités et services Azure que vous pouvez implémenter à chaque étape du cycle de vie du développement de votre logiciel pour vous aider à développer du code plus sécurisé et à déployer une application plus sécurisée dans le cloud.

## <a name="security-development-lifecycle"></a>Cycle de vie de développement de la sécurité

Suivre les bonnes pratiques pour développer des logiciels sécurisés nécessite d’intégrer la sécurité à chaque phase du cycle de vie du développement logiciel, de l’analyse des exigences à la maintenance, quelle que soit la méthodologie de projet ([en cascade](https://en.wikipedia.org/wiki/Waterfall_model), [agile](https://en.wikipedia.org/wiki/Agile_software_development), ou [DevOps](https://en.wikipedia.org/wiki/DevOps)). À la suite de violation de données de grande envergure et de l’exploitation de failles de sécurité opérationnelles, les développeurs sont de plus en plus nombreux à considérer que la sécurité doit être prise en compte tout au long du processus de développement.

Plus vous tardez à corriger le problème dans votre cycle de vie de développement, plus cette opération est onéreuse. Les problèmes de sécurité ne font pas exception. Si vous ignorez les problèmes de sécurité au cours des premières phases de votre développement logiciel, chaque phase qui s’ensuit peut hériter des vulnérabilités de la phase précédente. Votre produit final cumulera plusieurs problèmes de sécurité et pourra subir de potentielles violations. Intégrer la sécurité à chaque phase du cycle de vie du développement permet de repérer les problèmes au plus tôt, et de réduire les coûts liés au développement.

Nous suivons les phases de [Microsoft Security Development Lifecycle](/previous-versions/windows/desktop/cc307891(v=msdn.10)) pour introduire les activités et les services Azure que vous pouvez utiliser pour respecter les pratiques en matière de développement de logiciels sécurisés lors de chaque phase du cycle de vie.

Les phases SDL sont les suivantes :

  - Entrainement
  - Spécifications
  - Conception
  - Implémentation
  - Vérification
  - Libérer
  - response

![Cycle de vie de développement de la sécurité](./media/secure-dev-overview/01-sdl-phase.png)

Dans ces articles nous regroupons les phases SDL en conception, développement et déploiement.

## <a name="engage-your-organizations-security-team"></a>Impliquer l’équipe de sécurité de l’organisation

Il se peut que votre organisation dispose d’un programme de sécurité d’application officiel qui vous aide dans vos activités de sécurité tout au long du cycle de vie de développement. Si votre organisation dispose d’équipes chargées de la conformité et de la sécurité, veillez à les impliquer avant de développer votre application. Demandez-leur à chaque phase du Microsoft Security Development Lifecycle si vous avez manqué des tâches.

Nous sommes conscients du fait que de nombreux lecteurs peuvent ne pas disposer d’une équipe de sécurité ou de conformité à solliciter. Ces articles peuvent aider à appréhender les questions de sécurité et les décisions que vous devez envisager à chaque phase du Microsoft Security Development Lifecycle.

## <a name="resources"></a>Ressources

Utilisez les ressources suivantes pour en savoir plus sur le développement d’applications sécurisées et pour vous aider à sécuriser vos applications sur Azure :

[Microsoft Security Development Lifecycle (SDL)](/previous-versions/windows/desktop/cc307891(v=msdn.10)) : processus de développement logiciel de Microsoft qui permet aux développeurs de créer des logiciels plus sécurisés. Il permet de répondre aux exigences de conformité de sécurité tout en réduisant les coûts de développement.

[Open Web Application Security Project (OWASP)](https://www.owasp.org/index.php/Main_Page) : communauté en ligne qui produit des articles, des méthodologies, de la documentation, des outils et des technologies disponibles gratuitement dans le domaine de la sécurité des applications web.

[Pushing Left, Like a Boss](https://code.likeagirl.io/pushing-left-like-a-boss-part-1-80f1f007da95?WT.mc_id=docs-blog-tajanca) : série d’articles en ligne décrivant les différents types d’activités de sécurité des applications que les développeurs doivent effectuer pour créer un code plus sécurisé.

[Plateforme d’identités Microsoft](../../active-directory/develop/index.yml) : évolution du service d’identité Azure Active Directory (Azure AD) et de la plateforme de développement. C’est une plateforme complète qui se compose d’un service d’authentification, de bibliothèques open source, de fonctionnalités d’inscription et de configuration d’application, d’une documentation de développement exhaustive et d’exemples de code et autres contenus destinés aux développeurs. La plateforme d’identités Microsoft prend en charge les protocoles standard, tels qu’OAuth 2.0 et OpenID Connect.

[Bonnes pratiques de sécurité pour les solutions Azure](https://azure.microsoft.com/resources/security-best-practices-for-azure-solutions/) : collection de bonnes pratiques en matière de sécurité à utiliser lorsque vous concevez, déployez et gérez des solutions cloud à l’aide d’Azure. Ce document est destiné à constituer une ressource pour les professionnels de l’informatique. Ceux-ci peuvent être des concepteurs, architectes, développeurs et testeurs qui génèrent et déploient des solutions Azure sécurisées.

[Blueprints de sécurité et de conformité Azure](https://servicetrust.microsoft.com/ViewPage/BlueprintOverview) : ressources qui peuvent vous aider à générer et lancer des applications cloud qui sont conformes à des normes et réglementations strictes.

## <a name="next-steps"></a>Étapes suivantes
Dans les articles suivants, nous recommandons des contrôles et des activités de sécurité qui peuvent vous aider à concevoir, développer et déployer des applications sécurisées.

- [Concevoir des applications sécurisées](secure-design.md)
- [Développer des applications sécurisées](secure-develop.md)
- [Déployer des applications sécurisées](secure-deploy.md)