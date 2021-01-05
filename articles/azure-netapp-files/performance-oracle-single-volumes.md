---
title: Performances des bases de données Oracle sur un volume unique Azure NetApp Files | Microsoft Docs
description: Décrit les résultats des tests de performance d’un volume Azure NetApp Files unique sur la base de données Oracle.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/30/2020
ms.author: b-juche
ms.openlocfilehash: c6cdf2f6dada0aa4dea2f70f18237b7ee39e3ea1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91571185"
---
# <a name="oracle-database-performance-on-azure-netapp-files-single-volumes"></a>Performances des bases de données Oracle sur des volumes uniques Azure NetApp Files

Cet article aborde les sujets suivants relatifs à Oracle dans le cloud. Ces sujets peuvent présenter un intérêt particulier pour un administrateur de base de données, un architecte cloud ou un architecte de stockage :   

* Quand vous dirigez une charge de travail de traitement transactionnel en ligne (OLTP) (principalement des E/S aléatoires) ou une charge de travail de traitement analytique en ligne (OLAP) (principalement des E/S séquentielles), à quoi ressemblent les performances ?   
* Quelle est la différence au niveau des performances entre le client Linux NFS en mode noyau (kNFS) standard et le client NFS direct d’Oracle ?
* En ce qui concerne la bande passante, les performances d’un volume Azure NetApp Files unique sont-elles suffisantes ?

## <a name="testing-environment-and-components"></a>Environnement et composants de test

Le schéma ci-dessous illustre l’environnement utilisé pour les tests. Pour rester simples et cohérents, les guides opérationnels Ansible ont été utilisés pour déployer tous les éléments du banc d’essai.

![Environnement de test Oracle](../media/azure-netapp-files/performance-oracle-test-environment.png)  

### <a name="virtual-machine-configuration"></a>Configuration de la machine virtuelle

Pour les tests, la configuration suivante a été utilisée pour la machine virtuelle :
* Système d’exploitation :   
    Red Hat Enterprise Linux 7.8 (wle-ora01)
* Types d’instances :   
    Deux modèles ont été utilisés pour les tests : D32s_v3 et D64s_v3
* Nombre d’interfaces réseau :   
    Une (1) placée dans le sous-réseau 3  
* Disques :   
    Les fichiers binaires Oracle et le système d’exploitation ont été placés sur un seul disque Premium.

### <a name="azure-netapp-files-configuration"></a><a name="anf_config"></a>Configuration d’Azure NetApp Files
Pour les tests, la configuration suivante d’Azure NetApp Files a été utilisée :   

* Taille du pool de capacités :  
    Différentes tailles du pool ont été configurées : 4 Tio, 8 Tio, 16 Tio, 32 Tio 
* Niveau de service :  
    Ultra (128 Mio/s de bande passante pour 1 Tio de capacité de volume allouée)
* Volumes :  
    Des tests à un et deux volumes ont été évalués.

### <a name="workload-generator"></a>Générateur de charge de travail 

Le générateur de charge de travail SLOB 2.5.4.2 a été utilisé pour les tests. Le générateur de charge de travail SLOB (Silly Little Oracle Benchmark) est bien connu dans l’espace Oracle conçu pour contraindre et tester le sous-système d’E/S avec une charge de travail d’E/S physique dans la mémoire tampon de la zone SGA.  

Le générateur de charge de travail SLOB 2.5.4.2 ne prend pas en charge la base de données enfichable (PDB). Par conséquent, une modification a été ajoutée dans les scripts `setup.sh` et `runit.sh` pour y ajouter la prise en charge PDB.  

Les variables SLOB utilisées dans les tests sont décrites dans les sections suivantes.

#### <a name="workload-80-select-20-update--random-io--slobconf-variables"></a>Charge de travail 80 % SELECT, 20 % UPDATE | E/S aléatoires – variables de `slob.conf`   

`UPDATE_PCT=20`   
`SCAN_PCT=0`   
`RUN_TIME=600`   
`WORK_LOOP=0`   
`SCALE=75G`   
`SCAN_TABLE_SZ=50G`   
`WORK_UNIT=64`   
`REDO_STRESS=LITE`   
`LOAD_PARALLEL_DEGREE=12`   

#### <a name="workload-100-select--sequential-io--slobconf-variables"></a>Charge de travail 100 % SELECT | E/S séquentielles – variables de `slob.conf`

