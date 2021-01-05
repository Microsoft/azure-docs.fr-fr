---
title: Fichier include
description: Fichier include
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 10/12/2020
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: 086ebf71e2da19a96433f32cfb1bae133e875400
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518056"
---
![Graphique présentant les spécifications Dsv3.](media/vm-disk-performance/dsv3-documentation.jpg)

- Le débit disque maximum *non mis en cache* est la limite maximale de stockage par défaut que la machine virtuelle est en mesure de gérer.
- La limite de débit de stockage *mis en cache* maximale est une limite distincte lorsque vous activez la mise en cache de l’hôte.

La mise en cache de l’hôte rapproche le stockage de la machine virtuelle sur laquelle vous pouvez écrire ou lire rapidement. La quantité de stockage disponible pour la machine virtuelle pour la mise en cache de l’hôte se trouve dans la documentation. Par exemple, Standard_D8s_v3 correspond à 200 Gio de stockage cache.

Vous pouvez activer la mise en cache de l’hôte lorsque vous créez votre machine virtuelle et que vous attachez des disques. Vous pouvez également activer et désactiver la mise en cache de l’hôte de vos disques sur une machine virtuelle existante.

![Capture d’écran montrant la mise en cache de l’hôte.](media/vm-disk-performance/host-caching.jpg)

Vous pouvez ajuster la mise en cache de l’hôte pour correspondre à vos exigences de charge de travail pour chaque disque. Vous pouvez définir la mise en cache de l’hôte :

- **Lecture seule**  : Pour les charges de travail qui effectuent uniquement des opérations de lecture
- **Lecture/écriture** : Pour les charges de travail qui effectuent un équilibre entre opérations de lecture et d’écriture

Si votre charge de travail ne suit pas l’un de ces modèles, il est déconseillé d’utiliser la mise en cache de l’hôte.

Examinons quelques exemples de différents paramètres de mise en cache de l’hôte pour voir comment ils agissent sur le flux de données et les performances. Dans ce premier exemple, nous allons examiner ce qui se passe avec les demandes d’E/S lorsque le paramètre de mise en cache de l’hôte est défini sur **Lecture seule** .

**Configuration**  :

- Standard_D8s_v3
  - E/S par seconde mises en cache : 16 000
  - E/S par seconde non mises en cache : 12 800
- Disque de données P30
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Lecture seule**

Lorsqu’une lecture est effectuée et que les données souhaitées sont disponibles dans le cache, celui-ci renvoie les données demandées. Il n’est pas nécessaire de lire depuis le disque. Cette lecture est comptabilisée dans les limites mises en cache de la machine virtuelle.

![Diagramme montrant une correspondance de lecture avec mise en cache de l’hôte sur lecture.](media/vm-disk-performance/host-caching-read-hit.jpg)

Lorsqu’une lecture est effectuée et que les données souhaitées ne sont *pas* disponibles dans le cache, la demande de lecture est relayée au disque. Ensuite, le disque la met à la disposition à la fois au cache et de la machine virtuelle. Cette lecture est comptabilisée dans la limite non mise en cache ainsi que dans celle mise en cache de la machine virtuelle.

![Diagramme montrant une non-correspondance de lecture avec mise en cache de l’hôte sur lecture.](media/vm-disk-performance/host-caching-read-miss.jpg)

Lorsqu’une écriture est effectuée, elle doit être écrite dans le cache et dans le disque pour être considérée comme terminée. Cette écriture est comptabilisée dans la limite non mise en cache ainsi que dans celle mise en cache de la machine virtuelle.

![Diagramme montrant une écriture avec mise en cache de l’hôte sur lecture.](media/vm-disk-performance/host-caching-write.jpg)

Ensuite, examinons ce qui se passe avec les demandes d’E/S lorsque le paramètre de mise en cache de l’hôte est défini sur **Lecture/écriture** .

**Configuration**  :

- Standard_D8s_v3
  - E/S par seconde mises en cache : 16 000
  - E/S par seconde non mises en cache : 12 800
- Disque de données P30
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Lecture/écriture**

Une lecture est gérée de la même façon qu’une lecture seule. Les écritures sont la seule chose qui est différente pour la mise en cache en lecture/écriture. Lors de l’écriture avec la mise en cache de l’hôte définie sur **Lecture/écriture** , il suffit d’écrire l’écriture dans le cache de l’hôte pour que l’opération soit considérée comme terminée. L’écriture est ensuite écrite tardivement dans le disque dans le cadre d’un processus d’arrière-plan. Cela signifie qu’une écriture est comptabilisée pour les E/S de mise en cache lors de son écriture dans le cache. Lorsqu’elle est écrite tardivement sur le disque, elle est comptabilisée vers les E/S non mises en cache.

