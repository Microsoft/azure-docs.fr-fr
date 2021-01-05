---
title: Conseils relatifs au développement d'Azure Functions
description: Découvrez les concepts et techniques Azure Functions dont vous avez besoin pour développer des fonctions dans Azure, et ce, pour tous les langages de programmation et toutes les liaisons.
ms.assetid: d8efe41a-bef8-4167-ba97-f3e016fcd39e
ms.topic: conceptual
ms.date: 10/12/2017
ms.openlocfilehash: 54bfd770fba9a1766396d66c0c263111c233c9c2
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96167877"
---
# <a name="azure-functions-developer-guide"></a>Guide du développeur Azure Functions
Dans Azure Functions, des fonctions spécifiques partagent quelques concepts techniques et composants de base, quels que soient le langage et la liaison que vous utilisez. Avant de passer à l'apprentissage des détails propres à un langage ou une liaison donnés, veillez à lire cette présentation qui s'applique à l’ensemble d’entre eux.

Cet article suppose que vous avez déjà lu la [Présentation d’Azure Functions](functions-overview.md).

## <a name="function-code"></a>Code de fonction
La *fonction* est le principal concept d'Azure Functions. Une fonction contient deux éléments importants - votre code, qui peut être écrit dans un certain nombre de langages, et, dans certaines configurations, le fichier function.json. Pour les langages compilés, ce fichier de configuration est généré automatiquement à partir des annotations de votre code. Pour les langages de script, vous devez fournir le fichier de configuration vous-même.

Le fichier function.json définit le déclencheur, les liaisons de fonction et d’autres paramètres de configuration. Chaque fonction possède un seul déclencheur. Le runtime utilise ce fichier de configuration pour déterminer les événements à surveiller et comment passer des données et renvoyer des données à partir de l’exécution d’une fonction. Voici un exemple de fichier function.json.

```json
{
    "disabled":false,
    "bindings":[
        // ... bindings here
        {
            "type": "bindingType",
            "direction": "in",
            "name": "myParamName",
            // ... more depending on binding
        }
    ]
}
```

Pour plus d’informations, consultez [Concepts des déclencheurs et liaisons Azure Functions](functions-triggers-bindings.md).

La propriété `bindings` vous permet de configurer les liaisons et les déclencheurs. Chaque liaison partage quelques paramètres communs et des paramètres propres à un type de liaison donné. Chaque liaison requiert les paramètres suivants :

| Propriété | Valeurs/types | Commentaires |
| --- | --- | --- |
| `type` |string |Type de liaison. Par exemple : `queueTrigger`. |
| `direction` |'in', 'out' |Indique si la liaison sert à recevoir des données dans la fonction ou à envoyer des données à partir de la fonction. |
| `name` |string |Le nom utilisé pour les données liées dans la fonction. Pour C#, il s’agit d’un nom d'argument ; pour JavaScript, il s’agit de la clé dans une liste de clés/valeurs. |

## <a name="function-app"></a>Conteneur de fonctions
Une application de fonction fournit un contexte d’exécution dans Azure dans lequel vos fonctions s’exécutent. À ce titre, elle constitue l'unité de déploiement et de gestion de vos fonctions. Un conteneur de fonctions est constitué d’une ou de plusieurs des fonctions individuelles qui sont gérées, déployées et mises à l’échelle ensemble. Toutes les fonctions d'une application de fonction partagent le même plan de tarification, la même méthode de déploiement et la même version du runtime. Considérez un conteneur de fonctions comme un moyen d’organiser et de gérer collectivement vos fonctions. Pour en savoir plus, consultez [Gérer une application de fonction](functions-how-to-use-azure-function-app-settings.md). 

> [!NOTE]
> Toutes les fonctions d’une application de fonction doivent être créées dans le même langage. Dans les [version précédentes](functions-versions.md) du runtime Azure Functions, cela n’était pas obligatoire.

## <a name="folder-structure"></a>Structure de dossiers
[!INCLUDE [functions-folder-structure](../../includes/functions-folder-structure.md)]

La structure de dossiers ci-dessus est recommandée et utilisée par défaut pour une application de fonction. Si vous souhaitez changer l’emplacement du code d’une fonction, modifiez la section `scriptFile` du fichier _function.json_. Nous vous recommandons également d’utiliser un [package de déploiement](deployment-zip-push.md) pour déployer votre projet sur votre application de fonction dans Azure. Vous pouvez également utiliser des outils existants tels que [l’intégration et le déploiement continus](functions-continuous-deployment.md) et Azure DevOps.