`UPDATE_PCT=0`   
`SCAN_PCT=100`   
`RUN_TIME=600`   
`WORK_LOOP=0`   
`SCALE=75G`   
`SCAN_TABLE_SZ=50G`   
`WORK_UNIT=64`   
`REDO_STRESS=LITE`   
`LOAD_PARALLEL_DEGREE=12`   

### <a name="database"></a>Base de données

La version Oracle utilisée pour les tests est Oracle Database Enterprise Edition 19.3.0.0.

Les paramètres Oracle sont les suivants :  
* `sga_max_size` : 4096M
* `sga_target` : 4096
* `db_writer_processes` : 12
* `awr_pdb_autoflush_enabled` : true
* `filesystemio_options` : SETALL
* `log_buffer` : 134217728

Un fichier PDB a été créé pour la base de données SLOB.

Le diagramme ci-dessous montre l’espace disque logique nommé PERFIO d’une taille de 600 Go (20 fichiers de données de 30 Go chacun) créé pour héberger quatre schémas utilisateur SLOB. Chaque schéma utilisateur a une taille de 125 Go.

![Base de données Oracle](../media/azure-netapp-files/performance-oracle-tablespace.png)  

## <a name="performance-metrics"></a>Mesures de performances

L’objectif était de signaler les performances d’E/S telles qu’elles ont été rencontrées par l’application. Par conséquent, tous les diagrammes de cet article utilisent les métriques signalées par la base de données Oracle via ses rapports AWR (Automatic Workload Repository). Les métriques utilisées dans les diagrammes sont les suivantes :   

* **Nombre moyen de demandes d’E/S par seconde**   
    Correspond à la somme du nombre moyen de demandes d’E/S de lecture par seconde et du nombre moyen de demandes d’E/S d’écriture par seconde de la section relative au profil de charge
* **Nombre moyen d’E/S en Mo/s**   
    Correspond à la somme du nombre moyen d’E/S de lecture en Mo/s et du nombre moyen d’E/S d’écriture en Mo/s de la section relative au profil de charge
* **Latence de lecture moyenne**   
    Correspond à la latence moyenne de l’événement d’attente Oracle « lecture séquentielle du fichier de base de données » en microsecondes
* **Nombre de threads/schéma**   
    Correspond au nombre de threads SLOB par schéma utilisateur

## <a name="performance-measurement-results"></a>Résultats de mesure des performances  

Cette section décrit les résultats de la mesure des performances.

### <a name="linux-knfs-client-vs-oracle-direct-nfs"></a>Client Linux kNFS et Oracle Direct NFS

Ce scénario s’exécutait sur une machine virtuelle Azure Standard_D32s_v3 (Intel E5-2673 v4 cadencé à 2,30 GHz). La charge de travail est 75 % SELECT et 25 % UPDATE, principalement des E/S aléatoires et avec un taux d’accès au tampon de base de données d’environ 7,5 %. 

Comme l’indique le diagramme suivant, le client Oracle DNFS a fourni un débit jusqu’à 2,8 fois supérieur à celui fourni par le client Linux kNFS standard :  

![Client Linux kNFS comparé à Oracle Direct NFS](../media/azure-netapp-files/performance-oracle-kfns-compared-dnfs.png)  

Le diagramme suivant montre la courbe de latence pour les opérations de lecture. Dans ce contexte, le goulot d’étranglement du client kNFS correspond à la connexion de socket TCP NFS unique établie entre le client et le serveur NFS (le volume Azure NetApp Files).  

![Client Linux kNFS comparé à Oracle Direct NFS – Courbe de latence](../media/azure-netapp-files/performance-oracle-latency-curve.png)  

