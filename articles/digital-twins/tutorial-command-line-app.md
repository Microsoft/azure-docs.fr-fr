---
title: 'Tutoriel : Explorer les bases avec un exemple d’application cliente'
titleSuffix: Azure Digital Twins
description: Tutoriel permettant d’explorer les kits SDK Azure Digital Twins à l’aide d’un exemple d’application en ligne de commande.
author: baanders
ms.author: baanders
ms.date: 5/8/2020
ms.topic: tutorial
ms.service: digital-twins
ms.openlocfilehash: a9d8c42432d619ab120afd0bd5734f6fa269b514
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458098"
---
# <a name="tutorial-explore-azure-digital-twins-with-a-sample-client-app"></a>Tutoriel : Explorer Azure Digital Twins avec un exemple d’application cliente

Ce tutoriel présente un exemple d’application qui implémente une application cliente en ligne de commande, pour interagir avec une instance Azure Digital Twins. L’application cliente est semblable à celle écrite dans [*Tutoriel : Coder une application cliente*](tutorial-code.md).

Vous pouvez utiliser cet exemple pour effectuer des actions Azure Digital Twins essentielles, telles que le chargement de modèles, la création et la modification de jumeaux et la création de relations. Vous pouvez également examiner le code de l’exemple pour en savoir plus sur les API Azure Digital Twins et vous exercer à implémenter vos propres commandes en modifiant l’exemple de projet comme bon vous semble.

Dans ce tutoriel, vous allez :
> [!div class="checklist"]
> * Configurer une instance Azure Digital Twins.
> * Configurer l’exemple d’application en ligne de commande pour interagir avec l’instance.
> * Utiliser l’application en ligne de commande pour explorer Azure Digital Twins, notamment les **modèles**, les **jumeaux numériques**, les **relations** et les **requêtes**.

[!INCLUDE [Azure Digital Twins tutorial: sample prerequisites](../../includes/digital-twins-tutorial-sample-prereqs.md)]

[!INCLUDE [Azure Digital Twins tutorial: configure the sample project](../../includes/digital-twins-tutorial-sample-configure.md)]

## <a name="explore-with-the-sample-solution"></a>Explorer avec l’exemple de solution

Maintenant que l’instance et l’exemple d’application sont configurés, vous allez utiliser l’exemple de projet et un exemple de code pré-écrit pour générer et explorer une solution Azure Digital Twins de base. Les principaux composants de la solution sont les **modèles** , les **jumeaux numériques** et les **relations**, ce qui se traduit par un **graphe de jumeaux** d’environnement interrogeable.

### <a name="model-a-physical-environment-with-dtdl"></a>Modéliser un environnement physique avec DTDL

La première étape de la création d’une solution Azure Digital Twins consiste à définir des [**modèles**](concepts-models.md) de jumeaux pour votre environnement. 

Les modèles sont similaires aux classes dans les langages de programmation orientés objet. Ils fournissent des modèles de [jumeaux numériques](concepts-twins-graph.md) définis par l’utilisateur à suivre et à instancier ultérieurement. Ils sont écrits dans un langage de type JSON appelé **DTDL (Digital Twins Definition Language**) et peuvent définir les *propriétés*, la *télémétrie*, les *relations* et les *composants* d’un jumeau.

> [!NOTE]
> DTDL permet également de définir des *commandes* sur des jumeaux numériques. Toutefois, les commandes ne sont actuellement pas prises en charge dans le service Azure Digital Twins.

Dans la fenêtre Visual Studio où le projet _**AdtE2ESample**_ est ouvert, utilisez le volet *Explorateur de solutions* pour accéder au dossier *AdtSampleApp\SampleClientApp\Models*. Ce dossier contient des exemples de modèles.

Sélectionnez *Room.json* pour l’ouvrir dans la fenêtre d’édition, puis modifiez-le comme suit :

1. **Mettez à jour le numéro de version** pour indiquer que vous fournissez une version mise à jour de ce modèle. Pour ce faire, remplacez le *1* à la fin de la valeur `@id` par un *2*. Tout nombre supérieur au numéro de version actuel convient aussi.
1. **Modifiez une propriété**. Remplacez le nom de la propriété `Humidity` par *HumidityLevel* (ou une autre valeur si vous le souhaitez. Si vous utilisez autre chose que *HumidityLevel*, souvenez-vous de ce que vous avez utilisé et continuez à l’utiliser à la place de *HumidityLevel* tout au long du tutoriel).
1. **Ajoutez une propriété**. Sous la propriété `HumidityLevel` qui se termine à la ligne 15, collez le code suivant pour ajouter une propriété `RoomName` à la pièce :

    ```json
    ,
    {
      "@type": "Property",
      "name": "RoomName",
      "schema": "string"
    }
    ```
