---
title: Configurer des paramètres d’application de fonction dans Azure
description: Apprenez à configurer les paramètres d’application Azure Functions.
ms.assetid: 81eb04f8-9a27-45bb-bf24-9ab6c30d205c
ms.topic: conceptual
ms.date: 04/13/2020
ms.custom: cc996988-fb4f-47, devx-track-azurecli
ms.openlocfilehash: f597e58c70d6ac9daff753f5c0a54199c2383c42
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019501"
---
# <a name="manage-your-function-app"></a>Gérer votre application de fonction 

Dans Azure Functions, une Function App fournit le contexte d’exécution de vos fonctions individuelles. Les comportements de la Function App s’appliquent à toutes les fonctions hébergées par une Function App donnée. Toutes les fonctions d’une application de fonction doivent être exprimées dans le même [langage](supported-languages.md). 

Les fonctions individuelles dans une application de fonction sont déployées ensemble et sont mises à l’échelle ensemble. Toutes les fonctions de la même application de fonction partagent les ressources, par instance, à mesure que l’application de fonction est mise à l’échelle. 

Les chaînes de connexion, les variables d’environnement et d’autres paramètres d’application sont définis séparément pour chaque application de fonction. Toutes les données qui doivent être partagées entre les applications de fonction doivent être stockées en externe dans un magasin persistant.

Cet article explique comment configurer et gérer vos applications de fonction. 

> [!TIP]  
> De nombreuses options de configuration peuvent également être gérées à l’aide d’[Azure CLI]. 

## <a name="get-started-in-the-azure-portal"></a>Bien démarrer dans le portail Azure

1. Commencez par accéder au [Azure portal] et connectez-vous à votre compte Azure. Dans la barre de recherche en haut du portail, tapez le nom de votre application de fonction et sélectionnez-la dans la liste. 

2. Sous **Paramètres** dans le volet gauche, sélectionnez **Configuration**.

    :::image type="content" source="./media/functions-how-to-use-azure-function-app-settings/azure-function-app-main.png" alt-text="Vue d’ensemble de l’application de fonction dans le Portail Azure":::

