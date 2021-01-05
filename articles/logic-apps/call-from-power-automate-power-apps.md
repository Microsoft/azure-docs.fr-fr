---
title: Appeler des applications logiques à partir de Power Automate et Power Apps
description: Appelez des applications logiques à partir de flux Microsoft Power Automate en les exportant en tant que connecteurs.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: conceptual
ms.date: 10/05/2020
ms.openlocfilehash: b402dab4c6e94a7634e11f0330b5379315e43abf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91762408"
---
# <a name="call-logic-apps-from-power-automate-and-power-apps"></a>Appeler des applications logiques à partir de Power Automate et Power Apps

Pour appeler vos applications logiques à partir de Microsoft Power Automate et Microsoft Power Apps, vous pouvez les exporter en tant que connecteurs. Quand vous exposez une application logique en tant que connecteur personnalisé dans un environnement Power Automate ou Power Apps, vous pouvez l’y appeler à partir de flux.

Si vous souhaitez migrer votre flux de Power Automate ou Power Apps vers Logic Apps à la place, consultez [Exporter des flux de Power Automate et les déployer sur Azure Logic Apps](export-from-microsoft-flow-logic-app-template.md).

> [!NOTE]
> Les connecteurs Power Automate ne sont pas tous disponibles dans Azure Logic Apps. Vous pouvez migrer uniquement les flux Power Automate qui ont les connecteurs équivalents dans Azure Logic Apps. Par exemple, le déclencheur de bouton, le connecteur d’approbation et le connecteur de notification sont propres à Power Automate. Les flux basés sur OpenAPI dans Power Automate ne sont pas pris en charge pour l’exportation et le déploiement en tant que modèles d’application logique.
>
> * Pour connaître les connecteurs Power Automate qui n’ont pas d’équivalents Logic Apps, consultez [Connecteurs Power Automate](/connectors/connector-reference/connector-reference-powerautomate-connectors).
>
> * Pour connaître les connecteurs Logic Apps qui n’ont pas d’équivalents Power Automate, consultez [Connecteurs Logic Apps](/connectors/connector-reference/connector-reference-powerautomate-connectors).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Une licence Power Automate ou Power Apps.

* Une application logique avec un déclencheur de demande pour effectuer l’exportation.

* Un flux dans Power Automate ou Power Apps à partir duquel vous souhaitez appeler votre application logique.

## <a name="export-your-logic-app-as-a-custom-connector"></a>Exporter votre application logique en tant que connecteur personnalisé

Avant de pouvoir appeler votre application logique à partir de Power Automate ou Power Apps, vous devez l’exporter en tant que connecteur personnalisé.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la zone de recherche du portail Azure, entrez `Logic Apps`. Dans les résultats, sous **Services**, sélectionnez **Applications logiques**.

1. Sélectionnez l’application logique que vous souhaitez exporter.

1. Dans le menu de votre application logique, sélectionnez **Exporter**.

    :::image type="content" source="./media/call-logic-apps-from-power-automate-power-apps/export-logic-app.png" alt-text="Capture d’écran de la page de l’application logique dans le portail Azure, montrant le menu avec le bouton « Exporter » sélectionné":::

1. Dans le volet **exporter**, pour **Nom**, entrez un nom pour le connecteur personnalisé de votre application logique. Dans la liste **Environnement**, sélectionnez l’environnement Power Automate ou Power Apps à partir duquel vous souhaitez appeler votre application logique. Quand vous avez terminé, sélectionnez **OK**.

    :::image type="content" source="./media/call-logic-apps-from-power-automate-power-apps/export-logic-app2.png" alt-text="Capture d’écran de la page de l’application logique dans le portail Azure, montrant le menu avec le bouton « Exporter » sélectionné":::

1. Pour vérifier que votre application logique a été correctement exportée, consultez le volet des notifications.

### <a name="exporting-errors"></a>Erreurs d’exportation

Voici certaines erreurs qui peuvent se produire quand vous exportez votre application logique en tant que connecteur personnalisé et les solutions suggérées correspondantes :

* **L’obtention des environnements a échoué. Vérifiez que votre compte est configuré pour Power Automate et Power Apps, puis réessayez.**  : Vérifiez que votre compte Azure dispose d’un plan Power Automate.

