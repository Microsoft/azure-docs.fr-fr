---
title: 'Tutoriel : Utiliser des sessions de débogage pour diagnostiquer, corriger et valider les changements apportés à votre ensemble de compétences'
titleSuffix: Azure Cognitive Search
description: Les sessions de débogage (en préversion) fournissent une interface basée sur le portail pour évaluer et réparer les problèmes/erreurs rencontrés dans vos ensembles de compétences
author: HeidiSteen
ms.author: heidist
manager: nitinme
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 09/25/2020
ms.openlocfilehash: 8ec39c4616f5a34f8326b56d4f0ba6e15cdad91c
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94699115"
---
# <a name="tutorial-diagnose-repair-and-commit-changes-to-your-skillset"></a>Tutoriel : Diagnostiquer, réparer et valider les changements apportés à votre ensemble de compétences

Dans cet article, vous allez utiliser le portail Azure pour accéder à des sessions de débogage afin de résoudre les problèmes liés à l’ensemble de compétences fourni. L’ensemble de compétences comporte des erreurs qui doivent être traitées. Ce tutoriel va vous guider tout au long d’une session de débogage pour identifier et résoudre les problèmes liés aux entrées et sorties de compétences.

> [!Important]
> Sessions de débogage est une fonctionnalité en préversion fournie sans contrat de niveau de service ; elle n’est pas recommandée pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

