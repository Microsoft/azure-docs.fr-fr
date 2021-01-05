---
title: Fichier include
description: Fichier include
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 09/28/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 26a5537496d9e881ece135437c403baf4a4fd67c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016628"
---
Pour le moment, les disques Ultra ont des limitations supplémentaires, notamment :

Les seules options de redondance d’infrastructure disponibles pour les disques Ultra sont les zones de disponibilité. Les machines virtuelles utilisant d’autres options de redondance ne peuvent pas attacher un disque Ultra.

Le tableau suivant présente les régions où les disques Ultra sont disponibles ainsi que les options de disponibilité correspondantes :

> [!NOTE]
> Si l’une des régions de la liste suivante ne dispose pas de zones de disponibilité compatibles avec un disque Ultra, les machines virtuelles de cette région doivent être déployées sans option de redondance d’infrastructure pour pouvoir attacher un disque Ultra.

|Régions  |Options de redondance  |
|---------|---------|
|Brésil Sud     |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|Inde Centre     |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|Asie Est     |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|Allemagne Centre-Ouest     |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|Centre de la Corée     |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|États-Unis - partie centrale méridionale    |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|Gouvernement des États-Unis – Arizona     |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|Gouvernement américain - Virginie     |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|Gouvernement des États-Unis – Texas     |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|USA Ouest     |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)        |
|Centre de l’Australie    |Machines virtuelles uniques seulement (les groupes à haute disponibilité et les groupes de machines virtuelles identiques ne sont pas pris en charge)|
|Australie Est     |Trois zones de disponibilité         |
|Asie Sud-Est    |Trois zones de disponibilité        |
|Canada Centre*     |Trois zones de disponibilité          |
|USA Centre     |Trois zones de disponibilité          |
|USA Est     |Trois zones de disponibilité          |
|USA Est 2     |Trois zones de disponibilité         |
|France Centre    |Deux zones de disponibilité        |
|Japon Est    |Trois zones de disponibilité        |
|Europe Nord    |Trois zones de disponibilité        |
|Sud du Royaume-Uni    |Trois zones de disponibilité        |
|Europe Ouest    | Trois zones de disponibilité|
|USA Ouest 2    |Trois zones de disponibilité|

\* Contactez Support Azure pour obtenir l’accès à la fonctionnalité Zones de disponibilité pour cette région.

- Ils ne sont pris en charge que sur les séries de machines virtuelles suivantes :
    - [ESv3](../articles/virtual-machines/ev3-esv3-series.md#esv3-series)
    - [Easv4](../articles/virtual-machines/eav4-easv4-series.md#easv4-series)
    - [Edsv4](../articles/virtual-machines/edv4-edsv4-series.md#edsv4-series)
    - [Esv4](../articles/virtual-machines/ev4-esv4-series.md#esv4-series)
    - [DSv3](../articles/virtual-machines/dv3-dsv3-series.md#dsv3-series)
    - [Dasv4](../articles/virtual-machines/dav4-dasv4-series.md#dasv4-series)
    - [Ddsv4](../articles/virtual-machines/ddv4-ddsv4-series.md#ddsv4-series)
    - [Dsv4](../articles/virtual-machines/dv4-dsv4-series.md#dsv4-series)
    - [FSv2](../articles/virtual-machines/fsv2-series.md)
    - [LSv2](../articles/virtual-machines/lsv2-series.md)
    - [M](../articles/virtual-machines/workloads/sap/hana-vm-operations-storage.md)
    - [Mv2](../articles/virtual-machines/workloads/sap/hana-vm-operations-storage.md)
- Toutes les tailles de machines virtuelles ne sont pas disponibles dans toutes les régions prises en charge avec des disques Ultra.
- Ils sont disponibles uniquement comme disques de données. 
- Ils prennent en charge la taille de secteur physique de 4k par défaut. La taille du secteur 512E est disponible en tant qu’offre généralement disponible (aucun abonnement requis), mais elle n’est actuellement disponible qu’à l’aide de l’interface CLI ou de PowerShell. La plupart des applications sont compatibles avec les tailles de secteur de 4k mais certaines nécessitent des tailles de secteur de 512 octets. Citons à titre d’exemple Oracle Database, qui nécessite la version 12.2 ou une version ultérieure afin de prendre en charge les disques natifs 4k. Pour les versions antérieures d’Oracle DB, la taille de secteur de 512 octets est requise.
- Ils peuvent être créés seulement comme des disques vides.
- Ils ne prennent pas en charge les instantanés de disque, les images de machine virtuelle, les groupes à haute disponibilité, les hôtes dédiés Azure ou Azure Disk Encryption.
- Ils ne prennent pas en charge l’intégration aux services Sauvegarde Azure ou Azure Site Recovery.
- Ils prennent uniquement en charge les lectures non mises en cache et les écritures non mises en cache.
- À l’heure actuelle, le nombre limite d’IOPS sur les machines virtuelles en disponibilité générale est de 80 000.

Les disques Ultra Azure offrent jusqu’à 16 Tio par région et par abonnement par défaut, mais ils prennent en charge une plus grande capacité par demande. Pour demander une augmentation de la capacité, contactez le Support Azure.
