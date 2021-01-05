---
title: 'Stockage Premium Azure : Conception pour de hautes performances'
description: Concevez des applications hautes performances avec les disques managés SSD Azure Premium. Premium Storage offre une prise en charge très performante et à faible latence des disques pour les charges de travail utilisant beaucoup d'E/S exécutées sur les machines virtuelles Azure.
author: roygara
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 10/05/2020
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: acdddcd95883d13393838a47281fb888ac2f9274
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500391"
---
# <a name="azure-premium-storage-design-for-high-performance"></a>Stockage Azure Premium : conception sous le signe de la haute performance

Cet article fournit des instructions pour la création d’applications hautes performances avec Azure Premium Storage. Vous pouvez utiliser les instructions fournies dans ce document parallèlement aux bonnes pratiques de performances applicables aux technologies utilisées par votre application. Pour illustrer les instructions, nous avons utilisé comme exemple un SQL Server exécuté sur Premium Storage.

Bien que cet article couvre plusieurs scénarios de performances au niveau de la couche de stockage, vous devrez optimiser la couche applicative. Par exemple, si vous hébergez une batterie de serveurs SharePoint sur Azure Premium Storage, vous pouvez utiliser les exemples SQL Server de cet article pour optimiser le serveur de base de données. Vous devez également optimiser le serveur web et le serveur d’applications de la batterie de serveurs SharePoint pour obtenir de meilleures performances.

Cet article vous aidera à répondre à certaines questions courantes relatives à l’optimisation des performances applicatives sur Azure Premium Storage :

* Comment mesurer les performances de votre application ?  
* Pourquoi n’obtenez-vous pas les hautes performances attendues ?  
* Quels sont les facteurs qui influencent les performances de votre application sur Premium Storage ?  
* Comment ces facteurs influencent-ils les performances de votre application sur Premium Storage ?  
* Comment optimiser les performances d’E/S par seconde, de bande passante et de latence ?  

Ces instructions vous sont spécifiquement fournies pour Premium Storage, car les charges de travail exécutées sur Premium Storage sont extrêmement sensibles aux performances. Nous vous proposons des exemples lorsque cela s’y prête. Vous pouvez également appliquer certaines de ces instructions aux applications qui s’exécutent sur des machines virtuelles IaaS avec des disques de stockage Standard.

