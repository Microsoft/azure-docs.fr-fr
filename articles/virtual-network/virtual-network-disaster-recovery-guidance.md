---
title: Réseau virtuel – Continuité des activités | Microsoft Docs
description: Découvrez la procédure à suivre si une interruption du service Azure affecte vos instances réseaux virtuels Azure.
services: virtual-network
documentationcenter: ''
author: NarayanAnnamalai
manager: jefco
editor: ''
ms.assetid: ad260ab9-d873-43b3-8896-f9a1db9858a5
ms.service: virtual-network
ms.workload: virtual-network
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2016
ms.author: narayan
ms.reviewer: aglick
ms.openlocfilehash: 3f91d24bff0bec540ff0e7964f21c2f47c03638c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67876168"
---
# <a name="virtual-network--business-continuity"></a>Réseau virtuel – Continuité des activités

## <a name="overview"></a>Vue d’ensemble
Un réseau virtuel (VNet) est une représentation de votre réseau dans le cloud. Il vous permet de définir votre propre espace d’adressage IP privé et de segmenter le réseau en sous-réseaux. Les réseaux virtuels servent de limite d’approbation pour héberger vos ressources de calcul telles que les machines virtuelles et les services cloud Azure (rôles web/de travail). Un réseau virtuel permet la communication IP privée directe entre les ressources qu’il contient. Vous pouvez lier un réseau virtuel à un réseau local via une passerelle VPN ou ExpressRoute.

Un réseau virtuel est créé dans l’étendue d’une région. Vous pouvez *créer* des réseaux virtuels avec le même espace d’adressage dans deux régions différentes (par exemple, USA Est et USA Ouest), mais vous ne pourrez pas les connecter les uns aux autres en raison de leur espace d’adressage en commun. 

## <a name="business-continuity"></a>Continuité de l’activité

Peut-être que votre application peut être interrompue de différentes manières. Une région peut être coupée complètement des autres en raison d’une catastrophe naturelle ou d’une catastrophe partielle en raison d’une panne de plusieurs appareils ou services. L’impact sur le service de réseau virtuel est différent dans chacune de ces situations.

**Q : si une panne se produit pour une région entière, que faire ? Par exemple, si une région est entièrement coupée en raison d’une catastrophe naturelle ? Que se passe-t-il pour les réseaux virtuels hébergés dans la région ?**

A : le réseau virtuel et les ressources dans la région affectée restent inaccessibles pendant la durée de l’interruption de service.

![Diagramme de réseau virtuel simple](./media/virtual-network-disaster-recovery-guidance/vnet.png)

**Q : que faire pour recréer le même réseau virtuel dans une autre région ?**

A : les réseaux virtuels sont des ressources relativement légères. Vous pouvez appeler des API Azure pour créer un réseau virtuel avec le même espace d’adressage dans une autre région. Pour recréer le même environnement que celui qui était présent dans la région affectée, vous effectuez des appels d’API pour redéployer vos services cloud (rôles web et de travail) et les machines virtuelles que vous aviez. Vous devrez également créer une passerelle VPN et vous connecter à votre réseau local si vous aviez une connectivité locale (comme dans un déploiement hybride).

Pour créer un réseau virtuel, consultez [Création d’un réseau virtuel](manage-virtual-network.md#create-a-virtual-network).

**Q : un réplica d’un réseau virtuel dans une région donnée peut-il être recréé dans une autre région à l’avance ?**

A : oui, vous pouvez créer deux réseaux virtuels utilisant le même espace d’adressage IP privé et des ressources identiques dans deux régions différentes à l’avance. Si vous hébergez des services Internet dans le réseau virtuel, vous pourriez configurer Traffic Manager de manière à router géographiquement le trafic vers la région active. Toutefois, vous ne pouvez pas connecter deux réseaux virtuels avec le même espace d’adressage de réseau local sous peine de causer des problèmes de routage. En cas d’incident et de perte d’un réseau virtuel dans une région, vous pouvez connecter à votre réseau local l’autre réseau virtuel dans la région disponible avec l’espace d’adressage correspondant.

Pour créer un réseau virtuel, consultez [Création d’un réseau virtuel](manage-virtual-network.md#create-a-virtual-network).

