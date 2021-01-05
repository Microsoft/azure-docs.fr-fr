---
title: Déboguer des fonctions définies par l’utilisateur dans Azure Stream Analytics
description: Cet article explique comment déboguer des fonctions définies par l’utilisateur dans Azure Stream Analytics.
author: jenssuessmeyer
ms.author: jenss
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 06/24/2020
ms.custom: devx-track-js
ms.openlocfilehash: 567def7abaff3cd8d70eb56f0aa117d1eeb52a13
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91300426"
---
# <a name="debug-user-defined-functions-in-azure-stream-analytics"></a>Déboguer des fonctions définies par l’utilisateur dans Azure Stream Analytics 

Lorsque les fonctions définies par l’utilisateur (FDU) ne fonctionnent pas comme prévu, vous devez les déboguer pour trouver la cause du problème. Vous pouvez déboguer des FDU pour vos travaux Stream Analytics lorsque vous exécutez ces derniers localement à l’aide de [Visual Studio Code](visual-studio-code-local-run-live-input.md) ou [Visual Studio](stream-analytics-vs-tools-local-run.md).

Lorsque vous exécutez votre travail sur un flux d’entrée actif localement, il imite uniquement l’exécution du moteur cloud d’Azure Stream Analytics sur un nœud. Les tests locaux des données actives ne peuvent pas remplacer les tests de performance et de scalabilité que vous effectuez dans le cloud, mais vous gagnez du temps pendant les tests fonctionnels en n’ayant pas à soumettre votre travail au cloud chaque fois que vous voulez le tester. En outre, les stratégies temporelles sont désactivées pour l’exécution locale avec des données locales ou des échantillons, mais elles sont prises en charge pour les tests de données actives.

## <a name="pick-your-language"></a>Choisir le langage

