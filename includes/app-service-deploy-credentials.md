---
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 04/20/2020
ms.author: cephalin
ms.openlocfilehash: c3fa57dd162fbbfbf0d46f73bffc78f279ef2968
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83649133"
---
* **Informations d’identification au niveau de l’utilisateur** : un seul ensemble d’informations d’identification pour l’intégralité du compte Azure. Il peut être utilisé pour déployer sur App Service pour n’importe quelle application et dans n’importe quel abonnement auxquels le compte Azure est autorisé à accéder. C’est l’ensemble par défaut qui est présenté dans l’interface utilisateur graphique du portail, comme la **vue d’ensemble** et les **propriétés** de la [page Ressources](../articles/azure-resource-manager/management/manage-resources-portal.md#manage-resources) de l’application. Lorsqu'un utilisateur est autorisé à accéder à l’application via le contrôle d’accès en fonction du rôle (RBAC) ou des autorisations de coadmin, il peut se servir de ses propres informations d’identification tant que l’accès n’est pas révoqué. Ne partagez pas ces informations d’identification avec d’autres utilisateurs d’Azure.

* **Informations d’identification au niveau de l’application** : un seul ensemble d’informations d’identification pour chaque application. Celui-ci peut être utilisé pour déployer sur cette application uniquement. Les informations d’identification de chaque application sont générées automatiquement à la création de l’application. Elles ne peuvent pas être configurées manuellement, mais peuvent être réinitialisées à tout moment. Pour qu’un utilisateur puisse accéder aux informations d’identification de niveau application via RBAC, il doit avoir un rôle de contributeur ou supérieur sur l’application (y compris le rôle intégré de contributeur de site web). Les lecteurs ne sont pas autorisés à publier et n’ont pas accès à ces informations d’identification.