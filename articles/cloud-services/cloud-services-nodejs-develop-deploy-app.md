---
title: Guide de mise en route Node.js
description: Apprenez à créer une application web Node.js simple et à la déployer vers un service cloud Azure.
services: cloud-services
documentationcenter: nodejs
author: tgore03
ms.service: cloud-services
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 08/17/2017
ms.author: tagore
ms.custom: devx-track-js
ms.openlocfilehash: 86bc968c922bcfb90da9ae2babd456ce4db81cb7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91327474"
---
# <a name="build-and-deploy-a-nodejs-application-to-an-azure-cloud-service"></a>Création et déploiement d'une application Node.js dans Azure Cloud Services

Ce didacticiel explique comment créer une application Node.js simple s’exécutant dans Azure Cloud Services. Les services Cloud Services sont des composantes des applications cloud extensibles dans Azure. Ils permettent de séparer, de gérer et d'étendre de façon indépendante le composant frontal et le composant principal de votre application.  Cloud Services héberge de façon fiable chaque rôle sur une machine virtuelle dédiée.

Pour plus d'informations sur les services cloud et pour connaître les différences avec les sites Web Azure et les machines virtuelles, consultez la page [Comparaison des sites Web Azure, des services cloud et des machines virtuelles].

> [!TIP]
> Vous voulez créer un simple site Web ? Si votre scénario ne comporte qu’un simple composant frontal web, envisagez d’[utiliser une application web légère]. Vous pouvez facilement mettre à niveau vers un service cloud en fonction du développement de votre application et de vos besoins.

Dans ce didacticiel, vous allez créer une application Web simple, hébergée dans un rôle Web. Vous utiliserez l’émulateur de calcul pour tester votre application localement, puis déploierez cette dernière à l’aide d’outils en ligne de commande PowerShell.

Il s’agit d’une application « hello world » simple :

![Navigateur Web avec la page Web Hello World][A web browser displaying the Hello World web page]

## <a name="prerequisites"></a>Conditions préalables requises
> [!NOTE]
> Ce didacticiel utilise Azure PowerShell, qui nécessite Windows.

* Installez et configurez [Azure PowerShell].
* Téléchargez et installez le [Kit de développement logiciel (SDK) Azure pour .NET 2.7]. Dans le programme d'installation, sélectionnez :
  * MicrosoftAzureAuthoringTools
  * MicrosoftAzureComputeEmulator

## <a name="create-an-azure-cloud-service-project"></a>Créer un projet Azure Cloud Services
Effectuez les tâches suivantes pour créer un projet Azure Cloud Services, avec la structure Node.js de base :

1. Exécutez **Windows PowerShell** en tant qu’administrateur ; à partir du **menu Démarrer** ou de l’**écran d’accueil**, recherchez **Windows PowerShell**.
2. [Connectez PowerShell] à votre abonnement.
3. Entrez l’applet de commande PowerShell suivante pour créer le projet :

   ```powershell
   New-AzureServiceProject helloworld
   ```

   ![Le résultat de la nouvelle commande helloworld New-AzureService][The result of the New-AzureService helloworld command]

   La cmdlet **New-AzureServiceProject** génère une structure de base pour publier une application Node.js dans un service cloud. Elle contient les fichiers de configuration nécessaires à la publication sur Azure. La cmdlet change aussi votre répertoire de travail et le remplace par le répertoire du service.

   L’applet de commande crée les fichiers suivants :

   * **ServiceConfiguration.Cloud.cscfg**, **ServiceConfiguration.Local.cscfg** et **ServiceDefinition.csdef** sont des fichiers propres à Azure, nécessaires à la publication de votre application. Pour plus d'informations, consultez la page [Présentation de la création d'un service hébergé pour Azure].
   * **deploymentSettings.json**stocke les paramètres locaux utilisés par les cmdlets de déploiement Azure PowerShell.

4. Entrez la commande suivante pour ajouter un nouveau rôle Web :

   ```powershell
   Add-AzureNodeWebRole
   ```

   ![Résultat de la commande Add-AzureNodeWebRole][The output of the Add-AzureNodeWebRole command]

   La cmdlet **Add-AzureNodeWebRole** crée une application Node.js de base. Elle modifie également les fichiers **.csfg** et **.csdef** afin d’ajouter des entrées de configuration pour le nouveau rôle.

   > [!NOTE]
   > Si vous ne spécifiez pas de nom de rôle, un nom par défaut est utilisé. Vous pouvez indiquer un nom comme premier paramètre de la cmdlet : `Add-AzureNodeWebRole MyRole`

L’application Node.js est définie dans le fichier **server.js**, situé dans le répertoire du rôle web (**WebRole1** par défaut). Voici le code :

```js
var http = require('http');
var port = process.env.port || 1337;
http.createServer(function (req, res) {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World\n');
}).listen(port);
```

Ce code est essentiellement identique à l’exemple « Hello World » sur le site web [nodejs.org] , sauf qu’il utilise le numéro de port attribué par l’environnement de cloud.

## <a name="deploy-the-application-to-azure"></a>Déploiement de l’application dans Azure

> [!NOTE]
> Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez [activer les avantages de votre abonnement MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) ou [vous inscrire pour un compte gratuit](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A85619ABF).

### <a name="download-the-azure-publishing-settings"></a>Télécharger les paramètres de publication Azure
Pour déployer votre application sur Azure, vous devez télécharger les paramètres de publication de votre abonnement Azure.

