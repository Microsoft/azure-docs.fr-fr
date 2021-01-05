---
title: Mode Apprenti - Personalizer
description: ''
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 05/01/2020
ms.openlocfilehash: f793535f04b36d231cec384b7acd66e38a7eb039
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91253528"
---
# <a name="use-apprentice-mode-to-train-personalizer-without-affecting-your-existing-application"></a>Utilisez le mode Apprenti pour effectuer l'apprentissage de Personalizer sans affecter l'application existante

En raison de la nature de l'apprentissage par renforcement **réel**, un modèle Personalizer ne peut faire l'objet d'un apprentissage que dans un environnement de production. Lors du déploiement d'un nouveau cas d'usage, le modèle Personalizer n'est pas efficace tout de suite car son apprentissage prend du temps.  Le **mode Apprenti** est un comportement d'apprentissage qui facilite ce processus et vous permet de vous appuyer sur le modèle - sans modification du code de la part du développeur.

[!INCLUDE [Important Blue Box - Apprentice mode pricing tier](./includes/important-apprentice-mode.md)]

## <a name="what-is-apprentice-mode"></a>Qu'est-ce que le mode Apprenti ?

Comme un apprenti qui apprend de son maître et s'améliore à mesure qu'il acquiert de l'expérience, le mode Apprenti est un _comportement_ qui permet à Personalizer d'apprendre en observant les résultats obtenus à partir de la logique d'application existante.

Personalizer se forme en imitant l'application. Au fil des événements, Personalizer peut _rattraper_ l'application existante sans affecter la logique et les résultats existants. Les mesures, disponibles à partir du portail Azure et de l'API, vous aident à comprendre les performances au fil de l'apprentissage du modèle.

Une fois que Personalizer a atteint un certain niveau de compréhension, le développeur peut modifier le comportement d'apprentissage en passant du mode Apprenti au mode En ligne. Dès lors, Personalizer commence à influencer les actions dans l'API Rank.

## <a name="purpose-of-apprentice-mode"></a>Objectif du mode Apprenti

Le mode Apprenti vous permet de vous appuyer sur le service Personalizer et ses fonctionnalités d'apprentissage automatique. Il fournit également l'assurance que le service reçoit des informations dont il peut tirer parti dans le cadre de son apprentissage - sans mettre en péril le trafic en ligne.

Les deux principales raisons d'utiliser le mode Apprenti sont les suivantes :

* Atténuation des **démarrages à froid** : Le mode Apprenti permet de gérer et d’évaluer le coût du temps d’apprentissage d’un « nouveau » modèle – lorsqu’il ne retourne pas la meilleure action et n’atteint pas un niveau d’efficacité satisfaisant de 60 à 80 %.
* **Validation des caractéristiques d'action et de contexte** : Les caractéristiques liées aux actions et au contexte peuvent être inadéquates ou inexactes - insuffisantes, trop nombreuses, incorrectes ou trop spécifiques pour effectuer l'apprentissage de Personalizer en vue d'atteindre le taux d'efficacité idéal. Utilisez les [évaluations des caractéristiques](concept-feature-evaluation.md) pour rechercher et résoudre les problèmes liés aux caractéristiques.

## <a name="when-should-you-use-apprentice-mode"></a>Quand devez-vous utiliser le mode Apprenti ?

Utilisez le mode Apprenti pour effectuer l'apprentissage de Personalizer afin d'améliorer son efficacité lors des scénarios suivants sans altérer l'expérience de vos utilisateurs :

* Vous implémentez Personalizer dans un nouveau cas d'usage.
* Vous avez significativement modifié les caractéristiques liées au contexte ou aux actions.
* Vous avez significativement modifié le moment et la manière dont vous calculez les récompenses.

Le mode Apprenti n'est pas efficace pour mesurer l'impact de Personalizer sur les scores de récompense. Pour mesurer l'efficacité de Personalizer en termes de choix de la meilleure action possible pour chaque appel de l'API Rank, utilisez [Évaluations hors connexion](concepts-offline-evaluation.md).

## <a name="who-should-use-apprentice-mode"></a>Qui doit utiliser le mode Apprenti ?

Le mode Apprenti s'adresse aux développeurs, aux scientifiques des données et aux décisionnaires d'entreprise :

* Les **développeurs** peuvent utiliser le mode Apprenti pour s'assurer que les API Rank et Reward sont correctement utilisées dans l'application, et que les caractéristiques envoyées à Personalizer depuis l'application ne contiennent pas de bogues ou de caractéristiques non pertinentes telles qu'un horodateur ou un élément UserID.

* Les **scientifiques des données** peuvent utiliser le mode Apprenti pour vérifier l'efficacité des caractéristiques dans le cadre de l'apprentissage des modèles Personalizer, et pour vérifier que les délais d'attente des récompenses ne sont ni trop longs ni trop courts.

* Les **décisionnaires d'entreprise** peuvent utiliser le mode Apprenti pour évaluer la capacité de Personalizer à améliorer les résultats (c'est-à-dire les récompenses) par rapport à la logique métier existante. Ils peuvent ainsi prendre des décisions éclairées qui ont un impact sur l'expérience utilisateur lorsque le chiffre d'affaires et la satisfaction des utilisateurs sont en jeu.

## <a name="comparing-behaviors---apprentice-mode-and-online-mode"></a>Comparaison des comportements : mode Apprenti et mode En ligne

L'apprentissage n'est pas le même en mode Apprenti et en mode En ligne.

