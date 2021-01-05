---
title: Démarrage rapide – Déployer un conteneur Docker dans une instance de conteneur – Docker CLI
description: Dans ce démarrage rapide, vous utilisez Docker CLI pour déployer rapidement une application web conteneurisée qui s’exécute dans une instance de conteneur Azure isolée.
ms.topic: quickstart
ms.date: 09/14/2020
ms.custom: devx-track-js
ms.openlocfilehash: 9904fb8c2142816196a1939a16445318bdb245d0
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91262312"
---
# <a name="quickstart-deploy-a-container-instance-in-azure-using-the-docker-cli"></a>Démarrage rapide : Déployer une instance de conteneur dans Azure à l’aide de Docker CLI

Utilisez Azure Container Instances pour exécuter, facilement et rapidement, des conteneurs Docker serverless dans Azure. Déployez sur une instance de conteneur à la demande lorsque vous développez des applications cloud natives et que vous souhaitez passer de façon fluide d’un développement local à un déploiement cloud.

Dans ce démarrage rapide, vous utilisez des commandes Docker CLI natives pour déployer un conteneur Docker et rendre son application disponible dans Azure Container Instances. Cette fonctionnalité est activée par l’[intégration entre Docker et Azure](https://docs.docker.com/engine/context/aci-integration/). Quelques secondes après l’exécution d’une commande `docker run`, vous pouvez accéder à l’application en cours d’exécution dans le conteneur :

:::image type="content" source="media/quickstart-docker-cli/view-application-running-in-an-azure-container-instance.png" alt-text="L’application déployée à l’aide d’Azure Container Instances est affichée dans le navigateur":::

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][azure-account] avant de commencer.

Pour ce démarrage rapide, la version 2.3.0.5 ou ultérieure de Desktop doit être installée, disponible pour [Windows](https://desktop.docker.com/win/edge/Docker%20Desktop%20Installer.exe) ou [macOS](https://desktop.docker.com/mac/edge/Docker.dmg). Vous pouvez également installer [Docker ACI Integration CLI pour Linux](https://docs.docker.com/engine/context/aci-integration/#install-the-docker-aci-integration-cli-on-linux). 

> [!IMPORTANT]
> Toutes les fonctionnalités d’Azure Container Instances ne sont pas prises en charge. Fournissez des commentaires sur l’intégration Docker-Azure en créant un problème dans le dépôt GitHub [aci-integration-beta](https://github.com/docker/aci-integration-beta).

[!INCLUDE [container-instances-create-docker-context](../../includes/container-instances-create-docker-context.md)]

## <a name="create-a-container"></a>Créez un conteneur.

Après avoir créé un contexte Docker, vous pouvez créer un conteneur dans Azure. Dans ce démarrage rapide, vous utilisez l’image `mcr.microsoft.com/azuredocs/aci-helloworld` publique. Cette image contient une petite application web écrite en Node.js qui sert une page HTML statique.

Tout d’abord, accédez au contexte ACI. Toutes les commandes Docker ultérieures s’exécutent dans ce contexte.

```
docker context use myacicontext
```

Exécutez la commande `docker run` suivante pour créer l’instance de conteneur Azure avec le port 80 exposé sur Internet :

```
docker run -p 80:80 mcr.microsoft.com/azuredocs/aci-helloworld
```

Exemple de sortie pour un déploiement réussi :

```
[+] Running 2/2
 ⠿ hungry-kirch            Created                                                                               5.1s
 ⠿ single--container--aci  Done                                                                                 11.3s
hungry-kirch
```

Exécutez `docker ps` pour obtenir des détails sur le conteneur en cours d’exécution, notamment l’adresse IP publique :

```
docker ps
```


L’exemple de sortie montre une adresse IP publique, dans le cas présent *52.230.225.232* :

```
CONTAINER ID        IMAGE                                        COMMAND             STATUS              PORTS
hungry-kirch        mcr.microsoft.com/azuredocs/aci-helloworld                       Running             52.230.225.232:80->80/tcp
```

 À présent, accédez à l’adresse IP dans votre navigateur. Si vous voyez une page web similaire à l’image suivante, félicitations ! Cela signifie que vous avez réussi à déployer une application s’exécutant dans un conteneur Docker sur Azure.

:::image type="content" source="media/quickstart-docker-cli/view-application-running-in-an-azure-container-instance.png" alt-text="L’application déployée à l’aide d’Azure Container Instances est affichée dans le navigateur":::

## <a name="pull-the-container-logs"></a>Extraire les journaux d’activité de conteneur

Lorsque vous avez besoin de résoudre les problèmes relatifs à un conteneur ou à l’application qu’il exécute (ou simplement afficher son résultat), commencez par consulter les journaux d’activité de l’instance de conteneur.

Par exemple, exécutez la commande `docker logs` pour afficher les journaux du conteneur *hungry-kirch* dans le contexte ACI :

```azurecli-interactive
docker logs hungry-kirch
```

La sortie affiche les journaux d’activité du conteneur et, normalement, les requêtes HTTP GET générées lorsque vous avez affiché l’application dans votre navigateur.

```output
listening on port 80
::ffff:10.240.255.55 - - [07/Jul/2020:17:43:53 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [07/Jul/2020:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [07/Jul/2020:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
```


## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous avez terminé d’utiliser le conteneur, exécutez `docker rm` pour le supprimer. Cette commande arrête et supprime l’instance de conteneur Azure.

```
docker rm hungry-kirch
```


## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez créé une instance de conteneur Azure à partir d’une image publique en utilisant l’intégration entre Docker et Azure. Pour plus d’informations sur les scénarios d’intégration, consultez la [documentation Docker](https://docs.docker.com/engine/context/aci-integration/). 

Vous pouvez également utiliser l’[extension Docker pour Visual Studio Code](https://aka.ms/VSCodeDocker) afin de bénéficier d’une expérience intégrée de développement, d’exécution et de gestion des conteneurs, des images et des contextes.

Pour utiliser les outils Azure afin de créer et de gérer des instances de conteneur, consultez les autres démarrages rapides utilisant [Azure CLI](container-instances-quickstart.md), [Azure PowerShell](container-instances-quickstart-powershell.md), le [portail Azure](container-instances-quickstart-portal.md) et le [modèle Azure Resource Manager](container-instances-quickstart-template.md).

Si vous souhaitez utiliser Docker Compose pour définir et exécuter une application multiconteneur localement, puis basculer vers Azure Container Instances, passez au didacticiel.

> [!div class="nextstepaction"]
> [Didacticiel Docker Compose](./tutorial-docker-compose.md)

<!-- LINKS - External -->

[azure-account]: https://azure.microsoft.com/free/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

