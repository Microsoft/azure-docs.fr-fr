---
title: Utiliser l’IA pour comprendre les données de stockage blob
titleSuffix: Azure Cognitive Search
description: Découvrez les fonctionnalités d’analyse du langage naturel et des images dans Recherche cognitive Azure, ainsi que la façon dont ces processus s’appliquent au contenu stocké dans des blobs Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: a0d32f00bd3c7f8daa2984bdc7c9b9dfb5add218
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91362795"
---
# <a name="use-ai-to-understand-blob-storage-data"></a>Utiliser l’IA pour comprendre les données de stockage blob

Dans Stockage Blob Azure, les données sont souvent constituées de divers contenus non structurés comme des images, du texte long, des fichiers PDF et des documents Office. En utilisant les fonctionnalités d’IA dans Recherche cognitive Azure, vous pouvez arriver à comprendre et à extraire des informations précieuses des objets blob, et ce de différentes manières. Voici des exemples d’application de l’IA au contenu d’objets blob :

+ Extraction de texte d’images à l’aide de la reconnaissance optique de caractères (OCR)
+ Création d’une description de scène ou d’étiquettes à partir d’une photo
+ Détection de langue et traduction de texte dans différentes langues
+ Inférer une structure via un reconnaissance d’entité en recherchant des références à des personnes, des dates, des lieux ou des organisations

Même si vous n’êtes intéressé que par une seule de ces fonctionnalités d’IA, il est courant d’en combiner plusieurs dans le même pipeline (par exemple, extraction de texte d’une image numérisée et recherche des dates et des emplacements référencés). Il est également courant d’inclure un traitement personnalisé d’IA ou d’apprentissage automatique sous la forme de packages externes de pointe ou de modèles internes adaptés à vos données et à vos besoins.

L’enrichissement par IA crée de nouvelles informations, capturées sous forme de texte, stockées dans des champs. Après l’enrichissement, vous pouvez accéder à ces informations à partir d’un index de recherche en effectuant une recherche en texte intégral ou renvoyer les documents enrichis à Stockage Azure pour permettre de nouvelles expériences applicatives comme notamment l’exploration de données pour des scénarios de découverte ou d’analytique. 

Dans cet article, nous examinons l’enrichissement par IA à travers un objectif grand-angle pour vous permettre de saisir rapidement l’ensemble du processus, depuis la transformation des données brutes d’objets blob jusqu’aux informations requêtables dans un index de recherche ou une base de connaissances.

## <a name="what-it-means-to-enrich-blob-data-with-ai"></a>Que signifie « enrichir » des données blob avec l’IA ?

L’*enrichissement par IA * fait partie de l’architecture d’indexation de Recherche cognitive Azure qui intègre l’IA (intelligence artificielle) de Microsoft ou une IA personnalisée que vous fournissez. Il permet de mettre en œuvre des scénarios de bout en bout quand il s’agit de traiter des objets blob (existants ou nouvellement obtenus ou mis à jour), d’ouvrir tous les formats de fichiers pour en extraire des images et du texte, d’extraire les informations souhaitées avec différentes fonctionnalités d’IA, puis de les indexer dans un index de recherche pour accélérer la recherche, l’extraction et l’exploration. 

Les entrées sont vos objets blob, dans un même conteneur, dans Stockage Blob Azure. Les objets blob peuvent correspondre à pratiquement tout type de données texte ou image. 

La sortie est toujours un index de recherche, utilisé pour la recherche, l’extraction et l’exploration rapides de texte dans les applications clientes. Par ailleurs, la sortie peut aussi être une [*base de connaissances*](knowledge-store-concept-intro.md) qui projette des documents enrichis dans des blobs Azure ou des tables Azure pour effectuer une analyse en aval dans des outils comme Power BI ou des charges de travail de science des données.

Au milieu se trouve l’architecture du pipeline proprement dite. Le pipeline est basé sur la fonctionnalité d’*indexeur*, à laquelle vous pouvez affecter un *ensemble de compétences*, qui se compose d’une ou plusieurs *compétences* fournissant l’IA. L’objectif du pipeline est de produire des *documents enrichis* : le contenu brut de départ se dote d’une structure, d’un contexte et d’informations supplémentaires à mesure qu’il avance dans le pipeline. Les documents enrichis sont consommés pendant l’indexation pour créer des index inversés et d’autres structures utilisées dans la recherche en texte intégral ou l’exploration et l’analytique.

## <a name="required-resources"></a>Ressources nécessaires

Vous avez besoin des services Stockage Blob Azure et Recherche cognitive Azure, ainsi que d’un troisième service ou mécanisme fournissant l’intelligence artificielle :

+ Pour l’intelligence artificielle intégrée, le service Recherche cognitive s’intègre avec la vision d’Azure Cognitive Services et les API de traitement en langage naturel. Vous pouvez [attacher une ressource Cognitive Services](cognitive-search-attach-cognitive-services.md) pour ajouter la reconnaissance optique de caractères, l’analyse d’images ou le traitement en langage naturel (détection de la langue, traduction de texte, reconnaissance d’entité et extraction de phrases clés). 

