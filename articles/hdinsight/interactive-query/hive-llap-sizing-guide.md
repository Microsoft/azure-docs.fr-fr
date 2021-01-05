---
title: Guide de dimensionnement pour le cluster HDInsight Interactive Query (LLAP)
description: Guide de dimensionnement LLAP
ms.service: hdinsight
ms.topic: troubleshooting
author: aniket-ms
ms.author: aadnaik
ms.reviewer: HDI HiveLLAP Team
ms.date: 05/05/2020
ms.openlocfilehash: 57eba9cb6a9350ed66ae132e6a021098da2a4810
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92533051"
---
# <a name="azure-hdinsight-interactive-query-cluster-hive-llap-sizing-guide"></a>Guide de dimensionnement pour le cluster Azure HDInsight Interactive Query (Hive LLAP)

Ce document décrit le dimensionnement du cluster Interactive Query HDInsight (cluster Hive LLAP) pour une charge de travail classique, dans le but d’obtenir des performances raisonnables. Notez que les recommandations fournies dans ce document sont génériques et des indications ainsi que des charges de travail spécifiques peuvent nécessiter un paramétrage spécifique.

### <a name="azure-default-vm-types-for-hdinsight-interactive-query-clusterllap"></a>**Types de machines virtuelles Azure par défaut pour le cluster HDInsight Interactive Query (LLAP)**

| Type de nœud      | Instance | Taille     |
| :---        |    :----:   | :---     |
| Head      | D13 v2       | 8 processeurs virtuels, 56 Go de RAM, disque SSD de 400 Go   |
| Worker   | **D14 v2**        | **16 processeurs virtuels, 112 Go de RAM, disque SSD de 800 Go**       |
| ZooKeeper   | A4 v2        | 4 processeurs virtuels, 8 Go de RAM, disque SSD de 40 Go       |

**_Remarque : toutes les valeurs de configuration recommandées sont basées sur le nœud Worker de type D14 v2_* _  

### <a name="_configuration"></a>_ *Configuration :* *    
| Clé de configuration      | Valeur recommandée  | Description |
| :---        |    :----:   | :---     |
| yarn.nodemanager.resource.memory-mb | 102 400 (Mo) | Mémoire totale donnée, en Mo, pour tous les conteneurs YARN sur un nœud | 
| yarn.scheduler.maximum-allocation-mb | 102 400 (Mo) | Allocation maximale pour chaque requête de conteneur au gestionnaire des ressources, en Mo. Les requêtes de mémoire de valeur supérieure à celle-ci ne sont pas prises en compte |
| yarn.scheduler.maximum-allocation-vcores | 12 |Nombre maximal de cœurs de processeur pour chaque requête de conteneur adressée au gestionnaire des ressources. Les requêtes de mémoire de valeur supérieure à celle-ci ne sont pas prises en compte. |
| yarn.nodemanager.resource.cpu-vcores | 12 | Nombre de cœurs de processeur par NodeManager pouvant être alloués aux conteneurs. |
| yarn.scheduler.capacity.root.llap.capacity | 80 (%) | Allocation de la capacité YARN pour la file d’attente llap  |
| tez.am.resource.memory.mb | 4 096 (Mo) | Quantité de mémoire (en Mo) que doit utiliser le AppMaster Tez |
| hive.server2.tez.sessions.per.default.queue | <number_of_worker_nodes> |Nombre de sessions pour chaque file d’attente nommée dans hive.server2.tez.default.queues. Ce nombre correspond au nombre de coordinateurs de requêtes (AppMaster Tez) |
| hive.tez.container.size | 4 096 (Mo) | Taille du conteneur Tez spécifiée en Mo |
| hive.llap.daemon.num.executors | 12 | Nombre d’exécuteurs par démon LLAP | 
| hive.llap.io.threadpool.size | 12 | Taille du pool de threads pour les exécuteurs |
| hive.llap.daemon.yarn.container.mb | 77 824 (Mo) | Mémoire totale, en Mo, utilisée par les démons LLAP individuels (quantité de mémoire par démon)
| hive.llap.io.memory.size | 235 520 (Mo) | Taille, en Mo, du cache affecté à chaque démon LLAP, si le cache SSD est activé |
| hive.auto.convert.join.noconditionaltask.size | 2 048 (Mo) | Taille de la mémoire pour la jointure de mappage, en Mo |

