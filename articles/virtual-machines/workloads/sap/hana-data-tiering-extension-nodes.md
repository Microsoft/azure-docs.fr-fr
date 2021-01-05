---
title: Couches Données et nœuds d’extension pour SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Couches Données et nœuds d’extension pour SAP HANA sur Azure (grandes instances).
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
ms.date: 09/04/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 9a1fc1c3b332d033e453db11ce4451cd626c4217
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967803"
---
# <a name="use-sap-hana-data-tiering-and-extension-nodes"></a>Utiliser une hiérarchisation des données et des nœuds d’extension SAP HANA

SAP prend en charge un modèle de hiérarchisation des données pour SAP BW pour différentes versions SAP NetWeaver et pour SAP BW/4HANA. Pour plus d’informations sur le modèle de hiérarchisation des données, consultez le document SAP [SAP BW/4HANA et SAP BW sur HANA avec nœuds d’extension SAP HANA](https://www.sap.com/documents/2017/05/ac051285-bc7c-0010-82c7-eda71af511fa.html#).
La grande instance HANA vous permet d’utiliser la configuration de l’option 1 des nœuds d’extension SAP HANA, comme détaillé dans la FAQ et les documents de blog SAP. Les configurations de l’option 2 peuvent être définies avec les références SKU de grandes instances HANA suivantes : S72m, S192, S192m, S384 et S384m. 

Les avantages ne sont pas forcément visibles immédiatement lorsque l’on parcourt la documentation. Toutefois, en examinant les instructions portant sur le redimensionnement de SAP, vous pouvez trouver un avantage à utiliser les nœuds d’extension SAP HANA avec l’option 1 et l’option 2. Voici quelques exemples :

- Les instructions de dimensionnement de SAP HANA exigent généralement de doubler la quantité du volume de données en mémoire. Lorsque vous exécutez votre instance SAP HANA avec les données chaudes, vous n’avez que 50 % ou moins de la mémoire maximum remplis par les données. Ce qui reste de la mémoire est en principe conservé pour permettre à SAP HANA de faire son travail.
- Autrement dit, dans une unité S192 de grande instance HANA, dotée de 2 To de mémoire et exécutant une base de données SAP BW, vous ne disposez que de 1 To en volume de données.
- Si vous utilisez un nœud d’extension supplémentaire SAP HANA de l’option 1, ainsi qu’une référence SKU S192 de grande instance HANA, vous obtenez une capacité supplémentaire de 2 To pour le volume de données. Dans la configuration de l’option 2, vous disposez de 4 To supplémentaires pour le volume de données tièdes. Par rapport au nœud à accès rapide, la capacité de mémoire totale du nœud d’extension « à accès moyen » peut servir au stockage des données pour l’option 1. Le doublage de la mémoire peut être utilisé pour le volume de données dans la configuration du nœud d’extension SAP HANA de l’option 2.
- Vous vous retrouvez avec une capacité de 3 To pour vos données et d’un rapport accès rapide/accès moyen de 1:2 pour l’option 1. Vous avez 5 To de données et un rapport de 1:4 avec la configuration de nœud d’extension de l’option 2.

Plus le volume de données est important par rapport à la mémoire, plus la probabilité est forte que les données tièdes que vous demandez soient stockées dans le stockage sur disque.

**Étapes suivantes**
- Voir [Architecture SAP HANA (grandes instances) sur Azure](hana-architecture.md)