1. Exécutez l’applet de commande Azure PowerShell suivante :

    ```powershell
    Get-AzurePublishSettingsFile
    ```

   Ceci utilise votre navigateur pour accéder à la page de téléchargement des paramètres de publication. Il est possible que vous soyez invité à vous connecter avec un compte Microsoft. Si c’est le cas, utilisez le compte associé à votre abonnement Azure.

   Enregistrez le profil téléchargé dans un emplacement auquel vous pouvez accéder facilement.
2. Exécutez l’applet de commande suivante pour importer le profil de publication que vous avez téléchargé :

    ```powershell
    Import-AzurePublishSettingsFile [path to file]
    ```

    > [!NOTE]
    > Après l’importation des paramètres de publication, pensez à supprimer le fichier .publishSettings téléchargé, car il contient des informations qui pourraient permettre à d’autres personnes d’accéder à votre compte.

### <a name="publish-the-application"></a>Publication de l'application
Pour publier, exécutez les commande suivantes :

```powershell
$ServiceName = "NodeHelloWorld" + $(Get-Date -Format ('ddhhmm'))
Publish-AzureServiceProject -ServiceName $ServiceName  -Location "East US" -Launch
```

* **-ServiceName** indique le nom du déploiement. Ce nom doit être unique, sans quoi le processus de publication échouera. La commande **Get-Date** s'attache à une chaîne de date et d'heure qui doit rendre le nom unique.
* **-Emplacement** indique le centre de données dans lequel l'application sera hébergée. Pour afficher une liste des centres de données disponibles, utilisez la cmdlet **Get-AzureLocation** .
* **-Launch** ouvre une fenêtre de navigateur et accède au service hébergé une fois le déploiement terminé.

Une fois la publication effectuée, vous devez obtenir une réponse semblable à celle-ci :

![Résultat de la commande Publish-AzureService][The output of the Publish-AzureService command]

> [!NOTE]
> Il peut s’écouler plusieurs minutes avant que l’application soit déployée et disponible lors de la première publication.

Une fois le déploiement terminé, une fenêtre de navigateur s'ouvre et accède au service cloud.

![Fenêtre de navigateur affichant la page Hello World ; l’URL indique que la page est hébergée sur Azure.][A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]

Votre application s'exécute maintenant sur Azure.

La cmdlet **Publish-AzureServiceProject** effectue les opérations suivantes :

1. Elle crée un package à déployer. Ce package contient tous les fichiers de votre dossier d’application.
2. Elle crée un **compte de stockage** , si celui-ci n'existe pas. Le compte de stockage Azure permet de stocker le package de l'application au cours du déploiement. Vous pouvez supprimer en toute sécurité le compte de stockage une fois le déploiement terminé.
3. Elle crée un **service cloud** , si celui-ci n'existe pas. Le **service cloud** est le conteneur dans lequel votre application est hébergée lorsqu'elle est déployée sur Azure. Pour plus d'informations, consultez la page [Présentation de la création d'un service hébergé pour Azure].
4. Elle publie le package de déploiement sur Azure.

## <a name="stopping-and-deleting-your-application"></a>Arrêt et suppression de votre application
Après avoir déployé votre application, vous pouvez la désactiver afin de vous éviter des coûts supplémentaires. Azure facture les instances de rôle Web par heure de serveur consommée. Une fois votre application déployée, elle consomme du temps de serveur, même si les instances ne sont pas exécutées et sont arrêtées.

1. Dans la fenêtre Windows PowerShell, arrêtez le déploiement du service créé dans la section précédente à l'aide de la cmdlet suivante :

    ```powershell
    Stop-AzureService
    ```

   L'arrêt du service peut prendre plusieurs minutes. Une fois le service arrêté, vous recevez un message confirmant l'arrêt du service.

   ![État de la commande Stop-AzureService.][The status of the Stop-AzureService command]
2. Pour supprimer le service, utilisez la cmdlet suivante :

    ```powershell
    Remove-AzureService
    ```

   Lorsque vous y êtes invité, entrez **Y** pour supprimer le service.

   La suppression du service peut prendre plusieurs minutes. Une fois le service supprimé, vous recevez un message confirmant la suppression du service.

   ![État de la commande Remove-AzureService][The status of the Remove-AzureService command]

   > [!NOTE]
   > La suppression du service ne supprime pas le compte de stockage qui a été créé lors de la publication initiale du service. Le stockage utilisé continuera à vous être facturé. Si aucun autre élément n’utilise le stockage, il peut être préférable de le supprimer.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations, consultez le [Centre de développement Node.js].

<!-- URL List -->

[Comparaison des sites Web Azure, des services cloud et des machines virtuelles]: /azure/architecture/guide/technology-choices/compute-decision-tree
[utiliser une application web légère]: ../app-service/quickstart-nodejs.md
[Azure PowerShell]: /powershell/azure/
[Azure SDK for .NET 3.0]: https://www.microsoft.com/download/details.aspx?id=54917
[Connectez PowerShell]: /powershell/azure/
[nodejs.org]: https://nodejs.org/
[Présentation de la création d'un service hébergé pour Azure]: https://azure.microsoft.com/documentation/services/cloud-services/
[Centre de développement Node.js]: https://azure.microsoft.com/develop/nodejs/

<!-- IMG List -->

[The result of the New-AzureService helloworld command]: ./media/cloud-services-nodejs-develop-deploy-app/node9.png
[The output of the Add-AzureNodeWebRole command]: ./media/cloud-services-nodejs-develop-deploy-app/node11.png
[A web browser displaying the Hello World web page]: ./media/cloud-services-nodejs-develop-deploy-app/node14.png
[The output of the Publish-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node19.png
[A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]: ./media/cloud-services-nodejs-develop-deploy-app/node21.png
[The status of the Stop-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node48.png
[The status of the Remove-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node49.png



