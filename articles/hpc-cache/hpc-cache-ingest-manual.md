---
title: Ingestion de données Azure HPC Cache - Copie manuelle
description: Comment utiliser les commandes cp pour déplacer des données vers une cible de stockage Blob dans Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 10/30/2019
ms.author: v-erkel
ms.openlocfilehash: f96a0fa264124f9d050e667b003d98579da63b77
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87092335"
---
# <a name="azure-hpc-cache-data-ingest---manual-copy-method"></a>Ingestion de données Azure HPC Cache - Copie manuelle

Cet article fournit des instructions détaillées sur la copie manuelle de données vers un conteneur de stockage Blob en vue d’une utilisation avec Azure HPC Cache. Il utilise des opérations parallèles multithread pour accélérer la copie des données.

Pour plus d’informations sur le déplacement des données vers le stockage Blob pour Azure HPC Cache, consultez [Déplacer les données vers le stockage Blob Azure](hpc-cache-ingest.md).

## <a name="simple-copy-example"></a>Exemple simple de copie

Vous pouvez créer manuellement une copie multithread sur un client en exécutant plusieurs commandes de copie à la fois en arrière-plan sur des ensembles de fichiers ou chemins prédéfinis.

La commande Linux/UNIX ``cp`` inclut l’argument ``-p`` afin de préserver la propriété et les métadonnées mtime. L’ajout de cet argument aux commandes ci-dessous est facultatif. (Il augmente le nombre d'appels de système de fichiers envoyés par le client au système de fichiers de destination pour la modification des métadonnées.)

Ce simple exemple copie deux fichiers en parallèle :

```bash
cp /mnt/source/file1 /mnt/destination1/ & cp /mnt/source/file2 /mnt/destination1/ &
```

Après avoir émis cette commande, la commande `jobs` indique que deux threads sont en cours d’exécution.

## <a name="copy-data-with-predictable-file-names"></a>Copier des données avec des noms de fichiers prévisibles

Si vos noms de fichier sont prévisibles, vous pouvez utiliser des expressions pour créer des threads de copie parallèle.

Par exemple, si votre répertoire contient 1000 fichiers numérotés de manière séquentielle de `0001` à `1000`, vous pouvez utiliser les expressions suivantes pour créer 10 threads parallèles qui copient chacun 100 fichiers :

```bash
cp /mnt/source/file0* /mnt/destination1/ & \
cp /mnt/source/file1* /mnt/destination1/ & \
cp /mnt/source/file2* /mnt/destination1/ & \
cp /mnt/source/file3* /mnt/destination1/ & \
cp /mnt/source/file4* /mnt/destination1/ & \
cp /mnt/source/file5* /mnt/destination1/ & \
cp /mnt/source/file6* /mnt/destination1/ & \
cp /mnt/source/file7* /mnt/destination1/ & \
cp /mnt/source/file8* /mnt/destination1/ & \
cp /mnt/source/file9* /mnt/destination1/
```

## <a name="copy-data-with-unstructured-file-names"></a>Copier des données avec des noms de fichiers non structurés

Si votre structure de nommage de fichiers n’est pas prévisible, vous pouvez regrouper les fichiers selon les noms des répertoires.

Cet exemple recueille des répertoires entiers à envoyer aux commandes ``cp`` pour les exécuter en tant que tâches en arrière-plan :

```bash
/root
|-/dir1
| |-/dir1a
| |-/dir1b
| |-/dir1c
   |-/dir1c1
|-/dir1d
```

Une fois que les fichiers sont collectés, vous pouvez exécuter des commandes de copie parallèle pour copier de manière récursive les sous-répertoires et tout leur contenu :

```bash
cp /mnt/source/* /mnt/destination/
mkdir -p /mnt/destination/dir1 && cp /mnt/source/dir1/* mnt/destination/dir1/ &
cp -R /mnt/source/dir1/dir1a /mnt/destination/dir1/ &
cp -R /mnt/source/dir1/dir1b /mnt/destination/dir1/ &
cp -R /mnt/source/dir1/dir1c /mnt/destination/dir1/ & # this command copies dir1c1 via recursion
cp -R /mnt/source/dir1/dir1d /mnt/destination/dir1/ &
```

## <a name="when-to-add-mount-points"></a>Quand ajouter des points de montage

Une fois que suffisamment de threads parallèles sont transmis à un même point de montage du système de fichiers de destination, il arrive un moment où l'ajout de threads supplémentaires ne permet pas d'obtenir davantage de débit. (Le débit est mesuré en fichiers/seconde ou octets/seconde, selon votre type de données.) Pire encore, un threading excessif peut parfois provoquer une dégradation du débit.

Dans ce cas, vous pouvez ajouter des points de montage côté client à d'autres adresses de montage Azure HPC Cache, en utilisant le même chemin de montage du système de fichiers distant :

```bash
10.1.0.100:/nfs on /mnt/sourcetype nfs (rw,vers=3,proto=tcp,addr=10.1.0.100)
10.1.1.101:/nfs on /mnt/destination1type nfs (rw,vers=3,proto=tcp,addr=10.1.1.101)
10.1.1.102:/nfs on /mnt/destination2type nfs (rw,vers=3,proto=tcp,addr=10.1.1.102)
10.1.1.103:/nfs on /mnt/destination3type nfs (rw,vers=3,proto=tcp,addr=10.1.1.103)
```

L’ajout de points de montage côté client vous permet de dupliquer (fork) d’autres commandes de copie sur les points de montage `/mnt/destination[1-3]` supplémentaires et d’obtenir ainsi davantage de parallélisme.

Par exemple, si vos fichiers sont très volumineux, vous pouvez définir les commandes de copie de façon à utiliser des chemins de destination distincts pour envoyer plus de commandes en parallèle à partir du client à l’origine de la copie.

```bash
cp /mnt/source/file0* /mnt/destination1/ & \
cp /mnt/source/file1* /mnt/destination2/ & \
cp /mnt/source/file2* /mnt/destination3/ & \
cp /mnt/source/file3* /mnt/destination1/ & \
cp /mnt/source/file4* /mnt/destination2/ & \
cp /mnt/source/file5* /mnt/destination3/ & \
cp /mnt/source/file6* /mnt/destination1/ & \
cp /mnt/source/file7* /mnt/destination2/ & \
cp /mnt/source/file8* /mnt/destination3/ & \
```

Dans l’exemple ci-dessus, l’ensemble des trois points de montage de destination est ciblé par les processus de copie de fichiers clients.

## <a name="when-to-add-clients"></a>Quand ajouter des clients

Enfin, quand vous avez atteint la limite des capacités des clients, l’ajout d’autres threads de copie ou de points de montage supplémentaires ne génère pas d’autre augmentation de la métrique fichiers/seconde ni octets/seconde. Dans ce cas, vous pouvez déployer un autre client avec le même ensemble de points de montage qui exécutera ses propres ensembles de processus de copie de fichiers.

Exemple :

```bash
Client1: cp -R /mnt/source/dir1/dir1a /mnt/destination/dir1/ &
Client1: cp -R /mnt/source/dir2/dir2a /mnt/destination/dir2/ &
Client1: cp -R /mnt/source/dir3/dir3a /mnt/destination/dir3/ &

Client2: cp -R /mnt/source/dir1/dir1b /mnt/destination/dir1/ &
Client2: cp -R /mnt/source/dir2/dir2b /mnt/destination/dir2/ &
Client2: cp -R /mnt/source/dir3/dir3b /mnt/destination/dir3/ &

Client3: cp -R /mnt/source/dir1/dir1c /mnt/destination/dir1/ &
Client3: cp -R /mnt/source/dir2/dir2c /mnt/destination/dir2/ &
Client3: cp -R /mnt/source/dir3/dir3c /mnt/destination/dir3/ &

Client4: cp -R /mnt/source/dir1/dir1d /mnt/destination/dir1/ &
Client4: cp -R /mnt/source/dir2/dir2d /mnt/destination/dir2/ &
Client4: cp -R /mnt/source/dir3/dir3d /mnt/destination/dir3/ &
```

## <a name="create-file-manifests"></a>Créer des manifestes de fichiers

Une fois que vous avez compris les approches ci-dessus (plusieurs threads de copie par destination, plusieurs destinations par client, plusieurs clients par système de fichiers source accessible via le réseau), tenez compte de la recommandation suivante, celle de générer des manifestes de fichiers, puis de les utiliser avec les commandes de copie dans plusieurs clients.

Ce scénario utilise la commande UNIX ``find`` pour créer des manifestes de fichiers ou répertoires :

```bash
user@build:/mnt/source > find . -mindepth 4 -maxdepth 4 -type d
./atj5b55c53be6-01/support/gsi/2018-07-22T21:12:06EDT
./atj5b55c53be6-01/support/pcap/2018-07-23T01:34:57UTC
./atj5b55c53be6-01/support/trace/rolling
./atj5b55c53be6-03/support/gsi/2018-07-22T21:12:06EDT
./atj5b55c53be6-03/support/pcap/2018-07-23T01:34:57UTC
./atj5b55c53be6-03/support/trace/rolling
./atj5b55c53be6-02/support/gsi/2018-07-22T21:12:06EDT
./atj5b55c53be6-02/support/pcap/2018-07-23T01:34:57UTC
./atj5b55c53be6-02/support/trace/rolling
```

Redirigez ce résultat vers un fichier : `find . -mindepth 4 -maxdepth 4 -type d > /tmp/foo`

Vous pouvez ensuite itérer au sein du manifeste, à l’aide de commandes BASH pour compter les fichiers et déterminer les tailles des sous-répertoires :

```bash
ben@xlcycl1:/sps/internal/atj5b5ab44b7f > for i in $(cat /tmp/foo); do echo " `find ${i} |wc -l` `du -sh ${i}`"; done
244    3.5M    ./atj5b5ab44b7f-02/support/gsi/2018-07-18T00:07:03EDT
9      172K    ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-18T05:01:00UTC
124    5.8M    ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-19T01:01:01UTC
152    15M     ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-20T01:01:00UTC
131    13M     ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-20T21:59:41UTC_partial
789    6.2M    ./atj5b5ab44b7f-02/support/gsi/2018-07-20T21:59:41UTC
134    12M     ./atj5b5ab44b7f-02/support/gsi/stats_2018-07-20T22:22:55UTC_hpccache_catchup
7      16K     ./atj5b5ab44b7f-02/support/pcap/2018-07-18T17:12:19UTC
8      83K     ./atj5b5ab44b7f-02/support/pcap/2018-07-18T17:17:17UTC
575    7.7M    ./atj5b5ab44b7f-02/support/cores/armada_main.2000.1531980253.gsi
33     4.4G    ./atj5b5ab44b7f-02/support/trace/rolling
281    6.6M    ./atj5b5ab44b7f-01/support/gsi/2018-07-18T00:07:03EDT
15     182K    ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-18T05:01:00UTC
244    17M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-19T01:01:01UTC
299    31M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-20T01:01:00UTC
256    29M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-20T21:59:41UTC_partial
889    7.7M    ./atj5b5ab44b7f-01/support/gsi/2018-07-20T21:59:41UTC
262    29M     ./atj5b5ab44b7f-01/support/gsi/stats_2018-07-20T22:22:55UTC_hpccache_catchup
11     248K    ./atj5b5ab44b7f-01/support/pcap/2018-07-18T17:12:19UTC
11     88K     ./atj5b5ab44b7f-01/support/pcap/2018-07-18T17:17:17UTC
645    11M     ./atj5b5ab44b7f-01/support/cores/armada_main.2019.1531980253.gsi
33     4.0G    ./atj5b5ab44b7f-01/support/trace/rolling
244    2.1M    ./atj5b5ab44b7f-03/support/gsi/2018-07-18T00:07:03EDT
9      158K    ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-18T05:01:00UTC
124    5.3M    ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-19T01:01:01UTC
152    15M     ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-20T01:01:00UTC
131    12M     ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-20T21:59:41UTC_partial
789    8.4M    ./atj5b5ab44b7f-03/support/gsi/2018-07-20T21:59:41UTC
134    14M     ./atj5b5ab44b7f-03/support/gsi/stats_2018-07-20T22:25:58UTC_hpccache_catchup
7      159K    ./atj5b5ab44b7f-03/support/pcap/2018-07-18T17:12:19UTC
7      157K    ./atj5b5ab44b7f-03/support/pcap/2018-07-18T17:17:17UTC
576    12M     ./atj5b5ab44b7f-03/support/cores/armada_main.2013.1531980253.gsi
33     2.8G    ./atj5b5ab44b7f-03/support/trace/rolling
```

Enfin, vous devez élaborer les commandes de copie de fichiers réelles pour les clients.

Si vous avez quatre clients, utilisez cette commande :

```bash
for i in 1 2 3 4 ; do sed -n ${i}~4p /tmp/foo > /tmp/client${i}; done
```

Si vous avez cinq clients, utilisez quelque chose comme ceci :

```bash
for i in 1 2 3 4 5; do sed -n ${i}~5p /tmp/foo > /tmp/client${i}; done
```

Et pour six... Extrapolez en fonction des besoins.

```bash
for i in 1 2 3 4 5 6; do sed -n ${i}~6p /tmp/foo > /tmp/client${i}; done
```

Vous obtiendrez *N* fichiers résultants, un pour chacun de vos *N* clients qui contient les noms de chemin des répertoires de niveau quatre produits dans le cadre de la sortie de la commande `find`.

Chaque fichier permet de générer la commande de copie :

```bash
for i in 1 2 3 4 5 6; do for j in $(cat /tmp/client${i}); do echo "cp -p -R /mnt/source/${j} /mnt/destination/${j}" >> /tmp/client${i}_copy_commands ; done; done
```

La méthode ci-dessus vous donnera *N* fichiers, chacun avec une commande de copie par ligne, qui peut être exécutée comme un script BASH sur le client.

L’objectif est d’exécuter plusieurs threads de ces scripts simultanément par client en parallèle sur plusieurs clients.
