---
title: Références SKU pour SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Références SKU pour SAP HANA sur Azure (grandes instances).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: juergent
editor: ''
keywords: HLI, HANA, SKUs, S896, S224, S448, S672, Optane, SAP
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/21/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 886cd57b59bd4103ced9d496021e54ab0bdc99ad
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97723284"
---
# <a name="available-skus-for-hana-large-instances"></a>Références SKU disponibles pour les grandes instances HANA

Le service SAP HANA sur Azure (Grandes instances) basé sur des tampons Révision 3 uniquement est disponible dans plusieurs configurations dans les régions suivantes :

- Australie Est
- Sud-Australie Est
- Japon Est
- OuJapon Est

Le service SAP HANA sur Azure (Grandes instances) basé sur des tampons Révision 4 est disponible dans plusieurs configurations dans les régions suivantes :

- USA Ouest 2
- USA Est

Service BareMetal Infrastructure (certifié pour les charges de travail SAP HANA) basé sur les horodatages Révision 4.2. Disponible avec plusieurs configurations dans les régions Azure suivantes :
- Europe Ouest
- Europe Nord
- USA Est 2
- États-Unis - partie centrale méridionale




Liste des Grandes instances Azure disponibles qui proposent des listes telles que les suivantes.

> [!IMPORTANT]
> Soyez attentif à la première colonne qui représente l’état de la certification HANA pour chaque type de grande instance dans la liste. La colonne doit être corrélée avec le [répertoire matériel SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) pour les références (SKU) Azure qui commencent par la lettre **S**



