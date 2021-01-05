---
title: Déplacer des données vers un conteneur cloud Azure HPC Cache
description: Comment ajouter des données au stockage Blob Azure en vue de l’utiliser avec Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 10/30/2019
ms.author: v-erkel
ms.openlocfilehash: 0da8a4fc1b59976c50cd96f2155715a4cb178cc9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87072761"
---
# <a name="move-data-to-azure-blob-storage"></a>Déplacer les données dans le stockage d’objets blob Azure

Si votre workflow comprend le déplacement des données vers le stockage Blob Azure, vérifiez que vous utilisez une stratégie efficace. Vous pouvez précharger des données dans un nouveau conteneur d’objets blob avant de le définir en tant que cible de stockage, ou ajouter le conteneur, puis copier vos données à l’aide d’Azure HPC Cache.

Cet article fournit les meilleures méthodes pour déplacer des données vers le stockage Blob en vue d’une utilisation avec Azure HPC Cache.

Gardez à l’esprit les points suivants :

* Azure HPC Cache utilise un format de stockage spécialisé permettant d’organiser les données dans le stockage Blob. C’est pour cela qu’une cible de stockage Blob doit être soit un nouveau conteneur vide, soit un conteneur d’objets blob précédemment utilisé pour les données Azure HPC Cache

* La copie de données via Azure HPC Cache à une cible de stockage principal est plus efficace lorsque vous utilisez plusieurs clients et des opérations parallèles. Si vous utilisez une simple commande de copie sur un client, la copie des données se fera lentement.