![Diagramme montrant une écriture avec mise en cache de l’hôte sur lecture/écriture.](media/vm-disk-performance/host-caching-read-write.jpg)

Poursuivons avec notre machine virtuelle Standard_D8s_v3. La différence est que cette fois, nous allons activer la mise en cache de l’hôte sur les disques. En outre, la limite d’IOPS de la machine virtuelle est désormais de 16 000 IOPS. Trois disques P30 sous-jacents qui peuvent gérer chacun 5 000 IOPS sont attachés à la machine virtuelle.

**Configuration**  :

- Standard_D8s_v3
  - E/S par seconde mises en cache : 16 000
  - E/S par seconde non mises en cache : 12 800
- Disque de système d’exploitation P30
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Lecture/écriture**
- Deux disques de données P30 x 2
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Lecture/écriture**

![Diagramme montrant un exemple de mise en cache de l’hôte.](media/vm-disk-performance/host-caching-example-without-remote.jpg)

L’application utilise une machine virtuelle Standard_D8s_v3 avec mise en cache activée. Elle effectue une demande de 15 000 IOPS. Les demandes sont décomposées en 5 000 IOPS pour chaque disque sous-jacent attaché. Aucune limitation des performances ne se produit.

## <a name="combined-uncached-and-cached-limits"></a>Limites combinées de mise en cache et de non mise en cache

Les limites de mise en cache d’une machine virtuelle sont séparées de ses limites de non-mise en cache. Cela signifie que vous pouvez activer la mise en cache de l’hôte sur les disques attachés à une machine virtuelle tout en désactivant la mise en cache de l’hôte sur d’autres disques. Cette configuration permet à vos machines virtuelles d’obtenir un total d’E/S de stockage égal à la limite mise en cache plus la limite non mise en cache.

Nous allons parcourir un exemple pour vous aider à comprendre comment ces limites fonctionnent ensemble. Nous allons continuer avec la machine virtuelle Standard_D8s_v3 et la configuration avec disques Premium attachés.

**Configuration**  :

- Standard_D8s_v3
  - E/S par seconde mises en cache : 16 000
  - E/S par seconde non mises en cache : 12 800
- Disque de système d’exploitation P30
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Lecture/écriture**
- Deux disques de données P30 x 2
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Lecture/écriture**
- Deux disques de données P30 x 2
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Désactivé**

![Diagramme montrant un exemple de mise en cache de l’hôte avec stockage étendu.](media/vm-disk-performance/host-caching-example-with-remote.jpg)

Dans ce cas, l’application qui s’exécute sur cette machine virtuelle Standard_D8s_v3 est activée pour une demande de 25 000 IOPS. La requête est divisée en 5 000 IOPS sur chacun des disques attachés. Trois disques utilisent la mise en cache de l’hôte et deux disques ne l’utilisent pas.

- Étant donné que les trois disques qui utilisent la mise en cache de l’hôte se trouvent dans les limites de 16 000, ces demandes sont correctement honorées. Aucune limitation des performances de stockage ne se produit.
- Étant donné que les deux disques qui n’utilisent pas la mise en cache de l’hôte se trouvent dans les limites de non-mise en cache de 12 800, ces requêtes sont également effectuées correctement. Aucune limitation ne se produit.

## <a name="disk-performance-metrics"></a>Mesures de performances de disque

Nous disposons de métriques sur Azure qui fournissent des informations sur les performances de vos machines virtuelles et disques. Ces métriques peuvent être visualisées dans le portail Azure. Vous pouvez également les récupérer via un appel d’API. Les métriques sont calculées sur des intervalles d’une minute. Les métriques suivantes sont disponibles pour obtenir des informations sur les E/S des machines virtuelles et des disques ainsi que les performances de débit :

- **Longueur de file d’attente du disque du système d’exploitation** : Le nombre de requêtes d’E/S en attente de lecture ou d’écriture sur le disque du système d’exploitation.
- **Octets lus/s sur disque du système d’exploitation** : Le nombre d’octets lus en une seconde à partir du disque du système d’exploitation.
- **Opérations de lecture/s sur disque du système d’exploitation** : Le nombre d’opérations d’entrée lues en une seconde à partir du disque du système d’exploitation.
- **Octets/s écrits sur disque du système d’exploitation** : Le nombre d’octets écrits en une seconde à partir du disque du système d’exploitation.
- **Opérations d’écriture/s sur disque du système d’exploitation** : Le nombre d’opérations de sortie écrites en une seconde à partir du disque du système d’exploitation.
- **Longueur de file d’attente du disque de données** : Le nombre de requêtes d’E/S en attente de lecture ou d’écriture sur les disques de données.
- **Octets lus/s sur disque de données** : Le nombre d’octets lus en une seconde à partir du disque des disques de données.
- **Opérations de lecture/s sur disque de données** : Le nombre d’opérations d’entrée lues en une seconde à partir des disques de données.
- **Octets écrits/s sur disque de données** : Le nombre d’octets écrits en une seconde à partir du disque des disques de données.
- **Opérations d’écriture/s sur disque de données** : Le nombre d’opérations de sorties écrites en une seconde à partir du disque des disques de données.
- **Disk Read Bytes/Sec** : Le nombre total d’octets lus en une seconde à partir de tous les disques attachés à une machine virtuelle.
- **Disk Read Operations/Sec** : Le nombre d’opérations d’entrée lues en une seconde à partir de tous les disques attachés à une machine virtuelle.
- **Disk Write Bytes/Sec** : Le nombre d’octets écrits en une seconde à partir de tous les disques attachés à une machine virtuelle.
- **Disk Write Operations/Sec** : Le nombre d’opérations de sortie écrites en une seconde à partir de tous les disques attachés à une machine virtuelle.