Vous pouvez accéder à tout ce dont vous avez besoin pour gérer votre application de fonction à partir de la page Vue d’ensemble, en particulier les **[paramètres de l’application](#settings)** et les **[fonctionnalités de la plateforme](#platform-features)** .

## <a name="application-settings"></a><a name="settings"></a>Paramètres de l’application

L’onglet **Paramètres de l’application** conserve les paramètres qui sont utilisés par votre application de fonction. Ces paramètres sont chiffrés et stockés, et vous devez sélectionner **Afficher les valeurs** pour afficher les valeurs dans le portail. Vous pouvez également accéder aux paramètres d’application à l’aide d’Azure CLI.

### <a name="portal"></a>Portail

Pour ajouter un paramètre dans le portail, sélectionnez **Nouveau paramètre d’application** et ajoutez la nouvelle paire clé-valeur.

![Paramètres de l’application de fonction sur le portail Azure.](./media/functions-how-to-use-azure-function-app-settings/azure-function-app-settings-tab.png)

### <a name="azure-cli"></a>Azure CLI

La commande [`az functionapp config appsettings list`](/cli/azure/functionapp/config/appsettings#az-functionapp-config-appsettings-list) retourne les paramètres d’application existants, comme dans l’exemple suivant :

```azurecli-interactive
az functionapp config appsettings list --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME>
```

La commande [`az functionapp config appsettings set`](/cli/azure/functionapp/config/appsettings#az-functionapp-config-appsettings-set) ajoute ou met à jour un paramètre d’application. L’exemple suivant crée un paramètre avec une clé nommée `CUSTOM_FUNCTION_APP_SETTING` et la valeur `12345` :


```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings CUSTOM_FUNCTION_APP_SETTING=12345
```

### <a name="use-application-settings"></a>Utiliser des paramètres d’application

[!INCLUDE [functions-environment-variables](../../includes/functions-environment-variables.md)]

Quand vous développez une application de fonction localement, vous devez conserver des copies locales de ces valeurs dans le fichier de projet local.settings.json. Pour plus d’informations, consultez [Fichier de paramètres locaux](functions-run-local.md#local-settings-file).

## <a name="platform-features"></a>Fonctionnalités de la plate-forme

Les applications de fonction s’exécutent dans la plateforme Azure App Service et y sont entretenues. Par conséquent, vos Function Apps ont accès à la plupart des fonctionnalités de la plateforme d’hébergement web principale d’Azure. Dans le volet de gauche, vous pouvez accéder aux nombreuses fonctionnalités de la plateforme App Service que vous pouvez utiliser dans vos applications de fonction. 

> [!NOTE]
> Toutes les fonctionnalités App Service ne sont pas disponibles quand une Function App s’exécute dans le cadre du plan d’hébergement Consommation.

Le reste de ce article se concentre sur les fonctionnalités App Service du portail Azure qui sont utiles pour Functions :

+ [Éditeur App Service](#editor)
+ [Console](#console)
+ [Outils avancés (Kudu)](#kudu)
+ [Options de déploiement](#deployment)
+ [CORS](#cors)
+ [Authentification](#auth)

Pour plus d’informations sur l’utilisation des paramètres App Service, consultez [Configurer des applications web dans Azure App Service](../app-service/configure-common.md).

### <a name="app-service-editor"></a><a name="editor"></a>Éditeur App Service

![L’éditeur App Service](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-appservice-editor.png)

L’éditeur App Service est un éditeur avancé intégré au portail. Vous pouvez l’utiliser pour modifier les fichiers de configuration JSON et les fichiers de code. L’activation de cette option entraîne l’ouverture d’un onglet distinct du navigateur avec un éditeur de base. Vous pouvez ainsi l’intégrer au référentiel GitHub, exécuter et déboguer du code et modifier les paramètres de Function App. Cet éditeur fournit un environnement de développement amélioré pour vos fonctions en comparaison avec l’éditeur de fonction intégré.  

Nous vous recommandons d’envisager de développer vos fonctions sur votre ordinateur local. Quand vous développez localement et publiez sur Azure, vos fichiers projet sont en lecture seule dans le portail. Pour en savoir plus, consultez [Coder et tester Azure Functions localement](functions-develop-local.md).

### <a name="console"></a><a name="console"></a>Console

![Console Function App](./media/functions-how-to-use-azure-function-app-settings/configure-function-console.png)

La console intégrée au portail est un outil de développement idéal lorsque vous souhaitez interagir avec Function App à partir de la ligne de commande. Les commandes courantes incluent la création de fichiers et de répertoires et la navigation, ainsi que l’exécution de scripts et de fichiers de commandes. 

Quand vous développez localement, nous vous recommandons d’utiliser [Azure Functions Core Tools](functions-run-local.md) et [Azure CLI].

### <a name="advanced-tools-kudu"></a><a name="kudu"></a>Outils avancés (Kudu)

![Configurer Kudu](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-kudu.png)

Les outils avancés pour App Service (également appelé Kudu) donnent accès aux fonctionnalités d’administration avancées de votre Function App. Dans Kudu, vous pouvez gérer les informations système, les paramètres d’application, les variables d’environnement, les extensions de site, les en-têtes HTTP et les variables de serveur. Vous pouvez également lancer **Kudu** en naviguant vers le point de terminaison SCM pour votre Function App, comme `https://<myfunctionapp>.scm.azurewebsites.net/`. 


### <a name="deployment-center"></a><a name="deployment"></a>Centre de déploiement

Quand vous utilisez une solution de contrôle de code source pour développer et gérer le code de vos fonctions, le centre de déploiement vous permet de générer et de déployer à partir du contrôle de code source. Votre projet est généré et déployé sur Azure quand vous effectuez des mises à jour. Pour plus d’informations, consultez [Technologies de déploiement dans Azure Functions](functions-deployment-technologies.md).

### <a name="cross-origin-resource-sharing"></a><a name="cors"></a>Partage des ressources cross-origin

Pour empêcher l’exécution de code malveillant sur le client, les navigateurs modernes bloquent les requêtes des applications web vers les ressources qui s’exécutent dans un domaine distinct. Le [partage des ressources cross-origin (CORS)](https://developer.mozilla.org/docs/Web/HTTP/CORS) permet à un en-tête `Access-Control-Allow-Origin` de déclarer les origines autorisées à appeler des points de terminaison sur votre application de fonction.

#### <a name="portal"></a>Portail

Quand vous configurez la liste **Origines autorisées** pour votre application de fonction, l’en-tête `Access-Control-Allow-Origin` est automatiquement ajouté à toutes les réponses des points de terminaison HTTP dans votre application de fonction. 

![Configurer la liste CORS de l’application de fonction](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-cors.png)

Quand le caractère générique (`*`) est utilisé, tous les autres domaines sont ignorés. 

Utilisez la commande [`az functionapp cors add`](/cli/azure/functionapp/cors#az-functionapp-cors-add) pour ajouter un domaine à la liste des origines autorisées. L’exemple suivant ajoute le domaine contoso.com :

```azurecli-interactive
az functionapp cors add --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--allowed-origins https://contoso.com
```

Utilisez la commande [`az functionapp cors show`](/cli/azure/functionapp/cors#az-functionapp-cors-show) pour lister les origines autorisées actuelles.

### <a name="authentication"></a><a name="auth"></a>Authentification

![Configurer l’authentification pour une Function App](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-authentication.png)

Lorsque les fonctions utilisent un déclencheur HTTP, vous pouvez exiger l’authentification préalable des appels. App Service prend en charge l’authentification Azure Active Directory et la connexion avec des fournisseurs sociaux tels que Facebook, Microsoft et Twitter. Pour plus d’informations sur la configuration de fournisseurs d’authentification spécifiques, consultez [Azure App Service authentication overview](../app-service/overview-authentication-authorization.md) (Vue d’ensemble de l’authentification Azure App Service). 


## <a name="next-steps"></a>Étapes suivantes

+ [Configurer des applications web dans Azure App Service](../app-service/configure-common.md)
+ [Déploiement continu pour Azure Functions](functions-continuous-deployment.md)

[Azure CLI]: /cli/azure/
[Azure portal]: https://portal.azure.com
