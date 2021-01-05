---
title: Accords de partenariat commercial
description: Créer et gérer des accords entre partenaires commerciaux à l'aide d'Azure Logic Apps et d'Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 06/22/2019
ms.openlocfilehash: c8cbfb619c9eed325161503f705bf5c4c0746265
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95992934"
---
# <a name="create-and-manage-trading-partner-agreements-in-azure-logic-apps"></a>Créer et gérer des contrats de partenariat commercial dans Azure Logic Apps

Un [accord](../logic-apps/logic-apps-enterprise-integration-partners.md) 
*de partenariat commercial* aide les organisations et les entreprises à communiquer entre elles en toute transparence en définissant le protocole standard à utiliser lors de l'échange de messages interentreprises (B2B). Les accords offrent des avantages communs, par exemple :

* Ils permettent aux organisations d'échanger des informations en utilisant un format qu'elles connaissent bien.
* Ils améliorent l'efficacité des transactions B2B.
* Ils sont faciles à créer, gérer et utiliser lors de la conception de solutions d'intégration d'entreprise.

Cet article explique comment créer un accord AS2, EDIFACT ou X12 que vous pouvez utiliser lors de la création de solutions d'intégration d'entreprise pour les scénarios B2B à l'aide d'[Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md) et d'[Azure Logic Apps](../logic-apps/logic-apps-overview.md). Après avoir créé un accord, vous pouvez utiliser les connecteurs AS2, EDIFACT ou X12 pour échanger des messages B2B.

