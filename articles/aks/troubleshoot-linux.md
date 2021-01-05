---
title: Outils de performance Linux
titleSuffix: Azure Kubernetes Service
description: Découvrez comment utiliser les outils d'analyse des performances Linux résoudre les problèmes courants liés à l’utilisation d’AKS (Azure Kubernetes Service).
services: container-service
author: alexeldeib
ms.service: container-service
ms.topic: troubleshooting
ms.date: 02/10/2020
ms.author: aleldeib
ms.openlocfilehash: 74f65780594c7bc938ed6d59437473c4363e5848
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90982040"
---
# <a name="linux-performance-troubleshooting"></a>Résolution des problèmes de performances Linux

Sur les machines Linux, l’épuisement des ressources est un problème courant qui peut se manifester au travers d'un grand nombre de symptômes. Ce document fournit une vue d’ensemble des outils disponibles pour vous aider à diagnostiquer de tels problèmes.

Bon nombre de ces outils acceptent un intervalle de génération de sortie en continu. Ce format de sortie facilite considérablement la détection des modèles. Lorsqu'il est accepté, l’exemple d’appel inclura `[interval]`.

Un grand nombre de ces outils s'accompagnent d'un historique complet et d'un large éventail d’options de configuration. Cette page fournit uniquement un sous-ensemble simple d’appels pour mettre en évidence les problèmes courants. La source d’informations canonique correspond toujours à la documentation de référence pour chaque outil spécifique. Cette documentation s'avérera bien plus complète que celle fournie ici.

## <a name="guidance"></a>Assistance

Soyez méthodique lorsque vous examinez des problèmes de performances. Les deux approches courantes sont USE (utilization, saturation, errors - utilisation, saturation, erreurs) et RED (rate, errors, duration - taux, erreurs, durée). L'approche RED est généralement utilisée dans le contexte des services pour la surveillance basée sur les requêtes. L'approche USE est généralement utilisée pour la surveillance des ressources : pour chaque ressource d’une machine, surveillance de l’utilisation, de la saturation et des erreurs. Les quatre principaux types de ressources d'une machine sont l'UC, la mémoire, le disque et le réseau. Une utilisation, une saturation ou un taux d’erreurs élevés pour l’une de ces ressources indique un possible problème lié au système. En cas de problème, examinez la cause racine : pourquoi la latence d’E/S disque est-elle élevée ? Les disques ou la référence SKU de machine virtuelle sont-ils limités ? Quels processus écrivent sur les appareils, et dans quels fichiers ?

Voici quelques exemples de problèmes courants et d’indicateurs pour les diagnostiquer :
- Limitation d'IOPS : utilisez iostat pour mesurer les IOPS par appareil. Assurez-vous qu’aucun disque individuel ne dépasse sa limite et que la somme de tous les disques est inférieure à la limite de la machine virtuelle.
- Limitation de bande passante : utilisez iostat comme pour les IOPS, mais en mesurant le débit de lecture/écriture. Assurez-vous que le débit par appareil et le débit agrégé sont inférieurs aux limites de bande passante.
- Épuisement des ressources SNAT : cela peut se manifester par des connexions actives (sortantes) élevées dans SAR. 
- Perte de paquets : cela peut être mesuré par le proxy via le nombre de retransmissions TCP par rapport au nombre d’éléments envoyés/reçus. `sar` et `netstat` peuvent afficher ces informations.

## <a name="general"></a>Général

Ces outils sont à usage général et couvrent les informations système de base. Ils constituent un bon point de départ pour un examen approfondi.

### <a name="uptime"></a>uptime

```
$ uptime
 19:32:33 up 17 days, 12:36,  0 users,  load average: 0.21, 0.77, 0.69
```

uptime fournit une durée de fonctionnement du système et des moyennes de charge de 1, 5 et 15 minutes. Ces moyennes de charge correspondent approximativement aux threads en cours ou en attente de fin d'un travail. Dans l'absolu, ces nombres peuvent être difficiles à interpréter, mais mesurés au fil du temps, ils peuvent révéler de précieuses informations :

- Moyenne de 1 minute > moyenne de 5 minutes signifie que la charge augmente.
- Moyenne de 1 minute < moyenne de 5 minutes signifie que la charge diminue.

uptime peut également indiquer pourquoi des informations ne sont pas disponibles : le problème peut s'être résolu de lui-même ou moyennant un redémarrage avant que l’utilisateur n’ait accédé à la machine.

Des moyennes de charge supérieures au nombre de threads d'UC disponibles peuvent indiquer un problème de performances lié à une charge de travail donnée.

### <a name="dmesg"></a>dmesg

```
$ dmesg | tail 
$ dmesg --level=err | tail
```

