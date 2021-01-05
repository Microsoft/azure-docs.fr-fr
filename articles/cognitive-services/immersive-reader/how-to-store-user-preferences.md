---
title: Stocker les préférences utilisateur
titleSuffix: Azure Cognitive Services
description: Cet article vous indique comment stocker les préférences de l’utilisateur.
author: metanMSFT
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: conceptual
ms.date: 06/29/2020
ms.author: metang
ms.openlocfilehash: 31c1ef8d75b4c12e4dd6a360852feb27857ac412
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636542"
---
# <a name="how-to-store-user-preferences"></a>Stocker les préférences de l’utilisateur

Cet article explique comment stocker les paramètres d’interface utilisateur de l’utilisateur, précédemment appelés **préférences de l’utilisateur** , via les options [-preferences](./reference.md#options) et [-onPreferencesChanged](./reference.md#options) du SDK Lecteur immersif.

Lorsque l’option du SDK [CookiePolicy](./reference.md#cookiepolicy-options) est définie sur *Activé* , l’application Lecteur immersif stocke les **préférences de l’utilisateur** (taille du texte, couleur du thème, police, etc.) dans les cookies locaux, dans un navigateur et sur un appareil spécifiques. À chaque fois que l’utilisateur lance le Lecteur immersif sur le même navigateur et le même appareil, il s’ouvre avec les préférences de l’utilisateur de la dernière session sur cet appareil. Toutefois, si l’utilisateur ouvre le Lecteur immersif dans un autre navigateur ou sur un autre appareil, les paramètres sont initialement configurés avec les paramètres par défaut du Lecteur immersif, et l’utilisateur doit à nouveau définir ses préférences, et ainsi de suite pour chaque appareil qu’il utilise. Les options `-preferences` et `-onPreferencesChanged` du SDK Lecteur immersif offrent aux applications un moyen d’activer l’itinérance des préférences d’un utilisateur entre différents navigateurs et appareils, de sorte que l’utilisateur bénéficie d’une expérience cohérente partout où il utilise l’application.

Tout d’abord, en fournissant l’option de rappel de `-onPreferencesChanged` du SDK lors du lancement de l’application Lecteur immersif, le Lecteur immersif renvoie une chaîne `-preferences` à l’application hôte à chaque fois que l’utilisateur modifie ses préférences pendant la session du Lecteur immersif. L’application hôte est alors responsable du stockage des préférences de l’utilisateur dans son propre système. Ensuite, lorsque ce même utilisateur lance à nouveau le Lecteur immersif, l’application hôte peut récupérer les préférences de l’utilisateur à partir du stockage et les fournir comme option de chaîne `-preferences` du SDK lors du lancement de l’application Lecteur immersif, afin que les préférences de l’utilisateur soient restaurées.

Cette fonctionnalité peut être utilisée comme un autre moyen de stocker les **préférences de l’utilisateur** au cas où l’utilisation de cookies n’est pas souhaitable ou possible.

> [!CAUTION]
> **IMPORTANT** N’essayez pas de modifier par programmation les valeurs de la chaîne `-preferences` envoyée à destination et en provenance de l’application Lecteur immersif, car cela peut provoquer un comportement inattendu entraînant une dégradation de l’expérience utilisateur pour vos clients. Les applications hôtes ne doivent jamais attribuer une valeur personnalisée ou manipuler la chaîne `-preferences`. Quand vous utilisez l’option de chaîne `-preferences`, utilisez uniquement la valeur exacte qui a été retournée à partir de l’option de rappel de `-onPreferencesChanged`.

## <a name="how-to-enable-storing-user-preferences"></a>Comment activer le stockage des préférences de l’utilisateur

Le paramètre [launchAsync](./reference.md#launchasync) `options` du SDK Lecteur immersif contient le rappel de `-onPreferencesChanged`. Cette fonction est appelée à chaque fois que l’utilisateur modifie ses préférences. Le paramètre `value` contient une chaîne qui représente les préférences actuelles de l’utilisateur. Cette chaîne est ensuite stockée, pour cet utilisateur, par l’application hôte.

```typescript
const options = {
    onPreferencesChanged: (value: string) => {
        // Store user preferences here
    }
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, YOUR_DATA, options);
```

## <a name="how-to-load-user-preferences-into-the-immersive-reader"></a>Comment charger les préférences de l’utilisateur dans le Lecteur immersif

Transmettez les préférences de l’utilisateur au Lecteur immersif en utilisant l’option `-preferences`. Voici un exemple tout simple pour stocker et charger les préférences de l’utilisateur :

```typescript
const storedUserPreferences = localStorage.getItem("USER_PREFERENCES");
let userPreferences = storedUserPreferences === null ? null : storedUserPreferences;
const options = {
    preferences: userPreferences,
    onPreferencesChanged: (value: string) => {
        userPreferences = value;
        localStorage.setItem("USER_PREFERENCES", userPreferences);
    }
};
```

## <a name="next-steps"></a>Étapes suivantes

* Explorer le [Guide de référence du SDK du Lecteur immersif](./reference.md)