1. **Ajoutez une relation**. Sous la propriété `RoomName` que vous venez d’ajouter, collez le code suivant pour permettre à ce type de jumeau de former des relations *contains* avec d’autres jumeaux :

    ```json
    ,
    {
      "@type": "Relationship",
      "name": "contains"
    }
    ```

Quand vous avez terminé, le modèle mis à jour doit ressembler à ceci :

:::image type="content" source="media/tutorial-command-line-app/room-model.png" alt-text="Room.json modifié avec le numéro de version mis à jour, les propriétés HumidityLevel et RoomName et la relation contains" border="false":::

N’oubliez pas d’enregistrer le fichier avant de continuer.

> [!TIP]
> Si vous souhaitez essayer de créer votre propre modèle, vous pouvez coller le code du modèle *Room* dans un nouveau fichier que vous enregistrez avec une extension *.json* dans le dossier *AdtSampleApp\SampleClientApp\Models*. Ensuite, ajoutez différentes propriétés et relations pour représenter ce que vous souhaitez. Vous pouvez également consulter les autres exemples de modèles dans ce dossier pour obtenir des idées.

> [!TIP] 
> Il existe un [exemple de validateur DTDL](/samples/azure-samples/dtdl-validator/dtdl-validator) indépendant du langage qui vous permet de vérifier les documents de modèle et la validité du DTDL. Il repose sur la bibliothèque de l’analyseur DTDL, qui est décrite en détail dans [*Guide pratique pour analyser et valider les modèles*](how-to-parse-models.md).

### <a name="get-started-with-the-command-line-app"></a>Démarrer avec l’application en ligne de commande

Maintenant que vous avez défini un modèle, lors des étapes restantes vous allez utiliser l’exemple d’application pour interagir avec votre instance Azure Digital Twins. Exécutez le projet avec ce bouton dans la barre d’outils :

:::image type="content" source="media/tutorial-command-line-app/start-button-sample.png" alt-text="Bouton de démarrage dans Visual Studio (projet SampleClientApp)":::

Une fenêtre de console s’ouvre, exécute l’authentification et attend une commande. 
* L’authentification est gérée par le biais du navigateur : votre navigateur web par défaut s’ouvre avec une invite d’authentification. Utilisez cette invite pour vous connecter avec vos informations d’identification Azure. Vous pouvez ensuite fermer l’onglet ou la fenêtre du navigateur.

Voici une capture d’écran montrant à quoi ressemble la console projet :

:::image type="content" source="media/tutorial-command-line-app/command-line-app.png" alt-text="Message d’accueil de l’application en ligne de commande":::

> [!TIP]
> Pour obtenir la liste de toutes les commandes que vous pouvez utiliser avec ce projet, entrez `help` dans la console de projet et appuyez sur Entrée.
> :::image type="content" source="media/tutorial-command-line-app/command-line-app-help.png" alt-text="Sortie de la commande d’aide":::

Laissez la console de projet en cours d’exécution pour le reste des étapes de ce tutoriel.

#### <a name="upload-models-to-azure-digital-twins"></a>Charger des modèles sur Azure Digital Twins

Après avoir conçu les modèles, vous devez les charger sur votre instance Azure Digital Twins. Cela configure votre instance du service Azure Digital Twins avec votre propre vocabulaire de domaine personnalisé. Une fois que vous avez chargé les modèles, vous pouvez créer des instances de jumeau qui les utilisent.

Dans la fenêtre de console du projet, exécutez la commande suivante pour charger votre modèle *Room* mis à jour, ainsi qu’un modèle *Floor* que vous utiliserez aussi dans la section suivante pour créer différents types de jumeaux.

```cmd/sh
CreateModels Room Floor
```

La sortie doit indiquer que les modèles ont été créés avec succès.

> [!TIP]
> Si vous avez créé votre propre modèle, vous pouvez également le charger ici, en ajoutant son nom de fichier (vous pouvez ignorer l’extension) à la liste `Room Floor` dans la commande ci-dessus.

