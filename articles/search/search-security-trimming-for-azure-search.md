---
title: Filtres de sécurité pour le filtrage des résultats
titleSuffix: Azure Cognitive Search
description: Privilèges de sécurité au niveau du document pour les résultats de Recherche cognitive Azure, utilisation de filtres de sécurité et d’identités.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/04/2020
ms.openlocfilehash: 8562fd1afaa01e362bd6d95fd4dcf90cf3145c5a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88928521"
---
# <a name="security-filters-for-trimming-results-in-azure-cognitive-search"></a>Filtres de sécurité pour le filtrage des résultats dans Recherche cognitive Azure

Vous pouvez appliquer des filtres de sécurité pour filtrer les résultats de recherche dans Recherche cognitive Azure en fonction de l’identité de l’utilisateur. Cette expérience de recherche compare généralement l’identité de la personne qui lance la recherche à un champ contenant les principaux qui disposent d’autorisations d’accès au document. Quand une correspondance est trouvée, l’utilisateur ou le principal (comme un groupe ou un rôle) a accès à ce document.

Pour mettre en place le filtrage de sécurité, une méthode consiste à utiliser une disjonction complexe d’expressions d’égalité. Par exemple : `Id eq 'id1' or Id eq 'id2'`, etc. Cette approche est sujette aux erreurs et difficile à gérer. De plus, si la liste contient des centaines voire des milliers de valeurs, elle ralentit le temps de réponse de plusieurs secondes. 

Une approche plus simple et plus rapide consiste à utiliser la fonction `search.in`. En utilisant `search.in(Id, 'id1, id2, ...')` à la place d’une expression d’égalité, vous pouvez obtenir des temps de réponse inférieurs à une seconde.

Cet article explique les étapes à suivre pour mettre en place le filtrage de sécurité :
> [!div class="checklist"]
> * Créer un champ qui contient les identificateurs de principal 
> * Envoyer (push) ou mettre à jour les documents existants avec les identificateurs de principal appropriés
> * Émettre une demande de recherche avec `search.in` `filter`

>[!NOTE]
> Le processus de récupération des identificateurs de principal n’est pas abordé dans ce document. Obtenez-le auprès de votre fournisseur de services d’identité.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un [abonnement Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F), du [service Recherche cognitive Azure](search-create-service-portal.md) et d’un [index](search-what-is-an-index.md).  

## <a name="create-security-field"></a>Créer le champ de sécurité

Vos documents doivent inclure un champ qui spécifie les groupes disposant d’autorisations d’accès. Ces informations constituent les critères de filtre par rapport auxquels les documents sont sélectionnés ou non dans le jeu de résultats retourné à l’émetteur.
Imaginons que nous disposons d’un index de fichiers sécurisés et qu’un ensemble différent d’utilisateurs a accès à chaque fichier.

1. Ajoutez le champ `group_ids` (vous pouvez choisir n’importe quel nom ici) comme `Collection(Edm.String)`. Vérifiez que le champ a un attribut `filterable` défini avec la valeur `true` pour que les résultats de la recherche soient filtrés en fonction de l’accès dont dispose l’utilisateur. Par exemple, si vous définissez le champ `group_ids` avec la valeur `["group_id1, group_id2"]` pour le document ayant comme `file_name` « secured_file_b », seuls les utilisateurs qui appartiennent à l’ID de groupe « group_id1 » ou « group_id2 » ont accès en lecture au fichier.
   
   Vérifiez que l’attribut `retrievable` du champ a la valeur `false` pour qu’il ne soit pas retourné dans le cadre de la requête de recherche.

2. Pour les besoins de cet exemple, ajoutez également les champs `file_id` et `file_name`.  

    ```JSON
    {
        "name": "securedfiles",  
        "fields": [
            {"name": "file_id", "type": "Edm.String", "key": true, "searchable": false, "sortable": false, "facetable": false},
            {"name": "file_name", "type": "Edm.String"},
            {"name": "group_ids", "type": "Collection(Edm.String)", "filterable": true, "retrievable": false}
        ]
    }
    ```

## <a name="pushing-data-into-your-index-using-the-rest-api"></a>Envoi (push) des données à votre index à l’aide de l’API REST
  
