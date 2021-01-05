---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 2b2c043e70aac14c7fc6f0b58aae257624b05d13
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80673312"
---
Dans un projet Java, les liaisons sont définies comme annotations de liaison sur la méthode de fonction. Le fichier *function.json* est ensuite généré automatiquement en fonction de ces annotations.

Accédez à l’emplacement de votre code de fonction sous _src/main/java_, ouvrez le fichier projet *Function.java*, puis ajoutez le paramètre suivant à la définition de la méthode `run` :

```java
@QueueOutput(name = "msg", queueName = "outqueue", connection = "AzureWebJobsStorage") OutputBinding<String> msg
```

Le paramètre `msg` est un type [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.outputbinding), qui représente une collection de chaînes écrites en tant que messages à une liaison de sortie quand la fonction se termine. Dans ce cas, la sortie est une file d’attente de stockage nommée `outqueue`. La chaîne de connexion pour le compte de stockage est définie par la méthode `connection`. Au lieu de passer la chaîne de connexion proprement dite, vous passez le paramètre d’application qui contient la chaîne de connexion du compte de stockage.

La définition de la méthode `run` doit maintenant ressembler à l’exemple suivant :  

```java
@FunctionName("HttpTrigger-Java")
public HttpResponseMessage run(
        @HttpTrigger(name = "req", methods = {HttpMethod.GET, HttpMethod.POST}, authLevel = AuthorizationLevel.FUNCTION)  
        HttpRequestMessage<Optional<String>> request, 
        @QueueOutput(name = "msg", queueName = "outqueue", connection = "AzureWebJobsStorage") 
        OutputBinding<String> msg, final ExecutionContext context) {
    ...
}
```