|Domaine|Mode Apprenti|Mode En ligne|
|--|--|--|
|Impact sur l'expérience utilisateur|Vous pouvez utiliser le comportement utilisateur existant pour effectuer l'apprentissage de Personalizer en le laissant observer (et non pas influencer) ce qu'aurait été votre **action par défaut** et la récompense obtenue. Autrement dit, l'expérience et les résultats de vos utilisateurs ne seront pas affectés.|Afficher la première action retournée par l'appel de l'API Rank pour influencer le comportement de l'utilisateur.|
|Vitesse d'apprentissage|Personalizer apprendra plus lentement en mode Apprenti qu'en mode En ligne. Le mode Apprenti permet uniquement d'apprendre en observant les récompenses obtenues par votre **action par défaut**, ce qui limite la vitesse d'apprentissage car aucune exploration ne peut être effectuée.|Permet d'apprendre plus vite car il est à la fois possible d'exploiter le modèle en cours et d'explorer de nouvelles tendances.|
|Efficacité d'apprentissage « plafond »|Personalizer peut se rapprocher des performances de votre logique métier de base (récompenses totales obtenues par l'**action par défaut** de chaque appel de l'API Rank), mais il est très rare qu'il fasse aussi bien, et il ne fait jamais mieux.|Personalizer doit offrir de meilleures performances que la référence de l'application et, avec le temps, lorsqu'il stagne, vous devez procéder à une évaluation hors connexion et à une évaluation des caractéristiques pour continuer à améliorer le modèle. |
|Valeur de l'API Rank pour rewardActionId|L'expérience des utilisateurs n'est pas affectée, car _rewardActionId_ est toujours la première action que vous envoyez dans la requête Rank. En d'autres termes, l'API Rank ne fait rien de visible pour votre application en mode Apprenti. Les API Reward de votre application ne doivent pas modifier la façon dont elle les utilise d'un mode à l'autre.|L'expérience des utilisateurs sera modifiée par le paramètre _rewardActionId_ que Personalizer choisit pour votre application. |
|Évaluations|Personalizer compare les récompenses totales obtenues par votre logique métier par défaut et les récompense totales que Personalizer obtiendrait s'il était en mode En ligne. Une comparaison est disponible sur le portail Azure pour cette ressource|Évaluez l'efficacité de Personalizer en procédant à des [évaluations hors connexion](concepts-offline-evaluation.md) afin de pouvoir comparer les récompenses totales obtenues par Personalizer et les récompenses potentielles de la référence de l'application.|

Remarque sur l'efficacité du mode Apprenti :

* L'efficacité de Personalizer en mode Apprenti se rapproche rarement des 100 % de la référence de l'application ; et il ne fait jamais mieux.
* Les bonnes pratiques consistent à ne pas essayer d’atteindre 100 %. Selon le cas d’usage, une fourchette de 60 à 80 % doit être ciblée.

## <a name="using-apprentice-mode-to-train-with-historical-data"></a>Utilisation du mode Apprenti pour effectuer l'apprentissage avec des données historiques

Si vous disposez d'une grande quantité de données historiques que vous souhaitez utiliser dans le cadre de l'apprentissage de Personalizer, vous pouvez utiliser le mode Apprenti pour relire les données via Personalizer.

Configurez Personalizer en mode Apprenti et créez un script qui appelle l'API Rank avec les caractéristiques d'actions et de contexte des données historiques. Appelez l'API Reward en vous basant sur vos calculs des enregistrements de ces données. Comptez environ 50 000 événements historiques pour obtenir quelques résultats, et 500 000 pour des résultats plus fiables.

Lors d'un apprentissage à partir de données historiques, il est préférable que les données envoyées (caractéristiques liées au contexte et aux actions, disposition de celles-ci dans le fichier JSON utilisé pour les requêtes Rank, et calcul de la récompense dans ce jeu de données d'apprentissage) correspondent aux données (caractéristiques et calcul de la récompense) disponibles dans l'application existante.

Les données hors connexion et a posteriori ont tendance à être plus incomplètes et plus bruyantes, et leur format est différent. Bien qu'il soit possible d'effectuer l'apprentissage à partir de données historiques, les résultats obtenus sont parfois peu concluants et ne permettent pas toujours de déterminer si l'apprentissage de Personalizer est efficace, surtout si les caractéristiques varient entre les données antérieures et l'application existante.

Généralement, dans le cadre d'un apprentissage à l'aide de données historiques, le passage en mode Apprenti et l'apprentissage à partir d'une application existante permettent de bénéficier d'un modèle efficace, avec moins de travail, d'engineering données et de nettoyage.

## <a name="using-apprentice-mode-versus-ab-tests"></a>Utilisation du mode Apprenti et tests A/B

Les tests A/B des traitements de Personalizer ne sont utiles qu'une fois que Personalizer a été validé et qu'il utilise le mode d'apprentissage En ligne. En mode Apprenti, seule l'**action par défaut** est utilisée, ce qui signifie que tous les utilisateurs voient l'expérience de contrôle.

Même si Personalizer n'est que le _traitement_, le même problème se pose lorsque la validation des données est intéressante pour l'apprentissage de Personalizer. Le mode Apprenti pourrait être utilisé à la place, avec 100 % du trafic et des utilisateurs ayant tous accès à l'expérience de contrôle (non affectée).

Dès que vous disposez d'un cas d'usage qui utilise Personalizer et le mode d'apprentissage En ligne, les tests A/B vous permettent d'effectuer des cohortes contrôlées et une comparaison scientifique des résultats qui peuvent être plus complexes que les signaux utilisés pour les récompenses. Exemple de question à laquelle un test A/B peut répondre : `In a retail website, Personalizer optimizes a layout and gets more users to _check out_ earlier, but does this reduce total revenue per transaction?`

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [événements actifs et inactifs](concept-active-inactive-events.md)
