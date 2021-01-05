---
title: Configurer DHCPv6 pour les machines virtuelles Linux
titleSuffix: Azure Load Balancer
description: Dans cet article, découvrez comment configurer DHCPv6 pour les machines virtuelles Linux.
services: load-balancer
documentationcenter: na
author: asudbring
keywords: IPv6, équilibreur de charge azure, double pile, adresse ip publique, ipv6 natif, mobile, iot
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/22/2019
ms.author: allensu
ms.openlocfilehash: d8bd62bab627beb70a8fcba276bf8c2eca309c45
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006744"
---
# <a name="configure-dhcpv6-for-linux-vms"></a>Configurer DHCPv6 pour les machines virtuelles Linux


Certaines des images de machines virtuelles Linux de la Place de marché Microsoft Azure ne présentent pas de configuration DHCPv6 (Dynamic Host Configuration Protocol version 6) par défaut. Pour prendre en charge IPv6, DHCPv6 doit être configuré dans la distribution du système d’exploitation Linux que vous utilisez. Les distributions Linux configurent DHCPv6 de diverses façons, car elles utilisent différents packages.

> [!NOTE]
> Les images SUSE Linux et CoreOS récentes de la Place de marché Azure ont été préconfigurées avec DHCPv6. Aucun changement supplémentaire n’est exigé quand vous utilisez ces images.

Ce document vous explique comment activer DHCPv6 pour que votre machine virtuelle Linux obtienne une adresse IPv6.

> [!WARNING]
> Toute modification incorrecte des fichiers de configuration réseau peut vous faire perdre l’accès réseau à votre machine virtuelle. Nous vous recommandons de tester vos modifications de configuration sur des systèmes hors production. Les instructions de cet article ont été testées sur les dernières versions des images Linux dans la Place de marché Azure. Pour obtenir des instructions plus détaillées, consultez la documentation propre à votre version de Linux.

## <a name="ubuntu"></a>Ubuntu

1. Modifiez le fichier */etc/dhcp/dhclient6.conf* et ajoutez la ligne suivante :

    ```config
    timeout 10;
    ```

2. Modifiez la configuration du réseau pour l’interface eth0, selon la méthode suivante :

   * Sur **Ubuntu 12.04 et 14.04**, modifiez le fichier */etc/network/interfaces.d/eth0.cfg*. 
   * Sur **Ubuntu 16.04**, modifiez le fichier */etc/network/interfaces.d/50-cloud-init.cfg*.

    ```config
    iface eth0 inet6 auto
        up sleep 5
        up dhclient -1 -6 -cf /etc/dhcp/dhclient6.conf -lf /var/lib/dhcp/dhclient6.eth0.leases -v eth0 || true
    ```

3. Renouvelez l’adresse IPv6 :

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

À compter d’Ubuntu 17.10, [NETPLAN]( https://netplan.io) est le mécanisme de configuration de réseau par défaut.  Au moment de l’installation/instanciation, NETPLAN lit la configuration du réseau dans les fichiers de configuration YAML situés à cet emplacement : /{lib,etc,run}/netplan/*.yaml.

Vous devez ajouter une instruction *dhcp6:true* pour chaque interface Ethernet dans votre configuration.  Par exemple :

```config
network:
  version: 2
  ethernets:
    eno1:
      dhcp6: true
```

Pendant le démarrage anticipé, le NETPLAN « network renderer » écrit la configuration dans /run afin de passer le contrôle des appareils au démon réseau spécifié. Pour obtenir des informations de référence sur NETPLAN, consultez https://netplan.io/reference.
 
## <a name="debian"></a>Debian

1. Modifiez le fichier */etc/dhcp/dhclient6.conf* et ajoutez la ligne suivante :

    ```config
    timeout 10;
    ```

2. Modifiez le fichier */etc/network/interfaces*, puis ajoutez la configuration suivante :

    ```config
    iface eth0 inet6 auto
        up sleep 5
        up dhclient -1 -6 -cf /etc/dhcp/dhclient6.conf -lf /var/lib/dhcp/dhclient6.eth0.leases -v eth0 || true
    ```

3. Renouvelez l’adresse IPv6 :

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

## <a name="rhel-centos-and-oracle-linux"></a>RHEL, CentOS et Oracle Linux

1. Modifiez le fichier */etc/sysconfig/network*, puis ajoutez le paramètre suivant :

    ```config
    NETWORKING_IPV6=yes
    ```

2. Modifiez le fichier */etc/sysconfig/network-scripts/ifcfg-eth0*, puis ajoutez les deux paramètres suivants :

    ```config
    IPV6INIT=yes
    DHCPV6C=yes
    ```

3. Renouvelez l’adresse IPv6 :

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

## <a name="sles-11-and-opensuse-13"></a>SLES 11 et openSUSE 13

Les images SLES (SUSE Linux Enterprise Server) et openSUSE récentes d’Azure ont été préconfigurées avec DHCPv6. Aucun changement supplémentaire n’est exigé quand vous utilisez ces images. Si vous avez une machine virtuelle basée sur une image SUSE plus ancienne ou personnalisée, effectuez les étapes suivantes :

1. Installez le package `dhcp-client` , si nécessaire :

    ```bash
    sudo zypper install dhcp-client
    ```

2. Modifiez le fichier */etc/sysconfig/network/ifcfg-eth0*, puis ajoutez le paramètre suivant :

    ```config
    DHCLIENT6_MODE='managed'
    

3. Renew the IPv6 address:

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

## <a name="sles-12-and-opensuse-leap"></a>SLES 12 et openSUSE Leap

Les images SLES et openSUSE récentes d’Azure ont été préconfigurées avec DHCPv6. Aucun changement supplémentaire n’est exigé quand vous utilisez ces images. Si vous avez une machine virtuelle basée sur une image SUSE plus ancienne ou personnalisée, effectuez les étapes suivantes :

1. Modifiez le fichier */etc/sysconfig/network/ifcfg-eth0*, puis remplacez le paramètre `#BOOTPROTO='dhcp4'` par la valeur suivante :

    ```config
    BOOTPROTO='dhcp'
    ```

2. Ajoutez le paramètre suivant au fichier */etc/sysconfig/network/ifcfg-eth0* :

    ```config
    DHCLIENT6_MODE='managed'
    ```

3. Renouvelez l’adresse IPv6 :

    ```bash
    sudo ifdown eth0 && sudo ifup eth0
    ```

## <a name="coreos"></a>CoreOS

Les images CoreOS récentes d’Azure ont été préconfigurées avec DHCPv6. Aucun changement supplémentaire n’est exigé quand vous utilisez ces images. Si vous avez une machine virtuelle basée sur une image CoreOS plus ancienne ou personnalisée, effectuez les étapes suivantes :

1. Modifiez le fichier */etc/systemd/network/10_dhcp.network* :

    ```config
    [Match]
    eth0

    [Network]
    DHCP=ipv6
    ```

2. Renouvelez l’adresse IPv6 :

    ```bash
    sudo systemctl restart systemd-networkd
    ```