+ Pour une IA personnalisée qui utilise des ressources Azure, vous pouvez définir une compétence personnalisée qui encapsule la fonction ou le modèle externes que vous souhaitez utiliser. Des [compétences personnalisées](cognitive-search-custom-skill-interface.md) peuvent utiliser du code fourni par Azure Functions, Azure Machine Learning, Azure Form Recognizer ou une autre ressource accessible via le protocole HTTPS.

+ Pour une IA personnalisée non-Azure, votre modèle ou module doit être accessible à un indexeur via le protocole HTTP.

Si tous les services ne sont pas immédiatement disponibles, commencez directement sur la page du portail de votre compte de stockage. Dans la page de navigation de gauche, sous **Service Blob**, cliquez sur **Ajouter Recherche cognitive Azure** pour créer un nouveau service ou en sélectionner un existant. 

Une fois que vous avez ajouté Recherche cognitive Azure à votre compte de stockage, vous pouvez suivre le processus standard d’enrichissement de données dans une source de données Azure. Nous vous recommandons d’utiliser l’Assistant **Importation des données** de Recherche cognitive Azure pour une mise en route simple de l’enrichissement par IA. Vous pouvez attacher une ressource Cognitive Services pendant le flux de travail. Ce guide de démarrage rapide vous accompagne tout au long de la procédure : [Créez un pipeline d’enrichissement par IA sur le portail](cognitive-search-quickstart-blob.md). 

Les sections suivantes examinent de plus près les composants et le flux de travail.

## <a name="use-a-blob-indexer"></a>Utiliser un indexeur d’objets blob

L’enrichissement par IA est une extension du pipeline d’indexation, et dans Recherche cognitive Azure, ces pipelines reposent sur un *indexeur*. Un indexeur est un sous-service qui reconnaît les sources de données. Avec sa logique interne, il échantillonne les données, lit les données de métadonnées, extrait les données et les sérialise dans des documents JSON à partir de formats natifs pour être ensuite importées. Les indexeurs sont souvent utilisés seuls pour l’importation, indépendamment de l’IA, mais si vous voulez créer un pipeline d’enrichissement par IA, vous devrez l’associer à un indexeur et à un ensemble de compétences. Cette section met en lumière l’indexeur ; la section suivante porte sur les ensembles de compétences.

Les blobs dans le service Stockage Azure sont indexés à l’aide de l’[indexeur de blobs](search-howto-indexing-azure-blob-storage.md). Vous pouvez appeler cet indexeur à partir de l’Assistant **Importation de données**, d’une API REST ou d’un Kit de développement logiciel (SDK). Un indexeur de blobs est appelé quand la source de données que l’indexeur utilise est un conteneur d’objets blob Azure. Vous pouvez indexer un sous-ensemble de vos blobs en créant un répertoire virtuel, que vous pouvez ensuite transmettre comme paramètre, ou en filtrant sur une extension de type de fichier.

