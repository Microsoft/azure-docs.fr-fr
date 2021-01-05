---
title: Exigences réseau supplémentaires pour SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Exigences réseau supplémentaires pour SAP HANA sur Azure (grandes instances).
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
ms.date: 09/10/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 44e9712466734c0e42fd2bea05c5110cbff6924b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964794"
---
# <a name="additional-network-requirements-for-large-instances"></a>Exigences réseau supplémentaires pour les grandes instances

Vous risquez de devoir répondre à des exigences réseau supplémentaires dans le cadre d’un déploiement de grandes instances de SAP HANA sur Azure.

## <a name="add-more-ip-addresses-or-subnets"></a>Ajouter des adresses IP ou des sous-réseaux supplémentaires

Utilisez le portail Azure, PowerShell ou l’interface Azure CLI pour ajouter des adresses IP ou des sous-réseaux.

Ajoutez la nouvelle plage d’adresses IP comme nouvelle plage dans l’espace d’adressage du réseau virtuel, au lieu de générer une nouvelle plage agrégée. Envoyez ce changement à Microsoft. Cela vous permet de vous connecter à partir de cette nouvelle plage d’adresses IP aux unités de grande instance HANA de votre client. Vous pouvez ouvrir une demande de support Azure pour que le nouvel espace d’adressage du réseau virtuel soit ajouté. Après avoir reçu la confirmation, effectuez les étapes suivantes.

Pour créer un sous-réseau supplémentaire à partir du portail Azure, consultez [Créer un réseau virtuel à l’aide du portail Azure](../../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-virtual-network). Pour en créer un à partir de PowerShell, consultez [Créer un réseau virtuel à l’aide de PowerShell](../../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-virtual-network).

## <a name="add-virtual-networks"></a>Ajouter des réseaux virtuels

Après avoir connecté un ou plusieurs réseaux virtuels Azure, vous voudrez peut-être en connecter d’autres qui accèdent à SAP HANA sur Azure (grandes instances). Commencez par envoyer une demande de support Azure. Dans cette demande, fournissez les informations spécifiques qui identifient le déploiement Azure en question. Spécifiez aussi la ou les plages d’espace d’adressage IP de l’espace d’adressage du réseau virtuel Azure. SAP HANA sur Microsoft Service Management fournit alors les informations nécessaires pour connecter les réseaux virtuels supplémentaires et Azure ExpressRoute. Pour chaque réseau virtuel, vous avez besoin d’une clé d’autorisation unique pour connecter le circuit ExpressRoute aux grandes instances HANA.

## <a name="increase-expressroute-circuit-bandwidth"></a>Augmenter la bande passante du circuit ExpressRoute

Consultez SAP HANA sur Microsoft Service Management. S’il vous est recommandé d’augmenter la bande passante du circuit ExpressRoute de SAP HANA sur Azure (grandes instances), créez une demande de support Azure. (Vous pouvez demander une augmentation maximale de 10 Gbits/s pour une seule bande passante de circuit.) Vous recevrez ensuite une notification une fois l’opération terminée ; vous n’avez rien d’autre à faire pour activer cette vitesse supérieure dans Azure.

## <a name="add-an-additional-expressroute-circuit"></a>Ajouter un circuit ExpressRoute supplémentaire

Consultez SAP HANA sur Microsoft Service Management. S’il vous est conseillé d’ajouter un circuit ExpressRoute supplémentaire, créez une demande de support Azure (y compris une demande d’obtention des informations d’autorisation pour se connecter au nouveau circuit). Avant d’effectuer la demande, vous devez définir l’espace d’adressage utilisé sur les réseaux virtuels. SAP HANA sur Microsoft Service Management peut ensuite fournir l’autorisation.

Quand le nouveau circuit est créé et la configuration de SAP HANA sur Microsoft Service Management est terminée, vous recevez une notification avec les informations nécessaires pour continuer. Vous ne pouvez pas connecter des réseaux virtuels Azure à ce circuit supplémentaire si ceux-ci sont déjà connectés à un autre circuit ExpressRoute SAP HANA sur Azure (grande instance) dans la même région Azure.

## <a name="delete-a-subnet"></a>Supprimer un sous-réseau

Pour supprimer un sous-réseau de réseau virtuel, vous pouvez utiliser le portail Azure, PowerShell ou Azure CLI. Si l’espace d’adressage ou la plage d’adresses IP de votre réseau virtuel Azure était une plage agrégée, vous ne recevrez aucun suivi de la part de Microsoft. (Notez toutefois que le réseau virtuel continue à propager l’espace d’adressage de la route BGP qui inclut le sous-réseau supprimé.) Vous avez probablement défini l’espace d’adressage ou la plage d’adresses du réseau virtuel Azure sous la forme de plusieurs plages d’adresses IP, dont l’une d’entre elles a été affectée à votre sous-réseau supprimé. Veillez à la supprimer de l’espace d’adressage de votre réseau virtuel. Ensuite, demandez à SAP HANA sur Microsoft Service Management de la supprimer des plages avec lesquelles SAP HANA sur Azure (grandes instances) est autorisé à communiquer.

Pour plus d’informations, consultez [Supprimer un sous-réseau](../../../virtual-network/virtual-network-manage-subnet.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#delete-a-subnet).

## <a name="delete-a-virtual-network"></a>Supprimer un réseau virtuel

Pour plus d’informations, consultez [Supprimer un réseau virtuel](../../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#delete-a-virtual-network).

SAP HANA sur Microsoft Service Management supprime les autorisations existantes sur le circuit ExpressRoute de SAP HANA sur Azure (grandes instances). Il supprime également l’espace d’adressage ou la plage d’adresses IP du réseau virtuel Azure qui communiquent avec les grandes instances HANA.

Une fois que vous avez supprimé le réseau virtuel, ouvrez une demande de support Azure pour fournir la ou les plages d’adresses IP à supprimer.

Pour veiller à tout supprimer, supprimez la connexion ExpressRoute, la passerelle de réseau virtuel, l’adresse IP publique de la passerelle de réseau virtuel et le réseau virtuel.

## <a name="delete-an-expressroute-circuit"></a>Supprimer un circuit ExpressRoute

Pour supprimer un circuit ExpressRoute supplémentaire de SAP HANA sur Azure (grandes instances), ouvrez une demande de support Azure avec SAP HANA sur Microsoft Service Management. Demandez à ce que le circuit soit supprimé. Dans l’abonnement Azure, vous pouvez supprimer ou conserver le réseau virtuel, en fonction des besoins. Toutefois, vous devez supprimer la connexion entre le circuit ExpressRoute des grandes instances HANA et la passerelle de réseau virtuel liée.

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique d’installation et de configuration de SAP HANA (grandes instances) sur Azure](hana-installation.md)
