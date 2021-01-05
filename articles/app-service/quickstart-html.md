---
title: 'Démarrage rapide : Créer une application web HTML statique'
description: Déployez votre premier programme Hello World HTML dans Azure App Service en quelques minutes. Vous effectuez le déploiement avec Git, qui figure parmi les nombreuses façons de déployer vers App Service.
author: msangapu-msft
ms.assetid: 60495cc5-6963-4bf0-8174-52786d226c26
ms.topic: quickstart
ms.date: 08/23/2019
ms.author: msangapu
ms.custom: mvc, cli-validate, seodec18, devx-track-azurecli
ms.openlocfilehash: ec3929219a6297bd4a3d8d3e0eff93d2ed847ba0
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835218"
---
# <a name="create-a-static-html-web-app-in-azure"></a>Créer une application web HTML statique dans Azure

[Azure App Service](overview.md) offre un service d’hébergement web hautement évolutif appliquant des mises à jour correctives automatiques. Ce guide de démarrage rapide montre comment déployer un site HTML+CSS de base sur Azure App Service. Vous allez effectuer ce démarrage rapide dans [Cloud Shell](../cloud-shell/overview.md), mais vous pouvez également exécuter ces commandes localement avec [Azure CLI](/cli/azure/install-azure-cli).

![Page d’accueil de l’exemple d’application](media/quickstart-html/hello-world-in-browser-az.png)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="download-the-sample"></a>Télécharger l’exemple

Dans Cloud Shell, créez un répertoire de démarrage rapide, puis utilisez-le.

```bash
mkdir quickstart

cd $HOME/quickstart
```

Exécutez ensuite la commande suivante pour cloner le référentiel de l’exemple d’application sur votre répertoire de démarrage rapide.

```bash
git clone https://github.com/Azure-Samples/html-docs-hello-world.git
```

## <a name="create-a-web-app"></a>Créer une application web

Passez au répertoire qui contient l’exemple de code et exécutez la commande `az webapp up`. Dans l’exemple suivant, remplacez <nom de l’application> par un nom d’application unique. Le contenu statique est marqué de l’indicateur `--html`.

```bash
cd html-docs-hello-world

az webapp up --location westeurope --name <app_name> --html
```

La commande `az webapp up` exécute les actions suivantes :

- Créer un groupe de ressources par défaut

- Créer un plan App Service par défaut

- Créer une application avec le nom spécifié

- [Décompressez](./deploy-zip.md) les fichiers depuis le répertoire de travail en cours sur l’application web.

L’exécution de cette commande peut prendre quelques minutes. Pendant son exécution, des informations semblables à ce qui suit s’affichent :

<pre>
{
  "app_url": "https://&lt;app_name&gt;.azurewebsites.net",
  "location": "westeurope",
  "name": "&lt;app_name&gt;",
  "os": "Windows",
  "resourcegroup": "appsvc_rg_Windows_westeurope",
  "serverfarm": "appsvc_asp_Windows_westeurope",
  "sku": "FREE",
  "src_path": "/home/&lt;username&gt;/quickstart/html-docs-hello-world ",
  &lt; JSON data removed for brevity. &gt;
}
</pre>

Notez la valeur de `resourceGroup`. Elle vous sera utile pour la section de [nettoyage des ressources](#clean-up-resources).

## <a name="browse-to-the-app"></a>Accéder à l’application

Dans un navigateur, accédez à l’URL de l’application : `http://<app_name>.azurewebsites.net`.

La page s’exécute comme une application web Azure App Service.

![Page d’accueil de l’exemple d’application](media/quickstart-html/hello-world-in-browser-az.png)

**Félicitations !** Vous avez déployé votre première application HTML dans App Service.

## <a name="update-and-redeploy-the-app"></a>Mise à jour et redéploiement de l’application

Dans Cloud Shell, tapez `nano index.html` pour ouvrir l’éditeur de texte nano. Dans la balise de titre `<h1>`, remplacez « Azure App Service - Exemple de site HTML statique » par « Azure App Service », comme indiqué ci-dessous.

![Nano index.html](media/quickstart-html/nano-index-html.png)

Enregistrez vos modifications et quittez nano. Utilisez la commande `^O` pour enregistrer et `^X` pour quitter.

Vous allez maintenant redéployer l’application avec la même commande `az webapp up`.

```bash
az webapp up --location westeurope --name <app_name> --html
```

Une fois le déploiement terminé, revenez à la fenêtre du navigateur que vous avez ouverte à l’étape **Accéder à l’application**, puis actualisez la page.

![Mise à jour de la page d’accueil de l’exemple d’application](media/quickstart-html/hello-azure-in-browser-az.png)

## <a name="manage-your-new-azure-app"></a>Gérer votre nouvelle application Azure

Pour gérer l’application web que vous avez créée, dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **App Services**. 

![Sélectionner App Services dans le portail Azure](./media/quickstart-html/portal0.png)

Dans la page **App Services**, sélectionnez le nom de votre application Azure.

![Navigation au sein du portail pour accéder à l’application Azure](./media/quickstart-html/portal1.png)

Vous voyez apparaître la page Vue d’ensemble de votre application web. Ici, vous pouvez également des tâches de gestion de base (parcourir, arrêter, démarrer, redémarrer et supprimer des éléments, par exemple).

![Panneau App Service sur le portail Azure](./media/quickstart-html/portal2.png)

Le menu de gauche fournit différentes pages vous permettant de configurer votre application.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Au cours des étapes précédentes, vous avez créé des ressources Azure au sein d’un groupe de ressources. Si vous ne pensez pas avoir besoin de ces ressources à l’avenir, supprimez le groupe de ressources en exécutant la commande suivante dans Cloud Shell. Souvenez-vous que le nom du groupe de ressources a été généré automatiquement pour vous à l’étape [créer une application web](#create-a-web-app).

```bash
az group delete --name appsvc_rg_Windows_westeurope
```

L’exécution de cette commande peut prendre une minute.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Mapper un domaine personnalisé](app-service-web-tutorial-custom-domain.md)