dmesg vide la mémoire tampon du noyau. Des événements tels que OOMKill ajoutent une entrée à la mémoire tampon du noyau. La recherche d’un message OOMKill ou d’autres messages d’épuisement des ressources dans les journaux dmesg constitue un indicateur fort de problème.

### <a name="top"></a>top

```
$ top
Tasks: 249 total,   1 running, 158 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.2 us,  1.3 sy,  0.0 ni, 95.4 id,  1.0 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem : 65949064 total, 43415136 free,  2349328 used, 20184600 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 62739060 avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
116004 root      20   0  144400  41124  27028 S  11.8  0.1 248:45.45 coredns
  4503 root      20   0 1677980 167964  89464 S   5.9  0.3   1326:25 kubelet
     1 root      20   0  120212   6404   4044 S   0.0  0.0  48:20.38 systemd
     ...
```

`top` fournit une vue d’ensemble de l’état du système actuel. Les en-têtes offrent de précieuses informations agrégées :

- état des tâches : en cours d’exécution, veille, arrêt.
- Utilisation de l'UC, dans le cas présent montre principalement une durée d’inactivité.
- mémoire système totale, libre et utilisée.

`top` peuvent manquer des processus de courte durée ; des alternatives telles que `htop` et `atop` fournissent des interfaces similaires tout en corrigeant certains de ces défauts.

## <a name="cpu"></a>UC

Ces outils fournissent des informations sur l’utilisation de l'UC. Ils sont particulièrement utiles en termes de sortie en continu, lorsque les modèles deviennent faciles à repérer.

### <a name="mpstat"></a>mpstat

```
$ mpstat -P ALL [interval]
Linux 4.15.0-1064-azure (aks-main-10212767-vmss000001)  02/10/20        _x86_64_        (8 CPU)

19:49:03     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
19:49:04     all    1.01    0.00    0.63    2.14    0.00    0.13    0.00    0.00    0.00   96.11
19:49:04       0    1.01    0.00    1.01   17.17    0.00    0.00    0.00    0.00    0.00   80.81
19:49:04       1    1.98    0.00    0.99    0.00    0.00    0.00    0.00    0.00    0.00   97.03
19:49:04       2    1.01    0.00    0.00    0.00    0.00    1.01    0.00    0.00    0.00   97.98
19:49:04       3    0.00    0.00    0.99    0.00    0.00    0.99    0.00    0.00    0.00   98.02
19:49:04       4    1.98    0.00    1.98    0.00    0.00    0.00    0.00    0.00    0.00   96.04
19:49:04       5    1.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00   98.00
19:49:04       6    1.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00   98.00
19:49:04       7    1.98    0.00    0.99    0.00    0.00    0.00    0.00    0.00    0.00   97.03
```

`mpstat` affiche des informations sur l'UC en haut, mais ventilées par thread d’UC. L’affichage simultané de tous les cœurs peut être utile pour détecter une utilisation déséquilibrée de l'UC, par exemple lorsqu’une application à thread unique utilise un cœur à 100 %. Ce problème peut être plus difficile à repérer lorsqu’il est agrégé sur toutes les UC du système.

### <a name="vmstat"></a>vmstat

```
$ vmstat [interval]
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 43300372 545716 19691456    0    0     3    50    3    3  2  1 95  1  0
```

`vmstat` fournit des informations similaires `mpstat` et `top`, l’énumération du nombre de processus en attente sur l’UC (colonne r), les statistiques de mémoire et le pourcentage de temps processeur consacré à chaque état de travail.

## <a name="memory"></a>Mémoire

La mémoire est une ressource très importante et heureusement facile à suivre. Certains outils peuvent renseigner sur l'UC et la mémoire, notamment `vmstat`. Mais des outils comme `free` peuvent continuer d'être utiles à des fins de débogage rapide.

### <a name="free"></a>gratuit

```
$ free -m
              total        used        free      shared  buff/cache   available
Mem:          64403        2338       42485           1       19579       61223
Swap:             0           0           0
```

`free` affiche des informations de base sur la mémoire totale, ainsi que la mémoire utilisée et libre. `vmstat` peut être plus utile même pour l’analyse de mémoire de base en raison de sa capacité à fournir une sortie en continu.

## <a name="disk"></a>Disque

Ces outils mesurent les IOPS de disque, les files d’attente et le débit total. 

### <a name="iostat"></a>iostat

```
$ iostat -xy [interval] [count]
$ iostat -xy 1 1
Linux 4.15.0-1064-azure (aks-main-10212767-vmss000001)  02/10/20        _x86_64_        (8 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.42    0.00    2.92    1.90    0.00   91.76

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
loop0             0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sdb               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
sda               0.00    56.00    0.00   65.00     0.00   504.00    15.51     0.01    3.02    0.00    3.02   0.12   0.80
scd0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
```