* **L’application logique actuelle ne peut pas être exportée. Pour effectuer une exportation, sélectionnez une application logique qui a un déclencheur de demande.**  : Vérifiez que votre application logique commence par un [déclencheur de demande](./logic-apps-workflow-actions-triggers.md#request-trigger).

## <a name="connect-to-your-logic-app-from-power-automate"></a>Se connecter à l’application logique à partir de Power Automate

Pour vous connecter à l’application logique que vous avez exportée avec votre flux Power Automate :

1. Connectez-vous à [Power Automate](https://flow.microsoft.com).

1. Dans le menu de la page d’accueil de **Power Automate**, sélectionnez **Mes flux**.

1. Dans la page **Flux**, sélectionnez le flux que vous souhaitez connecter à votre application logique.

1. Dans le menu de la page du flux, sélectionnez **Modifier**.

1. Dans l’éditeur de flux, sélectionnez **&#43; Nouvelle étape**.

1. Sous **Choisir une action**, dans la zone de recherche, entrez le nom du connecteur de votre application logique. Si vous le souhaitez, pour afficher uniquement les connecteurs personnalisés dans votre environnement, filtrez les résultats en sélectionnant l’onglet **Personnalisé**.

    :::image type="content" source="./media/call-logic-apps-from-power-automate-power-apps/power-automate-custom-connector-action.png" alt-text="Capture d’écran de la page de l’application logique dans le portail Azure, montrant le menu avec le bouton « Exporter » sélectionné":::

1. Sélectionnez l’action que vous souhaitez effectuer avec le connecteur de votre application logique. 

1. Fournissez les informations que l’action passe au connecteur de l’application logique.

1. Pour enregistrer vos modifications, dans le menu de l’éditeur de Power Automate, sélectionnez **Enregistrer**.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le service Logic Apps, recherchez l’application logique que vous avez exportée.

1. Vérifiez que votre application logique fonctionne comme prévu dans votre flux Power Automate.

## <a name="delete-logic-app-connector-from-power-automate"></a>Supprimer le connecteur de l’application logique de Power Automate

1. Connectez-vous à [Power Automate](https://flow.microsoft.com).

1. Dans la page d’accueil de **Power Automate**, sélectionnez **Données** &gt; **Connecteurs personnalisés** dans le menu.

1. Dans la liste, recherchez votre connecteur personnalisé, puis sélectionnez le bouton de sélection ( **...** ) &gt; **Supprimer**.

    :::image type="content" source="./media/call-logic-apps-from-power-automate-power-apps/delete-custom-connector.png" alt-text="Capture d’écran de la page de l’application logique dans le portail Azure, montrant le menu avec le bouton « Exporter » sélectionné":::

1. Sélectionnez **OK** pour confirmer la suppression.

## <a name="connect-to-your-logic-app-from-power-apps"></a>Se connecter à l’application logique à partir de Power Apps

Pour vous connecter à l’application logique que vous avez exportée avec votre flux Power Apps :

1. Connectez-vous à [Power Apps](https://powerapps.microsoft.com/).

1. Dans la page d’accueil de **Power Apps**, sélectionnez **Flux** dans le menu.

1. Dans la page **Flux**, sélectionnez le flux que vous souhaitez connecter à votre application logique.

1. Dans la page de votre flux, sélectionnez **Modifier** dans le menu du flux.

1. Dans l’éditeur de flux, sélectionnez le bouton **&#43; Nouvelle étape**.

1. Sous **Choisir une action**, dans la nouvelle étape, entrez le nom du connecteur de votre application logique dans la zone de recherche. Si vous le souhaitez, filtrez les résultats au moyen de l’onglet **Personnalisé** pour voir uniquement les connecteurs personnalisés dans votre environnement.

    :::image type="content" source="./media/call-logic-apps-from-power-automate-power-apps/power-apps-custom-connector-action.png" alt-text="Capture d’écran de la page de l’application logique dans le portail Azure, montrant le menu avec le bouton « Exporter » sélectionné":::

1. Sélectionnez l’action que vous souhaitez effectuer avec le connecteur. 

1. Configurez les informations que votre action passe au connecteur de l’application logique.

1. Dans le menu de l’éditeur de Power Apps, sélectionnez **Enregistrer** pour enregistrer vos modifications. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le service Logic Apps, recherchez l’application logique que vous avez exportée.

1. Vérifiez que votre application logique fonctionne comme prévu avec votre flux Power Apps.

## <a name="delete-logic-app-connector-from-power-apps"></a>Supprimer le connecteur de l’application logique de Power Apps

1. Connectez-vous à [Power Apps](https://powerapps.microsoft.com).

1. Dans la page d’accueil de **Power Apps**, sélectionnez **Données** &gt; **Connecteurs personnalisés** dans le menu.

1. Dans la liste, recherchez votre connecteur personnalisé, puis sélectionnez le bouton de sélection ( **...** ) &gt; **Supprimer**.

    :::image type="content" source="./media/call-logic-apps-from-power-automate-power-apps/delete-custom-connector.png" alt-text="Capture d’écran de la page de l’application logique dans le portail Azure, montrant le menu avec le bouton « Exporter » sélectionné":::

1. Sélectionnez **OK** pour confirmer la suppression.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [les autres connecteurs pour Azure Logic Apps](../connectors/apis-list.md)
* En savoir plus sur [Azure Logic Apps](../logic-apps/logic-apps-overview.md)
