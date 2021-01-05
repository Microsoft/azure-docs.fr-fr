---
title: Comprendre le stockage des paires clé-valeur dans Azure App Configuration
description: Comprendre le stockage de paires clé-valeur dans Azure App Configuration, qui stocke les données de configuration sous cette forme. Les paires clé-valeur sont une représentation de paramètres d’application.
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 08/04/2020
ms.openlocfilehash: ad9e96433a7ee72476ae2251c684d17ec7a6d1ce
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96930243"
---
# <a name="keys-and-values"></a>Clés et valeurs

Azure App Configuration stocke les données de configuration sous la forme de paires clé-valeur. Les paires clé-valeur sont une représentation simple et flexible des paramètres d’application utilisés par les développeurs.

## <a name="keys"></a>Keys

Les clés servent d’identificateurs pour les paires clé-valeur. Elles sont utilisées pour stocker et récupérer des valeurs correspondantes. Il est courant d’organiser les clés dans un espace de noms hiérarchique en utilisant un caractère délimiteur, comme `/` ou `:`. Utilisez la convention qui convient le mieux à votre application. App Configuration traite les clés dans leur ensemble. Il n’analyse pas les clés pour déterminer la structure de leur nom ou pour appliquer une règle quelconque.

Voici un exemple de noms de clé structurés en une hiérarchie basée sur les services de composants :

```aspx
    AppName:Service1:ApiEndpoint
    AppName:Service2:ApiEndpoint
```

L’utilisation des données de configuration dans des infrastructures d’application peut exiger des schémas de nommage spécifiques pour les paires clé-valeur. Par exemple, le framework Spring Cloud de Java définit les ressources `Environment` qui fournissent des paramètres à une application Spring.  Celles-ci sont paramétrables par des variables qui incluent le *nom d’application* et le *profil*. Les clés des données de configuration associées à Spring Cloud commencent généralement par ces deux éléments, séparés par un délimiteur.

Les clés stockées dans App Configuration sont des chaînes Unicode qui respectent la casse. Les clés *app1* et *App1* sont distinctes dans un magasin App Configuration. Gardez cette précision à l’esprit quand vous utilisez des paramètres de configuration au sein d’une application, car certains frameworks gèrent les clés de configuration sans tenir compte de la casse. Nous vous déconseillons d’utiliser la casse pour différencier les clés.

Vous pouvez utiliser n’importe quel caractère Unicode dans les noms de clés, à l’exception de `%`. Un nom de clé ne peut pas contenir `.` ou `..` non plus. La taille combinée d’une paire clé-valeur est limitée à 10 ko. Sont inclus tous les caractères de la clé, sa valeur et tous les attributs facultatifs associés. À la hauteur de cette limite, vous pouvez avoir de nombreux niveaux hiérarchiques pour les clés.

### <a name="design-key-namespaces"></a>Concevoir des espaces de noms de clés

Il existe deux approches générales pour nommer des clés utilisées pour des données de configuration : la méthode plate ou la méthode hiérarchique. Ces méthodes sont similaires du point de vue de l’utilisation de l’application, mais le nommage hiérarchique offre plusieurs avantages :

* Plus facile à lire. Les délimiteurs inclus dans un nom de clé hiérarchique fonctionnent comme des espaces dans une phrase. Ils fournissent également des séparations naturelles entre les mots.
* Plus facile à gérer. Une hiérarchie de noms de clé représente des groupes logiques de données de configuration.
* Plus facile à utiliser. Il est plus simple d’écrire une requête qui fait correspondre des clés à un modèle dans une structure hiérarchique et de récupérer uniquement une partie des données de configuration. De plus, de nombreux frameworks de programmation récents prennent en charge nativement les données de configuration hiérarchiques de telle sorte que votre application peut utiliser des jeux spécifiques de configuration.

Vous pouvez organiser hiérarchiquement les clés dans App Configuration de nombreuses façons. Considérez ces clés comme des [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier). Chaque clé hiérarchique est un *chemin* de ressource constitué d’un ou de plusieurs composants qui sont joints par des délimiteurs. Choisissez le caractère à utiliser comme délimiteur en fonction des besoins de votre application, de votre langage de programmation ou de votre framework. Utilisez plusieurs délimiteurs pour différentes clés dans App Configuration.

### <a name="label-keys"></a>Clés d’étiquette