### <a name="llap-daemon-size-estimations"></a>**Estimation des tailles de démons LLAP :**  

#### <a name="1-determining-total-yarn-memory-allocation-for-all-containers-on-a-node"></a>**1. Détermination de l’allocation de mémoire YARN totale pour tous les conteneurs sur un nœud**    
Configuration : * *_yarn.nodemanager.resource.memory-mb_* _  

Cette valeur indique la quantité maximale de mémoire pouvant être utilisée par les conteneurs YARN sur chaque nœud, en Mo. La valeur spécifiée doit être inférieure à la quantité totale de mémoire physique sur ce nœud.   
Mémoire totale pour tous les conteneurs YARN sur un nœud = [mémoire physique totale] - [mémoire pour le système d’exploitation + autres services]  
Définissez cette valeur sur environ 90 % de l’espace de mémoire RAM disponible.  
Pour D14 v2, la valeur recommandée est _*102400 MB**. 

#### <a name="2-determining-maximum-amount-of-memory-per-yarn-container-request"></a>**2. Détermination de la quantité maximale de mémoire par requête de conteneur YARN**  
Configuration : * *_yarn.scheduler.maximum-allocation-mb_* _

Cette valeur indique l’allocation maximale pour chaque requête de conteneur au gestionnaire des ressources, en Mo. Les requêtes de mémoire de taille supérieure à celle-ci ne sont pas prises en compte. Le gestionnaire des ressources peut fournir de la mémoire aux conteneurs par incréments de _yarn.scheduler.minimum-allocation-mb* et ne peut pas dépasser la taille spécifiée par *yarn.scheduler.maximum-allocation-mb*. La valeur spécifiée ne doit pas être supérieure à la mémoire totale donnée pour tous les conteneurs sur le nœud spécifiée par *yarn.nodemanager.resource.memory-mb*.    
Pour les nœuds Worker D14 v2, la valeur recommandée est de **102 400 Mo**

#### <a name="3-determining-maximum-amount-of-vcores-per-yarn-container-request"></a>**3. Détermination de la quantité maximale de cœurs virtuels par requête de conteneur YARN**  
Configuration : * *_yarn.scheduler.maximum-allocation-vcores_* _  

Cette valeur indique le nombre maximal de cœurs de processeur virtuel pour chaque requête de conteneur adressée au gestionnaire des ressources. La demande d’un nombre de cœurs virtuels supérieur à cette valeur n’est pas prise en compte. Il s’agit d’une propriété globale du planificateur YARN. Pour le conteneur du démon LLAP, cette valeur peut être définie sur 75 % du nombre total de cœurs virtuels disponibles. Les 25 % restants doivent être réservés à NodeManager, DataNode et d’autres services s’exécutant sur les nœuds Worker.  
Il existe 16 cœurs virtuels sur les machines virtuelles D14 v2, 75 % du total de ces 16 cœurs virtuels peuvent être utilisés par le conteneur du démon LLAP.  
Pour D14 v2, la valeur recommandée est _*12**.  

#### <a name="4-number-of-concurrent-queries"></a>**4. Nombre de requêtes simultanées**  
Configuration : * *_hive.server2.tez.sessions.per.default.queue_* _

Cette valeur de configuration détermine le nombre de sessions Tez pouvant être lancées en parallèle. Ces sessions Tez seront lancées pour chacune des files d’attente spécifiées par « hive.server2.tez.default.queues ». Cela correspond au nombre d’AM Tez (coordinateurs de requêtes). Nous vous recommandons d’utiliser le même nombre de nœuds Worker. Le nombre d’AM Tez peut être supérieur au nombre de nœuds du démon LLAP. La principale responsabilité des AM Tez consiste à coordonner l’exécution de la requête et à affecter des fragments de plan de requête aux démons LLAP correspondants, à des fins d’exécution. Conservez cette valeur comme multiple du nombre de nœuds de démon LLAP pour obtenir un débit plus élevé.  

Le cluster HDInsight par défaut comporte quatre démons LLAP exécutés sur quatre nœuds Worker ; la valeur recommandée est donc _*4**.  

