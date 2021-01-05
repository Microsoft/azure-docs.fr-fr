---
title: 'Démarrage rapide : Créer un index de recherche en Python'
titleSuffix: Azure Cognitive Search
description: Explique comment créer un index, charger des données et exécuter des requêtes à l’aide de Python, de notebooks et de la bibliothèque Azure.Documents.Search.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: quickstart
ms.date: 11/19/2020
ms.custom: devx-track-python
ms.openlocfilehash: 126fc69678148d4d478c96ff8d05f194c7e3d1b3
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861865"
---
# <a name="quickstart-create-an-azure-cognitive-search-index-in-python-using-jupyter-notebooks"></a>Démarrage rapide : Créer un index Recherche cognitive Azure en Python à l’aide de Jupyter Notebooks

> [!div class="op_single_selector"]
> * [Python](search-get-started-python.md)
> * [PowerShell (REST)](./search-get-started-powershell.md)
> * [C#](./search-get-started-dotnet.md)
> * [REST](search-get-started-rest.md)
> * [Portail](search-get-started-portal.md)
>

Générez une instance de Jupyter Notebook qui crée, charge et interroge un index Recherche cognitive Azure à l’aide de Python et de la [bibliothèque azure-search-documents](/python/api/overview/azure/search-documents-readme) dans le kit Azure SDK pour Python. Cet article décrit la procédure à suivre pour créer un notebook. Vous pouvez [également télécharger et exécuter un notebook Jupyter Python complet](https://github.com/Azure-Samples/azure-search-python-samples).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Les services et outils suivants sont indispensables dans ce guide de démarrage rapide.

* [Anaconda 3.x](https://www.anaconda.com/distribution/#download-section), qui fournit Python 3.x et Jupyter Notebook.

* [Package azure-search-documents](https://pypi.org/project/azure-search-documents/)

* [Créez un service Recherche cognitive Azure](search-create-service-portal.md) ou [recherchez un service existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) dans votre abonnement actuel. Vous pouvez utiliser le niveau gratuit pour ce démarrage rapide. 

## <a name="copy-a-key-and-url"></a>Copier une clé et une URL

Les appels REST requièrent l’URL du service et une clé d’accès et ce, sur chaque demande. Un service de recherche est créé avec les deux. Ainsi, si vous avez ajouté la Recherche cognitive Azure à votre abonnement, effectuez ce qui suit pour obtenir les informations nécessaires :

1. [Connectez-vous au portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez l’URL. Voici un exemple de point de terminaison : `https://mydemo.search.windows.net`.

1. Dans **Paramètres** > **Clés**, obtenez une clé d’administration pour avoir des droits d’accès complets sur le service. Il existe deux clés d’administration interchangeables, fournies pour assurer la continuité de l’activité au cas où vous deviez en remplacer une. Vous pouvez utiliser la clé primaire ou secondaire sur les demandes d’ajout, de modification et de suppression d’objets.

![Obtenir un point de terminaison et une clé d’accès HTTP](media/search-get-started-rest/get-url-key.png "Obtenir un point de terminaison et une clé d’accès HTTP")

Toutes les demandes nécessitent une clé API sur chaque demande envoyée à votre service. L’utilisation d’une clé valide permet d’établir, en fonction de chaque demande, une relation de confiance entre l’application qui envoie la demande et le service qui en assure le traitement.

## <a name="connect-to-azure-cognitive-search"></a>Se connecter à la Recherche cognitive Azure

Dans cette tâche, démarrez une instance de Jupyter Notebook, puis vérifiez que vous pouvez vous connecter à la Recherche cognitive Azure. Pour ce faire, demandez une liste d’index de votre service. Sur Windows avec Anaconda3, vous pouvez utiliser Anaconda Navigator pour lancer un notebook.

1. Créez un notebook Python3.

1. Dans la première cellule, chargez les bibliothèques à partir du kit Azure SDK pour Python, y compris [azure-search-documents](/python/api/azure-search-documents).

   ```python
    !pip install azure-search-documents --pre
    !pip show azure-search-documents

    import os
    from azure.core.credentials import AzureKeyCredential
    from azure.search.documents.indexes import SearchIndexClient 
    from azure.search.documents import SearchClient
    from azure.search.documents.indexes.models import (
        ComplexField,
        CorsOptions,
        SearchIndex,
        ScoringProfile,
        SearchFieldDataType,
        SimpleField,
        SearchableField
    )
   ```

1. Dans la deuxième cellule, entrez les éléments de requête qui seront des constantes sur chaque requête. Indiquez le nom de votre service de recherche, la clé API d’administration et la clé API de requête, copiés à l’étape précédente. Cette cellule configure également les clients que utiliserez pour des opérations spécifiques : [SearchIndexClient](/python/api/azure-search-documents/azure.search.documents.indexes.searchindexclient) pour créer un index et [SearchClient](/python/api/azure-search-documents/azure.search.documents.searchclient) pour interroger un index.

   ```python
    service_name = ["SEARCH_ENDPOINT - do not include search.windows.net"]
    admin_key = ["Cognitive Search Admin API Key"]

    index_name = "hotels-quickstart"

    # Create an SDK client
    endpoint = "https://{}.search.windows.net/".format(service_name)
    admin_client = SearchIndexClient(endpoint=endpoint,
                          index_name=index_name,
                          credential=AzureKeyCredential(admin_key))

    search_client = SearchClient(endpoint=endpoint,
                          index_name=index_name,
                          credential=AzureKeyCredential(admin_key))
   ```

1. Dans la troisième cellule, exécutez une opération delete_index pour effacer votre service de tous les index *hotels-quickstart* existants. La suppression de l’index vous permet de créer un autre index *hotels-quickstart* du même nom.

   ```python
    try:
        result = admin_client.delete_index(index_name)
        print ('Index', index_name, 'Deleted')
    except Exception as ex:
        print (ex)
   ```

1. Exécutez chaque étape.

## <a name="1---create-an-index"></a>1 – Créer un index

Les éléments requis d’un index sont un nom, une collection de champs et une clé. La collection de champs définit la structure d’un *document de recherche* logique, utilisé pour charger les données et retourner les résultats. 

Chaque champ a un nom, un type et des attributs qui déterminent la façon dont il est utilisé (par exemple, s’il permet d’effectuer une recherche en texte intégral, et s’il est filtrable ou récupérable dans des résultats de recherche). Dans un index, l’un des champs de type `Edm.String` doit être désigné comme *clé* pour l’identité du document.

Cet index est nommé « hotels-quickstart » et contient les définitions de champ que vous voyez ci-dessous. Il s’agit d’un sous-ensemble d’un [index Hotels](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/Hotels_IndexDefinition.JSON) plus grand utilisé dans d’autres procédures pas à pas. Nous l’avons volontairement tronqué dans ce démarrage rapide par souci de concision.

1. Dans la cellule suivante, collez l’exemple suivant dans une cellule pour fournir le schéma.

    ```python
    name = index_name
    fields = [
            SimpleField(name="HotelId", type=SearchFieldDataType.String, key=True),
            SearchableField(name="HotelName", type=SearchFieldDataType.String, sortable=True),
            SearchableField(name="Description", type=SearchFieldDataType.String, analyzer_name="en.lucene"),
            SearchableField(name="Description_fr", type=SearchFieldDataType.String, analyzer_name="fr.lucene"),
            SearchableField(name="Category", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),

            SearchableField(name="Tags", collection=True, type=SearchFieldDataType.String, facetable=True, filterable=True),

            SimpleField(name="ParkingIncluded", type=SearchFieldDataType.Boolean, facetable=True, filterable=True, sortable=True),
            SimpleField(name="LastRenovationDate", type=SearchFieldDataType.DateTimeOffset, facetable=True, filterable=True, sortable=True),
            SimpleField(name="Rating", type=SearchFieldDataType.Double, facetable=True, filterable=True, sortable=True),

            ComplexField(name="Address", fields=[
                SearchableField(name="StreetAddress", type=SearchFieldDataType.String),
                SearchableField(name="City", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),
                SearchableField(name="StateProvince", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),
                SearchableField(name="PostalCode", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),
                SearchableField(name="Country", type=SearchFieldDataType.String, facetable=True, filterable=True, sortable=True),
            ])
        ]
    cors_options = CorsOptions(allowed_origins=["*"], max_age_in_seconds=60)
    scoring_profiles = []
    suggester = [{'name': 'sg', 'source_fields': ['Tags', 'Address/City', 'Address/Country']}]
    ```

1. Dans une autre cellule, formulez la requête. Cette requête create_index cible la collection d’index de votre service de recherche et crée un [SearchIndex](/python/api/azure-search-documents/azure.search.documents.indexes.models.searchindex) basé sur le schéma d’index que vous avez fourni dans la cellule précédente.

   ```python
    index = SearchIndex(
        name=name,
        fields=fields,
        scoring_profiles=scoring_profiles,
        suggesters = suggester,
        cors_options=cors_options)

    try:
        result = admin_client.create_index(index)
        print ('Index', result.name, 'created')
    except Exception as ex:
        print (ex)
   ```

1. Exécutez chaque étape.

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2 – Charger des documents

Pour charger des documents, créez une collection de documents à l’aide d’une [action d’index](/python/api/azure-search-documents/azure.search.documents.models.indexaction) correspondant au type d’opération (charger, fusionner et charger, etc.). Les documents proviennent de [HotelsData](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/HotelsData_toAzureSearch.JSON) sur GitHub.

1. Dans une nouvelle cellule, fournissez quatre documents conformes au schéma d’index. Spécifiez une action de chargement pour chaque document.

    ```python
    documents = {
        "value": [
        {
        "@search.action": "upload",
        "HotelId": "1",
        "HotelName": "Secret Point Motel",
        "Description": "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
        "Description_fr": "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
        "Category": "Boutique",
        "Tags": [ "pool", "air conditioning", "concierge" ],
        "ParkingIncluded": "false",
        "LastRenovationDate": "1970-01-18T00:00:00Z",
        "Rating": 3.60,
        "Address": {
            "StreetAddress": "677 5th Ave",
            "City": "New York",
            "StateProvince": "NY",
            "PostalCode": "10022",
            "Country": "USA"
            }
        },
        {
        "@search.action": "upload",
        "HotelId": "2",
        "HotelName": "Twin Dome Motel",
        "Description": "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
        "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
        "Category": "Boutique",
        "Tags": [ "pool", "free wifi", "concierge" ],
        "ParkingIncluded": "false",
        "LastRenovationDate": "1979-02-18T00:00:00Z",
        "Rating": 3.60,
        "Address": {
            "StreetAddress": "140 University Town Center Dr",
            "City": "Sarasota",
            "StateProvince": "FL",
            "PostalCode": "34243",
            "Country": "USA"
            }
        },
        {
        "@search.action": "upload",
        "HotelId": "3",
        "HotelName": "Triple Landscape Hotel",
        "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel's restaurant services.",
        "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
        "Category": "Resort and Spa",
        "Tags": [ "air conditioning", "bar", "continental breakfast" ],
        "ParkingIncluded": "true",
        "LastRenovationDate": "2015-09-20T00:00:00Z",
        "Rating": 4.80,
        "Address": {
            "StreetAddress": "3393 Peachtree Rd",
            "City": "Atlanta",
            "StateProvince": "GA",
            "PostalCode": "30326",
            "Country": "USA"
            }
        },
        {
        "@search.action": "upload",
        "HotelId": "4",
        "HotelName": "Sublime Cliff Hotel",
        "Description": "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
        "Description_fr": "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
        "Category": "Boutique",
        "Tags": [ "concierge", "view", "24-hour front desk service" ],
        "ParkingIncluded": "true",
        "LastRenovationDate": "1960-02-06T00:00:00Z",
        "Rating": 4.60,
        "Address": {
            "StreetAddress": "7400 San Pedro Ave",
            "City": "San Antonio",
            "StateProvince": "TX",
            "PostalCode": "78216",
            "Country": "USA"
            }
        }
    ]
    }
    ```  

1. Dans une autre cellule, formulez la requête. Cette requête upload_documents cible la collection de documents de l’index hotels-quickstart, et envoie les documents fournis à l’étape précédente dans l’index Recherche cognitive.


   ```python
    try:
        result = search_client.upload_documents(documents=documents)
        print("Upload of new document succeeded: {}".format(result[0].succeeded))
    except Exception as ex:
        print (ex.message)
   ```

1. Exécutez chaque étape pour envoyer les documents à un index dans votre service de recherche.

## <a name="3---search-an-index"></a>3 – Rechercher dans un index

Cette étape vous montre comment interroger un index à l’aide de l’[API REST Rechercher des documents](/rest/api/searchservice/search-documents).

1. Pour cette opération, utilisez search_client. Cette requête exécute une recherche vide (`search=*`), qui retourne une liste non classée (score de recherche = 1.0) de documents arbitraires. Étant donné qu’il n’y a aucun critère, tous les documents sont inclus dans les résultats. Cette requête affiche uniquement deux des champs de chaque document. Elle ajoute également `include_total_count=True` pour obtenir le nombre total de documents (4) dans les résultats.

   ```python
    results =  search_client.search(search_text="*", include_total_count=True)

    print ('Total Documents Matching Query:', results.get_count())
    for result in results:
        print("{}: {}".format(result["HotelId"], result["HotelName"]))
   ```

1. La requête suivante ajoute des termes entiers à l’expression de recherche (« wifi »). Cette requête spécifie que les résultats contiennent uniquement les champs de l’instruction `select`. Le fait de limiter les champs de retour réduit la quantité de données renvoyées sur le réseau ainsi que la latence de recherche.

   ```python
    results =  search_client.search(search_text="wifi", include_total_count=True, select='HotelId,HotelName,Tags')

    print ('Total Documents Matching Query:', results.get_count())
    for result in results:
        print("{}: {}: {}".format(result["HotelId"], result["HotelName"], result["Tags"]))
   ```

1. Ensuite, appliquez une expression de filtre afin de retourner uniquement les hôtels ayant une évaluation supérieure à 4, triés par ordre décroissant.

   ```python
    results =  search_client.search(search_text="hotels", select='HotelId,HotelName,Rating', filter='Rating gt 4', order_by='Rating desc')

    for result in results:
        print("{}: {} - {} rating".format(result["HotelId"], result["HotelName"], result["Rating"]))
   ```

1. Ajoutez `search_fields` pour étendre la correspondance de requête à un champ unique.

   ```python
    results =  search_client.search(search_text="sublime", search_fields='HotelName', select='HotelId,HotelName')

    for result in results:
        print("{}: {}".format(result["HotelId"], result["HotelName"]))
   ```

1. Les facettes sont des étiquettes qui peuvent être utilisées pour composer une structure de navigation à facettes. Cette requête retourne des facettes et des nombres pour la catégorie.

   ```python
    results =  search_client.search(search_text="*", facets=["Category"])

    facets = results.get_facets()

    for facet in facets["Category"]:
        print("    {}".format(facet))
   ```

1. Dans cet exemple, recherchez un document spécifique en fonction de sa clé. Vous souhaiterez généralement retourner un document quand un utilisateur clique sur un document dans un résultat de recherche.

   ```python
    result = search_client.get_document(key="3")

    print("Details for hotel '3' are:")
    print("        Name: {}".format(result["HotelName"]))
    print("      Rating: {}".format(result["Rating"]))
    print("    Category: {}".format(result["Category"]))
   ```

1. Dans cet exemple, nous allons utiliser la fonction d’autocomplétion. Elle est généralement utilisée dans une zone de recherche afin de permettre l’autocomplétion des correspondances potentielles à mesure que l’utilisateur tape dans la zone de recherche.

   Lors de la création de l’index, un suggesteur nommé « sg » a également été créé dans le cadre de la requête. Une définition de suggesteur spécifie les champs qui peuvent être utilisés pour rechercher les correspondances potentielles aux requêtes de suggesteur. Dans cet exemple, ces champs sont « Tags », « Address/City », « Address/Country ». Pour simuler l’autocomplétion, passez les lettres « sa » en tant que chaîne partielle. La méthode d’autocomplétion de [SearchClient](/python/api/azure-search-documents/azure.search.documents.searchclient) renvoie les correspondances de termes potentielles.

   ```python
    search_suggestion = 'sa'
    results = search_client.autocomplete(search_text=search_suggestion, suggester_name="sg", mode='twoTerms')

    print("Autocomplete for:", search_suggestion)
    for result in results:
        print (result['text'])
   ```

## <a name="clean-up"></a>Nettoyer

Lorsque vous travaillez dans votre propre abonnement, il est judicieux à la fin d’un projet de déterminer si vous avez encore besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite. 

## <a name="next-steps"></a>Étapes suivantes

Par souci de simplification, ce démarrage rapide utilise une version abrégée de l’index Hotels. Vous pouvez créer la version complète pour essayer des requêtes plus intéressantes. Pour obtenir la version complète et les 50 documents, exécutez l’Assistant **Importation de données** en sélectionnant *hotels-sample* à partir des exemples de sources de données intégrés.

> [!div class="nextstepaction"]
> [Démarrage rapide : Créer un index dans le portail Azure](search-get-started-portal.md)