Vérifiez que les modèles ont été créés en exécutant la commande `GetModels true`. Cette opération interroge l’instance Azure Digital Twins pour identifier tous les modèles qui ont été chargés, puis affiche leurs informations complètes. Recherchez le modèle *Room* modifié dans les résultats :

:::image type="content" source="media/tutorial-command-line-app/output-get-models.png" alt-text="Résultats de GetModels présentant le modèle Room mis à jour":::

#### <a name="errors"></a>Erreurs

L’exemple d’application gère également les erreurs du service. 

Réexécutez la commande `CreateModels` pour essayer de recharger l’un des modèles que vous venez de charger :

```cmd/sh
CreateModels Room
```

Comme les modèles ne peuvent pas être remplacés, une erreur de service est désormais retournée.
Pour plus d’informations sur la façon de supprimer des modèles existants, consultez [*Guide pratique : Gérer des modèles personnalisés*](how-to-manage-model.md).
```cmd/sh
Response 409: Service request failed.
Status: 409 (Conflict)

Content:
{"error":{"code":"ModelAlreadyExists","message":"Could not add model dtmi:example:Room;2 as it already exists. Use Model_List API to view models that already exist. See the Swagger example.(http://aka.ms/ModelListSwSmpl)"}}

Headers:
Strict-Transport-Security: REDACTED
Date: Wed, 20 May 2020 00:53:49 GMT
Content-Length: 223
Content-Type: application/json; charset=utf-8
```

### <a name="create-digital-twins"></a>Créer des jumeaux numériques

Maintenant que certains modèles ont été chargés sur votre instance Azure Digital Twins, vous pouvez créer des [**jumeaux numériques**](concepts-twins-graph.md) basés sur les définitions de modèle. Les jumeaux numériques représentent les entités au sein de votre environnement d’entreprise (par exemple les capteurs dans une ferme, les salles d’un bâtiment ou les voyants d’une voiture). 

Pour créer un jumeau numérique, utilisez la commande `CreateDigitalTwin`. Vous devez référencer le modèle sur lequel le jumeau est basé, et vous pouvez éventuellement définir des valeurs initiales pour les propriétés du modèle. Vous n’avez pas besoin de transmettre d’informations de relation à ce stade.

Exécutez ce code dans la console de projet en cours d’exécution pour créer plusieurs jumeaux, basés sur le modèle *Room* que vous avez mis à jour et sur un autre modèle, *Floor*. Rappelez-vous que *Room* a trois propriétés ; vous pouvez donc fournir des arguments avec les valeurs initiales de ces propriétés.

```cmd/sh
CreateDigitalTwin dtmi:example:Room;2 room0 RoomName string Room0 Temperature double 70 HumidityLevel double 30
CreateDigitalTwin dtmi:example:Room;2 room1 RoomName string Room1 Temperature double 80 HumidityLevel double 60
CreateDigitalTwin dtmi:example:Floor;1 floor0
CreateDigitalTwin dtmi:example:Floor;1 floor1
```

> [!TIP]
> Si vous avez chargé votre propre modèle, essayez de créer votre propre commande `CreateDigitalTwin` basée sur les commandes ci-dessus pour ajouter un jumeau de votre propre type de modèle.

La sortie de ces commandes doit indiquer que les jumeaux ont été créés avec succès. 

:::image type="content" source="media/tutorial-command-line-app/output-create-digital-twin.png" alt-text="Extrait des résultats des commandes CreateDigitalTwin, montrant floor0, floor1, room0 et room1":::

Vous pouvez aussi vérifier que les jumeaux ont été créés en exécutant la commande `Query`. Cette commande interroge votre instance Azure Digital Twins pour obtenir tous les jumeaux numériques qu’elle contient. Recherchez les jumeaux *floor0*, *floor1*, *room0* et *room1* dans les résultats.

#### <a name="modify-a-digital-twin"></a>Modifier un jumeau numérique

Vous pouvez également modifier les propriétés d’un jumeau que vous avez créé. Essayez d’exécuter cette commande pour changer le RoomName de *room0* de *Room0* en *PresidentialSuite* :

```cmd/sh
UpdateDigitalTwin room0 add /RoomName string PresidentialSuite
```

La sortie doit indiquer que le jumeau a été correctement mis à jour.

Vous pouvez également vérifier en exécutant cette commande pour voir les informations de *room0* :

