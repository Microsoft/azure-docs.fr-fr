---
title: Conseils sur la conception de l’enrichissement par IA
titleSuffix: Azure Cognitive Search
description: Conseils et résolution des problèmes pour la configuration des pipelines d’enrichissement de l’IA dans la Recherche cognitive Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/08/2020
ms.openlocfilehash: 1ab366cddbabf7e6d574189892e779ab49f6fad8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91403379"
---
# <a name="tips-for-ai-enrichment-in-azure-cognitive-search"></a>Conseils sur l’enrichissement par IA dans Recherche cognitive Azure

Cet article contient une liste de conseils et astuces destinés à vous permettre d’aller de l’avant lorsque vous commencez à utiliser les fonctionnalités d’enrichissement de l’IA dans la Recherche cognitive Azure. 

Si ce n’est déjà fait, suivez le [Tutoriel : Appeler des API d’enrichissement de l’IA](cognitive-search-quickstart-blob.md) pour apprendre à appliquer des enrichissements de l’IA à une source de données Blob.

## <a name="tip-1-start-with-a-small-dataset"></a>Conseil 1 : commencez avec un petit jeu de données
La meilleure façon de détecter rapidement des problèmes consiste à augmenter la vitesse à laquelle vous pouvez résoudre ceux-ci. La meilleure façon de réduire le temps d’indexation consiste à diminuer le nombre de documents à indexer. 

Commencez par créer une source de données ne contenant que quelques documents/enregistrements. Votre exemple de document doit refléter assez fidèlement la diversité des documents qui seront indexés. 

Exécutez votre exemple de document via le pipeline de bout en bout et vérifiez que les résultats correspondent à vos besoins. Lorsque vous êtes satisfait des résultats, vous pouvez ajouter des fichiers à votre source de données.