Émettez une requête HTTP POST au point de terminaison de l’URL de votre index. Le corps de la requête HTTP est un objet JSON contenant les documents à ajouter :

```
POST https://[search service].search.windows.net/indexes/securedfiles/docs/index?api-version=2020-06-30  
Content-Type: application/json
api-key: [admin key]
```

Dans le corps de la requête, spécifiez le contenu de vos documents :

```JSON
{
    "value": [
        {
            "@search.action": "upload",
            "file_id": "1",
            "file_name": "secured_file_a",
            "group_ids": ["group_id1"]
        },
        {
            "@search.action": "upload",
            "file_id": "2",
            "file_name": "secured_file_b",
            "group_ids": ["group_id1", "group_id2"]
        },
        {
            "@search.action": "upload",
            "file_id": "3",
            "file_name": "secured_file_c",
            "group_ids": ["group_id5", "group_id6"]
        }
    ]
}
```

Si vous avez besoin de mettre à jour un document existant avec la liste des groupes, vous pouvez utiliser l’action `merge` ou `mergeOrUpload` :

```JSON
{
    "value": [
        {
            "@search.action": "mergeOrUpload",
            "file_id": "3",
            "group_ids": ["group_id7", "group_id8", "group_id9"]
        }
    ]
}
```

Pour obtenir des informations détaillées sur l’ajout ou la mise à jour de documents, lisez [Modifier des documents](/rest/api/searchservice/addupdate-or-delete-documents).
   
## <a name="apply-the-security-filter"></a>Appliquer le filtre de sécurité

Pour filtrer des documents en fonction de l’accès de `group_ids`, vous devez émettre une requête de recherche avec un filtre `group_ids/any(g:search.in(g, 'group_id1, group_id2,...'))`, où « group_id1, group_id2,... » sont les groupes auxquels l’émetteur de la requête de recherche appartient.
Ce filtre correspond à tous les documents dont le champ `group_ids` contient l’un des identificateurs donnés.
Pour obtenir des informations détaillées sur la recherche de documents à l’aide de Recherche cognitive Azure, lisez [Recherche de documents](/rest/api/searchservice/search-documents).
Notez que cet exemple montre comment lancer une recherche dans des documents à l’aide d’une requête POST.

Émettez la requête HTTP POST :

```
POST https://[service name].search.windows.net/indexes/securedfiles/docs/search?api-version=2020-06-30
Content-Type: application/json  
api-key: [admin or query key]
```

Spécifiez le filtre dans le corps de la requête :

```JSON
{
   "filter":"group_ids/any(g:search.in(g, 'group_id1, group_id2'))"  
}
```

Vous devez obtenir les documents où `group_ids` contient « group_id1 » ou « group_id2 ». En d’autres termes, vous obtenez les documents auxquels l’émetteur de la requête a accès en lecture.

```JSON
{
 [
   {
    "@search.score":1.0,
     "file_id":"1",
     "file_name":"secured_file_a",
   },
   {
     "@search.score":1.0,
     "file_id":"2",
     "file_name":"secured_file_b"
   }
 ]
}
```
## <a name="conclusion"></a>Conclusion

Vous venez de voir comment filtrer des résultats en fonction de l’identité de l’utilisateur et de la fonction `search.in()` de Recherche cognitive Azure. Vous pouvez utiliser cette fonction pour passer les identificateurs de principal de l'utilisateur demandeur et les mettre en correspondance avec les identificateurs de principal associés à chaque document cible. Quand une requête de recherche est traitée, la fonction `search.in` exclut les résultats de la recherche inaccessibles en lecture aux principaux de l’utilisateur. Les identificateurs de principal peuvent représenter des groupes de sécurité, des rôles ou même la propre identité de l’utilisateur.
 
## <a name="see-also"></a>Voir aussi

+ [Contrôle d’accès à Active Directory basé sur l’identité à l’aide des filtres Recherche cognitive Azure](search-security-trimming-for-azure-search-with-aad.md)
+ [Filtres dans la Recherche cognitive Azure](search-filters.md)
+ [Sécurité des données et contrôle d'accès dans les opérations de Recherche cognitive Azure](search-security-overview.md)