Vous pouvez écrire des FDU pour Azure Stream Analytics à l’aide de .NET (C#) ou JavaScript. 

### <a name="functions-in-c"></a>Fonctions en C# 

Lorsque vous [écrivez des FDU .NET à l’aide de Visual Studio](stream-analytics-edge-csharp-udf-methods.md), vous bénéficiez du même niveau de prise en charge que pour n’importe quel projet de classe .NET. Cette prise en charge comprend les éléments suivants :

* Prise en charge de la compilation, telle que la vérification syntaxique et la prise en charge du compilateur.

* Possibilité d’ajouter, de générer et de référencer un projet C# et des artefacts dans votre solution Stream Analytics. 

* Réutilisation facile du code encapsulé dans un projet partageable. 

* Prise en charge du débogage directement dans Visual Studio : définissez le projet Stream Analytics comme projet de départ et fixez des points d’arrêt dans le code C#. Ensuite, appuyez sur **F5** pour déboguer le code C# comme vous le feriez pour tout autre projet C#. 

### <a name="functions-in-javascript"></a>Fonctions en JavaScript

JavaScript est une autre option pour créer des fonctions dans Stream Analytics. Le code JavaScript est directement placé dans la zone de fonction du projet Stream Analytics, ce qui complique le partage entre les projets.

La compilation se produit lorsque le projet de Stream Analytics est compilé ou exécuté. Le risque de détecter un problème uniquement au moment de l’exécution est plus élevé. Il n’existe pas de prise en charge du débogage pour les fonctions en JavaScript directement dans Stream Analytics.

## <a name="debug-options-for-javascript"></a>Options de débogage pour JavaScript

Étant donné qu’il n’existe pas de prise en charge du débogage pour les fonctions en JavaScript directement dans Stream Analytics, vous pouvez déboguer la fonction en l’encapsulant dans un site HTML et en obtenant la sortie de celui-ci.

L’exemple suivant montre comment déboguer des FDU JavaScript avec certaines limitations dans un environnement d’exécution intégré dans [Visual Studio Code](quick-create-visual-studio-code.md).

### <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que votre projet Azure Stream Analytics contient les éléments suivants :

* Une entrée 
* Une sortie 
* Une requête Stream Analytics (.asaql) 
* Une configuration du travail Stream Analytics (JobConfig.json)
* FDU JavaScript

### <a name="prepare-files"></a>Préparer les fichiers

Dans l’image suivante, le fichier de requête *.asaql* contient uniquement l’appel à la FDU, *fxcharCount*. Cette modification garantit que vous êtes toujours en mesure de compiler le projet une fois les modifications apportées.

> [!div class="mx-imgBorder"]
> ![Stream Analytics test query file](./media/debug-user-defined-functions/asaql-file.png)

Créez un dossier supplémentaire dans **Tests** pour héberger le fichier de test, qui est appelé pour exécuter le test avec la fonction JavaScript. Dans cet exemple, le nom du dossier est *fxcharCount*, et celui du test est *Test_UDF.js*. 

L’image ci-dessous montre le code dans le fichier de test, qui charge le fichier de fonction et exécute la fonction. Cet exemple est simple, mais vous pouvez charger des fichiers de données de test supplémentaires et passer en boucle des tests supplémentaires pour obtenir le résultat. La notation de l’appel de fonction est légèrement différente des appels courants, car le fichier est référencé et n’est pas chargé dans le runtime, ce qui rend le débogage possible. 

> [!div class="mx-imgBorder"]
> ![Stream Analytics test file](./media/debug-user-defined-functions/test-file.png)

Dans la fonction, ajoutez les lignes de code suivantes au fichier pour exposer les méthodes. Elles n’ont pas d’impact sur la capacité à compiler le code dans Visual Studio Code.

```javascript
var methods = {};
methods.fxchartCount = fxchartCount;
exports.data = methods;
``` 

> [!div class="mx-imgBorder"]
> ![Stream Analytics JavaScript UDF](./media/debug-user-defined-functions/udf-file.png)
  
## <a name="install-debug-support"></a>Installer la prise en charge du débogage

Pour déboguer, vous devez [télécharger](https://nodejs.org/en/download/) et installer **node.js**. Installez la version appropriée en fonction de la plateforme que vous utilisez. Après avoir installé le runtime node.js, redémarrez Visual Studio Code pour implémenter les modifications. 

Sélectionnez **Exécuter et déboguer** ou appuyez sur **CTRL + MAJ + D** pour démarrer le débogage. Une zone de liste modifiable s’affiche pour vous permettre de sélectionner **node.js** comme runtime. Si vous n’avez installé que node.js, il est utilisé par défaut. Vous devez être en mesure d’exécuter le code pas à pas et d’accéder au fichier satellite si nécessaire avec F11. 

> [!div class="mx-imgBorder"]
> ![Stream Analytics run and debug udf](./media/debug-user-defined-functions/run-debug-udf.png)

### <a name="debug-user-defined-aggregates"></a>Déboguer les agrégats définis par l’utilisateur 

Vous pouvez utiliser la méthode de débogage des FDU JavaScript pour déboguer des agrégats définis par l’utilisateur (ADU). Dans cet exemple, un ADU est ajouté au fichier de requête *.asaql* et au fichier de test.

Comme pour la FDU, vous incluez un appel à l’ADU pour vous assurer que le projet se compilera une fois les modifications apportées. 

> [!div class="mx-imgBorder"]
> ![Add uda to asaql](./media/debug-user-defined-functions/asaql-uda.png)

Dans le fichier *Test_UDA.js*, vous référencez le fichier ADU comme vous l’avez fait pour la FDU. En outre, vous appelez `main()`, `init()` et `accumulate()`. La méthode `accumulate()` est appelée dans une boucle pour placer les valeurs dans la pile d’état. La méthode `computeresult()` est appelée pour composer la requête finale. 

> [!div class="mx-imgBorder"]
> ![UDA test file](./media/debug-user-defined-functions/uda-test.png)

Comme dans l’exemple de FDU, du code doit être ajouté à l’ADU lui-même pour exposer les méthodes pertinentes.

```javascript
var methods = {};
methods.main = main;
methods.init = main.init;
methods.accumulate = main.accumulate;
methods.computeResult = main.computeResult;
exports.data = methods;
``` 

> [!div class="mx-imgBorder"]
> ![Code added to UDA](./media/debug-user-defined-functions/uda-expose-methods.png)

Sélectionnez **Exécuter et déboguer** ou appuyez sur **CTRL + MAJ + D** pour démarrer le débogage. Une zone de liste modifiable s’affiche pour vous permettre de sélectionner **node.js** comme runtime. Si vous n’avez installé que node.js, il est utilisé par défaut. Vous devez être en mesure d’exécuter le code pas à pas et d’accéder au fichier satellite si nécessaire avec F11.

> [!div class="mx-imgBorder"]
> ![Stream Analytics run and debug uda](./media/debug-user-defined-functions/run-debug-uda.png)


## <a name="next-steps"></a>Étapes suivantes

* [Développer et déboguer des travaux Azure Stream Analytics localement](develop-locally.md)
* [Déboguer des requêtes Azure Stream Analytics en local à l’aide du diagramme de travail dans Visual Studio](debug-locally-using-job-diagram.md)
* [Fonctions définies par l’utilisateur dans Azure Stream Analytics](functions-overview.md)
 
