---
title: Modifier et gérer des applications logiques à l’aide de Visual Studio avec Cloud Explorer
description: Modifier, mettre à jour, gérer, ajouter au contrôle de code source et déployer des applications logiques à l’aide de Visual Studio avec Cloud Explorer
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: article
ms.custom: mvc
ms.date: 04/29/2020
ms.openlocfilehash: cde8db5310e3ede2721ba327b28c789ccd0b7dd0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87280764"
---
# <a name="manage-logic-apps-with-visual-studio"></a>Gérer des applications logiques avec Visual Studio

Même si vous pouvez créer, modifier, gérer et déployer des applications logiques dans le [portail Azure](https://portal.azure.com), vous pouvez aussi utiliser Visual Studio pour ajouter vos applications logiques au contrôle de code source, publier des versions différentes et créer des modèles [Azure Resource Manager](../azure-resource-manager/management/overview.md) pour plusieurs environnements de déploiement. Avec Visual Studio Cloud Explorer, vous pouvez rechercher et gérer vos applications logiques, ainsi que d’autres ressources Azure. Par exemple, vous pouvez ouvrir, télécharger, modifier, exécuter, désactiver et activer des applications logiques déjà déployées dans le portail Azure ou encore en afficher l’historique des exécutions. Si vous ne savez pas utiliser Azure Logic Apps dans Visual Studio, apprenez à [créer des applications logiques avec Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

Vous pouvez également [gérer vos applications logiques dans le portail Azure](manage-logic-apps-with-azure-portal.md).

> [!IMPORTANT]
> Le déploiement ou la publication d’une application logique à partir de Visual Studio remplace la version de cette application dans le portail Azure. Par conséquent, si vous apportez dans le portail Azure des modifications que vous souhaitez conserver, veillez à [actualiser l’application logique dans Visual Studio](#refresh) à partir du portail Azure avant son prochain déploiement ou sa prochaine publication à partir de Visual Studio.

<a name="requirements"></a>

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Téléchargez et installez ces outils, si vous ne les avez pas déjà :

  * [Visual Studio 2019, 2017 ou 2015 - édition Community ou supérieure](https://aka.ms/download-visual-studio). Ce démarrage rapide utilise Visual Studio Community 2017, qui est gratuit.

    > [!IMPORTANT]
    > Quand vous installez Visual Studio 2019 ou 2017, veillez à sélectionner la charge de travail **Développement Azure**.
    > Pour plus d’informations, voir [Gérer les ressources associées à vos comptes Azure dans Visual Studio Cloud Explorer](/visualstudio/azure/vs-azure-tools-resources-managing-with-cloud-explorer).

    Pour installer Cloud Explorer pour Visual Studio 2015, [téléchargez Cloud Explorer à partir de Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=MicrosoftCloudExplorer.CloudExplorerforVisualStudio2015). Pour plus d’informations, voir [Gérer les ressources associées à vos comptes Azure dans Visual Studio Cloud Explorer (2015)](/visualstudio/azure/vs-azure-tools-resources-managing-with-cloud-explorer?view=vs-2015).

  * [Azure SDK (2.9.1 ou version ultérieure)](https://azure.microsoft.com/downloads/)

  * [Azure PowerShell](https://github.com/Azure/azure-powershell#installation)

  * Les derniers outils d’Azure Logic Apps pour l’extension Visual Studio correspondant à la version que vous souhaitez :

    * [Visual Studio 2019](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2019)

    * [Visual Studio 2017](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2017)

    * [Visual Studio 2015](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2015)

    Vous pouvez télécharger et installer les outils Azure Logic Apps directement à partir de Visual Studio Marketplace ou en apprendre davantage sur [l’installation de cette extension dans Visual Studio](/visualstudio/ide/finding-and-using-visual-studio-extensions). Veillez à redémarrer Visual Studio après l’installation.

  * Pour utiliser des abonnements Azure Government avec Visual Studio, consultez les rubriques suivantes, qui indiquent la procédure de configuration supplémentaire à effectuer :

    * Visual Studio 2019 : [Démarrage rapide : Connexion à Azure Government avec Visual Studio](../azure-government/documentation-government-connect-vs.md)

    * Visual Studio 2017 : [Introducing the Azure Environment Selector Visual Studio extension](https://devblogs.microsoft.com/azuregov/introducing-the-azure-environment-selector-visual-studio-extension/) ; vous pouvez télécharger et installer l’extension à partir de [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=SteveMichelotti.AzureEnvironmentSelector).

* Accès au web lors de l’utilisation du concepteur Logic Apps intégré

  Le Concepteur requiert une connexion Internet pour créer des ressources dans Azure et pour lire les propriétés et les données à partir de connecteurs dans votre application logique.

<a name="find-logic-apps-vs"></a>

## <a name="find-logic-apps"></a>Rechercher les applications logiques

Dans Visual Studio, vous pouvez trouver toutes les applications logiques associées à votre abonnement Azure et déployées sur le portail Azure à l’aide de Cloud Explorer.

1. Ouvrez Visual Studio. Dans le menu **Affichage**, sélectionnez **Cloud Explorer**.

1. Dans Cloud Explorer, sélectionnez l’icône **Gestion des comptes**. Sélectionnez l’abonnement Azure associé à vos applications logiques et sélectionnez **Appliquer**. Par exemple :

   ![Sélectionnez « Gestion des comptes ».](./media/manage-logic-apps-with-visual-studio/account-management-select-Azure-subscription.png)

1. À côté de l’icône **Gestion des comptes**, sélectionnez **Types de ressources**. Sous votre abonnement Azure, développez **Logic Apps** afin de pouvoir voir toutes les applications logiques déployées, associées à votre abonnement.

Ensuite, ouvrez votre application logique dans l’éditeur d’application logique.

<a name="open-designer"></a>

## <a name="open-logic-apps-in-visual-studio"></a>Ouvrir des applications logiques dans Visual Studio

Dans Visual Studio, vous pouvez ouvrir des applications logiques précédemment créées et déployées directement via le portail Azure ou en tant que projets Azure Resource Group avec Visual Studio.

1. [Ouvrez Cloud Explorer et recherchez votre application logique](#find-logic-apps-vs).

1. Dans le menu contextuel de l’application logique, sélectionnez **Ouvrir avec l’éditeur d’application logique**.

   > [!TIP]
   > Si vous ne voyez pas cette commande dans Visual Studio 2019, vérifiez que vous avez les dernières mises à jour pour Visual Studio.

   ![Ouvrir l’application logique déployée à partir du portail Azure](./media/manage-logic-apps-with-visual-studio/open-logic-app-in-editor.png)

   Une fois l'application logique ouverte dans le Concepteur d'applications logiques, au bas de celui-ci, vous pouvez sélectionner **Mode Code** pour examiner la structure de définition de l'application logique sous-jacente. Pour créer un modèle de déploiement pour l’application logique, apprenez à [télécharger un modèle Azure Resource Manager](#download-logic-app) pour cette application logique. Découvrez-en plus sur les [modèles Resource Manager](../azure-resource-manager/templates/overview.md).

<a name="download-logic-app"></a>

## <a name="download-from-azure"></a>Télécharger à partir d’Azure

Vous pouvez [télécharger](../azure-resource-manager/templates/export-template-portal.md#export-template-from-a-resource) des applications logiques à partir du [portail Azure](https://portal.azure.com) et les enregistrer en tant que modèles [Azure Resource Manager](../azure-resource-manager/management/overview.md). Vous pouvez ensuite modifier localement les modèles avec Visual Studio et personnaliser les applications logiques pour différents environnements de déploiement.  Le téléchargement d’applications logiques *paramètre* automatiquement leurs définitions dans des [modèles Resource Manager](../azure-resource-manager/templates/overview.md), qui utilisent également JSON (JavaScript Objet Notation).

1. Dans Visual Studio, avec Cloud Explorer, [ouvrez l’application logique à télécharger à partir d’Azure](#open-designer).

1. Dans le menu contextuel de l’application logique, sélectionnez **Ouvrir avec l’éditeur d’application logique**.

   > [!TIP]
   > Si vous ne voyez pas cette commande dans Visual Studio 2019, vérifiez que vous avez les dernières mises à jour pour Visual Studio.

   L’application logique s’ouvre dans l’éditeur d’application logique.

1. Sur la barre d'outils du Concepteur, sélectionnez **Télécharger**.

   ![Télécharger l’application logique à partir du Portail Azure](./media/manage-logic-apps-with-visual-studio/download-logic-app-from-portal.png)

1. Lorsque vous êtes invité à entrer un emplacement, accédez à cet emplacement et enregistrez le modèle Resource Manager de la définition d’application logique au format de fichier JSON (.json).

   Votre définition d’application logique apparaît dans la sous-section `resources` à l’intérieur du modèle Resource Manager. Vous pouvez maintenant modifier la définition d’application logique et le modèle Resource Manager avec Visual Studio. Vous pouvez également ajouter le modèle en tant que projet [Azure Resource Group](../azure-resource-manager/templates/create-visual-studio-deployment-project.md) à une solution Visual Studio. Découvrez plus d’informations sur les [projets Azure Resource Group pour les applications logiques dans Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

<a name="link-integration-account"></a>

## <a name="link-to-integration-account"></a>Lier à un compte d’intégration

Pour créer des applications logiques pour des scénarios d’intégration d’entreprise B2B (Business-to-Business), vous pouvez lier votre application logique à un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) précédemment créé qui existe dans la même région que votre application logique. Un compte d’intégration contient des artefacts B2B, comme des partenaires commerciaux, des contrats, des schémas et des mappages, et permet à votre application logique d’utiliser des connecteurs B2B pour la validation XML, et l’encodage ou le décodage de fichier plat. Vous pouvez [créer ce lien en utilisant le portail Azure](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account), mais vous pouvez aussi utiliser Visual Studio après avoir satisfait aux [prérequis](#requirements) : votre application logique existe en tant que fichier JSON (.json) à l’intérieur d’un projet [Azure Resource Group](../azure-resource-manager/templates/create-visual-studio-deployment-project.md). Découvrez plus d’informations sur les [projets Azure Resource Group pour les applications logiques dans Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#create-resource-group-project).

1. Dans Visual Studio, ouvrez le projet Azure Resource Group qui contient votre application logique.

1. Dans l’Explorateur de solutions, ouvrez le menu contextuel du fichier **<nom_application_logique>.json**, puis sélectionnez **Ouvrir avec le concepteur d’application logique**. (Clavier : Ctrl + L)

   ![Ouvrir le fichier .json de l’application logique avec le concepteur d’application logique](./media/manage-logic-apps-with-visual-studio/open-logic-app-designer.png)

   > [!TIP]
   > Si vous ne voyez pas cette commande dans Visual Studio 2019, vérifiez que vous avez les dernières mises à jour pour Visual Studio et l’extension Azure Logic Apps Tools.

1. Assurez-vous que le concepteur d’application logique a le focus en sélectionnant l’onglet ou la surface du concepteur, afin que la fenêtre Propriétés montre la propriété **Compte d’intégration** pour votre application logique.

   ![Fenêtre Propriétés - Propriété « Compte d’intégration »](./media/manage-logic-apps-with-visual-studio/open-logic-app-properties-integration-account.png)

   > [!TIP]
   > Si la fenêtre Propriétés n’est pas déjà ouverte, dans le menu **Afficher**, sélectionnez **Fenêtre Propriétés**. (Clavier : appuyez sur F4)

1. Ouvrez la liste de propriétés **Compte d’intégration**, puis sélectionnez le compte d’intégration que vous voulez lier à votre application logique, par exemple :

   ![Ouvrir la liste de propriétés « Compte d’intégration »](./media/manage-logic-apps-with-visual-studio/select-integration-account.png)

1. Quand vous avez terminé, n’oubliez pas d’enregistrer votre solution Visual Studio.

Quand vous définissez la propriété **Compte d’intégration** dans Visual Studio et que vous enregistrez votre application logique en tant que modèle Azure Resource Manager, ce modèle comprend également une déclaration de paramètre pour le compte d’intégration sélectionné. Pour plus d’informations sur les paramètres de modèle et les applications logiques, consultez [Vue d’ensemble : Automatiser le déploiement d’applications logiques](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#template-parameters).

<a name="change-location"></a>

## <a name="change-deployment-location"></a>Modifier l’emplacement de déploiement

Dans Visual Studio, si votre application logique existe en tant que fichier JSON (.json) au sein d’un [projet de groupe de ressources Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md) que vous utilisez pour automatiser le déploiement, cette application logique est définie sur un type d’emplacement et un emplacement spécifique. Cet emplacement est soit une région Azure, soit un [environnement de service d’intégration (ISE)](connect-virtual-network-vnet-isolated-environment.md) existant.

Pour modifier le type d’emplacement ou l’emplacement de votre application logique, vous devez ouvrir le fichier de définition de flux de travail de votre application logique (.json) à partir de l’Explorateur de solutions à l’aide du Concepteur d’application logique. Vous ne pouvez pas modifier ces propriétés à l’aide de Cloud Explorer.

> [!IMPORTANT]
> La modification du type d’emplacement de **Région** à [**Environnement de service d’intégration**](connect-virtual-network-vnet-isolated-environment-overview.md) affecte le [modèle de tarification](logic-apps-pricing.md#fixed-pricing) de votre application logique lequel est utilisé pour la facturation, [les limites](logic-apps-limits-and-config.md#integration-account-limits), [la prise en charge des comptes d’intégration](connect-virtual-network-vnet-isolated-environment-overview.md#ise-skus), etc. Avant de sélectionner un autre type d’emplacement, assurez-vous que vous comprenez l’impact qui en résulte sur votre application logique.

1. Dans Visual Studio, ouvrez le projet Azure Resource Group qui contient votre application logique.

1. Dans l’Explorateur de solutions, ouvrez le menu contextuel du fichier `<logic-app-name>.json` et sélectionnez **Ouvrir avec le concepteur d’application logique**. (Clavier : Ctrl + L)

   ![Ouvrir le fichier .json de l’application logique avec le concepteur d’application logique](./media/manage-logic-apps-with-visual-studio/open-logic-app-designer.png)

   > [!TIP]
   > Si vous ne voyez pas cette commande dans Visual Studio 2019, vérifiez que vous avez les dernières mises à jour pour Visual Studio et l’extension Azure Logic Apps Tools.

1. Assurez-vous que le concepteur d’application logique a le focus en sélectionnant l’onglet ou la surface du concepteur, afin que la fenêtre Propriétés montre les propriétés **Choisir un type d’emplacement** et **Emplacement** pour votre application logique. Le type d’emplacement du projet est défini sur **Région** ou **Environnement de service d’intégration**.

   ![Fenêtre Propriétés : « Choisir un type d’emplacement » et propriétés « Emplacement »](./media/manage-logic-apps-with-visual-studio/open-logic-app-properties-location.png)

   > [!TIP]
   > Si la fenêtre Propriétés n’est pas déjà ouverte, dans le menu **Afficher**, sélectionnez **Fenêtre Propriétés**. (Clavier : appuyez sur F4)

1. Pour modifier le type d’emplacement, ouvrez la liste de propriétés **Choisir un type d’emplacement**, puis sélectionnez le type d’emplacement de votre choix.

   Par exemple, si le type d’emplacement est **Environnement de service d’intégration**, vous pouvez sélectionner **Région**.

   ![Propriété « Choisir un type d’emplacement » : modifier le type d’emplacement](./media/manage-logic-apps-with-visual-studio/change-location-type.png)

1. Pour modifier l’emplacement spécifique, ouvrez la liste des propriétés **Emplacement**. En fonction du type d’emplacement, sélectionnez l’emplacement de votre choix, par exemple :

   * Sélectionner une autre région Azure :

     ![Ouvrez la liste de propriétés « Emplacement », puis sélectionnez une autre région Azure](./media/manage-logic-apps-with-visual-studio/change-azure-resource-group-region.png)

   * Sélectionnez un autre environnement ISE :

     ![Ouvrez la liste de propriétés « Emplacement », puis sélectionnez un autre environnement ISE](./media/manage-logic-apps-with-visual-studio/change-integration-service-environment.png)

1. Quand vous avez terminé, n’oubliez pas d’enregistrer votre solution Visual Studio.

Lorsque vous modifiez le type d’emplacement ou l’emplacement dans Visual Studio et que vous enregistrez votre application logique en tant que modèle Azure Resource Manager, ce modèle comprend également des déclarations de paramètres pour ce type d’emplacement et pour cet emplacement. Pour plus d’informations sur les paramètres de modèle et les applications logiques, consultez [Vue d’ensemble : Automatiser le déploiement d’applications logiques](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#template-parameters).

<a name="refresh"></a>

## <a name="refresh-from-azure"></a>Actualiser à partir d’Azure

Si vous modifiez votre application logique dans le portail Azure et souhaitez conserver ces modifications, veillez à actualiser cette version de l’application dans Visual Studio.

* Dans Visual Studio, accédez à la barre d'outils du Concepteur d'applications logiques et sélectionnez **Actualiser**.

  -ou-

* Dans Visual Studio Cloud Explorer, ouvrez le menu contextuel de l’application logique et sélectionnez **Actualiser**.

![Actualiser l’application logique avec des mises à jour](./media/manage-logic-apps-with-visual-studio/refresh-logic-app-with-updates-from-portal.png)

## <a name="publish-logic-app-updates"></a>Publier des mises à jour d’application logique

Lorsque vous êtes prêt à déployer vos mises à jour d'application logique de Visual Studio vers Azure, accédez à la barre d'outils du Concepteur d'applications logiques et sélectionnez **Publier**.

![Publier une application logique mise à jour sur le Portail Azure](./media/manage-logic-apps-with-visual-studio/publish-logic-app-to-azure-portal.png)

## <a name="manually-run-your-logic-app"></a>Exécuter manuellement votre application logique

Vous pouvez déclencher manuellement une application logique déployée dans Azure à partir de Visual Studio. Sur la barre d'outils du Concepteur d'applications logiques, sélectionnez **Déclencheur d'exécution**.

![Exécuter manuellement le déclencheur pour votre application logique](./media/manage-logic-apps-with-visual-studio/manually-run-logic-app.png)

## <a name="review-run-history"></a>Examiner l’historique des exécutions

Pour vérifier l’état des exécutions d’application logique et résoudre les problèmes qui y sont liés, vous pouvez consulter les détails, comme les entrées et sorties, de ces exécutions dans Visual Studio.

1. Dans Cloud Explorer, ouvrez le menu contextuel de l’application logique et sélectionnez **Ouvrir l’historique des exécutions**.

   ![Ouvrir l’historique des exécutions de votre application logique](./media/manage-logic-apps-with-visual-studio/open-run-history-for-logic-app.png)

1. Pour afficher les détails d’une exécution spécifique, double-cliquez dessus. Par exemple :

   ![Affichage des informations relatives à une exécution spécifique](./media/manage-logic-apps-with-visual-studio/view-run-history-details.png)
  
   > [!TIP]
   > Pour trier la table par propriété, sélectionnez l'en-tête de colonne de cette propriété.

1. Développez les étapes dont vous souhaitez examiner les entrées et sorties, par exemple :

   ![Afficher les entrées et sorties de chaque étape](./media/manage-logic-apps-with-visual-studio/view-run-history-inputs-outputs.png)

## <a name="disable-or-enable-logic-app"></a>Activer ou désactiver une application logique

Sans supprimer votre application logique, vous pouvez arrêter l’exécution du déclencheur dès que la prochaine condition de déclenchement est remplie. La désactivation de votre application logique empêche le moteur de Logic Apps de créer et d’exécuter les futures instances de workflow de votre application logique. Dans Cloud Explorer, ouvrez le menu contextuel de votre application logique et sélectionnez **Désactiver**.

![Désactiver votre application logique dans Cloud Explorer](./media/manage-logic-apps-with-visual-studio/disable-logic-app-cloud-explorer.png)

> [!NOTE]
> Lorsque vous désactivez une application logique, aucune nouvelle exécution n’est instanciée. Toutes les exécutions en cours et en attente continuent jusqu’à ce qu’elles soient terminées, ce qui peut prendre du temps.

Pour réactiver votre application logique, dans Cloud Explorer, ouvrez le menu contextuel de celle-ci et sélectionnez **Activer**.

![Activer votre application logique dans Cloud Explorer](./media/manage-logic-apps-with-visual-studio/enable-logic-app-cloud-explorer.png)

## <a name="delete-your-logic-app"></a>Supprimer votre application logique

Pour supprimer votre application logique à partir du portail Azure, dans Cloud Explorer, ouvrez le menu contextuel de votre application logique, puis sélectionnez **Supprimer**.

![Supprimer votre application logique de Portail Azure](./media/manage-logic-apps-with-visual-studio/delete-logic-app-from-azure-portal.png)

> [!NOTE]
> Lorsque vous supprimez une application logique, aucune nouvelle exécution n’est instanciée. Toutes les exécutions en cours et en attente sont annulées. Si vous avez des milliers d’exécutions, l’annulation peut prendre beaucoup de temps.

> [!NOTE]
> Si vous supprimez et recréez une application logique enfant, vous devez réenregistrer l’application logique parente. L’application enfant recréée aura des métadonnées différentes.
> Si vous ne réenregistrez pas l’application logique parente après avoir recréé son enfant, vos appels à l’application logique enfant échouent et une erreur de type « non autorisé » s’affiche. Ce comportement s’applique aux applications logiques parent-enfant, par exemple celles qui utilisent des artefacts dans les comptes d’intégration ou qui appellent des fonctions Azure.

## <a name="troubleshooting"></a>Dépannage

Lorsque vous ouvrez votre projet d’application logique dans le Concepteur d’applications logiques, il se peut que vous n’ayez pas la possibilité de sélectionner votre abonnement Azure. Au lieu de cela, votre application logique s’ouvre avec un abonnement Azure qui n’est pas celui que vous souhaitez utiliser. Ce comportement se produit car, après que vous ouvrez le fichier.json d’une application logique, Visual Studio met en cache le premier abonnement sélectionné pour une utilisation ultérieure. Pour résoudre ce problème, essayez l’une des opérations suivantes :

* Renommez le fichier .json de l’application logique. Le cache de l’abonnement varie selon le nom de fichier.

* Pour supprimer les abonnements précédemment sélectionnés pour *toutes* les applications logiques dans votre solution, supprimez le dossier de paramètres Visual Studio masqué (.vs) dans le répertoire de votre solution. Cet emplacement stocke vos informations d’abonnement.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris comment gérer des applications logiques déployées avec Visual Studio. À présent, découvrez-en plus sur la personnalisation des définitions d’application logique pour le déploiement :

> [!div class="nextstepaction"]
> [Créer des définitions d’application logique en JSON](../logic-apps/logic-apps-author-definitions.md)
