---
title: Se préparer à un changement d’adresse IP sortante
description: Si votre adresse IP sortante va être modifiée, découvrez les choses à faire pour que votre application continue de fonctionner après la modification.
ms.topic: article
ms.date: 06/28/2018
ms.custom: seodec18
ms.openlocfilehash: 2be4bc92dde278b054bd04f412f937440027ece7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74671662"
---
# <a name="how-to-prepare-for-an-outbound-ip-address-change"></a>Comment se préparer à un changement d’adresse IP sortante

Si vous avez reçu une notification indiquant que les adresses IP sortantes de votre application Azure App Service vont être modifiées, suivez les instructions de cet article.

## <a name="determine-if-you-have-to-do-anything"></a>Déterminer si vous avez quelque chose à faire

* Option 1 : si votre application App Service n’utilise pas le filtrage IP, une liste d’inclusion explicite, ou un traitement spécial du trafic sortant tel que le routage ou le pare-feu, aucune action n’est requise.

* Option 2 : si votre application procède à un traitement spécial des adresses IP sortantes (voir les exemples ci-dessous), ajoutez les nouvelles adresses IP sortantes partout où apparaissent les adresses existantes. Ne remplacez pas les adresses IP existantes. Vous trouverez les nouvelles adresses IP sortantes en suivant les instructions fournies dans la section suivante.

  Par exemple, une adresse IP sortante peut être explicitement incluse dans un pare-feu en dehors de votre application, ou un service de paiement externe peut avoir une liste autorisée qui contient l’adresse IP sortante pour votre application. Si votre adresse sortante est configurée dans une liste en dehors de votre application, elle doit être modifiée.

## <a name="find-the-outbound-ip-addresses-in-the-azure-portal"></a>Trouver les adresses IP sortantes dans le portail Azure

Les nouvelles adresses IP sortantes sont affichées dans le portail avant leur entrée en vigueur. Lorsqu’Azure démarre en utilisant les nouvelles adresses, les anciennes ne peuvent plus être utilisées. Un seul ensemble est utilisé à la fois, c’est pourquoi les entrées des listes d’inclusion doivent disposer des anciennes et des nouvelles adresses IP afin d’éviter une interruption lors du changement. 

1.  Ouvrez le [portail Azure](https://portal.azure.com).

2.  Dans le menu de navigation de gauche, sélectionnez **App Services**.

3.  Sélectionnez votre application App Service dans la liste.

1.  Si l’application est une application de fonction, consultez [Function app outbound IP addresses](../azure-functions/ip-addresses.md#find-outbound-ip-addresses) (Adresses IP sortantes de l’application de fonction).

4.  Sous l’en-tête **Paramètres**, cliquez sur **Propriétés** dans le volet de navigation de gauche, puis recherchez la section intitulée **Adresses IP sortantes**.

5. Copiez les adresses IP et ajoutez-les à votre dispositif de traitement spécial du trafic sortant (filtre, liste autorisée...). Ne supprimez pas les adresses IP existantes dans la liste.

## <a name="next-steps"></a>Étapes suivantes

Cet article a expliqué comment se préparer à un changement d’adresse IP initié par Azure. Pour plus d’informations sur les adresses IP dans Azure App Service, consultez [Inbound and Inbound IP addresses in Azure App Service](overview-inbound-outbound-ips.md) (Entrées et adresses IP entrantes dans Azure App Service).
