---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 12/06/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 4fc14083b0912cf18f98c229adcb297b89bc8971
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95559826"
---
Créez une connexion Bureau à distance pour vous connecter à une machine virtuelle déployée sur votre réseau virtuel. La meilleure méthode pour vérifier que vous pouvez vous connecter à votre machine virtuelle consiste à vous connecter à l’aide de son adresse IP privée, plutôt qu’avec le nom d’ordinateur. Vous testez ainsi si vous pouvez vous connecter, que la résolution de nom soit configurée correctement ou non. 

1. Recherchez l’adresse IP privée de votre machine virtuelle. Pour trouver l’adresse IP privée d’une machine virtuelle, consultez les propriétés de la machine virtuelle dans le portail Azure ou utilisez PowerShell.
2. Vérifiez que vous êtes connecté à votre réseau virtuel à l’aide de la connexion VPN de point à site. 
3. Pour ouvrir une connexion Bureau à distance, saisissez *RDP* ou *Connexion Bureau à distance* dans la zone de recherche de la barre des tâches, puis sélectionnez **Connexion Bureau à distance**. Vous pouvez également l'ouvrir à l’aide de la commande **mstsc** dans PowerShell. 
3. Dans **Connexion Bureau à distance**, entrez l’adresse IP privée de la machine virtuelle. Si nécessaire, sélectionnez **Afficher les Options** pour définir des paramètres supplémentaires, puis connectez-vous.

### <a name="to-troubleshoot-an-rdp-connection-to-a-vm"></a>Pour résoudre une connexion RDP à une machine virtuelle

Si vous rencontrez des problèmes de connexion à une machine virtuelle sur votre connexion VPN, vous pouvez vérifier plusieurs points. 

- Vérifiez que votre connexion VPN aboutit.
- Vérifiez que vous vous connectez à l’adresse IP privée de la machine virtuelle.
- Entrez **ipconfig** pour vérifier l’adresse IPv4 attribuée à l’adaptateur Ethernet sur l’ordinateur à partir duquel vous vous connectez. Si l’adresse IP est comprise dans la plage d’adresses du réseau virtuel auquel vous vous connectez, ou dans la plage d’adresses de votre VPNClientAddressPool, cette situation est désignée sous le terme d’espaces d’adressage qui se chevauchent. Lorsque vos espaces d’adressage se chevauchent de cette façon, le trafic réseau n’atteint pas Azure et reste sur le réseau local.
- Si vous pouvez vous connecter à la machine virtuelle à l’aide de l’adresse IP privée, mais pas à l’aide du nom d’ordinateur, vérifiez que vous avez correctement configuré DNS. Pour plus d’informations sur le fonctionnement de la résolution de noms pour les machines virtuelles, consultez [Résolution de noms pour les machines virtuelles](../articles/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).
- Vérifiez que le package de configuration du client VPN a été généré après avoir spécifié les adresses IP du serveur DNS pour le réseau virtuel. Si vous mettez à jour les adresses IP du serveur DNS, générez et installez un package de configuration du client VPN.

Pour plus d’informations sur la résolution d’une connexion, consultez [Résoudre des problèmes de connexion Bureau à distance à une machine virtuelle](../articles/virtual-machines/troubleshooting/troubleshoot-rdp-connection.md).