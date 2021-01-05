---
title: 'Fonctionnalités : Action et contexte - Personalizer'
titleSuffix: Azure Cognitive Services
description: Personalizer utilise des caractéristiques, qui sont des informations sur les actions et sur le contexte, pour faire de meilleures suggestions de classement. Les caractéristiques peuvent être très génériques ou bien spécifiques à un élément.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 10/14/2019
ms.openlocfilehash: edd1549ddabef0ae1ba37150ad75a371ac6e6d85
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94365514"
---
# <a name="features-are-information-about-actions-and-context"></a>Les caractéristiques sont des informations sur les actions et sur le contexte

Le service Personalizer fonctionne en apprenant ce que votre application devrait montrer aux utilisateurs dans un contexte donné.

Personalizer utilise des **caractéristiques** , qui sont des informations sur le **contexte actuel** pour choisir la meilleure **action**. Les caractéristiques représentent toutes les informations dont vous pensez qu’elles peuvent aider la personnalisation à obtenir des récompenses plus élevées. Les caractéristiques peuvent être très génériques ou bien spécifiques à un élément. 

Par exemple, vous pouvez avoir une **caractéristique** sur :

* Le _personnage de l’utilisateur_ , par exemple, `Sports_Shopper`. Il ne doit pas s’agir d’un ID utilisateur. 
* Le _contenu_ , par exemple si une vidéo est du type `Documentary`, `Movie` ou `TV Series`, ou si un élément en vente est disponible en magasin.
* La période de temps _actuelle_ , comme le jour de la semaine.

Personalizer n’impose pas, ne limite pas ou ne corrige pas les caractéristiques que vous pouvez envoyer pour les actions et le contexte :

* Vous pouvez envoyer certaines caractéristiques pour certaines actions, mais pas pour d’autres actions si vous ne les avez pas. Par exemple, les séries Télé peuvent avoir des attributs, mais pas les films.
* Certaines caractéristiques peuvent être disponibles seulement à certains moments. Par exemple, une application mobile peut fournir plus d’informations qu’une page web. 
* Au fil du temps, vous pouvez ajouter et supprimer des caractéristiques sur le contexte et les actions. Personalizer continue d’apprendre à partir des informations disponibles.
* Il doit y avoir au moins une caractéristique pour le contexte. Personalizer ne prend pas en charge un contexte vide. Si vous envoyez seulement un contexte fixe à chaque fois, Personalizer choisit l’action pour les classements en regardant seulement les caractéristiques dans les actions.
* Pour les fonctions catégorielles, vous n'avez pas besoin de définir les valeurs possibles ni de prédéfinir des plages pour les valeurs numériques.

## <a name="supported-feature-types"></a>Types de caractéristiques pris en charge

Personalizer prend en charge les caractéristiques de type chaîne, numérique et booléen.

### <a name="how-choice-of-feature-type-affects-machine-learning-in-personalizer"></a>Impact du choix du type de fonctionnalité sur Machine Learning dans Personalizer

* **Chaînes** : Pour les types de chaîne, chaque combinaison clé/valeur crée de nouveaux poids dans le modèle Machine Learning de Personalizer. 
* **Numérique** : Vous devez utiliser des valeurs numériques lorsque le nombre doit affecter proportionnellement le résultat de personnalisation. Cela dépend beaucoup du scénario. Dans un exemple simplifié, par exemple lors de la personnalisation d’une expérience de vente, NumberOfPetsOwned pourrait constituer une fonctionnalité numérique car vous voulez peut-être que des personnes avec 2 ou 3 animaux de compagnie puissent influencer le résultat de personnalisation deux fois ou trois fois plus que pour 1 animal de compagnie. Les fonctionnalités basées sur des unités numériques mais dont le sens n’est pas linéaire (comme l’âge, la température ou taille d’une personne) sont mieux encodées en chaînes, et la qualité d’une fonctionnalité peut généralement être améliorée à l’aide de plages. Par exemple, l’âge peut être encodé de cette façon : « Age » : « 0-5 », « Age » : « 6-10 », etc.
* Les valeurs **booléennes** envoyées avec des valeurs « false » agissent comme si elles n’avaient jamais été envoyées.