Pour créer des contrats pour l’échange de messages RosettaNet, consultez [Échanger des messages RosettaNet](../logic-apps/logic-apps-enterprise-integration-rosettanet.md).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un [compte d'intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) pour stocker votre accord et d'autres artefacts B2B. Ce compte d'intégration doit être associé à votre abonnement Azure.

* Au moins deux [partenaires commerciaux](../logic-apps/logic-apps-enterprise-integration-partners.md) que vous avez déjà créés dans votre compte d'intégration. Un accord requiert un partenaire hôte et un partenaire invité. Les deux partenaires doivent utiliser le même qualificateur d'« identité d'entreprise » que l'accord que vous souhaitez créer, comme AS2, X12 ou EDIFACT.

* Facultatif : l'application logique sur laquelle vous souhaitez utiliser votre accord et un déclencheur qui lance le flux de travail de votre application logique. Pour créer votre compte d'intégration et vos artefacts B2B, vous n'avez pas besoin d'application logique. Mais avant que votre application logique puisse utiliser les artefacts B2B dans votre compte d'intégration, vous devez lier celui-ci à votre application logique. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="create-agreements"></a>Créer des accords

1. Connectez-vous au [portail Azure](https://portal.azure.com).
Dans le menu principal Azure, sélectionnez **Tous les services**. Dans la zone de recherche, entrez « intégration » comme filtre. Dans les résultats, sélectionnez la ressource suivante : **Comptes d’intégration**

   ![Recherche du compte d’intégration](./media/logic-apps-enterprise-integration-agreements/find-integration-accounts.png)

1. Sous **Comptes d'intégration**, sélectionnez le compte d'intégration dans lequel vous voulez créer l'accord.

   ![Sélectionnez le compte d’intégration dans lequel vous souhaitez créer le contrat](./media/logic-apps-enterprise-integration-agreements/select-integration-account.png)

1. Dans le volet de droite, sous **Composants**, choisissez la vignette **Accords**.

   ![Choisissez « Accords ».](./media/logic-apps-enterprise-integration-agreements/agreement-1.png)

1. Sous **Contrats**, choisissez **Ajouter**. Dans le volet **Ajouter**, fournissez des informations sur votre accord, par exemple :

   ![Choisir « Ajouter »](./media/logic-apps-enterprise-integration-agreements/agreement-2.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Nom** | Oui | <*nom de l'accord*> | Nom de votre accord |
   | **Type d'accord** | Oui | **AS2**, **X12** ou **EDIFACT** | Type de protocole de votre accord. Lorsque vous créez le fichier de votre accord, son contenu doit correspondre au type de l'accord. | |  
   | **Partenaire hôte** | Oui | <*nom du partenaire hôte*> | Le partenaire hôte représente l'organisation qui spécifie l'accord. |
   | **Identité de l'hôte** | Oui | <*identificateur du partenaire hôte*> | Identificateur du partenaire hôte. |
   | **Partenaire invité** | Oui | <*nom du partenaire invité*> | Le partenaire invité représente l’organisation qui travaille avec le partenaire hôte. |
   | **Identité de l'invité** | Oui | <*identificateur du partenaire invité*> | Identificateur du partenaire invité. |
   | **Paramètres de réception** | Variable | Variable | Ces propriétés spécifient comment le partenaire hôte reçoit tous les messages entrants du partenaire invité dans le contrat. Pour plus d'informations, consultez le type d'accord correspondant : <p>- [Paramètres des messages AS2](../logic-apps/logic-apps-enterprise-integration-as2-message-settings.md) <br>- [Paramètres des messages EDIFACT](logic-apps-enterprise-integration-edifact.md) <br>- [Paramètres des messages X12](logic-apps-enterprise-integration-x12.md) |
   | **Paramètres d'envoi** | Variable | Variable | Ces propriétés spécifient comment le partenaire hôte envoie tous les messages sortants au partenaire invité dans le contrat. Pour plus d'informations, consultez le type d'accord correspondant : <p>- [Paramètres des messages AS2](../logic-apps/logic-apps-enterprise-integration-as2-message-settings.md) <br>- [Paramètres des messages EDIFACT](logic-apps-enterprise-integration-edifact.md) <br>- [Paramètres des messages X12](logic-apps-enterprise-integration-x12.md) |
   |||||

   > [!IMPORTANT]
   > La résolution d’un contrat dépend de la correspondance de ces éléments qui sont définis dans le partenaire et le message entrant :
   >
   > * Qualificateur et identificateur de l’expéditeur
   > * Qualificateur et identificateur du récepteur
   >
   > Si ces valeurs changent pour votre partenaire, veillez à également mettre à jour le contrat.

1. Une fois votre accord créé, sur la page **Ajouter**, choisissez **OK** et revenez à votre compte d'intégration.

   La liste **Accords** contient désormais votre nouvel accord.

## <a name="edit-agreements"></a>Modifier des accords

1. Sur le [portail Azure](https://portal.azure.com), accédez au menu principal Azure et sélectionnez **Tous les services**.

1. Dans la zone de recherche, entrez « intégration » comme filtre. Dans les résultats, sélectionnez la ressource suivante : **Comptes d’intégration**

1. Sous **Comptes d'intégration**, sélectionnez le compte d'intégration associé à l'accord à modifier.

1. Dans le volet de droite, sous **Composants**, choisissez la vignette **Accords**.

1. Sous **Accords**, sélectionnez votre accord et choisissez **Modifier**.

1. Apportez les modifications nécessaires et enregistrez-les.

## <a name="delete-agreements"></a>Supprimer des accords

1. Sur le [portail Azure](https://portal.azure.com), accédez au menu principal Azure et sélectionnez **Tous les services**.

1. Dans la zone de recherche, entrez « intégration » comme filtre. Dans les résultats, sélectionnez la ressource suivante : **Comptes d’intégration**

1. Sous **Comptes d'intégration**, sélectionnez le compte d'intégration associé à l'accord à supprimer.

1. Dans le volet de droite, sous **Composants**, choisissez la vignette **Accords**.

1. Sous **Accords**, sélectionnez votre accord et choisissez **Supprimer**.

1. Confirmez que vous souhaitez supprimer le contrat sélectionné.

## <a name="next-steps"></a>Étapes suivantes

* [Échanger des messages AS2](logic-apps-enterprise-integration-as2.md)
* [Échanger des messages EDIFACT](logic-apps-enterprise-integration-edifact.md)
* [Échanger des messages X12](logic-apps-enterprise-integration-x12.md)
