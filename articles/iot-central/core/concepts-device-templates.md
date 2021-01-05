---
title: Présentation des modèles d’appareils dans Azure IoT Central | Microsoft Docs
description: Les modèles d’appareils Azure IoT Central vous permettent de spécifier le comportement des appareils connectés à votre application. Un modèle d’appareil spécifie les données de télémétrie, les propriétés et les commandes que l’appareil doit implémenter. Un modèle d’appareil définit également l’interface utilisateur pour l’appareil dans IoT Central tels que les formulaires et les tableaux de bord utilisés par un opérateur.
author: dominicbetts
ms.author: dobett
ms.date: 12/19/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom: device-developer
ms.openlocfilehash: 04c2330ffee396f5fc30b85640e992df77c08263
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97795426"
---
# <a name="what-are-device-templates"></a>Que sont les modèles d’appareils ?

_Cet article s’applique aux développeurs d’appareils et aux créateurs de solutions._

Un modèle d’appareil dans Azure IoT Central est un blueprint qui définit les caractéristiques et comportements d’un type d’appareil se connectant à votre application. Par exemple, le modèle d’appareil définit la télémétrie qu’un appareil envoie afin qu’IoT Central puisse créer des visualisations qui utilisent les unités et types de données appropriés.

Un générateur de solutions ajoute des modèles d’appareils à une application IoT Central. Un développeur d’appareils écrit le code d’appareil qui implémente les comportements définis dans le modèle d’appareil.

Un modèle d’appareil comprend les sections suivantes :

- _Un modèle d’appareil_. Cette partie du modèle d’appareil définit la manière dont l’appareil interagit avec votre application. Un développeur d’appareils implémente les comportements définis dans le modèle.
    - _Composant par défaut_. Chaque modèle d’appareil possède un composant par défaut. L’interface du composant par défaut décrit les fonctionnalités spécifiques au modèle d’appareil.
    - _Composants_. En plus du composant par défaut, un modèle d’appareil peut inclure des composants pour décrire les fonctionnalités de l’appareil. Chaque composant inclut une interface qui décrit ses fonctionnalités. Les interfaces de composant peuvent être réutilisées dans d’autres modèles d’appareils. Par exemple, plusieurs modèles d’appareils téléphoniques peuvent utiliser la même interface d’appareil photo.
    - _Interfaces héritées_. Un modèle d’appareil contient une ou plusieurs interfaces qui étendent les fonctionnalités du composant par défaut.
- _Propriétés Cloud_. Cette partie du modèle d’appareil permet au développeur de solutions de spécifier toutes les métadonnées de l’appareil à stocker. Les propriétés Cloud ne sont jamais synchronisées avec les appareils et n’existent que dans l’application. Les propriétés Cloud n’affectent pas le code écrit par un développeur d’appareils pour implémenter le modèle d’appareil.
- _Personnalisations_. Cette partie du modèle d’appareil permet au développeur de solutions de remplacer certaines définitions dans le modèle d’appareil. Des personnalisations sont utiles si le développeur de solutions souhaite affiner la façon dont l’application gère une valeur, par exemple, en modifiant le nom d’affichage d’une propriété ou de la couleur utilisée pour afficher une valeur de télémétrie. Les personnalisations n’affectent pas le code qu’un développeur d’appareils a écrit pour implémenter le modèle d’appareil.
- _Affichages_. Cette partie du modèle d’appareil permet au développeur de solutions de définir des visualisations pour afficher les données de l’appareil, ainsi que des formulaires pour gérer et contrôler un appareil. Les affichages utilisent le modèle d’appareil, les propriétés Cloud et les personnalisations. Les affichages n’affectent pas le code qu’un développeur d’appareils a écrit pour implémenter le modèle d’appareil.

## <a name="device-models"></a>Modèles d’appareils

Un modèle d’appareil définit la manière dont un appareil interagit avec votre application IoT Central. Le développeur de l’appareil doit s’assurer que l’appareil implémente les comportements définis dans le modèle d’appareil afin qu’IoT Central puisse surveiller et gérer l’appareil. Un modèle d’appareil est constitué d’une ou plusieurs _interfaces_ pouvant chacune définir une collection de types de _télémétries_, des _propriétés d’appareil_ et des _commandes_. Un développeur de solutions peut importer un fichier JSON définissant le modèle d’appareil dans un modèle d’appareil, ou utiliser l’interface utilisateur web dans IoT Central pour créer ou modifier un modèle d’appareil. Les modifications apportées à un modèle d’appareil à l’aide de l’interface utilisateur web nécessitent que le modèle d’appareil fasse l’objet d’une [gestion des versions](./howto-version-device-template.md).

