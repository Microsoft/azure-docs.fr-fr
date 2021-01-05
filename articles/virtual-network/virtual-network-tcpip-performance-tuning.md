---
title: Optimisation des performances TCP/IP pour les machines virtuelles Azure | Microsoft Docs
description: Découvrez les différentes techniques courantes d'optimisation des performances TCP/IP et leur relation avec les machines virtuelles Azure.
services: virtual-network
documentationcenter: na
author: rimayber
manager: paragk
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/02/2019
ms.author: rimayber
ms.reviewer: dgoddard, stegag, steveesp, minale, btalb, prachank
ms.openlocfilehash: 67b635f09cb9407279e89b5f7b8526dab3c08946
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017608"
---
# <a name="tcpip-performance-tuning-for-azure-vms"></a>Optimisation des performances TCP/IP pour les machines virtuelles Azure

Cet article présente les techniques courantes d'optimisation des performances TCP/IP et certains éléments à prendre en compte lorsque vous les utilisez pour les machines virtuelles exécutées sur Azure. Il fournira un aperçu de base des techniques et détaillera les différentes méthodes d’optimisation.

## <a name="common-tcpip-tuning-techniques"></a>Techniques courantes d’optimisation TCP/IP

### <a name="mtu-fragmentation-and-large-send-offload"></a>MTU, fragmentation et déchargement d’envoi important

#### <a name="mtu"></a>MTU

L'unité de transmission maximale (MTU) est la trame de taille maximale (paquet), spécifiée en octets, qui peut être envoyée via une interface réseau. La MTU est un paramètre configurable. La MTU par défaut utilisée sur les machines virtuelles Azure, et le paramètre par défaut sur la plupart des périphériques réseau dans le monde, est de 1 500 octets.

#### <a name="fragmentation"></a>Fragmentation

La fragmentation survient lorsqu'un paquet envoyé dépasse la MTU d'une interface réseau. La pile TCP/IP décomposera le paquet en plus petits morceaux (fragments) conformes à la MTU de l'interface. La fragmentation se produit au niveau de la couche IP et est indépendante du protocole sous-jacent (par exemple TCP). Lorsqu'un paquet de 2 000 octets est envoyé via une interface réseau avec une MTU de 1 500, le paquet sera décomposé en un paquet de 1 500 octets et un autre paquet de 500 octets.

Les périphériques réseau dans le chemin qui sépare une source et une destination peuvent soit supprimer des paquets qui dépassent la MTU, soit fragmenter le paquet en plus petits morceaux.

#### <a name="the-dont-fragment-bit-in-an-ip-packet"></a>Bit Ne pas fragmenter d’un paquet IP