Le client DNFS a pu envoyer (push) plus de demandes d’E/S par seconde en raison de sa capacité à créer des centaines de connexions de socket TCP, tirant ainsi parti du parallélisme. Comme décrit dans [Configuration d’Azure NetApp Files](#anf_config), chaque Tio supplémentaire de capacité alloué permet de profiter de 128 Mio/s supplémentaires de bande passante. DNFS a atteint un maximum de 1 Gio/s de débit, ce qui correspond à la limite imposée par le choix d’une capacité de 8 Tio. Avec une capacité supérieure, un débit plus élevé aurait été obtenu.

Le débit n’est qu’un des points à prendre en compte. Un autre élément à prendre en compte est la latence, qui a l’impact principal sur l’expérience utilisateur. Comme le montre le diagramme suivant, des augmentations de la latence peuvent être attendues beaucoup plus rapidement avec kNFS qu’avec DNFS. 

![Client Linux kNFS comparé à Oracle Direct NFS – Latence de lecture](../media/azure-netapp-files/performance-oracle-read-latency.png)  

Les histogrammes fournissent un excellent insight concernant les latences de base de données. Le diagramme suivant fournit une vue complète du point de vue de la « lecture séquentielle du fichier de base de données » enregistrée, tout en utilisant DNFS au point de données de la simultanéité la plus élevée (32 threads/schéma). Comme indiqué dans le diagramme suivant, 47 % de toutes les opérations de lecture ont été honorées entre 512 microsecondes et 1000 microsecondes, tandis que 90 % de l’ensemble des opérations de lecture ont été traitées à une latence inférieure à 2 ms.

![Client Linux kNFS comparé à Oracle Direct NFS – Histogrammes](../media/azure-netapp-files/performance-oracle-histogram-read-latency.png)  

En conclusion, il est clair que DNFS est incontournable pour ce qui est d’améliorer les performances d’une instance de base de données Oracle sur NFS.

### <a name="single-volume-performance-limits"></a>Limites de performances d’un volume unique

Cette section décrit les limites de performances d’un volume unique avec des E/S aléatoires et des E/S séquentielles. 

#### <a name="random-io"></a>E/S aléatoires

DNFS est en mesure de consommer une bande passante bien supérieure à celle fournie par un quota de performances Azure NetApp Files de 8 To. En augmentant la capacité du volume Azure NetApp Files à 16 Tio, ce qui constitue un changement instantané, la bande passante du volume a doublé, de 1 024 Mio/s à 2 048 Mio/s. 

Le diagramme suivant montre une configuration pour une charge de travail de 80 % SELECT et 20 % UPDATE, et avec un taux d’accès au tampon de base de données de 8 %. Le générateur de charge de travail SLOB a pu conduire un volume unique à 200 000 demandes d’E/S NFS par seconde. En considérant que chaque opération a une taille de 8 Kio, le système testé a pu fournir environ 200 000 demandes d’E/S par seconde ou 1 600 Mio/s.
 
![Débit d’Oracle DNFS](../media/azure-netapp-files/performance-oracle-dnfs-throughput.png)  

Le diagramme de courbe de latence de lecture suivant montre que, quand le débit de lecture augmente, la latence augmente progressivement sous la barre de 1 ms et qu’elle atteint le coude de la courbe à un nombre moyen de demandes d’E/S de lecture par seconde d’environ 165 000 à la latence de lecture moyenne d’environ 1,3 ms.  Il s’agit là d’une valeur de latence incroyable pour un taux d’E/S inaccessible avec quasiment toute autre technologie dans le cloud Azure. 

![Courbe de latence d’Oracle DNFS](../media/azure-netapp-files/performance-oracle-dnfs-latency-curve.png)  

#### <a name="sequential-io"></a>E/S séquentielles  

Comme le diagramme suivant l’indique, les E/S ne sont pas toutes aléatoires par nature, en considérant une sauvegarde RMAN ou une analyse de table complète, par exemple, comme des charges de travail nécessitant autant de bande passante qu’elles peuvent en obtenir.  Avec la même configuration que celle décrite précédemment, mais en redimensionnant le volume à 32 Tio, le schéma suivant montre qu’une seule instance d’Oracle DB peut dépasser un débit de 3 900 Mo/s, très proche du quota de performances de 32 To (128 Mo/s * 32 = 4096 Mo/s) du volume Azure NetApp Files.

![E/S d’Oracle DNFS](../media/azure-netapp-files/performance-oracle-dnfs-io.png)  

En résumé, Azure NetApp Files vous aide à migrer vos bases de données Oracle dans le cloud. Il fournit les performances requises quand la base de données l’exige. Vous pouvez redimensionner votre quota de volume à tout moment, de façon dynamique et progressive.

## <a name="next-steps"></a>Étapes suivantes

- [Recommandations sur les tests de performances pour Azure NetApp Files](azure-netapp-files-performance-metrics-volumes.md)
- [Test d’évaluation des performances pour Linux](performance-benchmarks-linux.md)