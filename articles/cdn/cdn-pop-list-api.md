---
title: Récupérer la liste d’adresses IP POP actuelle pour Azure CDN | Microsoft Docs
description: Découvrez comment obtenir des serveurs POP à l’aide de l’API REST. Les serveurs POP envoient des requêtes aux serveurs d’origine associés aux points de terminaison Azure Content Delivery Network.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/22/2019
ms.author: allensu
ms.custom: ''
ms.openlocfilehash: 43b4bcaee447d84efa088e84340ccfc717fe2777
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005178"
---
# <a name="retrieve-the-current-pop-ip-list-for-azure-cdn"></a>Récupérer la liste d’adresses IP POP actuelle pour Azure CDN

## <a name="retrieve-the-current-verizon-pop-ip-list-for-azure-cdn"></a>Récupérer la liste d’adresses IP POP Verizon actuelle pour Azure CDN

Vous pouvez utiliser l’API REST pour récupérer l’ensemble des adresses IP pour les serveurs POP (point de présence) de Verizon. Ces serveurs POP effectuent des requêtes aux serveurs d’origine qui sont associés à des points de terminaison Azure Content Delivery Network (CDN) sur un profil Verizon (**Azure CDN Standard de Verizon** ou **Azure CDN Premium de Verizon**). Notez que cet ensemble d’adresses IP est différent des adresses IP qu’un client voit lorsqu’ils effectue des requêtes auprès des serveurs POP. 

Pour connaître la syntaxe de l’opération d’API REST pour la récupération de la liste POP, consultez [Nœuds de périphérie - Liste](/rest/api/cdn/edgenodes/list).

## <a name="retrieve-the-current-microsoft-pop-ip-list-for-azure-cdn"></a>Récupérer la liste d’adresses IP POP Microsoft actuelle pour Azure CDN

Pour verrouiller votre application de sorte qu’elle n’accepte que le trafic provenant de Microsoft Azure CDN, vous devez configurer les ACL IP pour votre back-end. Vous pouvez également restreindre l’ensemble des valeurs acceptées pour l’en-tête « X-Forwarded-Host » envoyé par Microsoft Azure CDN. Voici le détail de ces étapes :

Configurez les ACL IP pour vos back-ends de manière à accepter uniquement le trafic en provenance de l’espace d’adressage IP de Microsoft Azure CDN et des services d’infrastructure d’Azure. 

* Espace d’adressage IP du back-end IPv4 de Microsoft Azure CDN : 147.243.0.0/16
* Espace d’adressage IP du back-end IPv6 de Microsoft Azure CDN : 2a01:111:2050::/44

Pour utiliser des étiquettes de service avec Azure CDN de Microsoft, utilisez l’étiquette Azure Front Door. Vous trouverez les plages d’adresses IP et les étiquettes de service pour les services Microsoft [ici](https://www.microsoft.com/download/details.aspx?id=56519).


## <a name="typical-use-case"></a>Cas d’utilisation classique

Pour des raisons de sécurité, vous pouvez utiliser cette liste IP afin de faire en sorte que les requêtes à votre serveur d’origine sont effectuées uniquement à partir d’un serveur POP Verizon valide. Par exemple, si un utilisateur a découvert le nom d’hôte ou l’adresse IP du serveur d’origine d’un point de terminaison CDN, il peut effectuer des requêtes directement auprès du serveur d’origine, ignorant ainsi les fonctionnalités de mise à l’échelle et de sécurité fournies par Azure CDN. En définissant les adresses IP dans la liste retournée comme les seules adresses IP autorisées sur un serveur d’origine, ce scénario peut être évité. Pour vous assurer que vous avez la dernière liste POP, récupérez-la au moins une fois par jour. 

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’API REST, consultez [API REST Azure CDN](/rest/api/cdn/).