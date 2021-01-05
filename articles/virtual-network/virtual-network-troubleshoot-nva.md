---
title: Dépannage d’appliance virtuelle réseau dans Azure | Microsoft Docs
description: Résolvez les problèmes liés aux appliances virtuelles réseau (NVA) dans Azure et validez les exigences de base de la plateforme Azure pour les configurations NVA.
services: virtual-network
documentationcenter: na
author: genlin
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/26/2018
ms.author: genli
ms.openlocfilehash: 3a8982b5626e3c19dbd49a3d2e20542d44b1a1da
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368585"
---
# <a name="network-virtual-appliance-issues-in-azure"></a>Problèmes d’appliance virtuelle réseau dans Azure

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez rencontrer des problèmes de connectivité de machines virtuelles (MV) ou de réseau privé virtuel (VPN), ainsi que des erreurs lors de l’utilisation d’une appliance virtuelle réseau (NVA) dans Microsoft Azure. Cet article décrit les étapes élémentaires pour valider les exigences de plateforme Azure de base pour des configurations d’appliance virtuelle réseau.

Un support technique pour appliances virtuelles réseau tierces et leur intégration avec la plateforme Azure est fourni par le fournisseur de l’appliance virtuelle réseau.

> [!NOTE]
> Si vous rencontrez un problème de connectivité ou de routage qui implique une appliance virtuelle réseau, vous devez [contactez directement le fournisseur de l’appliance virtuelle réseau](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="checklist-for-troubleshooting-with-nva-vendor"></a>Liste de contrôle pour le dépannage avec un fournisseur d’appliance virtuelle réseau

- Mises à jour logicielles pour machine virtuelle d’appliance virtuelle réseau
- Fonctionnalités et configuration de compte de service
- Itinéraires définis par l’utilisateur sur des sous-réseaux de réseau virtuel qui dirigent le trafic vers l’appliance virtuelle réseau
- Itinéraires définis par l’utilisateur sur des sous-réseaux de réseau virtuel qui dirigent le trafic à partir de l’appliance virtuelle réseau
- Tables et règles de routage au sein de l’appliance virtuelle réseau (par exemple, de NIC1 à NIC2)
- Suivi sur les NIC d’appliance virtuelle réseau pour vérifier la réception et l’envoi de trafic réseau
- Lorsque vous utilisez une référence SKU Standard et des adresses IP publiques, vous devez disposer d’un groupe de sécurité réseau et d’une règle explicite pour autoriser le routage du trafic vers l’appliance virtuelle réseau.

## <a name="basic-troubleshooting-steps"></a>Étapes de dépannage de base

- Vérifier la configuration de base
- Vérifier les performances de l’appliance virtuelle réseau
- Dépannage de réseau avancé

## <a name="check-the-minimum-configuration-requirements-for-nvas-on-azure"></a>Vérifiez la configuration minimale requise pour les appliances virtuelles réseau sur Azure

Chaque appliance virtuelle réseau a des exigences de configuration de base pour fonctionner correctement sur Azure. La section suivante décrit les étapes de vérification de ces configurations de base. Pour plus d’informations, [contactez le fournisseur de l’appliance virtuelle réseau](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines).

**Vérifier si le transfert IP est activé sur l’appliance virtuelle réseau**

Utiliser le portail Azure

1. Localisez la ressource d’appliance virtuelle réseau sur le [portail Azure](https://portal.azure.com), sélectionnez Mise en réseau, puis sélectionnez l’interface réseau.
2. Sur la page Interface réseau, sélectionnez Configuration IP.
3. Assurez-vous que le transfert IP est activé.

Utiliser PowerShell

1. Ouvrez PowerShell, puis connectez-vous à votre compte Azure.
2. Exécutez la commande suivante (en remplaçant les valeurs entre crochets par vos informations) :

   ```powershell
   Get-AzNetworkInterface -ResourceGroupName <ResourceGroupName> -Name <NicName>
   ```

3. Vérifiez la propriété **EnableIPForwarding** .
4. Si le transfert IP n’est pas activé, exécutez les commandes suivantes pour l’activer :

   ```powershell
   $nic2 = Get-AzNetworkInterface -ResourceGroupName <ResourceGroupName> -Name <NicName>
   $nic2.EnableIPForwarding = 1
   Set-AzNetworkInterface -NetworkInterface $nic2
   Execute: $nic2 #and check for an expected output:
   EnableIPForwarding   : True
   NetworkSecurityGroup : null
   ```

**Vérifiez l’existence d’un groupe de sécurité réseau quand vous utilisez des adresses IP publiques et une référence SKU Standard** Quand vous utilisez une référence SKU Standard et des adresses IP publiques, il doit y avoir un groupe de sécurité réseau créé et une règle explicite pour autoriser le trafic vers l’appliance virtuelle réseau.

**Vérifier si le trafic peut être routé vers l’appliance virtuelle réseau**

1. Sur [le portail Azure](https://portal.azure.com), ouvrez **Network Watcher** , puis sélectionnez **Tronçon suivant** .
2. Spécifiez une machine virtuelle qui est configurée pour rediriger le trafic vers l’appliance virtuelle réseau, et une adresse IP de destination à laquelle afficher le tronçon suivant. 
3. Si l’appliance virtuelle réseau n’est pas répertoriée comme le **tronçon suivant** , vérifiez et mettez à jour les tables de routage Azure.

**Vérifier si le trafic peut atteindre l’appliance virtuelle réseau**

1. Dans le [portail Azure](https://portal.azure.com), ouvrez **Network Watcher** , puis sélectionnez **Vérification du flux IP** . 
2. Spécifiez la machine virtuelle et l’adresse IP de l’appliance virtuelle réseau, puis vérifiez si le trafic est bloqué par des groupes de sécurité réseau (NSG).
3. S’il existe une règle de groupe de sécurité réseau qui bloque le trafic, localisez le groupe de sécurité réseau dans les règles de **sécurité efficace** , puis mettez-le à jour pour autoriser le passage du trafic. Réexécutez ensuite la **Vérification du flux IP** et utilisez la **Résolution des problèmes de connexion** pour tester les communications TCP entre la machine virtuelle et votre adresse IP interne ou externe.

**Vérifier si l’appliance virtuelle réseau et les machines virtuelles écoutent le trafic attendu**

1. Connectez-vous à l’appliance virtuelle réseau en utilisant le protocole RDP ou SSH, puis exécutez la commande suivante :

    Pour Windows :

    ```console
   netstat -an
    ```

    Pour Linux :

    ```console
   netstat -an | grep -i listen
    ```
2. Si vous ne voyez pas le port TCP utilisé par le logiciel de l’appliance virtuelle réseau répertorié dans les résultats, vous devez configurer l’application sur l’appliance virtuelle réseau et la machine virtuelle pour écouter le trafic qui atteint ces ports et y répondre. [Contactez le fournisseur de l’appliance virtuelle réseau au besoin](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines).

## <a name="check-nva-performance"></a>Vérifier les performances de l’appliance virtuelle réseau

### <a name="validate-vm-cpu"></a>Valider le processeur de machine virtuelle

Si l’utilisation du processeur approche 100 pour cent, vous risquez de rencontrer des problèmes affectant les rejets de paquet réseau. Votre machine virtuelle indique l’utilisation moyenne du processeur pendant intervalle de temps spécifique dans le portail Azure. Lors d’un pic d’utilisation du processeur, examinez sur la machine virtuelle invitée le processus à l’origine de l’utilisation élevée du processeur, et atténuez celle-ci autant que possible. Il se peut que vous deviez également redimensionner la machine virtuelle en définissant une taille de référence (SKU) supérieure ou, pour un groupe de machines virtuelles identiques, augmenter le nombre d’instances ou définir une mise à l’échelle automatique en fonction de l’utilisation du processeur. Pour chacun de ces problèmes, [contactez le fournisseur de l’appliance virtuelle réseau](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines) en fonction des besoins.

### <a name="validate-vm-network-statistics"></a>Valider les statistiques du réseau de machines virtuelles

Si l’utilisation du réseau de machines virtuelles présente des pics ou des périodes d’utilisation intense, il se peut également que vous deviez augmenter la taille de la référence (SKU) de la machine virtuelle pour obtenir des capacités de débit supérieures. Vous pouvez également redéployer la machine virtuelle en activant la fonctionnalité Mise en réseau accélérée. Pour vérifier si l’appliance virtuelle réseau prend en charge la fonctionnalité Mise en réseau accélérée, [contactez le fournisseur de l’appliance virtuelle réseau pour obtenir de l’assistance](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines) si nécessaire.

## <a name="advanced-network-administrator-troubleshooting"></a>Dépannage d’administrateur réseau avancé

### <a name="capture-network-trace"></a>Capturer la trace réseau
Capturez une trace réseau simultanée sur la machine virtuelle source, l’appliance virtuelle réseau et la machine virtuelle de destination pendant que vous exécutez **[PsPing](https://docs.microsoft.com/sysinternals/downloads/psping)** ou **Nmap** , puis arrêtez la capture de la trace.

1. Pour capturer une trace réseau simultanée, exécutez la commande suivante :

   **Pour Windows**

   netsh trace start capture=yes tracefile=c:\server_IP.etl scenario=netconnection

   **Pour Linux**

   sudo tcpdump -s0 -i eth0 -X -w vmtrace.cap

2. Utilisez **PsPing** ou **Nmap** à partir de la machine virtuelle source vers la machine virtuelle de destination (par exemple, `PsPing 10.0.0.4:80` ou `Nmap -p 80 10.0.0.4`).
3. Ouvrez la trace réseau à partir de la machine virtuelle de destination en utilisant le [Moniteur réseau](https://download.cnet.com/s/network-monitor) ou la commande tcpdump. Appliquez un filtre d’affichage pour l’adresse IP de la machine virtuelle Source à partir de laquelle vous avez exécuté **PsPing** ou **Nmap** , par exemple, `IPv4.address==10.0.0.4 (Windows netmon)` ou `tcpdump -nn -r vmtrace.cap src or dst host 10.0.0.4` (Linux).

### <a name="analyze-traces"></a>Analyser les traces

Si vous ne voyez pas les paquets entrants dans la trace de machine virtuelle principale, il est probable qu’un groupe de sécurité réseau ou un itinéraire défini par l’utilisateur interfère, ou que les tables de routage de l’appliance virtuelle réseau soient incorrectes.

Si vous voyez les paquets entrants, mais pas de réponse, vous devrez peut-être résoudre un problème lié au pare-feu ou à l’application de la machine virtuelle. Pour chacun de ces problèmes, [contactez au besoin le fournisseur de l’appliance virtuelle réseau](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines).