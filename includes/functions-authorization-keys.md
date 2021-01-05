---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/06/2020
ms.author: glenga
ms.openlocfilehash: 617608f382d3331e59ae92c9eb272347c736b768
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91829084"
---
Functions vous permet d’utiliser des clés pour rendre plus difficile l’accès à vos points de terminaison de fonctions HTTP pendant le développement. Si le niveau d’accès HTTP sur une fonction déclenchée par HTTP n’est pas défini sur `anonymous`, les requêtes doivent contenir une clé d’accès API. 

Bien que les clés fournissent un mécanisme de sécurité par défaut, vous pouvez envisager d’utiliser des options supplémentaires pour sécuriser un point de terminaison HTTP en production. Par exemple, il n’est généralement pas recommandé de diffuser un secret partagé dans des applications publiques. Si votre fonction est appelée depuis un client public, vous pouvez envisager de mettre en place un autre mécanisme de sécurité. Pour plus d’informations, consultez [Sécuriser un point de terminaison HTTP en production](../articles/azure-functions/functions-bindings-http-webhook-trigger.md#secure-an-http-endpoint-in-production).

Lorsque vous renouvelez les valeurs clés de votre fonction, vous devez redistribuer manuellement les valeurs clé mises à jour à tous les clients qui appellent votre fonction.  

#### <a name="authorization-scopes-function-level"></a>Étendues d’autorisation (au niveau de la fonction)

Il existe deux étendues d’accès pour les clés au niveau de la fonction :

* **Fonction** : Ces clés s’appliquent uniquement aux fonctions spécifiques sous lesquelles elles sont définies. Utilisées en tant que clés API, elles permettent d’accéder uniquement à ces fonctions.

* **Hôte** : Des clés avec une étendue d’hôte permettent d’accéder à toutes les fonctions au sein de l’application de fonction. Utilisées en tant que clés API, elles permettent d’accéder à toute fonction au sein de la Function App. 

Chaque clé est nommée pour référence et il existe une clé par défaut (nommée « default ») au niveau fonction et hôte. Les clés de fonction prennent le pas sur les clés d’hôte. Quand deux clés portent le même nom, la clé de fonction est toujours utilisée.

#### <a name="master-key-admin-level"></a>Clé principale (au niveau de l’administrateur) 

Chaque application de fonction a également une clé d’hôte au niveau de l’administrateur nommée `_master`. En plus de fournir un accès au niveau de l’hôte à toutes les fonctions de l’application, la clé principale fournit un accès administratif aux API REST du runtime. Cette clé ne peut pas être révoquée. Quand vous définissez un niveau d’accès de `admin`, les requêtes doivent utiliser la clé principale ; toute autre clé provoque l’échec de l’accès.

> [!CAUTION]  
> En raison des autorisations élevées dans votre application de fonction accordées par la clé principale, vous ne devez pas partager celle-ci avec des tiers, ou la distribuer dans des applications clientes natives. Faites preuve de prudence lorsque vous choisissez le niveau d’accès administrateur.
