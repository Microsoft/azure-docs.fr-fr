---
title: Segments UNH2.5 dans les messages EDIFACT
description: Résoudre des messages EDIFACT avec des segments UNH2.5 dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/27/2017
ms.openlocfilehash: c6d95cb938a341b94a32989e82c42d37956c0a80
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89179842"
---
# <a name="handle-edifact-documents-with-unh25-segments-in-azure-logic-apps"></a>Gérer des documents EDIFACT avec des segments UNH2.5 dans Azure Logic Apps

Si un segment UNH2.5 existe dans un document EDIFACT, il est utilisé pour la recherche de schéma. Ainsi, dans cet exemple de message EDIFACT, le champ UNH est `EAN008` :

`UNH+SSDD1+ORDERS:D:03B:UN:EAN008`

Pour gérer ce message, effectuez les étapes décrites ci-après :

1. Mettre à jour le schéma

1. Vérifier les paramètres de l’accord

## <a name="update-the-schema"></a>Mettre à jour le schéma

Pour traiter le message, vous devez déployer un schéma qui a le nom de nœud racine UNH2.5. Ainsi, le nom de la racine du schéma pour l’exemple de champ UNH est `EFACT_D03B_ORDERS_EAN008`. Pour chaque `D03B_ORDERS` qui a un autre segment UNH2.5, vous devez déployer un schéma individuel.

## <a name="add-schema-to-edifact-agreement"></a>Ajouter un schéma à l’accord EDIFACT

### <a name="edifact-decode"></a>Décodage EDIFACT

Pour décoder le message entrant, configurez le schéma dans les paramètres de réception de l’accord EDIFACT :

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre compte d’intégration.

1. Ajoutez le schéma à votre compte d’intégration.

1. Configurez le schéma dans les paramètres de réception de l’accord EDIFACT.

1. Sélectionnez l’accord EDIFACT, puis sélectionnez **Modifier au format JSON**. Ajoutez la valeur UNH2.5 à la section `schemaReferences` de l’accord de réception :

   ![Ajouter UNH2.5 pour l’accord de réception](./media/logic-apps-enterprise-integration-edifact_inputfile_unh2.5/image1.png)

### <a name="edifact-encode"></a>Encodage EDIFACT

Pour encoder le message entrant, configurez le schéma EDIFACT dans les paramètres d’envoi de l’accord EDIFACT.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre compte d’intégration.

1. Ajoutez le schéma à votre compte d’intégration.

1. Configurez le schéma EDIFACT dans les paramètres d’envoi de l’accord EDIFACT.

1. Sélectionnez l’accord EDIFACT, puis sélectionnez **Modifier au format JSON**. Ajoutez la valeur UNH2.5 à la section `schemaReferences` de l’accord d’envoi :

   ![Ajouter UNH2.5 à l’accord d’envoi](./media/logic-apps-enterprise-integration-edifact_inputfile_unh2.5/image2.png)

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur les [accords de compte d’intégration](../logic-apps/logic-apps-enterprise-integration-agreements.md).