#### <a name="5-tez-container-and-tez-application-master-size"></a>**5. Taille du AppMaster Tez et du conteneur Tez**    
Configuration : * *_tez.am.resource.memory.mb, hive.tez.container.size_* _  

_tez.am.resource.memory.mb* : définit la taille du AppMaster Tez  
La valeur recommandée est de **4 096 Mo**.
   
*hive.tez.container.size* : définit la quantité de mémoire donnée pour le conteneur Tez. Cette valeur doit être comprise entre la taille minimale du conteneur YARN ( *yarn.scheduler.minimum-allocation-mb* ) et la taille maximale du conteneur YARN ( *yarn.scheduler.maximum-allocation-mb* ). Les exécuteurs du démon LLAP utilisent cette valeur pour limiter l’utilisation de la mémoire par exécuteur.  
La valeur recommandée est de **4 096 Mo**.  

#### <a name="6-llap-queue-capacity-allocation"></a>**6. Allocation de la capacité de la file d’attente LLAP**   
Configuration : * *_yarn.scheduler.capacity.root.llap.capacity_* _  

Cette valeur indique un pourcentage de capacité octroyé à la file d’attente llap. Les allocations de capacité peuvent avoir des valeurs différentes pour les différentes charges de travail en fonction de la configuration des files d’attente YARN. Si votre charge de travail est une opération en lecture seule, la définir sur une valeur allant jusqu’à 90 % de la capacité doit fonctionner. Toutefois, si votre charge de travail combine des opérations de mise à jour/suppression/fusion à l’aide de tables managées, il est recommandé de définir la capacité de la file d’attente llap sur 80 %. Les 20 % de capacité restante peuvent être utilisés par d’autres tâches, par exemple le compactage, pour allouer des conteneurs à partir de la file d’attente par défaut. Ainsi, les tâches de la file d’attente par défaut ne priveront pas les ressources YARN.    
Pour les nœuds Worker D14v2, la valeur recommandée pour la file d'attente llap est _*80**.   
(Pour les charges de travail en lecture seule, elle peut être augmentée jusqu’à 90, comme il convient.)  

#### <a name="7-llap-daemon-container-size"></a>**7. Taille du conteneur du démon LLAP**    
Configuration : * *_hive.llap.daemon.yarn.container.mb_* _  
   
Le démon LLAP est exécuté en tant que conteneur YARN sur chaque nœud Worker. La taille totale de la mémoire pour le conteneur du démon LLAP dépend des facteurs suivants,    
_  Configurations de la taille du conteneur YARN (yarn.scheduler.minimum-allocation-mb, yarn.scheduler.maximum-allocation-mb, yarn.nodemanager.resource.memory-mb)
*  Nombre de AM Tez sur un nœud
*  Mémoire totale configurée pour tous les conteneurs sur un nœud et capacité de la file d’attente LLAP  

La mémoire requise par les AppMaster Tez (AM Tez) peut être calculée comme suit.  
Pour le cluster HDInsight Interactive, par défaut, un nœud AM Tez par Worker joue le rôle de coordinateur de requêtes. Le nombre d’AM Tez peut être configuré en fonction d’un certain nombre de requêtes simultanées à servir.
Il est recommandé d’avoir 4 Go de mémoire par AM Tez.

Mémoire d’AM Tez par nœud = [ceil(nombre d’AM Tez/nombre de nœuds de démon LLAP)] x [Taille du conteneur d’AM Tez]  
Pour D14 v2, la configuration par défaut comprend quatre nœuds AM Tez et quatre nœuds LLAP daemon.  
Mémoire d’AM Tez par nœud = (ceil(4/4) x 4 Go) = 4 Go

La mémoire totale disponible pour la file d’attente LLAP par nœud Worker peut être calculée comme suit : Cette valeur dépend de la quantité totale de mémoire disponible pour tous les conteneurs YARN sur un nœud ( *yarn.nodemanager.resource.memory-mb* ) et du pourcentage de capacité configuré pour la file d’attente llap ( *yarn.scheduler.capacity.root.llap.capacity* ).  
Mémoire totale pour la file d’attente LLAP sur le nœud Worker = Mémoire totale disponible pour tous les conteneurs YARN sur un nœud x Pourcentage de la capacité de la file d’attente LLAP.  
Pour D14 v2, cette valeur est de [100 Go x 0,80] = 80 Go.

