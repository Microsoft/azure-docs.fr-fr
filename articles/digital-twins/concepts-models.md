---
title: Modèles personnalisés
titleSuffix: Azure Digital Twins
description: Découvrez comment Azure Digital Twins utilise des modèles définis par l’utilisateur pour décrire des entités dans votre environnement.
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 0a38f9b8135fed08a95df68f108e44c34fec6325
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94955325"
---
# <a name="understand-twin-models-in-azure-digital-twins"></a>Comprendre les modèles de jumeau dans Azure Digital Twins

L’une des principales caractéristiques d’Azure Digital Twins est la possibilité de définir votre propre vocabulaire et à créer votre graphique de jumeau dans les termes définis par votre entreprise. Cette fonctionnalité est fournie par le biais de **modèles** définis par l’utilisateur. Vous pouvez considérer les modèles comme des noms dans une description de votre monde. 

Un modèle est comparable à une **classe** dans un langage de programmation orienté objet, définissant une forme de données pour un concept particulier dans votre environnement de travail réel. Les modèles ont des noms (par exemple, *Room* ou *TemperatureSensor*) et contiennent des éléments tels que des propriétés, de la télémétrie ou des événements, ainsi que des commandes décrivant ce que peut faire ce type d’entité dans votre environnement. Plus tard, vous utiliserez ces modèles pour créer des [**jumeaux numériques**](concepts-twins-graph.md) représentant des entités spécifiques répondant à une description de ce type.

Les modèles Azure Digital Twins sont représentés avec le **langage DTDL (Digital Twins Definition Language)** basé sur JSON-LD.  

## <a name="digital-twin-definition-language-dtdl-for-models"></a>Langage DTDL (Digital Twin Definition Language) pour les modèles

Les modèles pour Azure Digital Twins sont définis à l’aide du DTDL (Digital Twin Definition Language). Le langage DTDL est basé sur JSON-LD et est indépendant du langage de programmation. Le langage DTDL n’est pas exclusif d’Azure Digital Twins. Il est également utilisé pour représenter des données d’appareil dans d’autres services IoT tels que [IoT Plug-and-Play](../iot-pnp/overview-iot-plug-and-play.md). 

Azure Digital Twins utilise le **langage DTDL _version 2_**. Pour plus d’informations sur cette version de DTDL, consultez sa documentation spec dans GitHub : [*DTDL (Digital Twins Definition Language) – version 2*](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). L’utilisation de DTDL _version 1_ avec Azure Digital Twins est désormais dépréciée.

