---
title: Fichier Include
description: Fichier Include
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 01/11/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 358e92d8e43473c168e24be9f4af504e6ffcc37a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96027970"
---
*Préchauffage du cache*  
Le disque dont la mise en cache de l’hôte est définie en lecture seule peut générer un taux d’E/S par seconde supérieur à sa propre limite. Pour obtenir ces performances de lecture maximales à partir du cache de l’hôte, vous devez tout d’abord préchauffer le cache du disque. Ainsi, les E/S en lecture que l’outil de benchmark génèrera sur le volume CacheReads atteindront le cache plutôt que le disque directement. Le fait d’intervenir au niveau du cache permet de générer des E/S supplémentaires à partir du seul disque ayant une mise en cache.

> [!IMPORTANT]
> Vous devez préchauffer le cache avant d’exécuter l’outil de benchmarking à chaque redémarrage de la machine virtuelle.

## <a name="iometer"></a>Iometer

[Téléchargez l’outil Iometer](http://sourceforge.net/projects/iometer/files/iometer-stable/1.1.0/iometer-1.1.0-win64.x86_64-bin.zip/download) sur la machine virtuelle.

### <a name="test-file"></a>Fichier de test

Iometer utilise un fichier de test stocké sur le volume où est exécuté le test de benchmark. Les lectures et écritures sont activées sur ce fichier de test afin de mesurer le taux d’E/S par seconde et le débit du disque. Iometer crée ce fichier de test si vous n’en avez pas fourni. Créez un fichier de test de 200 Go nommé iobw.tst sur les volumes CacheReads et NoCacheWrites.

### <a name="access-specifications"></a>Spécifications d’accès

Les spécifications (taille d’E/S de la demande, % de lecture-écriture, % aléatoire/séquentiel) sont configurées à l’aide de l’onglet « Access Specifications » d’Iometer. Créez une spécification d’accès pour chacun des scénarios ci-dessous. Créez les spécifications d’accès et enregistrez-les avec un nom approprié, comme RandomWrites\_8K, RandomReads\_8K. Sélectionnez la spécification correspondante lorsque vous exécutez le scénario de test.

Vous trouverez ci-dessous un exemple de spécifications d’accès pour un scénario d’E/S en écriture maximales.  
    ![Exemple de spécifications d’accès pour un taux d’E/S maximal en écriture](../articles/virtual-machines/linux/media/premium-storage-performance/image8.png)

### <a name="maximum-iops-test-specifications"></a>Spécifications de test du taux d’E/S maximal

Pour démontrer le taux maximal d’E/S par seconde, utilisez une taille de demande plus petite. Utilisez une taille de 8 Ko et créez des spécifications pour les lectures et écritures aléatoires.

| Spécification d’accès | Taille de la demande | % aléatoire | % écriture |
| --- | --- | --- | --- |
| RandomWrites\_8K |8 Ko |100 |0 |
| RandomReads\_8K |8 Ko |100 |100 |

### <a name="maximum-throughput-test-specifications"></a>Spécifications de test du débit maximal

Pour afficher le débit maximal, utilisez une plus grande taille de demande. Utilisez une taille de demande de 64 Ko et créez des spécifications pour des lectures et des écritures aléatoires.

| Spécification d’accès | Taille de la demande | % aléatoire | % écriture |
| --- | --- | --- | --- |
| RandomWrites\_64K |64 K |100 |0 |
| RandomReads\_64K |64 K |100 |100 |

### <a name="run-the-iometer-test"></a>Exécuter le test Iometer

Procédez comme suit pour préparer le cache

1. Créez deux spécifications d’accès avec les valeurs indiquées ci-dessous

   | Name | Taille de la demande | % aléatoire | % écriture |
   | --- | --- | --- | --- |
   | RandomWrites\_1MB |1 Mo |100 |0 |
   | RandomReads\_1MB |1 Mo |100 |100 |
1. Exécutez le test Iometer pour initialiser le disque de cache avec les paramètres suivants. Utilisez trois threads de travail pour le volume cible et une profondeur de file d’attente de 128. Définissez la durée « Temps d’exécution » du test sur 2 heures sous l’onglet « Tester la configuration ».

   | Scénario | Volume cible | Name | Duration |
   | --- | --- | --- | --- |
   | Initialisation du disque de cache |CacheReads |RandomWrites\_1MB |2 heures |
1. Exécutez le test Iometer pour préchauffer le disque de cache avec les paramètres suivants. Utilisez trois threads de travail pour le volume cible et une profondeur de file d’attente de 128. Définissez la durée « Temps d’exécution » du test sur 2 heures sous l’onglet « Tester la configuration ».

   | Scénario | Volume cible | Name | Duration |
   | --- | --- | --- | --- |
   | Préchauffage du disque de cache |CacheReads |RandomReads\_1MB |2 heures |

Une fois le disque de cache préchauffé, poursuivez avec les scénarios de test décrits ci-dessous. Pour exécuter le test Iometer, utilisez au moins trois threads de travail pour **chaque** volume cible. Pour chaque thread de travail, sélectionnez le volume cible, définissez une profondeur de file d’attente et sélectionnez l’une des spécifications de test enregistrées, comme illustré dans le tableau ci-dessous, pour exécuter le scénario de test correspondant. Le tableau indique également les résultats attendus pour les E/S par seconde et le débit lors de l’exécution de ces tests. Dans tous les scénarios, une petite taille d’E/S (8 Ko) et une profondeur de file d’attente élevée (128) sont utilisées.

| Scénario de test | Volume cible | Name | Résultats |
| --- | --- | --- | --- |
| Bande passante E/S par seconde en lecture |CacheReads |RandomWrites\_8K |50 000 E/S par seconde |
| Bande passante E/S par seconde en écriture |NoCacheWrites |RandomReads\_8K |64 000 E/S par seconde |
| Bande passante Taux d’E/S par seconde combiné |CacheReads |RandomWrites\_8K |100 000 E/S par seconde |
| NoCacheWrites |RandomReads\_8K | &nbsp; | &nbsp; |
| Bande passante Mo/s en lecture |CacheReads |RandomWrites\_64K |524 Mo/s |
| Bande passante Mo/s en écriture |NoCacheWrites |RandomReads\_64K |524 Mo/s |
| Mo/s combiné |CacheReads |RandomWrites\_64K |1 000 Mo/s |
| NoCacheWrites |RandomReads\_64K | &nbsp; | &nbsp; |

Voici les captures d’écran des résultats du test Iometer pour les scénarios de taux d’E/S par seconde et de débit combinés.

### <a name="combined-reads-and-writes-maximum-iops"></a>Taux d’E/S maximal combiné en lecture et en écriture

![Taux d’E/S maximal combiné en lecture et en écriture](../articles/virtual-machines/linux/media/premium-storage-performance/image9.png)

### <a name="combined-reads-and-writes-maximum-throughput"></a>Débit maximal combiné en lecture et en écriture

![Débit maximal combiné en lecture et en écriture](../articles/virtual-machines/linux/media/premium-storage-performance/image10.png)

## <a name="fio"></a>FIO

FIO est un outil communément utilisé pour tester le stockage sur des machines virtuelles Linux. Cet outil offre la possibilité de sélectionner différentes tailles d’E/S, avec des lectures et des écritures séquentielles ou aléatoires. Il génère des threads de travail ou des processus pour exécuter les opérations d’E/S spécifiées. Vous pouvez spécifier le type d’opérations d’E/S que chaque thread de travail doit exécuter à l’aide de fichiers de travail. Nous avons créé un fichier de travail par scénario, comme illustré dans les exemples ci-dessous. Vous pouvez modifier les spécifications de ces fichiers de travail pour tester différentes charges de travail exécutées sur Premium Storage. Dans ces exemples, nous utilisons une machine virtuelle DS 14 standard exécutée sous **Ubuntu**. Utilisez la configuration décrite au début de la section Benchmarking et préchauffez le cache avant d’exécuter les tests de benchmarking.

Avant de commencer, [téléchargez FIO](https://github.com/axboe/fio) et installez-le sur votre machine virtuelle.

Exécutez la commande ci-dessous pour Ubuntu.

```
apt-get install fio
```

Nous allons utiliser quatre threads de travail pour générer les opérations d’écriture et quatre threads de travail pour générer les opérations de lecture sur les disques. Les Workers d’écriture orientent le trafic sur le volume « nocache », qui comporte 10 disques avec une mise en cache définie sur « Aucun ». Les Workers de lecture orientent le trafic sur le volume « readcache », qui comporte un disque avec une mise en cache définie sur « Lecture seule ».

### <a name="maximum-write-iops"></a>Taux d’E/S maximal en écriture

Créez le fichier de travail avec les spécifications suivantes pour obtenir le taux maximal d’E/S par seconde en écriture. Nommez ce fichier « fiowrite.ini ».

```ini
[global]
size=30g
direct=1
iodepth=256
ioengine=libaio
bs=8k
numjobs=4

[writer1]
rw=randwrite
directory=/mnt/nocache
```

Tenez compte des remarques suivantes qui sont en phase avec les instructions de conception présentées dans les sections précédentes. Ces spécifications sont essentielles pour générer le taux d’E/S par seconde maximal.  

* Une profondeur de file d’attente élevée de 256.  
* Une petite taille de bloc de 8 Ko.  
* Plusieurs threads effectuant des écritures aléatoires.

Exécutez la commande suivante pour lancer le test FIO pendant 30 secondes.  

```
sudo fio --runtime 30 fiowrite.ini
```

Pendant l’exécution du test, le nombre d’E/S par seconde en écriture générées par la machine virtuelle et les disques Premium s’affiche. Comme indiqué dans l’exemple ci-dessous, la machine virtuelle DS14 produit un taux d’E/S par seconde maximal en écriture limité à 50 000 E/S par seconde.  
    ![Nombre d’E/S par seconde en lecture générées par la machine virtuelle et les disques Premium.](../articles/virtual-machines/linux/media/premium-storage-performance/image11.png)

### <a name="maximum-read-iops"></a>Taux d’E/S maximal en lecture

Créez le fichier de travail avec les spécifications suivantes pour obtenir le taux maximal d’E/S par seconde en lecture. Nommez ce fichier « fioread.ini ».

```ini
[global]
size=30g
direct=1
iodepth=256
ioengine=libaio
bs=8k
numjobs=4

[reader1]
rw=randread
directory=/mnt/readcache
```

Tenez compte des remarques suivantes qui sont en phase avec les instructions de conception présentées dans les sections précédentes. Ces spécifications sont essentielles pour générer le taux d’E/S par seconde maximal.

* Une profondeur de file d’attente élevée de 256.  
* Une petite taille de bloc de 8 Ko.  
* Plusieurs threads effectuant des écritures aléatoires.

Exécutez la commande suivante pour lancer le test FIO pendant 30 secondes.

```
sudo fio --runtime 30 fioread.ini
```

Pendant l’exécution du test, le nombre d’E/S par seconde en lecture générées par la machine virtuelle et les disques Premium s’affiche. Comme indiqué dans l’exemple ci-dessous, la machine virtuelle DS14 génère plus de 64 000 E/S par seconde en lecture. Cette valeur représente les performances combinées du disque et du cache.  
    ![Capture d’écran du nombre d’E/S par seconde en lecture générées par la machine virtuelle et les disques Premium.](../articles/virtual-machines/linux/media/premium-storage-performance/image12.png)

### <a name="maximum-read-and-write-iops"></a>Taux d’E/S maximal en lecture et en écriture

 Créez le fichier de travail avec les spécifications suivantes pour obtenir le taux maximal combiné d’E/S par seconde en lecture et en écriture. Nommez ce fichier « fioreadwrite.ini ».

```ini
[global]
size=30g
direct=1
iodepth=128
ioengine=libaio
bs=4k
numjobs=4

[reader1]
rw=randread
directory=/mnt/readcache

[writer1]
rw=randwrite
directory=/mnt/nocache
rate_iops=12500
```

Tenez compte des remarques suivantes qui sont en phase avec les instructions de conception présentées dans les sections précédentes. Ces spécifications sont essentielles pour générer le taux d’E/S par seconde maximal.

* Une profondeur de file d’attente élevée de 128.  
* Une petite taille de bloc de 4 Ko.  
* Plusieurs threads effectuant des opérations d’écriture et de lecture aléatoires.

Exécutez la commande suivante pour lancer le test FIO pendant 30 secondes.

```
sudo fio --runtime 30 fioreadwrite.ini
```

Pendant l’exécution du test, le nombre d’E/S par seconde combinées en lecture et en écriture générées par la machine virtuelle et les disques Premium s’affiche. Comme indiqué dans l’exemple ci-dessous, la machine virtuelle DS14 génère plus de 100 000 E/S par seconde en lecture et en écriture. Cette valeur représente les performances combinées du disque et du cache.  
    ![Taux d’E/S combiné en lecture et en écriture](../articles/virtual-machines/linux/media/premium-storage-performance/image13.png)

### <a name="maximum-combined-throughput"></a>Débit maximal combiné

 Pour obtenir le débit maximal combiné en lecture et en écriture, utilisez une plus grande taille de bloc et une grande profondeur de file d’attente avec plusieurs threads effectuant des opérations de lecture et d’écriture. Vous pouvez utiliser une taille de bloc de 64 Ko et une profondeur de file d’attente de 128.