La taille du conteneur de démon LLAP est calculée comme suit :

**Taille du conteneur de démon LLAP = [mémoire totale disponible pour la file d’attente LLAP] – [mémoire d’AM Tez par nœud]**  
    
Pour les nœuds Worker D14 v2, HDI 4.0 : la valeur recommandée est de (80 Go - 4 Go)) = **76 Go**   
(Pour HDI 3.6, la valeur recommandée est de **74 Go** , car vous devez réserver environ 2 Go supplémentaires pour l’AM Slider.)  

#### <a name="8-determining-number-of-executors-per-llap-daemon"></a>**8. Détermination du nombre d’exécuteurs par démon LLAP**  
Configuration : * *_hive.llap.daemon.num.executors_* _, _*_hive.llap.io.threadpool.size_*_

_*_hive.llap.daemon.num.executors_*_  :   
Cette configuration contrôle le nombre d’exécuteurs qui peuvent lancer des tâches en parallèle pour chaque démon LLAP. Cette valeur dépend du nombre de cœurs virtuels, de la quantité de mémoire donnée par exécuteur et de la quantité totale de mémoire disponible pour le démon LLAP. En règle générale, nous souhaitons que cette valeur soit aussi proche que possible du nombre de cœurs virtuels.
Les machines virtuelles D14 v2 comportent 16 cœurs virtuels. Toutefois, tous les cœurs virtuels ne peuvent pas être pris, car d’autres services tels que NodeManager, DataNode, la fonction de surveillance des métriques, etc. nécessitent également une partie des cœurs virtuels disponibles. 

Si vous devez ajuster le nombre d'exécuteurs, il est recommandé de prendre en compte 4 Go de mémoire par exécuteur, comme spécifié par _hive.tez.container.size*, et de vérifier que la mémoire totale nécessaire pour tous les exécuteurs ne dépasse pas la quantité totale de mémoire disponible pour le conteneur du démon LLAP.  
Cette valeur peut être définie sur un maximum de 75 % du nombre total de cœurs virtuels disponibles sur ce nœud.  
Pour D14 v2, la valeur recommandée est (0,75 x 16) = **12**

**_hive.llap.io.threadpool.size_ *_ : Cette valeur spécifie la taille du pool de threads pour les exécuteurs. Étant donné que les exécuteurs sont définis de manière fixe comme spécifié, leur nombre est égal à celui des exécuteurs par démon LLAP. Pour D14 v2, la valeur recommandée est _* 12**.

#### <a name="9-determining-llap-daemon-cache-size"></a>**9. Détermination de la taille du cache du démon LLAP**  
Configuration : * *_hive.llap.io.memory.size_* _

La mémoire du conteneur du démon LLAP est constituée des composants suivants : _  Head room
*  La mémoire de tas utilisée par les exécuteurs (Xmx)
*  Le cache en mémoire par démon (sa taille de mémoire hors tas, non applicable lorsque le cache SSD est activé)
*  La taille des métadonnées du cache en mémoire (applicable uniquement lorsque le cache SSD est activé)

**Taille de la marge** : Cette taille indique une partie de la mémoire hors tas utilisée pour la surcharge des machines virtuelles Java (espace de mémoire, pile de threads, structures de données GC, etc.). En règle générale, cette surcharge est d’environ 6 % de la taille du tas (Xmx). Par prudence, cette valeur peut être calculée comme correspondant à 6 % de la taille de la mémoire totale du démon LLAP.  
Pour D14 v2, la valeur recommandée est : ceil(76 Go x 0,06) ~= **5 Go**.  

**Taille du tas (Xmx)** : Il s’agit de la quantité de mémoire de tas disponible pour tous les exécuteurs.
Taille totale du tas = nombre d’exécuteurs x 4 Go  
Pour D14 v2, cette valeur est 12 x 4 Go = **48 Go**  

Si le cache SSD est désactivé, le cache en mémoire est la quantité de mémoire restante après avoir retiré la taille de la marge et la taille du tas de la taille du conteneur du démon LLAP.

