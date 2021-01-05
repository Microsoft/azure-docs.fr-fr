---
title: Vue d’ensemble des enregistrements d’alias – Azure DNS
description: Dans cet article, découvrez en plus sur la prise en charge des enregistrements d’alias dans Microsoft Azure DNS.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 08/09/2019
ms.author: rohink
ms.openlocfilehash: 8b2576669357aae7e5fe423515933c2ce4a23a7d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94954475"
---
# <a name="azure-dns-alias-records-overview"></a>Vue d’ensemble des enregistrements d’alias Azure DNS

Les enregistrements d’alias Azure DNS sont des qualifications sur un jeu d’enregistrements DNS. Ils peuvent référencer d’autres ressources Azure de votre zone DNS. Par exemple, vous pouvez créer un jeu d’enregistrements d’alias qui référence une adresse IP publique Azure, et non un enregistrement A. Votre jeu d’enregistrements d’alias pointe de façon dynamique vers une instance du service d’adresse IP publique Azure. Par conséquent, le jeu d’enregistrements d’alias se met à jour sans interruption pendant la résolution DNS.

Un jeu d’enregistrements d’alias est pris en charge pour les types d’enregistrements suivants dans une zone Azure DNS : 

- Un
- AAAA
- CNAME

> [!NOTE]
> Si vous envisagez d'utiliser un enregistrement d'alias pour les types d'enregistrements A ou AAAA afin de pointer vers un profil [​Azure Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md), vous devez vous assurer que le profil Traffic Manager ne comporte que des [points de terminaison externes](../traffic-manager/traffic-manager-endpoint-types.md#external-endpoints). Vous devez fournir l'adresse IPv4 ou IPv6 des points de terminaison externes dans Traffic Manager. Vous ne pouvez pas utiliser des noms de domaine complets (FQDN) dans des points de terminaison. Idéalement, utilisez des adresses IP statiques.

## <a name="capabilities"></a>Fonctionnalités

- **Pointer vers une ressource d’adresse IP publique à partir d’un jeu d’enregistrements A/AAAA DNS.** Vous pouvez créer un jeu d’enregistrements A/AAAA et en faire un jeu d’enregistrements d’alias pour pointer vers une ressource d’adresse IP publique (standard ou de base). Le jeu d’enregistrements DNS change automatiquement si l’adresse IP publique change ou est supprimée. Les enregistrements DNS non résolus qui pointent vers des adresses IP incorrectes sont évités.

   Il existe une limite actuelle de 20 jeux d’enregistrements d’alias par ressource.

