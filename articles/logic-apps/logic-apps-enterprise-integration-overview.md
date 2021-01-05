---
title: Intégration d’entreprise B2B
description: Découvrez comment générer des workflows B2B automatisés pour l’intégration d’entreprise avec Azure Logic Apps et Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, logicappspm
ms.topic: overview
ms.date: 08/01/2019
ms.openlocfilehash: adf15904d4490647074d6e2b46ac7e9e871dd388
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "86536400"
---
# <a name="b2b-enterprise-integration-solutions-with-azure-logic-apps-and-enterprise-integration-pack"></a>Solutions d’intégration d’entreprise B2B avec Azure Logic Apps et Enterprise Integration Pack

Pour les solutions B2B (Business-to-Business) et une communication fluide entre les organisations, vous pouvez générer des workflows d’intégration d’entreprise scalables automatisés en utilisant Enterprise Integration Pack (EIP) avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md). Les organisations peuvent échanger des messages électroniques, même si elles utilisent des formats et des protocoles différents. EIP transforme différents formats en un format que les systèmes de votre organisation peuvent traiter et prend en charge les protocoles standard, notamment [AS2](../logic-apps/logic-apps-enterprise-integration-as2.md), [X12](logic-apps-enterprise-integration-x12.md) et [EDIFACT](../logic-apps/logic-apps-enterprise-integration-edifact.md). Vous pouvez également améliorer la sécurité des messages en utilisant à la fois le chiffrement et les signatures numériques. EIP prend en charge les [connecteurs d’intégration d’entreprise](../connectors/apis-list.md#integration-account-connectors) et normes industrielles ci-dessous :

* Échange de données informatisé (EDI)
* Intégration des applications de l’entreprise (EAI)

Si vous connaissez Microsoft BizTalk Server ou Azure BizTalk Services, EIP suit des concepts similaires, ce qui rend les fonctionnalités faciles à utiliser. Toutefois, une différence majeure réside dans le fait qu’EIP repose du point de vue architectural sur les « comptes d’intégration » pour simplifier le stockage et la gestion des artefacts utilisés dans les communications B2B. Ces comptes sont des conteneurs basés sur le cloud qui stockent tous vos artefacts, tels que les schémas, les partenaires, les certificats, les mappages et les contrats. 

## <a name="why-use-the-enterprise-integration-pack"></a>Pourquoi utiliser Enterprise Integration Pack ?

* EIP vous permet de stocker tous vos artefacts à un même endroit : votre compte d’intégration.

* Vous pouvez générer des workflows B2B et les intégrer à des applications SaaS tierces, des applications locales et des applications personnalisées à l’aide d’Azure Logic Apps et des connecteurs.

* Vous pouvez créer un code personnalisé pour vos applications logiques avec Azure Functions.

## <a name="how-do-i-get-started"></a>Comment faire pour démarrer ?

Avant de pouvoir commencer à générer des workflows d’application logique B2B avec EIP, vous avez besoin des éléments suivants :

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) avec les artefacts que vous souhaitez utiliser

* Pour créer des mappages et des schémas, vous pouvez utiliser [Microsoft Azure Logic Apps Enterprise Integration Tools pour Visual Studio 2015 2.0](https://aka.ms/vsmapsandschemas) et Visual Studio 2015.

Après avoir créé un compte d’intégration et ajouté vos artefacts, vous pouvez commencer à générer des workflows B2B avec ces artefacts en créant une application logique dans le portail Azure. Si vous débutez avec les applications logiques, essayez de [créer une application logique de base](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour utiliser ces artefacts, veillez à lier d’abord votre compte d’intégration à votre application logique. Ainsi, votre application logique peut accéder à votre compte d’intégration. Vous pouvez également créer, gérer et déployer des applications logiques à l’aide de Visual Studio ou de [PowerShell](/powershell/module/az.logicapp).

Voici les étapes élémentaires pour commencer à générer des applications logiques B2B :

![Prérequis à la création d’applications logiques B2B](./media/logic-apps-enterprise-integration-overview/overview.png)  

## <a name="try-now"></a>Essayer dès maintenant

[Déployer un exemple d’application logique entièrement opérationnelle qui envoie et reçoit des messages AS2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-logic-app-as2-send-receive)

## <a name="next-steps"></a>Étapes suivantes

* [Créer des partenaires commerciaux](logic-apps-enterprise-integration-partners.md)
* [Créer des contrats](../logic-apps/logic-apps-enterprise-integration-agreements.md)
* [Ajouter des schémas](logic-apps-enterprise-integration-schemas.md)
* [Ajouter des mappages](../logic-apps/logic-apps-enterprise-integration-maps.md)
* [Migrer depuis BizTalk Services](../logic-apps/logic-apps-move-from-mabs.md)
