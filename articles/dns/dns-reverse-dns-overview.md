---
title: Aperçu du DNS inversé dans Azure - Azure DNS
description: Dans ce parcours d’apprentissage, commencez à découvrir comment fonctionne le DNS inversé et comment il peut être utilisé dans Azure
services: dns
documentationcenter: na
author: rohinkoul
manager: KumuD
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/29/2017
ms.author: rohink
ms.openlocfilehash: 8af9549efc3e8dab54f55dd404346d87201dee2c
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965610"
---
# <a name="overview-of-reverse-dns-and-support-in-azure"></a>Vue d’ensemble du DNS inversé et prise en charge dans Azure

Cet article donne une vue d’ensemble du fonctionnement du DNS inversé et des scénarios DNS inversés pris en charge dans Azure.

## <a name="what-is-reverse-dns"></a>Qu’est-ce que le DNS inversé ?

Les enregistrements DNS traditionnels permettent d’effectuer des mappages à partir d’un nom de DNS (par exemple, « www.contoso.com ») vers une adresse IP (par exemple, 64.4.6.100).  Le DNS inversé permet la traduction d’une adresse IP (64.4.6.100) en nom (« www.contoso.com »).

Les enregistrements DNS inversés sont utilisés dans de nombreuses situations. Par exemple, les enregistrements DNS inversés sont largement utilisés dans la lutte contre les messages indésirables en vérifiant l’expéditeur d’un message électronique.  Le serveur des courriers entrants récupère l’enregistrement DNS inversé de l’adresse IP du serveur d’envoi et vérifie si cet hôte est autorisé à envoyer des e-mails depuis le domaine d’origine. 

## <a name="how-reverse-dns-works"></a>Fonctionnement du DNS inversé

Les enregistrements DNS inversés sont hébergés dans des zones DNS spéciales, appelées zones « ARPA ».  Ces zones forment une hiérarchie DNS distincte en parallèle avec la hiérarchie classique hébergeant des domaines tels que « contoso.com ».

Par exemple, l’enregistrement DNS « www.contoso.com » est implémenté à l’aide d’un enregistrement DNS « A » avec le nom « www » dans la zone « contoso.com ».  Cet enregistrement A pointe vers l’adresse IP correspondante, dans ce cas 64.4.6.100.  La recherche inversée est implémentée séparément, à l’aide d’un enregistrement « PTR » nommé « 100 » dans la zone « 6.4.64.in-addr.arpa » (notez que les adresses IP sont inversées dans les zones ARPA).  Cet enregistrement PTR, s’il a été configuré correctement, pointe vers le nom « www.contoso.com ».

Lorsqu’une organisation est affectée à un bloc d’adresses IP, elle acquiert également le droit de gestion de la zone ARPA correspondante. Les zones ARPA qui correspondent aux blocs d’adresses IP utilisés par Azure sont hébergées et gérées par Microsoft. Votre fournisseur de services Internet peut héberger la zone ARPA pour vos propres adresses IP, ou peut vous autoriser à héberger la zone ARPA dans le service DNS de votre choix, par exemple Azure DNS.

> [!NOTE]
> Les recherches DNS directes et inversées sont implémentées dans des hiérarchies DNS distinctes, en parallèle. La recherche inversée pour « www.contoso.com » n’est **pas** hébergée dans la zone « contoso.com », mais dans la zone ARPA pour le bloc d’adresses IP correspondant. Plusieurs zones distinctes sont utilisées pour les blocs d’adresses IPv4 et IPv6.

### <a name="ipv4"></a>IPv4

Le nom d’une zone de recherche inversée IPv4 doit se présenter sous la forme suivante : `<IPv4 network prefix in reverse order>.in-addr.arpa`.

Par exemple, lorsque vous créez une zone inversée pour héberger des enregistrements pour les hôtes dont les adresses IP contiennent le préfixe 192.0.2.0/24, le nom de zone est créé en isolant le préfixe réseau de l’adresse (192.0.2), en inversant l’ordre (2.0.192), puis en ajoutant le suffixe `.in-addr.arpa`.

|Classe de sous-réseau|Préfixe réseau  |Préfixe réseau inversé  |Suffixe standard  |Nom de zone inversé |
|-------|----------------|------------|-----------------|---------------------------|
|Classe A|203.0.0.0/8     | 203        | .in-addr.arpa   | `203.in-addr.arpa`        |
|Classe B|198.51.0.0/16   | 51.198     | .in-addr.arpa   | `51.198.in-addr.arpa`     |
|Classe C|192.0.2.0/24    | 2.0.192    | .in-addr.arpa   | `2.0.192.in-addr.arpa`    |

### <a name="classless-ipv4-delegation"></a>Délégation de IPv4 sans classe

Dans certains cas, la plage d’adresses IP allouée à une organisation est inférieure à une plage de classe C (/ 24). Dans ce cas, la plage d’adresses IP ne tombe pas dans une zone limite à l’intérieur de la `.in-addr.arpa` hiérarchie de zone. Par conséquent, elle ne peut pas être déléguée en tant que zone enfant.

Au lieu de cela, un mécanisme différent est utilisé pour transférer le contrôle des enregistrements de chaque recherche inversée (PTR) vers une zone DNS dédiée. Ce mécanisme délègue une zone enfant pour chaque plage d’adresses IP, puis mappe chaque adresse IP de la plage individuellement vers cette zone enfant à l’aide d’enregistrements CNAME.

