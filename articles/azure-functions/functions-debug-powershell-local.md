---
title: Déboguer localement des fonctions PowerShell Azure
description: Découvrez comment déboguer des fonctions PowerShell en cas d’exécution locale.
author: tylerleonhardt
ms.topic: conceptual
ms.date: 04/22/2019
ms.author: tyleonha
ms.reviewer: glenga
ms.openlocfilehash: a668024db126c82f96756555aba513b77f7d7366
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422958"
---
# <a name="debug-powershell-azure-functions-locally"></a>Déboguer localement des fonctions PowerShell Azure

Azure Functions vous permet de développer vos fonctions en tant que scripts PowerShell.

Vous pouvez déboguer localement vos fonctions PowerShell comme vous le feriez pour tout script PowerShell, à l’aide des outils de développement standard suivants :

* [Visual Studio Code](https://code.visualstudio.com/) : Éditeur de texte gratuit, léger et open source de Microsoft avec l’extension PowerShell qui offre une expérience de développement PowerShell complète.
* Console PowerShell : Déboguez en utilisant les mêmes commandes que vous utiliseriez pour déboguer tout autre processus PowerShell.

[Azure Functions Core Tools](functions-run-local.md) prend en charge le débogage local d’Azure Functions, dont les fonctions PowerShell.

## <a name="example-function-app"></a>Exemple d’application de fonction

L’application de fonction utilisée dans cet article possède une seule fonction déclenchée via HTTP et possède les fichiers suivants :

```
PSFunctionApp
 | - HttpTriggerFunction
 | | - run.ps1
 | | - function.json
 | - local.settings.json
 | - host.json
 | - profile.ps1
```

Cette application de fonction est semblable à celle que vous obtenez lorsque vous effectuez les procédures de [démarrage rapide PowerShell](./create-first-function-vs-code-powershell.md).

Le code de fonction dans `run.ps1` ressemble au script suivant :

```powershell
param($Request)

$name = $Request.Query.Name

if($name) {
    $status = 200
    $body = "Hello $name"
}
else {
    $status = 400
    $body = "Please pass a name on the query string or in the request body."
}

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = $status
    Body = $body
})
```

## <a name="set-the-attach-point"></a>Définir le point d’attache

Pour déboguer une fonction PowerShell quelconque, la fonction doit s’arrêter pour que le débogueur soit attaché. L’applet de commande `Wait-Debugger` arrête son exécution et attend le débogueur.

>[!NOTE]
>Lorsque vous utilisez PowerShell 7, vous n’avez pas besoin d’ajouter l’appel `Wait-Debugger` à votre code.

Il vous suffit d’ajouter un appel à l’applet de commande `Wait-Debugger` juste au-dessus de l’instruction `if`, comme suit :

```powershell
param($Request)

$name = $Request.Query.Name

# This is where we will wait for the debugger to attach
Wait-Debugger

if($name) {
    $status = 200
    $body = "Hello $name"
}
# ...
```

Le débogage commence à l’instruction `if`. 

Avec `Wait-Debugger` en place, vous pouvez maintenant déboguer les fonctions à l’aide de Visual Studio Code ou d’une console PowerShell.

## <a name="debug-in-visual-studio-code"></a>Déboguer dans Visual Studio Code

Pour déboguer vos fonctions PowerShell dans Visual Studio Code, les éléments suivants doivent être installés :

* [Extension PowerShell pour Visual Studio Code](/powershell/scripting/components/vscode/using-vscode)
* [Extension Azure Functions pour Visual Studio Code](./create-first-function-cli-powershell.md)
* [PowerShell Core 6.2 ou version ultérieure](/powershell/scripting/install/installing-powershell-core-on-windows)

Après avoir installé ces dépendances, chargez un projet de fonctions PowerShell existant ou [créez votre premier projet de fonctions PowerShell](./create-first-function-vs-code-powershell.md).

>[!NOTE]
> Si votre projet n’a pas les fichiers de configuration nécessaires, vous êtes invité à les ajouter.

### <a name="set-the-powershell-version"></a>Définir la version de PowerShell

PowerShell Core s’installe côte à côte avec Windows PowerShell. Définissez PowerShell Core en tant que version de PowerShell à utiliser avec l’extension PowerShell pour Visual Studio Code.

1. Appuyez sur F1 pour afficher la palette de commandes, puis recherchez `Session`.

1. Choisissez **PowerShell : Afficher le menu de session**.

1. Si votre **session active** n’est pas **PowerShell Core 6** , choisissez **Basculer vers : PowerShell Core 6**.

Lorsqu’un fichier PowerShell est ouvert, vous voyez la version affichée en vert en bas à droite de la fenêtre. La sélection de ce texte affiche également le menu de la session. Pour plus d’informations, consultez [Choisir la version de PowerShell à utiliser avec l’extension](/powershell/scripting/components/vscode/using-vscode#choosing-a-version-of-powershell-to-use-with-the-extension).

### <a name="start-the-function-app"></a>Démarrer l’application de fonction

Vérifiez que `Wait-Debugger` est défini dans la fonction où vous souhaitez attacher le débogueur.  Avec `Wait-Debugger` ajouté, vous pouvez déboguer votre application de fonction à l’aide de Visual Studio Code.

Choisissez le volet **Déboguer** , puis **Attacher à la fonction PowerShell**.

![débogueur](https://user-images.githubusercontent.com/2644648/56166073-8a7b3780-5f89-11e9-85ce-36ed38e221a2.png)

Vous pouvez également appuyer sur la touche F5 pour démarrer le débogage.

L’opération Démarrer le débogage effectue les tâches suivantes :

* Exécute `func extensions install` dans le terminal pour installer les extensions Azure Functions requises par votre application de fonction.
* Exécute `func host start` dans le terminal pour démarrer l’application de fonction dans l’hôte Functions.
* Attache le débogueur PowerShell à l’instance d’exécution de PowerShell dans le runtime Functions.

>[!NOTE]
> Vous devez vous assurer que PSWorkerInProcConcurrencyUpperBound a la valeur 1 pour garantir une expérience de débogage correcte dans Visual Studio Code. Il s’agit de la valeur par défaut.

Lorsque votre application de fonction est en cours d’exécution, vous avez besoin d’une console PowerShell distincte pour appeler la fonction déclenchée via HTTP.

Dans ce cas, la console PowerShell est le client. `Invoke-RestMethod` permet de déclencher la fonction.

Dans une console PowerShell, exécutez la commande suivante :

```powershell
Invoke-RestMethod "http://localhost:7071/api/HttpTrigger?Name=Functions"
```

Vous remarquerez qu’une réponse n’est pas immédiatement retournée. Cela est dû au fait que `Wait-Debugger` a attaché le débogueur et que l’exécution PowerShell est passée en mode Arrêt dès qu’elle a pu. Cela est dû au [concept BreakAll](#breakall-might-cause-your-debugger-to-break-in-an-unexpected-place), qui est expliqué plus loin. Une fois que vous appuyez sur le bouton `continue`, le débogueur s’arrête désormais sur la ligne située juste après `Wait-Debugger`.

À ce stade, le débogueur est attaché et vous pouvez effectuer toutes les opérations normales de débogage. Pour plus d’informations sur l’utilisation du débogueur dans Visual Studio Code, consultez [la documentation officielle](https://code.visualstudio.com/Docs/editor/debugging#_debug-actions).

Lorsque vous continuez et appelez entièrement votre script, vous remarquez que :

* La console PowerShell qui a exécuté `Invoke-RestMethod` a retourné un résultat
* La console intégrée PowerShell dans Visual Studio Code est en attente d’un script à exécuter

Plus tard, lorsque vous appelez la même fonction, le débogueur dans l’extension PowerShell s’arrête juste après `Wait-Debugger`.

## <a name="debugging-in-a-powershell-console"></a>Débogage dans une console PowerShell

>[!NOTE]
> Cette section suppose que vous avez lu les [documents d’Azure Functions Core Tools](functions-run-local.md) et que vous savez comment utiliser la commande `func host start` pour démarrer votre application de fonction.

Ouvrez une console, accédez via `cd` dans le répertoire de votre application de fonction, puis exécutez la commande suivante :

```sh
func host start
```

Avec l’application de fonction en cours d’exécution et `Wait-Debugger` en place, vous pouvez attacher au processus. Vous avez besoin de deux consoles PowerShell supplémentaires.

L’une de ces consoles agit comme client. À partir de cela, vous appelez `Invoke-RestMethod` pour déclencher la fonction. Par exemple, vous pouvez exécuter la commande suivante :

```powershell
Invoke-RestMethod "http://localhost:7071/api/HttpTrigger?Name=Functions"
```

Vous remarquerez qu’elle ne retourne pas de réponse, ce qui est le résultat de `Wait-Debugger`. L’instance d’exécution PowerShell attend maintenant un débogueur à attacher. Nous allons l’attacher.

Dans l’autre console PowerShell, exécutez la commande suivante :

```powershell
Get-PSHostProcessInfo
```

Cette applet de commande retourne une table qui ressemble à la sortie suivante :

```output
ProcessName ProcessId AppDomainName
----------- --------- -------------
dotnet          49988 None
pwsh            43796 None
pwsh            49970 None
pwsh             3533 None
pwsh            79544 None
pwsh            34881 None
pwsh            32071 None
pwsh            88785 None
```

Notez le `ProcessId` de l’élément dans la table avec `ProcessName` comme `dotnet`. Ce processus est votre application de fonction.

Ensuite, exécutez l'extrait de code suivant :

```powershell
# This enters into the Azure Functions PowerShell process.
# Put your value of `ProcessId` here.
Enter-PSHostProcess -Id $ProcessId

# This triggers the debugger.
Debug-Runspace 1
```

Une fois démarré, le débogueur s’arrête et affiche quelque chose de similaire à la sortie suivante :

```
Debugging Runspace: Runspace1

To end the debugging session type the 'Detach' command at the debugger prompt, or type 'Ctrl+C' otherwise.

At /Path/To/PSFunctionApp/HttpTriggerFunction/run.ps1:13 char:1
+ if($name) { ...
+ ~~~~~~~~~~~
[DBG]: [Process:49988]: [Runspace1]: PS /Path/To/PSFunctionApp>>
```

À ce stade, vous êtes arrêté à un point d’arrêt dans le [débogueur PowerShell](/powershell/module/microsoft.powershell.core/about/about_debuggers). À partir de là, vous pouvez effectuer toutes les opérations de débogage habituelles, un pas à pas principal, un pas à pas détaillé, continuer, quitter, etc. Pour afficher l’ensemble complet des commandes de débogage disponibles dans la console, exécutez la commande `h` ou `?`.

Vous pouvez également définir des points d’arrêt à ce niveau avec l’applet de commande `Set-PSBreakpoint`.

Lorsque vous continuez et appelez entièrement votre script, vous remarquez que :

* La console PowerShell où vous avez exécuté `Invoke-RestMethod` a maintenant retourné un résultat.
* La console PowerShell où vous avez exécuté `Debug-Runspace` est en attente d’un script à exécuter.

Vous pouvez appeler à nouveau la même fonction (à l’aide de `Invoke-RestMethod` par exemple) et le débogueur s’arrête juste après la commande `Wait-Debugger`.

## <a name="considerations-for-debugging"></a>Considérations relatives au débogage

Gardez à l’esprit les problèmes suivants lors du débogage de votre code Functions.

### <a name="breakall-might-cause-your-debugger-to-break-in-an-unexpected-place"></a>`BreakAll` peut entraîner l’arrêt de votre débogueur dans un emplacement inattendu

L’extension PowerShell utilise `Debug-Runspace`, qui à son tour dépend de la fonctionnalité `BreakAll` de PowerShell. Cette fonctionnalité indique à PowerShell de s’arrêter à la première commande qui est exécutée. Ce comportement vous donne la possibilité de définir des points d’arrêt au sein de l’instance d’exécution déboguée.

Le runtime Azure Functions exécute quelques commandes avant d’appeler véritablement votre script `run.ps1`. Il est donc possible que le débogueur finisse par s’arrêter au sein de `Microsoft.Azure.Functions.PowerShellWorker.psm1` ou de `Microsoft.Azure.Functions.PowerShellWorker.psd1`.

Si cet arrêt doit se produire, exécutez la commande `continue` ou `c` pour ignorer ce point d’arrêt. Vous vous arrêtez ensuite au point d’arrêt attendu.

## <a name="troubleshooting"></a>Dépannage

Si vous rencontrez des difficultés pendant le débogage, vous devez vérifier les éléments suivants :

| Vérification | Action |
|------|------|
| Exécutez `func --version` à partir du terminal. Si vous recevez une erreur indiquant que `func` est introuvable, il se peut que Core Tools (func.exe) soit absent de la variable `path` locale.| [Réinstallez Core Tools](functions-run-local.md#v2).|  
| Dans Visual Studio Code, le terminal par défaut doit avoir accès à func.exe. Vérifiez que vous n’utilisez pas un terminal par défaut sur lequel Core Tools n’est pas installé, tel que le sous-système Windows pour Linux (WSL).  | Définissez PowerShell 7 (recommandé) ou Windows PowerShell 5.1 comme interpréteur de commandes par défaut dans Visual Studio Code.|
  

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le développement de Functions à l’aide de PowerShell, consultez [Guide des développeurs PowerShell sur Azure Functions](functions-reference-powershell.md).
