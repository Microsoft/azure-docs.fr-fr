---
title: Utilisation d’un DNS dynamique pour inscrire les noms d’hôte dans Azure | Microsoft Docs
description: Découvrez comment configurer un DNS dynamique pour inscrire les noms d’hôte sur vos propres serveurs DNS.
services: dns
documentationcenter: na
author: subsarma
manager: vitinnan
editor: ''
ms.assetid: c315961a-fa33-45cf-82b9-4551e70d32dd
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/23/2017
ms.author: subsarma
ms.openlocfilehash: 9f5b535a341956e5675ba96ba9570bd3f2ff3443
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84710947"
---
# <a name="use-dynamic-dns-to-register-hostnames-in-your-own-dns-server"></a>Utiliser un DNS dynamique pour inscrire les noms d’hôte sur votre propre serveur DNS

[Azure fournit la résolution de noms](virtual-networks-name-resolution-for-vms-and-role-instances.md) pour les machines virtuelles et les instances de rôle. Quand vous avez besoin d’autres fonctionnalités que celles fournies par le DNS par défaut d’Azure pour la résolution de noms, vous pouvez fournir vos propres serveurs DNS. Utiliser vos propres serveurs DNS vous permet de personnaliser votre solution DNS pour l’adapter à vos propres besoins. Par exemple, vous pouvez avoir besoin d’accéder à des ressources locales avec votre contrôleur de domaine Active Directory.

Quand vos serveurs DNS personnalisés sont hébergés en tant que machines virtuelles Azure, vous pouvez transférer les requêtes de nom d’hôte (pour le même réseau virtuel) vers Azure pour résoudre les noms d’hôte. Si vous ne souhaitez pas utiliser cette option, vous pouvez inscrire les noms d’hôte de vos machines virtuelles sur votre serveur DNS à l’aide du DNS dynamique (DDNS, Dynamic DNS). Comme Azure ne dispose pas des informations d’identification permettant de créer directement des enregistrements sur vos serveurs DNS, d’autres mécanismes sont souvent nécessaires. Nous présentons ci-après quelques scénarios courants avec des solutions alternatives :

## <a name="windows-clients"></a>Clients Windows
Les clients Windows non joints à un domaine tentent des mises à jour DDNS non sécurisées pendant le démarrage ou la modification de leurs adresses IP. Le nom DNS se compose du nom d’hôte et du suffixe DNS principal. Azure laisse le suffixe DNS principal vide, mais vous pouvez définir le suffixe dans la machine virtuelle, par le biais de [l’interface utilisateur](https://technet.microsoft.com/library/cc794784.aspx) ou de [PowerShell](/powershell/module/dnsclient/set-dnsclient).

Les clients Windows joints à un domaine inscrivent leurs adresses IP auprès du contrôleur de domaine à l’aide de la mise à jour DDNS sécurisée. Le processus de jonction de domaine définit le suffixe DNS principal sur le client, puis crée et gère la relation d’approbation.

## <a name="linux-clients"></a>Clients Linux
En général, les clients Linux ne s’inscrivent pas eux-mêmes auprès du serveur DNS au démarrage et présument que ce dernier le fait à leur place. Les serveurs DHCP d’Azure ne disposent pas des informations d’identification permettant de stocker des enregistrements sur votre serveur DNS. Vous pouvez utiliser un outil appelé `nsupdate`, qui est inclus dans le package Bind, pour envoyer des mises à jour DDNS. Comme le protocole DDNS est normalisé, vous pouvez utiliser `nsupdate` même quand vous n’utilisez pas Bind sur le serveur DNS.

Vous pouvez utiliser les raccordements fournis par le client DHCP pour créer et gérer l’entrée du nom d’hôte sur le serveur DNS. Au cours du cycle DHCP, le client exécute les scripts dans */etc/dhcp/dhclient-exit-hooks.d/* . Vous pouvez utiliser les raccordements pour inscrire la nouvelle adresse IP à l’aide `nsupdate`. Par exemple :

```bash
#!/bin/sh
requireddomain=mydomain.local

# only execute on the primary nic
if [ "$interface" != "eth0" ]
then
    return
fi

# When you have a new IP, perform nsupdate
if [ "$reason" = BOUND ] || [ "$reason" = RENEW ] ||
   [ "$reason" = REBIND ] || [ "$reason" = REBOOT ]
then
   host=`hostname`
   nsupdatecmds=/var/tmp/nsupdatecmds
     echo "update delete $host.$requireddomain a" > $nsupdatecmds
     echo "update add $host.$requireddomain 3600 a $new_ip_address" >> $nsupdatecmds
     echo "send" >> $nsupdatecmds

     nsupdate $nsupdatecmds
fi
```

Vous pouvez également utiliser la commande `nsupdate` pour effectuer des mises à jour DDNS sécurisées. Par exemple, quand vous utilisez un serveur DNS Bind, une paire de clés publique-privée est générée (`http://linux.yyz.us/nsupdate/`). Le serveur DNS est configuré (`http://linux.yyz.us/dns/ddns-server.html`) avec la partie publique de la clé, ce qui lui permet de vérifier la signature sur la demande. Pour fournir la paire de clés à `nsupdate`, utilisez l’option `-k`, afin que la demande de mise à jour DDNS soit signée.

Quand vous utilisez un serveur DNS Windows, vous pouvez utiliser l’authentification Kerberos avec le paramètre `-g` dans `nsupdate` (mais elle n’est pas disponible dans la version Windows de `nsupdate`). Pour utiliser Kerberos, utilisez `kinit` afin de charger les informations d’identification. Par exemple, vous pouvez charger les informations d’identification à partir d’un [fichier keytab](https://www.itadmintools.com/2011/07/creating-kerberos-keytab-files.html), puis `nsupdate -g` récupère les informations d’identification à partir du cache.

Si nécessaire, vous pouvez ajouter un suffixe de recherche DNS à vos machines virtuelles. Le suffixe DNS est spécifié dans le fichier */etc/resolv.conf* . Comme la plupart des distributions Linux gèrent automatiquement le contenu de ce fichier, il ne peut généralement pas être modifié. Toutefois, vous pouvez remplacer le suffixe à l’aide de la commande `supersede` du client DHCP. Pour remplacer le suffixe, ajoutez la ligne suivante au fichier */etc/dhcp/dhclient.conf* :

```
supersede domain-name <required-dns-suffix>;
```
