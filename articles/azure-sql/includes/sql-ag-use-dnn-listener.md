---
title: Utilisez un écouteur de nom de réseau distribué (DNN) au lieu d’un écouteur VNN pour les groupes de disponibilité sur les machines virtuelles SQL Server.
description: Message redirigeant les clients vers l’utilisation de l’écouteur DNN à la place de l’écouteur VNN.
ms.topic: include
author: MashaMSFT
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: b4ea8428ae23537cde584bf6944732f65ebd2f26
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92167988"
---
> [!NOTE]
> Les clients utilisant SQL Server 2019 CU8 et versions ultérieures sur Windows 2016 et versions ultérieures peuvent remplacer Azure Load Balancer et l’écouteur VNN classiques par un [écouteur de nom de réseau distribué (DNN)](../virtual-machines/windows/availability-group-distributed-network-name-dnn-listener-configure.md). Ignorez les étapes restantes de cet article qui créent l’écouteur et l’équilibreur de charge.
