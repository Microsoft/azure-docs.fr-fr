---
title: Déployer rapidement une application existante sur un cluster
description: Utilisez un cluster Microsoft Azure Service Fabric pour héberger une application Node.js existante avec Visual Studio.
ms.topic: conceptual
ms.date: 12/06/2017
ms.custom: devx-track-js
ms.openlocfilehash: dfd1fe9db54925bc17a53a7686fa34d5ea53cd5a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013596"
---
# <a name="host-a-nodejs-application-on-azure-service-fabric"></a>Héberger une application Node.js sur Microsoft Azure Service Fabric

Ce démarrage rapide vous permet de déployer une application existante (Node.js dans cet exemple) dans un cluster Service Fabric s’exécutant sur Azure.

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer, assurez-vous que vous avez bien [configuré votre environnement de développement](service-fabric-get-started.md). Cela inclut l’installation du SDK Service Fabric et de Visual Studio 2019 ou 2015.

Vous devez également disposer d’une application Node.js existante pour le déploiement. Ce démarrage rapide utilise un site web Node.js simple qui peut être téléchargé [ici][download-sample]. Extrayez ce fichier vers votre dossier `<path-to-project>\ApplicationPackageRoot\<package-name>\Code\` après avoir créé le projet dans l’étape suivante.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][create-account].

## <a name="create-the-service"></a>Créer le service

Lancez Visual Studio en tant qu’**administrateur**.

Créer un projet avec `CTRL`+`SHIFT`+`N`

Dans la boîte de dialogue **Nouveau projet**, sélectionnez **Cloud > Application Service Fabric**.

Nommez l’application **MyGuestApp**, puis appuyez sur **Créer**.

>[!IMPORTANT]
>Node.js peut facilement dépasser la limite de 260 caractères pour les chemins d’accès de Windows. Utilisez un chemin d’accès court pour le projet en lui-même, par exemple **c:\code\svc1**. Vous pouvez éventuellement suivre **[ces instructions](https://stackoverflow.com/a/41687101/1664231)** pour activer les chemins d’accès longs des fichiers dans Windows 10.
   
![Boîte de dialogue Nouveau projet dans Visual Studio][new-project]

Vous pouvez créer n’importe quel type de service Service Fabric à partir de la boîte de dialogue suivante. Pour ce démarrage rapide, choisissez **Exécutable invité**.

Nommez le service **MyGuestService** et définissez les options à droite sur les valeurs suivantes :

| Paramètre                   | Valeur |
| ------------------------- | ------ |
| Dossier du package de code       | _&lt;le dossier avec votre application Node.js&gt;_ |
| Comportement du package de code     | Copiez le contenu du dossier vers le projet |
| Programme                   | node.exe |
| Arguments                 | server.js |
| Dossier de travail            | CodePackage |

Appuyez sur **OK**.

![Boîte de dialogue Nouveau service dans Visual Studio.][new-service]

Visual Studio crée le projet d’application et le projet de service d’acteur et les affiche dans l’Explorateur de solutions.

Le projet d’application (**MyGuestApp**) ne contient pas de code directement. Le projet fait référence à un ensemble de projets de service. En outre, il contient trois autres types de contenu :

* **Profils de publication**  
Préférences d’outils pour différents environnements.

* **Scripts**  
Script PowerShell de déploiement/mise à niveau de votre application.

* **Définition d’application**  
Inclut le manifeste d’application dans le dossier *ApplicationPackageRoot*. Les fichiers de paramètres d’application associés, qui définissent l’application et vous permettent de la configurer spécifiquement pour un environnement donné, se trouvent dans le dossier *ApplicationParameters*.
    
Pour avoir une vue d’ensemble du contenu du projet de service, consultez l’article [Prise en main de Reliable Services](service-fabric-reliable-services-quick-start.md).

## <a name="set-up-networking"></a>Configurer la mise en réseau

L’exemple d’application Node.js que nous déployons utilise le port **80**, et nous devons indiquer à Service Fabric que nous avons besoin que ce port soit exposé.

Ouvrez le fichier **ServiceManifest.xml** dans le projet. Au bas du manifeste, il existe un `<Resources> \ <Endpoints>` avec une entrée déjà définie. Modifiez cette entrée pour ajouter `Port`, `Protocol` et `Type`. 

```xml
  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="MyGuestAppServiceTypeEndpoint" Port="80" Protocol="http" Type="Input" />
    </Endpoints>
  </Resources>
