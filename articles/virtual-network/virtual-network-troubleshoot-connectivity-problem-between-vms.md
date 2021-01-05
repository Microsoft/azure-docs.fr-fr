---
title: Résolution des problèmes de connectivité entre machines virtuelles Azure | Microsoft Docs
description: Découvrez comment dépanner et résoudre les problèmes de connectivité que vous pouvez rencontrer entre des machines virtuelles Azure.
services: virtual-network
documentationcenter: na
author: chadmath
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/30/2018
ms.author: genli
ms.openlocfilehash: eb94b67b026ed108f31f6cd802010577665ec0d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87286085"
---
# <a name="troubleshooting-connectivity-problems-between-azure-vms"></a>Résolution des problèmes de connectivité entre machines virtuelles Azure

Vous pouvez rencontrer des problèmes de connectivité entre machines virtuelles. Cet article fournit les étapes requises pour vous aider à résoudre ce problème. 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="symptom"></a>Symptôme

Une machine virtuelle Azure ne peut pas se connecter à une autre machine virtuelle Azure.

## <a name="troubleshooting-guidance"></a>Instructions pour la résolution des problèmes 

1. [Vérifiez si la carte réseau est mal configurée](#step-1-check-whether-nic-is-misconfigured)
2. [Vérifiez si le trafic réseau est bloqué par un groupe de sécurité réseau ou un itinéraire défini par l’utilisateur ](#step-2-check-whether-network-traffic-is-blocked-by-nsg-or-udr)
3. [Vérifiez si le trafic réseau est bloqué par un pare-feu de machine virtuelle](#step-3-check-whether-network-traffic-is-blocked-by-vm-firewall)
4. [Vérifiez si une application ou un service de la machine virtuelle écoute sur le port](#step-4-check-whether-vm-app-or-service-is-listening-on-the-port)
5. [Vérifiez si le problème est causé par SNAT](#step-5-check-whether-the-problem-is-caused-by-snat)
6. [Vérifiez si le trafic est bloqué par des ACL pour la machine virtuelle classique](#step-6-check-whether-traffic-is-blocked-by-acls-for-the-classic-vm)
7. [Vérifiez si le point de terminaison est créé pour la machine virtuelle classique](#step-7-check-whether-the-endpoint-is-created-for-the-classic-vm)
8. [Essayez de vous connecter à un partage réseau de machine virtuelle](#step-8-try-to-connect-to-a-vm-network-share)
9. [Vérifiez la connectivité entre réseaux virtuels](#step-9-check-inter-vnet-connectivity)

## <a name="troubleshooting-steps"></a>Étapes de dépannage

Suivez ces étapes pour résoudre le problème. Après chaque étape, vérifiez si le problème est résolu. 

### <a name="step-1-check-whether-nic-is-misconfigured"></a>Étape 1 : Vérifier si la carte réseau est mal configurée

Suivez les étapes de [Comment réinitialiser l’interface réseau pour une machine virtuelle Windows Azure](../virtual-machines/windows/reset-network-interface.md). 

Si le problème se produit après la modification de l’interface réseau (NIC), procédez comme suit :

**Machines virtuelles avec plusieurs cartes réseau**

1. Ajoutez une carte réseau.
2. Corrigez les problèmes dans la carte réseau défectueuse ou supprimez la carte réseau défectueuse.  Ensuite, ajoutez de nouveau la carte réseau.

Pour plus d’informations, consultez [Ajouter ou supprimer des cartes réseau de machines virtuelles](virtual-network-network-interface-vm.md).

**Machine virtuelle à carte réseau unique** 

- [Redéployez la machine virtuelle Windows.](../virtual-machines/windows/redeploy-to-new-node.md)
- [Redéployer la machine virtuelle Linux.](../virtual-machines/linux/redeploy-to-new-node.md)

### <a name="step-2-check-whether-network-traffic-is-blocked-by-nsg-or-udr"></a>Étape 2 : Vérifiez si le trafic réseau est bloqué par un groupe de sécurité réseau ou un itinéraire défini par l’utilisateur

Utilisez la [Vérification des flux d’IP de Network Watcher](../network-watcher/network-watcher-ip-flow-verify-overview.md) et [l’Enregistrement de flux de groupe de sécurité réseau](../network-watcher/network-watcher-nsg-flow-logging-overview.md) pour déterminer s’il existe un groupe de sécurité réseau (NSG) ou un itinéraire défini par l’utilisateur (UDR) qui dérange le flux de trafic.

### <a name="step-3-check-whether-network-traffic-is-blocked-by-vm-firewall"></a>Étape 3 : Vérifiez si le trafic réseau est bloqué par un pare-feu de machine virtuelle

Désactivez le pare-feu, puis testez le résultat. Si le problème est résolu, vérifiez les paramètres du pare-feu, puis réactivez le pare-feu.

### <a name="step-4-check-whether-vm-app-or-service-is-listening-on-the-port"></a>Étape 4 : Vérifiez si une application ou un service de la machine virtuelle écoute sur le port

Vous pouvez utiliser une des méthodes suivantes pour vérifier si une application ou un service de la machine virtuelle écoute sur le port.

- Exécutez les commandes suivantes pour vérifier si le serveur écoute sur ce port.

**Machine virtuelle Windows**

```console
netstat –ano
```

**Machine virtuelle Linux**

```console
netstat -l
```

- Exécutez la commande **telnet** sur la machine virtuelle elle-même pour tester le port. Si le test échoue, l’application ou le service n’est pas configuré pour écouter sur ce port.

### <a name="step-5-check-whether-the-problem-is-caused-by-snat"></a>Étape 5 : Vérifiez si le problème est causé par SNAT

Dans certains scénarios, la machine virtuelle est placée derrière une solution d’équilibrage de charge qui a une dépendance sur des ressources en dehors d’Azure. Dans ces scénarios, si vous rencontrez des problèmes de connexion intermittents, le problème peut être dû à [l’épuisement du port SNAT](../load-balancer/load-balancer-outbound-connections.md). Pour résoudre ce problème, créez une adresse IP virtuelle (ou ILPIP pour la version classique) pour chaque machine virtuelle qui se trouve derrière l’équilibrage de charge et est sécurisée avec un groupe de sécurité réseau ou des ACL. 

### <a name="step-6-check-whether-traffic-is-blocked-by-acls-for-the-classic-vm"></a>Étape 6 : Vérifiez si le trafic est bloqué par des ACL pour la machine virtuelle classique

Une liste ACL permet d’autoriser ou refuser le trafic de manière sélective pour un point de terminaison de machine virtuelle. Pour plus d’informations, consultez la page [Gestion de l’ACL sur un point de terminaison](/previous-versions/azure/virtual-machines/windows/classic/setup-endpoints#manage-the-acl-on-an-endpoint).

### <a name="step-7-check-whether-the-endpoint-is-created-for-the-classic-vm"></a>Étape 7 : Vérifiez si le point de terminaison est créé pour la machine virtuelle classique

Toutes les machines virtuelles créées dans Azure à l’aide du modèle de déploiement classique peuvent automatiquement communiquer sur un canal réseau privé avec d’autres machines virtuelles dans le même service cloud ou réseau virtuel. Toutefois, les ordinateurs sur d'autres réseaux virtuels requièrent des points de terminaison pour diriger le trafic réseau entrant vers une machine virtuelle. Pour plus d’informations, consultez [Configuration de points de terminaison](../virtual-machines/windows/classic/setup-endpoints.md).

### <a name="step-8-try-to-connect-to-a-vm-network-share"></a>Étape 8 : Essayez de vous connecter à un partage réseau de machine virtuelle

Si vous ne parvenez pas à vous connecter à un partage réseau de machine virtuelle, le problème peut être causé par des cartes réseau non disponibles sur la machine virtuelle. Pour supprimer les cartes réseau non disponibles, consultez [Supprimer les cartes réseau non disponibles](../virtual-machines/troubleshooting/reset-network-interface.md#delete-the-unavailable-nics)

### <a name="step-9-check-inter-vnet-connectivity"></a>Étape 9 : Vérifiez la connectivité entre réseaux virtuels

Utilisez la [Vérification des flux d’IP de Network Watcher](../network-watcher/network-watcher-ip-flow-verify-overview.md) et [l’Enregistrement de flux de groupe de sécurité réseau](../network-watcher/network-watcher-nsg-flow-logging-overview.md) pour déterminer s’il existe un groupe de sécurité réseau ou un itinéraire défini par l’utilisateur qui dérange le flux de trafic. Vous pouvez également vérifier votre configuration entre réseaux virtuels [ici](https://support.microsoft.com/en-us/help/4032151/configuring-and-validating-vnet-or-vpn-connections).

### <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contactez le support technique.
Si vous avez besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.