Le bit Ne pas fragmenter (Don’t Fragment, DF) représente un drapeau dans l'en-tête du protocole IP. Le bit DF indique que les périphériques réseau sur le chemin entre l'émetteur et le récepteur ne doivent pas fragmenter le paquet. Ce bit peut être défini pour de nombreuses raisons. (Voir la section « Détection MTU du chemin d'accès » de cet article pour visualiser un exemple). Lorsqu'un périphérique réseau reçoit un paquet avec le bit Ne pas fragmenter défini et que ce paquet dépasse la MTU de l'interface du périphérique, le comportement standard veut que le périphérique supprime le paquet. L'appareil envoie un message « ICMP Fragmentation Needed » (Fragmentation ICMP requise) à la source originale du paquet.

#### <a name="performance-implications-of-fragmentation"></a>Impact de la fragmentation sur les performances

La fragmentation peut avoir des répercussions négatives sur les performances. L'un des principaux effets sur les performances est l'impact au niveau du processeur/de la mémoire de la fragmentation et du réassemblage des paquets. Lorsqu'un périphérique réseau a besoin de fragmenter un paquet, il devra allouer des ressources processeur/mémoire pour effectuer la fragmentation.

La même chose se produit lorsque le paquet est réassemblé. Le périphérique réseau doit stocker tous les fragments jusqu'à ce qu'ils soient reçus afin de pouvoir les réassembler dans le paquet original. Ce processus de fragmentation et de réassemblage peut également provoquer une certaine latence.

L'autre conséquence négative possible de la fragmentation sur les performances est que les paquets fragmentés peuvent arriver en désordre. Lorsque les paquets sont reçus en désordre, certains types de périphériques réseau peuvent les supprimer. Lorsque cela se produit, le paquet entier doit être retransmis.

Les fragments sont généralement supprimés par les dispositifs de sécurité tels que les pare-feu réseau ou lorsque les tampons de réception d'un périphérique réseau sont épuisés. Lorsque les tampons de réception d'un périphérique réseau sont épuisés, un périphérique réseau tente de réassembler un paquet fragmenté mais ne dispose pas des ressources nécessaires pour stocker et consommer à nouveau ce paquet.

La fragmentation peut être considérée comme une opération négative, mais la prise en charge de cette fragmentation est nécessaire lorsque vous connectez divers réseaux via Internet.

#### <a name="benefits-and-consequences-of-modifying-the-mtu"></a>Avantages et conséquences de la modification de la MTU

En règle générale, vous pouvez créer un réseau plus performant en augmentant la MTU. Chaque paquet transmis contient des informations d'en-tête qui sont ajoutées au paquet d'origine. Lorsque la fragmentation crée d’autres paquets, une surcharge se produit au niveau de l’en-tête, et cela rend le réseau moins efficace.

Voici un exemple. La taille de l'en-tête Ethernet est de 14 octets plus une séquence de vérification de trame de 4 octets pour assurer la cohérence des trames. Si un paquet de 2 000 octets est envoyé, 18 octets de surcharge Ethernet sont ajoutés sur le réseau. Si le paquet est fragmenté en un paquet de 1 500 octets plus un paquet de 500 octets, chaque paquet aura 18 octets d'en-tête Ethernet, soit 36 octets au total.

Gardez à l'esprit que l'augmentation de la MTU ne rendra pas nécessairement un réseau plus efficace. Si une application n'envoie que des paquets de 500 octets, la même surcharge d’en-tête existe, que la MTU soit de 1 500 octets ou de 9 000 octets. Le réseau ne sera plus efficace que s'il utilise des paquets de plus grande taille, affectés par la MTU.

#### <a name="azure-and-vm-mtu"></a>MTU pour Azure et les machines virtuelles

La MTU par défaut pour les machines virtuelles Azure est de 1 500 octets. La pile du réseau virtuel Azure tentera de fragmenter un paquet à 1 400 octets.

Notez que la pile du réseau virtuel n'est pas intrinsèquement inefficace car elle fragmente les paquets à 1 400 octets même si les machines virtuelles ont une MTU de 1 500 octets. Un grand pourcentage des paquets réseau sont beaucoup plus petits que 1 400 ou 1 500 octets.

#### <a name="azure-and-fragmentation"></a>Azure et la fragmentation

La pile du réseau virtuel est configurée pour supprimer les « fragments en désordre », c'est-à-dire les paquets fragmentés qui n'arrivent pas dans leur ordre fragmenté d'origine. Ces paquets sont supprimés principalement à cause d'une vulnérabilité de sécurité réseau annoncée en novembre 2018 et appelée FragmentSmack.

FragmentSmack est un défaut dans la façon dont le noyau Linux traite le réassemblage des paquets IPv4 et IPv6 fragmentés. Un attaquant distant pourrait utiliser cette faille pour déclencher des opérations coûteuses de réassemblage des fragments, ce qui risquerait d’entraîner une augmentation du processeur et un déni de service sur le système cible.

#### <a name="tune-the-mtu"></a>Ajuster la MTU

Vous pouvez configurer une MTU de machine virtuelle Azure, comme dans tout autre système d'exploitation. Mais vous devriez tenir compte de la fragmentation qui se produit dans Azure, décrite ci-dessus, lorsque vous configurez une MTU.

Nous déconseillons aux clients d’augmenter les MTU des machines virtuelles. Cette discussion a pour but d'expliquer en détail comment Azure implémente la MTU et effectue la fragmentation.

> [!IMPORTANT]
>L’augmentation de la MTU ne vise pas à améliorer les performances et pourrait avoir un effet négatif sur les performances des applications.
>
>

#### <a name="large-send-offload"></a>Déchargement d’envoi important

Le déchargement d’envoi important (Large Send Offload ou LSO) peut améliorer les performances du réseau en déchargeant la segmentation des paquets sur l'adaptateur Ethernet. Lorsque LSO est activé, la pile TCP/IP crée un grand paquet TCP et l'envoie à l'adaptateur Ethernet pour segmentation avant de le transférer. L'avantage de LSO est qu'il évite au processeur d’effectuer la segmentation des paquets en tailles conformes à la MTU et décharge ce traitement sur l'interface Ethernet où il est effectué dans le matériel. Pour en savoir plus sur les avantages de LSO, consultez la rubrique [Prise en charge du déchargement d'envoi important](https://docs.microsoft.com/windows-hardware/drivers/network/performance-in-network-adapters#supporting-large-send-offload-lso).

Lorsque LSO est activé, les clients Azure peuvent voir de grandes tailles de trames lorsqu'ils effectuent des captures de paquets. Ces grandes tailles de trames peuvent amener certains clients à penser à tort qu’une fragmentation se produit ou qu'une grande MTU est utilisée. Avec LSO, l'adaptateur Ethernet peut annoncer une taille de segment maximale (MSS) plus grande à la pile TCP/IP afin de créer un paquet TCP plus grand. Toute cette trame non segmentée est ensuite transmise à l'adaptateur Ethernet et apparaît dans une capture de paquets effectuée sur la machine virtuelle. Mais le paquet sera décomposé en de nombreuses trames plus petites par l'adaptateur Ethernet, selon la MTU de l'adaptateur Ethernet.

### <a name="tcp-mss-window-scaling-and-pmtud"></a>Mise à l'échelle des fenêtres TCP MSS et PMTUD

#### <a name="tcp-maximum-segment-size"></a>Taille maximale des segments TCP

La taille maximale des segments TCP (MSS) est un paramètre qui limite la taille des segments TCP, évitant ainsi la fragmentation des paquets TCP. Les systèmes d'exploitation utilisent généralement cette formule pour définir les MSS :

`MSS = MTU - (IP header size + TCP header size)`

L'en-tête IP et l'en-tête TCP sont de 20 octets chacun, soit 40 octets au total. Ainsi, une interface avec une MTU de 1 500 octets aura une MSS de 1 460 octets. Mais la MSS est configurable.

Ce paramètre est validé dans la connexion en trois temps TCP lorsqu'une session TCP est configurée entre une source et une destination. Les deux côtés envoient une valeur MSS, et la valeur la plus faible est utilisée pour la connexion TCP.

Gardez à l'esprit que les MTU de la source et de la destination ne sont pas les seuls facteurs qui déterminent la valeur MSS. Les périphériques réseau intermédiaires, comme les passerelles VPN et notamment la passerelle VPN Azure, peuvent ajuster la MTU indépendamment de la source et de la destination pour assurer des performances réseau optimales.

#### <a name="path-mtu-discovery"></a>Détection MTU du chemin d'accès

La MSS est négociée, mais il se peut qu'elle n'indique pas la MSS réelle qui peut être utilisée. En effet, d'autres périphériques réseau dans le chemin entre la source et la destination peuvent avoir une valeur MTU inférieure à celle de la source et de la destination. Dans ce cas, le périphérique dont la MTU est inférieure au paquet supprimera le paquet. L'appareil renverra un message « ICMP Fragmentation Needed (Type 3, Code 4) » (Fragmentation ICMP requise (type 3, code 4)) contenant sa MTU. Ce message ICMP permet à l'hôte source de réduire la MTU de son chemin de manière appropriée. Ce processus est appelé Détection MTU du chemin d'accès ou PMTUD (Path MTU Discovery).

Le processus PMTUD est inefficace et affecte les performances du réseau. Lorsque des paquets sont envoyés et dépassent la MTU d'un chemin réseau, ils doivent être retransmis avec une MSS inférieure. Si l'expéditeur ne reçoit pas le message « ICMP Fragmentation Needed » (Fragmentation ICMP requise), la cause peut venir d'un pare-feu réseau installé dans le chemin d'accès (communément appelé *trou noir PMTUD*) : l'expéditeur ne sait pas qu'il doit baisser la MSS et retransmet continuellement le paquet. C'est pourquoi nous déconseillons d'augmenter la MTU des machines virtuelles Azure.

#### <a name="vpn-and-mtu"></a>VPN et MTU

Si vous utilisez des machines virtuelles qui effectuent une encapsulation (comme les VPN IPsec), vous devez tenir compte de quelques exigences supplémentaires concernant la taille des paquets et la MTU. Les VPN ajoutent d’autres en-têtes aux paquets, ce qui augmente la taille des paquets et nécessite une MSS inférieure.

Pour Azure, nous vous recommandons de définir la MSS TCP sur 1 350 octets et la MTU de l’interface tunnel sur 1 400 octets. Pour plus d'informations, voir la [page Appareils VPN et paramètres IPSec/IKE](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-devices).

### <a name="latency-round-trip-time-and-tcp-window-scaling"></a>Latence, durée des boucles et mise à l'échelle de la fenêtre TCP

#### <a name="latency-and-round-trip-time"></a>Latence et durée des boucles

La latence du réseau est régie par la vitesse de la lumière sur un réseau à fibre optique. Le débit réseau TCP est également régi par la durée des bouches (Round-Trip Time ou RTT) entre deux périphériques réseau.

| Routage | Distance | Durée (unidirectionnel) | RTT |
| ----- | -------- | ------------ | --- |
|New York à San Francisco|4 148 km|21 ms|42 ms|
|New York à London|5 585 km|28 ms|56 ms|
|New York à Sydney|15 993 km|80 ms|160 ms|

Ce tableau indique la distance en ligne droite entre deux emplacements. Dans les réseaux, la distance est généralement plus longue que la distance en ligne droite. Voici une formule simple pour calculer la RTT minimale en fonction de la vitesse de la lumière :

`minimum RTT = 2 * (Distance in kilometers / Speed of propagation)`

Vous pouvez utiliser 200 comme vitesse de propagation. C'est la distance, en kilomètres, que parcourt la lumière en 1 milliseconde.

Prenons comme exemple la distance de New York à San Francisco. La distance en ligne droite est de 4 148 km. En intégrant cette valeur à l'équation, nous obtenons le résultat suivant :

`Minimum RTT = 2 * (4,148 / 200)`

Le résultat de l'équation est exprimé en millisecondes.

Si vous voulez obtenir les meilleures performances réseau, l'option logique consiste à sélectionner les destinations les plus proches les unes des autres. Vous devriez également concevoir votre réseau virtuel en essayant d’optimiser le chemin du trafic et de réduire la latence. Pour plus d’informations, voir la section « Considérations relatives à la conception du réseau » de cet article.

#### <a name="latency-and-round-trip-time-effects-on-tcp"></a>Effets de la latence et de la durée des bouches sur TCP

La durée des boucles a un impact direct sur le débit TCP maximal. Dans le protocole TCP, la *taille de la fenêtre* est la quantité maximale de trafic qui peut être envoyée via une connexion TCP avant que l'expéditeur ait besoin de recevoir un accusé de réception du destinataire. Si la MSS TCP est définie sur 1 460 octets et la taille de la fenêtre TCP sur 65 535 octets, l'expéditeur peut envoyer 45 paquets avant de devoir recevoir un accusé de réception du destinataire. Si l'expéditeur n'obtient pas cet accusé de réception, il retransmet les données. Voici la formule :

`TCP window size / TCP MSS = packets sent`

Dans cet exemple, 65 535 / 1 460 est arrondi à 45.

Cet état d’« attente d’un accusé de réception », mécanisme permettant d'assurer une transmission fiable des données, est à l’origine de l’impact de la RTT sur le débit TCP. Plus l'expéditeur attend un accusé de réception, plus il doit patienter avant d'envoyer d'autres données.

Voici la formule pour calculer le débit maximal d'une connexion TCP unique :

`Window size / (RTT latency in milliseconds / 1,000) = maximum bytes/second`

Ce tableau indique le débit maximal en mégaoctets/seconde d'une connexion TCP unique. (Pour une meilleure lisibilité, les mégaoctets sont utilisés comme unité de mesure.)

| Taille de la fenêtre TCP (octets) | Latence RTT (ms) | Débit maximal en mégaoctets/seconde | Débit maximal en mégabits/seconde |
| ----------------------- | ---------------- | ---------------------------------- | --------------------------------- |
|65 535|1|65,54|524,29|
|65 535|30|2.18|17,48|
|65 535|60|1,09|8,74|
|65 535|90|0,73|5,83|
|65 535|120|0,55|4.37|

En cas de perte de paquets, le débit maximal d'une connexion TCP sera réduit pendant que l'expéditeur retransmet les données qu'il a déjà envoyées.

#### <a name="tcp-window-scaling"></a>Mise à l’échelle de la fenêtre TCP

La mise à l'échelle de la fenêtre TCP est une technique qui augmente dynamiquement la taille de la fenêtre TCP pour permettre d'envoyer plus de données avant qu'un accusé de réception ne soit nécessaire. Dans l'exemple précédent, 45 paquets seraient envoyés avant qu'un accusé de réception ne soit requis. Si vous augmentez le nombre de paquets pouvant être envoyés avant qu'un accusé de réception ne soit nécessaire, réduisez le nombre de fois qu'un expéditeur attend l'accusé de réception afin d’augmenter le débit TCP maximal.

Ce tableau illustre ces relations :

| Taille de la fenêtre TCP (octets) | Latence RTT (ms) | Débit maximal en mégaoctets/seconde | Débit maximal en mégabits/seconde |
| ----------------------- | ---------------- | ---------------------------------- | --------------------------------- |
|65 535|30|2.18|17,48|
|131 070|30|4.37|34,95|
|262 140|30|8,74|69,91|
|524 280|30|17,48|139,81|

Mais la valeur de l'en-tête TCP pour la taille de la fenêtre TCP n'est que de 2 octets, ce qui signifie que la valeur maximale pour une fenêtre de réception est de 65 535 octets. Pour augmenter la taille maximale de la fenêtre, un facteur d'échelle de la fenêtre TCP a été introduit.

Le facteur d'échelle est également un paramètre que vous pouvez configurer dans un système d'exploitation. Voici la formule pour calculer la taille de la fenêtre TCP en utilisant des facteurs d'échelle :

`TCP window size = TCP window size in bytes \* (2^scale factor)`

Voici le calcul pour un facteur d'échelle de fenêtre de 3 octets et une taille de fenêtre de 65 535 octets :

`65,535 \* (2^3) = 262,140 bytes`

Un facteur d'échelle de 14 octets donne une taille de fenêtre TCP de 14 octets (le décalage maximal autorisé). La taille de la fenêtre TCP sera de 1 073 725 440 octets (8,5 gigabits).

#### <a name="support-for-tcp-window-scaling"></a>Prise en charge de la mise à l'échelle de la fenêtre TCP

Windows peut définir différents facteurs d'échelle pour différents types de connexion. (Les classes de connexions incluent les centres de données, Internet, etc.) Vous utilisez la commande PowerShell `Get-NetTCPConnection` pour afficher le type de connexion de mise à l'échelle de la fenêtre :

```powershell
Get-NetTCPConnection
```

Vous pouvez utiliser la commande PowerShell `Get-NetTCPSetting` pour afficher les valeurs de chaque classe :

```powershell
Get-NetTCPSetting
```

Vous pouvez définir la taille initiale de la fenêtre TCP et le facteur d'échelle TCP dans Windows en utilisant la commande PowerShell `Set-NetTCPSetting`. Pour plus d’informations, voir [Set-NetTCPSetting](https://docs.microsoft.com/powershell/module/nettcpip/set-nettcpsetting?view=win10-ps).

```powershell
Set-NetTCPSetting
```

Voici les paramètres TCP effectifs pour `AutoTuningLevel` :

| AutoTuningLevel | Facteur de mise à l’échelle | Multiplicateur de mise à l’échelle | Formule pour<br/>calculer la taille maximale de la fenêtre |
| --------------- | -------------- | ------------------ | -------------------------------------------- |
|Désactivé|None|None|Taille de la fenêtre|
|Restreint|4|2^4|Taille de la fenêtre * (2^4)|
|Très restreinte|2|2^2|Taille de la fenêtre * (2^2)|
|Normal|8|2^8|Taille de la fenêtre * (2^8)|
|Expérimental|14|2^14|Taille de la fenêtre * (2^14)|

Ces paramètres sont les plus susceptibles d'affecter les performances TCP, mais gardez à l'esprit que de nombreux autres facteurs sur Internet, hors du contrôle d’Azure, peuvent également affecter les performances TCP.

#### <a name="increase-mtu-size"></a>Augmenter la taille de la MTU

Puisqu’une MTU plus grande signifie une MSS plus grande, vous pouvez vous demander si l'augmentation de la MTU pourrait augmenter les performances TCP. Probablement pas. Il y a des avantages et des inconvénients liés à la taille des paquets au-delà du seul trafic TCP. Comme nous l'avons vu précédemment, les facteurs les plus importants affectant la performance du débit TCP sont la taille de la fenêtre TCP, la perte de paquets et la valeur RTT.

> [!IMPORTANT]
> Nous déconseillons clients Azure de modifier la valeur MTU par défaut des machines virtuelles.
>
>

### <a name="accelerated-networking-and-receive-side-scaling"></a>Mise en réseau accélérée et mise à l'échelle côté réception

#### <a name="accelerated-networking"></a>Mise en réseau accélérée

Les fonctions réseau de la machine virtuelle ont toujours été gourmandes en ressources processeur, tant sur la machine virtuelle invitée que sur l'hyperviseur/hôte. Chaque paquet qui transite par l'hôte est traité dans un logiciel par le processeur hôte, y compris toutes les encapsulations et décapsulations du réseau virtuel. Ainsi, plus le trafic qui passe par l'hôte est important, plus le processeur est sollicité. Et si le processeur hôte est occupé par d’autres opérations, cela affectera également le débit et la latence du réseau. Azure règle ce problème grâce à la mise en réseau accélérée.

La mise en réseau accélérée offre une latence réseau ultra-faible constante grâce à un matériel programmable interne Azure et à des technologies comme SR-IOV. La mise en réseau accélérée déplace une grande partie de la pile réseau définie par le logiciel Azure des processeurs vers les cartes SmartNIC basées sur FPGA. Ce changement permet aux applications des utilisateurs finaux de récupérer des cycles de calcul, ce qui réduit la charge sur la machine virtuelle, diminue la gigue et l’incohérence de la latence. En d'autres termes, les performances peuvent être plus déterministes.

La mise en réseau accélérée améliore les performances en permettant à la machine virtuelle invitée de contourner l'hôte et d'établir un chemin de données directement avec la carte SmartNIC de l'hôte. Voici quelques avantages de la mise en réseau accélérée :

- **Latence plus faible/Nombre supérieur de paquets par seconde (pps)**  : la suppression du commutateur virtuel du chemin de données évite que les paquets liés au traitement des stratégies séjournent sur l'hôte, ce qui augmente le nombre de paquets pouvant être traités dans la machine virtuelle.

- **Instabilité réduite** : le traitement du commutateur virtuel dépend de la stratégie à appliquer et de la charge de travail du processeur qui effectue le traitement. Le déchargement des stratégies sur le matériel supprime cette variabilité en fournissant des paquets directement à la machine virtuelle, en supprimant l’hôte dans la communication de la machine virtuelle et toutes les interruptions de logiciel et les changements de contexte.

- **Utilisation réduite du processeur** : en contournant le commutateur virtuel de l'hôte, le processeur utilise moins de ressources pour traiter le trafic réseau.

Pour utiliser la mise en réseau accélérée, vous devez l'activer explicitement sur chaque machine virtuelle applicable. Consultez [Créer une machine virtuelle Linux avec la mise en réseau accélérée](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli) pour obtenir des instructions.

#### <a name="receive-side-scaling"></a>Mise à l'échelle côté réception

La mise à l'échelle côté réception (Receive Side Scaling ou RSS) est une technologie de pilote réseau qui distribue plus efficacement la réception du trafic réseau en répartissant le traitement de réception entre plusieurs processeurs dans un système multiprocesseur. Pour simplifier, RSS permet à un système de traiter plus de trafic reçu en utilisant tous les processeurs disponibles au lieu d'un seul. Pour obtenir des informations plus techniques sur RSS, consultez [Introduction à la mise à l'échelle côté réception](https://docs.microsoft.com/windows-hardware/drivers/network/introduction-to-receive-side-scaling).

Pour obtenir les meilleures performances lorsque la mise en réseau accélérée est activée sur une machine virtuelle, vous devez activer RSS. RSS peut également offrir des avantages sur les machines virtuelles qui n'utilisent pas la mise en réseau accélérée. Pour un aperçu sur la façon de vérifier si RSS est activé et comment le faire le cas échéant, voir [Optimiser le débit du réseau des machines virtuelles Azure](https://aka.ms/FastVM).

### <a name="tcp-time_wait-and-time_wait-assassination"></a>Assassinat TCP TIME_WAIT et TIME_WAIT

TCP TIME_WAIT est un autre paramètre courant qui affecte les performances du réseau et des applications. Sur les machines virtuelles occupées qui ouvrent et ferment plusieurs sockets, soit en tant que clients, soit en tant que serveurs (IP source : port source + IP de destination : port de destination), pendant le fonctionnement TCP normal, un socket donné peut finir dans un état TIME_WAIT sur une longue période. L'état TIME_WAIT a pour but de permettre aux données supplémentaires d'être transmises sur un socket avant sa fermeture. Ainsi, les piles TCP/IP empêchent généralement la réutilisation d'un socket en supprimant silencieusement le paquet TCP SYN du client.

La durée pendant laquelle un socket est à l’état TIME_WAIT est configurable. Elle peut varier de 30 à 240 secondes. Les sockets constituent une ressource limitée, et le nombre de sockets qui peuvent être utilisés à tout moment est configurable. (Le nombre de sockets disponibles est généralement d'environ 30 000.) Si les sockets disponibles sont consommés, ou si les paramètres TIME_WAIT des clients et des serveurs ne correspondent pas, et qu'une machine virtuelle tente de réutiliser un socket à un état TIME_WAIT, les nouvelles connexions échoueront car les paquets TCP SYN sont supprimés en silence.

La valeur de la plage de ports pour les sockets sortants est généralement configurable dans la pile TCP/IP d'un système d'exploitation. C’est également le cas pour les paramètres TCP TIME_WAIT et la réutilisation d’un socket. La modification de ces valeurs peut potentiellement améliorer l'évolutivité. Mais, selon la situation, ces changements pourraient provoquer des problèmes d'interopérabilité. Faites attention si vous modifiez ces valeurs.

Vous pouvez utiliser l'assassinat TIME_WAIT pour résoudre ce problème de mise à l'échelle. L'assassinat TIME_WAIT permet de réutiliser un socket dans certaines situations, comme lorsque le numéro de séquence dans le paquet IP de la nouvelle connexion dépasse le numéro de séquence du dernier paquet de la connexion précédente. Dans ce cas, le système d'exploitation permettra d'établir la nouvelle connexion (il acceptera le nouvelle valeur SYN/ACK) et forcera la fermeture de la connexion précédente qui était dans un état TIME_WAIT. Cette capacité est prise en charge sur les machines virtuelles Windows dans Azure. Pour en savoir plus sur la prise en charge dans d'autres machines virtuelles, contactez l’éditeur du système d'exploitation.

Pour en savoir plus sur la configuration des paramètres TCP TIME_WAIT et la plage de ports source, voir [Paramètres pouvant être modifiés pour améliorer les performances du réseau](https://docs.microsoft.com/biztalk/technical-guides/settings-that-can-be-modified-to-improve-network-performance).

## <a name="virtual-network-factors-that-can-affect-performance"></a>Facteurs du réseau virtuel pouvant affecter les performances

### <a name="vm-maximum-outbound-throughput"></a>Débit sortant maximal de la machine virtuelle

Azure offre une variété de tailles et de types de machines virtuelles, qui présentent une combinaison différente de capacités de performances. Une de ces capacités est le débit réseau (ou bande passante), mesurée en mégabits par seconde (Mbit/s). Étant donné que les machines virtuelles sont hébergées sur du matériel partagé, la capacité du réseau doit être répartie équitablement entre les machines virtuelles partageant le même matériel. Les machines virtuelles plus volumineuses bénéficient d’une plus grande bande passante que les machines plus petites.

La bande passante réseau allouée à chaque machine virtuelle est mesurée en fonction du trafic sortant de la machine virtuelle. L’ensemble du trafic réseau quittant la machine virtuelle est mesuré en fonction de la limite d’allocation, quelle que soit la destination. Par exemple, si une machine virtuelle se voit allouer une limite de 1 000 Mbit/s, cette limite s’applique de toute façon, que le trafic sortant soit destiné à une machine virtuelle du réseau virtuel ou à une machine en dehors d’Azure.

Le trafic en entrée n’est pas compté ou limité directement. Mais il existe d’autres facteurs, tels que les limites en termes de processeur et de stockage, susceptibles d’affecter la capacité d’une machine virtuelle à traiter les données entrantes.

La mise en réseau accélérée est conçue pour améliorer les performances du réseau, notamment la latence, le débit et le taux d’utilisation des processeurs. La mise en réseau accélérée peut améliorer le débit de la machine virtuelle, mais en deçà de la limite de bande passante allouée à la machine virtuelle.

Les machines virtuelles Azure doivent toujours être associées à au moins une interface réseau. Il peut y en avoir plusieurs. La bande passante allouée à une machine virtuelle représente l’intégralité du trafic sortant sur l’ensemble des interfaces réseau associées à la machine. En d’autres termes, la bande passante est allouée machine virtuelle par machine virtuelle, quel que soit le nombre d’interfaces réseau associées à cette machine.

Le débit sortant attendu et le nombre d’interfaces réseau prises en charge par chaque taille de machine virtuelle sont détaillés dans la section [Tailles des machines virtuelles Windows dans Azure](https://docs.microsoft.com/azure/virtual-machines/windows/sizes?toc=%2fazure%2fvirtual-network%2ftoc.json). Pour afficher le débit maximal, sélectionnez un type, par exemple **Usage général**, puis recherchez la section consacrée aux séries de tailles sur la page résultante (par exemple, « Série Dv2 »). Pour chaque série, un tableau fournit les spécifications réseau dans la dernière colonne, sous le titre « Nombre max de cartes réseau / Bande passante réseau attendue (Mbit/s) ».

Cette limite de débit s’applique à la machine virtuelle. Le débit n'est pas affecté par ces facteurs :

- **Nombre d'interfaces réseau** : La limite de bande passante s’applique à l’ensemble du trafic sortant en provenance de la machine virtuelle.

- **Mise en réseau accélérée** : bien que cette fonctionnalité puisse être utile pour atteindre la limite publiée, elle ne modifie pas cette limite.

- **Destination du trafic** : toutes les destinations s’inscrivent dans la limite de sortie.

- **Protocole** : L’intégralité du trafic sortant, sur l’ensemble des protocoles, s’inscrit dans la limite.

Pour plus d’informations, consultez [Bande passante réseau des machines virtuelles](https://aka.ms/AzureBandwidth).

### <a name="internet-performance-considerations"></a>Considérations relatives aux performances Internet

Comme expliqué tout au long de cet article, des facteurs sur Internet et hors du contrôle d'Azure peuvent affecter les performances du réseau. Voici quelques-uns de ces facteurs :

- **Latence** : La durée des boucles entre deux destinations peut être affectée par des problèmes sur les réseaux intermédiaires, par un trafic qui ne prend pas le chemin le plus court, et par des chemins de peering non optimaux.

- **Perte de paquets** : Une perte de paquets peut être causée par l'encombrement du réseau, des problèmes de chemin physique et des périphériques réseau moins performants.

- **Taille de MTU/fragmentation** : La fragmentation le long du chemin peut entraîner des retards dans la réception des données ou une réception de paquets en désordre, ce qui peut affecter la remise des paquets.

Traceroute est un bon outil pour mesurer les performances du réseau (notamment la perte de paquets et la latence) le long de chaque chemin réseau entre un périphérique source et un périphérique cible.

### <a name="network-design-considerations"></a>Considérations relatives à la conception du réseau

Outre les considérations abordées précédemment dans cet article, la topologie d'un réseau virtuel peut affecter les performances du réseau. Par exemple, une conception « hub-and-spoke » qui renvoie le trafic mondial vers un réseau virtuel à un seul concentrateur entraînera une latence du réseau, ce qui affectera ses performances globales.

Le nombre de périphériques réseau par lesquels le trafic réseau transite peut également affecter la latence globale. Par exemple, dans une conception « hub-and-spoke », si le trafic transite via une appliance virtuelle réseau spoke et une appliance virtuelle de concentrateur avant d’être dirigé vers Internet, les appliances virtuelles réseau peuvent entraîner une latence.

### <a name="azure-regions-virtual-networks-and-latency"></a>Régions Azure, réseaux virtuels et latence

Les régions Azure sont composées de plusieurs centres de données qui existent dans une zone géographique générale. Ces centres de données peuvent ne pas être physiquement adjacents. Dans certains cas, ils sont séparés par une distance pouvant atteindre 10 kilomètres. Le réseau virtuel est une superposition logique sur le réseau du centre de données physique Azure. Un réseau virtuel n'implique aucune topologie de réseau spécifique au sein du centre de données.

Par exemple, deux machines virtuelles figurant dans le même réseau virtuel et le même sous-réseau peuvent se trouver dans des racks, des lignes ou même des centres de données différents. Elles pourraient être séparées par plusieurs mètres ou plusieurs kilomètres de câble à fibre optique. Cette variation pourrait entraîner une latence variable (une différence de quelques millisecondes) entre différentes machines virtuelles.

L'emplacement géographique des machines virtuelles et la latence potentielle qui en résulte entre deux machines virtuelles peuvent être influencés par la configuration des groupes à haute disponibilité et des zones de disponibilité. Mais la distance entre les centres de données d'une région est spécifique à cette région et elle est principalement influencée par la topologie des centres de données de cette région.

### <a name="source-nat-port-exhaustion"></a>Épuisement du port NAT source

Un déploiement dans Azure peut communiquer avec des points de terminaison en dehors d’Azure sur le réseau Internet public et/dans l’espace d’adressage IP public. Quand une instance démarre une connexion sortante, Azure mappe dynamiquement l’adresse IP privée à une adresse IP publique. Une fois ce mappage créé, le trafic de retour pour le flux sortant peut aussi atteindre l’adresse IP privée d’où provient le flux.

Pour chaque connexion sortante, Azure Load Balancer doit maintenir ce mappage pendant un certain temps. Étant donné la nature multilocataire d'Azure, le maintien de ce mappage pour chaque flux sortant de chaque machine virtuelle peut consommer beaucoup de ressources. Il existe donc des limites fixées et basées sur la configuration du réseau virtuel Azure. Ou, pour simplifier, une machine virtuelle Azure ne peut uniquement établir un certain nombre de connexions sortantes à un moment donné. Lorsque ces limites sont atteintes, la machine virtuelle ne pourra plus établir de connexions sortantes.

Mais ce comportement est configurable. Pour plus d'informations sur SNAT et l'épuisement de port SNAT, consultez [cet article](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections).

## <a name="measure-network-performance-on-azure"></a>Mesurer les performances du réseau sur Azure

Un certain nombre des performances maximales de cet article sont liées à la latence du réseau et à la durée des bouches (RTT) entre deux machines virtuelles. Cette section fournit quelques suggestions sur la façon de tester la latence/RTT et les performances TCP et des machines virtuelles du réseau. Vous pouvez régler les valeurs TCP/IP et réseau et tester leurs performances, comme discuté précédemment, en utilisant les techniques décrites dans cette section. Vous pouvez intégrer des valeurs de latence, MTU, MSS et de taille de fenêtre aux calculs fournis précédemment et comparer les valeurs maximales théoriques aux valeurs réelles que vous observez pendant les tests.

### <a name="measure-round-trip-time-and-packet-loss"></a>Mesurer la durée des bouches (RTT) et la perte de paquets

Les performances TCP dépendent fortement de RTT et de la perte de paquets. L'utilitaire PING disponible sur Windows et Linux fournit le moyen le plus simple de mesurer la durée RTT et la perte de paquets. Le résultat affiché par PING indiquera la latence minimale/maximale/moyenne entre une source et une destination. Il précisera également la perte de paquets. PING utilise le protocole ICMP par défaut. Vous pouvez utiliser PsPing pour tester TCP RTT. Pour plus d'informations, consultez [PsPing](https://docs.microsoft.com/sysinternals/downloads/psping).

### <a name="measure-actual-throughput-of-a-tcp-connection"></a>Mesurer le débit réel d'une connexion TCP

NTttcp est un outil permettant de tester les performances TCP d'une machine virtuelle Linux ou Windows. Vous pouvez modifier divers paramètres TCP et tester leurs performances en utilisant NTttcp. Pour plus d’informations, consultez ces ressources :

- [Test de bande passante/débit (NTttcp)](https://aka.ms/TestNetworkThroughput)

- [Utilitaire NTttcp](https://gallery.technet.microsoft.com/NTttcp-Version-528-Now-f8b12769)

### <a name="measure-actual-bandwidth-of-a-virtual-machine"></a>Mesurer la bande passante réelle d'une machine virtuelle

Vous pouvez tester les performances de différents types de machines virtuelles, de mises en réseau accélérées, etc. en utilisant un outil appelé iPerf. iPerf est également disponible sur Linux et Windows. iPerf peut utiliser TCP ou UDP pour tester le débit global du réseau. Les tests de débit TCP iPerf sont influencés par les facteurs abordés dans cet article (notamment la latence et la durée RTT). Ainsi, UDP peut donner de meilleurs résultats si vous voulez juste tester le débit maximal.

Pour plus d’informations, voir les articles suivants :

- [Résolution des problèmes de performances réseau ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-troubleshooting-network-performance)

- [Comment valider un débit VPN sur un réseau virtuel](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-validate-throughput-to-vnet)

### <a name="detect-inefficient-tcp-behaviors"></a>Détecter les comportements TCP inefficaces

Dans les captures de paquets, les clients Azure peuvent voir des paquets TCP avec des indicateurs TCP (SACK, DUP ACK, RETRANSMIT et FAST RETRANSMIT) qui pourraient signaler des problèmes de performance réseau. Ces paquets indiquent spécifiquement les problèmes réseau qui résultent de la perte de paquets. Mais la perte de paquets n'est pas nécessairement causée par des problèmes de performance Azure. Ces baisses de performance pourraient être le résultat de problèmes au niveau des applications, du système d'exploitation, ou d’autres problèmes qui pourraient ne pas être directement liés à la plate-forme Azure.

Sans oublier que certains événements ACK de retransmission et de duplication sont normaux sur un réseau. Les protocoles TCP ont été conçus pour être fiables. La présence de ces paquets TCP dans une capture de paquets n'indique pas nécessairement un problème de réseau systémique, à moins que leur nombre soit excessif.

Néanmoins, ces types de paquets sont des indications que le débit TCP n'atteint pas ses performances maximales, pour les raisons discutées dans d’autres sections de cet article.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous en savez plus sur l'optimisation des performances TCP/IP pour les machines virtuelles Azure, vous pouvez examiner d'autres considérations sur [la planification des réseaux virtuels](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm) ou [en savoir plus sur la connexion et la configuration des réseaux virtuels](https://docs.microsoft.com/azure/virtual-network/).
