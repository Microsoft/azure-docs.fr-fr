---
title: Résoudre les redémarrages inattendus de machines virtuelles avec des VHD joints sur des machines virtuelles Azure | Microsoft Docs
description: Comment résoudre les redémarrages inattendus des machines virtuelles.
keywords: connexion ssh refusée, erreur ssh, ssh azure, échec de connexion SSH
services: virtual-machines
author: genlin
manager: dcscontentpm
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.service: virtual-machines
ms.topic: article
ms.date: 11/01/2018
ms.author: genli
ms.openlocfilehash: 3a06db1afd130d936af868d0d20632c3ec4fbfd2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75358524"
---
# <a name="troubleshoot-unexpected-reboots-of-vms-with-attached-vhds"></a>Résoudre les problèmes de redémarrage inattendu de machines virtuelles avec VHD joints

Si une machine virtuelle Azure est associée à un grand nombre de VHD dans le même compte de stockage, vous risquez de dépasser les cibles d’extensibilité pour un même compte de stockage, ce qui provoquera un redémarrage inattendu de la machine virtuelle. Consultez les métriques par minute pour le compte de stockage (**TotalRequests**/**TotalIngress**/**TotalEgress**) et vérifiez la présence de pics dépassant les cibles d’évolutivité pour un compte de stockage. La section [Les métriques indiquent une augmentation de la valeur PercentThrottlingError](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#metrics-show-an-increase-in-PercentThrottlingError) vous aidera à déterminer si une limitation a été appliquée à votre compte de stockage.

En général, chaque opération d’entrée ou de sortie individuelle sur un disque dur virtuel à partir d’une machine virtuelle traduit les opérations **Get Page** ou **Put Page** sur l’objet blob de la page sous-jacente. Vous pouvez donc utiliser les opérations d’E/S par seconde (IOPS) estimées pour votre environnement pour déterminer le nombre de disques durs virtuels que vous pouvez avoir dans un compte de stockage unique, sur base du comportement spécifique de votre application. Microsoft recommande qu’un compte de stockage ne comporte pas plus de 40 disques. Pour plus d’informations sur les objectifs d’évolutivité pour les comptes de stockage standard, consultez [Objectifs d’extensibilité pour les comptes de stockage standard](../../storage/common/scalability-targets-standard-account.md). Pour plus d’informations sur les objectifs d’évolutivité des comptes de stockage d’objets blob de pages Premium, consultez [Objectifs d’évolutivité pour les comptes de stockage d’objets blob de pages Premium](../../storage/blobs/scalability-targets-premium-page-blobs.md).

Si vous dépassez les cibles d’évolutivité pour votre compte de stockage, placez vos VHD dans plusieurs comptes de stockage afin de réduire l’activité dans chaque compte individuel.
