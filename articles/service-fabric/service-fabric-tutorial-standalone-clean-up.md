---
title: Nettoyer un cluster autonome
description: Dans ce tutoriel, découvrez comment supprimer des ressources AWS ou Azure pour votre cluster Service Fabric autonome.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: 0d46e9068a311594f779411c3ccee2b408febb3f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842884"
---
# <a name="tutorial-clean-up-your-standalone-cluster"></a>Didacticiel : Supprimer votre cluster autonome

Les clusters Service Fabric autonomes vous permettent de choisir votre propre environnement pour héberger Service Fabric. Dans le cadre de cette série de tutoriels, vous allez créer un cluster autonome hébergé sur AWS ou Azure, puis installer une application dans celui-ci.

Ce tutoriel est la quatrième partie de la série. Cette partie du tutoriel vous montre comment supprimer les ressources AWS ou Azure que vous avez créées pour héberger votre cluster Service Fabric.

Dans cet article, vous allez apprendre à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Enlever un cluster Service Fabric
> * Supprimer vos ressources AWS ou Azure

## <a name="remove-a-service-fabric-cluster"></a>Enlever un cluster Service Fabric

1. Établissez une connexion RDP à la machine virtuelle que vous avez utilisée pour installer Service Fabric.
2. Ouvrez PowerShell.
3. Basculez vers le dossier extrait à partir du deuxième didacticiel.
4. Exécutez la commande suivante pour supprimer le cluster Service Fabric :

  ```powershell
  .\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json
  ```

5. Quand vous y êtes invité, entrez `Y`. Si l’opération s’est déroulée correctement, vous devez obtenir le résultat suivant (avec vos propres adresses IP) :

  ```powershell
  Best Practices Analyzer completed successfully.
  Removing configuration from machine 172.31.21.141
  Removing configuration from machine 172.31.27.1
  Removing configuration from machine 172.31.20.163
  Configuration removed from machine 172.31.21.141
  Configuration removed from machine 172.31.27.1
  Configuration removed from machine 172.31.20.163
  The cluster is successfully removed.
  ```

## <a name="delete-aws-resources"></a>Supprimer des ressources AWS

1. Connectez-vous à votre compte AWS.
2. Accédez à la console EC2.
3. Sélectionnez les trois nœuds que vous avez créés dans la première partie du didacticiel.
4. Sélectionnez **Actions** > **État de l’instance** > **Terminer**.

## <a name="delete-azure-resources"></a>Supprimer les ressources Azure

1. Connectez-vous au portail Azure.
2. Accédez à la section **Machines virtuelles**.
3. Sélectionnez les cases des trois nœuds que vous avez créés dans la première partie du didacticiel.
4. Sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à supprimer les ressources que vous avez créées lors des étapes précédentes.

> [!div class="checklist"]
> * Nettoyage des ressources

> [!div class="nextstepaction"]
> [Revenir au début](service-fabric-tutorial-standalone-create-infrastructure.md)