Un utilitaire Python permet de charger du contenu dans un conteneur de stockage Blob. Pour plus d’informations, consultez [Précharger des données dans le stockage Blob](#pre-load-data-in-blob-storage-with-clfsload).

Si vous ne souhaitez pas utiliser l’utilitaire de chargement, ou si vous souhaitez ajouter du contenu à une cible de stockage existante, suivez les conseils de la section [Copier des données via Azure HPC Cache](#copy-data-through-the-azure-hpc-cache) concernant l’ingestion de données parallèle.

## <a name="pre-load-data-in-blob-storage-with-clfsload"></a>Précharger des données dans le stockage Blob avec CLFSLoad

Vous pouvez utiliser l’utilitaire Avere CLFSLoad pour copier des données dans un nouveau conteneur de stockage Blob avant de l’ajouter en tant que cible de stockage. Cet utilitaire s’exécute sur un système Linux unique et écrit des données au format propriétaire, qui est exigé par Azure HPC Cache. CLFSLoad est la méthode la plus efficace pour ajouter des données à un conteneur de stockage Blob devant être utilisé avec le cache.

L’utilitaire Avere CLFSLoad est disponible sur demande auprès de votre équipe Azure HPC Cache. Demandez-le à votre contact d’équipe ou ouvrez un [ticket de support](hpc-cache-support-ticket.md) pour demander de l’aide.

Cette option fonctionne uniquement avec les nouveaux conteneurs vides. Créez le conteneur avant d’utiliser Avere CLFSLoad.

Des informations détaillées sont incluses dans la distribution Avere CLFSLoad, disponible sur demande auprès de votre équipe Azure HPC Cache.

Vue d’ensemble du processus :

1. Préparez un système Linux (machine physique ou virtuelle) avec Python version 3.6 ou ultérieure. Python 3.7 est recommandé pour obtenir de meilleures performances.
1. Installez le logiciel Average-CLFSLoad sur le système Linux.
1. Exécutez le transfert à partir de la ligne de commande Linux.

L’utilitaire Average CLFSLoad a besoin des informations suivantes :

* L’ID du compte de stockage qui comprend votre conteneur de stockage Blob
* Le nom du conteneur de stockage Blob vide
* Un jeton de signature d’accès partagé (SAP) qui permet à l’utilitaire d’écrire dans le conteneur
* Un chemin local vers la source de données (soit un répertoire local qui contient les données à copier, soit le chemin local d’un système distant monté comprenant les données)

## <a name="copy-data-through-the-azure-hpc-cache"></a>Copier des données via Azure HPC Cache

Si vous ne souhaitez pas utiliser l’utilitaire Average CLFSLoad ou si vous souhaitez ajouter une grande quantité de données à une cible de stockage Blob existante, vous pouvez la copier via le cache. Azure HPC Cache est conçu pour servir plusieurs clients simultanément. Par conséquent, pour copier des données via le cache, vous devez utiliser des écritures parallèles à partir de plusieurs clients.

![Diagramme illustrant le déplacement des données multithread et multiclient : en haut à gauche, plusieurs flèches partent d’une icône représentant le stockage matériel local. Les flèches pointent vers quatre ordinateurs clients. À partir de chaque ordinateur client, trois flèches pointent vers Azure HPC Cache. À partir d’Azure HPC Cache, plusieurs flèches pointent vers le stockage Blob Azure.](media/hpc-cache-parallel-ingest.png)

Les commandes ``cp`` ou ``copy`` qui sont généralement utilisées pour transférer des données d’un système de stockage à un autre sont des processus monothread qui ne copient qu’un seul fichier à la fois. Cela signifie que le serveur de fichiers ne reçoit qu’un fichier à la fois, ce qui représente un gaspillage des ressources du cache.

Cette section décrit des stratégies de création d’un système de copie de fichiers multiclient et multithread pour déplacer des données vers le stockage Blob avec Azure HPC Cache. Il explique les concepts de transfert de fichiers et les points de décision qui peuvent être utilisés pour une copie efficace des données à l’aide de plusieurs clients et de simples commandes de copie.

Il décrit également quelques utilitaires qui peuvent être utiles. L’utilitaire ``msrsync`` peut servir à automatiser partiellement le processus de division d’un jeu de données en compartiments et d’utilisation des commandes rsync. Le script ``parallelcp`` est un autre utilitaire qui lit le répertoire source et émet automatiquement des commandes de copie.

### <a name="strategic-planning"></a>Planification stratégique

Quand vous créez une stratégie pour copier des données en parallèle, vous devez comprendre les compromis en termes de taille de fichier, nombre de fichiers et profondeur de répertoire.

* Lorsque les fichiers sont petits, la métrique d’intérêt est fichiers par seconde.
* Lorsque les fichiers sont volumineux (10 MiBi ou plus), la métrique d’intérêt est octets par seconde.

Chaque processus de copie a un débit et un taux de transfert de fichiers qui peuvent être mesurés en minutant la longueur de la commande de copie et en tenant compte de la taille de fichier et du nombre de fichiers. L’explication de la mesure des débits et taux n’entre pas dans le cadre de ce document, mais il est impératif de comprendre si vous allez traiter des fichiers de petite ou grande taille.

Les stratégies d’ingestion de données parallèle avec Azure HPC Cache sont les suivantes :

* La copie manuelle. Vous pouvez créer manuellement une copie multithread sur un client en exécutant plusieurs commandes de copie à la fois en arrière-plan sur des ensembles de fichiers ou de chemins prédéfinis. Pour plus d’informations, consultez [Ingestion de données Azure HPC Cache - Copie manuelle](hpc-cache-ingest-manual.md).

* La copie partiellement automatisée avec ``msrsync`` - ``msrsync`` est un utilitaire wrapper qui exécute plusieurs processus ``rsync`` parallèles. Pour plus d’informations, consultez [Ingestion de données Azure HPC Cache - Méthode msrsync](hpc-cache-ingest-msrsync.md).

* La copie par script avec ``parallelcp``. Découvrez comment créer et exécuter un script de copie parallèle dans [Ingestion de données Azure HPC Cache - Script de copie parallèle](hpc-cache-ingest-parallelcp.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré votre stockage, découvrez comment les clients peuvent monter le cache.

* [Accéder au système Azure HPC Cache](hpc-cache-mount.md)
