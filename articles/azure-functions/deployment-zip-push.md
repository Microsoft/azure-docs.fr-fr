---
title: Déploiement Push de fichier Zip pour Azure Functions
description: Utilisez les fonctionnalités de déploiement de fichier Zip du service de déploiement Kudu pour publier vos Azure Functions.
ms.topic: conceptual
ms.date: 08/12/2018
ms.openlocfilehash: e104661dcdf1f6c6fd6dd5eb1024748980e7931f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018425"
---
# <a name="zip-deployment-for-azure-functions"></a>Déploiement zip pour Azure Functions

Cet article décrit comment déployer vos fichiers projet d’application de fonction sur Azure à partir d’un fichier .zip (compressé). Vous allez découvrir comment effectuer un déploiement Push, à la fois à l’aide d’Azure CLI et des API REST. Les [outils principaux d’Azure Functions](functions-run-local.md) utilisent également ces API de déploiement lors de la publication d’un projet local sur Azure.

Azure Functions offre la gamme complète d’options d’intégration et de déploiement continu fournies par Azure App Service. Pour plus d’informations, consultez [Déploiement continu pour Azure Functions](functions-continuous-deployment.md).

Pour accélérer le développement, vous pouvez déployer les fichiers projet de vos applications de fonction directement à partir d’un fichier zip. L’API de déploiement zip prend le contenu d’un fichier zip et extrait son contenu dans le dossier `wwwroot` de votre application de fonction. Ce déploiement de fichier .zip utilise le même service Kudu que celui qui pilote les déploiements continus basés sur l’intégration, notamment :

+ Suppression des fichiers conservés suite aux déploiements antérieurs
+ Personnalisation du déploiement, notamment exécution de scripts de déploiement
+ Journaux d’activité de déploiement
+ La fonction de synchronisation se déclenche dans une application de fonction [Plan Consommation](functions-scale.md).

Pour plus d’informations, consultez la [documentation de référence sur le déploiement zip](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file).

## <a name="deployment-zip-file-requirements"></a>Spécifications du déploiement de fichier .zip

Le fichier zip que vous utilisez pour le déploiement Push doit contenir tous les fichiers nécessaires à l’exécution de votre fonction.

>[!IMPORTANT]
> Quand vous utilisez le déploiement zip, tous les fichiers d’un déploiement existant qui ne se trouvent pas dans le fichier zip sont supprimés de votre application de fonction.  

[!INCLUDE [functions-folder-structure](../../includes/functions-folder-structure.md)]

Une application de fonction inclut tous les fichiers et dossiers dans le répertoire `wwwroot`. Un déploiement de fichier .zip inclut le contenu du répertoire `wwwroot`, mais pas le répertoire proprement dit. Lorsque vous déployez un projet de bibliothèque de classes C#, vous devez inclure les fichiers et les dépendances de bibliothèque compilés dans un sous-dossier `bin` de votre package zip.

## <a name="download-your-function-app-files"></a>Télécharger vos fichiers d’application de fonction

Quand vous développez sur un ordinateur local, il est facile de créer un fichier .zip du dossier de projet d’application de fonction sur votre ordinateur de développement.

Toutefois, vous aurez peut-être créé vos fonctions à l’aide de l’éditeur dans le portail Azure. Vous pouvez télécharger un projet d’application de fonction existant d’une des manières suivantes :

+ **À partir du portail Azure :**

  1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre application de fonction.

  2. Sous l’onglet **Vue d’ensemble**, sélectionnez **Télécharger le contenu de l’application**. Sélectionnez vos options de téléchargement, puis **Télécharger**.

      ![Télécharger le projet d’application de fonction](./media/deployment-zip-push/download-project.png)

     Le fichier .zip téléchargé est au format correct pour être republié sur votre application de fonction en utilisant le déploiement Push de fichier .zip. En outre, le téléchargement à partir du portail peut ajouter les fichiers nécessaires pour ouvrir votre application de fonction directement dans Visual Studio.