## <a name="tip-2-make-sure-your-data-source-credentials-are-correct"></a>Conseil 2 : assurez-vous que les informations d’identification de la source de données sont correctes
La connexion de source de données n’est pas validée tant que vous ne définissez pas un indexeur qui l’utilise. Si vous rencontrez des erreurs signalant que l’indexeur ne peut pas accéder aux données, vérifiez les points suivants :
- Votre chaîne de connexion est correcte. En particulier lorsque vous créez des jetons SAP, veillez à utiliser le format attendu par la Recherche cognitive Azure. Pour découvrir les différents formats pris en charge, voir la section [Comment spécifier des informations d’identification](
https://docs.microsoft.com/azure/search/search-howto-indexing-azure-blob-storage#how-to-specify-credentials).
- Le nom de votre conteneur dans l’indexeur est correct.

## <a name="tip-3-see-what-works-even-if-there-are-some-failures"></a>Conseil 3 : regardez ce qui fonctionne même en présence d’erreurs
Parfois, une petite défaillance arrête un indexeur. C’est bien si vous prévoyez de résoudre les problèmes un par un. Cependant, vous pourriez vouloir ignorer un type particulier d’erreur et permettre à l’indexeur de continuer à opérer afin que vous puissiez voir quels flux fonctionnent réellement.

Dans ce cas, il se peut que vous vouliez demander à l’indexeur d’ignorer certaines erreurs. Pour ce faire, définissez les valeurs de *maxFailedItems* et *maxFailedItemsPerBatch* sur -1 dans le cadre de la définition de l’indexeur.

```
{
  "// rest of your indexer definition
   "parameters":
   {
      "maxFailedItems":-1,
      "maxFailedItemsPerBatch":-1
   }
}
```
> [!NOTE]
> Il est recommandé de définir maxFailedItems, maxFailedItemsPerBatch sur 0 pour les charges de travail de production

## <a name="tip-4-use-debug-sessions-to-identify-and-resolve-issues-with-your-skillset"></a>Conseil 4 : Utiliser des sessions de débogage pour identifier et résoudre les problèmes liés à vos compétences 

Les sessions de débogage consistent en un éditeur visuel qui fonctionne avec un ensemble de compétences existant dans le portail Azure. Au sein d’une session de débogage, vous pouvez identifier et résoudre les erreurs, valider les modifications et les transférer vers un ensemble de compétences de production dans le pipeline d’enrichissement par IA. Il s’agit d’une fonctionnalité d’évaluation [lire la documentation](./cognitive-search-debug-session.md). Pour plus d’informations sur les concepts et la mise en route, consultez [Sessions de débogage](./cognitive-search-tutorial-debug-sessions.md).

Les sessions de débogage qui fonctionnent sur un seul document sont un excellent moyen de créer de manière itérative des pipelines d’enrichissement plus complexes.

## <a name="tip-5-looking-at-enriched-documents-under-the-hood"></a>Conseil 5 : examinez les documents enrichis temporaires 
Les documents enrichis sont des structures temporaires créées pendant l’enrichissement, puis supprimées une fois le traitement terminé.

Pour capturer un instantané du document enrichi créé lors de l’indexation, ajoutez à votre index un champ nommé ```enriched```. L’indexeur vide automatiquement dans ce champ une représentation de chaîne de tous les enrichissements de ce document.

Le champ ```enriched``` contiendra une chaîne qui est une représentation logique du document enrichi en mémoire dans JSON.  Toutefois, la valeur du champ est un document JSON valide. Les guillemets étant échappés, vous devez remplacer `\"` par `"` afin d’afficher le document en tant que JSON formaté. 

Le champ enrichi est fourni à des fins de débogage uniquement, pour vous aider à comprendre la forme logique du contenu par rapport auquel les expressions sont évaluées. Vous ne devez pas dépendre ce champ pour l’indexation.

Ajoutez un champ ```enriched``` dans votre définition d’index à des fins de débogage :

#### <a name="request-body-syntax"></a>Syntaxe du corps de la demande
```json
{
  "fields": [
    // other fields go here.
    {
      "name": "enriched",
      "type": "Edm.String",
      "searchable": false,
      "sortable": false,
      "filterable": false,
      "facetable": false
    }
  ]
}
```

## <a name="tip-6-expected-content-fails-to-appear"></a>Conseil 6 : le contenu attendu n’apparaît pas

Un contenu manquant peut résulter d’une suppression de documents lors de l’indexation. Les limites de taille de document pour les niveaux Gratuit et De base sont basses. Tout fichier dépassant la limite est écarté lors de l’indexation. Vous pouvez vérifier la présence de documents écartés dans le portail Azure. Dans le tableau de bord du service de recherche, double-cliquez sur la vignette Indexeurs. Examinez le taux de documents correctement indexés. S’il n’est pas de 100 %, vous pouvez cliquer dessus pour obtenir plus de détails. 

Si le problème est lié à la taille du fichier, vous pouvez voir une erreur similaire à celle-ci : « L’objet blob \<file-name> a une taille de \<file-size> octets, ce qui est supérieur à la taille maximale d’extraction de document correspondant à votre niveau de service actuel. » Pour plus d’informations sur les limites de l’indexeur, voir [Limites du service](search-limits-quotas-capacity.md).

Un échec d’affichage du contenu peut également résulter d’erreurs de mappage d’entrée/sortie. Par exemple, un nom de cible de sortie est « Contacts », mais le nom du champ d’index est « contacts » en minuscules. Le système peut retourner 201 messages de réussite pour le pipeline entier, de sorte que vous pensez que l’indexation a réussi alors qu’en fait un champ est vide. 

## <a name="tip-7-extend-processing-beyond-maximum-run-time-24-hour-window"></a>Conseil 7 : prolongez le traitement au-delà du temps d’exécution maximal (fenêtre de 24 heures)

L’analyse d’image nécessite une grande capacité de calcul, même pour des cas simples. Ainsi, quand des images sont particulièrement volumineuses ou complexes, les temps de traitement peuvent dépasser le temps maximal imparti. 

Le temps d’exécution maximal varie selon le niveau : de quelques minutes pour le niveau Gratuit, à une durée d’indexation de 24 heures pour les niveaux facturables. Si le traitement n’aboutit pas dans un délai de 24 heures pour un traitement à la demande, passez à une planification telle que l’indexeur reprenne le traitement là où il l’a laissé. 

Pour les indexeurs planifiés, l’indexation reprend dans le délai prévu au dernier bon document connu. Avec une planification récurrente, l’indexeur peut opérer à sa manière dans le backlog d’images sur une série d’heures ou de jours, jusqu’à ce que toutes les images non traitées le soient. Pour plus d’informations sur la syntaxe de la planification, consultez [Étape 3 : Créer un indexeur](search-howto-indexing-azure-blob-storage.md#step-3-create-an-indexer) ou consultez [Guide pratique pour planifier des indexeurs pour la Recherche cognitive Azure](search-howto-schedule-indexers.md).

> [!NOTE]
> Si un indexeur est défini sur une certaine planification, mais échoue à plusieurs reprises sur le même document chaque fois qu’il s’exécute, l’indexeur commence à s’exécuter à un intervalle moins fréquent (jusqu’à un maximum d’au moins une fois toutes les 24 heures) jusqu’à ce qu’il progresse correctement à nouveau.  Si vous pensez avoir résolu le problème qui provoquait le blocage de l’indexeur à un moment donné, vous pouvez effectuer une exécution à la demande de l’indexeur, et en cas de progression, l’indexeur reprend son intervalle de planification défini.

Pour une indexation basée sur le portail (telle que décrite dans le démarrage rapide), le choix de l’option d’indexeur « Exécuter une fois » limite le traitement à 1 heure (`"maxRunTime": "PT1H"`). Vous pouvez étendre la fenêtre de traitement.

## <a name="tip-8-increase-indexing-throughput"></a>Conseil 8 : augmentez le débit d’indexation

Pour une [indexation parallèle](search-howto-large-index.md), placez vos données dans plusieurs conteneurs ou dans plusieurs dossiers virtuels au sein du même conteneur. Créez ensuite plusieurs paires source de données et indexeur. Tous les indexeurs pouvant utiliser le même jeu de compétences et écrire dans le même index de recherche cible, votre application de recherche n’a pas besoin d’être informée de ce partitionnement.

## <a name="see-also"></a>Voir aussi

+ [Démarrage rapide : Créer un pipeline d’enrichissement de l’IA dans le portail](cognitive-search-quickstart-blob.md)
+ [Tutoriel : Découvrir les API REST d’enrichissement de l’IA](cognitive-search-tutorial-blob.md)
+ [Spécification des informations d’identification de la source de données](search-howto-indexing-azure-blob-storage.md#how-to-specify-credentials)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Guide pratique pour mapper des champs enrichis à un index](cognitive-search-output-field-mapping.md)