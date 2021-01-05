---
title: Relations d’ancrage et orientation
description: En savoir plus sur le modèle conceptuel sous-jacent des relations entre les ancres. Apprenez à connecter des ancres dans un espace et à utiliser l’API À proximité pour un scénario de guidage.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.openlocfilehash: 8c321eae9451715077aa50d163b62b51ec88ce1a
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95496883"
---
# <a name="anchor-relationships-and-way-finding-in-azure-spatial-anchors"></a>Relations entre les ancres et guidage dans Azure Spatial Anchors

Grâce aux relations entre les ancres, vous pouvez créer des ancres connectées dans un espace et vous poser des questions telles que celles-ci :

* Existe-t-il des ancres à proximité ?
* À quelle distance se trouvent-elles ?

## <a name="examples"></a>Exemples

Vous pouvez utiliser des ancres connectées dans des cas tels que les suivants :

* Un employé doit effectuer une tâche qui implique de visiter différents sites d’une fabrique industrielle. La fabrique comprend des ancres spatiales à chaque emplacement. Une application HoloLens ou mobile permet de guider l’employé d’un emplacement vers le suivant. L’application demande tout d’abord les ancres spatiales à proximité, puis guide l’employé vers l’emplacement suivant. L’application affiche visuellement la direction générale et la distance jusqu’à l’emplacement suivant.

* Un musée crée des ancres spatiales au niveau des affiches publiques. Ensemble, ces ancres forment une visite guidée d’une heure des principales affiches publiques du musée. Sur une affiche publique, les visiteurs peuvent ouvrir l’application de réalité mixte du musée sur leur appareil mobile. Ils pointent ensuite l’appareil photo de leur téléphone autour de l’espace pour afficher la direction générale et la distance jusqu’aux autres affiches publiques de la visite guidée. Lorsqu’un utilisateur arrive devant une affiche publique, l’application met à jour la direction générale et la distance pour guider l’utilisateur.

## <a name="set-up-way-finding"></a>Configurer le guidage

Une application qui utilise une direction de ligne de vue et une distance entre des ancres pour fournir des conseils d’utilisation de *guidage*. Le guidage est différent de la navigation étape par étape. Dans la navigation étape par étape, les utilisateurs sont guidés à l’aide de murs, de portes et entre des étages. Avec le guidage, l’utilisateur dispose de conseils sur la direction générale jusqu’à la destination. L’inférence ou la connaissance de l’espace aide toutefois également l’utilisateur à parcourir la structure jusqu’à atteindre la destination.

Pour créer une expérience de guidage, préparez tout d’abord un espace pour l’expérience et développez une application avec laquelle les utilisateurs interagiront. Voici les étapes conceptuelles :

1. **Planifier l’espace** : Choisissez les emplacements dans l’espace qui seront inclus dans l’expérience de guidage. Dans nos scénarios, le superviseur de la fabrique ou le coordinateur de la visite guidée du musée peut choisir les emplacements à inclure dans l’expérience de guidage.
2. **Connecter des ancres** : Visitez les emplacements choisis pour créer des ancres spatiales. Vous pouvez le faire dans un mode d’administration de l’application de l’utilisateur final ou dans une autre application. Vous allez connecter ou lier chaque ancre aux autres. Le service gère ces relations.
3. **Démarrer l’expérience de l’utilisateur final** : Les utilisateurs exécutent l’application pour rechercher une ancre, qui peut se trouver dans l’un des emplacements choisis. Votre conception globale doit déterminer les emplacements où les utilisateurs peuvent entrer l’expérience.
4. **Rechercher des ancres à proximité** : Lorsque l’utilisateur a trouvé une ancre, l’application peut demander des ancres à proximité. Cette procédure retourne une pose entre l’appareil et ces ancres.
5. **Guider l’utilisateur** : L’application peut utiliser la pose de chacune de ces ancres pour fournir des conseils sur la direction générale de l’utilisateur et la distance. Par exemple, l’appareil photo de l’application peut afficher une icône et une flèche pour représenter chaque destination potentielle, comme le montre l’image suivante.
6. **Affiner les conseils** : Lorsque l’utilisateur se déplace, l’application peut calculer régulièrement une nouvelle pose entre l’appareil et l’ancre de destination. L’application continue à affiner les conseils qui permettent à l’utilisateur d’atteindre la destination.

    ![Exemple d’affichage des conseils de guidage d’une application](./media/meeting-spot.png)

## <a name="connect-anchors"></a>Connecter des ancres

Pour créer une expérience de guidage, vous devez d’abord placer les ancres aux emplacements choisis. Dans cette section, nous supposerons que l’administrateur de l’application est déjà terminé ce travail.

### <a name="connect-anchors-in-a-single-session"></a>Connecter des ancres dans une seule session

Pour connecter des ancres :

1. Rendez-vous au premier emplacement et créez l’ancre A à l’aide d’une CloudSpatialAnchorSession.
2. Rendez-vous au deuxième emplacement. La plateforme MR/AR sous-jacente suit le mouvement.
3. Créez une ancre B à l’aide de la même CloudSpatialAnchorSession. Les ancres A et B sont maintenant connectées. Le service Spatial Anchors gère cette relation.
4. Continuez la procédure pour les ancres restantes.

### <a name="connect-anchors-in-multiple-sessions"></a>Connecter des ancres dans plusieurs sessions

Vous pouvez connecter des ancres spatiales via plusieurs sessions. Grâce à cette méthode, vous pouvez créer et connecter quelques ancres en même temps puis créer et connecter d’autres ancres ultérieurement.

Pour connecter des ancres via plusieurs sessions :

1. L’application crée des ancres dans une CloudSpatialAnchorSession.
2. À un autre moment, l’application localise une de ces ancres (par exemple, l’ancre A) à l’aide d’une nouvelle CloudSpatialAnchorSession.
3. Rendez-vous à un nouvel emplacement. La plateforme de réalité mixte ou réalité augmentée sous-jacente suit le mouvement.
4. Créez une ancre C à l’aide de la même CloudSpatialAnchorSession. Les ancres A, B et C sont maintenant connectées. Le service Spatial Anchors gère cette relation.

Vous pouvez continuer cette procédure pour d’autres ancres et sessions au fil du temps.

### <a name="verify-anchor-connections"></a>Vérifier les connexions d’ancres

L’application peut vérifier que deux ancres sont connectées en émettant une requête pour les ancres avoisinantes, ce que vous faites en définissant le `NearAnchorCriteria` sur un `CloudSpatialAnchorWatcher`. Lorsque le résultat de la requête contient l’ancre cible, la connexion d’ancre est vérifiée. Si les ancres ne sont pas connectées, l’application peut essayer de les reconnecter.

Voici quelques raisons pour lesquelles la connexion d’ancres peut échouer :

* La plateforme de réalité mixte ou réalité augmentée sous-jacente perd le suivi pendant le processus de connexion des ancres.
* En raison d’une erreur réseau pendant la communication avec le service Spatial Anchors, la connexion d’ancre n’a pas pu être maintenue.

### <a name="find-sample-code"></a>Rechercher un exemple de code

Pour obtenir un exemple de code qui montre comment connecter des ancres et effectuer des requêtes de proximité, consultez les [exemples d’applications Spatial Anchors](https://github.com/Azure/azure-spatial-anchors-samples).