+ **Utilisation d’API REST :**

    Utilisez l’API GET de déploiement suivante pour télécharger les fichiers à partir de votre projet `<function_app>` : 

    ```http
    https://<function_app>.scm.azurewebsites.net/api/zip/site/wwwroot/
    ```

    L’inclusion de `/site/wwwroot/` permet de s’assurer que votre fichier zip comprend uniquement les fichiers de projet de l’application de fonction, et non l’intégralité du site. Si vous ne vous êtes pas encore connecté à Azure, vous serez invité à le faire.  

Vous pouvez également télécharger un fichier .zip à partir d’un dépôt GitHub. Quand vous téléchargez un dépôt GitHub sous la forme d’un fichier zip, GitHub ajoute un niveau de dossier supplémentaire à la branche. Ce niveau de dossier supplémentaire signifie que vous ne pouvez pas déployer le fichier .zip directement tel que vous l’avez téléchargé à partir de GitHub. Si vous utilisez un dépôt GitHub pour tenir à jour votre application de fonction, vous devez utiliser [l’intégration continue](functions-continuous-deployment.md) pour la déployer.  

## <a name="deploy-by-using-azure-cli"></a><a name="cli"></a>Déployer à l’aide d’Azure CLI

Vous pouvez utiliser Azure CLI pour déclencher un déploiement Push. Effectuez le déploiement Push d’un fichier .zip sur votre application de fonction à l’aide de la commande [az functionapp deployment source config-zip](/cli/azure/functionapp/deployment/source#az-functionapp-deployment-source-config-zip). Pour exécuter cette commande, vous devez utiliser Azure CLI version 2.0.21 ou ultérieure. Pour vérifier votre version d’Azure CLI, utilisez la commande `az --version`.

Dans la commande suivante, remplacez l’espace réservé `<zip_file_path>` par le chemin vers l’emplacement de votre fichier .zip. Remplacez également `<app_name>` par le nom unique de votre application de fonction, et `<resource_group>` par le nom de votre groupe de ressources.

```azurecli-interactive
az functionapp deployment source config-zip -g <resource_group> -n \
<app_name> --src <zip_file_path>
```

Cette commande déploie les fichiers projet à partir du fichier .zip téléchargé sur votre application de fonction dans Azure. Ensuite, il redémarre l’application. Pour afficher la liste des déploiements pour cette application de fonction, vous devez utiliser les API REST.

Quand vous utilisez Azure CLI sur votre ordinateur local, `<zip_file_path>` est le chemin du fichier .zip sur votre ordinateur. Vous pouvez également exécuter Azure CLI dans [Azure Cloud Shell](../cloud-shell/overview.md). Quand vous utilisez Cloud Shell, vous devez d’abord charger votre fichier .zip de déploiement sur le compte Azure Files associé à votre Cloud Shell. Dans ce cas, `<zip_file_path>` est l’emplacement de stockage utilisé par votre compte Cloud Shell. Pour plus d’informations, consultez [Conserver des fichiers dans Azure Cloud Shell](../cloud-shell/persisting-shell-storage.md).

[!INCLUDE [app-service-deploy-zip-push-rest](../../includes/app-service-deploy-zip-push-rest.md)]

## <a name="run-functions-from-the-deployment-package"></a>Exécuter des fonctions à partir du package de déploiement

Vous pouvez également choisir d’exécuter vos fonctions directement à partir du fichier de package de déploiement. Cette méthode permet d’éviter l’étape de déploiement qui consiste à copier des fichiers du package vers le répertoire `wwwroot` de votre application de fonction. Au lieu de cela, le fichier de package est monté par le runtime Functions, et le contenu du répertoire `wwwroot` passe en lecture seule.  

Le déploiement zip s’intègre à cette fonctionnalité, que vous pouvez activer en définissant le paramètre d’application de fonction `WEBSITE_RUN_FROM_PACKAGE` sur la valeur `1`. Pour plus d’informations, consultez [Exécuter vos fonctions Azure à partir d’un fichier de package](run-functions-from-deployment-package.md).

[!INCLUDE [app-service-deploy-zip-push-custom](../../includes/app-service-deploy-zip-push-custom.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déploiement continu pour Azure Functions](functions-continuous-deployment.md)

[.zip push deployment reference topic]: https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file
