---
title: Regénérer un index de recherche
titleSuffix: Azure Cognitive Search
description: Ajoutez de nouveaux éléments, mettez à jour des documents ou des éléments existants ou supprimez des documents obsolètes via une régénération complète ou une indexation partielle visant à actualiser un index Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/18/2020
ms.openlocfilehash: ae8fd7f405beb20d516835ccb80b86e769fd0393
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94697082"
---
# <a name="how-to-rebuild-an-index-in-azure-cognitive-search"></a>Guide pratique pour regénérer un index dans la Recherche cognitive Azure

Cet article explique comment regénerer un index Recherche cognitive Azure et les circonstances dans lesquelles les regénérations sont nécessaires, et il contient des suggestions pour atténuer l’impact des regénérations sur les demandes des requêtes en cours.

Une *regénération* fait référence à la suppression et à la recréation des structures de données physiques associées à un index, notamment tous les index inversés basés sur un champ. Dans Recherche cognitive Azure, vous ne pouvez pas supprimer et recréer des champs un par un. Pour regénerer un index, la totalité du stockage des champs doit être supprimé, recréé sur la base d’un schéma d’index existant ou révisé, puis à nouveau rempli avec les données envoyées à l’index ou extraites de sources externes. 

Il est courant de regénérer les index pendant le développement quand vous itérez sur la conception de l’index, mais il peut également être nécessaire de les regénérer au niveau de la production pour prendre en compte des modifications structurelles, comme l’ajout de types complexes ou l’ajout de champs à des suggesteurs.

## <a name="rebuild-versus-refresh"></a>« Regénérer » ou « actualiser » ?

Ne confondez pas la regénération avec l’actualisation du contenu d’un index avec des documents nouveaux, modifiés ou supprimés. L’actualisation d’un corpus de recherche est presque inévitable dans toute application de recherche, certains scénarios nécessitant même des mises à jour à la minute (par exemple quand un corpus de recherche doit refléter les modifications d’inventaire dans une application de vente en ligne).

Tant que vous ne changez pas la structure de l’index, vous pouvez l’actualiser en appliquant les mêmes techniques que celles appliquées pour le charger initialement :

* Pour l’indexation en mode Push, appelez [Ajouter, mettre à jour ou supprimer des documents](/rest/api/searchservice/addupdate-or-delete-documents) pour envoyer (push) les modifications vers un index.

* Pour les indexeurs, vous pouvez [planifier l’exécution de l’indexeur](search-howto-schedule-indexers.md) et utiliser le suivi des modifications ou des horodatages pour identifier le delta. Si les mises à jour doivent être reflétées plus rapidement que ce qu’un planificateur peut gérer, vous pouvez utiliser l’indexation en mode Push à la place.

## <a name="rebuild-conditions"></a>Conditions de la recréation

Supprimez un index et recréez-en un si l’une des conditions suivantes est vraie. 

