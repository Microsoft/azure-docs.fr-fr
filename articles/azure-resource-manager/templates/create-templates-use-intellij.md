---
title: Déployer un modèle - IntelliJ IDEA
description: Découvrez comment créer votre premier modèle Azure Resource Manager (modèle ARM) avec IntelliJ IDEA et comment le déployer.
ms.devlang: java
ms.date: 08/01/2019
ms.topic: conceptual
ms.custom: devx-track-java
ms.openlocfilehash: f0e007fe8930b30cee1c95159d7e964e3792449f
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905959"
---
# <a name="create-and-deploy-arm-templates-by-using-the-intellij-idea"></a>Créer et déployer des modèles Resource Manager à l’aide d’IntelliJ IDEA

Découvrez comment déployer un modèle Azure Resource Manager (modèle ARM) sur Azure avec IntelliJ IDEA, comment le modifier et comment le mettre à jour directement dans l’environnement de développement intégré (IDE). Les modèles Resource Manager sont des fichiers JSON qui définissent les ressources nécessaires pour déployer votre solution. Pour comprendre les concepts associés au déploiement et à la gestion de vos solutions Azure, consultez [Vue d’ensemble du déploiement de modèles](overview.md).

![Schéma du portail de modèle Resource Manager](./media/quickstart-create-templates-use-the-portal/azure-resource-manager-export-deploy-template-portal.png)

Après avoir suivi le tutoriel, vous déployez un compte de Stockage Azure. Le même processus peut être utilisé pour déployer d’autres ressources Azure.

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

* [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) Ultimate Edition ou Community Edition
* Une fois que [Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053) est installé, consultez le [guide de gestion des plug-ins IntelliJ](https://www.jetbrains.com/help/idea/managing-plugins.html) pour plus d’informations.
* [Être](/azure/developer/java/toolkit-for-intellij/sign-in-instructions) connecté à votre compte Azure pour Azure Toolkit for IntelliJ

## <a name="deploy-a-quickstart-template"></a>Déployer un modèle de démarrage rapide

Au lieu de créer un modèle à partir de zéro, ouvrez un modèle à partir de [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/). Le référentiel Modèles de démarrage rapide Azure contient les modèles ARM. Le modèle utilisé dans cet article se nomme [Créer un compte de stockage standard](https://github.com/Azure/azure-quickstart-templates/tree/master/101-storage-account-create/). Il définit une ressource de compte de stockage Azure.

1. Cliquez avec le bouton droit et enregistrez [`azuredeploy.json`](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json) et [`azuredeploy.parameters.json`](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.parameters.json) sur votre ordinateur local.

1. Si Azure Toolkit est correctement installé et connecté, vous devriez voir Azure Explorer dans la barre latérale d’IntelliJ IDEA. Cliquez avec le bouton droit sur **Gestion des ressources** , puis sélectionnez **Créer un déploiement**.

    ![Modèle Resource Manager : cliquer avec le bouton droit pour créer un déploiement](./media/create-templates-use-intellij/resource-manager-create-deployment-right-click.png)

1. Configurez les éléments **Nom du déploiement**, **Abonnement**, **Groupe de ressources** et **Région**. Dans le cas présent, nous déployons le modèle dans un nouveau groupe de ressources `testRG`. Ensuite, sous **Modèle de ressource**, sélectionnez le chemin vers `azuredeploy.json` et, sous **Paramètres de ressource**, le chemin vers le fichier `azuredeploy.parameters.json` que vous avez téléchargé.

    ![Modèle Resource Manager : sélectionner les fichiers pour créer un déploiement](./media/create-templates-use-intellij/resource-manager-create-deployment-select-files.png)

1. Une fois que vous avez cliqué sur OK, le déploiement démarre. La progression est indiquée en bas, dans la **barre d’état** d’IntelliJ IDEA, jusqu’à l’achèvement du déploiement.

    ![Modèle Resource Manager : état du déploiement](./media/create-templates-use-intellij/resource-manager-create-deployment-status.png)

## <a name="browse-an-existing-deployment"></a>Parcourir un déploiement existant

1. Une fois le déploiement terminé, vous pouvez voir le nouveau groupe de ressources `testRG` et un nouveau déploiement créé. Cliquez avec le bouton droit sur le déploiement pour voir la liste des actions possibles. À présent, sélectionnez **Afficher les propriétés**.

    ![Modèle Resource Manager : parcourir un déploiement](./media/create-templates-use-intellij/resource-manager-deployment-browse.png)

1. Un onglet s’ouvre et présente des propriétés utiles telles que l’état du déploiement et la structure du modèle.

    ![Modèle Resource Manager : afficher les propriétés du déploiement](./media/create-templates-use-intellij/resource-manager-deployment-show-properties.png)

## <a name="edit-and-update-an-existing-deployment"></a>Modifier et mettre à jour un déploiement existant

1. Sélectionnez **Modifier le déploiement** dans le menu contextuel ou la vue Afficher les propriétés ouverte précédemment. Un autre onglet s’ouvre, présentant les fichiers de modèles et de paramètres pour le déploiement sur Azure. Pour enregistrer ces fichiers sur l’ordinateur local, vous pouvez cliquer sur **Exporter le fichier de modèle** ou **Exporter le fichier de paramètres**.

    ![Modèle Resource Manager : modifier le déploiement](./media/create-templates-use-intellij/resource-manager-edit-deployment.png)

1. Vous pouvez modifier les deux fichiers sur cette page et déployer les modifications dans Azure. Dans le cas présent, nous modifions la valeur de **storageAccountType** dans les fichiers de paramètres en remplaçant `Standard_LRS` par `Standard_GRS`. Ensuite, cliquez sur **Mettre à jour un déploiement** en bas de l’écran, puis confirmez la mise à jour.

    ![La capture d’écran montre le modèle Resource Manager avec l’invite de mise à jour du déploiement affichée.](./media/create-templates-use-intellij/resource-manager-edit-deployment-update.png)

1. Une fois le déploiement mis à jour, vous pouvez accéder au portail pour vérifier que le compte de stockage créé est remplacé par `Standard_GRS`.

## <a name="clean-up-resources"></a>Nettoyer les ressources

1. Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources. Vous pouvez le faire par le biais du portail Azure ou de l’interface de ligne de commande Azure. Dans Azure Explorer, à partir d’IntelliJ IDEA, cliquez avec le bouton droit sur le **groupe de ressources** que vous avez créé, puis sélectionnez Supprimer.

    ![Supprimer un groupe de ressources dans Azure Explorer à partir d’IntelliJ IDEA](./media/create-templates-use-intellij/delete-resource-group.png)

> [!NOTE]
> Notez que la suppression d’un déploiement ne supprime pas les ressources créées par le déploiement. Supprimez le groupe de ressources correspondant ou des ressources spécifiques quand vous n’en avez plus besoin.

## <a name="next-steps"></a>Étapes suivantes

Le principal objectif de cet article est d’utiliser IntelliJ IDEA pour déployer un modèle existant à partir de modèles de démarrage rapide Azure. Vous avez également appris à consulter et à mettre à jour un déploiement existant sur Azure. Il est possible que les modèles tirés des modèles de démarrage ne vous apportent pas tout ce dont vous avez besoin. Pour en savoir plus sur le développement de modèles, consultez notre nouvelle série de tutoriels pour les débutants :

> [!div class="nextstepaction"]
> [Tutoriels pour les débutants](./template-tutorial-create-first-template.md)

> [!div class="nextstepaction"]
> [Visiter le centre de développement Java sur Azure](/azure/java)