> [!NOTE]
> Parfois, ce qui semble être un problème de performances disque est en fait un goulot d’étranglement sur le réseau. Dans ces situations, vous devez optimiser vos [performances réseau](../virtual-network/virtual-network-optimize-network-bandwidth.md).
>
> Si vous souhaitez évaluer votre disque, consultez les articles consacrés au benchmarking d'un disque :
>
> * Pour Linux : [Créer un benchmark de votre application sur Stockage sur disque Azure](linux/disks-benchmarks.md)
> * Pour Windows : [Benchmarking d'un disque](windows/disks-benchmarks.md).
>
> Si votre machine virtuelle prend en charge la mise en réseau accélérée, vous devez vous assurer qu’elle est activée. Si elle n’est pas activée, vous pouvez l’activer sur les machines virtuelles déjà déployées sur [Windows](../virtual-network/create-vm-accelerated-networking-powershell.md#enable-accelerated-networking-on-existing-vms) et [Linux](../virtual-network/create-vm-accelerated-networking-cli.md#enable-accelerated-networking-on-existing-vms).

Avant de commencer, si vous débutez avec le Stockage Premium, commencez par lire les articles [Sélectionner un type de disque Azure pour les machines virtuelles IaaS](disks-types.md) et [Cibles de scalabilité pour les comptes de stockage d’objets blob de pages Premium](../storage/blobs/scalability-targets-premium-page-blobs.md).

## <a name="application-performance-indicators"></a>Indicateurs de performances d’une application

Pour évaluer le degré de performances d’une application, nous nous appuyons sur divers indicateurs de performances, par exemple la vitesse à laquelle une application traite une demande utilisateur, la quantité de données qu’une application est capable de traiter pour chaque demande, le nombre de requêtes traitées par une application dans un intervalle donné, ou encore le temps d’attente que doit subir l’utilisateur avant d’obtenir une réponse à sa demande. Sur le plan technique, ces indicateurs de performances s’expriment en termes d’E/S par seconde, de débit (ou bande passante) et de latence.

Dans cette section, nous allons aborder les indicateurs de performances courants dans le contexte de Premium Storage. Dans la section suivante, intitulée « Collecte des exigences de performances de l’application », vous apprendrez à mesurer ces indicateurs de performances pour votre propre application. Dans la section consacrée à l’optimisation des performances applicatives, vous découvrirez les facteurs qui affectent ces indicateurs de performances et obtiendrez des recommandations afin de les optimiser.

## <a name="iops"></a>E/S par seconde

Le nombre d’E/S par seconde représente le nombre de demandes que votre application envoie aux disques de stockage en une seconde. Une opération d’entrée/sortie peut être accessible en lecture ou écriture, et être séquentielle ou aléatoire. Les applications OLTP, comme un site web de vente en ligne, par exemple, doivent traiter immédiatement de nombreuses requêtes d’utilisateurs simultanées. Les requêtes utilisateurs représentent des transactions d’insertion et de mise à jour qui pèsent lourdement sur la base de données, et que l’application doit traiter rapidement. Les applications OLTP requièrent dont un nombre d’E/S par seconde très élevé. Ces applications gèrent des millions de demandes d’E/S petites et aléatoires. Si vous possédez une telle application, vous devez concevoir votre infrastructure applicative de manière à optimiser les E/S. Dans la section *Optimisation des performances applicatives*, nous aborderons en détail tous les facteurs que vous devez prendre en compte pour obtenir un nombre d’E/S par seconde élevé.

Lorsque vous attachez un disque de stockage Premium à votre machine virtuelle à grande échelle, Azure met à disposition un nombre garanti d’E/S par seconde, conformément à la spécification de disque. Par exemple, un disque P50 configure 7500 E/S par seconde. Chaque taille de machine virtuelle à grande échelle est également associée à une limite spécifique d’E/S par seconde qu’elle peut prendre en charge. Par exemple, une machine virtuelle GS5 Standard a une limite de 80 000 E/S par seconde.

## <a name="throughput"></a>Débit

Le débit ou la bande passante est la quantité de données que votre application envoie aux disques de stockage dans un intervalle spécifié. Si votre application effectue des opérations d’entrée/sortie avec des tailles d’unité d’E/S importantes, elle aura besoin d’un débit élevé. Les applications d’entrepôt de données ont tendance à émettre des opérations d’analyse intensives qui accèdent simultanément à de grandes quantités de données et exécutent généralement des opérations en bloc. En d’autres termes, ces applications nécessitent un débit plus élevé. Si vous possédez une telle application, vous devez concevoir votre infrastructure de manière à en optimiser le débit. Dans la section suivante, nous décrirons en détail les facteurs que vous devrez ajuster pour y parvenir.

Lorsque vous attachez un disque de stockage Premium à une machine virtuelle à grande échelle, Azure met à disposition un débit conforme à la spécification de ce disque. Par exemple, un disque P50 configure un débit de disque de 250 Mo par seconde. Chaque taille de machine virtuelle à grande échelle est également associée à une limite spécifique de débit qu’elle peut prendre en charge. Par exemple, une machine virtuelle GS5 Standard a un débit maximal de 2 000 Mo par seconde.

La formule ci-dessous illustre la relation entre le débit et le nombre d’E/S par seconde.

![Relation entre le nombre d’E/S par seconde et le débit](linux/media/premium-storage-performance/image1.png)

Par conséquent, il est important de déterminer les valeurs optimales de débit et d’E/S par seconde dont a besoin votre application. Lorsque vous essayez d’optimiser une de ces valeurs, l’autre est également affectée. Dans la section *Optimisation des performances applicatives*, nous aborderons plus en détail la question de l’optimisation des E/S par seconde et du débit.

## <a name="latency"></a>Latence

La latence est le temps nécessaire à une application pour recevoir une demande unique, l’envoyer aux disques de stockage et transmettre la réponse au client. Il s’agit d’une mesure critique des performances d’une application, qui s’ajoute à celle des E/S par seconde et du débit. La latence d’un disque de stockage premium correspond au temps nécessaire pour récupérer les informations d’une demande et les retourner à votre application. Premium Storage offre de faibles latences. Les disques Premium sont conçus pour offrir des latences en millisecondes à un chiffre pour la plupart des opérations d'E/S. Si vous activez une mise en cache de l’hôte en lecture seule sur des disques de stockage premium, vous pourrez obtenir une latence de lecture bien plus faible. Nous aborderons la mise en cache du disque plus en détail dans la section *Optimisation des performances applicatives*.

Lorsque vous optimisez votre application pour augmenter le nombre d’E/S par seconde et le débit, la latence de votre application s’en trouve affectée. Après avoir ajusté les performances de votre application, pensez toujours à évaluer la latence de l’application afin d’éviter un comportement de latence élevée inattendu.

Le suivi des opérations de plan de contrôle sur les disques managés peut impliquer le déplacement du disque d’un emplacement de stockage à un autre. Cette opération est effectuée via une copie en arrière-plan des données qui peut prendre plusieurs heures (généralement moins de 24 heures) selon la quantité de données stockées sur les disques. Pendant ce temps, votre application peut afficher une latence de lecture supérieure à la normale car certaines lectures sont redirigées vers l’emplacement d’origine et prennent donc plus de temps. Il n’y a aucun impact sur la latence d’écriture pendant cette période.

- Mettez à jour le type de stockage.
- Détachez un disque d’une machine virtuelle et attachez-le à une autre.
- Créez un disque managé à partir d’un VHD.
- Créez un disque managé à partir d’un instantané.
- Convertissez des disques non managés en disques managés.

## <a name="performance-application-checklist-for-disks"></a>Liste de vérification des applications hautes performances pour les disques

La première étape de la conception d’applications hautes performances exécutées sur le Stockage Azure Premium consiste à comprendre les exigences de performances de votre application. Après avoir recueilli ces exigences de performances, vous pourrez optimiser votre application de manière à obtenir les meilleures performances possibles.

Dans la section précédente, nous avons expliqué les indicateurs de performances courants, à savoir le nombre d’E/S par seconde, le débit et la latence. Vous devez déterminer, parmi ces indicateurs de performances, lesquels sont essentiels à votre application pour fournir l’expérience utilisateur recherchée. Par exemple, un nombre d’E/S par seconde élevé est plus important pour les applications OLTP qui traitent des millions de transactions en une seconde. En revanche, un débit élevé sera plus pertinent pour les applications d’entrepôt de données qui traitent de grandes quantités de données en une seconde. De même, une très faible latence sera cruciale pour les applications en temps réel, telles que les sites web de diffusion en continu de vidéo en direct.

Ensuite, vous devrez mesurer les exigences de performances maximales de votre application tout au long de sa durée de vie. Utilisez l’exemple de liste de contrôle ci-dessous comme point de départ. Notez les exigences de performances maximales relevées au cours des périodes de charge de travail normales, des périodes de pointe et des heures creuses. En identifiant les conditions requises pour tous les niveaux de charges de travail, vous serez en mesure de déterminer les exigences de performances globales de votre application. Par exemple, la charge de travail normale d’un site web de commerce électronique correspond aux transactions habituellement traitées au quotidien au cours d’une année. Les pics de charge du site web correspondront aux transactions traitées pendant la période de Noël ou les périodes de soldes. Les pics de charge ne durent généralement que pendant un temps limité, mais ils peuvent vous obliger à mettre à l’échelle votre application en multipliant au moins par deux sa capacité par rapport à son fonctionnement normal. Déterminez les besoins au 50e percentile, au 90e percentile et au 99e percentile. Cela vous permettra d’éliminer les observations aberrantes en termes d’exigences de performances et de concentrer vos efforts sur l’optimisation des valeurs adéquates.

## <a name="application-performance-requirements-checklist"></a>Liste de contrôle pour les exigences de performances de l’application

| **Exigences de performances** | **50e percentile** | **90e percentile** | **99e percentile** |
| --- | --- | --- | --- |
| Bande passante Transactions par seconde | | | |
| % d’opérations de lecture | | | |
| % d’opérations d’écriture | | | |
| % d’opérations aléatoires | | | |
| % d’opérations séquentielles | | | |
| Taille de la demande d’E/S | | | |
| Débit moyen | | | |
| Bande passante Débit | | | |
| min. Latence | | | |
| Latence moyenne | | | |
| Bande passante UC | | | |
| Utilisation moyenne de l’UC | | | |
| Bande passante Mémoire | | | |
| Utilisation moyenne de la mémoire | | | |
| Profondeur de file d’attente | | | |

> [!NOTE]
> vous devez anticiper la mise à l’échelle de ces nombres en fonction de la croissance prévue de votre application. Il est judicieux de planifier la croissance, car il pourrait être plus difficile de modifier l’infrastructure ultérieurement afin d’en améliorer les performances.

Si vous possédez une application existante et que vous souhaitez migrer vers Premium Storage, commencez par compléter la liste de contrôle ci-dessus pour l’application existante. Développez ensuite un prototype de votre application sur Premium Storage et concevez l’application selon les instructions décrites dans la section *Optimisation des performances applicatives* dans la suite de ce document. L’article suivant décrit les outils que vous pouvez utiliser pour collecter les mesures de performances.

### <a name="counters-to-measure-application-performance-requirements"></a>Compteurs de mesure des exigences de performances de l’application

La meilleure façon de mesurer les exigences de performances de votre application consiste à utiliser les outils d’analyse des performances fournies par le système d’exploitation du serveur. Vous pouvez utiliser PerfMon pour Windows et iostat pour Linux. Ces outils capturent des compteurs correspondant à chaque mesure expliquée dans la section ci-dessus. Vous devez capturer les valeurs de ces compteurs lorsque votre application exécute ses charges de travail normales, de pointe et d’heures creuses.

Les compteurs PerfMon sont disponibles pour le processeur, pour la mémoire et pour chaque disque logique et chaque disque physique de votre serveur. Lorsque vous utilisez les disques de stockage premium avec une machine virtuelle, les compteurs de disque physique s’appliquent à chaque disque de stockage premium et les compteurs de disque logique à chaque volume créé sur les disques de stockage premium. Vous devez capturer les valeurs des disques qui hébergent la charge de travail de votre application. S’il existe un mappage un-à-un entre les disques physiques et logiques, vous pouvez consulter les compteurs de disque physique ; sinon, référez-vous aux compteurs de disque logique. Sous Linux, la commande iostat génère un rapport d’utilisation du processeur et du disque. Le rapport d’utilisation du disque fournit des statistiques par unité physique ou par partition. Si vous avez un serveur de base de données dont les données et les journaux résident sur des disques distincts, récupérez ces données pour les deux disques. Le tableau ci-dessous décrit les compteurs de disques, de processeurs et de mémoire :

| Compteur | Description | PerfMon | Iostat |
| --- | --- | --- | --- |
| **Nombre d’E/S par seconde ou de transactions par seconde** |Nombre de demandes d’E/S émises sur le disque de stockage par seconde. |Nb d’opérations de lectures de disque/s <br> Nb d’opération d’écriture de disque/s |tps <br> r/s <br> w/s |
| **Opérations de lecture et d’écriture de disque** |% d’opérations de lecture et d’écriture exécutées sur le disque. |% temps de lecture du disque <br> % temps d’écriture du disque |r/s <br> w/s |
| **Débit** |Quantité de données lues ou écrites sur le disque par seconde. |Nb d’octets de lecture de disque/s <br> Nb d’octets d’écriture de disque/s |kB_read/s <br> kB_wrtn/s |
| **Latence** |Durée totale d’exécution d’une demande d’E/S sur le disque. |Temps de lecture moyen du disque/s <br> Temps d’écriture moyen du disque/s |await <br> svctm |
| **Taille d’E/S** |La taille des demandes d’E/S émises sur les disques de stockage. |Nb moyen d’octets en lecture du disque <br> Nb moyen d’octets en écriture du disque |avgrq-sz |
| **Profondeur de file d’attente** |Nombre de demandes d’E/S en attente de lecture ou d’écriture sur le disque de stockage. |Longueur actuelle de la file d'attente du disque |avgqu-sz |
| **Bande passante Mémoire** |Quantité de mémoire nécessaire pour une exécution fluide de l’application |% d’octets dédiés utilisés |Use vmstat |
| **Bande passante UC** |Quantité d’UC nécessaire pour une application fluide de l’application |% temps processeur |%util |

En savoir plus sur [iostat](https://linux.die.net/man/1/iostat) et [PerfMon](/windows/win32/perfctrs/performance-counters-portal).



## <a name="optimize-application-performance"></a>Optimiser les performances de l'application

La nature des demandes d’E/S, la taille de machine virtuelle, la taille de disque, le nombre de disques, la mise en cache du disque, le traitement multithread et la profondeur de file d’attente représentent les principaux facteurs qui influencent les performances d’une application exécutée sur le stockage Premium. Vous pouvez contrôler certains de ces facteurs avec les dispositifs fournis par le système. La plupart des applications ne vous permettront peut-être pas de modifier directement la taille d’E/S et la profondeur de file d’attente. Par exemple, si vous utilisez SQL Server, vous ne pouvez choisir ni la taille d’E/S ni la profondeur de file d’attente. SQL Server choisit les valeurs optimales de taille d’E/S et de profondeur de file d’attente profondeur de manière à obtenir les meilleures performances. Il est important de comprendre les effets de ces deux types de facteurs sur les performances de votre application, afin que vous puissiez configurer les ressources appropriées pour répondre à vos besoins de performances.

Dans cette section, reportez-vous à la liste de contrôle des exigences de performances applicatives que vous avez créée pour identifier le degré nécessaire d’optimisation des performances de votre application. Sur cette base, vous serez en mesure de déterminer les facteurs de cette section que vous devrez paramétrer. Pour évaluer les effets de chaque facteur sur les performances de votre application, exécutez les outils de benchmarking sur l’installation de votre application. Reportez-vous à l’article Benchmarking (voir le lien à la fin) pour connaître les étapes à suivre pour exécuter les outils d’évaluation courants sur les machines virtuelles Windows et Linux.

### <a name="optimize-iops-throughput-and-latency-at-a-glance"></a>Aperçu de l’optimisation des IOPS, du débit et de la latence

Le tableau ci-dessous récapitule les facteurs de performances et les étapes nécessaires à l’optimisation des IOPS, du débit et de la latence. Les sections qui suivent ce résumé décrivent en détail chaque facteur.

Pour plus d'informations sur les tailles de machines virtuelles et sur les E/S par seconde, le débit et la latence disponibles pour chaque type de machine virtuelle, consultez [Tailles des machines virtuelles dans Azure](sizes.md).

| | **D’OPÉRATIONS D’E/S PAR SECONDE** | **Débit** | **Latence** |
| --- | --- | --- | --- |
| **Exemple de scénario** |Application OLTP d’entreprise nécessitant un taux très élevé de transactions par seconde. |Application d’entreposage de données d’entreprise traitant de grandes quantités de données. |Applications en temps quasi-réel nécessitant une réponse instantanée aux demandes utilisateur, telles que les jeux en ligne. |
| **Facteurs de performances** | &nbsp; | &nbsp; | &nbsp; |
| **Taille d’E/S** |Une plus petite taille d’E/S génère un nombre d’E/S par seconde plus élevé. |Une plus grande taille d’E/S génère un débit plus élevé. | &nbsp;|
| **Taille de la machine virtuelle** |Utilisez une taille de machine virtuelle qui offre un nombre d’E/S par seconde supérieur aux besoins de votre application. |Utilisez une taille de machine virtuelle qui offre une limite de débit supérieure aux besoins de votre application. |Utilisez une taille de machine virtuelle qui offre une limite de mise à l’échelle supérieure aux besoins de votre application. |
| **Taille du disque** |Utilisez une taille de disque qui offre un nombre d’E/S par seconde supérieur aux besoins de votre application. |Utilisez une taille de disque qui offre une limite de débit supérieure aux besoins de votre application. |Utilisez une taille de disque qui offre une limite de mise à l’échelle supérieure aux besoins de votre application. |
| **Limites de mises à l’échelle des machines virtuelles et des disques** |La limite d’E/S par seconde de la taille de machine virtuelle choisie doit être supérieure au nombre total d’E/S par seconde générées par les disques de stockage qui lui sont associés. |La limite de débit par seconde de la taille de machine virtuelle choisie doit être supérieure au débit total généré par les disques de stockage premium qui lui sont associés. |Les limites de mise à l’échelle de la taille de machine virtuelle choisie doivent être supérieures aux limites totales de mise à l’échelle des disques de stockage premium qui lui sont associés. |
| **Mise en cache du disque** |Activez le cache en lecture seule sur les disques de stockage premium avec des opérations de lecture intensives pour obtenir un taux d’E/S en lecture par seconde plus élevé. | &nbsp; |Activez le cache en lecture seule sur les disques de stockage premium avec des opérations de lecture intensives pour obtenir de très faibles latences en lecture. |
| **Entrelacement de disques** |Utilisez plusieurs disques et entrelacez-les pour augmenter la limite combinée de débit et d’E/S par seconde. La limite combinée par machine virtuelle doit être supérieure aux limites combinées des disques premium associés. | &nbsp; | &nbsp; |
| **Taille de l’entrelacement** |Taille d’entrelacement plus petite pour les petits schémas d’E/S aléatoires associés aux applications OLTP. Utilisez par exemple une taille d’entrelacement de 64 Ko pour les applications OLTP sous SQL Server. |Plus grande taille d’entrelacement pour les grands schémas d’E/S séquentiels associés aux applications d’entrepôt de données. Utilisez par exemple une taille d’entrelacement de 256 Ko pour les applications d’entrepôt de données sous SQL Server. | &nbsp; |
| **Traitement multithread** |Utilisez le traitement multithread pour envoyer un plus grand nombre de demandes à Premium Storage et augmenter ainsi le débit et le nombre d’E/S par seconde. Sur SQL Server, par exemple, définissez une valeur MAXDOP élevée pour allouer plus de ressources processeur à SQL Server. | &nbsp; | &nbsp; |
| **Profondeur de file d’attente** |Une plus grande profondeur de file d’attente génère un nombre plus élevé d’E/S par seconde. |Une plus grande profondeur de file d’attente génère un débit plus élevé. |Une plus petite profondeur de file d’attente réduit les latences. |

## <a name="nature-of-io-requests"></a>Nature des demandes d’E/S

Une demande d’E/S représente une unité d’opération d’entrée/sortie exécutée par votre application. Identifier la nature des demandes d’E/S (aléatoires ou séquentielles, en lecture ou en écriture, petites ou grandes) vous permet de déterminer les exigences de performances de votre application. Il est important de comprendre la nature des demandes d’E/S afin de prendre les bonnes décisions lors de la conception de votre infrastructure d’applications. Pour bénéficier des meilleures performances possibles, veillez à ce que les E/S soient distribuées de façon uniforme.

La taille des E/S compte parmi les facteurs les plus importants. La taille des E/S correspond à la taille de la demande d’opération d’entrée/sortie générée par votre application. La taille des E/S a un impact significatif sur les performances, en particulier sur le nombre d’E/S par seconde et sur la bande passante, que l’application sera en mesure de délivrer. La formule suivante montre la relation entre le nombre d’IOPS, la taille des E/S et la bande passante ou le débit.  
    ![Diagramme montrant l’équation IOPS et taille d’E/S correspondant au débit.](media/premium-storage-performance/image1.png)

Certaines applications vous permettent de modifier leur taille d’E/S, contrairement à d’autres. Par exemple, SQL Server détermine lui-même la taille d’E/S optimale et ne fournit aux utilisateurs aucun dispositif permettant de la modifier. De son côté, Oracle propose un paramètre appelé [DB\_BLOCK\_SIZE](https://docs.oracle.com/cd/B19306_01/server.102/b14211/iodesign.htm#i28815) que vous pouvez utiliser pour configurer la taille de la demande d’E/S de la base de données.

Si vous utilisez une application qui ne permet pas de modifier la taille d’E/S, suivez les indications de cet article pour optimiser les indicateurs de performances qui correspondent le mieux à votre application. Par exemple,

* Une application OLTP génère des millions de demandes d’E/S petites et aléatoires. Pour gérer ces types de demandes d’E/S, vous devez concevoir votre infrastructure d’applications de manière à augmenter le nombre d’IOPS.  
* Une application d’entrepôt de données génère d’importantes demandes d’E/S séquentielles. Pour gérer ces types de demandes d’E/S, vous devez concevoir votre infrastructure d’applications de manière à augmenter la bande passante ou le débit.

Si vous utilisez une application qui vous permet de modifier la taille d’E/S, utilisez cette règle générale tout en respectant les autres instructions de performances.

* Une plus petite taille d’E/S pour générer un nombre d’E/S par seconde plus élevé. Par exemple, 8 Ko pour une application OLTP.  
* Une plus grande taille d’E/S pour augmenter la bande passante ou le débit. Par exemple, 1 024 Ko pour une application d’entrepôt de données.

Voici un exemple de calcul du nombre d’E/S par seconde et du débit ou de la bande passante pour votre application. Supposons une application utilisant un disque P30. Un disque P30 peut atteindre au maximum 5 000 E/S par seconde et 200 Mo de débit/bande passante par seconde. À présent, si votre application a besoin d’utiliser le nombre maximal d’E/S par seconde du disque P30 alors que vous utilisez une taille d’E/S inférieure à 8 Ko, vous devriez obtenir 40 Mo de bande passante par seconde. En revanche, si votre application requiert la valeur maximale de débit et de bande passante à partir du disque P30 et que vous utilisez une taille d’E/S supérieure à 1 024 Ko, vous obtiendrez seulement 200 E/S par seconde. Vous devez donc ajuster la taille d’E/S pour qu’elle réponde aux besoins de votre application tant en terme d’E/S que de débit et de bande passante. Le tableau suivant récapitule les différentes tailles d’E/S en indiquant le nombre d’IOPS et le débit correspondants dans le cas d’un disque P30.

| Spécification de l’application | Taille des E/S | E/S par seconde | Débit/bande passante |
| --- | --- | --- | --- |
| Nb max. d’E/S par seconde |8 Ko |5 000 |40 Mo par seconde |
| Débit max. |1 024 Ko |200 |200 Mo par seconde |
| Débit max + nb élevé d’E/S par seconde |64 Ko |3 200 |200 Mo par seconde |
| Nb max d’E/S par seconde + débit élevé |32 Ko |5 000 |160 Mo par seconde |

Pour obtenir une valeur d’E/S par seconde et de bande passante supérieure à la valeur maximale d’un seul disque de stockage premium, utilisez plusieurs disques premium entrelacés ensemble. Par exemple, entrelacez deux disques P30 pour obtenir un total de 10 000 E/S par seconde ou un débit combiné de 400 Mo par seconde. Comme nous l’expliquons dans la section suivante, vous devez utiliser une taille de machine virtuelle qui prend en charge la valeur combinée d’E/S par seconde et de débit du disque.

> [!NOTE]
> lorsque vous augmentez les E/S par seconde ou le débit, l’autre valeur augmente également ; assurez-vous de ne pas franchir les limites de débit ou d’E/S par seconde du disque ou de la machine virtuelle lorsque vous augmentez l’une de ces valeurs.

Pour évaluer les effets de la taille des E/S sur les performances de l’application, vous pouvez exécuter des outils d’évaluation sur votre machine virtuelle et sur vos disques. Créez plusieurs séries de tests et utilisez une taille d’E/S différente pour chaque exécution afin d’en déterminer l’impact. Pour plus de détails, reportez-vous à l’article Benchmarking, dont un lien est fourni à la fin.

## <a name="high-scale-vm-sizes"></a>Tailles des machines virtuelles à grande échelle

Lorsque vous commencez la conception d’une application, l’une des premières choses à faire est de choisir une machine virtuelle qui hébergera votre application. Premium Storage est fourni avec des tailles de machine virtuelle à grande échelle capables d’exécuter des applications qui requièrent une plus grande puissance de calcul et de hautes performances d’E/S du disque local. Ces machines virtuelles se caractérisent par des processeurs plus rapides, un rapport mémoire-cœur plus élevé et l’utilisation d’un disque SSD comme disque local. Les machines virtuelles de la série DS et GS sont des exemples de machines virtuelles à grande échelle prenant en charge le stockage Premium.

Ces machines virtuelles sont disponibles en différentes tailles, avec un nombre de cœurs de processeur, une mémoire, un système d’exploitation et une taille de disque temporaire différents. Chaque taille de chaque machine virtuelle possède également un nombre maximal de disques de données que vous pouvez attacher à la machine virtuelle. Par conséquent, la taille de machine virtuelle choisie affectera la quantité de traitement, de mémoire et de capacité de stockage disponible pour votre application. Elle affecte également le coût de traitement et de stockage. Par exemple, voici les spécifications de la plus grande taille de machine virtuelle dans une série DS et une série GS :

| Taille de la machine virtuelle | Cœurs d’unité centrale | Mémoire | Tailles du disque de la machine virtuelle | Bande passante disques de données | Taille du cache | E/S par seconde | Limites d’E/S du cache de bande passante |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_DS14 |16 |112 Go |OS = 1023 Go <br> SSD local = 224 Go |32 |576 Go |50 000 E/S par seconde <br> 512 Mo par seconde |4 000 E/S par seconde et 33 Mo par seconde |
| Standard_GS5 |32 |448 Go |OS = 1023 Go <br> SSD local = 896 Go |64 |4 224 Go |80 000 E/S par seconde <br> 2 000 Mo par seconde |5 000 E/S par seconde et 50 Mo par seconde |

Pour afficher la liste complète de toutes les tailles de machines virtuelles Azure disponibles, consultez [Tailles des machines virtuelles dans Azure](sizes.md) ou . Choisissez une taille de machine virtuelle capable de s’adapter aux exigences de performances souhaitées de votre application. En outre, prenez en compte les considérations suivantes lors du choix de tailles de machine virtuelle.

*Limites de mise à l’échelle*  
Les limites d’E/S par seconde par machine virtuelle et par disque sont différentes et indépendantes les unes des autres. Assurez-vous que l’application génère un nombre d’E/S par seconde dans les limites de la machine virtuelle et des disques premium qui lui sont associés. Dans le cas contraire, les performances de l’application seront limitées.

Par exemple, supposons une application exigeant un maximum de 4 000 E/S par seconde. Pour atteindre cette valeur, vous devez configurer un disque P30 sur une machine virtuelle DS1. Le disque P30 peut fournir jusqu’à 5 000 E/S par seconde. En revanche, la machine virtuelle DS1 est limitée à 3 200 E/S par seconde. Par conséquent, les performances de l’application seront contraintes par la limite de la machine virtuelle à 3 200 E/S par seconde, ce qui affectera les performances. Pour éviter cette situation, choisissez une taille de machine virtuelle et de disque qui satisfont toutes deux aux exigences de l’application.

*Coût d’exploitation*  
Dans de nombreux cas, il est possible que le coût global d’exploitation lié à l’utilisation de Premium Storage soit inférieur à celui associé à l’utilisation d’un stockage Standard.

Imaginez par exemple une application nécessitant 16 000 E/S par seconde. Pour atteindre ces performances, vous aurez besoin d’une machine virtuelle IaaS Azure Standard\_D14, qui peut délivrer 16 000 IOPS maximum en utilisant 32 disques de stockage standard de 1 To. Chaque disque de stockage standard de 1 To peut atteindre un maximum de 500 IOPS. Le coût mensuel estimé de cette machine virtuelle sera de 1 570 $. Le coût mensuel de 32 disques de stockage standard sera de 1 638 $. Le coût total mensuel estimé sera donc de 3 208 $.

Toutefois, si vous aviez hébergé la même application sur Premium Storage, vous auriez besoin d’une plus petite taille de machine virtuelle et de moins de disques de stockage premium, ce qui réduirait le coût global. Une machine virtuelle Standard\_StandardDS13 peut couvrir les 16 000 E/S par seconde requis à l’aide de quatre disques P30. La machine virtuelle DS13 délivre 25 600 E/S par seconde au maximum et chaque disque P30 délivre 5 000 E/S par seconde. Globalement, cette configuration peut atteindre 5 000 x 4 = 20 000 E/S par seconde. Le coût mensuel estimé de cette machine virtuelle sera de 1 003 $. Le coût mensuel de quatre disques de stockage premium P30 sera de 544,34 $. Le coût total mensuel estimé sera donc de 1 544 $.

Le tableau ci-dessous résume la répartition des coûts de ce scénario pour un stockage Standard et Premium.

| &nbsp; | **Standard** | **Premium** |
| --- | --- | --- |
| **Coût de la machine virtuelle par mois** |1 570,58 $ (Standard\_StandardD14) |1 003,66 $ (Standard\_StandardDS13) |
| **Coût des disques par mois** |1 638,40 $ (32 disques de 1 To) |544,34 $ (4 disques P30) |
| **Coût global par mois** |3 208,98 $ |1 544,34 $ |

*Distributions Linux*  

Avec Azure Premium Storage, vous obtenez le même niveau de performances pour les machines virtuelles exécutant Windows et Linux. Nous prenons en charge diverses distributions Linux, dont vous trouverez la liste complète [ici](linux/endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Il est important de noter que les différentes distributions sont mieux adaptées à différents types de charges de travail. Vous verrez différents niveaux de performances selon la distribution sur laquelle votre charge de travail est exécutée. Testez les distributions Linux avec votre application et choisissez celle qui vous convient le mieux.

Lorsque vous exécutez Linux avec Premium Storage, vérifiez les dernières mises à jour sur les pilotes requis pour garantir de meilleures performances.

## <a name="premium-storage-disk-sizes"></a>Tailles de disques Stockage Premium

Le stockage Premium Azure offre une diversité de tailles et vous permet de choisir celle qui répond le mieux à vos besoins. Chaque taille de disque a une limite de mise à l’échelle bien spécifique pour le nombre d’IOPS, la bande passante et le stockage. Choisissez la taille de disque Premium Storage adaptée aux exigences de l’application et à la taille de machine virtuelle à grande échelle. Le tableau ci-dessous présente les tailles de disque et leurs fonctionnalités. Les tailles P4, P6, P15, P60, P70 et P80 ne sont actuellement prises en charge que par la fonctionnalité Disques managés.

[!INCLUDE [disk-storage-premium-ssd-sizes](../../includes/disk-storage-premium-ssd-sizes.md)]

Le nombre de disques que vous choisissez dépend de la taille de disque choisie. Vous pouvez utiliser un seul disque P50 ou plusieurs disques P10 pour répondre aux besoins de votre application. Tenez compte des remarques ci-dessous pour faire votre choix.

*Limites de mise à l’échelle (E/S par seconde et débit)*  
Les limites d’E/S par seconde et de débit de chaque taille de disque Premium sont différentes et indépendantes des limites de mise à l’échelle de la machine virtuelle. Assurez-vous que le nombre total d’E/S par seconde et le débit total des disques respectent bien les limites de mise à l’échelle de la taille de machine virtuelle choisie.

Imaginez par exemple que votre application requiert un débit maximum de 250 Mo/s et que vous utilisez une machine virtuelle DS4 avec un seul disque P30. La machine virtuelle DS4 peut délivrer un débit maximum de 256 Mo/s. Cependant, un disque P30 a une limite de débit de 200 Mo par seconde. De ce fait, l’application sera contrainte à 200 Mo/s en raison de la limite du disque. Pour contourner cette limite, configurez plusieurs disques de données sur la machine virtuelle ou redimensionnez vos disques (de P40 à P50).

> [!NOTE]
> Les lectures traitées par le cache n’étant pas incluses dans les E/S et le débit de disque, elles ne sont pas soumises aux limites de disque. Le cache possède sa propre limite de débit et d’E/S pour chaque machine virtuelle.
>
> Par exemple, les lectures et écritures sont initialement de 60 Mo/s et 40 Mo/s respectivement. Au fil du temps, le cache augmente et sert de plus en plus de lectures. Vous pouvez alors obtenir un plus grand débit en écriture à partir du disque.

*Nombre de disques*  
Déterminez le nombre de disques dont vous aurez besoin en évaluant les besoins de l’application. Chaque taille de chaque machine virtuelle est limitée quant au nombre de disques que vous pouvez lui associer. En règle générale, ce nombre est égal à deux fois le nombre de cœurs. Assurez-vous que la taille de machine virtuelle que vous choisissez est capable de prendre en charge le nombre de disques nécessaires.

N’oubliez pas les disques Premium Storage délivrent des performances supérieures à celles des disques de stockage Standard. Par conséquent, si vous migrez votre application d’un stockage Standard à un stockage Premium à partir d’une machine virtuelle IaaS Azure, vous aurez probablement besoin de moins de disques premium pour atteindre des performances identiques ou supérieures pour votre application.

## <a name="disk-caching"></a>Mise en cache du disque

Les machines virtuelles à grande échelle qui exploitent Azure Premium Storage ont une technologie de mise en cache à plusieurs niveaux appelée BlobCache. BlobCache utilise à la fois la RAM de l’hôte et un SSD local pour la mise en cache. Ce cache est disponible pour les disques persistants Premium Storage et pour les disques locaux de la machine virtuelle. Par défaut, ce paramètre de cache est défini en lecture/écriture pour les disques du système d’exploitation et en lecture seule pour les disques de données hébergés sur Premium Storage. Lorsque la mise en cache du disque est activée sur les disques Premium Storage, les machines virtuelles à grande échelle peuvent atteindre des niveaux de performances extrêmement élevés qui dépassent les performances du disque sous-jacent.

> [!WARNING]
> La mise en cache du disque n’est pas prise en charge pour les disques 4 Tio et supérieurs. Si plusieurs disques sont attachés à votre machine virtuelle, chaque disque d’une taille inférieure à 4 Tio prend en charge la mise en cache.
>
> La modification du paramètre de cache d’un disque Azure détache et rattache le disque cible. S’il s’agit du disque du système d’exploitation, la machine virtuelle redémarre. Arrêtez toutes les applications et services qui risquent d’être affectés par cette indisponibilité avant de modifier le paramètre de cache du disque. Le non-respect de ces recommandations peut entraîner une altération des données.

Pour en savoir plus sur le fonctionnement de BlobCache, reportez-vous à l’article du blog interne [Azure Premium Storage](https://azure.microsoft.com/blog/azure-premium-storage-now-generally-available-2/) .

Il est important d’activer le cache sur le jeu de disques approprié. Le fait de devoir activer ou non la mise en cache du disque sur un disque premium dépendra du modèle de charge de travail que ce disque doit gérer. Le tableau ci-dessous répertorie les paramètres de cache par défaut pour les disques du système d’exploitation et les disques de données.

| **Type de disque** | **Paramètre de cache par défaut** |
| --- | --- |
| Disque de système d’exploitation |Lecture/écriture |
| Disque de données |Lecture seule |

Voici les paramètres de cache de disque recommandés pour les disques de données :

| **Paramètre de mise en cache du disque** | **Conditions recommandées d’utilisation de ce paramètre** |
| --- | --- |
| None |Configurer le cache hôte sur cette option pour les disques en lecture seule et les disques gourmands en écriture. |
| Lecture seule |Configurer le cache hôte en lecture seule pour les disques en lecture seule et les disques en lecture/écriture. |
| Lecture/écriture |Configurer le cache hôte en lecture/écriture uniquement si votre application gère correctement l’écriture des données mises en cache sur les disques persistants lorsque cela est nécessaire. |

*Lecture seule*  
En configurant une mise en cache en lecture seule sur des disques de données Premium Storage, vous pouvez obtenir une faible latence de lecture et obtenir de très hautes performances d’E/S et de débit en lecture pour votre application. Cela est dû à deux raisons :

1. Les lectures effectuées à partir du cache, qui se trouvent sur la mémoire virtuelle et sur le SSD local, sont beaucoup plus rapides que les lectures effectuées à partir du disque de données, qui réside sur Azure Blob Storage.  
1. Premium Storage ne tient pas compte des lectures traitées à partir du cache pour le calcul du nombre d’E/S par seconde et du débit du disque. Par conséquent, votre application est en mesure de délivrer de meilleures performances totales en termes d’E/S par seconde et de débit.

*Lecture/écriture*  
Par défaut, le cache en lecture/écriture est activé sur les disques du système d’exploitation. Nous avons récemment ajouté la prise en charge de la mise en cache en lecture/écriture sur les données des disques. Si vous utilisez la mise en cache en lecture/écriture, vous devez disposer d’un moyen approprié d’écrire les données du cache sur des disques persistants. Par exemple, SQL Server gère lui-même l’écriture de données mises en cache sur les disques de stockage persistants. L’utilisation d’un cache en lecture/écriture avec une application qui ne gère pas la persistance des données requises peut entraîner des pertes de données en cas de panne de la machine virtuelle.

*Aucun*  
Actuellement, l’option **Aucune** n’est prise en charge que sur les disques de données. Elle n’est pas prise en charge sur les disques du système d’exploitation. Si vous définissez l’option **Aucun** sur un disque de système d’exploitation, il est remplacé en interne et défini sur **Lecture seule**.

Par exemple, vous pouvez appliquer ces instructions à une instance SQL Server exécutée sur Premium Storage de la manière suivante :

1. Configurez le cache en lecture seule sur les disques de stockage Premium qui hébergent des fichiers de données.  
   a.  Les lectures rapides du cache réduisent le temps de requête de SQL Server puisque les pages de données sont récupérées à partir du cache bien plus rapidement que lorsque l’opération s’effectue directement à partir des disques de données.  
   b.  Le traitement des lectures à partir du cache signifie que les disques de données premium délivrent un débit supplémentaire. SQL Server peut utiliser ce débit supplémentaire pour la récupération d’un plus grand nombre de pages de données et d’autres opérations telles que la sauvegarde/restauration, les charges de traitement par lots et les reconstructions d’index.  
1. Choisissez l’option « Aucun » pour le cache sur les disques de stockage Premium qui hébergent des fichiers journaux.  
   a.  Les fichiers journaux ont principalement des opérations d’écriture intensives. Ils ne bénéficient donc pas du cache en lecture seule.

## <a name="optimize-performance-on-linux-vms"></a>Optimiser les performances sur les machines virtuelles Linux

Pour tous les disques SSD Premium ou disques Ultra, vous pourrez peut-être désactiver les « barrières » pour les systèmes de fichiers sur le disque afin d’améliorer les performances lorsque l’on sait qu’il n’existe aucun cache susceptible de perdre des données.  Si la mise en cache de disque Azure est définie sur ReadOnly ou Aucune, vous pouvez désactiver les barrières.  En revanche, si la mise en cache est définie sur ReadWrite, les barrières doivent rester activées pour garantir une durabilité d’écriture.  Les barrières sont généralement activées par défaut, mais vous pouvez les désactiver en appliquant l’une des méthodes suivantes, en fonction du type de système de fichiers :

* Pour **reiserFS**, utilisez l’option de montage barrier=none pour désactiver les barrières.  Pour activer explicitement les barrières, utilisez barrier=flush.
* Pour **ext3/ext4**, utilisez l’option de montage barrier=0 pour désactiver les barrières.  Pour activer explicitement les barrières, utilisez barrier=1.
* Pour **XFS**, utilisez l’option de montage nobarrier pour désactiver les barrières.  Pour activer explicitement les barrières, utilisez barrier.  Notez que dans les versions de noyau Linux ultérieures, la conception du système de fichiers XFS garantit toujours la durabilité, et la désactivation des barrières n’a aucun effet.  

## <a name="disk-striping"></a>Entrelacement de disques

Lorsqu’une machine virtuelle à grande échelle est connectée à plusieurs disques de stockage premium persistants, les disques peuvent être entrelacés ensemble pour agréger les E/S par seconde, la bande passante et la capacité de stockage.

Sous Windows, vous pouvez utiliser les espaces de stockage pour entrelacer les disques. Vous devez configurer une seule colonne pour chaque disque dans un pool. Dans le cas contraire, les performances globales du volume entrelacé peuvent être limitées, en raison d’une distribution inégale du trafic sur les disques.

Important : À l’aide de l’IU du Gestionnaire de serveur, vous pouvez définir un nombre maximal de 8 colonnes au total pour un volume agrégé par bandes. Si vous souhaitez attacher plus de huit disques, utilisez PowerShell pour créer le volume. PowerShell vous permet de définir un nombre de colonnes égal au nombre de disques. Par exemple, s’il existe 16 disques dans un agrégat unique, spécifiez 16 colonnes dans le paramètre *NumberOfColumns* de l’applet de commande *New-VirtualDisk*.

Sous Linux, utilisez l’utilitaire MDADM pour entrelacer les disques. Pour obtenir des instructions détaillées sur l’entrelacement de disques sous Linux, reportez-vous à [Configuration d’un RAID logiciel sous Linux](/previous-versions/azure/virtual-machines/linux/configure-raid).

*Taille de l’entrelacement*  
La taille d’entrelacement constitue un facteur important dans la configuration de l’entrelacement de disques. La taille d’entrelacement ou la taille de bloc représente la plus petite partie des données que l’application peut traiter sur un volume entrelacé. La taille d’entrelacement que vous configurez varie selon le type d’application et le modèle de demande associé. Si vous choisissez une taille d’entrelacement incorrecte, vous risquez de rencontrer un défaut d’alignement des E/S, ce qui conduirait à une dégradation des performances de votre application.

Par exemple, si une demande d’E/S générée par votre application est supérieure à la taille d’entrelacement de disque, le système de stockage écrira au-delà des limites d’unité de bande sur plusieurs disques. Au moment d’accéder aux données, il devra effectuer une recherche sur plusieurs unités de bande pour exécuter la demande. L’effet cumulatif de ce comportement peut entraîner une dégradation significative des performances. En revanche, si la taille de la demande d’E/S est inférieure à la taille d’entrelacement, et si ces E/S sont aléatoires par nature, les demandes d’E/S peuvent s’ajouter sur le même disque et provoquer un goulot d’étranglement responsable d’une dégradation des performances d’E/S.

Selon le type de charge de travail que votre application exécute, choisissez une taille d’entrelacement appropriée. Pour les petites demandes d’E/S aléatoires, utilisez une plus petite taille d’entrelacement. Pour de grandes demandes d’E/S séquentielles, utilisez une plus grande taille d’entrelacement. Découvrez les tailles d’entrelacement recommandées pour l’application exécutée sur Premium Storage. Pour SQL Server, configurez une taille d’entrelacement de 64 Ko pour les charges de travail OLTP et de 256 Ko pour les charges de travail d’entrepôt de données. Pour en savoir plus, consultez [Meilleures pratiques relatives aux performances de SQL Server dans Azure Virtual Machines](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md#disks-guidance)

> [!NOTE]
> vous pouvez entrelacer au maximum 32 disques de stockage premium sur une machine virtuelle DS et 64 disques de stockage premium sur une machine virtuelle GS.

## <a name="multi-threading"></a>Multithreading

Azure a conçu une plate-forme Premium Storage hautement parallèle. Par conséquent, une application multithread délivre de bien meilleures performances qu’une application à thread unique. Une application multithread répartit ses tâches sur plusieurs threads et augmente l’efficacité de son exécution en utilisant au maximum les ressources de machine virtuelle et de disque.

Par exemple, si votre application s’exécute sur une machine virtuelle monocœur utilisant deux threads, le processeur peut basculer entre les deux threads pour gagner en efficacité. En attendant l’exécution d’un thread sur une E/S de disque, le processeur peut basculer vers l’autre thread. De cette façon, deux threads peuvent accomplir bien plus qu’un seul thread. Si la machine virtuelle possède plusieurs cœurs, elle réduit davantage le temps d’exécution dans la mesure où chaque cœur peut exécuter des tâches en parallèle.

Vous n’êtes peut-être pas en mesure de modifier la façon dont une application du commerce implémente un seul thread ou plusieurs threads. Par exemple, SQL Server est capable de gérer plusieurs processeurs et plusieurs nœuds. SQL Server détermine cependant dans quelles conditions il utilisera un ou plusieurs threads pour traiter une requête. Il peut exécuter des requêtes et créer des index à l’aide du traitement multithread. Dans le cas d’une requête impliquant une fusion de tables volumineuses et un tri des données avant le renvoi des résultats à l’utilisateur, SQL Server utilisera probablement plusieurs threads. Toutefois, un utilisateur ne peut pas décider si SQL Server exécutera une requête à l’aide d’un ou plusieurs threads.

Certains paramètres de configuration peuvent être modifiés pour influencer ce traitement multithread ou parallèle d’une application. Dans le cas de SQL Server, par exemple, il s’agit de la configuration maximale de degré de parallélisme. Ce paramètre appelé MAXDOP permet de configurer le nombre maximal de processeurs que SQL Server peut utiliser lors d’un de traitement parallèle. Vous pouvez configurer MAXDOP pour des requêtes individuelles ou pour des opérations d’index. Cela vous sera utile si vous souhaitez équilibrer les ressources de votre système pour une application stratégique en termes de performances.

Supposons par exemple que votre application basée sur SQL Server exécute simultanément une grande requête et une opération d’index. Imaginons que vous souhaitiez que l’opération d’index soit plus performante que la requête volumineuse. Dans ce cas, vous pouvez spécifier une valeur MAXDOP de l’opération d’index supérieure à la valeur MAXDOP de la requête. SQL Server dispose ainsi d’un plus grand nombre de processeurs exploitables pour l’opération d’index par rapport au nombre de processeurs qu’il peut consacrer à la requête volumineuse. N’oubliez pas que vous n’avez aucun contrôle sur le nombre de threads que SQL Server utilisera pour chaque opération. Vous pouvez contrôler le nombre maximal de processeurs dédiés au traitement multithread.

En savoir plus sur [Degrés de parallélisme](/previous-versions/sql/sql-server-2008-r2/ms188611(v=sql.105)) dans SQL Server. Découvrez les paramètres qui influencent le traitement multithread dans votre application et leurs configurations pour optimiser les performances.

## <a name="queue-depth"></a>Profondeur de file d’attente

La profondeur, longueur ou taille de file d’attente correspond au nombre de demandes d’E/S en attente dans le système. La valeur de profondeur de file d’attente détermine le nombre d’opérations d’E/S que votre application peut aligner et qui seront traitées par les disques de stockage. Elle affecte les trois indicateurs de performance que nous avons abordés dans cet article, à savoir le nombre d’IOPS, le débit et la latence.

La profondeur de file d’attente et le traitement multithread sont étroitement liés. La valeur de profondeur de file d’attente indique dans quelle mesure l’application peut prendre en charge un traitement multithread. Si la profondeur de file d’attente est importante, l’application peut exécuter davantage d’opérations simultanément, autrement dit un meilleur traitement multithread. Si la profondeur de file d’attente est petite, même si l’application est de type multithread, le nombre de demandes alignées sera insuffisant pour permettre une exécution simultanée.

En règle générale, les applications du commerce ne vous permettent pas de modifier la profondeur de la file d’attente, car un paramétrage incorrect peut faire plus de mal que de bien. Les applications définissent la valeur de la profondeur de file d’attente appropriée pour obtenir des performances optimales. Il est toutefois important de comprendre ce concept afin de pouvoir résoudre les problèmes de performances que vous pouvez rencontrer avec votre application. Vous pouvez également observer les effets de la profondeur de file d’attente en exécutant les outils de benchmarking sur votre système.

Certaines applications fournissent des paramètres qui influencent la profondeur de file d’attente. Le paramètre MAXDOP (degré maximal de parallélisme) dans SQL Server, par exemple, est expliqué dans la section précédente. MAXDOP offre un moyen d’influencer la profondeur de file d’attente et le traitement multithread, sans pour autant modifier directement la valeur de profondeur de file d’attente de SQL Server.

*Grande profondeur de file d’attente*  
Une grande profondeur de file d’attente permet d’aligner davantage d’opérations sur le disque. Le disque peut anticiper la prochaine demande placée dans sa file d’attente. Il peut ainsi planifier les opérations à l’avance et les traiter dans un ordre optimal. Étant donné que l’application envoie davantage de requêtes sur le disque, ce dernier peut traiter un plus grand nombre d’E/S parallèles. Au final, l’application sera en mesure d’atteindre un taux supérieur d’E/S par seconde. Puisque l’application traite davantage de demandes, le débit total de l’application augmente également.

En général, une application peut atteindre un débit maximal avec 8-16 E/S en attente par disque connecté. Si la profondeur de file d’attente est de 1, l’application n’envoie (push) pas suffisamment d’E/S sur le système et traite moins de données pendant une période donnée. En d’autres termes, elle atteindra un débit inférieur.

Par exemple, dans SQL Server, une valeur MAXDOP de 4 pour une requête indique à SQL Server qu’il peut utiliser jusqu’à quatre cœurs pour exécuter la requête. SQL Server déterminera la meilleure valeur de profondeur de file d’attente ainsi que le nombre de cœurs à utiliser pour l’exécution de la requête.

*Profondeur de file d’attente optimale*  
Une valeur de profondeur de file d’attente très élevée présente aussi des inconvénients. Si la valeur de profondeur de file d’attente est trop importante, l’application tentera de générer un taux d’E/S par seconde très élevé. À moins que l’application repose sur des disques persistants associés à un nombre suffisant d’E/S par seconde, cela peut affecter sérieusement les latences d’application. La formule suivante montre la relation entre les IOPS, la latence et la profondeur de file d’attente.  
    ![Diagramme montrant l’équation IOPS et latence correspondant à la profondeur de file d’attente.](media/premium-storage-performance/image6.png)

Vous ne devez pas configurer la profondeur de file d’attente à une valeur élevée, mais plutôt à une valeur optimale, qui peut générer suffisamment d’E/S par seconde pour l’application sans affecter les latences. Par exemple, si la latence de l’application doit être de 1 milliseconde, la profondeur de file d’attente requise pour atteindre 5 000 E/S par seconde sera : PF = 5 000 x 0,001 = 5.

*Profondeur de file d’attente pour le volume entrelacé*  
Pour un volume entrelacé, conservez une profondeur de file d’attente suffisamment élevée de sorte que chaque disque dispose individuellement d’un pic de profondeur de file d’attente. Prenons l’exemple d’une application qui envoie (push) une profondeur de file d’attente de 2 dans un scénario d’entrelacement à quatre disques. Les deux demandes d’E/S seront transmises à deux disques et les deux disques restants seront inactifs. Vous devez par conséquent configurer la profondeur de file d’attente afin que tous les disques puissent être occupés. La formule ci-dessous montre comment déterminer la profondeur de file d’attente des volumes entrelacés.  
    ![Diagramme montrant l’équation Q D par disque et nombre de colonnes par volume correspondant à Q D du volume agrégé par bandes.](media/premium-storage-performance/image7.png)

## <a name="throttling"></a>Limitation

Azure Premium Storage configure la valeur spécifiée d’E/S par seconde et de débit en fonction des tailles de machine virtuelle et des tailles de disque que vous choisissez. Chaque fois que votre application tentera de dépasser les limites de ce que la machine virtuelle ou le disque peut gérer, Premium Storage lui imposera une limitation. Cette limitation se manifeste sous la forme d’une dégradation des performances de votre application, à savoir une latence plus élevée, un débit réduit ou un nombre inférieur d’IOPS. Sans cette limitation, votre application risquerait de planter en demandant plus que ses ressources ne lui permettent d’effectuer. Par conséquent, pour éviter les problèmes de performances associés à une limitation, veillez à toujours fournir suffisamment de ressources pour votre application. Tenez compte des explications données ci-dessus dans les sections relatives aux tailles de disque et aux tailles de machine virtuelle. Le benchmarking offre le meilleur moyen de déterminer les ressources dont vous aurez besoin pour héberger votre application.

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez évaluer votre disque, consultez les articles consacrés au benchmarking d'un disque :

* Pour Linux : [Créer un benchmark de votre application sur Stockage sur disque Azure](linux/disks-benchmarks.md)
* Pour Windows : [Benchmarking d'un disque](windows/disks-benchmarks.md).

En savoir plus sur les types de disque disponibles :

* Pour Linux : [Sélectionner un type de disque](disks-types.md)
* Pour Windows : [Sélectionner un type de disque](disks-types.md)

Pour les utilisateurs de SQL Server, consultez les articles relatifs aux meilleures pratiques de performances de SQL Server :

* [Meilleures pratiques relatives aux performances de SQL Server sur les machines virtuelles Azure](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md)
* [Azure Premium Storage provides highest performance for SQL Server in Azure VM (en anglais)](https://cloudblogs.microsoft.com/sqlserver/2015/04/23/azure-premium-storage-provides-highest-performance-for-sql-server-in-azure-vm/)