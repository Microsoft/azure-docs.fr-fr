---
title: Monter le volume emptyDir pour le groupe de conteneurs
description: Découvrez comment monter un volume emptyDir pour partager des données entre les conteneurs d’un groupe de conteneurs dans Azure Container Instances
ms.topic: article
ms.date: 01/31/2020
ms.openlocfilehash: 64a3c83008f163167528a5e5987fe2316942d5bc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77117738"
---
# <a name="mount-an-emptydir-volume-in-azure-container-instances"></a>Monter un volume emptyDir dans Azure Container Instances

Découvrez comment monter un volume *emptyDir* pour partager des données entre les conteneurs d’un groupe de conteneurs dans Azure Container Instances. Utilisez les volumes *emptyDir* comme caches éphémères pour vos charges de travail conteneurisées.

> [!NOTE]
> Le montage d’un volume *emptyDir* est actuellement limité aux conteneurs Linux. Nous travaillons actuellement à proposer toutes ces fonctionnalités dans des conteneurs Windows. En attendant, nous vous invitons à découvrir les différences actuelles de la plateforme dans la [Vue d’ensemble](container-instances-overview.md#linux-and-windows-containers).

## <a name="emptydir-volume"></a>Volume emptyDir

Le volume *emptyDir* fournit un répertoire accessible en écriture à chaque conteneur d’un groupe de conteneurs. Les conteneurs du groupe peuvent lire et écrire les mêmes fichiers dans le volume qui peut être monté en utilisant des chemins d’accès identiques ou différents dans chaque conteneur.

Exemples d’utilisation d’un volume *emptyDir*:

* Espace scratch
* Points de contrôle au cours de tâches longues
* Stocker des données extraites par un conteneur sidecar et servies par un conteneur d’applications

Les données figurant dans un volume *emptyDir* sont conservées au fil des pannes de conteneur. Toutefois, il n’est pas garanti que les conteneurs redémarrés conservent les données figurant dans un volume *emptyDir*. Si vous arrêtez un groupe de conteneurs, le volume *emptyDir* n’est pas conservé.

La taille maximale d’un volume *emptyDir* Linux est de 50 Go.

## <a name="mount-an-emptydir-volume"></a>Monter un volume emptyDir

Pour monter un volume emptyDir dans une instance de conteneur, vous pouvez effectuer le déploiement à l’aide d’un [modèle Azure Resource Manager](/azure/templates/microsoft.containerinstance/containergroups), d’un [fichier YAML](container-instances-reference-yaml.md) ou d’autres méthodes programmatique permettant de déployer un groupe de conteneurs.

Tout d’abord, remplissez le tableau `volumes` dans la section `properties` du groupe de conteneurs du fichier. Ensuite, pour chaque conteneur du groupe de conteneurs dans lequel vous souhaitez monter le volume *emptyDir*, remplissez le tableau `volumeMounts` dans la section `properties` de la définition de conteneur.

Par exemple, le modèle Resource Manager suivant crée un groupe de conteneurs consistant en deux conteneurs montant chacun le volume *emptyDir* :

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-emptydir.json -->
[!code-json[volume-emptydir](~/azure-docs-json-samples/container-instances/aci-deploy-volume-emptydir.json)]

Pour voir des exemples de déploiement d’un groupe de conteneurs, consultez [Déployer un groupe multiconteneur avec un modèle Resource Manager](container-instances-multi-container-group.md) et [Déployer un groupe multiconteneur à l’aide d’un fichier YAML](container-instances-multi-container-yaml.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment monter d’autres types de volumes dans Azure Container Instances :

* [Monter un partage de fichiers Azure dans Azure Container Instances](container-instances-volume-azure-files.md)
* [Monter un volume gitRepo dans Azure Container Instances](container-instances-volume-gitrepo.md)
* [Monter un volume secret dans Azure Container Instances](container-instances-volume-secret.md)