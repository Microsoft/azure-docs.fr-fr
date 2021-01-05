---
title: Autorité de certification autorisée pour le protocole HTTPS personnalisé sur Azure CDN
description: Si vous utilisez votre propre certificat pour activer HTTPS sur un domaine personnalisé, vous devrez utiliser une autorité de certification autorisée (CA) pour le créer.
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
ms.topic: troubleshooting
ms.date: 10/18/2018
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 29b6cb25e021e86ce6663b4db5c89217aaf70a37
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84887397"
---
# <a name="allowed-certificate-authorities-for-enabling-custom-https-on-azure-cdn"></a>Autorités de certification autorisées pour le protocole HTTPS personnalisé sur Azure CDN

Lorsque vous [activez la fonctionnalité HTTPS à l’aide de votre propre certificat](cdn-custom-ssl.md?tabs=option-2-enable-https-with-your-own-certificate#tlsssl-certificates) pour un domaine personnalisé Azure Content Delivery Network (CDN), vous devez répondre à des exigences de certificat spécifiques. Le profil **Azure CDN Standard de Microsoft** requiert un certificat de l’une des autorités de certification approuvées répertoriées dans la liste suivante. En cas d’utilisation d’un certificat d’une autorité de certification non approuvée ou d’un certificat auto-signé, la demande est rejetée. Les profils **Azure CDN Standard de Verizon** et **Azure CDN Premium de Verizon** acceptent tout certificat valide de toute autorité de certification.

> [!NOTE]
> L’option permettant d’utiliser votre propre certificat pour activer la fonctionnalité HTTPS de domaine personnalisé n’est *pas* disponible pour des profils **Azure CDN Standard d’Akamai**. 
>

[!INCLUDE [cdn-front-door-allowed-ca](../../includes/cdn-front-door-allowed-ca.md)]