```cmd/sh
GetDigitalTwin room0
```

La sortie doit refléter le nom mis à jour.

> [!NOTE]
> L’API REST sous-jacente utilise JSON Patch pour définir les mises à jour d’un jumeau. L’application en ligne de commande reflète ce format ; vous pouvez donc faire des essais avec ce que les API sous-jacentes attendent réellement.

### <a name="create-a-graph-by-adding-relationships"></a>Créer un graphe en ajoutant des relations

Ensuite, vous pouvez créer des **relations** entre ces jumeaux, afin de les raccorder sur un [**graphe de jumeaux**](concepts-twins-graph.md). Les graphes de jumeaux servent à représenter un environnement entier. 

Pour ajouter une relation, utilisez la commande `CreateRelationship`. Spécifiez le jumeau d’où provient la relation, le type de relation à ajouter et le jumeau avec lequel la relation établit une connexion. Pour finir, spécifiez un nom (ID) pour la relation.

Exécutez le code suivant pour ajouter une relation « contains » entre chaque jumeau *Floor* que vous avez créé et un jumeau *Room* correspondant. Notez que pour que cela soit possible, une relation *contains* doit être définie sur le modèle *Floor*.

```cmd/sh
CreateRelationship floor0 contains room0 relationship0
CreateRelationship floor1 contains room1 relationship1
```

La sortie de ces commandes confirme que les relations ont été créées correctement :

:::image type="content" source="media/tutorial-command-line-app/output-create-relationship.png" alt-text="Extrait des résultats des commandes CreateRelationship, montrant relationship0 et relationship1":::

Vous pouvez également vérifier les relations avec l’une des commandes suivantes, qui interrogent les relations dans votre instance Azure Digital Twins.
* Pour voir toutes les relations en provenance de chaque étage (en affichant les relations d’un côté) :
    ```cmd/sh
    GetRelationships floor0
    GetRelationships floor1
    ```
* Pour voir toutes les relations arrivant à chaque pièce (en affichant la relation de l’« autre » côté) :
    ```cmd/sh
    GetIncomingRelationships room0
    ```
* Pour interroger ces relations individuellement : 
    ```cmd/sh
    GetRelationship floor0 relationship0
    GetRelationship floor1 relationship1
    ```

Les jumeaux et les relations que vous avez configurés dans ce tutoriel forment le graphe conceptuel suivant :

:::image type="content" source="media/tutorial-command-line-app/sample-graph.png" alt-text="Graphe montrant floor0 raccordé par le biais de relationship0 à room0, et floor1 raccordé par le biais de relationship1 à room1" border="false":::

### <a name="query-the-twin-graph-to-answer-environment-questions"></a>Interroger le graphe de jumeaux pour répondre à des questions environnementales

L’une des principales fonctionnalités d’Azure Digital Twins est la capacité à [interroger facilement et efficacement](concepts-query-language.md) votre graphe de jumeaux pour répondre à des questions sur votre environnement. Exécutez les commandes suivantes dans la console de projet en cours d’exécution pour avoir une idée de ce à quoi cela ressemble.

* **Quelles sont les entités de mon environnement représentées dans Azure Digital Twins ?** (interroger tout)

    ```cmd/sh
    Query
    ```

    Cela vous permet d’examiner votre environnement en un coup d’œil, et de vous assurer que tout est représenté comme vous le souhaitez dans Azure Digital Twins. Le résultat est une sortie contenant chaque jumeau numérique avec ses détails. Voici un extrait :

    :::image type="content" source="media/tutorial-command-line-app/output-query-all.png" alt-text="Résultats partiels de requête de jumeaux, avec room0 et floor1":::

    >[!NOTE]
    >Dans l’exemple de projet, la commande `Query` sans argument supplémentaire équivaut à `Query SELECT * FROM DIGITALTWINS`. Pour interroger tous les jumeaux dans votre instance à l’aide des [API de requête](/rest/api/digital-twins/dataplane/query) ou des [commandes CLI](how-to-use-cli.md), utilisez la requête la plus longue (complète).

