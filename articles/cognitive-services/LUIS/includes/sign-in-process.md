---
title: Fichier include
description: Fichier include
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 10/30/2020
ms.topic: include
ms.openlocfilehash: e006f804b8ab6411f4949424147acf567dc2ed24
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97371307"
---
## <a name="sign-in-to-luis-portal"></a>Se connecter au portail LUIS

[!INCLUDE [Note about portal deprecation](luis-portal-note.md)]

Un nouvel utilisateur de LUIS doit suivre cette procédure :

1. Connectez-vous au [portail LUIS](https://www.luis.ai), sélectionnez votre pays/région, puis acceptez les conditions d’utilisation. Si vous voyez à la place **Mes applications**, c’est qu’une ressource LUIS existe déjà et vous devez passer directement à la création d’une application. Vous avez deux options pour vous inscrire :

    * Utilisation d’une ressource Azure (recommandé) – vous permet de lier votre compte LUIS à une ressource de création Azure nouvelle ou existante. Cela équivaut à une inscription déjà migrée. Vous n’aurez pas besoin de passer ultérieurement par le [processus de migration](../luis-migration-authoring.md#what-is-migration).

    * Utilisation d’une clé d’évaluation. Cela vous permet de vous connecter à LUIS avec une ressource d’évaluation que vous n’avez pas besoin de configurer. Si vous choisissez cette option, vous devrez plus tard [migrer votre compte](../luis-migration-authoring.md#migration-steps) et lier vos applications à une ressource de création.

1. Dans la fenêtre **Choisir une ressource de création** qui s’affiche, recherchez votre abonnement Azure et la ressource de création LUIS. Si vous n’avez pas de ressource, vous pouvez en créer une nouvelle.

    :::image type="content" source="../media/luis-how-to-azure-subscription/choose-authoring-resource.png" alt-text="Choisissez un type de ressource de création LUIS.":::
    
    Quand vous créez une ressource de création, fournissez les informations suivantes :
    * **Nom du locataire** – locataire auquel votre abonnement Azure est associé.
    * **Nom de l’abonnement Azure** – abonnement auquel la ressource sera facturée.
    * **Nom du groupe de ressources Azure** – nom de groupe de ressources personnalisé que vous choisissez ou créez. Les groupes de ressources vous permettent de regrouper des ressources Azure pour l’accès et la gestion.
    * **Nom de la ressource Azure** – nom personnalisé que vous choisissez, utilisé comme élément de l’URL pour vos requêtes de point de terminaison de création et de prédiction.
    * **Pricing tier** (Niveau tarifaire) : le niveau tarifaire détermine la transaction maximale par seconde et par mois.


