---
title: Surveiller, gérer et accéder à votre référentiel GitHub
description: Surveiller les événements GitHub et gérer votre référentiel GitHub en créant des flux de travail automatisés avec Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 03/02/2018
tags: connectors
ms.openlocfilehash: b67d2e860e8c69f6f2f43441144ef2f60ed3a104
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999539"
---
# <a name="monitor-and-manage-your-github-repo-by-using-azure-logic-apps"></a>Superviser et gérer votre référentiel GitHub à l’aide d’Azure Logic Apps

GitHub est un service d’hébergement de référentiel Git basé sur le web. Outre d’autres fonctionnalités, il offre toutes les fonctionnalités distribuées de contrôle de révision et de gestion du code source (SCM) de Git.

Pour bien commencer avec le connecteur GitHub, [commencez par créer une application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="create-a-connection-to-github"></a>Créer une connexion à GitHub

Pour utiliser le connecteur GitHub dans une application logique, commencez par créer une *connexion*, puis spécifiez les détails des propriétés suivantes : 

| Propriété | Obligatoire | Description | 
| -------- | -------- | ----------- | 
| par jeton | Oui | Indiquez vos informations d’identification GitHub. |

Après avoir créé la connexion, vous pouvez exécuter les actions et rechercher les déclencheurs décrits dans cet article.

> [!INCLUDE [Steps to create a connection to GitHub](../../includes/connectors-create-api-github.md)]
> 

## <a name="connector-reference"></a>Référence de connecteur

Pour obtenir des détails techniques sur les déclencheurs, les actions et les limites, qui sont décrits par la description OpenAPI du connecteur (anciennement Swagger), consultez la [page de référence du connecteur](/connectors/github/).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)