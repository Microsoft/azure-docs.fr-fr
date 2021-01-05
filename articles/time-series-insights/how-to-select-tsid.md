---
title: Meilleures pratiques pour le choix d’un ID de série chronologique - Azure Time Series Insights | Microsoft Docs
description: Découvrez les meilleures pratiques pour le choix d’un ID de série chronologique dans la préversion d’Azure Time Series Insights Gen2.
author: shipramishra
ms.author: shmishr
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 09/30/2020
ms.custom: seodec18
ms.openlocfilehash: c3f6f6a5ac1068f2eabca351e85376b8e16d1058
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95016749"
---
# <a name="best-practices-for-choosing-a-time-series-id"></a>Meilleures pratiques pour choisir un ID de série chronologique

Cet article résume l’importance de l’identifiant de l’ID de série chronologique pour la préversion de votre environnement Azure Time Series Insights Gen2 et les meilleures pratiques afin d’en choisir un.

## <a name="choose-a-time-series-id"></a>Choisir un ID Time Series

Il est essentiel de sélectionner un ID de série chronologique approprié. Le choix d’un ID de série chronologie est identique au choix d’une clé de partition pour une base de données. Cela est requis lorsque vous créez un environnement Azure Time Series Insights Gen2.

> [!IMPORTANT]
> Les ID des séries chronologiques sont les suivants :
>
> * Une propriété *respectant la casse* : les cachants de lettres et de caractères sont utilisés dans les recherches, les comparaisons, les mises à jour et le partitionnement.
> * Une *propriété immuable* : une fois créée, elle ne peut pas être modifiée.

> [!TIP]
> Si votre source d’événement est un hub IoT, votre ID de série chronologique sera probablement ***iothub-connection-device-id** _.

Les bonnes pratiques principales à suivre sont les suivantes :

_ Sélectionnez une clé de partition comportant de nombreuses valeurs distinctes (plusieurs centaines ou plusieurs milliers). Dans de nombreux cas, il peut s'agir de l'ID de l’appareil, de l'ID du capteur ou de l'ID de l'étiquette de votre JSON.
* L’ID de série chronologique doit être unique au niveau du nœud de feuille de votre [modèle de série chronologique](./concepts-model-overview.md).
* La limite de caractères pour la chaîne de nom de propriété de l’ID de série chronologie est 128. Pour la valeur de propriété de l’ID de série chronologie, la limite de caractères est de 1 024.
* Si une valeur de propriété unique pour l'ID de série chronologique est manquante, elle est traitée comme une valeur nulle et suit la même règle de la contrainte d'unicité.
* Si votre ID de série chronologique est imbriqué dans un objet JSON complexe, veillez à suivre les [règles d’aplanissement](./concepts-json-flattening-escaping-rules.md) d’entrée quand vous fournissez votre nom de propriété. Consultez l’exemple [B](concepts-json-flattening-escaping-rules.md#example-b).
* Vous pouvez également sélectionner jusqu'à *trois* propriétés clés comme ID de série chronologique. Leur combinaison sera une clé composite qui représente l'ID de série chronologique.  
  > [!NOTE]
  > Vos trois propriétés de clé doivent être des chaînes.
  > Vous devrez interroger cette clé composite au lieu d'une propriété à la fois.

## <a name="select-more-than-one-key-property"></a>Sélectionnez plus d’une propriété de clé

Les scénarios suivants décrivent la sélection de plusieurs propriétés de clé en tant qu’ID de série chronologique.  

### <a name="example-1-time-series-id-with-a-unique-key"></a>Exemple 1 : ID de série chronologique avec une clé unique

* Vous avez des parcs hérités d’actifs, chacun avec une clé unique.
* Un parc est identifié de façon unique par la propriété **deviceId**, et un par la propriété unique **objectId**. Aucun des parcs ne contient la propriété unique de l’autre parc. Dans cet exemple, vous sélectionnez deux clés, **deviceId** et **objectId**, comme clés uniques.
* Nous acceptons les valeurs null, et l’absence d’une propriété dans la charge utile d’événement est comptabilisée comme une valeur nulle. Il s’agit également de la méthode appropriée pour gérer l’envoi de données à deux sources d’événements, où les données de chaque source d’événement ont un ID unique de série chronologique.

### <a name="example-2-time-series-id-with-a-composite-key"></a>Exemple 2 : ID de série chronologique avec une clé composite

* Plusieurs propriétés doivent être uniques au sein du même parc d’actifs.
* Vous construisez des bâtiments intelligents et déployez des capteurs dans chaque pièce. Dans chaque pièce, vous avez généralement les mêmes valeurs pour **sensorId**, telles que **sensor1**, **sensor2** et **sensor3**.
* Votre bâtiment a des numéros d’étages et de pièces qui se chevauchent sur les différents sites dans la propriété **flrRm**, qui ont des valeurs telles que **1a**, **2b**, **3a**, et ainsi de suite.
* Vous disposez d’une propriété, **location**, qui contient des valeurs telles que **Redmond**, **Barcelona** et **Tokyo**. Pour créer l’unicité, vous désignez les trois propriétés suivantes en tant que clés d’ID de série chronologique : **sensorId**, **flrRm**, et **location**.

Exemple d’événement brut :

```JSON
{
  "sensorId": "sensor1",
  "flrRm": "1a",
  "location": "Redmond",
  "temperature": 78
}
```

Dans le portail Azure, vous pouvez ensuite saisir cette clé composite sous la forme :

[![Configurez l’ID de la série chronologique de l’environnement](media/v2-how-to-tsid/configure-environment-key.png)](media/v2-how-to-tsid/configure-environment-key.png#lightbox)

  > [!NOTE]
  > Dans le portail Azure, n’entrez pas de noms de propriétés séparés par des virgules dans une zone de texte. Autrement, ils seront traités comme un nom de propriété unique contenant des virgules.
  > Entrez chaque nom de propriété dans sa propre zone de texte.

## <a name="next-steps"></a>Étapes suivantes

* Lisez les [Règles de mise à plat et d’échappement JSON](./concepts-json-flattening-escaping-rules.md) pour comprendre comment les événements seront stockés.

* Planifier votre [environnement Azure Time Series Insights Gen2](./how-to-plan-your-environment.md).