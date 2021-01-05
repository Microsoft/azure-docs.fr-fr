---
title: Comprendre l’espace de noms agrégé d’Azure HPC Cache
description: Comment planifier l’espace de noms virtuel pour votre compte Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 09/30/2020
ms.author: v-erkel
ms.openlocfilehash: 1c28f549cf93d77f6aef6bcde6a2225345a79cc9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91612946"
---
# <a name="plan-the-aggregated-namespace"></a>Planifier l’espace de noms agrégé

Azure HPC Cache permet aux clients d’accéder à un large éventail de systèmes de stockage par le biais d’un espace de noms virtuels qui masque les détails du système de stockage back-end.

Après avoir ajouté une cible de stockage, vous configurez un ou plusieurs chemins d’accès d’espace de noms côté client pour la cible de stockage. Les ordinateurs clients montent ce chemin de fichier et peuvent effectuer des requêtes de lecture de fichier dans le cache au lieu de monter directement le système de stockage.

Azure HPC Cache gérant ce système de fichiers virtuel, vous pouvez modifier la cible de stockage sans modifier le chemin d’accès côté client. Par exemple, vous pouvez remplacer un système de stockage matériel par du stockage cloud sans avoir à réécrire les procédures côté client.

## <a name="aggregated-namespace-example"></a>Exemple d’espace de noms agrégé

Planifiez votre espace de noms agrégé pour que les ordinateurs clients puissent facilement accéder aux informations dont ils ont besoin, et pour que les administrateurs et les ingénieurs du workflow puissent facilement distinguer les chemins.

Par exemple, imaginez un système dans lequel une instance Azure HPC Cache est utilisée pour traiter les données stockées dans un objet blob Azure. L’analyse nécessite des fichiers modèles stockés dans un centre de données local.

Les données des modèle sont stockées dans un centre de données et les informations nécessaires à ce travail sont stockées dans les sous-répertoires suivants :

* */goldline/templates/acme2017/sku798*
* */goldline/templates/acme2017/sku980*

Le système de stockage du centre de données expose les exportations suivantes :

* */*
* */goldline*
* */goldline/templates*

Les données à analyser ont été copiées dans un conteneur de stockage Blob Azure nommé « sourcecollection » à l’aide de l’[utilitaire CLFSLoad](hpc-cache-ingest.md#pre-load-data-in-blob-storage-with-clfsload).

Pour permettre un accès facile via le cache, vous pouvez créer des cibles de stockage avec les chemins d’espace de noms virtuels suivants :

| Système de stockage back-end <br/> (Chemin de fichier NFS ou conteneur d’objets blob) | Chemin d’espace de noms virtuels |
|-----------------------------------------|------------------------|
| /goldline/templates/acme2017/sku798     | /templates/sku798      |
| /goldline/templates/acme2017/sku980     | /templates/sku980      |
| sourcecollection                        | /source/               |

Une cible de stockage NFS peut avoir plusieurs chemins d’accès d’espace de noms virtuels, à condition que chacun d’eux fasse référence à un chemin d’exportation unique. (Lisez [Chemins d’accès aux espaces de noms NFS](add-namespace-paths.md#nfs-namespace-paths) pour connaître le nombre maximal recommandé de chemins d’accès d’espace de noms par cible de stockage NFS).

Les chemins sources NFS étant des sous-répertoires d’une même exportation, vous devez définir plusieurs chemins d’espaces de noms à partir de la même cible de stockage.

| Nom d’hôte de la cible de stockage  | Chemin de l’exportation NFS     | Chemin du sous-répertoire | Chemin de l’espace de noms    |
|--------------------------|---------------------|-------------------|-------------------|
| *Adresse IP ou nom d’hôte* | /goldline/templates | acme2017/sku798   | /templates/sku798 |
| *Adresse IP ou nom d’hôte* | /goldline/templates | acme2017/sku980   | /templates/sku980 |

Une application cliente peut monter le cache et accéder facilement aux chemins d’espaces de noms agrégés ``/source``, ``/templates/sku798`` et ``/templates/sku980``.

Une autre approche consiste à créer un chemin d’accès virtuel comme `/templates` qui établit un lien vers le répertoire parent, `acme2017`, puis permet aux clients d’accéder aux répertoires individuels `sku798` et `sku980` après le montage du cache. Toutefois, vous ne pouvez pas créer de chemin d’accès à l’espace de noms qui est un sous-répertoire du chemin d’accès d’un autre espace de noms. Par conséquent, si vous créez un chemin d’accès au répertoire `acme2017`, vous ne pouvez pas non plus créer de chemins d’accès à des espace de noms pour accéder directement à ses sous-répertoires.

La page de paramètres **Espace de noms** d’Azure HPC Cache reprend le système de fichiers côté client et vous permet d’ajouter ou de modifier des chemins d’accès. Pour en savoir plus, consultez [Planifier l’espace de noms agrégé](add-namespace-paths.md).

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez déterminé la configuration de votre système de fichiers virtuel, suivez les étapes ci-dessous pour le créer :

* [Créez des cibles de stockage](hpc-cache-add-storage.md) pour ajouter vos systèmes de stockage principaux à votre Azure HPC Cache
* [Ajoutez des chemins d’accès à l’espace de noms](add-namespace-paths.md) pour créer l’espace de noms agrégé utilisé par les ordinateurs clients pour accéder aux fichiers
