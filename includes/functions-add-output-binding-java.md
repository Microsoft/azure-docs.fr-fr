---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/18/2020
ms.author: glenga
ms.openlocfilehash: c27f3fea6d2dd8b891220ba06b375b33e6cd950b
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "80673488"
---
Dans un projet Java, les liaisons sont définies comme annotations de liaison sur la méthode de fonction. Le fichier *function.json* est ensuite généré automatiquement en fonction de ces annotations.

Accédez à l’emplacement de votre code de fonction sous _src/main/java_, ouvrez le fichier projet *Function.java*, puis ajoutez le paramètre suivant à la définition de la méthode `run` :

:::code language="java" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/src/main/java/com/function/Function.java" range="20-21":::

Le paramètre `msg` est un type [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.outputbinding), qui représente une collection de chaînes écrites en tant que messages à une liaison de sortie quand la fonction se termine. Dans ce cas, la sortie est une file d’attente de stockage nommée `outqueue`. La chaîne de connexion pour le compte de stockage est définie par la méthode `connection`. Au lieu de passer la chaîne de connexion proprement dite, vous passez le paramètre d’application qui contient la chaîne de connexion du compte de stockage.

La définition de la méthode `run` doit maintenant ressembler à l’exemple suivant :  

:::code language="java" source="~/functions-quickstart-java/functions-add-output-binding-storage-queue/src/main/java/com/function/Function.java" range="16-22":::