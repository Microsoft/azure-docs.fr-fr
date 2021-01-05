---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 07/05/2019
ms.author: glenga
ms.openlocfilehash: 5e1a2622df0038141dd5cb05237f93d5e33e0bfb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78190911"
---
Dans un projet de bibliothèque de classes C#, les liaisons sont définies comme attributs de liaison sur la méthode de fonction. Le fichier *function.json* nécessaire à Functions est ensuite généré automatiquement en fonction de ces attributs.

Ouvrez le fichier projet *HttpExample.cs*, puis ajoutez le paramètre suivant à la définition de la méthode `Run` :

:::code language="csharp" source="~/functions-docs-csharp/functions-add-output-binding-storage-queue-cli/HttpExample.cs" range="17":::

Le paramètre `msg` est un type `ICollector<T>`, qui représente une collection de messages écrits dans une liaison de sortie quand la fonction se termine. Dans ce cas, la sortie est une file d’attente de stockage nommée `outqueue`. La chaîne de connexion pour le compte de stockage est définie par `StorageAccountAttribute`. Cet attribut indique le paramètre qui contient la chaîne de connexion du compte de stockage et qui peut être appliqué au niveau de la classe, de la méthode ou du paramètre. Dans ce cas, vous pourriez omettre `StorageAccountAttribute`, car vous utilisez déjà le compte de stockage par défaut.

La définition de la méthode Run doit maintenant se présenter comme ceci :  

:::code language="csharp" source="~/functions-docs-csharp/functions-add-output-binding-storage-queue-cli/HttpExample.cs" range="14-18":::