`iostat` fournit des informations détaillées sur l’utilisation des disques. Cet appel transmet `-x` pour les statistiques étendues, `-y` pour ignorer les moyennes du système d’impression de la sortie initiale depuis le démarrage, et `1 1` pour spécifier que nous souhaitons un intervalle de 1 seconde, se terminant après un bloc de sortie. 

`iostat` expose de nombreuses statistiques utiles :

- `r/s` et `w/s` correspondent aux lectures et écritures par seconde. La somme de ces valeurs est IOPS.
- `rkB/s` et `wkB/s` correspondent aux kilo-octets lus/écrits par seconde. La somme de ces valeurs correspond au débit.
- `await` correspond à la durée iowait moyenne en millisecondes pour les requêtes mises en file d’attente.
- `avgqu-sz` correspond à la taille de file d’attente moyenne sur l’intervalle fourni.

Sur une machine virtuelle Azure :

- la somme de `r/s` et `w/s` pour un appareil de bloc individuel ne peut pas dépasser les limites de référence SKU de ce disque.
- la somme de `rkB/s` et `wkB/s` pour un appareil de bloc individuel ne peut pas dépasser les limites de référence SKU de ce disque
- la somme de `r/s` et `w/s` pour tous les appareils de bloc ne peut pas dépasser les limites de la références SKU de la machine virtuelle.
- la somme de `rkB/s` et wkB/s pour tous les appareils de bloc ne peut pas dépasser les limites de la références SKU de la machine virtuelle.

Notez que le disque du système d’exploitation compte en tant que disque managé de la plus petite référence SKU correspondant à sa capacité. Par exemple, un disque de système d’exploitation de 1024 Go correspond à un disque P30. Les disques de système d’exploitation éphémères et les disques temporaires n’ont pas de limites de disque individuelles ; ils sont uniquement limités en fonction des limites totales des machines virtuelles.

Les valeurs non nulles await ou avgqu-SZ sont également d'excellents indicateurs de contention d’E/S.

## <a name="network"></a>Réseau

Ces outils mesurent les statistiques réseau telles que le débit, les échecs de transmission et l’utilisation. Une analyse plus approfondie peut exposer des statistiques TCP affinées en termes de congestion et de paquets ignorés.

### <a name="sar"></a>sar

```
$ sar -n DEV [interval]
22:36:57        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
22:36:58      docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
22:36:58    azv604be75d832      1.00      9.00      0.06      1.04      0.00      0.00      0.00      0.00
22:36:58       azure0     68.00     79.00     27.79     52.79      0.00      0.00      0.00      0.00
22:36:58    azv4a8e7704a5b    202.00    207.00     37.51     21.86      0.00      0.00      0.00      0.00
22:36:58    azve83c28f6d1c     21.00     30.00     24.12      4.11      0.00      0.00      0.00      0.00
22:36:58         eth0    314.00    321.00     70.87    163.28      0.00      0.00      0.00      0.00
22:36:58    azva3128390bff     12.00     20.00      1.14      2.29      0.00      0.00      0.00      0.00
22:36:58    azvf46c95ddea3     10.00     18.00     31.47      1.36      0.00      0.00      0.00      0.00
22:36:58       enP1s1     74.00    374.00     29.36    166.94      0.00      0.00      0.00      0.00
22:36:58           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
22:36:58    azvdbf16b0b2fc      9.00     19.00      3.36      1.18      0.00      0.00      0.00      0.00
```

`sar` est un outil puissant pour un large éventail d’analyses. Bien que cet exemple utilise sa capacité à mesurer les statistiques réseau, il permet aussi de mesurer la consommation d’UC et de mémoire. Cet exemple appelle `sar` avec l’indicateur `-n` pour spécifier le mot clé `DEV` (périphérique réseau), en affichant le débit du réseau par périphérique.

- La somme de `rxKb/s` et `txKb/s` correspond au débit total d’un périphérique donné. Lorsque cette valeur dépasse la limite de carte réseau Azure approvisionnée, les charges de travail de la machine se heurtent à une latence réseau accrue.
- `%ifutil` mesure l’utilisation d’un appareil donné. Lorsque cette valeur avoisine 100 %, les charges de travail se heurtent à une latence réseau accrue.

