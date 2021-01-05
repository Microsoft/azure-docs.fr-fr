---
title: Filtrage de nom de domaine complet du Pare-feu Azure dans les règles de réseau
description: Utiliser le filtrage de nom de domaine complet Pare-feu Azure dans des règles de réseau
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 11/06/2020
ms.author: victorh
ms.openlocfilehash: f7196c7715ad5d2c02759040b780b96218e1655e
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94695943"
---
# <a name="use-fqdn-filtering-in-network-rules"></a>Utiliser le filtrage de nom de domaine complet dans les règles de réseau

Un nom de domaine complet (FQDN) représente le nom de domaine d’un hôte ou d’une adresse IP. Vous pouvez utiliser des noms de domaine complets dans les règles de réseau basées sur la résolution DNS dans le pare-feu et la stratégie de pare-feu Azure. Cette fonctionnalité vous permet de filtrer le trafic sortant avec n’importe quel protocole TCP/UDP (y compris NTP, SSH, RDP, etc.). Pour utiliser des noms de domaine complet (FQDN) dans des règles de réseau, vous devez activer le proxy DNS. Pour plus d’informations, consultez [Paramètres DNS du Pare-feu Azure](dns-settings.md).

> [!NOTE]
> Par conception, le filtrage FQDN ne prend pas en charge les caractères génériques.

## <a name="how-it-works"></a>Fonctionnement

Une fois que vous avez défini le serveur DNS dont votre organisation a besoin (Azure DNS ou votre propre DNS personnalisé), le pare-feu Azure convertit le nom de domaine complet en adresse IP basée sur le serveur DNS sélectionné. Cette traduction se produit à la fois pour le traitement des règles d’application et de réseau.

Quand une nouvelle résolution DNS prend place, de nouvelles adresses IP sont ajoutées aux règles de pare-feu. Les anciennes adresses IP qui ne sont plus renvoyées par le serveur DNS expirent après 15 minutes. Les règles du Pare-feu Azure sont mises à jour toutes les 15 secondes à partir de la résolution DNS des noms de domaine complets dans les règles réseau.

### <a name="differences-in-application-rules-vs-network-rules"></a>Différences entre les règles d’application et les règles de réseau

- Le filtrage de nom de domaine complet dans les règles d’application pour HTTP/S et MSSQL est basé sur un proxy transparent au niveau de l’application et sur l’en-tête SNI. Par conséquent, il peut discerner deux noms de domaine complets qui sont résolus sur la même adresse IP. Ce n’est pas le cas avec le filtrage de nom de domaine complet dans les règles de réseau. 

   Utilisez des règles d’application lorsque cela est possible :
     - Si le protocole est HTTP/S ou MSSQL, utilisez des règles d’application pour le filtrage de nom de domaine complet.
   - Pour tous les protocoles autres que HTTP/S ou MSSQL, vous pouvez utiliser des règles d’application ou de réseau pour le filtrage de nom de domaine complet.

## <a name="next-steps"></a>Étapes suivantes

[Paramètres DNS du Pare-feu Azure](dns-settings.md)
