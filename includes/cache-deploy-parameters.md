---
author: wesmc7777
ms.service: redis-cache
ms.topic: include
ms.date: 04/02/2019
ms.author: wesmc
ms.openlocfilehash: 498a7ee28b9404d0733e4615f4df635a8c904b51
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67176872"
---
### <a name="cacheskuname"></a>cacheSKUName

Niveau de tarification du nouveau cache Azure pour Redis.

```json
    "cacheSKUName": {
      "type": "string",
      "allowedValues": [
        "Basic",
        "Standard",
        "Premium"
      ],
      "defaultValue": "Basic",
      "metadata": {
        "description": "The pricing tier of the new Azure Cache for Redis."
      }
    },
```

Le modèle définit les valeurs autorisées pour ce paramètre (De base, Standard ou Premium) et affecte une valeur par défaut (De base) si aucune valeur n’est spécifiée. Le niveau De base fournit un nœud unique disponible en plusieurs tailles, jusqu’à 53 Go. Le niveau Standard fournit deux nœuds primaire/réplica disponibles en plusieurs tailles jusqu’à 53 Go et un contrat SLA de 99,9 %.

### <a name="cacheskufamily"></a>cacheSKUFamily

Famille de la référence (SKU).

```json
    "cacheSKUFamily": {
      "type": "string",
      "allowedValue/s": [
        "C",
        "P"
      ],
      "defaultValue": "C",
      "metadata": {
        "description": "The family for the sku."
      }
    },
```

### <a name="cacheskucapacity"></a>cacheSKUCapacity

Taille de la nouvelle instance de cache Azure pour Redis.

Pour les familles De base et Standard :

```json
    "cacheSKUCapacity": {
      "type": "int",
      "allowedValues": [
        0,
        1,
        2,
        3,
        4,
        5,
        6
      ],
      "defaultValue": 0,
      "metadata": {
        "description": "The size of the new Azure Cache for Redis instance. "
      }
    }
```

La capacité de la mémoire cache de la valeur Premium est définie de la même manière, sauf que les valeurs autorisées vont de 1 à 5 au lieu de 0 à 6.

Le modèle définit les valeurs entières autorisées pour ce paramètre (0 à 6 pour les familles De base et Standard ; 1 à 5 pour la famille Premium). Si aucune valeur n'est spécifiée, le modèle affecte une valeur par défaut de 0 pour les valeurs De base et Standard, et de 1 pour Premium.

Les valeurs correspondent aux tailles de cache suivantes :

| Valeur | De base et Standard<br>taille du cache | Premium<br>taille du cache |
| :---: | :------------------------------: | :-------------------: |
| 0     | 250 Mo (par défaut)                 | n/a                   |
| 1     | 1 Go                             | 6 Go (par défaut)        |
| 2     | 2,5 Go                           | 13 Go                 |
| 3     | 6 Go                             | 26 Go                 |
| 4     | 13 Go                            | 53 Go                 |
| 5     | 26 Go                            | 120 Go                |
| 6     | 53 Go                            | n/a                   |
