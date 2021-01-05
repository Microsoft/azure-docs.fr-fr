---
author: vhorne
ms.service: dns
ms.topic: include
ms.date: 11/25/2018
ms.author: victorh
ms.openlocfilehash: 74031a8dbc9b64d6a09533789eed1296ff334d47
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177724"
---
Une zone DNS permet d’héberger les enregistrements DNS d’un domaine particulier. Pour commencer à héberger votre domaine dans le DNS Azure, vous devez créer une zone DNS pour ce nom de domaine. Chaque enregistrement DNS pour votre domaine est ensuite créé à l’intérieur de cette zone DNS.

Par exemple, le domaine « contoso.com » peut contenir plusieurs enregistrements DNS, tels que « mail.contoso.com » (pour un serveur de messagerie) et « www.contoso.com » (pour un site web).

Lorsque vous créez une zone DNS dans Azure DNS :

* Le nom de la zone doit être unique dans le groupe de ressources et la zone ne doit pas déjà exister. Dans le cas contraire, l’opération échoue.
* Vous pouvez réutiliser le même nom de zone dans un autre groupe de ressources ou abonnement Azure.
* Lorsque plusieurs zones partagent le même nom, des adresses de serveur de noms différentes sont attribuées à chaque instance. Vous ne pouvez configurer qu’un seul ensemble d’adresses auprès du bureau d’enregistrement de noms de domaine.

> [!NOTE]
> Pour créer une zone DNS avec un nom de domaine dans le DNS Azure, vous ne devez pas nécessairement être propriétaire de ce nom. En revanche, vous devez être propriétaire du domaine pour pouvoir configurer les serveurs de noms du DNS Azure en tant que serveurs de noms corrects pour le nom de domaine auprès du bureau d’enregistrement de noms de domaine.
> 
> Pour plus d’informations, voir [Délégation de domaine à Azure DNS](../articles/dns/dns-domain-delegation.md).