> [!div class="checklist"]
> * Un abonnement Azure. Créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) ou utilisez votre abonnement actuel
> * Une instance du service Recherche cognitive Azure
> * Un compte Azure Storage
> * [Application de bureau Postman](https://www.getpostman.com/)

## <a name="create-services-and-load-data"></a>Créer des services et charger des données

Ce tutoriel utilise la Recherche cognitive Azure et les services de Stockage Azure.

* [Téléchargez les exemples de données](https://github.com/Azure-Samples/azure-search-sample-data/tree/master/clinical-trials-pdf-19) qui se composent de 19 fichiers.

* [Créez un compte de stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal) ou [recherchez un compte existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/). 

   Choisissez la même région que celle de la Recherche cognitive Azure pour éviter des frais de bande passante.
   
   Il doit être de type StorageV2 (V2 universel).

* Ouvrez les pages des services de stockage et créez un conteneur. Une bonne pratique consiste à spécifier le niveau d’accès «privé ». Nommez votre conteneur `clinicaltrialdataset`.

* Dans le conteneur, cliquez sur **Charger** pour charger les exemples de fichiers que vous avez téléchargés et décompressés au cours de la première étape.

* [Créez un service Recherche cognitive Azure](search-create-service-portal.md) ou [recherchez un service existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Vous pouvez utiliser un service gratuit pour ce guide de démarrage rapide.

## <a name="get-a-key-and-url"></a>Obtenir une clé et une URL

Les appels REST requièrent l’URL du service et une clé d’accès et ce, sur chaque demande. Un service de recherche est créé avec les deux. Ainsi, si vous avez ajouté la Recherche cognitive Azure à votre abonnement, effectuez ce qui suit pour obtenir les informations nécessaires :

1. [Connectez-vous au portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez l’URL. Voici un exemple de point de terminaison : `https://mydemo.search.windows.net`.

1. Dans **Paramètres** > **Clés**, obtenez une clé d’administration pour avoir des droits d’accès complets sur le service. Il existe deux clés d’administration interchangeables, fournies pour assurer la continuité de l’activité au cas où vous deviez en remplacer une. Vous pouvez utiliser la clé primaire ou secondaire sur les demandes d’ajout, de modification et de suppression d’objets.

:::image type="content" source="media/search-get-started-rest/get-url-key.png" alt-text="Obtenir un point de terminaison et une clé d’accès HTTP" border="false":::

Toutes les demandes nécessitent une clé API sur chaque demande envoyée à votre service. L’utilisation d’une clé valide permet d’établir, en fonction de chaque demande, une relation de confiance entre l’application qui envoie la demande et le service qui en assure le traitement.

## <a name="create-data-source-skillset-index-and-indexer"></a>Créer une source de données, un ensemble de compétences, un index et un indexeur

Dans cette section, Postman et une collection fournie sont utilisés pour créer la source de données, l’ensemble de compétences, l’index et l’indexeur du service de recherche.

1. Si vous ne disposez pas de Postman, vous pouvez [télécharger l’application de bureau Postman ici](https://www.getpostman.com/).
1. [Télécharger la collection Postman Sessions de débogage](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/Debug-sessions)
1. Démarrer Postman
1. Sous **Fichiers** > **Nouveau**, sélectionnez la collection à importer.
1. Une fois la collection importée, développez la liste des actions (...).
1. Cliquez sur **Modifier**.
1. Entrez le nom de votre searchService (par exemple, si le point de terminaison est `https://mydemo.search.windows.net`, le nom du service est « `mydemo` »).
1. Entrez la valeur apiKey avec la clé primaire ou secondaire de votre service de recherche.
1. Entrez la valeur storageConnectionString à partir de la page des clés de votre compte de Stockage Azure.
1. Entrez la valeur containerName pour le conteneur que vous avez créé dans le compte de stockage.

> :::image type="content" source="media/cognitive-search-debug/postman-enter-variables.png" alt-text="modifier des variables dans Postman":::

La collection contient quatre appels REST différents utilisés pour terminer cette section.

Le premier appel crée la source de données. `clinical-trials-ds`. Le deuxième appel crée l’ensemble de compétences, `clinical-trials-ss`. Le troisième appel crée l’index, `clinical-trials`. Le quatrième et dernier appel crée l’indexeur, `clinical-trials-idxr`. Une fois tous les appels de la collection terminés, fermez Postman, puis revenez au portail Azure.

> :::image type="content" source="media/cognitive-search-debug/postman-create-data-source.png" alt-text="utilisation de Postman pour créer une source de données":::

## <a name="check-the-results"></a>Vérification des résultats

L’ensemble de compétences contient quelques erreurs courantes. Dans cette section, l’exécution d’une requête vide pour retourner tous les documents affiche plusieurs erreurs. Dans les étapes suivantes, les problèmes sont résolus à l’aide d’une session de débogage.

1. Accédez à votre service de recherche dans le portail Azure. 
1. Sélectionnez l’onglet **Index**. 
1. Sélectionnez l’index `clinical-trials`.
1. Cliquez sur **Rechercher** pour exécuter une requête vide. 

Une fois la recherche terminée, deux champs sans données listées, « organizations » et « locations », sont affichés dans la fenêtre. Suivez les étapes permettant de découvrir tous les problèmes produits par l’ensemble de compétences.

1. Retournez à la page Vue d’ensemble du service de recherche.
1. Sélectionnez l’onglet **Indexeurs**. 
1. Cliquez sur `clinical-trials-idxr`, puis sélectionnez la notification d’avertissement. 

Il existe de nombreux problèmes avec les mappages de champs de sortie de projection, et la page 3 comporte des avertissements car une ou plusieurs entrées de compétences ne sont pas valides.

Retournez à l’écran de vue d’ensemble du service de recherche.

## <a name="start-your-debug-session"></a>Démarrer votre session de débogage

> :::image type="content" source="media/cognitive-search-debug/new-debug-session-screen-required.png" alt-text="démarrer une nouvelle session de débogage":::

1. Cliquez sur l’onglet Sessions de débogage (préversion).
1. Sélectionnez +NewDebugSession.
1. Donnez un nom à la session. 
1. Connectez la session à votre compte de stockage. 
1. Indiquez le nom de l’indexeur. L’indexeur a des références à la source de données, à l’ensemble de compétences et à l’index.
1. Acceptez le choix de document par défaut pour le premier document de la collection. 
1. **Enregistrez** la session. L’enregistrement de la session lance le pipeline d’enrichissement par IA tel que défini par l’ensemble de compétences.

> [!Important]
> Une session de débogage ne fonctionne qu’avec un seul document. Un document spécifique du jeu de données peut être sélectionné. Si ce n’est pas le cas, la session est définie par défaut sur le premier document.

> :::image type="content" source="media/cognitive-search-debug/debug-execution-complete1.png" alt-text="Nouvelle session de débogage démarrée":::

Quand l’exécution de la session de débogage est terminée, la session affiche par défaut l’onglet Enrichissements par IA, en mettant en évidence le graphe des compétences.

+ Le graphe des compétences fournit une hiérarchie visuelle de l’ensemble de compétences et son ordre d’exécution de haut en bas. Les compétences qui sont côte à côte dans le graphe sont exécutées en parallèle. Le code couleur des compétences dans le graphe indique les types de compétences qui sont exécutées dans l’ensemble de compétences. Dans l’exemple, les compétences en vert sont du texte et la compétence en rouge est la vision. Si vous cliquez sur une compétence individuelle dans le graphe, les détails de cette instance de la compétence s’affichent dans le volet droit de la fenêtre de session. Les paramètres des compétences, un éditeur JSON, les détails de l’exécution et les erreurs/avertissements sont tous disponibles pour que vous puissiez les consulter et les modifier.
+ La structure de données enrichie détaille les nœuds de l’arborescence d’enrichissement générés par les compétences à partir du contenu du document source.

L’onglet Erreurs/avertissements fournit une liste bien plus petite que celle affichée précédemment car cette liste détaille uniquement les erreurs d’un seul document. Comme avec la liste affichée par l’indexeur, vous pouvez cliquer sur un message d’avertissement pour voir les détails le concernant.

## <a name="fix-missing-skill-input-value"></a>Remédier à la valeur d’entrée de compétence manquante

Sous l’onglet Erreurs/avertissements, une erreur est signalée pour une opération libellée `Enrichment.NerSkillV2.#1`. Les détails de cette erreur expliquent qu’il y a un problème avec une valeur d’entrée de compétence « /document/languageCode ». 

1. Retournez à l’onglet Enrichissements par IA.
1. Cliquez sur le **graphe des compétences**.
1. Cliquez sur la compétence étiquetée #1 pour afficher ses détails dans le volet de droite.
1. Recherchez l’entrée correspondant à « languageCode ».
1. Sélectionnez le symbole **</>** au début de la ligne, puis ouvrez l’évaluateur d’expression.
1. Cliquez sur le bouton **Évaluer** pour vérifier que cette expression génère une erreur. Cette opération permet de vérifier que la propriété « languageCode » n’est pas une entrée valide.

> :::image type="content" source="media/cognitive-search-debug/expression-evaluator-language.png" alt-text="Évaluateur d’expression":::

Il existe deux façons de rechercher cette erreur dans la session. La première consiste à examiner d’où provient l’entrée, c’est-à-dire quelle compétence de la hiérarchie est supposée produire ce résultat ? L’onglet Exécutions du volet des détails des compétences doit afficher la source de l’entrée. Si aucune source n’est affichée, cela indique une erreur de mappage de champ.

1. Cliquez sur l’onglet **Exécutions**.
1. Examinez les entrées (INPUTS) et recherchez « languageCode ». Aucune source n’est indiquée pour cette entrée. 
1. Basculez vers le volet de gauche pour afficher la structure de données enrichie. Aucun chemin mappé ne correspond à « languageCode ».

> :::image type="content" source="media/cognitive-search-debug/enriched-data-structure-language.png" alt-text="Structure de données enrichie":::

Il existe un chemin mappé pour « language ». Par conséquent, il y a une faute de frappe dans les paramètres des compétences. Pour résoudre ce problème, l’expression dans « /document/language » de la compétence #1 doit être mise à jour.

1. Ouvrez l’évaluateur d’expression **</>** pour le chemin « language ».
1. Copiez l’expression. Fermez la fenêtre.
1. Accédez aux paramètres des compétences de la compétence #1, puis ouvrez l’évaluateur d’expression **</>** pour l’entrée « languageCode ».
1. Collez la nouvelle valeur « /document/language » dans la zone Expression, puis cliquez sur **Évaluer**.
1. Elle doit afficher l’entrée correcte « en ». Cliquez sur Appliquer pour mettre à jour l’expression.
1. Cliquez sur **Enregistrer** dans le volet des détails des compétences, à droite.
1. Cliquez sur **Exécuter** dans le menu Fenêtre de la session. Cela lance une autre exécution de l’ensemble de compétences avec le document. 

Une fois l’exécution de la session de débogage terminée, cliquez sur l’onglet Erreurs/avertissements. Il indique alors que l’erreur intitulée « Enrichment.NerSkillV2.#1 » a disparu. Toutefois, il y a toujours deux avertissements indiquant que le service n’a pas pu mapper les champs de sortie des organisations (organizations) et des emplacements (locations) à l’index de recherche. Des valeurs sont manquantes : « /document/merged_content/organizations » et « /document/merged_content/locations ».

## <a name="fix-missing-skill-output-values"></a>Remédier aux valeurs de sortie de compétence manquantes

> :::image type="content" source="media/cognitive-search-debug/warnings-missing-value-locations-organizations.png" alt-text="Erreurs et avertissements":::

Il manque des valeurs de sortie d’une compétence. Pour identifier la compétence avec l’erreur, accédez à la structure de données enrichie, recherchez le nom de la valeur, puis examinez sa source d’origine. Dans le cas des valeurs « organizations » et « locations » manquantes, il s’agit de sorties de la compétence #1. Le fait d’ouvrir l’évaluateur d’expression </> pour chaque chemin affiche les expressions listées sous la forme « /document/content/organizations » et « /document/content/locations », respectivement.

> :::image type="content" source="media/cognitive-search-debug/expression-eval-missing-value-locations-organizations.png" alt-text="Entité organizations de l’évaluateur d’expression":::

La sortie de ces entités est vide alors qu’elle ne doit pas l’être. Quelles sont les entrées produisant ce résultat ?

1. Accédez au **graphe des compétences**, puis sélectionnez la compétence #1.
1. Sélectionnez l’onglet **Exécutions** dans le volet des détails des compétences, à droite.
1. Ouvrez l’évaluateur d’expression **</>** pour l’entrée (INPUT) « text ».

> :::image type="content" source="media/cognitive-search-debug/input-skill-missing-value-locations-organizations.png" alt-text="Entrée de la compétence de texte":::

Le résultat affiché pour cette entrée ne ressemble pas à une entrée de texte. Il ressemble à une image qui est entourée de nouvelles lignes. L’absence de texte signifie qu’aucune entité ne peut être identifiée. Si vous observez la hiérarchie de l’ensemble de compétences, vous pouvez voir que le contenu est d’abord traité par la compétence #6 (OCR), puis transmis à la compétence #5 (Fusion). 

1. Sélectionnez la compétence #5 (Fusion) dans le **graphe des compétences**.
1. Sélectionnez l’onglet **Exécutions** dans le volet des détails des compétences, à droite, puis ouvrez l’évaluateur d’expression **</>** pour les sorties (OUTPUTS) « mergedText ».

> :::image type="content" source="media/cognitive-search-debug/merge-output-detail-missing-value-locations-organizations.png" alt-text="Sortie de la compétence Fusion":::

Ici, le texte est associé à l’image. Si vous observez l’expression « /document/merged_content », l’erreur dans les chemins « organizations » et « locations » pour la compétence #1 est visible. Au lieu d’utiliser « /document/content », elle doit utiliser « /document/merged_content » pour les entrées « text ».

1. Copiez l’expression correspondant à la sortie « mergedText », puis fermez la fenêtre de l’évaluateur d’expression.
1. Sélectionnez la compétence #1 dans le **graphe des compétences**.
1. Sélectionnez l’onglet **Paramètres des compétences** dans le volet des détails des compétences, à droite.
1. Ouvrez l’évaluateur d’expression **</>** pour l’entrée « text ».
1. Collez la nouvelle expression dans la zone. Cliquez sur **Évaluer**.
1. L’entrée correcte avec le texte ajouté doit s’afficher. Cliquez sur **Appliquer** pour mettre à jour les paramètres des compétences.
1. Cliquez sur **Enregistrer** dans le volet des détails des compétences, à droite.
1. Cliquez sur **Exécuter** dans le menu Fenêtre de la session. Cela lance une autre exécution de l’ensemble de compétences avec le document.

Une fois l’exécution de l’indexeur terminée, les erreurs sont toujours présentes. Revenez à la compétence #1, puis recherchez la cause du problème. L’entrée pour la compétence a été corrigée de « contenu » en « merged_content ». Quelles sont les sorties de ces entités dans la compétence ?

1. Sélectionnez l’onglet **Enrichissements par IA**.
1. Sélectionnez le **graphe des compétences**, puis cliquez sur la compétence #1.
1. Parcourez les **paramètres des compétences** pour rechercher «outputs » (sorties).
1. Ouvrez l’évaluateur d’expression **</>** pour l’entité « organizations ».

> :::image type="content" source="media/cognitive-search-debug/skill-output-detail-missing-value-locations-organizations.png" alt-text="Sortie de l’entité organizations":::

L’évaluation du résultat de l’expression donne le résultat correct. La compétence consiste à identifier la valeur correcte pour l’entité, « organizations ». Toutefois, le mappage de sortie dans le chemin de l’entité génère toujours une erreur. En comparant le chemin de sortie de la compétence avec le chemin de sortie de l’erreur, la compétence apparente les sorties, les organisations et les emplacements sous le nœud /document/content, tandis que le mappage de champs de sortie s’attend à ce que les résultats soient apparentés sous le nœud /document/merged_content. À l’étape précédente, l’entrée est passée de « /document/content » à « /document/merged_content ». Le contexte dans les paramètres des compétences doit être changé afin de garantir que la sortie est générée avec le contexte approprié.

1. Sélectionnez l’onglet **Enrichissements par IA**.
1. Sélectionnez le **graphe des compétences**, puis cliquez sur la compétence #1.
1. Accédez aux **Paramètres des compétences** pour rechercher « context » (contexte).
1. Double-cliquez sur le paramètre pour « context », puis remplacez-le pour qu’il indique « /document/merged_content ».
1. Cliquez sur **Enregistrer** dans le volet des détails des compétences, à droite.
1. Cliquez sur **Exécuter** dans le menu Fenêtre de la session. Cela lance une autre exécution de l’ensemble de compétences avec le document.

> :::image type="content" source="media/cognitive-search-debug/skill-setting-context-correction-missing-value-locations-organizations.png" alt-text="Correction du contexte dans les paramètres des compétences":::

Toutes les erreurs ont été résolues.

## <a name="commit-changes-to-the-skillset"></a>Valider les changements apportés à l’ensemble de compétences

Quand la session de débogage a été lancée, le service de recherche a créé une copie de l’ensemble de compétences. Cela a pour but que les changements apportés n’affectent pas le système de production. Maintenant que vous avez terminé le débogage de votre ensemble de compétences, les correctifs peuvent être validés (remplacer l’ensemble de compétences d’origine) pour le système de production. Si vous voulez continuer à apporter des changements à l’ensemble de compétences sans affecter le système de production, la session de débogage peut être enregistrée et rouverte ultérieurement.

1. Cliquez sur **Valider les changements** dans le menu principal Sessions de débogage.
1. Cliquez sur **OK** pour vérifier que vous souhaitez mettre à jour votre ensemble de compétences.
1. Fermez la session de débogage, puis sélectionnez l’onglet **Indexeurs**.
1. Ouvrez votre indexeur « clinical-trials-idxr ».
1. Cliquez sur **Réinitialiser**.
1. Cliquez sur **Exécuter**. Cliquez sur **OK** pour confirmer.

Une fois l’exécution de l’indexeur terminée, il doit y avoir une coche verte et le mot « Success » (Réussite) en regard de l’horodatage de la dernière exécution sous l’onglet Historique d’exécution. Pour vérifier que les changements ont été appliqués :

1. Quittez **Indexeur**, puis sélectionnez l’onglet **Index**.
1. Ouvrez l’index « clinical-trials », puis sous l’onglet Explorateur de recherche, cliquez sur **Rechercher**.
1. La fenêtre de résultats doit indiquer que les entités organizations et locations sont désormais remplies avec les valeurs attendues.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les ensembles de compétences](./cognitive-search-working-with-skillsets.md)
> [En savoir plus sur l’enrichissement incrémentiel et la mise en cache](./cognitive-search-incremental-indexing-conceptual.md)