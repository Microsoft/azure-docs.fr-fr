---
title: Ajouter des ressources à des environnements de service d’intégration
description: Ajoutez des applications logiques, des comptes d’intégration, des connecteurs personnalisés et des connecteurs managés à votre environnement de service d’intégration.
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 02/28/2020
ms.openlocfilehash: 147247c663311cfb3e05a986c6fb2bffbb41158b
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92675203"
---
# <a name="add-resources-to-your-integration-service-environment-ise-in-azure-logic-apps"></a>Ajouter des ressources à votre environnement de service d’intégration (ISE) dans Azure Logic Apps

Après avoir créé un [environnement de service d’intégration](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) (ISE, Integration Service Environment), ajoutez des ressources telles que des applications logiques, des comptes d’intégration et des connecteurs pour qu’ils puissent accéder aux ressources de votre réseau virtuel Azure. Par exemple, les connecteurs ISE managés qui deviennent disponibles une fois que vous avez créé votre ISE n’apparaissent pas automatiquement dans le concepteur d’applications logiques. Avant de pouvoir utiliser ces connecteurs ISE, vous devez manuellement [ajouter et déployer ces connecteurs dans votre ISE](#add-ise-connectors-environment) afin qu’ils apparaissent dans le concepteur d’applications logiques.

> [!IMPORTANT]
> Pour que les applications logiques et les comptes d’intégration fonctionnent ensemble dans un environnement ISE, il faut que tous deux utilisent le *même ISE* que leur emplacement.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* L’environnement ISE que vous avez créé pour exécuter vos applications logiques. Si vous ne disposez pas d’un environnement ISE, commencez par [créer cet environnement](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

* Pour créer, ajouter ou mettre à jour des ressources qui sont déployées dans un ISE, vous devez disposer du rôle Propriétaire ou Contributeur sur cet ISE, ou avoir des autorisations héritées via l’abonnement Azure ou le groupe de ressources Azure associé à l’ISE. Les personnes qui ne disposent pas d’autorisations de Propriétaire ou de Contributeur, ou d’autorisations héritées, peuvent se voir attribuer le rôle Contributeur de l’environnement de service d’intégration ou Développeur de l’environnement de service d’intégration. Pour plus d’informations, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../role-based-access-control/overview.md) ?

<a name="create-logic-apps-environment"></a>

## <a name="create-logic-apps"></a>Créer des applications logiques

Pour créer des applications logiques qui s’exécutent dans votre environnement ISE, effectuez les étapes suivantes :

1. Recherchez et ouvrez votre environnement ISE, s’il n’est pas encore ouvert. Dans le menu de l’environnement ISE, sous **Paramètres** , sélectionnez **Applications logiques** > **Ajouter**.

   ![Ajouter une nouvelle application logique à l’environnement ISE](./media/add-artifacts-integration-service-environment-ise/add-logic-app-to-ise.png)

1. Fournissez des informations sur l’application logique que vous souhaitez créer, par exemple :

   ![Capture d’écran montrant la fenêtre de création d’une application logique avec des exemples d’informations entrés](./media/add-artifacts-integration-service-environment-ise/create-logic-app-integration-service-environment.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Nom** | Oui | Nom de l’application logique à créer |
   | **Abonnement** | Oui | Nom de l’abonnement Azure à utiliser. |
   | **Groupe de ressources** | Oui | Nom du groupe de ressources Azure (nouveau ou existant) à utiliser |
   | **Lieu** | Oui | Sous **Environnements de service d’intégration** , sélectionnez l’ISE à utiliser, s’il n’est pas déjà sélectionné. <p><p> **Important !** Pour utiliser vos applications logiques avec un compte d’intégration, tous doivent utiliser le même ISE. |
   ||||

1. Sélectionnez **Créer** lorsque vous avez terminé.

1. Continuez à [créer votre application logique de la façon habituelle](../logic-apps/quickstart-create-first-logic-app-workflow.md).

   Pour connaître les différences dans le fonctionnement des déclencheurs et des actions et dans la façon dont ils sont étiquetés quand vous utilisez un ISE par rapport au service Logic Apps multilocataire, consultez [Isolé ou multilocataire dans la vue d’ensemble de l’ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#difference).

1. Pour gérer les applications logiques et les connexions d’API dans votre environnement ISE, consultez [Gérer votre environnement de service d’intégration](../logic-apps/ise-manage-integration-service-environment.md).

<a name="create-integration-account-environment"></a>

## <a name="create-integration-accounts"></a>Créer des comptes d’intégration

En fonction de la [référence SKU d’environnement ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level) sélectionnée lors de la création, votre environnement ISE comprend une utilisation de compte d’intégration spécifique sans coût supplémentaire. Les applications logiques qui existent dans un environnement ISE peuvent référencer uniquement des comptes d’intégration qui existent dans le même environnement ISE. Ainsi, pour qu’un compte d’intégration fonctionne avec des applications logiques dans un environnement ISE, le compte d’intégration et les applications logiques doivent utiliser le *même environnement* comme emplacement. Pour plus d’informations sur les comptes d’intégration et les environnements ISE, consultez [Comptes d’intégration et environnement de service d’intégration](connect-virtual-network-vnet-isolated-environment-overview.md#create-integration-account-environment).

Pour créer un compte d’intégration qui utilise un environnement ISE, effectuez les étapes suivantes :

1. Recherchez et ouvrez votre environnement ISE, s’il n’est pas encore ouvert. Dans le menu de l’environnement ISE, sous **Paramètres** , sélectionnez **Comptes d’intégration** > **Ajouter**.

   ![Ajouter un nouveau compte d’intégration à un environnement ISE](./media/add-artifacts-integration-service-environment-ise/add-integration-account-to-ise.png)

1. Fournissez des informations sur l’application logique que vous souhaitez créer, par exemple :

   ![Sélection d’un environnement de service d’intégration](./media/add-artifacts-integration-service-environment-ise/create-integration-account-integration-service-environment.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Nom** | Oui | Nom du compte d’intégration que vous souhaitez créer |
   | **Abonnement** | Oui | Nom de l’abonnement Azure que vous souhaitez utiliser |
   | **Groupe de ressources** | Oui | Nom du groupe de ressources Azure (nouveau ou existant) à utiliser |
   | **Niveau tarifaire** | Oui | Niveau tarifaire à utiliser pour le compte d’intégration |
   | **Lieu** | Oui | Sous **Environnements de service d’intégration** , sélectionnez le même ISE que celui utilisé par vos applications logiques, s’il n’est pas déjà sélectionné. <p><p> **Important !** Pour utiliser votre compte d’intégration avec des applications logiques, tous doivent utiliser le même ISE. |
   ||||

1. Sélectionnez **Créer** lorsque vous avez terminé.

1. [Liez votre application logique à votre compte d’intégration de la façon habituelle](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account).

1. Continuez en ajoutant des ressources à votre compte d’intégration, telles que des [parties](../logic-apps/logic-apps-enterprise-integration-partners.md) et des [contrats](../logic-apps/logic-apps-enterprise-integration-agreements.md).

1. Pour gérer les comptes d’intégration dans votre environnement ISE, consultez [Gérer votre environnement de service d’intégration](../logic-apps/ise-manage-integration-service-environment.md).

<a name="add-ise-connectors-environment"></a>

## <a name="add-ise-connectors"></a>Ajouter des connecteurs ISE

Une fois que vous avez créé votre ISE, les connecteurs ISE managés n’apparaissent pas automatiquement dans le sélecteur de connecteur du concepteur d’applications logiques. Avant de pouvoir utiliser ces connecteurs ISE, vous devez manuellement ajouter et déployer ces connecteurs dans votre ISE afin qu’ils apparaissent dans le concepteur d’applications logiques.

> [!IMPORTANT]
> Pour le moment, les connecteurs ISE managés ne prennent pas en charge les [balises](../azure-resource-manager/management/tag-support.md). Si vous configurez une stratégie qui applique le balisage, la tentative d’ajout de connecteurs ISE peut échouer avec une erreur semblable à l’exemple suivant :
> 
> ```json
> {
>    "error": { 
>       "code": "IntergrationServiceEnvironmentManagedApiDefinitionTagsNotSupported", 
>       "message": "The tags are not supported in the managed API 'azureblob'."
>    }
> }
> ```
> 
> Ainsi, pour ajouter des connecteurs ISE, vous devez désactiver ou supprimer votre stratégie. 

1. Dans le menu de votre environnement ISE, sous **Paramètres** , sélectionnez **Connecteurs managés**. Dans la barre d’outils, sélectionnez **Ajouter**.

   ![Voir les connecteurs managés](./media/add-artifacts-integration-service-environment-ise/ise-view-managed-connectors.png)

1. Dans le volet **Ajouter un nouveau connecteur managé** , ouvrez la liste **Rechercher un connecteur**. Sélectionnez le connecteur ISE que vous souhaitez utiliser, mais qui n’est pas encore déployé dans votre ISE. Sélectionnez **Create** (Créer).

   ![Sélectionnez le connecteur ISE que vous souhaitez déployer dans votre ISE.](./media/add-artifacts-integration-service-environment-ise/add-managed-connector.png)

   Seuls les connecteurs ISE qui sont admissibles, mais qui n’ont pas encore été déployés dans votre ISE apparaissent comme disponibles. Les connecteurs déjà déployés dans votre environnement ISE apparaissent comme étant indisponibles pour la sélection.

<a name="create-custom-connectors-environment"></a>

## <a name="create-custom-connectors"></a>Créer des connecteurs personnalisés

Pour utiliser des connecteurs personnalisés dans votre environnement ISE, créez-les directement dans l’environnement.

1. Recherchez et ouvrez votre environnement ISE, s’il n’est pas encore ouvert. Dans le menu de l’environnement ISE, sous **Paramètres** , sélectionnez **Connecteurs personnalisés** > **Ajouter**.

   ![Créer un connecteur personnalisé](./media/add-artifacts-integration-service-environment-ise/add-custom-connector-to-ise.png)

1. Indiquez le nom, l’abonnement Azure et le groupe de ressources Azure (nouveau ou existant) à utiliser pour votre connecteur personnalisé.

1. Dans la liste **Emplacement** , sous la section **Environnements de service d’intégration** , sélectionnez le même environnement ISE que celui utilisé par vos applications logiques et sélectionnez **Créer** , par exemple :

   ![Capture d’écran montrant la fenêtre « Créer un connecteur personnalisé Logic Apps » avec des exemples d’informations sélectionnés](./media/add-artifacts-integration-service-environment-ise/create-custom-connector-integration-service-environment.png)

1. Sélectionnez votre nouveau connecteur personnalisé, puis sélectionnez **Modifier** , par exemple :

   ![Sélectionner et modifier un connecteur personnalisé](./media/add-artifacts-integration-service-environment-ise/edit-custom-connectors.png)

1. Continuez en créant le connecteur de la manière habituelle à partir d’une [définition OpenAPI](/connectors/custom-connectors/define-openapi-definition#import-the-openapi-definition) ou du [protocole SOAP](/connectors/custom-connectors/create-register-logic-apps-soap-connector#2-define-your-connector).

1. Pour gérer les connecteurs personnalisés dans votre environnement ISE, consultez [Gérer votre environnement de service d’intégration](../logic-apps/ise-manage-integration-service-environment.md).

## <a name="next-steps"></a>Étapes suivantes

* [Gérer les environnements de service d’intégration](../logic-apps/ise-manage-integration-service-environment.md)
