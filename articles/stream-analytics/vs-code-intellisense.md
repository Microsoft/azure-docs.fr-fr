---
title: IntelliSense dans Azure Stream Analytics Tools pour Visual Studio Code
description: Cet article explique comment utiliser les fonctionnalités IntelliSense disponibles dans Azure Stream Analytics Tools pour Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 4/11/2020
ms.topic: how-to
ms.openlocfilehash: 988b9d364f09a4c88bcaab1fe81ae80c8de5d4d9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96491568"
---
# <a name="intellisense-in-azure-stream-analytics-tools-for-visual-studio-code"></a>IntelliSense dans Azure Stream Analytics Tools pour Visual Studio Code

IntelliSense est disponible pour le [langage de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference?bc=https%253a%2f%2fdocs.microsoft.com%2fazure%2fbread%2ftoc.json&toc=https%253a%2f%2fdocs.microsoft.com%2fazure%2fstream-analytics%2ftoc.json) dans [Azure Stream Analytics Tools pour VS Code](https://marketplace.visualstudio.com/items?itemName=ms-bigdatatools.vscode-asa&ssr=false#overview). IntelliSense est une aide à la saisie semi-automatique de code qui comprend un certain nombre de fonctionnalités : Liste des membres, Informations sur les paramètres, Info Express et Compléter le mot. Les fonctionnalités IntelliSense sont parfois désignées sous d'autres noms tels que « saisie semi-automatique de code », « assistance au contenu » et « optimisation de code ».

![Démo d'IntelliSense](./media/vs-code-intellisense/intellisense.gif)

## <a name="intellisense-features"></a>Fonctionnalités IntelliSense

Les fonctionnalités IntelliSense de Stream Analytics Tools pour VS Code sont alimentées par un service de langage. Un service de langage analyse votre code source et fournit une saisie semi-automatique de code intelligente basée sur la sémantique du langage. Si un service de langage connaît des compléments possibles, des suggestions IntelliSense apparaissent à mesure que vous tapez. Si vous continuez à taper, une liste de membres (variables, méthodes, etc.) est filtrée pour inclure uniquement les membres qui contiennent les caractères que vous avez tapés. Lorsque vous appuyez sur les touches `Tab` ou `Enter`, IntelliSense insère le membre que vous avez sélectionné.

Vous pouvez déclencher IntelliSense dans n'importe quelle fenêtre de l'éditeur en tapant un caractère déclencheur, tel que le point `.`.

![Saisie semi-automatique IntelliSense](./media/vs-code-intellisense/auto-completion.gif)

> [!TIP]
> Le widget de suggestions prend en charge le filtrage de type Casse mixte. Vous pouvez taper les lettres qui sont en majuscules dans le nom d'une méthode pour limiter les suggestions. Par exemple, « cra » fera rapidement apparaître « createApplication ».

### <a name="types-of-completions"></a>Types de saisie semi-automatique

IntelliSense dans Stream Analytics Tools pour VS Code offre différents types de saisie semi-automatique, y compris des suggestions de serveur de langage, des extraits de code et des saisies textuelles simples à base de mots.

|Completion     |  Type       |
| ----- | ------- |
| Mots clés | `keyword`
| Fonctions | `build-in function`, `user defined function`  |
| Nom de jeu de données| `input`, `output`, `intermediate result set`|
| Nom de colonne de jeu de données|`input`, `intermediate result set`|

#### <a name="name-completion"></a>Saisie semi-automatique des noms

Outre la saisie semi-automatique des mots clés, Stream Analytics Tools pour VS Code lit la liste des noms d'entrée et de sortie des tâches, ainsi que les noms des colonnes de vos sources de données au moment de leur configuration. L'extension mémorise ces informations pour fournir des fonctionnalités de saisie semi-automatique des noms qui permettent d'entrer des instructions en limitant le nombre de frappes :

Pendant le codage, vous n'avez pas besoin de quitter l'éditeur pour effectuer des recherches sur les noms d'entrée des tâches, le nom de sortie et les noms de colonne. Tout en conservant votre contexte, vous pouvez rechercher les informations dont vous avez besoin, insérer des éléments directement dans votre code et laisser IntelliSense compléter le code à votre place.

Notez que vous devez configurer l'entrée locale ou l'entrée en direct, et enregistrer le fichier de configuration pour pouvoir utiliser la saisie semi-automatique de nom.

![Saisie semi-automatique de nom](./media/vs-code-intellisense/name-completion.gif)

### <a name="parameter-info"></a>Informations sur les paramètres

L'option **Informations sur les paramètres** d'IntelliSense ouvre une liste de paramètres qui fournit des informations sur le nombre, le nom et le type des paramètres exigés par une fonction. Le paramètre en gras indique le prochain paramètre requis lorsque vous tapez une fonction.

La liste des paramètres est également affichée pour les fonctions imbriquées. Si vous tapez une fonction comme paramètre d'une autre fonction, la liste affiche dans un premier temps les paramètres de la fonction interne. Dans un second temps, la liste affiche les paramètres de la fonction externe.

![Informations sur les paramètres](./media/vs-code-intellisense/parameter-info.gif)

### <a name="quick-info"></a>Infos express

Comme fourni par le service de langage, vous pouvez voir **Info Express** pour chaque identificateur figurant dans votre code. Voici quelques exemples d'identificateurs : entrée, sortie, jeu de résultats intermédiaire ou fonction. Lorsque vous placez le pointeur de la souris sur un identificateur, sa déclaration apparaît dans une fenêtre contextuelle. Les propriétés et les schémas de données des entrées, s'ils sont configurés, et le jeu de données intermédiaire sont présentés.

![Infos express](./media/vs-code-intellisense/quick-info.gif)

## <a name="troubleshoot-intellisense"></a>Résoudre les problèmes liés à IntelliSense

Ce problème est dû à l'absence de configuration d'entrée pour fournir des données. Vous pouvez déterminer si une [entrée locale](visual-studio-code-local-run.md#define-a-local-input) ou une [entrée en direct](visual-studio-code-local-run-live-input.md#define-a-live-stream-input) a été correctement configurée.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer une tâche Azure Stream Analytics dans Visual Studio Code](quick-create-visual-studio-code.md)
* [Tester des requêtes Stream Analytics localement avec des exemples de données à l’aide de Visual Studio Code](visual-studio-code-local-run.md)
* [Tester des requêtes Stream Analytics localement par rapport à une entrée de stream en direct à l'aide de Visual Studio Code](visual-studio-code-local-run-live-input.md)