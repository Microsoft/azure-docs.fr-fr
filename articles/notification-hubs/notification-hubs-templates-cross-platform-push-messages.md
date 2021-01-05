---
title: Modèles de hubs de notifications Azure
description: En savoir plus sur l’utilisation de modèles pour les hubs de notifications Azure.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: a41897bb-5b4b-48b2-bfd5-2e3c65edc37e
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: e3c64155053517205ab006673bb8f400325ad3c4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86219961"
---
# <a name="notification-hubs-templates"></a>Modèles Notification Hubs

Les modèles permettent à une application cliente de spécifier le format exact des notifications que celle-ci souhaite recevoir. Une application peut utiliser des modèles pour en tirer plusieurs avantages, notamment les suivants :

- Un serveur principal non spécifique à une plateforme
- Des notifications personnalisées
- Indépendance vis-à-vis de la version du client
- Localisation facile

Cette section fournit deux exemples détaillés de la manière d’utiliser des modèles pour envoyer des notifications non spécifiques à une plateforme ciblant tous vos appareils sur des plateformes, et pour personnaliser la notification à diffusion générale pour chaque appareil.

## <a name="using-templates-cross-platform"></a>Utilisation des modèles entre plateformes

La méthode classique d’envoi de notifications Push est d’envoyer, pour chaque notification à envoyer, une charge utile spécifique aux services de notification de plateforme (WNS, APNS). Par exemple, pour envoyer une alerte aux APNS, la charge utile est un objet JSON au format suivant :

```json
{"aps": {"alert" : "Hello!" }}
```

Pour envoyer un message toast similaire à une application Windows Store, la charge utile XML est la suivante :

```xml
<toast>
  <visual>
    <binding template=\"ToastText01\">
      <text id=\"1\">Hello!</text>
    </binding>
  </visual>
</toast>
```

Vous pouvez créer des charges utiles similaires pour les plateformes MPNS (Windows Phone) et FCM (Android).

Cette condition force le serveur principal de l’application à générer différentes charges utiles pour chaque plateforme, et rend en partie le serveur principal responsable de la couche présentation de l’application. Certaines préoccupations concernent les présentations graphiques et la localisation (en particulier pour les applications Windows Store qui incluent des notifications pour divers types de mosaïques).

La fonctionnalité de modèle Notification Hubs permet à une application cliente de créer des inscriptions spéciales, appelées inscriptions de modèles, qui incluent, en plus de l’ensemble de balises, un modèle. La fonctionnalité de modèle Notification Hubs permet à une application cliente d’associer des appareils à des modèles, que vous travailliez avec des installations (recommandé) ou des inscriptions. Dans les exemples de charge utile précédents, les seules informations non spécifiques à la plateforme sont le message d’alerte réel (Hello!). Un modèle est un ensemble d’instructions destiné au hub de notification concernant la mise en forme d’un message non spécifique à une plateforme pour l’inscription de cette application cliente spécifique. Dans l’exemple précédent, le message non spécifique à une plateforme est une propriété unique : `message = Hello!`.

L’image suivante illustre le processus :

