---
author: cephalin
ms.service: app-service-web
ms.topic: include
ms.date: 11/03/2016
ms.author: cephalin
ms.openlocfilehash: 733fc9620cbd17e5e4d8bb101c54ff646a06d6a4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86050373"
---
### <a name="app-service-plan"></a>Plan App Service
Crée le plan de service pour l'hébergement de l'application web. Vous fournissez le nom du plan à l'aide du paramètre **hostingPlanName** . L’emplacement du plan est identique à celui utilisé pour le groupe de ressources. Le niveau tarifaire et la taille de worker sont spécifiés dans les paramètres **sku** et **workerSize**.

```config
{
  "apiVersion": "2015-08-01",
  "name": "[parameters('hostingPlanName')]",
  "type": "Microsoft.Web/serverfarms",
  "location": "[resourceGroup().location]",
  "sku": {
    "name": "[parameters('sku')]",
    "capacity": "[parameters('workerSize')]"
  },
  "properties": {
    "name": "[parameters('hostingPlanName')]"
  }
},
```