```

## <a name="deploy-to-azure"></a>Déployer dans Azure

Si vous appuyez sur **F5** et exécutez le projet, il est déployé sur le cluster local. Toutefois, nous allons le déployer vers Azure.

Cliquez avec le bouton droit sur le projet et choisissez **Publier...** , qui ouvre une boîte de dialogue de publication sur Azure.

![Boîte de dialogue de publication sur Azure pour un service Service Fabric][publish]

Sélectionnez le profil cible **PublishProfiles\Cloud.xml**.

Si vous ne l’avez pas fait précédemment, choisissez un compte Azure vers lequel effectuer le déploiement. Si vous n’en avez pas, [obtenez-en un][create-account].

Sous **Point de terminaison de connexion**, sélectionnez le cluster Service Fabric vers lequel effectuer le déploiement. Si vous n’en avez pas, sélectionnez **&lt;Créer un cluster...&gt;** qui ouvre la fenêtre de navigateur web dans le portail Azure. Pour plus d’informations, consultez [Création d’un cluster dans le portail Azure](service-fabric-cluster-creation-via-portal.md#create-cluster-in-the-azure-portal). 

Lorsque vous créez le cluster Service Fabric, veillez à définir le paramètre **Points de terminaison personnalisés** sur **80**.

![Configuration de type nœud Service Fabric avec le point de terminaison personnalisé][custom-endpoint]

La création d’un cluster Service Fabric prend un certain temps. Une fois qu’il a été créé, revenez à la boîte de dialogue de publication et sélectionnez **&lt;Actualiser&gt;** . Le nouveau cluster est répertorié dans la zone de liste déroulante ; sélectionnez-le.

Appuyez sur **Publier** et attendez que le déploiement se termine.

Cela peut prendre quelques minutes. Une fois terminé, quelques minutes peuvent être nécessaires pour que l’application soit totalement disponible.

## <a name="test-the-website"></a>Tester le site web

Une fois que votre service a été publié, testez-le dans un navigateur web. 

Tout d’abord, ouvrez le portail Azure et recherchez votre service Service Fabric.

Consultez le panneau Vue d’ensemble de l’adresse du service. Utilisez le nom de domaine de la propriété _Point de terminaison de connexion du client_. Par exemple : `http://mysvcfab1.westus2.cloudapp.azure.com`.

![Panneau Vue d’ensemble de Service Fabric sur le portail Azure][overview]

Accédez à cette adresse où vous verrez la réponse `HELLO WORLD`.

## <a name="delete-the-cluster"></a>Supprimer le cluster

N’oubliez pas de supprimer toutes les ressources que vous avez créées pour ce démarrage rapide, car celles-ci vous sont facturées.

## <a name="next-steps"></a>Étapes suivantes
Apprenez-en davantage sur les [exécutables invités](service-fabric-guest-executables-introduction.md).

<!-- Image References -->

[new-project]: ./media/quickstart-guest-app/new-project.png
[new-service]: ./media/quickstart-guest-app/template.png
[solution-exp]: ./media/quickstart-guest-app/solution-explorer.png
[publish]: ./media/quickstart-guest-app/publish.png
[overview]: ./media/quickstart-guest-app/overview.png
[custom-endpoint]: ./media/quickstart-guest-app/custom-endpoint.png

[download-sample]: https://github.com/MicrosoftDocs/azure-cloud-services-files/raw/temp/service-fabric-node-website.zip
[create-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