Les caractéristiques qui ne sont pas présentes doivent être omises de la demande. Évitez d’envoyer des caractéristiques avec une valeur null, car elles seront traitées comme des caractéristiques existantes et avec la valeur « null » lors de l’entraînement du modèle.

## <a name="categorize-features-with-namespaces"></a>Catégoriser les caractéristiques avec des espaces de noms

Personalizer accepte des caractéristiques organisées en espaces de noms. Vous déterminez dans votre application si des espaces de noms sont utilisés et ce qu’ils doivent être. Les espaces de noms sont utilisés pour regrouper des caractéristiques sur un sujet similaire ou qui proviennent d’une certaine source.

Voici quelques exemples d’espaces de noms de caractéristiques utilisés par des applications :

* User_Profile_from_CRM
* Temps
* Mobile_Device_Info
* http_user_agent
* VideoResolution
* UserDeviceInfo
* Météo
* Product_Recommendation_Ratings
* current_time
* NewsArticle_TextAnalytics

Vous pouvez nommer les espaces de noms de caractéristiques suivant vos propres conventions, pour autant qu’il s’agisse de clés JSON valides. Les espaces de noms sont utilisés pour organiser les fonctionnalités en ensembles distincts et pour lever l’ambiguïté entre des fonctionnalités ayant des noms similaires. Vous pouvez considérer les espaces de noms comme un « préfixe » ajouté aux noms de fonctionnalités. Les espaces de noms ne peuvent pas être imbriqués.


Dans le JSON suivant, `user`, `state` et `device` sont des espaces de noms de caractéristiques. 

> [!Note]
> Pour le moment, nous vous recommandons vivement d’utiliser des noms pour les espaces de noms de fonctionnalités qui sont basés sur UTF-8 et qui commencent par des lettres différentes. Par exemple, `user`, `state`, et `device` commencent par `u`, `s`, et `d`. Si certains espaces de noms commencent par les mêmes caractères, cela risque de provoquer des collisions dans les index utilisés pour l’apprentissage automatique.

Les objets JSON peuvent inclure des objets JSON imbriqués et de simples valeurs/propriétés. Un tableau peut être inclus uniquement si ces éléments sont des nombres. 

```JSON
{
    "contextFeatures": [
        { 
            "user": {
                "profileType":"AnonymousUser",
                "latlong": [47.6, -122.1]
            }
        },
        {
            "state": {
                "timeOfDay": "noon",
                "weather": "sunny"
            }
        },
        {
            "device": {
                "mobile":true,
                "Windows":true
            }
        }
    ]
}
```

### <a name="restrictions-in-character-sets-for-namespaces"></a>Restrictions dans les jeux de caractères pour les espaces de noms

La chaîne que vous utilisez pour nommer l’espace de noms doit respecter certaines restrictions : 
* Elle ne peut pas être unicode.
* Vous pouvez utiliser certains des symboles imprimables avec les codes < 256 pour les noms d’espaces de noms. 
* Vous ne pouvez pas utiliser de symboles avec des codes < 32 (non imprimable), 32 (espace), 58 (deux-points), 124 (canal) et 126 à 140.
* Elle ne doit pas commencer par un trait de soulignement « _ », autrement la fonctionnalité sera ignorée.

## <a name="how-to-make-feature-sets-more-effective-for-personalizer"></a>Comment rendre des ensembles de caractéristiques plus efficaces pour Personalizer

Un bon ensemble de caractéristiques aide Personalizer à prédire l’action qui va amener la récompense la plus élevée. 

Envisagez d’envoyer des caractéristiques qui suivent ces recommandations à l’API de classement (Rank) de Personalizer :

* Il existe suffisamment de caractéristiques pour piloter la personnalisation. Plus la précision avec laquelle le contenu doit être ciblé est élevée, plus grand est le nombre de caractéristiques nécessaires.

* Il existe suffisamment de caractéristiques de *densités* différentes. Une caractéristique est *dense* si beaucoup d’éléments sont regroupés dans un petit nombre de compartiments. Par exemple, des milliers de vidéos peuvent être classifiées comme étant « Longues » (d’une durée de plus de 5 minutes) et « Courtes » (d’une durée de moins de 5 minutes). Ceci est une caractéristique *très dense*. En revanche, ces mêmes milliers d’éléments peuvent avoir un attribut appelé « Titre », qui n’aura presque jamais la même valeur pour ces éléments. Il s’agit d’une caractéristique très peu dense ou *éparse*.  

