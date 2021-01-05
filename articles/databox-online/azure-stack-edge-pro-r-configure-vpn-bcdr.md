---
title: Configurer la continuité d’activité et reprise d’activité (BCDR) sur un réseau privé virtuel (VPN) Azure Stack Edge
description: Décrit comment configurer la BCDR sur un VPN Azure Stack Edge.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 04/17/2020
ms.author: alkohli
ms.openlocfilehash: 4d735c623a6dffe24108d06d00caa7fba987c7df
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465321"
---
# <a name="configure-business-continuity-and-disaster-recovery-for-azure-stack-edge-vpn"></a>Configurer la continuité d’activité et reprise d’activité pour un VPN Azure Stack Edge

[!INCLUDE [applies-to-r-skus](../../includes/azure-stack-edge-applies-to-r-sku.md)]

Cet article explique comment configurer la continuité d’activité et reprise d’activité (BCDR) sur un réseau privé virtuel (VPN) configuré sur un appareil Azure Stack Edge.

Ils s’applique aux appareils Azure Stack Edge Pro R et Azure Stack Edge Mini R.

## <a name="configure-failover-to-a-paired-region"></a>Configurer le basculement vers une région jumelée

Votre appareil Azure Stack Edge utilise d’autres services Azure tels que Stockage Azure. Vous pouvez configurer la BCDR sur n’importe quel service Azure que l’appareil Azure Stack Edge utilise. Si un service Azure qu’Azure Stack Edge utilise bascule vers sa région jumelée, l’appareil Azure Stack Edge se connecte aux nouvelles adresses IP et la communication n’est pas doublement chiffrée. 

L’appareil Azure Stack Edge utilise le tunneling fractionné et l’ensemble des données et services configurés dans la région d’accueil (région associée à votre appareil Azure Stack Edge) passent par le tunnel VPN. Si les services Azure basculent vers une région jumelée qui se trouve en dehors de la région d’accueil, les données ne passent plus par le VPN et ne sont donc pas doublement chiffrées. 

Dans ce scénario, en général, seul un petit nombre de services Azure sont affectés. Pour résoudre ce problème, il convient d’apporter les modifications suivantes à la configuration du VPN Azure Stack Edge :

1. Ajoutez la ou les plages d’adresses IP du service Azure de basculement dans les itinéraires inclusifs pour le VPN sur Azure Stack Edge. Cela entraîne le routage des services via le VPN.

    Pour ajouter les itinéraires inclusifs, vous devez télécharger le fichier json contenant les itinéraires spécifiques du service. Veillez à mettre à jour ce fichier avec les nouveaux itinéraires.
2. Ajoutez la ou les plages d’adresses IP du service Azure correspondantes dans la table de routage Azure.
3. Ajoutez les itinéraires au pare-feu.

> [!NOTE]
>
> 1. Le basculement d’une passerelle VPN Azure et d’un réseau virtuel (VNet) Azure est traité dans la section [Récupérer à partir d’une région Azure défaillante](#recover-from-a-failed-azure-region).
> 2. Il peut arriver que les plages d’adresses IP ajoutées à la table de routage Azure dépassent la limite de 400. Dans ce cas, vous devez suivre les instructions de la section, [Passer d’une région Azure à une autre](#move-from-an-azure-region-to-another).

## <a name="recover-from-a-failed-azure-region"></a>Récupérer à partir d’une région Azure défaillante

Si la totalité de la région Azure bascule en raison d’un événement catastrophique tel qu’un séisme, tous les services Azure de cette région, y compris le service Azure Stack Edge, basculent. Compte tenu de la multiplicité des services, les itinéraires inclusifs peuvent facilement se compter par centaines. Azure limite le nombre d’itinéraires à 400. 

Lorsque la région bascule, le réseau virtuel (VNet) bascule également vers la nouvelle région, tout comme la passerelle de réseau virtuel (passerelle VPN). En réponse à ce changement, apportez les modifications suivantes à votre configuration de VPN Azure Stack Edge :

1. Déplacez votre réseau virtuel vers la région cible. Pour plus d'informations, consultez : [Déplacer un réseau virtuel Azure vers une autre région en utilisant le portail Azure](../virtual-network/move-across-regions-vnet-portal.md).
2. Déployez une nouvelle passerelle VPN Azure dans la région cible où vous avez déplacé le réseau virtuel. Pour plus d’informations, consultez [Créer une passerelle de réseau virtuel](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#creategw).
3. Mettez la configuration du VPN Azure Stack Edge de façon à utiliser la passerelle VPN ci-dessus dans la connexion VPN, puis sélectionnez la région cible pour ajouter des itinéraires qui utilisent la passerelle VPN.
4. Mettez à jour la table de routage Azure entrante si le pool d’adresses du client change. 

## <a name="move-from-an-azure-region-to-another"></a>Passer d’une région Azure à une autre

Vous pouvez déplacer votre appareil Azure Stack Edge d’un emplacement vers un autre. Pour utiliser la région la plus proche de l’emplacement où votre appareil est déployé, vous devez configurer celui-ci pour une nouvelle région d’accueil. Effectuez les modifications suivantes :

1. Vous pouvez mettre à jour la configuration du VPN Azure Stack Edge de façon à utiliser la passerelle VPN d’une nouvelle région, puis sélectionner la nouvelle région pour ajouter des itinéraires qui utilisent la passerelle VPN.

## <a name="next-steps"></a>Étapes suivantes

[Sauvegardez votre appareil Azure Stack Edge](azure-stack-edge-gpu-prepare-device-failure.md).