---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/18/2020
ms.author: glenga
ms.openlocfilehash: 179ae760f146a5ac3041a54065ae12147f3f9bf0
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97739810"
---
Étant donné que l’archétype crée également un ensemble de tests, vous devez mettre à jour ces tests pour gérer le nouveau paramètre `msg` dans la signature de méthode `run`.  

Accédez à l’emplacement de votre code de test sous _src/test/Java_, ouvrez le fichier projet *Function.java* et remplacez la ligne de code sous `//Invoke` par le code suivant.

:::code language="java" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/src/test/java/com/function/FunctionTest.java" range="48-50":::
