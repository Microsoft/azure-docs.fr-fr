---
title: Effectuer une migration de l’API de traduction de conversation Translator Speech vers le service Speech
titleSuffix: Azure Cognitive Services
description: Découvrez comment effectuer la migration de vos applications entre l’API de traduction de conversation Translator Speech et le service Speech.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/21/2020
ms.author: aahi
ms.openlocfilehash: 2fb03721baa80e77a5fd387600a272e6b1cfc7d3
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95013644"
---
# <a name="migrate-from-the-translator-speech-api-to-the-speech-service"></a>Effectuer une migration de l’API de traduction de conversation Translator Speech vers le service Speech

Utilisez cet article pour effectuer la migration de vos applications entre l’API de traduction de conversation Microsoft Translator Speech et le [service Speech](index.yml). Ce guide souligne les différences entre l’API de traduction de conversation Translator Speech et le service Speech. Il propose également des stratégies pour la migration de vos applications.

> [!NOTE]
> La clé d’abonnement de votre API de traduction de conversation Translator Speech ne sera pas acceptée par le service Speech. Vous devrez créer un abonnement au service Speech.

## <a name="comparison-of-features"></a>Comparaison des fonctionnalités

| Fonctionnalité                                           | API de traduction de conversation Translator Speech                                  | Service Speech | Détails                                                                                                                                                                                                                                                                            |
|---------------------------------------------------|-----------------------------------------------------------------|------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Traduction en texte                               | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Traduction vocale                             | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Point de terminaison global                                   | :heavy_check_mark:                                              | :heavy_minus_sign:                 | Le service Speech n’offre pas de point de terminaison global. Un point de terminaison global dirige automatiquement le trafic vers le point de terminaison régional le plus proche, en réduisant la latence dans votre application.                                                    |
| Points de terminaison régionaux                                | :heavy_minus_sign:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Limite en termes de durée de connexion                             | 90 minutes                                               | Illimitée avec le kit de développement logiciel (SDK). 10 minutes avec une connexion WebSockets.                                                                                                                                                                                                                                                                                   |
| Clé d’authentification de l’en-tête                                | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Plusieurs langues traduites en une seule requête | :heavy_minus_sign:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Kits de développement logiciel (SDK) disponibles                                    | :heavy_minus_sign:                                              | :heavy_check_mark:                 | Pour connaître les kits SDK disponibles, consultez la [documentation du service Speech](index.yml).                                                                                                                                                    |
| Connexions WebSockets                            | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| API de langues                                     | :heavy_check_mark:                                              | :heavy_minus_sign:                 | Le service Speech prend en charge les langues décrites dans l’article [Informations de référence sur les langues du service Translator](). |
| Filtre et marqueur de propos vulgaires                       | :heavy_minus_sign:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| .WAV/PCM en tant qu’entrée                                 | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Autres types de fichiers en tant qu’entrée                         | :heavy_minus_sign:                                              | :heavy_minus_sign:                 |                                                                                                                                                                                                                                                                                    |
| Résultats partiels                                   | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Informations de durée                                       | :heavy_check_mark:                                              | :heavy_minus_sign:                 |                                                                                                                                                                 |
| ID de corrélation :                                    | :heavy_check_mark:                                              | :heavy_minus_sign:                 |                                                                                                                                                                                                                                                                                    |
| Modèles vocaux personnalisés                              | :heavy_minus_sign:                                              | :heavy_check_mark:                 | Le service Speech offre des modèles vocaux personnalisés qui vous permettent de personnaliser la reconnaissance vocale en fonction du vocabulaire propre à votre organisation.                                                                                                                                           |
| Modèles de traduction personnalisée                         | :heavy_minus_sign:                                              | :heavy_check_mark:                 | L’abonnement à l’API de traduction de texte Text Translation de Microsoft vous permet d’avoir recours à l’outil de traduction personnalisée [Custom Translator](https://www.microsoft.com/translator/business/customization/) afin d’obtenir des traductions plus précises en utilisant vos propres données.                                                 |

## <a name="migration-strategies"></a>Stratégies de migration

Si vous ou votre organisation disposez d’applications en développement ou en production qui utilisent l’API de traduction de conversation Translator Speech, vous devez les mettre à jour pour utiliser le service Speech. Pour accéder aux SDK, aux exemples de code et aux tutoriels disponibles, consultez la documentation du [service Speech](index.yml). Prenez en compte les éléments suivants lors de la migration :

* Le service Speech n’offre pas de point de terminaison global. Vous devez déterminer si votre application fonctionne efficacement lorsqu’elle utilise un point de terminaison régional unique pour tout son trafic. Autrement, utilisez la géolocalisation pour déterminer le point de terminaison plus efficace.

* Si votre application utilise des connexions de longue durée et ne peut pas utiliser les SDK disponibles, vous pouvez utiliser une connexion WebSockets. Gérez la limite de délai d’expiration de 10 minutes en vous reconnectant aux moments appropriés.

* Si votre application utilise le service Translator et l’API de traduction de conversation Translator Speech pour activer des modèles de traduction personnalisés, vous pouvez ajouter des ID Catégorie directement à l’aide du service Speech.

* Contrairement à l’API de traduction de conversation Translator Speech, le service Speech peut traduire plusieurs langues en une seule requête.

## <a name="next-steps"></a>Étapes suivantes

* [Essayer gratuitement le service Speech](overview.md#try-the-speech-service-for-free)
* [Démarrage rapide : Reconnaissance vocale dans une application UWP à l’aide du kit SDK Speech](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=uwp)

## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que le Service Speech](overview.md)
* [Documentation sur le SDK Speech et sur le service Speech](./speech-devices-sdk-quickstart.md?pivots=platform-android)