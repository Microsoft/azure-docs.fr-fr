---
author: vhorne
ms.service: dns
ms.topic: include
ms.date: 11/25/2018
ms.author: victorh
ms.openlocfilehash: 8ca054b3a3d5147b7d98a021ce1e26d02d5581b0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011560"
---
### <a name="record-names"></a>Noms d’enregistrement

Dans Azure DNS, les enregistrements sont spécifiés à l’aide de noms relatifs. Un *nom de domaine complet (FQDN)* inclut le nom de la zone, contrairement à un *nom relatif*. Par exemple, le nom d’enregistrement relatif `www` dans la zone `contoso.com` donne le nom d’enregistrement complet `www.contoso.com`.

Un enregistrement *apex* est un enregistrement DNS à la racine (ou *apex*) d’une zone DNS. Par exemple, dans la zone DNS `contoso.com`, un enregistrement apex porte également le nom complet `contoso.com` (on parle parfois de domaine *nu*).  Par convention, le nom relatif « \@ » est utilisé pour représenter des enregistrements apex.

### <a name="record-types"></a>Types d’enregistrements

Chaque enregistrement DNS a un nom et un type. Les enregistrements sont organisés selon différents types, en fonction des données qu’ils contiennent. Le type le plus courant est un enregistrement « A » qui mappe un nom à une adresse IPv4. Un autre type courant est un enregistrement « MX » qui mappe un nom à un serveur de messagerie.

Azure DNS prend en charge tous les types d’enregistrement DNS courants, à savoir : A, AAAA, CAA, CNAME, MX, NS, PTR, SOA, SRV et TXT. Notez que les [enregistrements SPF sont représentés à l’aide d’enregistrements TXT](../articles/dns/dns-zones-records.md#spf-records).

### <a name="record-sets"></a>Jeux d’enregistrements

Vous devez parfois créer plusieurs enregistrements DNS avec un nom et un type donnés. Par exemple, supposons que le site web « www.contoso.com » est hébergé sur deux adresses IP différentes. Ce site web requiert deux enregistrements A différents, à savoir un pour chaque adresse IP. Voici un exemple de jeu d’enregistrements :

```dns
www.contoso.com.        3600    IN    A    134.170.185.46
www.contoso.com.        3600    IN    A    134.170.188.221
```

Le DNS Azure gère l’ensemble des enregistrements DNS à l’aide de *jeux d’enregistrements*. Un jeu d’enregistrements (également appelé *jeu d’enregistrements de ressource*) est la collection d’enregistrements DNS dans une zone qui portent le même nom et sont du même type. La plupart des jeux d’enregistrements contiennent un seul enregistrement. Toutefois, les exemples comme celui ci-dessus, dans lequel un jeu d’enregistrements contient plusieurs enregistrements, sont relativement courants.

Par exemple, supposons que vous avez déjà créé un enregistrement A nommé « www » dans la zone « contoso.com », qui pointe vers l’adresse IP « 134.170.185.46 » (le premier enregistrement ci-dessus).  Pour créer le deuxième enregistrement, vous devez l’ajouter au jeu d’enregistrements existant, au lieu de créer un autre jeu.

Les types d’enregistrements SOA et CNAME font exception. Comme les normes DNS n’autorisent pas que plusieurs enregistrements portent le même nom pour ces types, ces jeux d’enregistrements ne peuvent contenir qu’un seul enregistrement.