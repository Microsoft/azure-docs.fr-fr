---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/07/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: e84a77629026bb8885a48b8ed928699825f07115
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77111216"
---
| **Fournisseur** | **Famille de périphériques** | **Version du microprogramme** |
| --- | --- | --- |
|Cisco | ISR| IOS 15.1 (préversion)|
|Cisco | ASA | RouteBased ASA ( * ) (BGP No IKEv2) pour les versions ASA antérieures à 9.8 |
|Cisco | ASA | ASA RouteBased (IKEv2 - No BGP) pour ASA 9.8 ou versions postérieures |
|Juniper | SRX_GA | 12.x|
|Juniper | SSG_GA | ScreenOS 6.2|
|Juniper | JSeries_GA | JunOS 12.x|
|Juniper | SRX | JunOS 12.x BGP RouteBased |
|Ubiquiti| EdgeRouter| EdgeOS v1.10x RouteBased VTI|
|Ubiquiti| EdgeRouter| EdgeOS v1.10x RouteBased BGP|

> [!NOTE]
> ( * ) Obligatoire : NarrowAzureTrafficSelectors (activer l’option UsePolicyBasedTrafficSelectors) et CustomAzurePolicies (IKE/IPsec)
>