Les paires clé-valeur peuvent éventuellement porter un attribut d’étiquette dans App Configuration. Les étiquettes sont utilisées pour différencier les paires clé-valeur qui ont la même clé. Une clé *app1* avec des étiquettes *A* et *B* forme deux clés distinctes dans un magasin App Configuration. Par défaut, une paire clé-valeur n’a pas d’étiquette. Pour référencer explicitement une paire clé-valeur sans étiquette, utilisez `\0` (URL encodée sous la forme `%00`).

Label offre un moyen pratique de créer des variantes d’une clé. Les étiquettes sont souvent utilisées pour spécifier plusieurs environnements pour la même clé :

```
    Key = AppName:DbEndpoint & Label = Test
    Key = AppName:DbEndpoint & Label = Staging
    Key = AppName:DbEndpoint & Label = Production
```

### <a name="version-key-values"></a>Version de paires clé-valeur

Utilisez les étiquettes comme un moyen de créer plusieurs versions d’une paire clé-valeur . Par exemple, vous pouvez entrer un numéro de version d’application ou un ID de validation Git dans des étiquettes pour identifier les paires clé-valeur associées à une build logicielle particulière.

> [!NOTE]
> Si vous recherchez des versions de modification, Azure App Configuration conserve automatiquement toutes les modifications d’une paire clé-valeur survenues pendant une certaine période de temps. Pour plus d’informations, consultez [Capture instantanée à un point dans le temps](./concept-point-time-snapshot.md).

### <a name="query-key-values"></a>Interroger des paires clé-valeur

Chaque paire clé-valeur est identifiée de manière unique par sa clé et une étiquette pouvant être `\0`. Pour interroger un magasin App Configuration afin de connaître des paires clé-valeur, vous devez spécifier un modèle. Le magasin App Configuration retourne toutes les paires clé-valeur qui correspondent au modèle, ainsi que leurs valeurs et attributs correspondants. Utilisez les modèles de clé suivants dans les appels d’API REST à App Configuration :

| Clé | Description |
|---|---|
| `key` est omis ou `key=*` | Correspond à toutes les clés |
| `key=abc` | Correspond exactement au nom de clé **abc** |
| `key=abc*` | Correspond aux noms de clé qui commencent par **abc** |
| `key=abc,xyz` | Correspond aux noms de clé **abc** ou **xyz**. Limité à cinq valeurs séparées par des virgules |

Vous pouvez également inclure les modèles d’étiquette suivants :

| Étiquette | Description |
|---|---|
| `label` est omis ou `label=*` | Correspond à toute étiquette, y compris `\0` |
| `label=%00` | Correspond à l’étiquette `\0` |
| `label=1.0.0` | Correspond exactement à l’étiquette **1.0.0** |
| `label=1.0.*` | Correspond aux étiquettes qui commencent par **1.0.** |
| `label=%00,1.0.0` | Correspond aux étiquettes `\0` ou **1.0.0**, limitées à cinq valeurs séparées par des virgules (CSV) |

> [!NOTE]
> `*`, `,` et `\` sont des caractères réservés dans les requêtes. Si un caractère réservé est utilisé dans vos noms de clé ou vos étiquettes, vous devez le placer dans une séquence d’échappement en utilisant `\{Reserved Character}` dans les requêtes.

## <a name="values"></a>Valeurs

Les valeurs attribuées aux clés sont également des chaînes Unicode. Vous pouvez utiliser tous les caractères Unicode pour les valeurs.

### <a name="use-content-type"></a>Utiliser Content-Type
Chaque paire clé-valeur dans App Configuration a un attribut content-type. Vous pouvez éventuellement utiliser cet attribut pour stocker des informations sur le type de valeur dans une paire clé-valeur qui permet à votre application de les traiter correctement. Vous pouvez utiliser n’importe quel format pour content-type. App Configuration utilise des [types de médias]( https://www.iana.org/assignments/media-types/media-types.xhtml) (également appelés types MIME) pour les types de données intégrés tels que les indicateurs de fonctionnalité, les références Key Vault et les paires clé-valeur JSON.

## <a name="next-steps"></a>Étapes suivantes

* [Capture instantanée à un point dans le temps](./concept-point-time-snapshot.md)
* [Gestion des fonctionnalités](./concept-feature-management.md)
* [Gestion des événements](./concept-app-configuration-event.md)