- **Pointer vers un profil Traffic Manager à partir d’un jeu d’enregistrements A/AAAA/CNAME DNS.** Vous pouvez créer un jeu d'enregistrements A/AAAA ou CNAME et utiliser des enregistrements d'alias pour le diriger vers un profil Traffic Manager. Cela s’avère particulièrement utile quand vous devez router le trafic à un apex de zone, car les enregistrements CNAME traditionnels ne sont pas pris en charge pour un apex de zone. Supposons par exemple que votre profil Traffic Manager soit myprofile.trafficmanager.net et que la zone DNS de votre entreprise soit contoso.com. Vous pouvez créer un jeu d'enregistrements d'alias de type A/AAAA pour contoso.com (l'extrémité de la zone) et pointer vers myprofile.trafficmanager.net.
- **Pointer vers un point de terminaison Azure CDN (Content Delivery Network)** . C’est utile quand vous créez des sites web statiques à l’aide du Stockage Azure et d’Azure CDN.
- **Pointer vers un autre jeu d’enregistrements DNS au sein de la même zone.** Les enregistrements d’alias peuvent référencer d’autres jeux d’enregistrements du même type. Par exemple, un jeu d'enregistrements DNS CNAME peut être un alias d'un autre jeu d'enregistrements CNAME. Cette disposition est utile si vous voulez que certains jeux d’enregistrements d’alias soient des alias et d’autres des non-alias.

## <a name="scenarios"></a>Scénarios

Voici quelques scénarios courants mettant en œuvre des enregistrements d’alias.

### <a name="prevent-dangling-dns-records"></a>Empêcher les enregistrements DNS non résolus

Les enregistrements non résolus constituent un problème courant avec les enregistrements DNS traditionnels. Par exemple, les enregistrements DNS qui n’ont pas été mis à jour pour refléter les changements apportés aux adresses IP. Ce problème se produit en particulier avec les types d'enregistrements A/AAAA ou CNAME.

Avec un enregistrement de zone DNS traditionnel, si l'adresse IP ou l'enregistrement CNAME cible n'existe plus, l'enregistrement DNS qui lui est associé doit être mis à jour manuellement. Dans certaines organisations, une mise à jour manuelle peut ne pas avoir lieu à temps en raison de problèmes liées au processus ou de la séparation des rôles et des niveaux d’autorisation associés. Par exemple, un rôle peut avoir l’autorisation de supprimer un CNAME ou une adresse IP appartenant à une application. Mais il ne dispose pas d’autorisations suffisantes pour mettre à jour l’enregistrement DNS qui pointe vers ces cibles. Un retard de mise à jour de l'enregistrement DNS peut potentiellement provoquer une interruption pour les utilisateurs.

Les enregistrements d'alias empêchent les références non résolues en associant étroitement le cycle de vie d'un enregistrement DNS à une ressource Azure. Prenons l'exemple d'un enregistrement DNS qualifié en tant qu'enregistrement d'alias et pointant vers une adresse IP publique ou un profil Traffic Manager. Si vous supprimez ces ressources sous-jacentes, l’enregistrement d’alias DNS devient un jeu d’enregistrements vide. Il ne fait plus référence à la ressource supprimée.

### <a name="update-dns-record-set-automatically-when-application-ip-addresses-change"></a>Mise à jour automatique de l'enregistrement DNS lorsque les adresses IP de l'application changent

Ce scénario est semblable au précédent. Une application est peut-être déplacée ou la machine virtuelle sous-jacente redémarrée. Un enregistrement d’alias est alors automatiquement mis à jour quand l’adresse IP change pour la ressource d’adresse IP publique sous-jacente. Cela prévient les risques de sécurité liés à l'acheminement des utilisateurs vers une autre application à laquelle l'ancienne adresse IP publique a été attribuée.

### <a name="host-load-balanced-applications-at-the-zone-apex"></a>Héberger des applications à charge équilibrée au niveau de l’apex de zone

Le protocole DNS empêche l'attribution d'enregistrements CNAME à l'extrémité de la zone. Par exemple, si votre domaine est contoso.com, vous pouvez créer des enregistrements CNAME pour somelabel.contoso.com, mais vous ne pouvez pas en créer pour contoso.com lui-même.
Cette restriction pose un problème aux propriétaires d'applications qui disposent d'applications à charge équilibrée derrière [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md). Dans la mesure où l’utilisation d’un profil Traffic Manager nécessite la création d’un enregistrement CNAME, il est impossible de pointer vers le profil Traffic Manager à partir de l’apex de zone.

Ce problème est résolu en utilisant des enregistrements d’alias. Contrairement aux enregistrements CNAME, les enregistrements d’alias sont créés à l’apex de zone, et les propriétaires d’applications peuvent l’utiliser pour pointer leur enregistrement d’apex de zone vers un profil Traffic Manager doté de points de terminaison externes. Les propriétaires d’applications peuvent pointer vers le même profil Traffic Manager que celui utilisé pour tout autre domaine de leur zone DNS.

Par exemple, contoso.com et www\.contoso.com peuvent pointer vers le même profil Traffic Manager. Pour en savoir plus sur l'utilisation des enregistrements d'alias avec les profils Azure Traffic Manager, consultez la section Étapes suivantes.

### <a name="point-zone-apex-to-azure-cdn-endpoints"></a>Pointer l’apex de zone vers des points de terminaison Azure CDN

Tout comme un profil Traffic Manager, vous pouvez également utiliser des enregistrements d’alias pour pointer votre apex de zone DNS vers des points de terminaison Azure CDN. C’est utile quand vous créez des sites web statiques à l’aide du Stockage Azure et d’Azure CDN. Vous pouvez ensuite accéder au site web sans ajouter le préfixe « www » à votre nom DNS.

Par exemple, si votre site web statique se nomme `www.contoso.com`, vos utilisateurs peuvent accéder à votre site à l’aide de `contoso.com` sans avoir besoin d’ajouter le préfixe « www » au nom DNS.

Comme décrit précédemment, les enregistrements CNAME ne sont pas pris en charge à l’apex de zone. Par conséquent, vous ne pouvez pas utiliser un enregistrement CNAME pour pointer contoso.com vers votre point de terminaison CDN. À la place, vous pouvez utiliser un enregistrement d’alias pour pointer l’apex de zone directement sur un point de terminaison CDN.

> [!NOTE]
> Le fait de pointer un apex de zone vers des points de terminaison CDN pour Azure CDN fourni par Akamai n’est actuellement pas pris en charge.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les enregistrements d’alias, consultez les articles suivant :

- [Tutoriel : Configurer un enregistrement d'alias pour faire référence à une adresse IP publique](tutorial-alias-pip.md)
- [Tutoriel : Configurer un enregistrement d'alias pour prendre en charge des noms de domaine d'apex avec Traffic Manager](tutorial-alias-tm.md)
- [FAQ DNS](./dns-faq.md#alias-records)