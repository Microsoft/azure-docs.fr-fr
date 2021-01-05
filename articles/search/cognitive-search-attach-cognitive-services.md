---
title: Joindre Cognitive Services à un ensemble de compétences
titleSuffix: Azure Cognitive Search
description: Instructions à suivre pour attacher un abonnement Cognitive Services tout-en-un à un pipeline d’enrichissement par IA dans Recherche cognitive Azure.
manager: nitinme
author: LuisCabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/17/2019
ms.openlocfilehash: c9f6a5ebc4f3242181196bd40b62f7522d025b84
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88924975"
---
# <a name="attach-a-cognitive-services-resource-to-a-skillset-in-azure-cognitive-search"></a>Attacher une ressource Cognitive Services à un ensemble de compétences dans Recherche cognitive Azure 

Lors de la configuration d’un pipeline d’enrichissement dans la Recherche cognitive Azure, vous pouvez enrichir un nombre limité de documents gratuitement. Vous pouvez également associer une ressource Cognitive Services facturable pour des charges de travail plus volumineuses et plus fréquentes.

Dans cet article, vous allez apprendre à joindre une ressource en affectant une clé à un ensemble de compétences qui définit un pipeline d’enrichissement.

## <a name="resources-used-during-enrichment"></a>Ressources utilisées pendant l’enrichissement