## <a name="storage-io-utilization-metrics"></a>Métriques d’utilisation des E/S de stockage

Mesures permettant de diagnostiquer les limitations des E/S disque :

- **Pourcentage d’IOPS du disque de données consommées** : Le pourcentage calculé en divisant les opérations d’IOPS de disque de données terminées par les IOPS de disque de données approvisionnées. Si cette valeur est de 100 %, votre application en cours d’exécution sera limitée par les IOPS de votre disque de données.
- **Pourcentage de bande passante du disque de données consommée**  : Le pourcentage calculé par la bande passante consommée par rapport à la bande passante de disque de données approvisionnée. Si cette valeur est de 100 %, votre application en cours d’exécution sera limitée par la bande passante de votre disque de données.
- **Pourcentage d’IOPS du disque de système d’exploitation consommées** : Le pourcentage calculé en divisant les opérations d’IOPS de disque de système d’exploitation terminées par les IOPS de disque du système d’exploitation approvisionnées. Si cette valeur est de 100 %, votre application en cours d’exécution sera limitée par les IOPS de votre disque de système d’exploitation.
- **Pourcentage de bande passante du disque de système d’exploitation consommée**  : Le pourcentage calculé par le débit du disque du système d’exploitation consommé par rapport au débit de disque du système d’exploitation approvisionné. Si cette valeur est de 100 %, votre application en cours d’exécution sera limitée par la bande passante de votre disque de système d’exploitation.

Métriques permettant de diagnostiquer les limitations des E/S de machine virtuelle :

- **Pourcentage d’IOPS en cache de machine virtuelle consommées** : Le pourcentage calculé en divisant le total des opérations d’IOPS terminées par le nombre maximal d’IOPS en cache de la machine virtuelle. Si cette valeur est de 100 %, votre application en cours d’exécution sera limitée par les IOPS en cache de votre machine virtuelle.
- **Pourcentage de bande passante en cache consommée par la machine virtuelle**  : Le pourcentage calculé en divisant la bande passante de disque totale terminée par le débit maximal en cache de la machine virtuelle. Si cette valeur est de 100 %, votre application en cours d’exécution sera limitée par la bande passante en cache de votre machine virtuelle.
- **Pourcentage d’IOPS non mises en cache de machine virtuelle consommées** : Le pourcentage calculé en divisant le total des opérations d’IOPS terminées sur une machine virtuelle par la limite d’IOPS non mises en cache de la machine virtuelle. Si cette valeur est de 100 %, votre application en cours d’exécution sera limitée par les IOPS non mises en cache de votre machine virtuelle.
- **Pourcentage de bande passante non mise en cache consommée par la machine virtuelle**  : Le pourcentage calculé en divisant la bande passante de disque totale terminée sur une machine virtuelle par le débit maximal approvisionné de la machine virtuelle. Si cette valeur est de 100 %, votre application en cours d’exécution sera limitée par la bande passante non mise en cache de votre machine virtuelle.

## <a name="storage-io-utilization-metrics-example"></a>Exemples de métriques d’utilisation des E/S de stockage

Examinons un exemple d’utilisation de ces nouvelles métriques d’utilisation des E/S de stockage pour nous aider à localiser un goulot d’étranglement dans notre système. La configuration système est identique à l’exemple précédent, sauf que cette fois, le disque de système d’exploitation attaché n’est *pas* mis en cache.

**Configuration**  :

- Standard_D8s_v3
  - E/S par seconde mises en cache : 16 000
  - E/S par seconde non mises en cache : 12 800
- Disque de système d’exploitation P30
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Désactivé**
- Deux disques de données P30 x 2
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Lecture/écriture**
- Deux disques de données P30 x 2
  - IOPS : 5 000
  - Mise en cache de l’hôte : **Désactivé**
