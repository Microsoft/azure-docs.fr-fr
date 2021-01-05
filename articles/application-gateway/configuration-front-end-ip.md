---
title: Configuration d’adresse IP frontale d’Azure Application Gateway
description: Cet article explique comment configurer l’adresse IP frontale d’Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: surmb
ms.openlocfilehash: dc5efd6ad478710ba839634a49f041211756af71
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397669"
---
# <a name="application-gateway-front-end-ip-address-configuration"></a>Configuration d’adresse IP frontale d’Application Gateway

Vous pouvez configurer la passerelle d’application pour qu’elle ait une adresse IP publique, une adresse IP privée ou les deux. Une adresse IP publique est nécessaire quand vous hébergez un back-end auquel les clients doivent accéder par Internet par le biais d’une adresse IP virtuelle Internet.

## <a name="public-and-private-ip-address-support"></a>Prise en charge d’adresses IP publique et privée

Application Gateway v2 ne prend actuellement pas en charge le mode IP privé. Elle prend en charge les combinaisons suivantes :

* Adresse IP privée et adresse IP publique
* Adresse IP publique uniquement

Pour plus d'informations, consultez [Forum Aux Questions sur le App Gateway](application-gateway-faq.md#how-do-i-use-application-gateway-v2-with-only-private-frontend-ip-address).


Une adresse IP publique n’est pas nécessaire pour un point de terminaison interne non exposé à Internet. Ce dernier est appelé point de terminaison d’ *équilibreur de charge interne* ou adresse IP front-end privée. L’équilibreur de charge interne de la passerelle d’application s’avère utile pour les applications métier internes non exposées à Internet. Il s’avère également utile pour les services et niveaux inclus dans une application multiniveau qui se trouve dans une limite de sécurité non exposée à Internet, mais qui a besoin d’une distribution de charge par tourniquet, de l’adhérence de session ou de la terminaison TLS.

Une seule adresse IP publique ou une seule adresse IP privée est prise en charge. Vous choisissez l’adresse IP front-end quand vous créez la passerelle d’application.

- Concernant l’adresse IP publique, vous pouvez en créer une ou en utiliser une existante au même emplacement que la passerelle d’application. Pour plus d’informations, consultez [Adresse IP statique ou dynamique](./application-gateway-components.md#static-versus-dynamic-public-ip-address).

- Concernant l’adresse IP privée, vous pouvez spécifier une adresse IP privée du sous-réseau dans lequel la passerelle d’application est créée. Si vous n’en spécifiez aucune, une adresse IP arbitraire est automatiquement sélectionnée dans le sous-réseau. Le type d’adresse IP que vous sélectionnez (statique ou dynamique) n’est pas modifiable par la suite. Pour plus d’informations, consultez [Créer une passerelle d’application avec un équilibreur de charge interne](./application-gateway-ilb-arm.md).

Une adresse IP front-end est associée à un *écouteur* , qui vérifie les demandes entrantes sur l’adresse IP front-end.

## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus sur la configuration de l’écouteur](configuration-listeners.md)