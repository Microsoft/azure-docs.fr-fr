---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 7ef3bd0f401ba54d56ed42df34cd2e761681dbc7
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96904065"
---
::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"

## <a name="add-an-output-binding-definition-to-the-function"></a>Ajouter une définition de liaison de sortie à la fonction

Bien qu’une fonction ne puisse avoir qu’un seul déclencheur, elle peut avoir plusieurs liaisons d’entrée et de sortie. Ainsi, vous pouvez vous connecter à d’autres services et ressources Azure sans avoir à écrire du code d’intégration personnalisé. 
::: zone-end
::: zone pivot="programming-language-python,programming-language-javascript,programming-language-powershell,programming-language-typescript"  
Vous déclarez ces liaisons dans le fichier *function.json* situé dans le dossier de la fonction. Compte tenu du guide de démarrage rapide précédent, votre fichier *function.json* situé dans le dossier *HttpExample* contient deux liaisons dans la collection `bindings` :  
::: zone-end

::: zone pivot="programming-language-javascript,programming-language-typescript"  
:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-JavaScript/function.json" range="2-18":::  
::: zone-end

::: zone pivot="programming-language-python"  
:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/function.json" range="2-18":::  
::: zone-end

::: zone pivot="programming-language-powershell"  
:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-PowerShell/function.json" range="2-18":::
::: zone-end  

::: zone pivot="programming-language-python,programming-language-javascript, programming-language-powershell, programming-language-typescript"  
Chaque liaison a au moins un type, une direction et un nom. Dans l’exemple ci-dessus, la première liaison est de type `httpTrigger`, et sa direction est `in`. Pour la direction `in`, `name` spécifie le nom d’un paramètre d’entrée qui est envoyé à la fonction quand il est appelé par le déclencheur.  
::: zone-end

::: zone pivot="programming-language-javascript,programming-language-typescript"  
La deuxième liaison dans la collection est nommée `res`. Cette liaison `http` est une liaison de sortie (`out`) qui est utilisée pour écrire la réponse HTTP. 

Pour écrire dans une file d’attente Stockage Azure à partir de cette fonction, ajoutez une liaison `out` de type `queue` nommée `msg`, comme indiqué dans le code ci-dessous :

:::code language="json" source="~/functions-docs-javascript/functions-add-output-binding-storage-queue-cli/HttpExample/function.json" range="3-26":::
::: zone-end  

::: zone pivot="programming-language-python"  
La deuxième liaison dans la collection est de type `http`, et sa direction est `out`. Dans ce cas, le `name` spécial de `$return` indique que cette liaison utilise la valeur retournée par la fonction au lieu de fournir un paramètre d’entrée.

Pour écrire dans une file d’attente Stockage Azure à partir de cette fonction, ajoutez une liaison `out` de type `queue` nommée `msg`, comme indiqué dans le code ci-dessous :

:::code language="json" source="~/functions-docs-python/functions-add-output-binding-storage-queue-cli/HttpExample/function.json" range="3-26":::
::: zone-end  

::: zone pivot="programming-language-powershell"  
La deuxième liaison dans la collection est nommée `res`. Cette liaison `http` est une liaison de sortie (`out`) qui est utilisée pour écrire la réponse HTTP. 

Pour écrire dans une file d’attente Stockage Azure à partir de cette fonction, ajoutez une liaison `out` de type `queue` nommée `msg`, comme indiqué dans le code ci-dessous :

:::code language="json" source="~/functions-docs-powershell/functions-add-output-binding-storage-queue-cli/HttpExample/function.json" range="3-26":::
::: zone-end  

::: zone pivot="programming-language-python,programming-language-javascript,programming-language-powershell,programming-language-typescript"  
Dans ce cas, `msg` est fourni à la fonction en tant qu’argument de sortie. Pour un type `queue`, vous devez également spécifier le nom de la file d’attente dans `queueName` et fournir le *nom* de la connexion Stockage Azure (à partir de *local.settings.json*) dans `connection`. 
::: zone-end  
