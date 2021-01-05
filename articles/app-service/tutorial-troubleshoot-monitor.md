---
title: 'Tutoriel : Résolution des problèmes avec Azure Monitor'
description: Découvrez comment Azure Monitor et Log Analytics vous permettent de superviser une application web App Service. Azure Monitor optimise la disponibilité grâce à une solution complète permettant de superviser vos environnements.
author: msangapu-msft
ms.author: msangapu
ms.topic: tutorial
ms.date: 06/20/2020
ms.openlocfilehash: 049adac5521efd68ae8aa77af2d1007f9dfe0c0e
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586990"
---
# <a name="tutorial-troubleshoot-an-app-service-app-with-azure-monitor"></a>Tutoriel : Résoudre les problèmes d’une application App Service avec Azure Monitor

> [!NOTE]
> L’intégration d’Azure Monitor à App Service est proposée en [préversion](https://aka.ms/appsvcblog-azmon).
>

Ce tutoriel montre comment résoudre les problèmes liés à une application [App Service](overview.md) à l'aide d'[Azure Monitor](../azure-monitor/overview.md). L’exemple d’application inclut le code destiné à saturer la mémoire et à générer des erreurs HTTP 500. Il vous permet ainsi de diagnostiquer et de résoudre le problème à l’aide d’Azure Monitor. Quand vous aurez terminé, vous disposerez d’un exemple d’application s’exécutant dans App Service sur Linux intégré à [Azure Monitor](../azure-monitor/overview.md).

[Azure Monitor](../azure-monitor/overview.md) optimise la disponibilité et les performances de vos applications et services en fournissant une solution complète pour collecter, analyser et utiliser la télémétrie de vos environnements cloud et locaux.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer une application web avec Azure Monitor.
> * Envoyer des journaux de console à Log Analytics.
> * Utiliser des requêtes de journal pour identifier et résoudre les erreurs d’une application web.

Vous pouvez suivre les étapes de ce tutoriel sur macOS, Linux, Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous aurez besoin des éléments suivants :

- [Abonnement Azure](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
- [Azure CLI](/cli/azure/install-azure-cli)
- [Git](https://git-scm.com/)

## <a name="create-azure-resources"></a>Créer des ressources Azure

Tout d’abord, vous devez exécuter plusieurs commandes localement pour configurer l’exemple d’application que vous utiliserez avec ce tutoriel. Les commandes clonent un exemple d’application, créent des ressources Azure, créent un utilisateur de déploiement et déploient l’application sur Azure. Le mot de passe que vous avez fourni dans le cadre de la création de l’utilisateur du déploiement vous sera demandé. 

```bash
git clone https://github.com/Azure-Samples/App-Service-Troubleshoot-Azure-Monitor
az group create --name myResourceGroup --location "South Central US"
az webapp deployment user set --user-name <username> --password <password>
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku B1 --is-linux
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "PHP|7.3" --deployment-local-git
git remote add azure <url_from_previous_step>
git push azure main
```

## <a name="configure-azure-monitor-preview"></a>Configurer Azure Monitor (préversion)

### <a name="create-a-log-analytics-workspace"></a>Créer un espace de travail Log Analytics

Vous avez déployé l’exemple d’application dans Azure App Service. À présent, vous allez configurer la fonctionnalité de supervision pour résoudre les problèmes affectant l’application. Azure Monitor stocke les données de journal dans un espace de travail Log Analytics. Un espace de travail est un conteneur qui renferme des données ainsi que des informations de configuration.

Au cours de cette étape, vous allez créer un espace de travail Log Analytics pour configurer Azure Monitor avec votre application.

```bash
az monitor log-analytics workspace create --resource-group myResourceGroup --workspace-name myMonitorWorkspace
```

> [!NOTE]
> [Avec Azure Monitor Log Analytics, vous payez pour l’ingestion et la conservation des données.](https://azure.microsoft.com/pricing/details/monitor/)
>

### <a name="create-a-diagnostic-setting"></a>Créer un paramètre de diagnostic

Les paramètres de diagnostic permettent de collecter des métriques pour certains services Azure dans les journaux Azure Monitor afin de les analyser avec d’autres données de surveillance à l’aide de requêtes de journal. Pour ce tutoriel, vous devez activer les journaux de serveur web et de sorties/d’erreurs standard. Pour obtenir la liste complète des types de journaux avec leurs descriptions, consultez [Types de journaux pris en charge](./troubleshoot-diagnostic-logs.md#supported-log-types).

Exécutez les commandes suivantes pour créer des paramètres de diagnostic pour AppServiceConsoleLogs (sorties/erreurs standard) et AppServiceHTTPLogs (journaux de serveur web). Remplacez _\<app-name>_ et _\<workspace-name>_ par vos valeurs. 

> [!NOTE]
> Les deux premières commandes, `resourceID` et `workspaceID`, sont des variables à utiliser dans la commande `az monitor diagnostic-settings create`. Pour plus d’informations sur cette commande, consultez [Créer des paramètres de diagnostic à l’aide d’Azure CLI](../azure-monitor/platform/diagnostic-settings.md#create-using-azure-cli).
>

```bash
resourceID=$(az webapp show -g myResourceGroup -n <app-name> --query id --output tsv)

workspaceID=$(az monitor log-analytics workspace show -g myResourceGroup --workspace-name <workspace-name> --query id --output tsv)

az monitor diagnostic-settings create --resource $resourceID \
 --workspace $workspaceID \
 -n myMonitorLogs \
 --logs '[{"category": "AppServiceConsoleLogs", "enabled": true},
  {"category": "AppServiceHTTPLogs", "enabled": true}]'
```

## <a name="troubleshoot-the-app"></a>Résoudre les problèmes de l’application

Accédez à `http://<app-name>.azurewebsites.net`.

L’exemple d’application ImageConverter convertit les images incluses du format `JPG` au format `PNG`. Pour ce tutoriel, un bogue a été délibérément placé dans le code. Si vous sélectionnez un certain nombre d’images, l’application génère une erreur HTTP 500 durant leur conversion. Imaginez que ce scénario n’ait pas été envisagé pendant la phase de développement. Vous utiliserez Azure Monitor pour corriger l’erreur.

### <a name="verify-the-app-is-works"></a>Vérifier le bon fonctionnement de l’application

Pour convertir des images, cliquez sur `Tools` et sélectionnez `Convert to PNG`.

![Cliquez sur « Outils » et sélectionnez « Convertir en PNG ».](./media/tutorial-azure-monitor/sample-monitor-app-tools-menu.png)

Sélectionnez les deux premières images, puis cliquez sur `convert`. La conversion s’effectue correctement.

![Sélectionner les deux premières images](./media/tutorial-azure-monitor/sample-monitor-app-convert-two-images.png)

### <a name="break-the-app"></a>Faire planter l’application

Vous avez vérifié le bon fonctionnement de l’application en convertissant deux images. À présent, vous allez essayer de convertir les cinq premières images.

![Convertir les cinq premières images](./media/tutorial-azure-monitor/sample-monitor-app-convert-five-images.png)

Cette action échoue et génère une erreur `HTTP 500` qui n’a pas été testée pendant le développement.

![La conversion génère une erreur HTTP 500](./media/tutorial-azure-monitor/sample-monitor-app-http-500.png)

## <a name="use-log-query-to-view-azure-monitor-logs"></a>Utiliser une requête de journal pour visualiser les journaux Azure Monitor

Voyons quels journaux sont disponibles dans l’espace de travail Log Analytics. 

Cliquez sur ce [lien vers l’espace de travail Log Analytics](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.OperationalInsights%2Fworkspaces) pour accéder à votre espace de travail dans le portail Azure.

Dans le portail Azure, sélectionnez votre espace de travail Log Analytics.

### <a name="log-queries"></a>Requêtes dans les journaux

Les requêtes de journal vous aident à tirer pleinement parti de la valeur des données collectées dans les journaux Azure Monitor. Vous utilisez des requêtes de journal pour identifier les journaux dans AppServiceHTTPLogs et AppServiceConsoleLogs. Pour plus d’informations sur les requêtes de journal, consultez la [vue d’ensemble des requêtes de journal](../azure-monitor/log-query/log-query-overview.md).

### <a name="view-appservicehttplogs-with-log-query"></a>Visualiser AppServiceHTTPLogs en utilisant une requête de journal

Nous avons accédé à l’application. À présent, nous allons visualiser les données associées aux requêtes HTTP, se trouvant dans `AppServiceHTTPLogs`.

1. Cliquez sur `Logs` dans le volet de navigation de gauche.

![Journaux de l’espace de travail Log Analytics](./media/tutorial-azure-monitor/log-analytics-workspace-logs.png)

2. Recherchez `appservice`, puis double-cliquez sur `AppServiceHTTPLogs`.

![Tables de l’espace de travail Log Analytics](./media/tutorial-azure-monitor/log-analytics-workspace-app-service-tables.png)

3. Cliquez sur `Run`.

![Journaux HTTP App Service de l’espace de travail Log Analytics](./media/tutorial-azure-monitor/log-analytics-workspace-app-service-http-logs.png)

La requête `AppServiceHTTPLogs` retourne toutes les requêtes des dernières 24 heures. La colonne `ScStatus` contient l’état HTTP. Pour diagnostiquer les erreurs `HTTP 500`, limitez l’état `ScStatus` à 500 et exécutez la requête comme indiqué ci-dessous :

```kusto
AppServiceHTTPLogs
| where ScStatus == 500
```

### <a name="view-appserviceconsolelogs-with-log-query"></a>Visualiser AppServiceConsoleLogs en utilisant une requête de journal

Vous avez identifié les erreurs HTTP 500. À présent, vous allez observer les sorties/erreurs standard de l’application. Ces journaux se trouvent dans « AppServiceConsoleLogs ».

(1) Cliquez sur `+` pour créer une requête. 

(2) Double-cliquez sur la table `AppServiceConsoleLogs`, puis cliquez sur `Run`. 

Comme la conversion de cinq images entraîne des erreurs de serveur, vous pouvez voir si l’application consigne également des erreurs en filtrant `ResultDescription` sur les erreurs comme indiqué ci-dessous :

```kusto
AppServiceConsoleLogs |
where ResultDescription  contains "error" 
```

Vous voyez l’erreur suivante dans la colonne `ResultDescription` :

<pre>
PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted 
(tried to allocate 16384 bytes) in /home/site/wwwroot/process.php on line 20, 
referer: http://<app-name>.azurewebsites.net/
</pre>

### <a name="join-appservicehttplogs-and-appserviceconsolelogs"></a>Joindre AppServiceHTTPLogs et AppServiceConsoleLogs

Vous avez identifié les erreurs HTTP 500 et les erreurs standard. À présent, vous devez vérifier s’il existe une corrélation entre ces messages. Ensuite, vous devez joindre les tables en fonction de l’horodatage, `TimeGenerated`.

> [!NOTE]
> Une requête effectuant les opérations suivantes a été préparée pour vous :
>
> - Elle filtre HTTPLogs sur les erreurs 500.
> - Elle interroge les journaux de la console.
> - Elle joint les tables sur `TimeGenerated`.
>

Exécutez la requête suivante :

```kusto
let myHttp = AppServiceHTTPLogs | where  ScStatus == 500 | project TimeGen=substring(TimeGenerated, 0, 19), CsUriStem, ScStatus;  

let myConsole = AppServiceConsoleLogs | project TimeGen=substring(TimeGenerated, 0, 19), ResultDescription;

myHttp | join myConsole on TimeGen | project TimeGen, CsUriStem, ScStatus, ResultDescription;
```

La colonne `ResultDescription` présente l’erreur suivante en même temps que les erreurs de serveur web :

<pre>
PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted 
(tried to allocate 16384 bytes) in /home/site/wwwroot/process.php on line 20, 
referer: http://<app-name>.azurewebsites.net/
</pre>

Le message indique que la mémoire est saturée à la ligne 20 de `process.php`. Vous avez pu constater que l’application a généré une erreur quand l’erreur HTTP 500 s’est produite. À présent, examinons le code pour identifier le problème.

## <a name="identify-the-error"></a>Identifier l’erreur

Dans le répertoire local, ouvrez le fichier `process.php` et examinez la ligne 20. 

```php
imagepng($imgArray[$x], $filename);
```

Le premier argument, `$imgArray[$x]`, est une variable qui contient toutes les images JPG (en mémoire) à convertir. Toutefois, `imagepng` a besoin de l’image en cours de conversion uniquement et non de toutes les images. Le préchargement des images n’est pas nécessaire et peut entraîner la saturation de la mémoire, générant des erreurs HTTP 500. Nous allons mettre à jour le code pour charger les images à la demande et voir si ceci résout le problème. Ensuite, vous allez améliorer le code pour résoudre le problème de mémoire.

## <a name="fix-the-app"></a>Corriger l’application

### <a name="update-locally-and-redeploy-the-code"></a>Mettre à jour localement et redéployer le code

Vous devez apporter les modifications suivantes à `process.php` pour résoudre le problème de saturation de la mémoire :

```php
<?php

//Retrieve query parameters
$maxImages = $_GET['images'];
$imgNames  = explode(",",$_GET['imgNames']);

//Load JPEGs into an array (in memory)
for ($x=0; $x<$maxImages; $x++){
    $filename = './images/converted_' . substr($imgNames[$x],0,-4) . '.png';
    imagepng(imagecreatefromjpeg("./images/" . $imgNames[$x]), $filename);
}
```

Validez vos modifications dans Git, puis envoyez les modifications de code à Azure.

```bash
git commit -am "Load images on-demand in process.php"
git push azure main
```

### <a name="browse-to-the-azure-app"></a>Accéder à l’application Azure

Accédez à `http://<app-name>.azurewebsites.net`. 

La conversion des images ne doit plus générer d’erreurs HTTP 500.

![Application PHP exécutée dans Azure App Service](./media/tutorial-azure-monitor/sample-monitor-app-working.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

Supprimez le paramètre de diagnostic à l’aide de la commande suivante :

```bash
az monitor diagnostic-settings delete --resource $resourceID -n myMonitorLogs
```
Vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Configurer une application web avec Azure Monitor.
> * Envoyer des journaux à Log Analytics.
> * Utiliser des requêtes de journal pour identifier et corriger les erreurs d’application web.

## <a name="next-steps"></a><a name="nextsteps"></a>Étapes suivantes
* [Interrogation de journaux d’activité grâce à Azure Monitor](../azure-monitor/log-query/log-query-overview.md)
* [Dépannage d’une application web dans le Service d’application Microsoft Azure à l’aide de Visual Studio](troubleshoot-dotnet-visual-studio.md)
* [Analyse des journaux d’activité d’application dans HDInsight (en anglais)](https://gallery.technet.microsoft.com/scriptcenter/Analyses-Windows-Azure-web-0b27d413)