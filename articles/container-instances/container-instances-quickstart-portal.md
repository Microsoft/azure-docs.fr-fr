---
title: Guide de démarrage rapide – Déployer un conteneur Docker dans une instance de conteneur – Portail
description: Dans ce guide de démarrage rapide, vous utilisez le portail Azure pour déployer rapidement une application web conteneurisée qui s’exécute dans une instance de conteneur Azure isolé.
ms.topic: quickstart
ms.date: 08/24/2020
ms.custom: seodec18, mvc, devx-track-js
ms.openlocfilehash: c8477bd91c3a02a2cd02d341c38c16da251902ae
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004804"
---
# <a name="quickstart-deploy-a-container-instance-in-azure-using-the-azure-portal"></a>Démarrage rapide : Déployer un instance de conteneur dans Azure à l’aide du portail Azure

Utilisez Azure Container Instances pour exécuter, facilement et rapidement, des conteneurs Docker serverless dans Azure. Déployez une application sur une instance de conteneur à la demande lorsque vous n’avez pas besoin d’une plateforme d’orchestration de conteneur complète telle qu’Azure Kubernetes Service.

Dans cette procédure de démarrage rapide, vous utilisez le portail Azure pour déployer un conteneur Docker isolé et mettre son application à disposition avec un nom de domaine complet (FQDN). Après avoir configuré quelques paramètres et déployé le conteneur, vous pouvez accéder à l’application en cours d’exécution :

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-07.png" alt-text="L’application déployée à l’aide d’Azure Container Instances est affichée dans le navigateur":::

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][azure-free-account] avant de commencer.

## <a name="create-a-container-instance"></a>Créer une instance de conteneur

Sélectionnez **Créer une ressource** > **Conteneurs** > **Container Instances**.

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-01.png" alt-text="Commencer à créer une instance de conteneur dans le portail Azure":::

Dans la page **De base**, entrez les valeurs suivantes dans les zones de texte **Groupe de ressources**, **Nom du conteneur** et **Image du conteneur**. Conservez les autres valeurs par défaut, puis cliquez sur **OK**.

* Groupe de ressources : **Créer** > `myresourcegroup`
* Nom du conteneur : `mycontainer`
* Source d’image : **Images du guide de démarrage rapide**
* Image conteneur : `mcr.microsoft.com/azuredocs/aci-helloworld` (Linux)

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-03.png" alt-text="Configuration des paramètres de base pour une nouvelle instance de conteneur dans le portail Azure":::

Pour ce guide de démarrage rapide, vous conservez les paramètres par défaut pour déployer l’image Microsoft publique `aci-helloworld`. Cet exemple d’image Linux contient une petite application web écrite en Node.js qui se présente sous la forme d’une page HTML statique. Vous pouvez également utiliser vos propres images conteneur stockées dans Azure Container Registry, Docker Hub ou d’autres registres.

Dans la page **Mise en réseau**, spécifiez une **Étiquette du nom DNS** pour votre conteneur. Le nom doit être unique au sein de la région Azure dans laquelle vous créez l’instance de conteneur. Votre conteneur sera publiquement accessible avec `<dns-name-label>.<region>.azurecontainer.io`. Si vous recevez un message d’erreur « Étiquette de nom DNS indisponible », essayez d’utiliser une autre étiquette de nom DNS.

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-04.png" alt-text="Configuration des paramètres réseau pour une nouvelle instance de conteneur dans le portail Azure":::

Conservez les autres paramètres comme valeurs par défaut, puis sélectionnez **Vérifier + créer**.

Une fois la validation terminée, un résumé des paramètres de votre conteneur s’affiche. Sélectionnez **Créer** pour envoyer votre demande de déploiement de conteneur.

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-05.png" alt-text="Résumé des paramètres d’une nouvelle instance de conteneur dans le portail Azure":::

Quand le déploiement commence, une notification s’affiche indiquant que le déploiement est en cours. Une autre notification s’affiche lorsque le groupe de conteneurs a été déployé.

Ouvrez la vue d’ensemble du groupe de conteneurs en accédant à **Groupes de ressources** > **myresourcegroup** > **mycontainer**. Prenez note du **nom de domaine complet** de l’instance de conteneur, ainsi que de son **état**.

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-06.png" alt-text="Vue d’ensemble du groupe de conteneurs dans le portail Azure":::

Une fois que son **État** est *En cours d’exécution*, accédez au nom de domaine complet du conteneur dans votre navigateur.

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-07.png" alt-text="L’application déployée à l’aide d’Azure Container Instances est affichée dans le navigateur":::

Félicitations ! En configurant simplement quelques paramètres, vous avez déployé une application accessible publiquement dans Azure Container Instances.

## <a name="view-container-logs"></a>Afficher les journaux d’activité du conteneur

Il est utile d’afficher les journaux d’activité d’une instance de conteneur pour résoudre des problèmes liés au conteneur ou à l’application qu’il exécute.

Pour afficher les journaux d’activité du conteneur, sous **Paramètres**, sélectionnez **Conteneurs**, puis **Journaux d’activité**. Vous devez normalement voir la requête HTTP GET générée quand vous affichez l’application dans votre navigateur.

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-11.png" alt-text="Journaux d’activité du conteneur dans le portail Azure":::


## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous avez terminé avec le conteneur, sélectionnez **Vue d’ensemble** pour l’instance de conteneur *mycontainer*, puis sélectionnez **Supprimer**.

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-09.png" alt-text="Suppression de l’instance de conteneur dans le portail Azure":::

Cliquez sur **Oui** quand la boîte de dialogue de confirmation apparaît.

:::image type="content" source="media/container-instances-quickstart-portal/qs-portal-10.png" alt-text="Supprimer la confirmation d’une instance de conteneur dans le portail Azure":::

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une instance de conteneur Azure à partir d’une image Microsoft publique. Si vous voulez créer une image conteneur et la déployer à partir d’un registre de conteneurs Azure privé, passez au didacticiel Azure Container Instances.

> [!div class="nextstepaction"]
> [Didacticiel Azure Container Instances](./container-instances-tutorial-prepare-app.md)

<!-- LINKS - External -->
[azure-free-account]: https://azure.microsoft.com/free/