---
title: Azure VMware Solution by CloudSimple – Adresse IP publique
description: En savoir plus sur les adresses IP publiques et leurs avantages sur Azure VMware Solution by CloudSimple
author: sharaths-cs
ms.author: dikamath
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 147ab6b5ace4493e1a0b303c320cb2e81d829b29
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77024974"
---
# <a name="cloudsimple-public-ip-address-overview"></a>Présentation de l’adresse IP publique CloudSimple

Une adresse IP publique permet à des ressources Internet d’envoyer des communications entrantes à des ressources de cloud privé vie une adresse IP privée. L’adresse IP privée est une machine virtuelle ou un équilibreur de charge logiciel sur votre vCenter de cloud privé. L’adresse IP publique vous permet d’exposer à Internet des services s’exécutant sur votre cloud privé.

L’adresse IP publique est dédiée à l’adresse IP privée jusqu’à ce que vous supprimiez l’attribution. Une adresse IP publique ne peut être attribuée qu’à une adresse IP privée.

Une ressource associée à une adresse IP publique toujours utilise l’adresse IP publique pour accéder à Internet. Par défaut, seul un accès Internet sortant est autorisé sur une adresse IP publique.  Le trafic entrant sur l’adresse IP publique est refusé.  Pour autoriser le trafic entrant, créez une règle de pare-feu pour l’adresse IP publique sur le port spécifique.

## <a name="benefits"></a>Avantages

L’utilisation d’une adresse IP publique pour des communications entrantes offre :

* Prévention des attaques par déni de service distribué (DDoS). Cette protection est automatiquement activée pour l’adresse IP publique.
* Surveillance Always On et atténuation en temps réel des attaques courantes au niveau du réseau. Ces défenses sont les mêmes que celles utilisées par les services en ligne Microsoft.
* Mise à l’échelle complète du réseau global Azure. La distribution et l’atténuation du trafic d’attaque peuvent être réalisées entre différentes régions.  

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur l’[allocation d’une adresse IP publique](public-ips.md)