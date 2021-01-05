---
title: Ajouter des partenaires commerciaux pour les intégrations B2B
description: Créer dans votre compte d’intégration des partenaires commerciaux à utiliser avec Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 06/22/2019
ms.openlocfilehash: 8e3805fae5bf6cc5ad8cf759d3ba75220c6ddbd8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91565069"
---
# <a name="add-trading-partners-to-integration-accounts-for-azure-logic-apps"></a>Ajouter des partenaires commerciaux à des comptes d’intégration pour Azure Logic Apps

Dans [Azure Logic Apps](../logic-apps/logic-apps-overview.md), vous pouvez créer des workflows d’intégration B2B (Business-to-Business) automatisés en utilisant un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) avec vos applications logiques. Pour représenter votre organisation et d’autres organisations, vous créez et ajoutez des partenaires commerciaux en tant qu’artefacts à votre compte d’intégration. Les partenaires sont des entités qui participent aux transactions B2B et qui échangent des messages entre eux.

Avant de les créer, veillez à examiner et partager des informations avec vos partenaires sur la façon d’identifier et de valider les messages envoyés par les autres utilisateurs. Après vous être mis d’accord sur ces détails, vous êtes prêt à créer des partenaires dans votre compte d’intégration.

## <a name="partner-roles-in-integration-accounts"></a>Rôles des partenaires dans les comptes d’intégration

Pour définir les détails relatifs aux messages échangés avec vos partenaires, vous créez et ajoutez des [contrats](../logic-apps/logic-apps-enterprise-integration-agreements.md) en tant qu’artefacts à votre compte d’intégration. Les contrats nécessitent au moins deux partenaires dans votre compte d’intégration. Votre organisation est toujours le *partenaire hôte* dans le contrat. L’organisation qui échange des messages avec votre organisation est le *partenaire invité*. Un partenaire invité peut être une autre entreprise ou même un service au sein de votre organisation. Après avoir ajouté ces partenaires, vous pouvez créer un contrat.

Dans un contrat, vous spécifiez les détails pour la gestion des messages entrants et sortants du point de vue du partenaire hôte. Pour les messages entrants, les **paramètres de réception** déterminent la façon dont le partenaire hôte reçoit les messages du partenaire invité dans le contrat. Pour les messages sortants, les **paramètres d’envoi** indiquent la façon dont le partenaire hôte envoie des messages au partenaire invité.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) pour stocker vos partenaires, contrats et autres artefacts B2B. Ce compte d'intégration doit être associé à votre abonnement Azure.

## <a name="create-partner"></a>Créer un partenaire

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le menu principal Azure, sélectionnez **Tous les services**. Dans la zone de recherche, entrez « intégration », puis sélectionnez **Comptes d’intégration**.

   ![Sélectionner « Comptes d’intégration »](./media/logic-apps-enterprise-integration-partners/find-integration-accounts.png)

1. Sous **Comptes d’intégration**, sélectionnez le compte d’intégration dans lequel vous souhaitez ajouter vos partenaires.

   ![Sélectionner le compte d’intégration](./media/logic-apps-enterprise-integration-partners/select-integration-account.png)

1. Choisissez la mosaïque **Partenaires**.

   ![Capture d’écran montrant la vignette Partenaires.](./media/logic-apps-enterprise-integration-partners/choose-partners.png)

1. Sous **Partenaires**, choisissez **Ajouter**. Sous **Ajouter un partenaire**, indiquez les détails sur le partenaire comme décrit dans le tableau ci-dessous.

   ![Choisir « Ajouter » et indiquer les détails sur le partenaire](./media/logic-apps-enterprise-integration-partners/add-partners.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Nom** | Oui | Nom du partenaire |
   | **Qualificateur** | Oui | Corps d’authentification qui fournit des identités d’entreprise uniques aux organisations, par exemple **D-U-N-S (Dun & Bradstreet)** . <p>Les partenaires peuvent opter pour une identité d’entreprise définie mutuellement. Pour ces scénarios, sélectionnez **Défini mutuellement** pour EDIFACT ou **Défini mutuellement (X12)** pour X12. <p>Pour RosettaNet, sélectionnez uniquement **DUNS**, qui est la norme. |
   | **Valeur** | Oui | Valeur qui identifie les documents que vos applications logiques reçoivent. <p>Pour RosettaNet, cette valeur doit être un nombre à neuf chiffres qui correspond au numéro DUNS. |
   ||||

   > [!NOTE]
   > Pour les partenaires qui utilisent RosettaNet, vous pouvez spécifier des informations supplémentaires en créant d’abord ces partenaires, puis [en les modifiant par la suite](#edit-partner).

1. Une fois que vous avez terminé, sélectionnez **OK**.

   Votre nouveau partenaire apparaît maintenant dans la liste **Partenaires**. En outre, la vignette **Partenaires** met à jour le nombre actuel de partenaires.

   ![Nouveau partenaire](./media/logic-apps-enterprise-integration-partners/new-partner.png)

<a name="edit-partner"></a>

## <a name="edit-partner"></a>Modifier un partenaire

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez votre compte d’intégration.
Choisissez la mosaïque **Partenaires**.

   ![Choisir la mosaïque « Partenaires »](./media/logic-apps-enterprise-integration-partners/edit.png)

1. Sous **Partenaires**, sélectionnez le partenaire à modifier, puis choisissez **Modifier**. Sous **Modifier**, apportez vos modifications.

   ![Effectuer et enregistrer vos modifications](./media/logic-apps-enterprise-integration-partners/edit-partner.png)

   Pour RosettaNet, sous **Propriétés du partenaire RosettaNet**, vous pouvez spécifier ces informations supplémentaires :

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Classification partenaire** | Non | Type d’organisation du partenaire |
   | **Code de la chaîne d’approvisionnement** | Non | Code de la chaîne d’approvisionnement du partenaire, par exemple « Technologies de l’information » ou « Composants électroniques » |
   | **Nom du contact** | Non | Nom de contact du partenaire |
   | **E-mail** | Non | Adresse e-mail du partenaire |
   | **Fax** | Non | Numéro de télécopie du partenaire |
   | **Téléphone** | Non | Numéro de téléphone du partenaire |
   ||||

1. Lorsque vous avez terminé, choisissez **OK** pour enregistrer vos modifications.

## <a name="delete-partner"></a>Supprimer un partenaire

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez votre compte d’intégration. Choisissez la mosaïque **Partenaires**.

   ![Capture d’écran montrant la vignette Partenaires que vous sélectionnez lorsque vous souhaitez supprimer un partenaire.](./media/logic-apps-enterprise-integration-partners/choose-partners-to-delete.png)

1. Sous **Partenaires**, sélectionnez le partenaire à supprimer. Choisissez **Supprimer**.

   ![Supprimer un partenaire](./media/logic-apps-enterprise-integration-partners/delete-partner.png)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [contrats](../logic-apps/logic-apps-enterprise-integration-agreements.md)