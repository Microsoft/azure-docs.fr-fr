---
title: Azure VMware Solution by CloudSimple - Configurer le DNS et le DHCP de la charge de travail pour le cloud privé
description: Décrit comment configurer le DNS et le DHCP des applications et des charges de travail s’exécutant dans votre environnement de cloud privé CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: d7531a22d5b0f44fdaae837a75d17ea18852acf0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87077285"
---
# <a name="set-up-dns-and-dhcp-applications-and-workloads-in-your-cloudsimple-private-cloud"></a>Configurer des applications et des charges de travail DNS et DHCP dans votre cloud privé CloudSimple

Les applications et charges de travail exécutées dans un environnement de cloud privé nécessitent une résolution de nom et des services DHCP pour la recherche et l’attribution d’adresses IP.  Une infrastructure DHCP et DNS appropriée est nécessaire pour fournir ces services.  Vous pouvez configurer une machine virtuelle pour fournir ces services dans votre environnement de cloud privé.  

## <a name="prerequisites"></a>Conditions préalables requises

* Un groupe de ports distribués avec un réseau VLAN configuré
* Configuration du routage vers des serveurs DNS locaux ou situés sur Internet
* Modèle de machine virtuelle ou ISO pour créer une machine virtuelle

## <a name="linux-based-dns-server-setup"></a>Configuration d’un serveur DNS basé sur Linux

Linux offre différents packages pour la configuration de serveurs DNS.  Voici un [exemple de configuration de DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-18-04) avec des instructions pour la configuration d’un serveur DNS BIND open source.

## <a name="windows-based-setup"></a>Configuration Windows

Ces rubriques Microsoft expliquent comment configurer un serveur Windows en tant que serveur DNS et en tant que serveur DHCP.

* [Serveur Windows en tant que serveur DNS](/windows-server/networking/dns/dns-top)
* [Serveur Windows en tant que serveur DHCP](/windows-server/networking/technologies/dhcp/dhcp-top)
