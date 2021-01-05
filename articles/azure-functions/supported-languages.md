---
title: Langages pris en charge dans Azure Functions
description: Découvrez les langues prises en charge (GA) et celles qui sont en préversion, ainsi que les méthodes d’extension du développement des fonctions dans d’autres langages.
ms.topic: conceptual
ms.date: 11/27/2019
ms.openlocfilehash: 91a6ea886c3828678771b24d69bb7987af1fb105
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83994899"
---
# <a name="supported-languages-in-azure-functions"></a>Langages pris en charge dans Azure Functions

Cet article explique les niveaux de prise en charge offerts pour les langages que vous pouvez utiliser avec Azure Functions. Il décrit également les stratégies de création de fonctions à l’aide de langages non pris en charge en mode natif.

## <a name="levels-of-support"></a>Niveaux de prise en charge

Il y a deux niveaux de prise en charge :

* **Disposition générale (GA)** : entièrement pris en charge et approuvé pour la production.
* **Préversion** : pas encore pris en charge, mais le statut de disponibilité générale est prévu à l’avenir.

## <a name="languages-by-runtime-version"></a>Langues par version du runtime 

[Trois versions du runtime Azure Functions](functions-versions.md) sont disponibles. Le tableau suivant montre les langages qui sont pris en charge dans chaque version du runtime.

[!INCLUDE [functions-supported-languages](../../includes/functions-supported-languages.md)]

## <a name="custom-handlers-preview"></a>Gestionnaires personnalisés (préversion) 

Les gestionnaires personnalisés sont des serveurs web légers qui reçoivent des événements de l’hôte Azure Functions. Tout langage qui prend en charge les primitives HTTP peut implémenter un gestionnaire personnalisé. Cela signifie que les gestionnaires personnalisés peuvent être utilisés pour créer des fonctions dans des langages qui ne sont pas officiellement prises en charge. Pour en savoir plus, consultez [Gestionnaires personnalisés Azure Functions (préversion)](functions-custom-handlers.md).

## <a name="language-extensibility"></a>Extensibilité de langage

À compter de la version 2.x, le runtime est conçu pour offrir une [extensibilité de langage](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Language-Extensibility). Les langages JavaScript et Java dans le runtime 2.x sont générés avec cette extensibilité.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir plus en détails comment développer des fonctions dans les langages pris en charge, voir les ressources suivantes :

+ [Informations de référence pour les développeurs de bibliothèques de classes C#](functions-dotnet-class-library.md)
+ [Informations de référence pour les développeurs de scripts C#](functions-reference-csharp.md)
+ [Informations de référence pour les développeurs Java](functions-reference-java.md)
+ [Informations de référence pour les développeurs JavaScript](functions-reference-node.md)
+ [Informations de référence pour les développeurs PowerShell](functions-reference-powershell.md)
+ [Informations de référence pour les développeurs Python](functions-reference-python.md)
+ [Informations de référence pour les développeurs TypeScript](functions-reference-node.md#typescript)