> [!NOTE]
> En cas de déploiement manuel d’un package, veillez à déployer votre fichier _host.json_ et vos dossiers de fonction directement dans le dossier `wwwroot`. N’incluez pas le dossier `wwwroot` dans vos déploiements. Sinon, vous vous retrouverez avec `wwwroot\wwwroot` dossiers.

#### <a name="use-local-tools-and-publishing"></a>Utiliser des outils locaux pour la publication
Vous pouvez créer et publier des applications de fonction à l’aide de différents outils, notamment [Visual Studio](./functions-develop-vs.md), [Visual Studio Code](./create-first-function-vs-code-csharp.md), [IntelliJ](./functions-create-maven-intellij.md), [Eclipse](./functions-create-maven-eclipse.md) et [Azure Functions Core Tools](./functions-develop-local.md). Pour plus d’informations, consultez [Coder et tester Azure Functions localement](./functions-develop-local.md).

<!--NOTE: I've removed documentation on FTP, because it does not sync triggers on the consumption plan --glenga -->

## <a name="how-to-edit-functions-in-the-azure-portal"></a><a id="fileupdate"></a> Comment modifier des fonctions dans le portail Azure
L’éditeur de fonctions intégré au portail Azure vous permet de mettre à jour votre code et votre fichier *function.json* directement en ligne. Cela n’est recommandé que pour les petites modifications ou les preuves de concept - la meilleure pratique est d’utiliser un outil de développement local comme VS Code.

## <a name="parallel-execution"></a>Exécution parallèle
Lorsque plusieurs événements de déclenchement se produisent plus rapidement qu'un runtime de fonction monothread ne peut les traiter, le runtime peut appeler la fonction plusieurs fois en parallèle.  Si un conteneur de fonctions utilise le [plan d’hébergement de consommation](functions-scale.md#how-the-consumption-and-premium-plans-work), il peut monter automatiquement effectuer un scale-out.  Que l’application s’exécute sur le plan d’hébergement de consommation ou sur un [plan d’hébergement App Service](../app-service/overview-hosting-plans.md) standard, chaque instance de l’application de fonction peut traiter en parallèle des appels de fonction simultanés en utilisant plusieurs threads.  Le nombre maximal d’appels de fonction simultanés dans chaque instance de chaque application de fonction varie en fonction du type de déclencheur utilisé, ainsi que des ressources utilisées par d’autres fonctions au sein de l’application de fonction.

## <a name="functions-runtime-versioning"></a>Contrôle de version du runtime Functions

Vous pouvez configurer la version du runtime Functions en utilisant le paramètre d’application `FUNCTIONS_EXTENSION_VERSION`. Par exemple, la valeur « ~3 » indique que votre application de fonction utilise 3.x comme version principale. Les applications de fonction sont mises à niveau vers chaque nouvelle version mineure lorsqu’elles sont disponibles. Pour obtenir plus d’informations, notamment sur la façon d’afficher la version exacte de votre application de fonction, consultez [Guide pratique pour cibler des versions du runtime Azure Functions](set-runtime-version.md).

## <a name="repositories"></a>Référentiels
Le code pour Azure Fonctions est open source et stocké dans des dépôts GitHub :

* [Azure Functions](https://github.com/Azure/Azure-Functions)
* [Hôte Azure Functions](https://github.com/Azure/azure-functions-host/)
* [Portail Azure Functions](https://github.com/azure/azure-functions-ux)
* [Modèles Azure Functions](https://github.com/azure/azure-functions-templates)
* [Kit de développement logiciel Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk/)
* [Extensions du kit de développement logiciel Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk-extensions/)

## <a name="bindings"></a>Liaisons
Voici un tableau de toutes les liaisons prises en charge.

[!INCLUDE [dynamic compute](../../includes/functions-bindings.md)]

Vous rencontrez des problèmes avec des erreurs de liaisons ? Consultez la documentation [Codes d’erreur de liaison d’Azure Functions](functions-bindings-error-pages.md).

## <a name="reporting-issues"></a>Problèmes liés aux rapports
[!INCLUDE [Reporting Issues](../../includes/functions-reporting-issues.md)]

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations, consultez les ressources suivantes :

* [Azure Functions triggers and bindings (Déclencheurs et liaisons Azure Functions)](functions-triggers-bindings.md)
* [Coder et tester Azure Functions localement](./functions-develop-local.md)
* [Meilleures pratiques pour Azure Functions](functions-best-practices.md)
* [Informations de référence pour les développeurs C# sur Azure Functions](functions-dotnet-class-library.md)
* [Informations de référence pour les développeurs Node.js sur Azure Functions](functions-reference-node.md)