Un indexeur effectue le « craquage de document » en ouvrant un objet blob pour en inspecter le contenu. Une fois connecté à la source de données, il s’agit de la première étape du pipeline. Pour les données blob, c’est à ce stade que les fichiers PDF, les documents Office, les images et d’autres types de contenu sont détectés. Le craquage de document avec extraction de texte n’est pas facturé. Le craquage de document avec extraction d’images est facturé aux tarifs indiqués dans la [page des tarifs](https://azure.microsoft.com/pricing/details/search/).

Même si le craquage concerne tous les documents, l’enrichissement ne se produit que si vous fournissez explicitement les compétences nécessaires. Par exemple, si votre pipeline est constitué exclusivement par l’analyse d’images, le texte contenu dans votre conteneur ou vos documents est ignoré.

L’indexeur de blobs intègre des paramètres de configuration et prend en charge le suivi des modifications si les données sous-jacentes fournissent suffisamment d’informations. Pour plus d’informations, consultez [Comment configurer un indexeur de blobs](search-howto-indexing-azure-blob-storage.md).

## <a name="add-ai-components"></a>Ajouter des composants d’IA

L’enrichissement par IA s’appuie sur des modules qui recherchent des modèles ou des caractéristiques, puis effectue une opération en conséquence. La reconnaissance faciale dans les photos, les descriptions textuelles de photos, la détection d’expressions clés dans un document et la reconnaissance optique (ou la reconnaissance de texte imprimé ou manuscrit dans des fichiers binaires) sont autant d’exemples représentatifs.

Dans Recherche cognitive Azure, les *compétences* sont les composants individuels du traitement de l’IA que vous pouvez utiliser de façon autonome ou en association avec d’autres compétences. 

+ Les compétences intégrées sont adossées à Cognitive Services, l’analyse d’images reposant sur Vision par ordinateur et le traitement en langage naturel sur Analyse de texte. Pour obtenir la liste complète, consultez [Compétences intégrées pour l’enrichissement de contenu](cognitive-search-predefined-skills.md).

+ Les compétences personnalisées correspondent à du code personnalisé, encapsulé dans la [définition d’une interface](cognitive-search-custom-skill-interface.md) qui permet une intégration dans le pipeline. Dans les solutions des clients, il est courant d’utiliser les deux, les compétences personnalisées fournissant des modules d’IA open source, tiers ou internes.

L’*ensemble de compétences* est l’assortiment de compétences utilisé dans un pipeline ; il est appelé après que la phase de craquage de document a mis le contenu à disposition. Un indexeur peut consommer un seul ensemble de compétences, mais celui-ci existe indépendamment d’un indexeur. Vous pouvez donc le réutiliser dans d’autres scénarios.

Si les compétences personnalisées peuvent paraître complexes, elles peuvent s’avérer simples du point de vue de l’implémentation. Si les packages dont vous disposez fournissent des critères spéciaux ou des modèles de classification, le contenu que vous extrayez des objets blob peut être transmis à ces modèles pour traitement. Comme l’enrichissement par IA est basé sur Azure, votre modèle doit aussi se trouver dans Azure. Certaines méthodologies d’hébergement courantes incluent l’utilisation d’[Azure Functions](cognitive-search-create-custom-skill-example.md) ou de [Containers](https://github.com/Microsoft/SkillsExtractorCognitiveSearch).

Les compétences intégrées adossées à Cognitive Services nécessitent une clé d’abonnement tout-en-un [Cognitive Services ](cognitive-search-attach-cognitive-services.md) attachée qui vous donne accès à la ressource. Une clé tout-en-un vous fait bénéficier de l’analyse d’images, de la détection de langue, de la traduction de texte et de l’analyse de texte. Les autres compétences intégrées sont les fonctionnalités de Recherche cognitive Azure et ne nécessitent pas de service ou de clé supplémentaire. Les fonctionnalités de mise en forme, de fractionnement et de fusion de texte sont des exemples de compétences d’assistance qui sont parfois nécessaires au moment de concevoir le pipeline.

Si vous utilisez uniquement des compétences personnalisées et des compétences d’utilitaire intégrées, il n’existe aucune dépendance ou de coûts en rapport avec Cognitive Services.

## <a name="consume-ai-enriched-output-in-downstream-solutions"></a>Consommer une sortie enrichie par l’IA dans des solutions en aval

La sortie de l’enrichissement par IA est soit un index de recherche dans Recherche cognitive Azure, soit une [base de connaissances](knowledge-store-concept-intro.md) dans Stockage Azure.

Dans Recherche cognitive Azure, un index de recherche sert à l’exploration interactive dans une application cliente à partir de requêtes de texte libre et filtrées. Les documents enrichis créés via l’IA sont au format JSON et sont indexés comme tous les documents dans Recherche cognitive Azure, tirant ainsi parti de tous les avantages offerts par un indexeur. Par exemple, pendant l’indexation, l’indexeur d’objets blob se réfère aux paramètres de configuration pour utiliser les mappages de champs ou la logique de détection des changements. Ces paramètres sont entièrement disponibles pour l’indexation normale et les charges de travail enrichies par l’IA. Après l’indexation, quand le contenu est stocké dans Recherche cognitive Azure, vous pouvez créer des requêtes élaborées et des expressions de filtre pour comprendre votre contenu.

Dans Stockage Azure, une base de connaissances peut prendre deux formes : celle d’un conteneur d’objets blob ou celle de tables dans Stockage Table. 

+ Un conteneur d’objets blob capture les documents enrichis dans leur intégralité, ce qui est utile si vous souhaitez alimenter d’autres processus. 

+ En revanche, Stockage Table peut accueillir des projections physiques de documents enrichis. Vous pouvez créer des tranches ou des couches de documents enrichis qui incluent ou excluent des parties spécifiques. Pour l’analyse dans Power BI, les tables stockées dans Table Azure deviennent la source de données pour une visualisation et une exploration plus poussées.

Un document enrichi à la fin du pipeline est différent de sa version d’entrée initiale du fait de la présence de champs supplémentaires contenant les nouvelles informations extraites ou générées pendant l’enrichissement. Ainsi, vous pouvez utiliser en même temps du contenu d’origine et du contenu créé, quelle que soit la structure de sortie que vous utilisez.

## <a name="next-steps"></a>Étapes suivantes

L’enrichissement par IA offre bien plus de possibilités, qui permettent d’exploiter au mieux les données contenues dans Stockage Azure. Vous pouvez notamment combiner les services Cognitive Services de différentes manières ou créer des compétences personnalisées s’il n’existe pas de service cognitif pour le scénario. Pour en savoir plus, suivez les liens ci-dessous.

+ [Charger, télécharger et répertorier des blobs à l’aide du portail Azure (Stockage blob Azure)](../storage/blobs/storage-quickstart-blobs-portal.md)
+ [Configurer un indexeur blob (Recherche cognitive Azure)](search-howto-indexing-azure-blob-storage.md) 
+ [Vue d’ensemble de l’enrichissement de l’IA (Recherche cognitive Azure)](cognitive-search-concept-intro.md) 
+ [Créer un ensemble de compétences (Recherche cognitive Azure)](cognitive-search-defining-skillset.md)
+ [Mapper des nœuds dans une arborescence d’annotation (Recherche cognitive Azure)](cognitive-search-output-field-mapping.md)