Des caractéristiques de haute densité aident Personalizer à extrapoler l’apprentissage d’un élément à l’autre. Cependant, s’il n’existe que quelques caractéristiques et qu’elles sont trop denses, Personalizer va essayer de cibler de façon précise du contenu avec seulement quelques compartiments entre lesquels il doit choisir.

### <a name="improve-feature-sets"></a>Améliorer les ensembles de caractéristiques 

Analysez le comportement de l’utilisateur en effectuant une évaluation hors connexion. Ceci vous permet d’examiner les données du passé pour voir quelles caractéristiques ont fortement contribué à des récompenses positives et celles qui y ont le moins contribué. Vous pouvez voir quelles caractéristiques contribuent positivement : c’est à vous et à votre application de trouver de meilleures caractéristiques à envoyer à Personalizer pour améliorer encore les résultats.

Les sections qui suivent portent sur les pratiques courantes permettant d’améliorer les caractéristiques envoyées à Personalizer.

#### <a name="make-features-more-dense"></a>Rendre les caractéristiques plus denses

Il est possible d’améliorer vos ensembles de caractéristiques en les modifiant pour les agrandir, et pour les rendre plus ou moins denses.

Par exemple, un horodatage avec une précision inférieure à la seconde est une caractéristique très éparse. Il peut être rendu plus dense (plus efficace) en classifiant les dates/heures en « matin », « mi-journée », « après-midi », etc.

En général, la création de classifications plus larges est bénéfique pour les informations d’emplacement. Par exemple, des coordonnées de latitude et longitude telles que Lat : 47,67402° N, Long : 122,12154° O est trop précis et force le modèle à apprendre la latitude et la longitude en tant que dimensions distinctes. Lorsque vous essayez de personnaliser en fonction d’informations d’emplacement, il est utile de regrouper celles-ci dans des secteurs plus grands. Pour ce faire, une méthode aisée consiste à choisir une précision d’arrondi appropriée pour les valeurs de latitude et longitude, et de combiner celles-ci en « zones » dans une seule chaîne. Par exemple, un bon moyen de représenter 47,67402° N, long : 122,12154° O dans des régions approximatives de quelques kilomètres de large, est « Location » : « 34.3, 12,1 ».


#### <a name="expand-feature-sets-with-extrapolated-information"></a>Développer les ensembles de caractéristiques avec des informations extrapolées

Vous pouvez également obtenir plus de caractéristiques en considérant des attributs non explorés qui peuvent être dérivés d’informations que vous avez déjà. Par exemple, dans la personnalisation d’une liste de films fictive, est-il possible que le weekend et les jours de semaine suscitent un comportement différent des utilisateurs ? Les dates/heures peuvent être étendues avec un attribut « weekend » ou « semaine ». Les jours fériés nationaux entraînent-ils une attention particulière pour certains types de films ? Par exemple, un attribut « Halloween » est utile dans les endroits où il est approprié. Est-il possible qu’un temps pluvieux ait un impact significatif sur le choix d’un film pour de nombreuses personnes ? Avec le moment et le lieu, un service météo peut fournir ces informations, que vous pouvez ajouter comme caractéristique supplémentaire. 

#### <a name="expand-feature-sets-with-artificial-intelligence-and-cognitive-services"></a>Développer des ensembles de caractéristiques avec l’intelligence artificielle et des services Cognitive Services

L’intelligence artificielle et des services Cognitive Services prêts à l’emploi peuvent constituer un apport très puissant pour le Personalizer. 

En prétraitant vos éléments avec des services d’intelligence artificielle, vous pouvez extraire automatiquement des informations susceptibles d’être pertinentes pour la personnalisation.

Par exemple :

