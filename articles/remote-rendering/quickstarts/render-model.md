---
title: Afficher un modèle avec Unity
description: Guide de démarrage rapide accompagnant l’utilisateur tout au long du rendu d’un modèle
author: florianborn71
ms.author: flborn
ms.date: 01/23/2020
ms.topic: quickstart
ms.openlocfilehash: 74bad563de60abc0c1eb7623a81c83fea03f5ee6
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207221"
---
# <a name="quickstart-render-a-model-with-unity"></a>Démarrage rapide : Afficher un modèle avec Unity

Ce guide de démarrage rapide explique comment exécuter un exemple Unity qui effectue le rendu d’un modèle intégré à distance à l’aide du service Azure Remote Rendering (ARR).

Nous n’aborderons pas en détail l’API ARR elle-même ni la configuration d’un nouveau projet Unity. Ces rubriques sont traitées dans le [tutoriel : Affichage de modèles rendus à distance](../tutorials/unity/view-remote-models/view-remote-models.md).

Dans ce guide de démarrage rapide, vous allez apprendre à :
> [!div class="checklist"]
>
>* Configurer votre environnement de développement local
>* Obtenir et générer l’exemple d’application de démarrage rapide ARR pour Unity
>* Effectuer le rendu d’un modèle dans l’exemple d’application de démarrage rapide ARR

## <a name="prerequisites"></a>Prérequis

Pour pouvoir accéder au service Azure Remote Rendering, vous devez d’abord [créer un compte](../how-tos/create-an-account.md).

Les logiciels suivants doivent être installés :