Le calcul de la taille du cache diffère lorsque le cache SSD est activé.
Définir *hive.llap.io.allocator.mmap* = true active la mise en cache SSD.
Lorsque le cache SSD est activé, une partie de la mémoire est utilisée pour stocker les métadonnées du cache SSD. Les métadonnées sont stockées en mémoire et doivent représenter environ 10 % de la taille du cache SSD.   
Taille des métadonnées en mémoire du cache SSD = [taille du conteneur du démon LLAP] - [marge + taille du tas]  
Pour D14 v2, avec HDI 4.0, la taille des métadonnées en mémoire du cache SSD = [76 Go] - [5 Go + 48 Go] = **23 Go**  
Pour D14 v2, avec HDI 3.6, la taille des métadonnées en mémoire du cache SSD = [76 Go] - [5 Go + 48 Go + 2 Go Slider] = **21 Go**

Compte tenu de la taille de la mémoire disponible pour le stockage des métadonnées du cache SSD, nous pouvons calculer la taille du cache SSD pouvant être prise en charge.  
Taille des métadonnées en mémoire pour le cache SSD = 10 % de la taille du cache SSD       
Taille du cache SSD = taille des métadonnées en mémoire pour le cache SSD x 10  

Pour D14 v2 et HDI 4.0, la taille de cache SSD recommandée est de 23 Go x 10 = **230 Go**  
Pour D14 v2 et HDI 3.6, la taille de cache SSD recommandée est de 21 Go x 10 = **210 Go**

#### <a name="10-adjusting-map-join-memory"></a>**10. Réglage de la mémoire de jointure de mappage**   
Configuration : * *_hive.auto.convert.join.noconditionaltask.size_* _

Vérifiez que _hive.auto.convert.join.noconditionaltask* est activé pour que ce paramètre prenne effet.
Cette configuration permet à l’utilisateur de spécifier la taille des tables qui peuvent tenir dans la mémoire pour effectuer une jointure de mappage. Si la taille totale des tables et partitions n-1 pour la jointure à N voies est inférieure à la valeur configurée, la jointure de mappage est choisie. La taille de la mémoire de l’exécuteur LLAP doit être utilisée pour calculer le seuil de conversion automatique en jointure de mappage.
Chaque exécuteur est supposé avoir 4 Go de taille de tas, mais tous ne sont pas disponibles pour la jointure de mappage. Une partie de la mémoire de tas sera utilisée pour les tampons de tri, les tampons de lecture aléatoire, les tables de hachage, etc., par d’autres opérations. Par conséquent, vous pouvez fournir 50 % de la mémoire du tas de 4 Go pour la jointure de mappage.  
Remarque : Cette valeur peut nécessiter des ajustements adaptés à votre charge de travail. Définir une valeur trop faible peut causer une non utilisation de la fonctionnalité de conversion automatique. Et définir une valeur trop élevée peut entraîner des exceptions de mémoire insuffisante ou des interruptions du GC qui peuvent entraîner des performances indésirables.  
Pour D14 v2, avec 4 Go de mémoire par exécuteur, il est recommandé de définir cette valeur sur **2 048 Mo**.


#### <a name="next-steps"></a>**Next Steps**
Si la définition de ces valeurs n’a pas résolu votre problème, envisagez l’une des solutions suivantes...

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous avec [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client en connectant la communauté Azure aux ressources appropriées (réponses, support et experts).

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support**. Pour en savoir plus, voir [Création d’une requête de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).  

* ##### <a name="other-references"></a>**Autres références :**
  * [Configurer d’autres propriétés LLAP](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/performance-tuning/content/hive_setup_llap.html)  
  * [Configurer la taille du tas du serveur Hive](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/performance-tuning/content/hive_hiveserver_heap_sizing.html)  
  * [Dimensionnement de la mémoire pour la jointure de mappage pour LLAP](https://community.cloudera.com/t5/Community-Articles/Map-Join-Memory-Sizing-For-LLAP/ta-p/247462)  
  * [Propriétés du moteur d’exécution Tez](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/performance-tuning/content/hive_tez_engine_properties.html)  
  * [Présentation approfondie de Hive LLAP](https://community.cloudera.com/t5/Community-Articles/Hive-LLAP-deep-dive/ta-p/248893)