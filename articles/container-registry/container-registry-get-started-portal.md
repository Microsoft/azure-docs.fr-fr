---
title: Guide de démarrage rapide - Créer un registre dans le portail
description: Apprenez rapidement à créer un registre de conteneurs Azure privé à l’aide du portail Azure.
ms.topic: quickstart
ms.date: 08/04/2020
ms.custom: seodec18, mvc
ms.openlocfilehash: ff74232edf5c96f091082f8486c44fed6125b0b1
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825846"
---
# <a name="quickstart-create-an-azure-container-registry-using-the-azure-portal"></a>Démarrage rapide : Créer un registre de conteneurs Azure à l’aide du portail Azure

Un registre de conteneurs Azure est un registre Docker privé dans Azure, où vous pouvez stocker et gérer des images de conteneurs Docker privés ainsi que les artefacts associés. Dans ce guide de démarrage rapide, vous allez créer un registre de conteneurs à partir du portail Azure. Vous allez ensuite utiliser des commandes Docker pour envoyer (push) une image conteneur dans le registre, puis tirer (pull) et exécuter l’image à partir de votre registre.

Pour vous connecter au registre et utiliser des images conteneur, ce guide de démarrage rapide vous demande d’exécuter Azure CLI (version 2.0.55 ou ultérieure recommandée). Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli].

Docker doit également être installé en local. Docker fournit des packages qui le configurent facilement sur n’importe quel système [Mac][docker-mac], [Windows][docker-windows] ou [Linux][docker-linux].

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-a-container-registry"></a>Créer un registre de conteneur

Sélectionnez **Créer une ressource** > **Conteneurs** > **Container Registry**.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-01.png" alt-text="Accéder au registre de conteneurs dans le portail":::

Sous l’onglet **Informations de base**, entrez les valeurs appropriées pour **Groupe de ressources** et **Nom du registre**. Le nom du registre doit être unique dans Azure et contenir entre 5 et 50 caractères alphanumériques. Pour ce guide de démarrage rapide, créez un groupe de ressources dans l’emplacement `West US` nommé `myResourceGroup`, et sélectionnez « De base » pour **SKU**.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-03.png" alt-text="Créer un registre de conteneurs dans le portail":::

Acceptez les valeurs par défaut pour les autres paramètres. Sélectionnez ensuite **Passer en revue + créer** . Après avoir passé en revue les paramètres, sélectionnez **Créer**.

Dans ce guide de démarrage rapide, vous allez créer un registre *De base*. Il s’agit d’une option à coût optimisé pour les développeurs qui apprennent à se servir d’Azure Container Registry. Pour plus d’informations sur les niveaux de service disponibles (SKU), consultez [Niveaux de service des registres de conteneurs][container-registry-skus].

Quand le message **Déploiement réussi** s’affiche, sélectionnez le registre de conteneurs dans le portail. 

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-05.png" alt-text="Vue d’ensemble du registre de conteneurs dans le portail":::

Prenez note du nom du registre et de la valeur du **serveur de connexion**. Vous utilisez ces valeurs dans les étapes suivantes quand vous effectuez l’envoi (push) et le tirage (pull) d’images avec Docker.

## <a name="log-in-to-registry"></a>Se connecter au registre

Avant d’envoyer (push) et de tirer (pull) des images conteneur, vous devez vous connecter à l’instance du registre. [Connectez-vous à Azure CLI][get-started-with-azure-cli] sur votre machine locale, puis exécutez la commande [az acr login][az-acr-login]. Spécifiez uniquement le nom du registre au moment de la connexion à Azure CLI. N’utilisez pas le nom du serveur de connexion, qui comprend un suffixe de domaine comme `azurecr.io`.

```azurecli
az acr login --name <registry-name>
```

Exemple :

```azurecli
az acr login --name mycontainerregistry
```

Une fois l’opération terminée, la commande renvoie `Login Succeeded`. 

[!INCLUDE [container-registry-quickstart-docker-push](../../includes/container-registry-quickstart-docker-push.md)]

## <a name="list-container-images"></a>Répertorier les images conteneur

Pour lister les images de votre registre, accédez à ce dernier dans le portail, puis sélectionnez **Référentiels** et le référentiel **hello-world** que vous avez créé avec `docker push`.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-09.png" alt-text="Lister les images conteneur dans le portail":::

En sélectionnant le référentiel **hello-world**, vous voyez l’image étiquetée `v1` sous **Étiquettes**.

[!INCLUDE [container-registry-quickstart-docker-pull](../../includes/container-registry-quickstart-docker-pull.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer vos ressources, accédez au groupe de ressources **myResourceGroup** dans le portail. Une fois le groupe de ressources chargé, cliquez sur **Supprimer le groupe de ressources** pour supprimer le groupe de ressources, le registre de conteneurs et les images conteneur stockées à cet endroit.

:::image type="content" source="media/container-registry-get-started-portal/qs-portal-08.png" alt-text="Supprimer un groupe de ressources dans le portail":::


## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un registre de conteneurs Azure avec le portail Azure, envoyé (push) une image conteneur, puis tiré (pull) et exécuté l’image à partir du registre. Passez à présent au tutoriel sur Azure Container Registry (ACR) pour approfondir vos connaissances.

> [!div class="nextstepaction"]
> [Tutoriels sur Azure Container Registry][container-registry-tutorial-prepare-registry]

> [!div class="nextstepaction"]
> [Tutoriels Azure Container Registry Tasks][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[container-registry-tutorial-prepare-registry]: container-registry-tutorial-prepare-registry.md
[container-registry-skus]: container-registry-skus.md
[azure-cli]: /cli/azure/install-azure-cli
[get-started-with-azure-cli]: /cli/azure/get-started-with-azure-cli
[az-acr-login]: /cli/azure/acr#az-acr-login
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
