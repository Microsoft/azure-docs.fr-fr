---
title: Valider des messages XML pour l’intégration d’entreprise B2B
description: Valider des documents XML à l’aide de schémas dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 10/21/2019
ms.openlocfilehash: ff21b059e712489c1914b2d12c6aa6a3d78d66d3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74792168"
---
# <a name="validate-xml-for-b2b-enterprise-integration-in-azure-logic-apps-with-enterprise-integration-pack"></a>Valider des documents XML dans le cadre d’une intégration d’entreprise B2B dans Azure Logic Apps avec Enterprise Integration Pack

Souvent, dans les scénarios B2B, les parties d’un contrat doivent s’assurer que les messages qu’elles échangent sont valides avant le début de tout traitement de données. Vous pouvez valider des documents par rapport à un schéma prédéfini à l’aide de l’action de validation XML, qui est disponible avec Enterprise Integration Pack.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Une application logique vide ou existante dans laquelle vous souhaitez utiliser l’action de validation XML. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) qui est associé à votre abonnement Azure, qui est lié à l’application logique dans laquelle vous envisagez d’utiliser l’action de validation XML et qui contient le schéma que vous souhaitez utiliser pour valider le contenu XML. Votre application logique et votre compte d’intégration doivent tous deux exister dans le même emplacement ou dans la même région Azure.

## <a name="add-xml-validation-action"></a>Ajouter l’action de validation XML

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Si vous disposez d’une application logique vide, dans le Concepteur d’application logique, dans la zone de recherche, entrez `HTTP request` comme filtre, puis sélectionnez le déclencheur **Lors de la réception d’une demande HTTP**. Sinon, passez à l’étape suivante.

1. Sous la dernière étape de votre workflow, sélectionnez **Nouvelle étape**.

   Pour ajouter une action entre des étapes existantes, déplacez votre pointeur sur la flèche qui connecte ces étapes afin que le signe plus ( **+** ) apparaisse. Sélectionnez ce signe plus, puis sélectionnez **Ajouter une action**.

1. Sous **Choisir une action**, sélectionnez **Intégré**. Dans la zone de recherche, entrez `xml validation` en guise de filtre. Dans la liste des actions, sélectionnez **Validation XML**.

   ![Rechercher et sélectionner l’action « Validation XML »](./media/logic-apps-enterprise-integration-xml-validation/select-xml-validation-action.png)

1. Pour spécifier le contenu XML que vous souhaitez valider, cliquez à l’intérieur de la zone **Contenu** afin que la liste de contenu dynamique s’affiche.

   ![Ouvrir la liste de contenu dynamique](./media/logic-apps-enterprise-integration-xml-validation/open-dynamic-content-list.png)

   La liste de contenu dynamique affiche des jetons de propriété qui représentent les sorties des étapes précédentes du workflow. Si la liste n’affiche pas de propriété attendue, vérifiez si vous pouvez sélectionner **Voir plus** dans l’en-tête du déclencheur ou de l’action.

1. Dans la liste de contenu dynamique, sélectionnez la propriété qui contient le contenu que vous souhaitez valider.

   Cet exemple sélectionne la sortie **Corps** du déclencheur.

   ![Sélectionner le contenu à valider](./media/logic-apps-enterprise-integration-xml-validation/select-content-to-validate.png)

1. Pour spécifier le schéma à utiliser pour la validation, ouvrez la liste **Nom du schéma** et sélectionnez le schéma de validation que vous avez ajouté à votre compte d’intégration lié.

   ![Sélectionner le schéma à utiliser pour la validation](./media/logic-apps-enterprise-integration-xml-validation/select-validation-schema.png)

1. Enregistrez votre application logique.

   Vous avez maintenant terminé la configuration de votre validation. Dans une application réelle, vous souhaiterez peut-être stocker les données validées dans une application métier telle que Salesforce. Pour envoyer la sortie validée à Salesforce, ajoutez une action.

1. Pour tester votre action de validation, vous pouvez envoyer une demande de déclenchement du workflow de votre application logique.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)