La Recherche cognitive Azure a une dépendance vis-à-vis de Cognitive Services, y compris [Vision par ordinateur](https://azure.microsoft.com/services/cognitive-services/computer-vision/), pour l’analyse d’images et la reconnaissance optique de caractères (OCR), [Analyse de texte](https://azure.microsoft.com/services/cognitive-services/text-analytics/) pour le traitement en langage naturel et d’autres enrichissements tels que la [Traduction de texte](https://azure.microsoft.com/services/cognitive-services/translator-text-api/). Dans le contexte de l’enrichissement dans la Recherche cognitive Azure, ces algorithmes d’IA sont encapsulés dans une *qualification*, placés dans un *ensemble de qualifications* et référencés par un *indexeur* lors de l’indexation.

## <a name="how-billing-works"></a>Comment la facturation fonctionne

+ La Recherche cognitive Azure utilise la clé de ressource Cognitive Services que vous fournissez sur un ensemble de compétences pour facturer l’enrichissement des images et du texte. L’exécution de qualifications facturables est facturée au [tarif de paiement à l’utilisation de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/).

+ L’extraction d’images est une opération de la Recherche cognitive Azure qui se produit lorsque les documents sont décodés avant l’enrichissement. L’extraction d’images est facturable. Pour connaître les prix appliqués à l’extraction d’images, voir la [page de tarification du service Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/).

+ L’extraction de texte se produit également lors de la phrase de décodage de document. Elle n’est pas facturable.

+ Les qualifications qui n’appellent pas Cognitive Services, y compris les compétences conditionnelles, le modélisateur, la fusion de texte et le fractionnement du texte, ne sont pas facturables.

## <a name="same-region-requirement"></a>Exigence de même région

Recherche cognitive Azure et Azure Cognitive Services doivent obligatoirement exister au sein de la même région. Autrement, vous obtenez ce message lors de l’exécution : `"Provided key is not a valid CognitiveServices type key for the region of your search service."` 

Il n’existe aucun moyen de changer un service de région. Si vous obtenez cette erreur, vous devez créer une ressource Cognitive Services située dans la même région que le service Recherche cognitive Azure.

> [!NOTE]
> Certaines compétences intégrées sont basées sur des services cognitifs non régionaux (par exemple la [compétence de traduction de texte](cognitive-search-skill-text-translation.md)). L’utilisation d’une compétence non régionale signifie que votre requête peut être desservie dans une région autre que la région de la Recherche cognitive Azure. Pour plus d’informations sur les services non régionaux, consultez la page [Produit Cognitive Services par région](https://aka.ms/allinoneregioninfo).

## <a name="use-free-resources"></a>Utiliser des ressources gratuites

Vous pouvez utiliser une option de traitement gratuite, limitée aux exercices des guides de démarrage rapide et des tutoriels d’enrichissement par IA.

Les ressources du niveau tarifaire Gratuit (enrichissements limités) sont limitées à 20 documents par jour, par indexeur. Vous pouvez supprimer et recréer l’indexeur pour réinitialiser le compteur.

1. Ouvrez l’Assistant Importation de données :

   ![Ouvrir l’Assistant Importation de données](media/search-get-started-portal/import-data-cmd.png "Ouvrir l’Assistant Importation de données")

1. Choisissez une source de données, puis passez à **Ajouter l’enrichissement par IA (facultatif)** . Pour suivre une procédure pas à pas de cet Assistant, consultez [Créer un index dans le Portail Azure](search-get-started-portal.md).

1. Développez **Attacher Cognitive Services**, puis sélectionnez **Gratuit (enrichissements limités)** .

   ![Section Attacher Cognitive Services développée](./media/cognitive-search-attach-cognitive-services/attach1.png "Section Attacher Cognitive Services développée")

1. Vous pouvez maintenant passer aux étapes suivantes, notamment **Ajouter les compétences cognitives**.

## <a name="use-billable-resources"></a>Utiliser des ressources facturables

Pour les charges de travail créant plus de 20 enrichissements par jour, veillez à attacher une ressource Cognitive Services facturable. Nous vous recommandons de toujours attacher une ressource Cognitive Services facturable, même si vous n’avez aucune intention d’appeler les API Cognitive Services. L’attachement d’une ressource modifie la limite quotidienne.

Vous n’êtes facturé que pour les qualifications qui appellent les API Cognitive Services. Vous n’êtes pas facturé pour les [qualifications personnalisées](cognitive-search-create-custom-skill-example.md) ou des qualifications telles que [Fusion de texte](cognitive-search-skill-textmerger.md), [Séparateur de texte ](cognitive-search-skill-textsplit.md) et [Modélisateur](cognitive-search-skill-shaper.md) qui ne sont pas basées sur des API.

1. Ouvrez l’Assistant Importation de données, choisissez une source de données, puis passez à **Ajouter l’enrichissement par IA (facultatif)** .

1. Développez **Attacher Cognitive Services**, puis sélectionnez **Créer une ressource Cognitive Services**. Un nouvel onglet s’ouvre pour vous permettre de créer la ressource :

   ![Créez une ressource Cognitive Services](./media/cognitive-search-attach-cognitive-services/cog-services-create.png "Créer une ressource Cognitive Services")

1. Dans la liste **Emplacement**, sélectionnez la région où se trouve votre service Recherche cognitive Azure. Veillez à utiliser cette région à des fins de performances. L’utilisation de cette région évite également les frais de bande passante sortante entre les différentes régions.

1. Dans la liste **Niveau tarifaire**, sélectionnez **S0** pour obtenir la collection complète de fonctionnalités Cognitive Services, y compris les fonctionnalités Vision et Langue qui sous-tendent les compétences prédéfinies fournies par Recherche cognitive Azure.

   Pour le niveau S0, vous pouvez trouver les tarifs des charges de travail spécifiques dans la [page de tarification de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/).
  
   + Dans la liste **Sélectionner une offre**, assurez-vous que **Cognitive Services** est sélectionné.
   + Sous les fonctionnalités **Langue**, les tarifs d’**Analyse de texte** standard s’appliquent à l’indexation basée sur l’intelligence artificielle.
   + Sous les fonctionnalités **Vision**, les tarifs de **Vision par ordinateur S1** s’appliquent.

1. Sélectionnez **Créer** pour approvisionner la nouvelle ressource Cognitive Services.

1. Revenez à l’onglet précédent contenant l’Assistant Importation de données. Sélectionnez **Actualiser** pour afficher la ressource Cognitive Services, puis sélectionnez-la :

   ![Sélectionner la ressource Cognitive Services](./media/cognitive-search-attach-cognitive-services/attach2.png "Sélectionner la ressource Cognitive Services")

1. Développez la section **Ajouter des compétences cognitives** pour sélectionner les qualifications cognitives spécifiques à exécuter sur vos données. Suivez les recommandations restantes de l’Assistant.

## <a name="attach-an-existing-skillset-to-a-cognitive-services-resource"></a>Attacher un ensemble de qualifications à une ressource Cognitive Services existante

Si vous avez un ensemble de qualifications existant, vous pouvez l’attacher à une ressource Cognitive Services nouvelle ou différente.

1. Dans la page **Vue d’ensemble du service**, sélectionnez l’onglet **Ensembles de qualifications** :

   ![Onglet Ensemble de compétences](./media/cognitive-search-attach-cognitive-services/attach-existing1.png "Onglet Ensemble de compétences")

1. Sélectionnez le nom de l’ensemble de qualifications, puis choisissez une ressource existante ou créez-en une. Cliquez sur **OK** pour confirmer vos modifications.

   ![Liste de ressources de l’ensemble de compétences](./media/cognitive-search-attach-cognitive-services/attach-existing2.png "Liste de ressources de l’ensemble de compétences")

   N’oubliez pas que l’option **Gratuit (enrichissements limités)** vous limite à 20 documents par jour et que vous pouvez utiliser l’option **Créer une ressource Cognitive Services** pour approvisionner une nouvelle ressource facturable. Si vous avez créé une ressource, sélectionnez **Actualiser** pour rafraîchir la liste des ressources Cognitive Services, puis sélectionnez la ressource.

## <a name="attach-cognitive-services-programmatically"></a>Attacher Cognitive Services par programme

Lorsque vous définissez l’ensemble de qualifications par programme, ajoutez une section `cognitiveServices` à l’ensemble de qualifications. Dans cette section, incluez la clé de la ressource Cognitive Services que vous souhaitez associer à l’ensemble de qualifications. Rappelez-vous que la ressource doit se trouver dans la même région que votre ressource Recherche cognitive Azure. Incluez également `@odata.type`, que vous définissez sur `#Microsoft.Azure.Search.CognitiveServicesByKey`.

L’exemple ci-après illustre ce modèle. Notez la section `cognitiveServices` à la fin de la définition.

```http
PUT https://[servicename].search.windows.net/skillsets/[skillset name]?api-version=2020-06-30
api-key: [admin key]
Content-Type: application/json
```
```json
{
    "name": "skillset name",
    "skills": 
    [
      {
        "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
        "categories": [ "Organization" ],
        "defaultLanguageCode": "en",
        "inputs": [
          {
            "name": "text", "source": "/document/content"
          }
        ],
        "outputs": [
          {
            "name": "organizations", "targetName": "organizations"
          }
        ]
      }
    ],
    "cognitiveServices": {
        "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
        "description": "mycogsvcs",
        "key": "<your key goes here>"
    }
}
```

## <a name="example-estimate-costs"></a>Exemple : Estimer les coûts

Pour estimer les coûts associés à l’indexation de recherche cognitive, commencez par vous représenter à quoi ressemble un document moyen afin de pouvoir donner une approximation. Par exemple, vous pourriez faire une approximation :

+ 1 000 fichiers PDF.
+ Six pages par fichier.
+ Une image par page (6 000 images).
+ 3 000 caractères par page.

Supposons un pipeline consistant en un craquage de document PDF, une extraction d’image et de texte, une reconnaissance optique de caractères (OCR) d’images, et une reconnaissance d’entités d’organisations.

Les prix indiqués dans cet article sont hypothétiques. Ils sont utilisés pour illustrer le processus d’estimation. Vos coûts peuvent être inférieurs. Pour obtenir les prix réels des transactions, voir [Tarification Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services).

1. Pour le décodage de documents avec contenu de texte et d’image, l’extraction de texte est actuellement gratuite. Pour 6 000 images, supposez un prix de 1 USD pour chaque tranche de 1 000 images extraites. Cela revient à un coût de 6,00 USD pour cette étape.

2. Pour la reconnaissance optique de caractères (OCR) de 6 000 images en anglais, la qualification cognitive OCR utilise le meilleur algorithme (DescribeText). À raison de 2,50 $ par lot de 1 000 images à analyser, cette étape vous coûterait 15 $.

3. Pour l’extraction d’entité, vous auriez un total de trois enregistrements texte par page. Chaque enregistrement contient 1 000 caractères. Trois enregistrements texte par page multipliés par 6 000 pages équivalent à 18 000 enregistrements texte. À raison de 2 $ par lot de 1 000 enregistrements texte, cette étape vous coûterait 36 $.

En additionnant tout cela, l’ingestion de 1 000 documents PDF de ce type avec l’ensemble de qualifications décrit vous reviendrait à environ 57 USD.

## <a name="next-steps"></a>Étapes suivantes
+ [Page de tarification de Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Créer un jeu de compétences (REST)](/rest/api/searchservice/create-skillset)
+ [Guide pratique pour mapper des champs enrichis](cognitive-search-output-field-mapping.md)