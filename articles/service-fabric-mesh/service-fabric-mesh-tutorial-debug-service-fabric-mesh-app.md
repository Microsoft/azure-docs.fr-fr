---
title: Déboguer une application web Azure Service Fabric Mesh s’exécutant localement
description: Dans ce didacticiel, vous allez déboguer une application de maillage Azure Service Fabric s’exécutant dans votre cluster local.
author: georgewallace
ms.topic: tutorial
ms.date: 10/31/2018
ms.author: gwallace
ms.custom: mvc, devcenter
ms.openlocfilehash: 56cc8b4010dc17cf2b723a72898034de8d6a7175
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91843292"
---
# <a name="tutorial-debug-a-service-fabric-mesh-application-running-in-your-local-development-cluster"></a>Tutoriel : Déboguer une application Service Fabric Mesh s’exécutant dans votre cluster de développement local

Ce tutoriel est la deuxième partie d’une série de tutoriels. Il vous explique comment générer et déboguer une application Azure Service Fabric Mesh dans votre cluster de développement local.

Dans ce tutoriel, vous allez apprendre ce qui suit :

> [!div class="checklist"]
> * Que se passe-t-il lorsque vous créez une application de maillage Azure Service Fabric
> * Comment définir un point d’arrêt pour observer un appel de service à service

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Créer une application Service Fabric Mesh dans Visual Studio](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * Déboguer une application Service Fabric Mesh s’exécutant dans votre cluster de développement local
> * [Déployer une application Service Fabric Mesh](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * [Mettre à niveau une application Service Fabric Mesh](service-fabric-mesh-tutorial-upgrade.md)
> * [Nettoyer des ressources Service Fabric Mesh](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Assurez-vous que vous avez [configuré votre environnement de développement](service-fabric-mesh-howto-setup-developer-environment-sdk.md) qui inclut l’installation du runtime Service Fabric, du Kit de développement logiciel (SDK), de Docker et de Visual Studio 2017.

## <a name="download-the-to-do-sample-application"></a>Télécharger l’exemple d’application de tâches

Si vous n’avez pas créé l’exemple d’application de tâches dans la [première partie de cette série de tutoriels](service-fabric-mesh-tutorial-create-dotnetcore.md), vous pouvez le télécharger. Dans une fenêtre Commande, exécutez la commande suivante pour cloner le référentiel de l’exemple d’application sur votre ordinateur local.

```
git clone https://github.com/azure-samples/service-fabric-mesh
```

L’application se trouve dans le répertoire `src\todolistapp`.

## <a name="build-and-debug-on-your-local-cluster"></a>Générer et déboguer l’application dans votre cluster local

Une image Docker est automatiquement générée et déployée dans votre cluster local dès que votre projet est chargé. Ce processus peut prendre un certain temps. Pour suivre la progression dans le volet **Sortie** de Visual Studio, sélectionnez **Outils Service Fabric** dans le menu déroulant **Afficher la sortie à partir de :** du volet Sortie.

Appuyez sur **F5** pour compiler et exécuter votre service localement. Lorsque le projet est exécuté et débogué localement, Visual Studio va :

* Vérifier que Docker pour Windows est en cours d’exécution et configuré pour utiliser Windows comme système d’exploitation du conteneur.
* Télécharger toute image de base Docker manquante. Cette opération peut prendre un certain temps.
* Générer (ou régénérer) l’image Docker utilisée pour héberger votre projet de code.
* Déployer et exécuter le conteneur dans le cluster de développement Service Fabric local.
* Exécuter vos services et accéder à tous les points d’arrêt que vous avez définis.

Une fois que le déploiement local est terminé et que Visual Studio exécute votre application, une fenêtre de navigateur s’ouvre sur un exemple de page web par défaut.

## <a name="debugging-tips"></a>Conseils de débogage

Accélérez considérablement votre premier débogage (F5) en suivant les instructions fournies dans [Optimiser le niveau de performance de Visual Studio](service-fabric-mesh-howto-optimize-vs.md).

Il existe actuellement un problème entraînant l’échec de la connexion au service par l’appel de la méthode `using (HttpResponseMessage response = client.GetAsync("").GetAwaiter().GetResult())`. Cette situation peut se produire chaque fois que votre adresse IP hôte change. Pour résoudre ce problème :

1. Supprimez l’application du cluster local (dans Visual Studio, **Générer** > **Nettoyer la solution**).
2. À partir du gestionnaire de cluster local Service Fabric, sélectionnez **Arrêter le cluster local**, puis **Démarrer le cluster local**.
3. Redéployez l’application (dans Visual Studio, **F5**).

Si vous obtenez l’erreur **No Service Fabric local cluster is running** (Aucun cluster Service Fabric local n’est en cours d’exécution), assurez-vous que le gestionnaire de cluster local (LCM) Service Fabric est en cours d’exécution, cliquez avec le bouton droit sur l’icône LCM dans la barre des tâches, puis cliquez sur **Démarrer le cluster local**. Une fois le cluster démarré, revenez à Visual Studio et appuyez sur la touche **F5**.

Si vous obtenez une erreur **404** lorsque l’application démarre, cela signifie probablement que vos variables d’environnement dans le fichier **service.yaml** sont incorrectes. Assurez-vous que les valeurs `ApiHostPort` et `ToDoServiceName` sont correctement définies conformément aux instructions de l’article [Créer des variables d’environnement](./service-fabric-mesh-tutorial-create-dotnetcore.md#create-environment-variables).

Si vous obtenez des erreurs de build dans le fichier **service.yaml**, assurez-vous que des espaces, et non des tabulations, sont utilisés pour mettre en retrait les lignes. En outre, pour l’instant, vous devez générer l’application en utilisant les paramètres régionaux anglais.

### <a name="debug-in-visual-studio"></a>Déboguer dans Visual Studio

Pour déboguer une application Service Fabric Mesh dans Visual Studio, vous utilisez un cluster de développement Service Fabric local. Pour voir comment des éléments de tâche sont récupérés à partir du service back-end, effectuez un débogage dans la méthode OnGet().
1. Dans le projet **WebFrontEnd**, ouvrez **Pages** > **Index.cshtml** > **Index.cshtml.cs** et définissez un point d’arrêt dans la méthode **OnGet** (ligne 17).
2. Dans le projet **ToDoService**, ouvrez **TodoController.cs** et définissez un point d’arrêt dans la méthode **Get** (ligne 15).
3. Accédez à votre navigateur et actualisez la page. Vous avez atteint le point d’arrêt dans la méthode `OnGet()` du service web frontal. Vous pouvez examiner la variable `backendUrl` pour voir comment les variables d’environnement que vous avez définies dans le fichier **service.yaml** forment l’URL utilisée pour contacter le service back-end.
4. Survolez (F10) l’appel `client.GetAsync(backendUrl).GetAwaiter().GetResult())` pour atteindre le point d'arrêt `Get()` du contrôleur. Dans cette méthode, vous pouvez voir comment la liste des éléments de tâche est récupérée à partir de la liste en mémoire.
5. Lorsque vous avez terminé, arrêtez le débogage de votre projet dans Visual Studio en appuyant sur **Maj+F5**.

## <a name="next-steps"></a>Étapes suivantes

Cette partie du didacticiel vous a permis de répondre aux questions suivantes :

> [!div class="checklist"]
> * Que se passe-t-il lorsque vous créez une application de maillage Azure Service Fabric
> * Comment définir un point d’arrêt pour observer un appel de service à service

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Déployer une application Service Fabric Mesh](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
