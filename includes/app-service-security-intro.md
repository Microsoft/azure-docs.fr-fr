---
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 04/15/2020
ms.author: cephalin
ms.openlocfilehash: 0fccb1ed092190f88d197c9bf2cd17678a4d1731
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83649050"
---
Les composants de plateforme d’App Service, notamment les machines virtuelles Azure, le stockage, les connexions réseau, les frameworks web, les fonctionnalités de gestion et d’intégration, sont activement sécurisés et renforcés. App Service fait régulièrement l’objet de vérifications de conformité strictes pour garantir les points suivants :

- Les ressources de votre application sont [protégées](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox) des ressources Azure des autres clients.
- Les [instances de machine virtuelle et les logiciels de runtime sont régulièrement mis à jour](../articles/app-service/overview-patch-os-runtime.md) pour que soient traitées les vulnérabilités récemment découvertes. 
- La communication de secrets (tels que des chaînes de connexion) entre votre application et d’autres ressources Azure (telles que [SQL Database](https://azure.microsoft.com/services/sql-database/)) reste dans Azure et ne franchit pas les limites du réseau. Les secrets sont toujours chiffrés quand ils sont stockés.
- Toutes les communications via les fonctionnalités de connectivité d’App Service, telles que la [connexion hybride](../articles/app-service/app-service-hybrid-connections.md), sont chiffrées. 
- Toutes les connexions avec les outils de gestion à distance, tels qu’Azure PowerShell, Azure CLI, les SDK Azure et les API REST, sont chiffrées.
- La gestion continue des menaces protège l’infrastructure et la plateforme contre les programmes malveillants, le déni de service distribué (DDoS), les attaques de l’intercepteur (man-in-the-middle, MITM) et bien d’autres menaces.

Pour plus d’informations sur la sécurité de l’infrastructure et de la plateforme dans Azure, consultez [Centre de confidentialité Azure](https://azure.microsoft.com/overview/trusted-cloud/).