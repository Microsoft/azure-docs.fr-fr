---
title: Vue d’ensemble - Azure serverless pour les applications et solutions informatiques
description: Découvrez comment créer des solutions et des applications informatiques sans vous soucier de l’infrastructure à l’aide d’Azure Logic Apps et d’Azure Functions
services: logic-apps
ms.suite: integration
author: jeffhollan
ms.author: jehollan
ms.reviewer: klam, estfan, logicappspm
ms.topic: article
ms.date: 03/30/2017
ms.openlocfilehash: 0f20bb5fb249ad6bac862afe2b0e8eee4b32e2a9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75666548"
---
# <a name="azure-serverless-overview-for-building-cloud-based-apps-and-solutions-with-azure-logic-apps-and-azure-functions"></a>Azure serverless : Vue d’ensemble de la création d’applications et de solutions informatiques avec Azure Logic Apps et Azure Functions

Les applications [sans serveur](https://azure.microsoft.com/solutions/serverless/) offrent des avantages tels que la vitesse de développement accrue, la réduction de codes, la simplicité et la mise à l’échelle. Cet article présente les différents attributs des solutions sans serveur et des offres sans serveur Azure.

## <a name="what-is-serverless"></a>Que sont les environnements sans serveur ?

« Sans serveur » ne signifie pas qu’il n’y a pas de serveurs, mais simplement que le développeur n’a pas à se soucier des serveurs. Une grande partie du développement traditionnel d’application traite de la mise à l’échelle, l’hébergement et la surveillance des solutions pour répondre aux besoins de l’application. Avec les offres sans serveur, ces questions sont prises en charge dans le cadre de la solution. En outre, les applications sans serveur sont facturées sur un plan basé sur la consommation. Si l’application n’est jamais utilisée, aucun frais n’est facturé. Ces fonctionnalités permettent aux développeurs de se concentrer uniquement sur la logique métier d’une solution.

Les principaux services sans serveur dans Azure sont [Azure Functions](https://azure.microsoft.com/services/logic-apps/) et [Azure Logic Apps](https://azure.microsoft.com/services/functions/). Les deux solutions suivent les principes décrits précédemment et aident les développeurs à créer des applications Cloud robustes avec un minimum de code.

## <a name="what-is-azure-logic-apps"></a>Qu’est-ce qu’Azure Logic Apps ?

[Azure Logic Apps](logic-apps-overview.md) permet de simplifier et d’implémenter des intégrations et des flux de travail évolutifs dans le cloud. Ce service fournit un concepteur visuel pour modéliser et automatiser votre processus sous la forme d’une série d’étapes appelées flux de travail. Il existe de nombreux [connecteurs](../connectors/apis-list.md) entre les services Cloud et les systèmes locaux qui connectent rapidement une application sans serveur à d’autres API. Chaque application logique commence par un déclencheur, tel que « lorsqu’un compte est ajouté à Dynamics CRM ». Une fois le déclencheur activé, le flux de travail peut exécuter des combinaisons d’actions, de conversions et de logiques conditionnelles. Logic Apps est un choix parfait lorsque vous orchestrez différentes fonctions Azure dans un processus, en particulier lorsque le processus doit interagir avec un système externe ou une API.

Pour commencer avec Logic Apps, [créez votre première application logique](quickstart-create-first-logic-app-workflow.md). Pour plus d’informations techniques sur Logic Apps, consultez les [références du développeur](logic-apps-workflow-definition-language.md).

## <a name="what-is-azure-functions"></a>Qu’est-ce que Azure Functions ?

Azure Functions est un service qui permet d’exécuter facilement des éléments de code ou des « fonctions » dans le Cloud. Vous pouvez écrire uniquement le code nécessaire pour le problème actuel, sans vous soucier d’une application complète ou de l’infrastructure requise. Grâce à Functions, le développement gagne en productivité et vous pouvez utiliser votre langage de développement préféré, tel que C#, F#, Node.js, Python ou PHP. Vous payez uniquement pour le temps d’exécution de votre code et Azure évolue en fonction des besoins.

Pour commencer à utiliser Azure Functions, commencez par [créer votre première fonction Azure](../azure-functions/functions-create-first-azure-function.md). Pour plus d’informations techniques sur les fonctions, consultez la [référence du développeur](../azure-functions/functions-reference.md).

## <a name="how-can-i-build-and-deploy-serverless-apps-in-azure"></a>Comment créer et déployer des applications sans serveur dans Azure ?

Azure fournit des outils riches pour le développement, le déploiement et la gestion d’applications sans serveur. Vous pouvez créer des applications directement dans le portail Azure, avec les [outils de Visual Studio](logic-apps-serverless-get-started-vs.md) ou avec [Visual Studio Code](quickstart-create-logic-apps-visual-studio-code.md). Après avoir créé votre application, vous pouvez [déployer cette application rapidement avec les modèles Azure Resource Manager](logic-apps-deploy-azure-resource-manager-templates.md). Azure fournit également une supervision, à laquelle vous pouvez accéder via la Portail Azure, via l’API ou les kits de développement logiciel (SDK), ou avec des outils intégrés aux journaux Azure Monitor et à Application Insights.

## <a name="next-steps"></a>Étapes suivantes

* [Créer une application sans serveur dans Visual Studio](logic-apps-serverless-get-started-vs.md)
* [Créer un tableau de bord des insights client sans serveur](logic-apps-scenario-social-serverless.md)
* [Automatiser le déploiement d’application logique](logic-apps-azure-resource-manager-templates-overview.md)