* **Quelles sont les pièces dans mon environnement ?** (requête par modèle)

    ```cmd/sh
    Query SELECT * FROM DIGITALTWINS T WHERE IS_OF_MODEL(T, 'dtmi:example:Room;2')
    ```

    Vous pouvez limiter votre requête aux jumeaux d’un certain type, afin d’obtenir des informations plus spécifiques sur ce qui est représenté. Le résultat montre *room0* et *room1*, mais ne montre **pas** *floor0* ni *floor1* (car il s’agit d’étages, et non de salles).
    
    :::image type="content" source="media/tutorial-command-line-app/output-query-model.png" alt-text="Résultats de requête de modèle, montrant uniquement room0 et room1":::

* **Quelles sont les pièces de *floor0* ?** (requête par relation)

    ```cmd/sh
    Query SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.$dtId = 'floor0'
    ```

    Vous pouvez interroger en fonction des relations de votre graphe, afin d’obtenir des informations sur la façon dont les jumeaux sont raccordés ou de limiter votre requête à une certaine zone. Seule *room0* se trouve à l’étage *floor0* ; il s’agit donc de la seule pièce dans le résultat.

    :::image type="content" source="media/tutorial-command-line-app/output-query-relationship.png" alt-text="Résultats de requête de relation, avec room0":::

* **Quels sont les jumeaux dans mon environnement dont la température est supérieure à 75 ?** (requête par propriété)

    ```cmd/sh
    Query SELECT * FROM DigitalTwins T WHERE T.Temperature > 75
    ```

    Vous pouvez interroger le graphe en fonction de propriétés afin de répondre à diverses questions, notamment pour rechercher les anomalies dans votre environnement qui peuvent nécessiter votre attention. D’autres opérateurs de comparaison ( *<* , *>* , *=* ou *!=* ) sont également pris en charge. *room1* apparaît ici dans les résultats, car elle a une température de 80.

    :::image type="content" source="media/tutorial-command-line-app/output-query-property.png" alt-text="Résultats de requête de propriété, montrant uniquement room1":::

* **Quelles sont les pièces de l’étage *floor0* dont la température est supérieure à 75 ?** (requête composée)

    ```cmd/sh
    Query SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.$dtId = 'floor0' AND IS_OF_MODEL(room, 'dtmi:example:Room;2') AND room.Temperature > 75
    ```

    Vous pouvez également combiner les requêtes précédentes comme vous le feriez dans SQL, à l’aide d’opérateurs de combinaison tels que `AND`, `OR` et `NOT`. Cette requête utilise `AND` pour rendre plus spécifique la requête précédente sur les températures des jumeaux. Le résultat contient désormais uniquement les pièces dont la température est supérieure à 75 et qui se trouvent à l’étage *floor0* (en l’occurrence, aucune). Le jeu de résultat est vide.

    :::image type="content" source="media/tutorial-command-line-app/output-query-compound.png" alt-text="Résultats de requête composée, sans résultat":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Le projet de ce tutoriel forme la base du tutoriel suivant, intitulé [*Tutoriel : Connecter une solution de bout en bout*](tutorial-end-to-end.md). Si vous envisagez de continuer avec le tutoriel suivant, vous pouvez conserver les ressources configurées ici pour continuer à utiliser cette instance Azure Digital Twins et l’exemple d’application configurée.
* Dans ce cas, vous pouvez utiliser les commandes `DeleteAllTwins` et `DeleteAllModels` de l’exemple d’application pour effacer respectivement les jumeaux et les modèles de votre instance. Vous repartirez alors sur de nouvelles bases pour le tutoriel suivant.

[!INCLUDE [digital-twins-cleanup-basic.md](../../includes/digital-twins-cleanup-basic.md)]

Pour finir, supprimez le dossier d’exemple de projet que vous avez téléchargé sur votre ordinateur local.

## <a name="next-steps"></a>Étapes suivantes 

Dans ce tutoriel, vous avez commencé à utiliser Azure Digital Twins en configurant une instance et une application cliente pour interagir avec l’instance. Vous avez utilisé l’application cliente pour explorer Azure Digital Twins, et vous avez créé des modèles, des jumeaux numériques et des relations. Vous avez également exécuté des requêtes sur la solution, pour vous faire une idée des types de questions auxquelles Azure Digital Twins peut répondre sur un environnement.

Passez au tutoriel suivant pour utiliser l’exemple d’application en ligne de commande conjointement avec d’autres services Azure afin de bénéficier d’un scénario de bout en bout piloté par les données :
> [!div class="nextstepaction"]
> [*Tutoriel : Connecter une solution de bout en bout*](tutorial-end-to-end.md)