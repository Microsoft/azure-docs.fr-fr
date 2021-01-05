---
title: Questions courantes sur les machines virtuelles dans Place de marché Azure
description: Questions courantes rencontrées lors de la création d’une machine virtuelle dans Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: guide
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 10/15/2020
ms.openlocfilehash: 88ec58ca9bcfa5c64036d3b65e77f3248e26357d
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93124949"
---
# <a name="common-questions-about-vm-in-azure-marketplace"></a>Questions courantes sur les machines virtuelles dans Place de marché Azure

Ces questions fréquentes couvrent les problèmes courants que vous pouvez rencontrer lors de la création d’une offre de machine virtuelle dans Place de marché Azure.

## <a name="how-do-i-configure-a-virtual-private-network-vpn-to-work-with-my-vms"></a>Comment configurer un réseau privé virtuel (VPN) pour qu’il fonctionne avec mes machines virtuelles ?

Si vous utilisez le modèle de déploiement Azure Resource Manager, vous disposez de trois options :

- [Créer une passerelle VPN basée sur des itinéraires à l’aide du Portail Azure](../vpn-gateway/tutorial-create-gateway-portal.md)
- [Créer une passerelle VPN basée sur un itinéraire à l’aide d’Azure PowerShell](../vpn-gateway/create-routebased-vpn-gateway-powershell.md)
- [Créer une passerelle VPN basée sur des itinéraires à l’aide de l’interface CLI](../vpn-gateway/create-routebased-vpn-gateway-cli.md)

## <a name="what-are-microsoft-support-policies-for-running-microsoft-server-software-on-azure-based-vms"></a>Quelles sont les stratégies de prise en charge de Microsoft pour exécuter des logiciels serveur Microsoft sur des machines virtuelles Azure ?

Pour plus d’informations, consultez [Prise en charge des logiciels serveurs Microsoft pour les machines virtuelles Microsoft Azure](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines).

## <a name="in-a-vm-how-do-i-manage-the-custom-script-extension-in-the-startup-task"></a>Dans une machine virtuelle, comment gérer l’extension de script personnalisé lors de la tâche de démarrage ?

Pour plus d’informations sur l’utilisation de l’extension de script personnalisé avec le module Azure PowerShell, sur les modèles Azure Resource Manager et sur les étapes de dépannage des systèmes Windows, consultez [Extension de script personnalisé pour Windows](../virtual-machines/extensions/custom-script-windows.md).

## <a name="are-32-bit-applications-or-services-supported-in-azure-marketplace"></a>Les services et les applications 32 bits sont-ils pris en charge dans la Place de marché Azure ?

Non. Les systèmes d’exploitation et services standard pris en charge pour les machines virtuelles Azure sont tous au format 64 bits. Bien que la plupart des systèmes d’exploitation 64 bits prennent en charge les versions 32 bits des applications à des fins de compatibilité descendante, l’utilisation d’applications 32 bits dans le cadre de votre solution de machine virtuelle n’est pas prise en charge et est fortement déconseillée. Vous devez recréer votre application sous la forme d’un projet 64 bits.

Pour plus d’informations, voir les articles suivants :

- [Running 32-bit applications](/windows/desktop/WinProg64/running-32-bit-applications) (Exécution d’applications 32 bits)
- [Prise en charge pour les systèmes d’exploitation 32 bits sur des machines virtuelles Azure](https://support.microsoft.com/help/4021388/support-for-32-bit-operating-systems-in-azure-virtual-machines)
- [Prise en charge des logiciels serveurs Microsoft pour les machines virtuelles Microsoft Azure](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)

## <a name="error-vhd-is-already-registered-with-image-repository-as-the-resource"></a>Erreur : Le disque dur virtuel est déjà inscrit auprès du référentiel d’images en tant que ressource

Chaque fois que j’essaie de créer une image à partir de mes disques durs virtuels, j’obtiens l’erreur « VHD is already registered with image repository as the resource » (Le disque dur virtuel est déjà inscrit auprès du référentiel d’images en tant que ressource) dans Azure PowerShell. Or, je n’ai pas créé d’image, et je n’ai pas non plus trouvé d’image portant ce nom dans Azure. Comment résoudre ce problème ?

Ce problème se produit généralement si vous avez créé une machine virtuelle à partir d’un disque dur virtuel verrouillé. Vérifiez qu’aucune machine virtuelle n’est allouée à partir de ce disque dur virtuel, puis tentez à nouveau l’opération. Si le problème persiste, ouvrez un ticket de support. Consultez [Prise en charge de l’Espace partenaires](support.md).

## <a name="next-steps"></a>Étapes suivantes

- [Résolution des problèmes de certification de machines virtuelles](azure-vm-create-certification-faq.md)