Un développeur de solutions peut également exporter un fichier JSON contenant le modèle d’appareil. Un développeur d’appareils peut utiliser ce document JSON pour comprendre comment l’appareil doit communiquer avec l’application IoT Central.

Le fichier JSON qui définit le modèle d’appareil utilise le [langage DTDL (Digital Twin Definition Language) v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). IoT Central s’attend à ce que le fichier JSON contienne le modèle d’appareil avec les interfaces définies incluses plutôt que dans des fichiers distincts.

Un appareil IoT classique est constitué des éléments suivants :

- Des parties personnalisées, qui rendent votre appareil unique.
- Des parties standard, communes à tous les appareils.

Ces éléments sont appelés _interfaces_ dans un modèle d’appareil. Les interfaces définissent les détails de chaque partie implémentée par votre appareil. Les interfaces sont réutilisables entre les modèles d’appareils. Dans le DTDL, un composant fait référence à une interface définie dans un fichier DTDL distinct.

L’exemple suivant montre le plan du modèle d’appareil pour un contrôleur de température. Le composant par défaut comprend des définitions pour `workingSet`, `serialNumber` et `reboot`. Le modèle d’appareil inclut également les interfaces `thermostat` et `deviceInformation` :

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:TemperatureController;1",
  "@type": "Interface",
  "displayName": "Temperature Controller",
  "description": "Device with two thermostats and remote reboot.",
  "contents": [
    {
      "@type": [
        "Telemetry", "DataSize"
      ],
      "name": "workingSet",
      "displayName": "Working Set",
      "description": "Current working set of the device memory in KiB.",
      "schema": "double",
      "unit" : "kibibyte"
    },
    {
      "@type": "Property",
      "name": "serialNumber",
      "displayName": "Serial Number",
      "description": "Serial number of the device.",
      "schema": "string"
    },
    {
      "@type": "Command",
      "name": "reboot",
      "displayName": "Reboot",
      "description": "Reboots the device after waiting the number of seconds specified.",
      "request": {
        "name": "delay",
        "displayName": "Delay",
        "description": "Number of seconds to wait before rebooting the device.",
        "schema": "integer"
      }
    },
    {
      "@type" : "Component",
      "schema": "dtmi:com:example:Thermostat;1",
      "name": "thermostat",
      "displayName": "Thermostat",
      "description": "Thermostat One."
    },
    {
      "@type": "Component",
      "schema": "dtmi:azure:DeviceManagement:DeviceInformation;1",
      "name": "deviceInformation",
      "displayName": "Device Information interface",
      "description": "Optional interface with basic device hardware information."
    }
  ]
}
```

Une interface contient certains champs obligatoires :

- `@id` : ID unique sous la forme d’un nom de ressource uniforme simple.
- `@type` : déclare que cet objet est une interface.
- `@context` : spécifie la version DTDL utilisée pour l’interface.
- `contents` : liste les propriétés, données de télémétrie et commandes qui composent votre appareil. Les fonctionnalités peuvent être définies dans plusieurs interfaces.

Il existe des champs facultatifs que vous pouvez utiliser pour ajouter des détails au modèle de capacité, tels que le nom d’affichage et la description.

Chaque entrée de la liste d’interfaces dans la section implements comprend :

- `name` : nom de programmation de l’interface.
- `schema` : interface implémentée par le modèle de capacité.

## <a name="interfaces"></a>Interfaces

Le DTDL vous permet de décrire les fonctionnalités de votre appareil. Les fonctionnalités associées sont regroupées dans des interfaces. Les interfaces décrivent les propriétés, la télémétrie et les commandes qu’une partie de votre appareil implémente :

- `Properties`. Les propriétés sont des champs de données qui représentent l’état de votre appareil. Utilisez les propriétés pour représenter l’état durable de l’appareil, par exemple l’état marche/arrêt d’une pompe de refroidissement. Les propriétés peuvent également représenter des propriétés d’appareil de base, telles que la version du microprogramme de l’appareil. Vous pouvez déclarer des propriétés comme étant accessibles en lecture seule ou en écriture. Seuls les appareils peuvent mettre à jour la valeur d’une propriété en lecture seule. Un opérateur peut définir la valeur d’une propriété accessible en écriture à envoyer à un appareil.
- `Telemetry`. Les champs de télémétrie représentent les mesures des capteurs. Chaque fois que votre appareil prend une mesure de capteur, il doit envoyer un événement de télémétrie contenant les données de capteur.
- `Commands`. Les commandes représentent les méthodes que les utilisateurs de votre appareil peuvent exécuter sur l’appareil. Par exemple, une commande de réinitialisation ou une commande pour allumer ou éteindre un ventilateur.

L’exemple suivant montre la définition d’une interface de thermostat :

```json
{
  "@context": "dtmi:dtdl:context;2",
  "@id": "dtmi:com:example:Thermostat;1",
  "@type": "Interface",
  "displayName": "Thermostat",
  "description": "Reports current temperature and provides desired temperature control.",
  "contents": [
    {
      "@type": [
        "Telemetry",
        "Temperature"
      ],
      "name": "temperature",
      "displayName" : "Temperature",
      "description" : "Temperature in degrees Celsius.",
      "schema": "double",
      "unit": "degreeCelsius"
    },
    {
      "@type": [
        "Property",
        "Temperature"
      ],
      "name": "targetTemperature",
      "schema": "double",
      "displayName": "Target Temperature",
      "description": "Allows to remotely specify the desired target temperature.",
      "unit" : "degreeCelsius",
      "writable": true
    },
    {
      "@type": [
        "Property",
        "Temperature"
      ],
      "name": "maxTempSinceLastReboot",
      "schema": "double",
      "unit" : "degreeCelsius",
      "displayName": "Max temperature since last reboot.",
      "description": "Returns the max temperature since last device reboot."
    },
    {
      "@type": "Command",
      "name": "getMaxMinReport",
      "displayName": "Get Max-Min report.",
      "description": "This command returns the max, min and average temperature from the specified time to the current time.",
      "request": {
        "name": "since",
        "displayName": "Since",
        "description": "Period to return the max-min report.",
        "schema": "dateTime"
      },
      "response": {
        "name" : "tempReport",
        "displayName": "Temperature Report",
        "schema": {
          "@type": "Object",
          "fields": [
            {
              "name": "maxTemp",
              "displayName": "Max temperature",
              "schema": "double"
            },
            {
              "name": "minTemp",
              "displayName": "Min temperature",
              "schema": "double"
            },
            {
              "name" : "avgTemp",
              "displayName": "Average Temperature",
              "schema": "double"
            },
            {
              "name" : "startTime",
              "displayName": "Start Time",
              "schema": "dateTime"
            },
            {
              "name" : "endTime",
              "displayName": "End Time",
              "schema": "dateTime"
            }
          ]
        }
      }
    }
  ]
}
```

Cet exemple montre deux propriétés (une en lecture seule et une autre accessible en écriture), un type de télémétrie et une commande. Une description de champ minimale comprend :

- `@type` pour spécifier le type de fonctionnalité : `Telemetry`, `Property` ou `Command`.  Dans certains cas, le type comprend un type sémantique pour permettre à IoT Central de faire des hypothèses sur la façon de gérer la valeur.
- `name` pour la valeur de télémétrie.
- `schema` pour spécifier le type de données pour la télémétrie ou la propriété. Cette valeur peut être un type primitif, tel que double, entier, valeur booléenne ou chaîne. Les types d’objets complexes et les mappages sont également pris en charge.

Des champs facultatifs, tels que le nom d’affichage et la description, vous permettent d’ajouter des détails à l’interface et aux fonctionnalités.

## <a name="properties"></a>Propriétés

Par défaut, les propriétés sont en lecture seule. Les propriétés en lecture seule signifient que l’appareil signale des mises à jour de valeurs de propriété à votre application IoT Central. Votre application IoT Central ne peut pas définir la valeur d’une propriété en lecture seule.

Vous pouvez également marquer une propriété comme étant accessible en écriture sur une interface. Un appareil peut recevoir de votre application IoT Central une mise à jour de propriété accessible en écriture ainsi que signaler des mises à jour de valeurs de propriété à votre application.

Les appareils n’ont pas besoin d’être connectés pour définir des valeurs de propriété. Les valeurs mises à jour sont transférées quand l’appareil se connecte ensuite à l’application. Ce comportement s’applique aux propriétés en lecture seule et en écriture.

N’utilisez pas de propriétés pour envoyer des données de télémétrie à partir de votre appareil. Par exemple, une propriété en lecture seule telle que `temperatureSetting=80` doit signifier que la température de l’appareil a été définie sur 80 et que l’appareil tente d’atteindre cette température ou d’y rester.

Pour les propriétés accessibles en écriture, l’application d’appareil retourne un code d’état, une version et une description de l’état souhaité pour indiquer si elle a reçu et appliqué la valeur de la propriété.

## <a name="telemetry"></a>Télémétrie

IoT Central vous permet d’afficher une télémétrie sur des tableaux de bord et des graphiques, ainsi que d’utiliser des règles pour déclencher des actions lorsque certains seuils sont atteints. IoT Central utilise les informations du modèle d’appareil, telles que les types de données, les unités et les noms d’affichage, pour déterminer la manière d’afficher les valeurs de télémétrie.

La fonctionnalité d’exportation de données d’IoT Central permet de diffuser en continu la télémétrie vers d’autres destinations, telles que le stockage ou Event Hubs.

## <a name="commands"></a>Commandes

Une commande doit être exécutée dans un délai de 30 secondes par défaut, et l’appareil doit être connecté quand la commande arrive. Si l’appareil ne répond pas à temps ou qu’il n’est pas connecté, la commande échoue.

Les commandes peuvent avoir des paramètres de requête et retourner une réponse.

### <a name="offline-commands"></a>Commandes hors connexion

Vous pouvez choisir des commandes de mise en file d’attente si un appareil est actuellement hors connexion en activant l’option **Mettre en file d’attente si hors connexion** pour une commande dans le modèle d’appareil.

Les commandes hors connexion représentent des notifications unidirectionnelles de votre solution vers l’appareil. Les commandes hors connexion peuvent avoir des paramètres de requête, mais elles ne renvoient pas de réponse.

> [!NOTE]
> Cette option est disponible uniquement dans l’interface utilisateur web d’IoT Central. Ce paramètre n’est pas inclus si vous exportez un modèle ou une interface à partir du modèle d’appareil.

## <a name="cloud-properties"></a>Propriétés du cloud

Les propriétés Cloud font partie du modèle d’appareil, mais pas du modèle d’appareil. Les propriétés Cloud permettent au développeur de solutions de spécifier les métadonnées d’appareil à stocker dans l’application IoT Central. Les propriétés Cloud n’affectent pas le code écrit par un développeur d’appareils pour implémenter le modèle d’appareil.

Un développeur de solutions peut ajouter des propriétés Cloud à des tableaux de bord et des affichages, en même temps que des propriétés de l’appareil pour permettre à un opérateur de gérer les appareils connectés à l’application. Un développeur de solutions peut également utiliser des propriétés Cloud dans le cadre d’une définition de règle pour rendre une valeur de seuil modifiable par un opérateur.

## <a name="customizations"></a>Personnalisations

Les personnalisations font partie du modèle d’appareil, mais pas du modèle d’appareil. Les personnalisations permettent au développeur de solutions d’améliorer ou de remplacer certaines définitions dans le modèle d’appareil. Par exemple, un développeur de solutions peut modifier le nom d’affichage d’un type ou d’une propriété de télémétrie. Un développeur de solutions peut également utiliser des personnalisations pour ajouter une validation telle qu’une longueur minimale ou maximale pour une propriété d’appareil de type chaîne.

Les personnalisations peuvent affecter le code qu’un développeur d’appareils écrit pour implémenter le modèle d’appareil. Par exemple, une personnalisation peut définir des longueurs de chaîne minimales et maximales ou des valeurs numériques minimales et maximales pour la télémétrie.

## <a name="views"></a>Les vues

Un développeur de solutions crée des affichages permettant aux opérateurs de surveiller et de gérer les appareils connectés. Les affichage faisant partie du modèle d’appareil, ils sont associés à un type d’appareil spécifique. Un affichage peut inclure les éléments suivants :

- Graphiques pour tracer la télémétrie.
- Vignettes pour afficher les propriétés d’appareil en lecture seule.
- Vignettes pour permettre à l’opérateur de modifier des propriétés d’appareil accessibles en écriture.
- Vignettes pour permettre à l’opérateur de modifier des propriétés Cloud.
- Vignettes pour permettre à l’opérateur d’appeler des commandes, y compris des commandes qui attendent une charge utile.
- Vignettes pour afficher des étiquettes, des images ou un texte Markdown.

La télémétrie, les propriétés et les commandes que vous pouvez ajouter à un affichage sont déterminées par le modèle d’appareil, des propriétés Cloud et des personnalisations dans le modèle d’appareil.

## <a name="next-steps"></a>Étapes suivantes

En tant que développeur d’appareils, maintenant que vous avez découvert les modèles d’appareils, une prochaine étape suggérée consiste à lire [Charges utiles de télémétrie, de propriété et de commande](./concepts-telemetry-properties-commands.md) pour en savoir plus sur les données qu’un appareil échange avec IoT Central.

En tant que développeur de solutions, nous vous suggérons de lire [Définir un nouveau type d’appareil IoT dans votre application Azure IoT Central](./howto-set-up-template.md) pour en savoir plus sur la création d’un modèle d’appareil.