![Diagramme illustrant le processus d'utilisation des modèles multiplateformes](./media/notification-hubs-templates/notification-hubs-hello.png)

Le modèle pour l’inscription d’une application cliente iOS est le suivant :

```json
{"aps": {"alert": "$(message)"}}
```

Le modèle correspondant de l’application cliente Windows Store est :

```xml
<toast>
    <visual>
        <binding template=\"ToastText01\">
            <text id=\"1\">$(message)</text>
        </binding>
    </visual>
</toast>
```

Notez que le message réel est remplacé par l’expression $(message). Cette expression indique au hub de notification de créer un message qui respecte ce modèle et insère la valeur courante chaque fois qu’il envoie un message à cette inscription particulière.

Si vous travaillez avec le modèle Installation, la clé « templates » de l’installation conserve un objet JSON de plusieurs modèles. Si vous travaillez avec le modèle Inscription, l’application cliente peut créer plusieurs inscriptions afin d’utiliser plusieurs modèles ; par exemple, un modèle pour les messages d’alerte et un modèle pour les mises à jour des mosaïques. Les applications clientes peuvent également combiner inscriptions natives (inscriptions sans modèle) et inscriptions avec modèle.

Le hub de notification envoie une notification pour chaque modèle, sans tenir compte de leur appartenance, ou non, à la même application cliente. Ce comportement permet de convertir les notifications non spécifiques à une plateforme en plusieurs notifications. Par exemple, le même message non spécifique à une plateforme envoyé au hub de notification peut être converti de façon transparente en une alerte toast et une mise à jour de mosaïque, sans que le serveur principal n’en soit nécessairement averti. Certaines plateformes (par exemple, iOS) peuvent regrouper plusieurs notifications sur le même appareil si elles sont envoyées sur une durée très courte.

## <a name="using-templates-for-personalization"></a>Utilisation des modèles à des fins de personnalisation

Un autre avantage de l’utilisation des modèles est la possibilité d’utiliser les hubs de notification pour effectuer une personnalisation de notifications en fonction de l’inscription. Par exemple, prenez une application météo qui affiche une mosaïque contenant les conditions climatiques à un emplacement spécifique. Un utilisateur peut choisir entre les degrés Celsius ou Fahrenheit, et une prévision à un jour ou sur les cinq prochains jours. Grâce aux modèles, chaque installation d’application cliente peut s’inscrire au format approprié (Celsius sur 1 jour, Fahrenheit sur 1 jour, Celsius sur 5 jours, Fahrenheit sur 5 jours) et faire appel au serveur principal pour envoyer un message unique contenant toutes les informations nécessaires pour compléter ces modèles (par exemple, une prévision sur cinq jours en degrés Celsius et Fahrenheit).

Le modèle pour la prévision sur un jour avec les températures en degrés Celsius est comme suit :

```xml
<tile>
  <visual>
    <binding template="TileWideSmallImageAndText04">
      <image id="1" src="$(day1_image)" alt="alt text"/>
      <text id="1">Seattle, WA</text>
      <text id="2">$(day1_tempC)</text>
    </binding>  
  </visual>
</tile>
```

Le message envoyé au hub de notification contient les propriétés suivantes :

```html
<table border="1">

<tr><td>day1_image</td><td>day2_image</td><td>day3_image</td><td>day4_image</td><td>day5_image</td></tr>

<tr><td>day1_tempC</td><td>day2_tempC</td><td>day3_tempC</td><td>day4_tempC</td><td>day5_tempC</td></tr>

<tr><td>day1_tempF</td><td>day2_tempF</td><td>day3_tempF</td><td>day4_tempF</td><td>day5_tempF</td></tr>
</table><br/>
```

En utilisant ce modèle, le serveur principal envoie uniquement un message sans avoir à stocker des options de personnalisation spécifiques pour les utilisateurs de l’application. L’image suivante illustre ce scénario :

![Diagramme montrant que le back-end n'envoie qu'un seul message à chaque plateforme.](./media/notification-hubs-templates/notification-hubs-registration-specific.png)

## <a name="how-to-register-templates"></a>Inscription des modèles

Pour vous inscrire auprès de modèles à l’aide du modèle Installation (recommandé) ou du modèle Inscription, consultez [Gestion des inscriptions](notification-hubs-push-notification-registration-management.md).

## <a name="template-expression-language"></a>Langage d’expression de modèle

Les modèles sont limités aux formats de documents XML ou JSON. Vous pouvez également placer des expressions à des endroits particuliers ; par exemple, des attributs de nœud ou des valeurs pour XML, des valeurs de propriété de chaîne pour JSON.

Le tableau suivant indique le langage autorisé dans les modèles :

| Expression       | Description |
| ---------------- | --- |
| $(prop)          | Référence à une propriété d’événement avec le nom donné. Les noms de propriétés ne respectent pas la casse. Cette expression se résout en valeur texte de la propriété ou en une chaîne vide si la propriété n’est pas présente. |
| $(prop, n)       | Comme ci-dessus, mais le texte est explicitement coupé à n caractères ; par exemple, $(title, 20) coupe le contenu de la propriété de titre à 20 caractères. |
| .(prop, n)       | Comme ci-dessus, mais le texte est suivi de trois points lorsqu’il est coupé. La taille totale de la chaîne coupée et du suffixe ne dépasse pas n caractères. .(title, 20) avec une propriété d’entrée de « Ceci est la ligne du titre » donne **Ceci est la ligne...** |
| %(prop)          | Similaire à $(name), sauf que la sortie est codée URI. |
| #(prop)          | Utilisé dans les modèles JSON (par exemple, pour les modèles iOS et Android).<br><br>Cette fonction agit exactement comme l’expression $(prop) spécifiée précédemment, sauf lorsqu’elle est utilisée dans les modèles JSON (par exemple, les modèles Apple). Dans ce cas, si cette fonction n’est pas placée entre « {’,’} » (par exemple, ’myJsonProperty’ : ’#(name)’), et qu’elle retourne un nombre au format JavaScript, par exemple, regexp: (0&#124;(&#91;1-9&#93;&#91;0-9&#93;*))(\.&#91;0-9&#93;+)?((e&#124;E)(+&#124;-)?&#91;0-9&#93;+)?, la sortie JSON est un nombre.<br><br>Par exemple, ‘badge: ‘#(name)’ devient ‘badge’ : 40 (et non ‘40‘). |
| ‘text’ ou « text » | Littéral. Les littéraux contiennent du texte arbitraire placé entre guillemets simples ou doubles. |
| expr1 + expr2    | L’opérateur de concaténation joint deux expressions en une seule chaîne. |

Les expressions peuvent avoir n’importe laquelle des formes précédentes.

Lors de l’utilisation de la concaténation, l’expression tout entière doit être placée entre `{}`. Par exemple : `{$(prop) + ‘ - ’ + $(prop2)}`.

Par exemple, le modèle suivant n’est pas un modèle XML valide :

```xml
<tile>
  <visual>
    <binding $(property)>
      <text id="1">Seattle, WA</text>
    </binding>  
  </visual>
</tile>
```

Comme expliqué précédemment, lors de l’utilisation de la concaténation, les expressions doivent être placées entre accolades. Par exemple :

```xml
<tile>
  <visual>
    <binding template="ToastText01">
      <text id="1">{'Hi, ' + $(name)}</text>
    </binding>  
  </visual>
</tile>
```

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur les hubs de notifications Azure](notification-hubs-push-notification-overview.md)