```
$ sar -n TCP,ETCP [interval]
Linux 4.15.0-1064-azure (aks-main-10212767-vmss000001)  02/10/20        _x86_64_        (8 CPU)

22:50:08     active/s passive/s    iseg/s    oseg/s
22:50:09         2.00      0.00     19.00     24.00

22:50:08     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
22:50:09         0.00      0.00      0.00      0.00      0.00

Average:     active/s passive/s    iseg/s    oseg/s
Average:         2.00      0.00     19.00     24.00

Average:     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
Average:         0.00      0.00      0.00      0.00      0.00
```

Cet appel de `sar` utilise les mots clés `TCP,ETCP` pour examiner les connexions TCP. La troisième colonne de la dernière ligne, « retrans », correspond au nombre de retransmissions TCP par seconde. La présence de valeurs élevées dans ce champ indique une connexion réseau non fiable. Dans les première et troisième lignes, « active » signifie qu’une connexion provient de l’appareil local et « remote » indique une connexion entrante.  L'épuisement des ports SNAT est un problème courant sur Azure et `sar` facilite sa détection. L’épuisement des ports SNAT se manifeste en tant que valeurs « actives » élevées, car le problème est dû à un taux élevé de connexions TCP établies localement et sortantes.

`sar` prenant un intervalle, il imprime la sortie en continu, puis imprime les dernières lignes de sortie contenant les résultats moyens de l’appel.

### <a name="netstat"></a>netstat

```
$ netstat -s
Ip:
    71046295 total packets received
    78 forwarded
    0 incoming packets discarded
    71046066 incoming packets delivered
    83774622 requests sent out
    40 outgoing packets dropped
Icmp:
    103 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 103
    412802 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 412802
IcmpMsg:
        InType3: 103
        OutType3: 412802
Tcp:
    11487089 active connections openings
    592 passive connection openings
    1137 failed connection attempts
    404 connection resets received
    17 connections established
    70880911 segments received
    95242567 segments send out
    176658 segments retransmited
    3 bad segments received.
    163295 resets sent
Udp:
    164968 packets received
    84 packets to unknown port received.
    0 packet receive errors
    165082 packets sent
UdpLite:
TcpExt:
    5 resets received for embryonic SYN_RECV sockets
    1670559 TCP sockets finished time wait in fast timer
    95 packets rejects in established connections because of timestamp
    756870 delayed acks sent
    2236 delayed acks further delayed because of locked socket
    Quick ack mode was activated 479 times
    11983969 packet headers predicted
    25061447 acknowledgments not containing data payload received
    5596263 predicted acknowledgments
    19 times recovered from packet loss by selective acknowledgements
    Detected reordering 114 times using SACK
    Detected reordering 4 times using time stamp
    5 congestion windows fully recovered without slow start
    1 congestion windows partially recovered using Hoe heuristic
    5 congestion windows recovered without slow start by DSACK
    111 congestion windows recovered without slow start after partial ack
    73 fast retransmits
    26 retransmits in slow start
    311 other TCP timeouts
    TCPLossProbes: 198845
    TCPLossProbeRecovery: 147
    480 DSACKs sent for old packets
    175310 DSACKs received
    316 connections reset due to unexpected data
    272 connections reset due to early user close
    5 connections aborted due to timeout
    TCPDSACKIgnoredNoUndo: 8498
    TCPSpuriousRTOs: 1
    TCPSackShifted: 3
    TCPSackMerged: 9
    TCPSackShiftFallback: 177
    IPReversePathFilter: 4
    TCPRcvCoalesce: 1501457
    TCPOFOQueue: 9898
    TCPChallengeACK: 342
    TCPSYNChallenge: 3
    TCPSpuriousRtxHostQueues: 17
    TCPAutoCorking: 2315642
    TCPFromZeroWindowAdv: 483
    TCPToZeroWindowAdv: 483
    TCPWantZeroWindowAdv: 115
    TCPSynRetrans: 885
    TCPOrigDataSent: 51140171
    TCPHystartTrainDetect: 349
    TCPHystartTrainCwnd: 7045
    TCPHystartDelayDetect: 26
    TCPHystartDelayCwnd: 862
    TCPACKSkippedPAWS: 3
    TCPACKSkippedSeq: 4
    TCPKeepAlive: 62517
IpExt:
    InOctets: 36416951539
    OutOctets: 41520580596
    InNoECTPkts: 86631440
    InECT0Pkts: 14
```

`netstat` peut examiner un large éventail de statistiques réseau, appelées ici sous forme de résumé. Il existe ici de nombreux champs utiles selon le problème. Le champs « échecs de tentatives de connexion » constitue un champ particulièrement intéressant de la section TCP. Il peut indiquer un épuisement des ports SNAT ou d'autres problèmes liés aux connexions sortantes. Un taux élevé de segments retransmis (également sous la section TCP) peut indiquer des problèmes de remise de paquets. 