* SDK Windows 10.0.18362.0 [(télécharger)](https://developer.microsoft.com/windows/downloads/windows-10-sdk)
* Installez la dernière version de Visual Studio 2019 [(télécharger)](https://visualstudio.microsoft.com/vs/older-downloads/)
* [Outils Visual Studio pour Mixed Reality](/windows/mixed-reality/install-the-tools). Plus précisément, les installations de *charge de travail* suivantes sont obligatoires :
  * **Développement Desktop en C++**
  * **Développement de la plateforme Windows universelle (UWP)**
* Git [(télécharger)](https://git-scm.com/downloads)
* Unity 2019.3.1 [(télécharger)](https://unity3d.com/get-unity/download)
  * Installez les modules ci-dessous dans Unity :
    * **UWP** : prise en charge de la build de la plateforme Windows universelle
    * **IL2CPP** - Prise en charge de la génération dans Windows (IL2CPP)

## <a name="clone-the-sample-app"></a>Clonage de l’exemple d’application

Ouvrez une invite de commandes (tapez `cmd` dans le menu Démarrer de Windows) et accédez au répertoire dans lequel vous souhaitez stocker l’exemple de projet ARR.

Exécutez les commandes suivantes :

```cmd
mkdir ARR
cd ARR
git clone https://github.com/Azure/azure-remote-rendering
```

La dernière commande crée dans le répertoire ARR un sous-répertoire contenant les différents exemples de projets pour Azure Remote Rendering.

L’exemple d’application de démarrage rapide pour Unity se trouve dans le sous-répertoire *Unity/Quickstart*.

## <a name="rendering-a-model-with-the-unity-sample-project"></a>Rendu d’un modèle avec l’exemple de projet Unity

Ouvrez Unity Hub et ajoutez l’exemple de projet situé dans le dossier *ARR\azure-remote-rendering\Unity\Quickstart*.
Ouvrez le projet. Si nécessaire, autorisez Unity à mettre à niveau le projet vers la version installée.

Le modèle par défaut dont nous effectuons le rendu est un [exemple de modèle intégré](../samples/sample-model.md). Nous expliquerons comment convertir un modèle personnalisé à l’aide du service de conversion ARR dans le [guide de démarrage rapide suivant](convert-model.md).

### <a name="enter-your-account-info"></a>Entrer les informations de compte

1. Dans le navigateur d’éléments Unity, accédez au dossier *Scenes*, puis ouvrez la scène **Quickstart**.
1. Sous *Hierarchy*, sélectionnez l’objet de jeu **RemoteRendering**.
1. Dans le panneau *Inspector*, saisissez vos [informations d’identification de compte](../how-tos/create-an-account.md). Si vous n’avez pas encore de compte, [créez-en un](../how-tos/create-an-account.md).

![Informations de compte ARR](./media/arr-sample-account-info.png)

> [!IMPORTANT]
> Le portail Azure affiche le domaine de votre compte uniquement sous la forme *mixedreality.azure.com*. Ceci n’est pas suffisant pour établir correctement la connexion.
> Définissez **AccountDomain** sur `<region>.mixedreality.azure.com`, où `<region>` est [l’une des régions disponibles proches de votre emplacement](../reference/regions.md).

Plus tard, nous déploierons ce projet sur un appareil HoloLens et nous nous connecterons au service Remote Rendering à partir de cet appareil. Nous ne disposons pas d’un moyen simple pour entrer les informations d’identification sur l’appareil. Cet exemple de démarrage rapide va donc **enregistrer les informations d’identification dans la scène Unity**.

> [!WARNING]
> Veillez à ne pas placer le projet avec vos informations d’identification enregistrées dans un dépôt susceptible de compromettre des informations de connexion secrètes.

### <a name="create-a-session-and-view-the-default-model"></a>Créer une session et voir le modèle par défaut

Sélectionnez le bouton de **lecture** de Unity pour démarrer la session. Dans le panneau *Game*, un message d’état doit apparaître en superposition en bas de la fenêtre d’affichage. Différents changements d’état ont lieu au cours de la session. À l’état **Starting**, le serveur est lancé, ce qui prend plusieurs minutes. En cas de réussite, l’état devient **Ready**. À présent, la session passe à l’état **Connecting**. Elle tente alors d’atteindre le runtime de rendu sur ce serveur. En cas de réussite, l’exemple passe à l’état **Connected**. À ce stade, il commence à télécharger le modèle pour le rendu. De par la taille du modèle, le téléchargement peut prendre quelques minutes supplémentaires. Le modèle rendu à distance s’affiche alors.

![Sortie de l’exemple](media/arr-sample-output.png)

Félicitations ! Vous voyez maintenant un modèle rendu à distance.

## <a name="inspecting-the-scene"></a>Inspection de la scène

Quand la connexion est établie pour le rendu à distance, le panneau Inspector est mis à jour avec des informations d’état supplémentaires :

![Lecture de l’exemple dans Unity](./media/arr-sample-configure-session-running.png)

Vous pouvez maintenant explorer le graphique de scène en sélectionnant le nouveau nœud et en cliquant sur **Show children** dans le panneau Inspector.

![Unity - Panneau Hierarchy](./media/unity-hierarchy.png)

Un objet de [plan de coupe](../overview/features/cut-planes.md) figure dans la scène. Essayez de l’activer dans ses propriétés et de la déplacer :

![Modification du plan de coupe](media/arr-sample-unity-cutplane.png)

Pour synchroniser les transformations, cliquez sur **Sync now** ou activez l’option **Sync every frame**. Pour les propriétés de composant, il suffit de les modifier.

## <a name="next-steps"></a>Étapes suivantes

Dans le prochain guide de démarrage rapide, nous déploierons l’exemple sur un appareil HoloLens pour voir le modèle rendu à distance dans sa taille d’origine.

> [!div class="nextstepaction"]
> [Démarrage rapide : Déployer l’exemple Unity sur HoloLens](deploy-to-hololens.md)

Vous pouvez également déployer l’exemple sur un PC de bureau.

> [!div class="nextstepaction"]
> [Démarrage rapide : Déployer l’exemple Unity sur un Bureau](deploy-to-desktop.md)