---
title: Configurer des règles et des actions dans Azure IoT Central
description: Dans le cadre de cet article de guide pratique, vous découvrez, en tant que générateur, comment configurer des règles et des actions basées sur la télémétrie dans votre application Azure IoT Central.
author: vavilla
ms.author: vavilla
ms.date: 12/23/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: philmea
ms.openlocfilehash: 378a1dbcfbc89cdf9b24dc2490db583f1135b9a2
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97796786"
---
# <a name="configure-rules"></a>Configurer des règles

*Cet article s’applique aux opérateurs, aux créateurs et aux administrateurs.*

Dans IoT Central, les règles servent d’outil de réponse personnalisable qui déclenche des événements faisant l’objet d’une supervision active à partir d’appareils connectés. Les sections suivantes expliquent comment les règles sont évaluées.

## <a name="select-target-devices"></a>Sélectionner les appareils cibles

Utilisez la section Appareils cibles pour sélectionner le type d’appareils auxquels cette règle sera appliquée. Des filtres vous permettent d’affiner davantage les appareils à inclure. Les filtres utilisent des propriétés sur le modèle d’appareil pour filtrer l’ensemble d’appareils. Les filtres ne déclenchent pas eux-mêmes une action. Dans la capture d’écran suivante, les appareils ciblés sont de type de modèle d’appareil **Refrigerator** (Réfrigérateur). Le filtre indique que la règle doit inclure uniquement les réfrigérateurs (**Refrigerator**) où la propriété **Manufactured State** (État de fabrication) a la valeur **Washington**.

![Conditions](media/howto-configure-rules/filters.png)

## <a name="use-multiple-conditions"></a>Utiliser plusieurs conditions

Les conditions déterminent ce qui entraîne le déclenchement des règles. Actuellement, quand vous ajoutez plusieurs conditions à une règle, elles sont agrégées (AND) de manière logique. En d’autres termes, toutes les conditions doivent être remplies pour que la règle soit évaluée comme vraie.  

Dans la capture d’écran suivante, les conditions vérifient que la température est supérieure à 70 &deg;F et que l’humidité est inférieure à 10. Quand ces deux instructions sont vraies (true), la règle prend la valeur true et déclenche une action.

![La capture d’écran représente un moniteur de réfrigérateur avec des conditions spécifiées pour la température et l’humidité.](media/howto-configure-rules/conditions.png)

### <a name="use-a-cloud-property-in-a-value-field"></a>Utiliser une propriété Cloud dans un champ de valeur

Vous pouvez référencer une propriété de Cloud du modèle d’appareil dans le champ **Valeur** pour une condition. La propriété de Cloud et la valeur de télémétrie doivent être de types similaires. Par exemple, si **Température** est un double, seules les propriétés Cloud de type double s’affichent en tant qu’options dans la liste déroulante **Valeur**.

Si vous choisissez une valeur de télémétrie de type d’événement, la liste déroulante **Valeur** comprend l’option **Toute**. L’option **Toute** signifie que la règle se déclenche quand votre application reçoit un événement de ce type, quelle que soit la charge utile.

## <a name="use-aggregate-windowing"></a>Utiliser le fenêtrage des temps d’agrégation

Les règles évaluent les fenêtres des temps d’agrégation sous forme de fenêtres bascule. Dans la capture d’écran ci-dessous, la fenêtre de temps est de cinq minutes. Toutes les cinq minutes, la règle est évaluée sur les cinq dernières minutes de données. Les données ne sont évaluées qu’une seule fois dans la fenêtre à laquelle elles correspondent.

![Fenêtres bascule](media/howto-configure-rules/tumbling-window.png)

## <a name="use-rules-with-iot-edge-modules"></a>Utiliser des règles avec des modules IoT Edge

Une restriction s’applique aux règles qui sont appliquées aux modules IoT Edge. Les règles sur les données de télémétrie provenant de différents modules ne sont pas évaluées en tant que règles valides. Considérez l’exemple suivant. La première condition de la règle s’applique à des données de télémétrie de température provenant du module A. La deuxième condition de la règle s’applique à des données de télémétrie d’humidité provenant du module B. Étant donné que les deux conditions proviennent de modules différents, il s’agit d’un ensemble de conditions non valide. La règle n’est pas valide et génère une erreur quand vous tentez de l’enregistrer.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à configurer une règle dans votre application Azure IoT Central, vous pouvez apprendre à [Configurer des règles avancées](howto-configure-rules-advanced.md) à l’aide de Power Automate ou d’Azure Logic Apps.
