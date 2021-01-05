---
title: Installer une application sur un cluster autonome
description: Dans ce tutoriel, vous allez apprendre à installer une application dans votre cluster Service Fabric autonome.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: ae946321b34f12c816a717db4a3d07f57feefe52
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96485358"
---
# <a name="tutorial-deploy-an-application-on-your-service-fabric-standalone-cluster"></a>Didacticiel : déployer une application sur votre cluster autonome Service Fabric

Les clusters autonomes Service Fabric vous permettent de choisir votre propre environnement et de créer un cluster dans le cadre de l’approche « n’importe quel SE, n’importe quel cloud » suivie par Service Fabric. Dans cette série de tutoriels, vous créez un cluster autonome hébergé sur AWS et vous déployez une application dans celui-ci.

Ce tutoriel est le troisième de la série.  Les clusters autonomes Service Fabric vous permettent de choisir votre propre environnement et de créer un cluster dans le cadre de l’approche « n’importe quel SE, n’importe quel cloud » avec Service Fabric. Ce didacticiel vous montre comment créer l’infrastructure AWS nécessaire pour héberger ce cluster autonome.

Dans cet article, vous allez apprendre à :

> [!div class="checklist"]
> * Télécharger l’exemple d’application
> * Déployer sur le cluster

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer ce tutoriel :

* [Installez Visual Studio 2019](https://www.visualstudio.com/) et les charges de travail **Développement Azure** et **Développement web et ASP.NET**.
* [Installez le Kit de développement logiciel (SDK) Service Fabric](service-fabric-get-started.md)

## <a name="download-the-voting-sample-application"></a>Télécharger l’exemple d’application de vote

Dans une fenêtre Commande, exécutez la commande suivante pour cloner le référentiel de l’exemple d’application sur votre ordinateur local.

```
git clone https://github.com/Azure-Samples/service-fabric-dotnet-quickstart
```

### <a name="deploy-the-app-to-the-service-fabric-cluster"></a>Déployer l’application sur le cluster Service Fabric

À présent que l’application est téléchargée, vous pouvez la déployer sur un cluster directement à partir de Visual Studio.

1. Ouvrez Visual Studio.

2. Sélectionnez **Fichier** > **Ouvrir**.

3. Accédez au dossier dans lequel vous avez cloné le référentiel git et sélectionnez Voting.sln.

4. Cliquez avec le bouton droit sur le projet de l’application `Voting` dans l’Explorateur de solutions et choisissez **Publier**.

5. Sélectionnez la flèche déroulante pour le **point de terminaison de connexion** et entrez le nom DNS public de l’un des nœuds de votre cluster.  Par exemple : `ec2-34-215-183-77.us-west-2.compute.amazonaws.com:19000`. Dans Azure, un nom de domaine complet (FQDN) n’est pas accordé automatiquement, mais peut facilement [être défini dans la page Vue d’ensemble de la machine virtuelle.](../virtual-machines/create-fqdn.md)

6. Ouvrez votre navigateur préféré et tapez l’adresse du cluster (le point de terminaison de connexion, cette application se déploie sur le port 8080 - par exemple, ec2-34-215-183-77.us-west-2.compute.amazonaws.com:8080).

    ![Réponse de l’API à partir du cluster](./media/service-fabric-tutorial-standalone-cluster/deployed-app.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert comment déployer une application sur votre cluster :

> [!div class="checklist"]
> * Télécharger l’exemple d’application
> * Déployer sur le cluster

Passez au quatrième volet de la série pour nettoyer votre cluster.

> [!div class="nextstepaction"]
> [Nettoyez vos ressources](service-fabric-tutorial-standalone-clean-up.md)