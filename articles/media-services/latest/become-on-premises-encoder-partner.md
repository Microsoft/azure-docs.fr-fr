---
title: Devenir partenaire d’encodeur local - Azure Media Services
description: Cet article explique comment vérifier vos encodeurs de streaming en direct locaux.
services: media-services
author: johndeu
manager: johndeu
ms.author: johndeu
ms.date: 08/31/2020
ms.topic: how-to
ms.service: media-services
ms.openlocfilehash: 1383b6afac9ad2332169328ef71981e23e86bdcc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89297448"
---
# <a name="how-to-verify-your-on-premises-live-streaming-encoder"></a>Guide pratique pour vérifier votre encodeur de streaming en direct local

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En tant que partenaire d’encodeur local d’Azure Media Services, Media Services promeut votre produit en recommandant votre encodeur aux clients d’entreprise. Pour devenir un partenaire d’encodeur local, vous devez vérifier la compatibilité de votre encodeur local avec Media Services. Pour ce faire, effectuez les vérifications suivantes.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="pass-through-live-event-verification"></a>Vérification d'un Événement en temps réel de type pass-through

1. Accédez à votre compte Media Services et vérifiez que le **point de terminaison de streaming** est en cours d’exécution. 
2. Créez et démarrez un Événement en temps réel de type **pass-through**. <br/> Pour plus d’informations, consultez [États et facturation des événements en direct](live-event-states-billing.md).
3. Récupérez les URL d’ingestion et configurez votre encodeur local de façon à ce qu’il utilise ces URL pour envoyer un flux temps réel multidébit à Media Services.
4. Récupérez l’URL d’aperçu et utilisez-la pour vérifier que l’entrée de l’encodeur est bien reçue.
5. Créez un objet **Asset**.
6. Créez un objet **LiveOutput** et utilisez le nom de l’objet Asset que vous venez de créer.
7. Créez un **localisateur de streaming**  avec les types intégrés de la **stratégie de streaming**.
8. Listez les chemins d'accès dans le **Localisateur de streaming** pour récupérer les URL à utiliser.
9. Récupérez le nom d’hôte du **Point de terminaison de streaming** à partir duquel vous souhaitez effectuer le streaming.
10. Combinez l’URL de l’étape 8 avec le nom d’hôte de l’étape 9 pour obtenir l’URL complète.
11. Exécutez votre encodeur live pendant environ 10 minutes.
12. Arrêtez l’événement en direct. 
13. Utilisez un lecteur, par exemple [Lecteur multimédia Azure](https://aka.ms/azuremediaplayer) pour regarder l’élément multimédia archivé afin de vous assurer que la lecture est dépourvue de problèmes visibles à tous les niveaux de qualité. Vous pouvez également regarder et valider via l’URL de l’aperçu pendant la session active.
14. Enregistrez l’ID de la ressource, l’URL de streaming publié pour l’archive en temps réel, ainsi que les paramètres et la version utilisée à partir de votre encodeur live.
15. Réinitialisez l'état de l'Événement en temps réel après la création de chaque exemple.
16. Répétez les étapes 5 à 15 pour toutes les configurations prises en charge par votre encodeur (avec et sans signalisation des annonces, légendes ou vitesses d’encodage différentes).

## <a name="live-encoding-live-event-verification"></a>Vérification d'Événement en temps réel d'encodage en direct

1. Accédez à votre compte Media Services et vérifiez que le **point de terminaison de streaming** est en cours d’exécution. 
2. Créez et démarrez un Événement en temps réel d'**encodage en direct**. <br/> Pour plus d’informations, consultez [États et facturation des événements en direct](live-event-states-billing.md).
3. Récupérez les URL d’ingestion et configurez votre encodeur de façon à ce qu’il envoie (push) un flux temps réel à débit binaire à Media Services.
4. Récupérez l’URL d’aperçu et utilisez-la pour vérifier que l’entrée de l’encodeur est bien reçue.
5. Créez un objet **Asset**.
6. Créez un objet **LiveOutput** et utilisez le nom de l’objet Asset que vous venez de créer.
7. Créez un **localisateur de streaming**  avec les types intégrés de la **stratégie de streaming**.
8. Listez les chemins d'accès dans le **Localisateur de streaming** pour récupérer les URL à utiliser.
9. Récupérez le nom d’hôte du **Point de terminaison de streaming** à partir duquel vous souhaitez effectuer le streaming.
10. Combinez l’URL de l’étape 8 avec le nom d’hôte de l’étape 9 pour obtenir l’URL complète.
11. Exécutez votre encodeur live pendant environ 10 minutes.
12. Arrêtez l’événement en direct.
13. Utilisez un lecteur, par exemple [Lecteur multimédia Azure](https://aka.ms/azuremediaplayer) pour regarder l’élément multimédia archivé afin de vous assurer que la lecture est dépourvue de problèmes visibles à tous les niveaux de qualité. Vous pouvez également regarder et valider via l’URL de l’aperçu pendant la session active.
14. Enregistrez l’ID de la ressource, l’URL de streaming publié pour l’archive en temps réel, ainsi que les paramètres et la version utilisée à partir de votre encodeur live.
15. Réinitialisez l'état de l'Événement en temps réel après la création de chaque exemple.
16. Répétez les étapes 5 à 15 pour toutes les configurations prises en charge par votre encodeur (avec et sans signalisation des annonces, légendes ou vitesses d’encodage différentes).

## <a name="longevity-verification"></a>Vérification de longévité

Suivez les mêmes étapes que pour la [Vérification d’un Événement en temps réel de type pass-through](#pass-through-live-event-verification), à l’exception de l’étape 11. <br/>Au lieu de 10 minutes, exécutez votre encodeur live pendant une semaine ou plus. Utilisez un lecteur, par exemple [Lecteur multimédia Azure](https://aka.ms/azuremediaplayer) pour regarder de temps en temps le streaming en direct (ou un élément multimédia archivé) afin de vous assurer que la lecture est dépourvue de problèmes visibles.

## <a name="email-your-recorded-settings"></a>Envoi par e-mail de vos paramètres enregistrés

Enfin, envoyez par e-mail vos paramètres enregistrés et paramètres d’archivage en direct à Azure Media Services à l’adresse amshelp@microsoft.com en guise de notification indiquant que tous les contrôles de vérification automatique ont réussi. Incluez également vos informations de contact à des fins de suivi. Vous pouvez contacter l’équipe Azure Media Services pour toute question sur ce processus.

## <a name="see-also"></a>Voir aussi

[Tester des encodeurs locaux](recommended-on-premises-live-encoders.md)

## <a name="next-steps"></a>Étapes suivantes

[Streaming en direct avec Media Services v3](live-streaming-overview.md)
