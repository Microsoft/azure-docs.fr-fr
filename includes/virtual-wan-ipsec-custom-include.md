---
title: Fichier include
description: Fichier include
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 10/07/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: e2a950037aed2a8ded4d4e55920721285cbfc05c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82204660"
---
Quand vous utilisez des stratégies IPsec personnalisées, gardez à l’esprit les exigences suivantes :

* **IKE** - Pour IKE, vous pouvez sélectionner n’importe quel paramètre de chiffrement IKE, d’intégrité IKE et de groupe DH.
* **IPsec** - Pour IPsec, vous pouvez sélectionner n’importe quel paramètre de chiffrement IPsec, d’intégrité IPsec et de PFS. Si GCM est utilisé dans les paramètres de chiffrement IPsec ou d’intégrité IPsec, alors il doit être utilisé pour le chiffrement et l’intégrité.

>[!NOTE]
> Les stratégies IPsec personnalisées n’impliquent aucun concept de répondeur et d’initiateur (contrairement aux stratégies IPsec par défaut). Les deux côtés (passerelle locale et passerelle VPN Azure) utilisent les mêmes paramètres pour IKE Phase 1 et IKE Phase 2. Les protocoles IKEv1 et IKEv2 sont pris en charge. Azure ne peut pas jouer le rôle de répondeur uniquement.
>

**Paramètres et configurations disponibles**

| Paramètre | Paramètres |
|--- |--- |
| Chiffrement IKE | GCMAES256, GCMAES128, AES256, AES128 |
| Intégrité IKE | SHA384, SHA256 |
| Groupe DH | ECP384, ECP256, DHGroup24, DHGroup14 |
| Chiffrement IPsec | GCMAES256, GCMAES128, AES256, AES128, aucun |
| Intégrité IPsec | GCMAES256, GCMAES128, SHA256 |
| Groupe PFS | ECP384, ECP256, PFS24, PFS14, aucun |
