---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/23/2019
ms.author: glenga
ms.openlocfilehash: 8bdd8b9d900cc50fdeb34ff7d233ac4d7e17a45c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78191023"
---
Mettez à jour *HttpExample\\\_\_init\_\_.py* pour correspondre au code suivant, en ajoutant le paramètre `msg` à la définition de fonction et `msg.set(name)` sous l’instruction `if name:`.

:::code language="python" source="~/functions-docs-python/functions-add-output-binding-storage-queue-cli/HttpExample/__init__.py":::

Le paramètre `msg` est une instance de la [`azure.functions.InputStream class`](/python/api/azure-functions/azure.functions.httprequest). Sa méthode `set` écrit un message de type chaîne dans la file d’attente. Dans le cas présent, il s’agit du nom passé à la fonction dans la chaîne de requête de l’URL.
