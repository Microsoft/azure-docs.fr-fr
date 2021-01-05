---
title: Supervision de SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Superviser SAP HANA sur Azure (grandes instances).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1a8d6fc6acb33af0c36978aa79dc90aeac3cf596
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966749"
---
# <a name="how-to-monitor-sap-hana-large-instances-on-azure"></a>Comment surveiller SAP HANA (grandes instances) sur Azure

SAP HANA sur Azure (grandes instances) est semblable aux autres déploiements IaaS, dans le sens où vous devez surveiller les actions du système d’exploitation et de l’application, et comment les applications consomment les ressources suivantes :

- UC
- Mémoire
- Bande passante réseau
- Espace disque

Avec les machines virtuelles Azure, vous devez déterminer si les classes de ressources ci-dessus sont suffisantes, ou si elles s’épuisent. Voici plus de détails sur chacune des classes :

**Consommation des ressources du processeur :** le ratio défini par SAP pour certaines charges de travail sur HANA est appliqué pour vous assurer que les ressources du processeur disponibles sont suffisantes pour travailler avec les données stockées en mémoire. Toutefois, il peut arriver que HANA consomme un grand nombre de requêtes d’exécution du processeur en raison d’index manquants ou de problèmes similaires. Cela signifie que vous devez surveiller la consommation des ressources du processeur de l’unité de grande instance HANA, ainsi que les ressources du processeur consommées par les services HANA spécifiques.

**Consommation de mémoire :** sa surveillance est importante dans HANA, ainsi qu’en dehors de HANA sur l’unité. Dans HANA, surveillez comment les données consomment la mémoire allouée à HANA, afin de respecter les instructions de dimensionnement de SAP. Vous devez également surveiller la consommation de mémoire au niveau de la grande instance afin de vous assurer que les logiciels supplémentaires non HANA installés ne consomment pas trop de mémoire, et ne concurrencent donc pas HANA en termes de mémoire.

**Bande passante réseau :** la passerelle de réseau virtuel Azure est limitée en bande passante pour le déplacement des données dans le réseau virtuel Azure. Il est donc utile de surveiller les données reçues par toutes les machines virtuelles Azure au sein d’un réseau virtuel pour déterminer s’il vous reste de la marge par rapport aux limites définies par la référence SKU de la passerelle Azure choisie. Sur l’unité de grande instance HANA, il est pertinent de surveiller également le trafic réseau entrant et sortant, et de suivre les volumes qui sont gérés au fil du temps.

**Espace disque :** la consommation d’espace disque augmente généralement au fil du temps. Les causes les plus courantes sont les suivantes : augmentation du volume de données, exécution de sauvegardes des journaux de transaction, stockage des fichiers de trace et création d’instantanés du stockage. Par conséquent, il est important de surveiller l’espace disque et de gérer l’espace disque associé à l’unité de grande instance HANA.

Pour les **références SKU de Type II** des grandes instances HANA, le serveur est fourni avec des outils de diagnostic système préchargés. Vous pouvez utiliser ces outils de diagnostic pour vérifier l’intégrité du système. Exécutez la commande suivante pour générer le fichier journal de vérification d’intégrité sur /var/log/health_check.
```
/opt/sgi/health_check/microsoft_tdi.sh
```
Lorsque vous contactez l’équipe de Support Microsoft pour résoudre un problème, vous pouvez également être invité à utiliser ces outils de diagnostic pour fournir les fichiers journaux. Pour zipper le fichier, vous pouvez utiliser la commande suivante.
```
tar  -czvf health_check_logs.tar.gz /var/log/health_check
```

**Étapes suivantes**

- Voir [Comment surveiller SAP HANA (grandes instances) sur Azure](./hana-monitor-troubleshoot.md).