| Condition | Description |
|-----------|-------------|
| Modifier une définition de champ | La révision d’un nom de champ, d’un type de données ou [d’attributs d’index](/rest/api/searchservice/create-index) spécifiques (interrogeable, filtrable, triable, à choix multiples) exige une régénération complète. |
| Affecter un analyseur à un champ | Les [analyseurs](search-analyzers.md) sont définis dans un index, puis affectés à des champs. Vous pouvez à tout moment ajouter une nouvelle définition d’analyseur à un index, mais il n’est possible *d’affecter* l’analyseur qu’à la création du champ. Cette condition s’applique à la fois à la propriété **analyzer** et à la propriété **indexAnalyzer**. La propriété **searchAnalyzer** fait figure d’exception (elle peut être affectée à un champ existant). |
| Mettre à jour ou supprimer une définition d’analyseur dans un index | Il n’est pas possible de supprimer ou de modifier une configuration d’analyseur existante (analyseur, générateur de jetons, filtre de jetons ou filtre de caractères) dans l’index, à moins de regénérer la totalité de l’index. |
| Ajouter un champ à un suggesteur | Pour pouvoir ajouter un champ existant à une construction [Suggesteurs](index-add-suggesters.md), il faut regénérer l’index. |
| Supprimer un champ | Pour supprimer physiquement toutes les traces d’un champ, vous devez regénerer l’index. Lorsqu’une régénération immédiate n’est pas pratique, vous pouvez modifier le code de l’application pour désactiver l’accès au champ « supprimé » ou utiliser le [paramètre de requête $select](search-query-odata-select.md) pour choisir les champs représentés dans le jeu de résultats. Physiquement, le contenu et la définition du champ restent dans l’index jusqu’à la régénération suivante, lors de laquelle est appliqué un schéma omettant le champ en question. |
| Changer de niveau | Si vous avez besoin de davantage de capacité, il n’y a pas de mise à niveau sur place dans le portail Azure. Vous devez créer un service et regénérer entièrement les index sur le nouveau service. Pour faciliter l’automatisation de ce processus, vous pouvez utiliser l’exemple de code **index-backup-restore** dans cet [exemple de dépôt .NET Recherche cognitive Azure](https://github.com/Azure-Samples/azure-search-dotnet-samples). Cette application va sauvegarder votre index dans une série de fichiers JSON, puis recréer l’index dans un service de recherche que vous spécifiez.|

## <a name="update-conditions"></a>Conditions de mise à jour

Beaucoup d’autres modifications peuvent être effectuées sans impacter les structures physiques existantes. Plus précisément, les modifications suivantes n’exigent *pas* de regénération d’index. Pour ces modifications, vous pouvez [mettre à jour une définition d’index](/rest/api/searchservice/update-index) avec vos modifications.

+ Ajouter un nouveau champ
+ Définir l’attribut **retrievable** sur un champ existant
+ Définir un **searchAnalyzer** sur un champ existant
+ Ajouter une nouvelle définition d’analyseur dans un index
+ Ajouter, mettre à jour ou supprimer des profils de scoring
+ Ajouter, mettre à jour ou supprimer des paramètres CORS
+ Ajouter, mettre à jour ou supprimer des synonymMaps

Quand vous ajoutez un nouveau champ, les documents indexés existants reçoivent une valeur null pour le nouveau champ. Lors de l’actualisation suivante des données, les valeurs provenant des données sources externes remplacent les valeurs null ajoutées par Recherche cognitive Azure. Pour plus d’informations sur la mise à jour du contenu des index, consultez [Ajouter, mettre à jour ou supprimer des documents](/rest/api/searchservice/addupdate-or-delete-documents).

## <a name="how-to-rebuild-an-index"></a>Comment regénérer un index

Durant le développement, le schéma d’index change fréquemment. Anticipez cela en créant des index qui peuvent être rapidement supprimés, recréés et rechargés avec un petit jeu de données représentatif.

Pour les applications déjà en production, nous recommandons de créer un nouvel index qui s’exécute côte à côte avec un index existant pour éviter des temps d’arrêt dans les requêtes. Votre code d’application fournit la redirection vers le nouvel index.

L’indexation n’est pas exécutée en arrière-plan et le service équilibre l’indexation supplémentaire par rapport aux requêtes en cours. Pendant l’indexation, vous pouvez [surveiller les demandes de requête](search-monitor-queries.md) dans le portail pour vous assurer que les demandes sont traitées en temps voulu.

1. Déterminez si une regénération est nécessaire. Si vous ajoutez juste des champs, ou modifiez une partie de l’index qui n’est pas liée à des champs, vous pouvez simplement [mettre à jour la définition](/rest/api/searchservice/update-index) sans supprimer, recréer et recharger entièrement l’index.

1. [Obtenez une définition d’index](/rest/api/searchservice/get-index) pour vous y référer ultérieurement au besoin.

1. [Supprimez l’index existant](/rest/api/searchservice/delete-index), en supposant que vous n’exécutez pas le nouvel index et l’ancien index simultanément. 

   Toutes les requêtes ciblant cet index sont immédiatement supprimées. Souvenez-vous que la suppression d’un index est irréversible, détruisant le stockage physique pour la collection de champs et d’autres constructions. Prenez le temps de réfléchir aux implications avant de supprimer l’index. 

1. [Créez un index revu](/rest/api/searchservice/create-index), où le corps de la requête contient les définitions des champs nouveaux ou modifiés.

1. [Chargez l’index avec des documents ](/rest/api/searchservice/addupdate-or-delete-documents) d’une source externe.

Quand vous créez l’index, du stockage physique est alloué pour chaque champ dans le schéma d’index, avec un index inversé créé pour chaque champ avec possibilité de recherche. Des champs sans possibilité de recherche peuvent être utilisés dans des filtres ou des expressions, mais ils n’ont pas d’index inversé et n’autorisent pas la recherche approximative ou en texte intégral. Lors d’une regénération d’index, ces index inversés sont supprimés et recréés sur la base du schéma d’index que vous fournissez.

Quand vous chargez l’index, l’index inversé de chaque champ est rempli avec tous les mots uniques tokenisés de chaque document, avec un mappage aux ID des documents correspondants. Par exemple, lors de l’indexation d’un jeu de données avec des hôtels, un index inversé créé pour un champ Ville peut contenir des termes pour Seattle, Portland, etc. L’ID des documents qui incluent « Seattle » ou « Portland » dans le champ Ville figure à côté du terme. Lors d’une opération [Ajouter, mettre à jour ou supprimer](/rest/api/searchservice/addupdate-or-delete-documents), les termes et la liste des ID de document sont mis à jour en conséquence.

> [!NOTE]
> Si vous avez des exigences strictes dans le cadre d’un contrat SLA, vous pouvez envisager de provisionner un nouveau service spécifiquement pour ce travail, le développement et l’indexation se produisant dans une isolation complète d’un index de production. Un service distinct s’exécute sur son propre matériel, éliminant toute possibilité de contention des ressources. Une fois le développement terminé, vous laissez le nouvel index en place et vous redirigez les requêtes vers le nouveau point de terminaison et le nouvel index, ou bien vous exécutez le code terminé pour publier un index revu sur votre service Recherche cognitive Azure d’origine. Il n’existe actuellement aucun mécanisme pour déplacer un index prêt à l’emploi vers un autre service.

## <a name="check-for-updates"></a>Rechercher les mises à jour

Vous pouvez commencer à interroger un index dès que le premier document est chargé. Si vous connaissez l’ID d’un document, l’[API REST de recherche de document](/rest/api/searchservice/lookup-document) retourne le document spécifique. Pour un test plus large, attendez que l’index soit entièrement chargé, puis utilisez des requêtes pour vérifier le contexte que vous vous attendez à voir.

Vous pouvez utiliser l’[Explorateur de recherche](search-explorer.md) ou un outil de test web comme [Postman ou Visual Studio Code](search-get-started-rest.md) pour rechercher du contenu mis à jour.

Si vous avez ajouté ou renommé un champ, utilisez [$select](search-query-odata-select.md) pour retourner ce champ : `search=*&$select=document-id,my-new-field,some-old-field&$count=true`.

## <a name="see-also"></a>Voir aussi

+ [Présentation de l’indexeur](search-indexer-overview.md)
+ [Indexer de grands jeux de données à grand échelle](search-howto-large-index.md)
+ [Indexation dans le portail](search-import-data-portal.md)
+ [Indexeur Azure SQL Database](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
+ [Indexeur Azure Cosmos DB](search-howto-index-cosmosdb.md)
+ [Indexeur Stockage Blob Azure](search-howto-indexing-azure-blob-storage.md)
+ [Indexeur Stockage Table Azure](search-howto-indexing-azure-tables.md)
+ [Sécurité dans Recherche cognitive Azure](search-security-overview.md)