| Certifié SAP HANA | Modèle | Mémoire totale | Mémoire DRAM | Mémoire Optane | Stockage | Disponibilité |
| --- | --- | --- | --- | --- | --- | --- |
| YES <br />[OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2185), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2265) | SAP HANA sur Azure S96<br /> \- 2 x processeurs Intel® Xeon® E7-8890 v4 <br /> 48 cœurs et 96 threads d’UC |  768 Go | 768 Go | --- | 3 TO | Disponible |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2186), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2269) | SAP HANA sur Azure S224<br /> \- 4 x processeurs Intel® Xeon® Platinum 8276 <br /> 112 cœurs et 224 threads de processeur |  3 TO | 3 TO | --- | 6,3 To | Disponible |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2297) | SAP HANA sur Azure S224m<br /> \- 4 x processeurs Intel® Xeon® Platinum 8276 <br /> 112 cœurs et 224 threads de processeur |  6,0 To | 6,0 To | --- | 10,5 To | Disponible |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2381) | SAP HANA sur Azure S224om<br /> \- 4 x processeurs Intel® Xeon® Platinum 8276 <br /> 112 cœurs et 224 threads de processeur | 6,0 To |  3 TO |  3 TO | 10,5 To | Disponible |
| Non | SAP HANA sur Azure S224oo<br /> \- 4 x processeurs Intel® Xeon® Platinum 8276 <br /> 112 cœurs et 224 threads de processeur | 4,5 To |  1,5 To |  3 TO | 8,4 To | Disponible |
| Non | SAP HANA sur Azure S224ooo<br /> \- 4 x processeurs Intel® Xeon® Platinum 8276 <br /> 112 cœurs et 224 threads de processeur | 7,5 To |  1,5 To |  6,0 To | 12,7 To | Disponible |
| Non | SAP HANA sur Azure S224oom<br /> \- 4 x processeurs Intel® Xeon® Platinum 8276 <br /> 112 cœurs et 224 threads de processeur | 9,0 To |  3 TO |  6,0 To | 14,8 To | Disponible |
| YES <br />[OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1983), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2268) | SAP HANA sur Azure S384<br /> \- 8 x processeurs Intel® Xeon® E7-8890 v4<br /> 192 cœurs et 384 threads d’UC |  4 TO | 4 TO | --- | 16 TO | Disponible |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2080) | SAP HANA sur Azure S384m<br /> \- 8 x processeurs Intel® Xeon® E7-8890 v4<br /> 192 cœurs et 384 threads d’UC |  6,0 To | 6,0 To | --- | 18 To |  Disponible  |
| YES <br />[OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1984), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2267) | SAP HANA sur Azure S384xm<br /> \- 8 x processeurs Intel® Xeon® E7-8890 v4<br /> 192 cœurs et 384 threads d’UC |  8,0 To | 8,0 To | --- | 22 To | Disponible |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2411), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2378) | SAP HANA sur Azure S448<br /> \- 8 x processeurs Intel® Xeon® Platinum 8276 <br /> 224 cœurs et 448 threads de processeur | 6,0 To |  6,0 To |  --- | 10,5 To | Disponible (Rev 4 uniquement) |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2410), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2377) | SAP HANA sur Azure S448m<br /> \- 8 x processeurs Intel® Xeon® Platinum 8276 <br /> 224 cœurs et 448 threads de processeur | 12,0 To |  12,0 To |  --- | 18,9 To | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S448oo<br /> \- 8 x processeurs Intel® Xeon® Platinum 8276 <br /> 224 cœurs et 448 threads de processeur | 9,0 To |  3 TO |  6,0 To | 14,8 To  | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S448om<br /> \- 8 x processeurs Intel® Xeon® Platinum 8276 <br /> 224 cœurs et 448 threads de processeur | 12,0 To |  6,0 To |  6,0 To | 18,9 To  | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S448ooo<br /> \- 8 x processeurs Intel® Xeon® Platinum 8276 <br /> 224 cœurs et 448 threads de processeur | 15,0 To |  3 TO |  12,0 To | 23,2 To  | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S448oom<br /> \- 8 x processeurs Intel® Xeon® Platinum 8276 <br /> 224 cœurs et 448 threads de processeur | 18,0 To |  6,0 To |  12,0 To | 27,4 To  | Disponible (Rev 4 uniquement) |
| YES <br /> [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2049) | SAP HANA sur Azure S576m<br /> \- 12 x processeurs Intel® Xeon® E7-8890 v4<br /> 288 cœurs et 576 threads d’UC |  12,0 To | 12,0 To | --- | 28 To | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S576xm<br /> \- 12 x processeurs Intel® Xeon® E7-8890 v4<br /> 288 cœurs et 576 threads d’UC |  18,0 To | 18.0 | --- |  41 To | Disponible |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2409), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2380) | SAP HANA sur Azure S672<br /> \- 12 x processeurs Intel® Xeon® Platinum 8276 <br /> 336 cœurs et 672 threads de processeur | 9,0 To |  9,0 To |  --- | 14,7 To | Disponible (Rev 4 uniquement) |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2408), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2379) | SAP HANA sur Azure S672m<br /> \- 12 x processeurs Intel® Xeon® Platinum 8276 <br /> 336 cœurs et 672 threads de processeur | 18,0 To |  18,0 To |  --- | 27,4 To | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S672oo<br /> \- 12 x processeurs Intel® Xeon® Platinum 8276 <br /> 336 cœurs et 672 threads de processeur | 13,5 To |  4,5 To |  9,0 To | 21,1 To  | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S672om<br /> \- 12 x processeurs Intel® Xeon® Platinum 8276 <br /> 336 cœurs et 672 threads de processeur | 18,0 To |  9,0 To |  9,0 To | 27,4 To  | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S672ooo<br /> \- 12 x processeurs Intel® Xeon® Platinum 8276 <br /> 336 cœurs et 672 threads de processeur | 22,5 To |  4,5 To |  18,0 To | 33,7 To  | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S672oom<br /> \- 12 x processeurs Intel® Xeon® Platinum 8276 <br /> 336 cœurs et 672 threads de processeur | 27,0 To |  9,0 To |  18,0 To | 40,0 To  | Disponible (Rev 4 uniquement) |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1985) | SAP HANA sur Azure S768m<br /> \- 16 x processeurs Intel® Xeon® E7-8890 v4<br /> 384 cœurs et 768 threads d’UC |  16,0 To | 16,0 To | -- | 36 To | Disponible |
| Non | SAP HANA sur Azure S768xm<br /> \- 16 x processeurs Intel® Xeon® E7-8890 v4<br /> 384 cœurs et 768 threads d’UC |  24,0 To | 24,0 To | --- | 56 To | Disponible |
|  YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2407), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2376)  | SAP HANA sur Azure S896<br /> \- 16 processeurs Intel® Xeon® Platinum 8276 <br /> 448 cœurs et 896 threads de processeur | 12,0 To |  12,0 To |  --- | 18,9 To | Disponible (Rev 4 uniquement) |
| YES <br /> [OLAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/#/solutions?filters=iaas;ve:24&id=s:2406), [OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=2328) | SAP HANA sur Azure S896m<br /> \- 16 processeurs Intel® Xeon® Platinum 8276 <br /> 448 cœurs et 896 threads de processeur | 24,0 To | 24,0 To | -- | 35,8 To | Disponible |
| Non | SAP HANA sur Azure S896oo<br /> \- 16 processeurs Intel® Xeon® Platinum 8276 <br /> 448 cœurs et 896 threads de processeur | 18,0 To |  6,0 To |  12,0 To | 27,4 To  | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S896om<br /> \- 16 processeurs Intel® Xeon® Platinum 8276 <br /> 448 cœurs et 896 threads de processeur | 24,0 To |  12,0 To |  12,0 To | 35,8 To  | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S896ooo<br /> \- 16 processeurs Intel® Xeon® Platinum 8276 <br /> 448 cœurs et 896 threads de processeur | 30,0 To |  6,0 To |  24,0 To | 44,3 To  | Disponible (Rev 4 uniquement) |
| Non | SAP HANA sur Azure S896oom<br /> \- 16 processeurs Intel® Xeon® Platinum 8276 <br /> 448 cœurs et 896 threads de processeur | 36,0 To |  12,0 To |  24,0 To | 52,7 To  | Disponible (Rev 4 uniquement) |
| YES <br />[OLTP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure&recordid=1986) | SAP HANA sur Azure S960m<br /> \- 20 x processeurs Intel® Xeon® E7-8890 v4<br /> 480 cœurs et 960 threads d’UC |  20,0 To | 20,0 To | -- | 46 To | Disponible (Rev 4 uniquement) |


- Cœurs d’unité centrale = somme des cœurs d’UC non multithreads dérivée de la somme des processeurs de l’unité de serveur.
- Threads d’UC = somme des threads de calcul fournis par les cœurs d’UC multithreads dérivée de la somme des processeurs de l’unité de serveur. La plupart des unités sont configurées par défaut pour utiliser la technologie Hyper-Threading.
- D’après les recommandations du fournisseur, S768xm et S960m ne sont pas configurées en vue d’utiliser la technologie Hyper-Threading pour l’exécution de SAP HANA.


> [!IMPORTANT]
> Les références SKU suivantes, bien que toujours prises en charge, ne peuvent plus être achetées : S72, S72m, S144, S144m, S192 et S192m 

Les configurations spécifiques choisies dépendent de la charge de travail, des ressources d’UC et de la mémoire souhaitée. La charge de travail OLTP est en mesure d’utiliser des références SKU optimisées pour les charges de travail OLAP. 

Deux classes de matériel différentes divisent les références SKU comme suit :

- Les références S72, S72m, S96, S144, S144m, S192, S192m, S192xm, S224 et S224m, S224oo, S224om, S224ooo, S224oom sont nommées références SKU de « Classe de type I ».
- Toutes les autres sont nommées références SKU de « Classe de type II ».
- Si vous êtes intéressé par des références SKU qui ne figurent pas encore dans le répertoire matériel SAP, contactez votre équipe de compte Microsoft pour obtenir plus d’informations. 


Un tampon de grande instance HANA complet n’est pas exclusivement alloué à l’utilisation d’un client unique. Cela s’applique également aux racks de ressources de calcul et de stockage connectés par le biais d’une structure réseau déployée dans Azure. L’infrastructure de grande instance HANA, par exemple Azure, déploie plusieurs &quot;abonnés&quot; client isolés les uns des autres selon les trois niveaux suivants :

- **Réseau** : isolation par le biais des réseaux virtuels dans le tampon de grande instance HANA.
- **Stockage** : isolation par le biais de machines virtuelles auxquelles des volumes de stockage sont assignés et qui isolent les volumes de stockage entre les abonnés.
- **Calcul** : assignation dédiée des unités de serveur à un seul abonné. Aucun partitionnement matériel ou logiciel des unités de serveur. Aucun partage d’une unité de serveur ou hôte unique entre les locataires. 

Les déploiements d’unités de grande instance HANA entre les différents abonnés ne sont pas visibles entre eux. Les unités de grande instance HANA déployées dans différents abonnés ne peuvent pas communiquer directement entre elles au niveau du tampon de grande instance HANA. Seules les unités de grande instance HANA d’un abonné unique peuvent communiquer entre elles au niveau du tampon de grande instance HANA.

Un abonné déployé dans le tampon de grande instance est assigné à un abonnement Azure dans le cadre de la facturation. En termes de réseau, il est accessible à partir des réseaux virtuels d’autres abonnements Azure dans la même inscription Azure. Si vous effectuez un déploiement avec un autre abonnement Azure dans la même région Azure, vous pouvez également demander à obtenir un locataire de grande instance HANA distinct.

Il existe des différences importantes entre l’exécution de SAP HANA sur la grande instance HANA et l’exécution de SAP HANA sur des machines virtuelles déployées dans Azure :

- Il n’existe aucune couche Virtualisation pour SAP HANA sur Azure (grandes instances). Vous bénéficiez des performances du matériel nu sous-jacent.
- Contrairement à Azure, le serveur SAP HANA sur Azure (grandes instances) est dédié à un client spécifique. Il est impossible qu’une unité de serveur ou un hôte fasse l’objet d’un partitionnement matériel ou logiciel. Par conséquent, une unité de grande instance HANA est assignée dans son intégralité à un abonné, et donc à vous en tant que client. Un redémarrage ou un arrêt du serveur n’entraîne pas automatiquement le déploiement du système d’exploitation et de SAP HANA sur un autre serveur. (Pour les références SKU de classe de type I, la seule exception est lorsqu’un serveur rencontre des problèmes et qu’un redéploiement doit être effectué sur un autre serveur.)
- Contrairement à Azure, où les types de processeurs hôtes sont sélectionnés en fonction du meilleur rapport prix/performances, les types de processeurs choisis pour SAP HANA sur Azure (grandes instances) sont les plus performants de la gamme de processeurs Intel E7v3 et E7v4.

**Étapes suivantes**
- Voir [Dimensionnement de HLI](hana-sizing.md)