* Vous pouvez lire un fichier vidéo via [Video Indexer](https://azure.microsoft.com/services/media-services/video-indexer/) pour extraire des éléments sur les scènes, du texte, des sentiments et beaucoup d’autres attributs. Ces attributs peuvent ensuite être rendus plus denses de façon à refléter des caractéristiques que n’avaient pas les métadonnées des éléments d’origine. 
* Des images peuvent être soumises à la détection d’objets, des visages à la détection des sentiments, etc.
* Les informations présentes dans les textes peuvent être augmentées en extrayant des entités et des sentiments, en développant des entités avec le Knowledge Graph de Bing, etc.

Vous pouvez utiliser plusieurs autres services [Azure Cognitive Services](https://www.microsoft.com/cognitive-services), comme

* [Entity Linking](../text-analytics/index.yml)
* [Analyse de texte](../text-analytics/overview.md)
* [Émotion](../face/overview.md)
* [Vision par ordinateur](../computer-vision/overview.md)

## <a name="actions-represent-a-list-of-options"></a>Les actions représentent une liste d’options

Chaque action :

* A un ID d’ _événement_. Si vous disposez déjà d’un ID d’événement, vous devez l’envoyer. Si vous ne disposez pas d’un ID d’événement, il n’est pas nécessaire d’en envoyer un. Personalizer va en créer un pour vous et le retourner dans la réponse de la requête Rank. L’ID est associé à l’événement Rank, et non à l’utilisateur. Si vous devez créer un ID, il est préférable d’utiliser un GUID. 
* A une liste de caractéristiques.
* La liste des caractéristiques peut être grande (plusieurs centaines), mais nous recommandons d’évaluer l’efficacité des caractéristiques de façon à supprimer celles qui ne contribuent pas à l’obtention des récompenses. 
* Les caractéristiques des **actions** peuvent ou non avoir une corrélation avec les caractéristiques du **contexte** utilisées par Personalizer.
* Les caractéristiques pour les actions peuvent être présentes dans certaines actions et pas dans d’autres. 
* Les caractéristiques pour un ID d’action particulier peuvent être disponibles un jour, mais devenir par la suite indisponibles. 

Les algorithmes de machine learning de Personalizer fonctionnent mieux quand il existe des ensembles de caractéristiques stables, même si les appels à l’API de classement n’échouent pas si l’ensemble de caractéristiques change au fil du temps.

N’envoyez pas plus de 50 actions lors du classement des actions. Il peut s’agir chaque fois des mêmes 50 actions ou elles peuvent changer. Par exemple, si vous avez un catalogue de produits avec 10 000 éléments pour une application d’e-commerce, vous pouvez utiliser un moteur de recommandation ou de filtrage pour déterminer les 40 premiers qu’un client peut aimer, et utiliser Personalizer pour trouver celui qui génère la plus grande récompense (par exemple celui que l’utilisateur ajoute au panier) pour le contexte actuel.


### <a name="examples-of-actions"></a>Exemples d’actions

Les actions que vous envoyez à l’API de classement dépendent de ce que vous essayez de personnaliser.

Voici quelques exemples :

|Objectif|Action|
|--|--|
|Personnaliser le choix de l’article qui est mis en évidence sur un site web d’actualités.|Chaque action est un article d’actualités potentiel.|
|Optimiser le positionnement des publicités sur un site web.|Chaque action est une disposition ou des règles pour créer une disposition pour les publicités (par exemple en haut, à droite, des petites images, des grandes images).|
|Afficher un classement personnalisé d’éléments recommandés sur un site web de vente en ligne.|Chaque action est un produit spécifique.|
|Suggérer des éléments d’interface utilisateur, comme des filtres à appliquer à une photo spécifique.|Chaque action peut être un filtre différent.|
|Choisir la réponse d’un chatbot pour clarifier l’intention de l’utilisateur ou suggérer une action.|Chaque action est une possibilité d’interprétation de la réponse.|
|Choisir ce qu’il faut montrer en haut d’une liste des résultats d’une recherche|Chaque action est un des premiers résultats de la recherche.|


### <a name="examples-of-features-for-actions"></a>Exemples de caractéristiques pour les actions

Voici de bons exemples de caractéristiques pour les actions. Elles dépendent beaucoup de chaque application.

* Caractéristiques avec des attributs des actions. Par exemple, s’agit-il d’un film ou d’une série télé ?
* Caractéristiques sur la façon dont les utilisateurs peuvent avoir interagi avec cette action dans le passé. Par exemple, ce film est principalement regardé par des personnes de la catégorie démographique A ou B, et il n’est généralement pas regardé plus d’une fois.
* Caractéristiques portant sur la façon dont l’utilisateur *voit* les actions. Par exemple, l’affiche du film montrée dans la miniature inclut-elle des visages, des voitures ou des paysages ?

### <a name="load-actions-from-the-client-application"></a>Charger des actions à partir de l’application cliente

Les caractéristiques provenant d’actions peuvent en général provenir de systèmes de gestion de contenu, de catalogues et de systèmes de recommandation. Votre application est responsable du chargement des informations sur les actions à partir des bases de données et des systèmes appropriés dont vous disposez. Si vos actions ne changent pas ou que les charger chaque fois a un impact non nécessaire sur les performances, vous pouvez ajouter de la logique dans votre application pour mettre en cache ces informations.

### <a name="prevent-actions-from-being-ranked"></a>Empêcher le classement des actions

Dans certains cas, vous ne souhaitez pas montrer certaines actions aux utilisateurs. La meilleure façon d’empêcher une action d’être classée en premier est de ne pas l’inclure en premier dans la liste des actions pour l’API de classement.

Dans certains cas, le fait qu’une _action_ résultant d’un appel de l’API de classement doit ou non être montrée à un utilisateur ne peut être déterminé qu’ultérieurement dans votre logique métier. Dans ce cas, vous devez utiliser des _événements inactifs_.

## <a name="json-format-for-actions"></a>Format JSON pour les actions

Lors de l’appel de l’API de classement, vous envoyez plusieurs actions parmi lesquelles choisir :

Les objets JSON peuvent inclure des objets JSON imbriqués et de simples valeurs/propriétés. Un tableau peut être inclus uniquement si ces éléments sont des nombres. 

```json
{
    "actions": [
    {
      "id": "pasta",
      "features": [
        {
          "taste": "salty",
          "spiceLevel": "medium",
          "grams": [400,800]
        },
        {
          "nutritionLevel": 5,
          "cuisine": "italian"
        }
      ]
    },
    {
      "id": "ice cream",
      "features": [
        {
          "taste": "sweet",
          "spiceLevel": "none",
          "grams": [150, 300, 450]
        },
        {
          "nutritionalLevel": 2
        }
      ]
    },
    {
      "id": "juice",
      "features": [
        {
          "taste": "sweet",
          "spiceLevel": "none",
          "grams": [300, 600, 900]
        },
        {
          "nutritionLevel": 5
        },
        {
          "drink": true
        }
      ]
    },
    {
      "id": "salad",
      "features": [
        {
          "taste": "salty",
          "spiceLevel": "low",
          "grams": [300, 600]
        },
        {
          "nutritionLevel": 8
        }
      ]
    }
  ]
}
```

## <a name="examples-of-context-information"></a>Exemples d’informations du contexte

Les informations pour le _contexte_ varient selon chaque application et chaque cas d’utilisation, mais elles peuvent en général inclure des informations comme celles-ci :

* Informations démographiques et de profil sur votre utilisateur.
* Informations extraites à partir des en-têtes HTTP, comme l’agent utilisateur, ou dérivées des informations HTTP, comme les recherches géographiques inverses basées sur les adresses IP.
* Informations sur la date/heure actuelle, comme le jour de la semaine, weekend ou pas, matin ou après-midi, période de vacances ou pas, etc.
* Informations extraites d’applications mobiles, comme l’emplacement, le déplacement ou le niveau de la batterie.
* Agrégats de l’historique du comportement des utilisateurs, comme les genres de films qu’un utilisateur a le plus regardés.

Votre application est responsable du chargement des informations sur le contexte à partir des bases de données, des capteurs et des systèmes appropriés dont vous disposez. Si vos informations sur le contexte ne changent pas, vous pouvez ajouter de la logique dans votre application pour mettre en cache ces informations, avant de les envoyer à l’API de classement.

## <a name="json-format-for-context"></a>Format JSON pour le contexte 

Le contexte est exprimé sous la forme d’un objet JSON qui est envoyé à l’API de classement :

Les objets JSON peuvent inclure des objets JSON imbriqués et de simples valeurs/propriétés. Un tableau peut être inclus uniquement si ces éléments sont des nombres. 

```JSON
{
    "contextFeatures": [
        { 
            "user": {
                "name":"Doug"
            }
        },
        {
            "state": {
                "timeOfDay": "noon",
                "weather": "sunny"
            }
        },
        {
            "device": {
                "mobile":true,
                "Windows":true,
                "screensize": [1680,1050]
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Étapes suivantes

[Apprentissage par renforcement](concepts-reinforcement-learning.md)