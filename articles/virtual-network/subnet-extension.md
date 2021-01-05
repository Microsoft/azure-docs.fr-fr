---
title: Extension de sous-réseau dans Azure | Microsoft Docs
description: En savoir plus sur l’extension de sous-réseau dans Azure.
services: virtual-network
documentationcenter: na
author: anupam-p
manager: narayan
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/31/2019
ms.author: anupand
ms.openlocfilehash: f718471c3f79e9a33b0e03b088f8c8d2ae0231d3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73586846"
---
# <a name="subnet-extension"></a>Extension de sous-réseau
La migration des charges de travail vers le cloud public requiert une planification et une coordination rigoureuses. L’une des principales considérations à prendre en compte est la possibilité de conserver vos adresses IP. Ce qui peut être important surtout si vos applications ont une dépendance d’adresse IP ou si vous avez des exigences de conformité pour utiliser des adresses IP spécifiques. Le réseau virtuel Azure résout ce problème pour vous en vous permettant de créer des réseaux virtuels et des sous-réseaux à l’aide d’une plage d’adresses IP de votre choix.

Les migrations peuvent être délicates lorsque la configuration requise ci-dessus est couplée à une exigence supplémentaire pour conserver certaines applications localement. Dans ce type de situation, vous devrez fractionner les applications entre Azure et en local, sans renuméroter les adresses IP de chaque côté. En outre, vous devrez autoriser les applications à communiquer comme si elles se trouvaient sur le même réseau.

Une solution au problème ci-dessus est l’extension de sous-réseau. L’extension d’un réseau permet aux applications de communiquer sur le même domaine de diffusion lorsqu’elles se trouvent à des emplacements physiques différents, ce qui évite de devoir remanier la topologie de votre réseau. 

Bien que l’extension de votre réseau ne soit pas une bonne pratique en général, les cas d’utilisation ci-dessous peuvent être nécessaires.

- **Migration échelonnée** : Le scénario le plus courant est que vous souhaitez échelonner votre migration. Vous souhaitez placer quelques applications en premier et au fil du temps migrer les autres applications vers Azure.
- **Latence** : Les exigences de faible latence peuvent être une autre raison pour laquelle vous souhaitez conserver certaines applications localement, afin de vous assurer qu’elles sont aussi proches que possible de votre centre de données.
- **Conformité** : Dans un autre cas d’utilisation, vous pouvez avoir des exigences de conformité et souhaiter conserver certaines de vos applications localement.
 
> [!NOTE] 
> Vous ne devez pas étendre vos sous-réseaux, sauf si cela est nécessaire. Dans les cas où vous étendez vos sous-réseaux, vous devez essayer d’en faire une étape intermédiaire. Avec le temps, vous devez essayer de renuméroter les applications sur votre réseau local et de les migrer vers Azure.

Dans la section suivante, nous allons aborder la façon dont vous pouvez étendre vos sous-réseaux dans Azure.


## <a name="extend-your-subnet-to-azure"></a>Étendre votre sous-réseau à Azure
 Vous pouvez étendre vos sous-réseaux locaux à Azure à l’aide d’une solution basée sur un réseau de superposition de couche 3. La plupart des solutions utilisent une technologie de superposition telle que VXLAN pour étendre le réseau de couche 2 à l’aide d’un réseau de superposition de couche 3. Le diagramme ci-dessous illustre une solution généralisée. Dans cette solution, le même sous-réseau existe des deux côtés, côté Azure et en local. 

![Exemple d’extension de sous-réseau](./media/subnet-extension/subnet-extension.png)

Les adresses IP du sous-réseau sont attribuées aux machines virtuelles sur Azure et en local. La solution Azure et la solution locale ont une appliance virtuelle réseau insérée dans leurs réseaux. Quand une machine virtuelle dans Azure tente de communiquer avec une machine virtuelle dans un réseau local, l’appliance virtuelle réseau Azure capture le paquet, l’encapsule et l’envoie via un routage VPN/Express vers le réseau local. L’appliance virtuelle réseau locale reçoit le paquet, le désencapsule et le transmet au destinataire prévu dans son réseau. Le trafic de retour utilise un chemin d’accès et une logique similaires.

Dans l’exemple ci-dessus, l’appliance virtuelle réseau Azure et l’appliance virtuelle réseau locale communiquent et apprennent les adresses IP les unes après les autres. Des réseaux plus complexes peuvent également avoir un service de mappage, qui gère le mappage entre les appliances virtuelles réseau et les adresses IP qui se trouvent derrière elles. Lorsqu’une appliance virtuelle réseau reçoit un paquet, elle interroge le service de mappage pour connaître l’adresse de l’appliance virtuelle réseau qui possède l’adresse IP de destination en arrière-plan.

Dans la section suivante, vous trouverez des informations sur les solutions d’extension de sous-réseau que nous avons testées sur Azure.

## <a name="next-steps"></a>Étapes suivantes 
[Étendre votre sous-réseau à Azure à l’aide des solutions de fournisseurs.](https://github.com/microsoft/Azure-LISP)