> [!NOTE] 
> Les services qui utilisent le langage DTDL n’implémentent pas tous exactement les mêmes fonctionnalités de ce langage. Par exemple, la technologie IoT Plug-and-Play n’utilise pas les fonctionnalités DTDL destinées aux graphiques, tandis qu’Azure Digital Twins n’implémente pas actuellement de commandes DTDL.
>
> Pour plus d’informations sur les fonctionnalités DTDL spécifiques d’Azure Digital Twins, consultez, plus loin dans cet article, la section concernant les [spécificités de l’implémentation du langage DTDL dans Azure Digital Twins](#azure-digital-twins-dtdl-implementation-specifics).

## <a name="elements-of-a-model"></a>Éléments d’un modèle

Dans une définition de modèle, l’élément de code de niveau supérieur est une **interface**. Celle-ci encapsule le modèle entier, le reste du modèle étant défini dans l’interface. 

Une interface de modèle DTDL peut contenir de zéro à un grand nombre des champs suivants :
* **Propriété** : certaines propriétés sont des champs de données représentant l’état d’une entité ; c’est par exemple le cas des propriétés dans de nombreux langages de programmation orientés objet. Les propriétés ont un stockage de sauvegarde et peuvent être lues à tout moment.
* **Télémétrie** : les champs de télémétrie représentent des mesures ou des événements, et sont souvent utilisés pour décrire des lectures de capteur d’appareil. Contrairement aux propriétés, la télémétrie n’est pas stockée sur un jumeau numérique. Il s’agit d’une série d’événements de données limités dans le temps qui doivent être traités au fur et à mesure qu’ils se produisent. Pour plus d’informations sur les différences entre les propriétés et la télémétrie, consultez la section [*Propriétés et télémétrie*](#properties-vs-telemetry) ci-dessous.
* **Composant** : les composants vous permettent, le cas échéant, de créer votre interface de modèle en tant qu’assembly d’autres interfaces. Un exemple de composant est une interface *frontCamera*, ainsi qu’une autre interface de composant *backCamera*, utilisées pour la définition d’un modèle de *téléphone*. Vous devez commencer par définir une interface pour le composant *frontCamera*, comme s’il s’agissait de son propre modèle, puis la référencer lors de la définition de *téléphone*.

    Utilisez un composant pour décrire un élément qui fait partie intégrante de votre solution mais n’a pas besoin d’identité séparée, et qu’il n’est pas nécessaire de créer, supprimer ou réorganiser de manière indépendante dans le graphique de jumeaux. Si vous souhaitez que les entités aient des existences indépendantes dans le graphique de jumeaux, représentez-les en tant que jumeaux numériques séparés de différents modèles connectés par des *relations* (voir la puce suivante).
    
    >[!TIP] 
    >Des composants peuvent également être utilisés pour une organisation, afin de regrouper des ensembles de propriétés connexes à l’intérieur d’une interface de modèle. Dans ce cas, vous pouvez considérer chaque composant comme un espace de noms, ou « dossier », à l’intérieur de l’interface.
* **Relation** : les relations vous permettent de représenter la manière dont un jumeau numérique peut être impliqué avec d’autres jumeaux numériques. Elles peuvent avoir différents sens sémantiques. Par exemple *contains* (« floor contains room »), *cools* (« hvac cools room »), *isBilledTo* (« compressor is billed to user »), etc. Les relations permettent à la solution de fournir un graphique d’entités associées.

> [!NOTE]
> La [spec pour le langage DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) définit également des **commandes**, c’est-à-dire des méthodes qui peuvent être exécutées sur un jumeau numérique (comme une commande de réinitialisation ou d’allumage/extinction d’un ventilateur). Cependant, les *commandes ne sont actuellement pas prises en charge dans Azure Digital Twins*.

### <a name="properties-vs-telemetry"></a>Propriétés et télémétrie

Voici quelques conseils supplémentaires sur la distinction entre les champs DTDL **Propriétés** et **Télémétrie** dans Azure Digital Twins.

Les différences entre les propriétés et la télémétrie pour les modèles Azure Digital Twins sont les suivantes :
* Les **propriétés** sont supposées avoir un stockage de sauvegarde. Cela signifie que vous pouvez lire une propriété à tout moment et en récupérer la valeur. Si la propriété est accessible en écriture, vous pouvez également stocker une valeur dans la propriété.  
* La **télémétrie** ressemble davantage à un flux d’événements. Il s’agit d’un ensemble de messages de données qui ont des durées de vie courtes. Si vous ne configurez pas l’écoute pour l’événement et les actions à entreprendre lorsqu’il se produit, il n’y a plus de trace de l’événement par la suite. Vous ne pouvez pas y revenir et le lire plus tard. 
  - En termes C#, la télémétrie est comme un événement C#. 
  - En termes IoT, la télémétrie est généralement une mesure unique envoyée par un appareil.

La **télémétrie** est souvent utilisée avec les appareils IoT, car de nombreux appareils ne sont pas en mesure de stocker les valeurs de mesure qu’ils génèrent, ou n’en ont pas besoin. Ils les envoient simplement sous forme de flux d’événements de « télémétrie ». Dans ce cas, vous ne pouvez pas interroger l’appareil à tout moment pour connaître la valeur la plus récente du champ de télémétrie. Au lieu de cela, vous devez écouter les messages envoyés par l’appareil et effectuer des actions à mesure que les messages arrivent. 

Par conséquent, lors de la conception d’un modèle dans Azure Digital Twins, vous utiliserez probablement des **propriétés** dans la plupart des cas pour modéliser vos jumeaux. Cela vous permet de disposer du stockage de sauvegarde et de lire et d’interroger les champs de données.

La télémétrie et les propriétés collaborent souvent pour traiter l’entrée de données provenant des appareils. Comme toute entrée dans Azure Digital Twins se fait via des [API](how-to-use-apis-sdks.md), vous utiliserez généralement votre fonction d’entrée pour lire les événements de télémétrie ou de propriété des appareils et définirez une propriété dans ADT en réponse. 

Vous pouvez également publier un événement de télémétrie à partir de l’API Azure Digital Twins. Comme pour toute autre télémétrie, il s’agit d’un événement de courte durée qui nécessite un écouteur pour le traiter.

### <a name="azure-digital-twins-dtdl-implementation-specifics"></a>Spécificités de l’implémentation du langage DTDL dans Azure Digital Twins

Pour qu’un modèle DTDL soit compatible avec Azure Digital Twins, il doit répondre à ces exigences.

* Tous les éléments DTDL de niveau supérieur dans un modèle doivent être de type *interface*. Cela est dû au fait que les API de modèle Azure Digital Twins peuvent recevoir des objets JSON représentant une interface ou un tableau d’interfaces. Par conséquent, aucun autre type d’élément DTDL n’est autorisé au niveau supérieur.
* Le langage DTDL pour Azure Digital Twins ne doit pas définir de *commandes*.
* Azure Digital Twins n’autorise qu’un seul niveau d’imbrication de composant. Cela signifie qu’une interface utilisée en tant que composant ne peut pas avoir de composants. 
* Les interfaces ne peuvent pas être définies comme incluses dans d’autres interfaces DTDL. Elles doivent être définies en tant qu’entités de niveau supérieur distinctes avec leurs propres ID. Ensuite, si une autre interface souhaite inclure cette interface en tant que composant ou par héritage, elle peut référencer son ID.

En outre, Azure Digital Twins ne tient pas compte de l’attribut `writable` sur les propriétés ou les relations. Bien que cet attribut puisse être défini en fonction des spécifications DTDL, la valeur n’est pas utilisée par Azure Digital Twins. Au lieu de cela, les propriétés ou les relations sont toujours traitées comme accessibles en écriture par les clients externes qui ont des autorisations d’écriture générales sur le service Azure Digital Twins.

## <a name="example-model-code"></a>Exemple de code de modèle

Des modèles de type de jumeau peuvent être écrits dans n’importe quel éditeur de texte. Le langage DTDL suivant la syntaxe JSON, vous devez donc stocker les modèles avec l’extension *.json*. L’utilisation de l’extension JSON permet à de nombreux éditeurs de texte de programmation de fournir une vérification et une mise en évidence de syntaxe de base pour vos documents DTDL. Il existe également une [extension DTDL](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-dtdl) disponible pour [Visual Studio Code](https://code.visualstudio.com/).

Cette section contient un exemple de modèle classique, écrit sous la forme d’une interface DTDL. Le modèle décrit des **planètes**, chacune avec un nom, une masse et une température.
 
Sachez que les planètes peuvent également interagir avec des **lunes** qui sont leurs satellites et qu’elles peuvent contenir des **cratères**. Dans l’exemple ci-dessous, le modèle de `Planet` exprime les connexions à ces autres entités en référençant deux modèles externes, `Moon` et `Crater`. Ces modèles sont également définis dans l’exemple de code ci-dessous, mais ils sont très simples pour ne pas porter atteinte à l’exemple de `Planet` principal.

```json
[
  {
    "@id": "dtmi:com:contoso:Planet;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2",
    "displayName": "Planet",
    "contents": [
      {
        "@type": "Property",
        "name": "name",
        "schema": "string"
      },
      {
        "@type": "Property",
        "name": "mass",
        "schema": "double"
      },
      {
        "@type": "Telemetry",
        "name": "Temperature",
        "schema": "double"
      },
      {
        "@type": "Relationship",
        "name": "satellites",
        "target": "dtmi:com:contoso:Moon;1"
      },
      {
        "@type": "Component",
        "name": "deepestCrater",
        "schema": "dtmi:com:contoso:Crater;1"
      }
    ]
  },
  {
    "@id": "dtmi:com:contoso:Crater;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2"
  },
  {
    "@id": "dtmi:com:contoso:Moon;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2"
  }
]
```

Les champs du modèle sont les suivants :

| Champ | Description |
| --- | --- |
| `@id` | Identificateur pour le modèle. Doit être au format `dtmi:<domain>:<unique model identifier>;<model version number>`. |
| `@type` | Identifie le type des informations décrites. Pour une interface, le type est *interface*. |
| `@context` | Définit le [contexte](https://niem.github.io/json/reference/json-ld/context/) pour le document JSON. Les modèles doivent utiliser `dtmi:dtdl:context;2`. |
| `displayName` | [facultatif] Vous permet, le cas échéant, d’attribuer au modèle un nom convivial. |
| `contents` | Toutes les données d’interface restantes sont placées ici, sous la forme d’un tableau de définitions d’attributs. Chaque attribut doit fournir un `@type` (*Property*, *Telemetry*, *Command*, *Relationship* ou *Component*) pour identifier le type d’informations d’interface qu’il décrit, puis un ensemble de propriétés qui définissent l’attribut réel (par exemple, `name` et `schema` pour définir une *Propriété*). |

> [!NOTE]
> Notez que l’interface de composant (*Crater* dans cet exemple) est définie dans le même tableau que l’interface qui l’utilise (*Planet*). Les composants doivent être définis de cette façon dans les appels d’API, afin que l’interface soit trouvée.

### <a name="possible-schemas"></a>Schémas possibles

Comme pour langage DTDL, le schéma pour les attributs *Property* et *Telemetry* peuvent être composés de types primitifs standard (`integer`, `double`, `string` et `Boolean`) et d’autres types tels que `DateTime` et `Duration`. 

En plus des types primitifs, les champs *Property* et *Telemetry* peuvent avoir les types complexes suivants :
* `Object`
* `Map`
* `Enum`

Les champs *Telemetry* prennent également en charge `Array`.

### <a name="model-inheritance"></a>Héritage de modèle

Il est parfois nécessaire de spécialiser un modèle. Par exemple, il peut être utile de disposer d’un modèle générique, *Room*, et de variantes spécialisées, *ConferenceRoom* et *Gym*. Pour exprimer la spécialisation, le langage DTDL prend en charge l’héritage : des interfaces peuvent hériter d’une ou plusieurs autres interfaces. 

L’exemple suivant revisite le modèle *Planet* de l’exemple de DTDL précédent en tant que sous-type d’un modèle *CelestialBody* plus grand. Une fois le modèle « parent » défini, il est utilisé pour élaborer le modèle « enfant » à l’aide du champ `extends`.

```json
[
  {
    "@id": "dtmi:com:contoso:CelestialBody;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2",
    "displayName": "Celestial body",
    "contents": [
      {
        "@type": "Property",
        "name": "name",
        "schema": "string"
      },
      {
        "@type": "Property",
        "name": "mass",
        "schema": "double"
      },
      {
        "@type": "Telemetry",
        "name": "temperature",
        "schema": "double"
      }
    ]
  },
  {
    "@id": "dtmi:com:contoso:Planet;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2",
    "displayName": "Planet",
    "extends": "dtmi:com:contoso:CelestialBody;1",
    "contents": [
      {
        "@type": "Relationship",
        "name": "satellites",
        "target": "dtmi:com:contoso:Moon;1"
      },
      {
        "@type": "Component",
        "name": "deepestCrater",
        "schema": "dtmi:com:contoso:Crater;1"
      }
    ]
  },
  {
    "@id": "dtmi:com:contoso:Crater;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2"
  }
]
```

Dans cet exemple, le modèle *CelestialBody* fournit un nom, une masse et une température au modèle *Planet*. La section `extends` est un nom d’interface ou un tableau de noms d’interface (permettant, le cas échéant, à l’interface d’extension d’hériter de plusieurs modèles parents).

Une fois l’héritage appliqué, l’interface d’extension expose toutes les propriétés de la chaîne d’héritage entière.

L’interface d’extension ne peut pas modifier les définitions des interfaces parentes, mais simplement y faire des ajouts. Elle ne peut pas non plus redéfinir une fonctionnalité déjà définie dans ses interfaces parentes (même si les fonctionnalités sont définies pour être identiques). Par exemple, si une interface parente définit une propriété `double` *mass*, l’interface d’extension ne peut pas contenir une déclaration de *mass*, même s’il s’agit également d’un `double`.

## <a name="best-practices-for-designing-models"></a>Bonnes pratiques en matière de conception de modèles

Lors de la conception de modèles pour refléter les entités dans votre environnement, il peut s’avérer utile d’examiner les implications des [requêtes](concepts-query-language.md) dans votre conception. Vous avez peut-être envie de concevoir des propriétés de manière à éviter les grands jeux de résultats provenant de la traversée de graphe. Vous pouvez également modéliser les relations qui doivent être traitées dans une seule requête en tant que relations à un seul niveau.

### <a name="validating-models"></a>Validation des modèles

[!INCLUDE [Azure Digital Twins: validate models info](../../includes/digital-twins-validate.md)]

## <a name="integrating-with-industry-standard-models"></a>Intégration à des modèles aux normes du secteur

L’utilisation de modèles basés sur des normes du secteur ou d’une représentation d’ontologie standard, comme RDF ou OWL, constitue un point de départ intéressant quand vous concevez vos modèles Azure Digital Twins. L’utilisation de modèles sectoriels facilite également la normalisation et le partage d’informations.

Pour être utilisé avec Azure Digital Twins, un modèle doit être représenté à l’aide du [**langage DTDL (Digital Twins Definition Language)**](concepts-models.md) basé sur JSON-LD. Par conséquent, cet article décrit comment représenter vos modèles standard du secteur en DTDL, en intégrant les concepts existants à la sémantique DTDL, de sorte qu’Azure Digital Twins puissent les utiliser. Le modèle DTDL sert alors de source de vérité pour le modèle dans Azure Digital Twins.

Deux démarches existent pour intégrer des modèles aux normes du secteur avec DTDL, selon votre situation :
* Si vous n’avez pas encore créé vos modèles, vous pouvez les concevoir autour d’**ontologies DTDL de démarrage existantes** qui contiennent la langue propre à votre secteur d’activité.
* Si vous avez déjà des modèles existants basés sur une norme du secteur, vous devez **les convertir en DTDL** afin de les intégrer à Azure Digital Twins.

Pour plus d’informations sur ces deux processus, consultez [*Guide pratique : Intégrer des modèles standard du secteur*](how-to-integrate-models.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment gérer les modèles avec les API de DigitalTwinModels :
* [*Guide pratique : Gérer des modèles personnalisés*](how-to-manage-model.md)

Vous pouvez également découvrir comment des jumeaux numériques sont créés à partir de modèles :
* [*Concepts : Jumeaux numériques et graphe de jumeaux*](concepts-twins-graph.md)

