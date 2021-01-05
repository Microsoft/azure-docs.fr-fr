---
title: Présentation du pare-feu d’applications web Azure
description: Cet article fournit une vue d’ensemble du pare-feu d’applications web (WAF) Azure
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 03/18/2020
ms.author: victorh
ms.topic: overview
ms.openlocfilehash: 49024c86d09f5cdd9e8b04d5a49f60021660b0c4
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "79475785"
---
# <a name="what-is-azure-web-application-firewall"></a>Qu’est-ce que le pare-feu d’applications web Azure ?

Le pare-feu d’applications web (WAF) permet de centraliser la protection de vos applications web contre les vulnérabilités et les attaques courantes. Les applications web sont de plus en plus visées par des attaques malveillantes qui exploitent des vulnérabilités connues. L’injection de code SQL et l’exécution de scripts de site à site font partie des attaques les plus courantes.

![Vue d’ensemble de WAF](media/overview/wafoverview.png)

Il est difficile d’empêcher de telles attaques dans le code d’application. Cela peut nécessiter une maintenance rigoureuse, une mise à jour corrective et la surveillance au niveau de plusieurs couches de la topologie de l’application. Un pare-feu d’applications web centralisé permet de simplifier considérablement la gestion de la sécurité. Un pare-feu WAF offre également aux administrateurs d’application une meilleure garantie de protection contre les menaces et intrusions .

Une solution WAF peut également réagir plus rapidement à une menace de sécurité en corrigeant une vulnérabilité connue de façon centralisée plutôt que de sécuriser individuellement chacune des applications web.

## <a name="supported-service"></a>Service pris en charge

WAF peut être déployé avec Azure Application Gateway, Azure Front Door et le service Azure Content Delivery Network (CDN) de Microsoft. WAF sur Azure CDN est actuellement en préversion publique.  WAF comprend des fonctionnalités qui sont personnalisées pour chaque service spécifique. Pour plus d’informations sur les fonctionnalités de WAF pour chaque service, consultez la vue d’ensemble de chaque service.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur le pare-feu d’applications Web sur Application Gateway, consultez [Pare-feu d’applications web sur Application Gateway](./ag/ag-overview.md).
- Pour plus d’informations sur le pare-feu d’applications web sur Azure Front Door Service, consultez [Pare-feu d’applications web sur Azure Front Door Service](./afds/afds-overview.md).
- Pour plus d’informations sur le pare-feu d’applications web sur le service Azure CDN, consultez [Pare-feu d’applications web sur le service Azure CDN](./cdn/cdn-overview.md)
