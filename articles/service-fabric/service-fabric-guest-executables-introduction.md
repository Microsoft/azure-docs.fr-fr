---
title: Empaqueter un fichier exécutable existant sur Azure Service Fabric
description: Découvrez-en plus sur l’empaquetage d’une application existante en tant que fichier exécutable invité afin de la déployer sur un cluster Service Fabric.
ms.topic: conceptual
ms.date: 03/15/2018
ms.openlocfilehash: 8b808d092001196a4d2150e44d508e031db95554
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017744"
---
# <a name="deploy-an-existing-executable-to-service-fabric"></a>Déployer un fichier exécutable existant sur Service Fabric
Vous pouvez exécuter n’importe quel type de code, comme Node.js, Java ou C++ dans Azure Service Fabric en tant que service. Dans la terminologie Service Fabric, ces types de service sont appelés des exécutables invités.

Les invités exécutables sont traités par le Service Fabric comme des services sans état. En conséquence, ils sont placés dans des nœuds dans un cluster, sur la base de la disponibilité et d’autres mesures. Cet article explique comment empaqueter et déployer un exécutable invité dans un cluster Service Fabric, en utilisant Visual Studio ou un utilitaire en ligne de commande.

## <a name="benefits-of-running-a-guest-executable-in-service-fabric"></a>Avantages de l'exécution d'un exécutable invité dans Service Fabric
L’exécution d’un exécutable invité dans un cluster Service Fabric présente plusieurs avantages :

* Haute disponibilité : Les applications qui sont exécutées dans Service Fabric sont hautement disponibles. Service Fabric s’assure que les instances d’une application sont en cours d’exécution.
* Contrôle d'intégrité. La fonction d’analyse du fonctionnement de Service Fabric détecte si une application est en cours d’exécution et fournit des informations de diagnostic en cas d’échec.   
* Gestion du cycle de vie des applications. Outre les mises à niveau sans temps d’arrêt, Service Fabric assure la restauration automatique de la version précédente en cas d’événement signalant un problème d’intégrité lors d’une mise à niveau.    
* Densité. Vous pouvez exécuter plusieurs applications dans un cluster, ce qui élimine le besoin d’exécuter chaque application sur son propre matériel.
* Découvertabilité : À l’aide de REST, vous pouvez appeler le service de nommage de Service Fabric pour trouver d’autres services dans le cluster. 

## <a name="samples"></a>Exemples
* [Exemple pour empaqueter et déployer un fichier exécutable invité](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Exemple de deux exécutables invités (C# et nodejs) communiquant via le service d’attribution de noms à l’aide de REST](https://github.com/Azure-Samples/service-fabric-dotnet-containers)

## <a name="overview-of-application-and-service-manifest-files"></a>Vue d’ensemble des fichiers du manifeste de service et d’application
Dans le cadre du déploiement d’un exécutable invité, il est utile de comprendre le modèle d’empaquetage et de déploiement Service Fabric tel que décrit dans [Modèle d’application](service-fabric-application-model.md). Le modèle d’empaquetage Service Fabric repose sur deux fichiers XML : les manifestes d’application et de service. La définition de schéma pour les fichiers ApplicationManifest.xml et ServiceManifest.xml est installée avec le Kit de développement logiciel (SDK) Service Fabric sous *C:\Program Files\Microsoft SDKs\Service Fabric\schemas\ServiceFabricServiceModel.xsd*.

* **Manifeste d’application** : le manifeste d’application permet de décrire l’application. Il répertorie les services qui la composent, ainsi que d’autres paramètres utilisés pour définir la manière dont les services doivent être déployés (comme le nombre d’instances).

  Dans Service Fabric, une application est une unité de déploiement et de mise à niveau. Une application peut être mise à niveau en tant qu’unité simple, dans laquelle les défaillances (et restaurations) potentielles sont gérées. Service Fabric garantit la réussite de la mise à niveau. En cas d’échec, elle fait en sorte que l’application ne reste pas dans un état inconnu ou instable.
* **Manifeste de service** : le manifeste de service décrit les composants d’un service. Il inclut des données telles que le nom et le type du service, ainsi que son code et sa configuration. Le manifeste de service inclut également des paramètres supplémentaires qui peuvent être utilisés pour configurer le service après son déploiement.

## <a name="application-package-file-structure"></a>Structure de fichier d'un package d'application
Pour que vous puissiez la déployer sur Service Fabric, votre application doit respecter une structure de répertoire prédéfinie. Voici un exemple de structure de ce type.

```
|-- ApplicationPackageRoot
    |-- GuestService1Pkg
        |-- Code
            |-- existingapp.exe
        |-- Config
            |-- Settings.xml
        |-- Data
        |-- ServiceManifest.xml
    |-- ApplicationManifest.xml
```

Le paramètre ApplicationPackageRoot contient le fichier ApplicationManifest.xml, qui définit l’application. Un sous-répertoire pour chaque service inclus dans l’application est utilisé pour contenir tous les artefacts nécessaires au service. Ces sous-répertoires correspondent au fichier ServiceManifest.xml et généralement aux éléments suivants :

* *Code*. Ce répertoire contient le code du service.
* *Config*. Ce répertoire contient un fichier Settings.xml (et d’autres fichiers si nécessaire) auquel le service peut accéder lors de l’exécution pour récupérer des paramètres de configuration spécifiques.
* *Data*. Il s’agit d’un répertoire supplémentaire pour stocker des données locales supplémentaires dont le service peut avoir besoin. Le répertoire Data doit être utilisé uniquement pour stocker des données éphémères. Service Fabric ne copie et ne réplique pas les modifications dans le répertoire des données si le service doit être déplacé, par exemple, pendant le basculement.

> [!NOTE]
> Il est inutile de créer les répertoires `config` et `data` si vous n’en avez pas besoin.
>
>

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants pour obtenir des informations supplémentaires et connaître les tâches connexes.
* [Déployer un fichier exécutable invité](service-fabric-deploy-existing-app.md)
* [Déploiement de plusieurs exécutables invités](./service-fabric-deploy-existing-app.md)
* [Créer votre première application exécutable invitée avec Visual Studio](quickstart-guest-app.md)
* [Exemple pour empaqueter et déployer un exécutable invité](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started), avec un lien vers la version préliminaire de l’outil d’empaquetage
* [Exemple de deux exécutables invités (C# et nodejs) communiquant via le service d’attribution de noms à l’aide de REST](https://github.com/Azure-Samples/service-fabric-containers)