Par exemple, supposons qu’une organisation se voit attribué la plage d’adresses IP 192.0.2.128/26 par son fournisseur de services Internet. Cela représente 64 adresses IP, allant de 192.0.2.128 à 192.0.2.191. Le DNS inversé de cette plage est mis en œuvre comme suit :
- L’organisation crée une zone de recherche inversée appelée 128-26.2.0.192.in-addr.arpa. Le préfixe « 128-26 » représente le segment réseau attribué à l’organisation au sein de la plage de Classe C (/ 24).
- Le fournisseur de services Internet crée des enregistrements NS afin de configurer la délégation DNS pour la zone ci-dessus depuis la zone parente de Classe C. Il crée également des enregistrements CNAME dans la zone de recherche inversée parente (Classe C), en mappant chaque adresse IP dans la plage d’adresses IP vers la nouvelle zone créée par l’organisation :

```
$ORIGIN 2.0.192.in-addr.arpa
; Delegate child zone
128-26    NS       <name server 1 for 128-26.2.0.192.in-addr.arpa>
128-26    NS       <name server 2 for 128-26.2.0.192.in-addr.arpa>
; CNAME records for each IP address
129       CNAME    129.128-26.2.0.192.in-addr.arpa
130       CNAME    130.128-26.2.0.192.in-addr.arpa
131       CNAME    131.128-26.2.0.192.in-addr.arpa
; etc
```
- L’organisation gère les enregistrements PTR individuels au sein de leur zone enfant.

```
$ORIGIN 128-26.2.0.192.in-addr.arpa
; PTR records for each UIP address. Names match CNAME targets in parent zone
129      PTR    www.contoso.com
130      PTR    mail.contoso.com
131      PTR    partners.contoso.com
; etc
```
Une recherche inversée pour les requêtes de l’adresse IP « 192.0.2.129 » pour un enregistrement PTR nommé « 129.2.0.192.in-addr.arpa ». Cette requête résout l’enregistrement PTR dans la zone enfant via le CNAME dans la zone parente.

### <a name="ipv6"></a>IPv6

Le nom d’une zone de recherche inversée IPv6 doit se présenter sous la forme suivante : `<IPv6 network prefix in reverse order>.ip6.arpa`

Par exemple, lorsque vous créez une zone inversée afin d’héberger des enregistrements pour des hôtes dont les adresses IP contiennent le préfixe 2001:db8:1000:abdc::/64, le nom de zone est créé en isolant le préfixe réseau de l’adresse (2001:db8:abdc::). Développez ensuite le préfixe réseau IPv6 pour supprimer [la compression des zéros](/previous-versions/windows/it-pro/windows-server-2003/cc781672(v=ws.10)), s’il a été utilisé pour raccourcir le préfixe d’adresse IPv6 (2001:0db8:abdc:0000::). Inversez l’ordre, à l’aide d’un point comme séparateur entre chaque nombre hexadécimal dans le préfixe, pour générer le préfixe réseau inversé (`0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2`) et ajouter le suffixe `.ip6.arpa`.


|Préfixe réseau  |Préfixe réseau développé et inversé |Suffixe standard |Nom de zone inversé  |
|---------|---------|---------|---------|
|2001:db8:ABDC::/64    | 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2        | .ip6.arpa        | `0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa`       |
|2001:db8:1000:9102::/64    | 2.0.1.9.0.0.0.1.8.b.d.0.1.0.0.2        | .ip6.arpa        | `2.0.1.9.0.0.0.1.8.b.d.0.1.0.0.2.ip6.arpa`        |


## <a name="azure-support-for-reverse-dns"></a>Prise en charge Azure pour le DNS inversé

Azure prend en charge deux scénarios distincts relatifs au DNS inversé :

**Hébergement de la zone de recherche inversée correspondant à votre bloc d’adresses IP.**
Azure DNS peut être utilisé pour [héberger vos zones de recherche inversée et gérer les enregistrements PTR pour chaque recherche DNS inversée](dns-reverse-dns-hosting.md), IPv4 et IPv6.  Le processus de création de la zone de recherche inversée (ARPA), de configuration de la délégation et des enregistrements PTR est identique à celui des zones DNS standard.  Les seules différences sont que la délégation doit être configurée via votre fournisseur de services Internet plutôt que via votre bureau d’enregistrement de DNS, et seul le type d’enregistrement PTR doit être utilisé.

**Configurer l’enregistrement DNS inversé pour l’adresse IP attribuée à votre service Azure.** Azure vous permet de [configurer la recherche inversée pour les adresses IP allouées à votre service Azure](dns-reverse-dns-for-azure-services.md).  Cette recherche inversée est configurée par Azure comme un enregistrement PTR dans la zone ARPA correspondante.  Ces zones ARPA, qui correspondent à toutes les plages d’adresses IP utilisées par Azure, sont hébergées par Microsoft.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le DNS inversé, consultez [Recherche DNS inversée sur Wikipedia](https://en.wikipedia.org/wiki/Reverse_DNS_lookup).
<br>
Découvrez comment [héberger la zone de recherche inversée pour la plage d’adresses IP fournie par votre fournisseur de services Internet dans Azure DNS](dns-reverse-dns-for-azure-services.md).
<br>
Découvrez comment [gérer des enregistrements DNS inversés pour vos services Azure](dns-reverse-dns-for-azure-services.md).