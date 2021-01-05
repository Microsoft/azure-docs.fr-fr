---
title: Autorité de certification autorisée pour le protocole HTTPS personnalisé sur Azure Front Door
description: Si vous utilisez votre propre certificat pour activer HTTPS sur un domaine personnalisé Azure Front Door, vous devrez utiliser une autorité de certification autorisée (CA) pour le créer.
services: frontdoor
documentationcenter: ''
author: duongau
ms.assetid: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/30/2020
ms.author: duau
ms.openlocfilehash: 20c5d611272ee2159ce8ddcc2865797a225a7ebb
ms.sourcegitcommit: 06ba80dae4f4be9fdf86eb02b7bc71927d5671d3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/01/2020
ms.locfileid: "91613677"
---
# <a name="allowed-certificate-authorities-for-enabling-custom-https-on-azure-front-door"></a>Autorités de certification autorisées pour l’activation du protocole HTTPS personnalisé sur Azure Front Door

Lorsque vous [activez la fonctionnalité HTTPS à l’aide de votre propre certificat](front-door-custom-domain-https.md?tabs=option-2-enable-https-with-your-own-certificate) pour un domaine personnalisé Azure Front Door. Vous avez besoin d’une autorité de certification autorisée pour créer votre certificat TLS/SSL. Si vous utilisez une autorité de certification non autorisée ou un certificat auto-signé, votre demande sera rejetée.

[!INCLUDE [cdn-front-door-allowed-ca](../../includes/cdn-front-door-allowed-ca.md)]
