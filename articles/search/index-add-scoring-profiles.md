---
title: Améliorer le classement de recherche avec des profils de score
titleSuffix: Azure Cognitive Search
description: Améliorer le classement par ordre de priorité des scores des résultats de Recherche cognitive Azure en ajoutant des profils de score.
manager: nitinme
author: shmed
ms.author: ramero
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/06/2020
ms.openlocfilehash: 97797e309c32c6ea996d5ae1901b9a266a683173
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91537631"
---
# <a name="add-scoring-profiles-to-an-azure-cognitive-search-index"></a>Ajouter des profils de score à un index Recherche cognitive Azure

Un *scoring* détermine un score de recherche pour chaque élément dans un jeu de résultats classés par rang. Un score de recherche est attribué à chaque élément d'un jeu de résultats de recherche. Ils sont ensuite classés du rang le plus élevé au rang le plus bas.

 Le service Recherche cognitive Azure utilise un score par défaut pour calculer un score initial, mais vous pouvez personnaliser le calcul à l'aide d'un *profil de score*. Les profils de score vous permettent de mieux contrôler le classement d'éléments dans des résultats de recherche. Par exemple, vous pouvez privilégier des éléments en fonction de leur revenu potentiel, promouvoir des éléments plus récents, voire en favoriser d'autres restés trop longtemps en stock.  

 Le segment vidéo suivant permet d’accéder rapidement au fonctionnement des profils de score dans Recherche cognitive Azure.
 
> [!VIDEO https://www.youtube.com/embed/Y_X6USgvB1g?version=3&start=463&end=970]

## <a name="scoring-profile-definitions"></a>Définitions de profil de score

 Un profil de score fait partie de la définition d'index, composée de champs, fonctions et paramètres pondérés.  

 Pour vous donner une idée de l'apparence d'un profil de score, l'exemple suivant présente un simple profil « géographique ». Celui-ci privilégie les éléments dont le champ **hotelName** contient le terme recherché. Il utilise également la fonction `distance` pour favoriser les éléments qui se situent dans un rayon de dix kilomètres par rapport à l'emplacement actuel. Si quelqu'un effectue une recherche sur le terme « inn » dans un rayon de 10 km par rapport à la position actuelle, les documents d'hôtels dont le nom contient cette chaîne de caractères apparaissent en tête des résultats de la recherche.  


```json
"scoringProfiles": [
  {  
    "name":"geo",
    "text": {  
      "weights": {  
        "hotelName": 5
      }                              
    },
    "functions": [
      {  
        "type": "distance",
        "boost": 5,
        "fieldName": "location",
        "interpolation": "logarithmic",
        "distance": {
          "referencePointParameter": "currentLocation",
          "boostingDistance": 10
        }                        
      }                                      
    ]                     
  }            
]
```  


 Pour utiliser ce profil de score, votre requête est formulée de façon à spécifier le profil de la chaîne de caractères de requête. Dans la requête ci-dessous, notez la présence du paramètre de requête `scoringProfile=geo`.  

```  
GET /indexes/hotels/docs?search=inn&scoringProfile=geo&scoringParameter=currentLocation--122.123,44.77233&api-version=2020-06-30 
```  

 Cette requête effectue une recherche du terme « inn », puis transmet l'emplacement actuel. Notez que cette requête inclut d’autres paramètres, tels que `scoringParameter`. Les paramètres de requête sont décrits dans [Recherche dans des documents &#40;API REST de Recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents).  

 Pour voir un exemple plus détaillé de profil de score, cliquez sur [Exemple](#bkmk_ex)  

## <a name="what-is-default-scoring"></a>Qu'est-ce qu'un calcul de score par défaut ?  
 Un calcul de score détermine un score de recherche pour chaque élément dans un jeu de résultats classés par rang. Un score de recherche est attribué à chaque élément d'un jeu de résultats de recherche. Ils sont ensuite classés du rang le plus élevé au rang le plus bas. Les éléments dont les scores sont plus élevés sont renvoyés à l'application. Par défaut, il s'agit des 50 premiers éléments, mais le paramètre `$top` vous permet de définir le renvoi d'un nombre supérieur ou inférieur d'éléments (jusqu'à 1 000 par réponse).  

Le score de recherche est calculé sur la base de propriétés statistiques des données et de la requête. Recherche cognitive Azure trouve les documents qui contiennent (en totalité ou en partie, selon le `searchMode`) les termes recherchés indiqués dans la chaîne de requête, en favorisant les documents qui contiennent de nombreuses occurrences du terme recherché. Le score de recherche augmente davantage si le terme est rare dans l’index de données, mais courant au sein du document. La base de cette approche de la pertinence du calcul est appelée [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) ou Term Frequency-Inverse Document Frequency (fréquence de terme-fréquence inverse de document).  

 En supposant qu'il n'y a pas de tri personnalisé, les résultats sont classés par score de recherche avant d'être renvoyés à l'application appelante. Si la valeur $top n'est pas spécifiée, les 50 éléments dont le score de recherche est le plus élevé sont renvoyés.  

 Des valeurs de score de recherche peuvent être répétées dans un jeu de résultats. Par exemple, vous pouvez avoir dix éléments dont le score est 1,2, vingt éléments dont le score est 1,0, et vingt éléments dont le score est 0,5. Quand plusieurs correspondances ont le même score de recherche, le classement des éléments ayant le même score n'est ni défini ni stable. Si vous exécutez de nouveau la requête, il se peut que des éléments changent de position. Si deux éléments ont un score identique, il est impossible de prédire celui qui apparaîtra en première position.  

## <a name="when-to-add-scoring-logic"></a>Quand ajouter une logique de notation 
 Quand le classement par défaut produit des résultats trop éloignés de vos objectifs, vous devez créer un ou plusieurs profils de calcul de score. Par exemple, vous pouvez décider que la pertinence de la recherche doit privilégier les éléments récemment ajoutés. De même, il se peut qu'un champ affiche une marge bénéficiaire, ou un autre un revenu potentiel. La volonté de privilégier les correspondances qui génèrent des bénéfices pour votre entreprise peut être un facteur important dans la décision d'utiliser des profils de calcul de score.  

 Un classement basé sur la pertinence peut également être mis en œuvre par l'intermédiaire de profils de calcul de score. Songez aux pages de résultats de recherche que vous avez utilisées par le passé, qui vous permettaient de trier par prix, date, évaluation ou pertinence. Dans Recherche cognitive Azure, les profils de calcul de score déterminent l'option « pertinence ». Vous contrôlez la définition de la pertinence en fonction de vos objectifs et du type d'expérience de recherche que vous souhaitez.  

##  <a name="example"></a><a name="bkmk_ex"></a> Exemple  
 Comme indiqué précédemment, un calcul de score personnalisé est mis en œuvre à l'aide d'un ou de plusieurs profils de calcul de score définis dans un schéma d'index.  

 Cet exemple montre le schéma d'un index comprenant deux profils de calcul de score (`boostGenre`, `newAndHighlyRated`). Toute requête sur cet index qui comprend un profil comme paramètre de requête utilise le profil pour évaluer le jeu de résultats.  

```json
{  
  "name": "musicstoreindex",  
  "fields": [  
    { "name": "key", "type": "Edm.String", "key": true },  
    { "name": "albumTitle", "type": "Edm.String" },  
    { "name": "albumUrl", "type": "Edm.String", "filterable": false },  
    { "name": "genre", "type": "Edm.String" },  
    { "name": "genreDescription", "type": "Edm.String", "filterable": false },  
    { "name": "artistName", "type": "Edm.String" },  
    { "name": "orderableOnline", "type": "Edm.Boolean" },  
    { "name": "rating", "type": "Edm.Int32" },  
    { "name": "tags", "type": "Collection(Edm.String)" },  
    { "name": "price", "type": "Edm.Double", "filterable": false },  
    { "name": "margin", "type": "Edm.Int32", "retrievable": false },  
    { "name": "inventory", "type": "Edm.Int32" },  
    { "name": "lastUpdated", "type": "Edm.DateTimeOffset" }  
  ],  
  "scoringProfiles": [  
    {  
      "name": "boostGenre",  
      "text": {  
        "weights": {  
          "albumTitle": 1.5,  
          "genre": 5,  
          "artistName": 2  
        }  
      }  
    },  
    {  
      "name": "newAndHighlyRated",  
      "functions": [  
        {  
          "type": "freshness",  
          "fieldName": "lastUpdated",  
          "boost": 10,  
          "interpolation": "quadratic",  
          "freshness": {  
            "boostingDuration": "P365D"  
          }  
        },  
        {
          "type": "magnitude",  
          "fieldName": "rating",  
          "boost": 10,  
          "interpolation": "linear",  
          "magnitude": {  
            "boostingRangeStart": 1,  
            "boostingRangeEnd": 5,  
            "constantBoostBeyondRange": false  
          }  
        }  
      ]  
    }  
  ],  
  "suggesters": [  
    {  
      "name": "sg",  
      "searchMode": "analyzingInfixMatching",  
      "sourceFields": [ "albumTitle", "artistName" ]  
    }  
  ]   
}  
```  

## <a name="workflow"></a>Workflow  
 Pour mettre en œuvre un calcul de score personnalisé, ajoutez un profil de calcul de score au schéma qui définit l'index. Vous pouvez avoir jusqu’à 100 profils de calcul de score au sein d’un même index (consultez [Limites de service](search-limits-quotas-capacity.md)), mais vous ne pouvez spécifier qu’un seul profil à la fois dans une requête donnée.  

 Commencez par le [Modèle](#bkmk_template) fourni dans cette rubrique.  

 Donnez-lui un nom. Les profils de calcul de score sont facultatifs mais, si vous en ajoutez un, le nom est obligatoire. Veillez à respecter les conventions d'affectation des noms pour les champs (commencer par une lettre, en évitant les caractères spéciaux et les mots réservés). Consultez [Règles d'attribution de noms &#40;Recherche cognitive Azure&#41; ](/rest/api/searchservice/naming-rules) pour obtenir la liste complète.  

 Le corps du profil de calcul de score est construit à partir de champs et de fonctions pondérés.  

|||  
|-|-|  
|**Pondérations**|Spécifiez les paires nom-valeur qui affectent une pondération relative à un champ. Dans l’[exemple](#bkmk_ex), les valeurs de pondération des champs albumTitle, genre et artistName sont respectivement 1,5, 5 et 2. Pourquoi la pondération du champ genre est-elle beaucoup plus élevée que celle des autres champs ? Si la recherche est effectuée sur des données relativement homogènes (comme c'est le cas du « genre » dans le `musicstoreindex`), il se peut que vous ayez besoin d'une variance plus importante dans les pondérations relatives. Par exemple, dans le `musicstoreindex`, « rock » apparaît à la fois comme genre et dans des descriptions de genre formulées de façon identique. Si vous souhaitez que le genre ait une pondération plus élevée que la description du genre, la pondération relative du champ Genre doit être sensiblement plus importante.|  
|**Fonctions**|Utilisées lorsque des calculs supplémentaires sont nécessaires dans des contextes spécifiques. Les valeurs valides sont `freshness`, `magnitude`, `distance` et `tag`. Chaque fonction a des paramètres qui sont uniques à cette dernière.<br /><br /> -   `freshness` doit être utilisée pour privilégier des résultats en fonction de la nouveauté ou de l’ancienneté d’un élément. Cette fonction peut être utilisée uniquement avec des champs `datetime` (edm.DataTimeOffset). Notez que l’attribut `boostingDuration` est utilisé uniquement avec la fonction `freshness`.<br />-   `magnitude` doit être utilisée pour privilégier des résultats en fonction de l’importance ou de la faiblesse d’une valeur numérique. Parmi les scénarios qui appellent cette fonction figurent la valorisation de la marge bénéficiaire, du prix le plus élevé, du prix le plus bas ou du nombre de téléchargements. Cette fonction peut être utilisée uniquement avec des champs double et integer.<br />     Pour la fonction `magnitude`, vous pouvez inverser la plage (de la valeur la plus élevée à la valeur la plus basse) si vous souhaitez inverser le schéma (par exemple, pour privilégier des éléments dont le prix est plus bas par rapport aux éléments dont le prix est plus élevé). Dans une gamme de prix allant de $100USD à $1USD, vous devez définir `boostingRangeStart` sur 100 et `boostingRangeEnd` sur 1 pour privilégier les éléments dont le prix est plus bas.<br />-   `distance` doit être utilisée pour privilégier des résultats en fonction de la proximité ou de l’emplacement géographique. Cette fonction peut être utilisée uniquement avec des champs `Edm.GeographyPoint` .<br />-   `tag` doit être utilisée pour privilégier des résultats en fonction de balises communes entre des documents et des requêtes de recherche. Cette fonction peut être utilisée uniquement avec les champs `Edm.String` et `Collection(Edm.String)`.<br /><br /> **Règles d'utilisation des fonctions**<br /><br /> Le type de fonction (`freshness`, `magnitude`, `distance`), `tag` doit être en lettres minuscules.<br /><br /> Les fonctions ne peuvent pas contenir de valeurs null ou vides. En particulier, si vous incluez la valeur fieldname, vous devez la spécifier.<br /><br /> Les fonctions ne peuvent être appliquées qu'à des champs filtrables. Consultez [Créer un index &#40;API REST de Recherche cognitive Azure&#41;](/rest/api/searchservice/create-index) pour plus d’informations sur les champs filtrables.<br /><br /> Vous ne pouvez pas appliquer de fonctions à des champs définis dans la collection de champs d'un index.|  

 Une fois l'index défini, générez-le en chargeant le schéma d'index, puis des documents. Consultez [Créer un index &#40;API REST de Recherche cognitive Azure&#41;](/rest/api/searchservice/create-index) et [Ajouter, mettre à jour ou supprimer des documents &#40;API REST de Recherche cognitive Azure&#41; ](/rest/api/searchservice/addupdate-or-delete-documents) pour obtenir des instructions relatives à ces opérations. Une fois l'index généré, vous disposez d'un profil de calcul de score fonctionnel qui opère avec vos données de recherche.  

##  <a name="template"></a><a name="bkmk_template"></a> Modèle  
 Cette section présente la syntaxe et le modèle de profils de calcul de score. Pour obtenir la description des attributs, consultez [Référence des attributs d'index](#bkmk_indexref) dans la section suivante.  

```  
. . .   
"scoringProfiles": [  
  {   
    "name": "name of scoring profile",   
    "text": (optional, only applies to searchable fields) {   
      "weights": {   
        "searchable_field_name": relative_weight_value (positive #'s),   
        ...   
      }   
    },   
    "functions": (optional) [  
      {   
        "type": "magnitude | freshness | distance | tag",   
        "boost": # (positive number used as multiplier for raw score != 1),   
        "fieldName": "...",   
        "interpolation": "constant | linear (default) | quadratic | logarithmic",   

        "magnitude": {
          "boostingRangeStart": #,   
          "boostingRangeEnd": #,   
          "constantBoostBeyondRange": true | false (default)
        }  

        // ( - or -)  

        "freshness": {
          "boostingDuration": "..." (value representing timespan over which boosting occurs)   
        }  

        // ( - or -)  

        "distance": {
          "referencePointParameter": "...", (parameter to be passed in queries to use as reference location)   
          "boostingDistance": # (the distance in kilometers from the reference location where the boosting range ends)   
        }   

        // ( - or -)  

        "tag": {
          "tagsParameter":  "..."(parameter to be passed in queries to specify a list of tags to compare against target field)   
        }
      }
    ],   
    "functionAggregation": (optional, applies only when functions are specified) "sum (default) | average | minimum | maximum | firstMatching"   
  }   
],   
"defaultScoringProfile": (optional) "...",   
. . .  
```  

##  <a name="index-attributes-reference"></a><a name="bkmk_indexref"></a> Référence des attributs d'index  

> [!NOTE]  
>  Vous pouvez appliquer une fonction de calcul de score uniquement à des champs filtrables.  

|Attribut|Description|  
|---------------|-----------------|  
|`name`|Obligatoire. Nom du profil de calcul de score. Il suit les conventions d'affectation de noms applicables aux champs. Il doit commencer par une lettre, ne peut pas contenir les signes point, deux-points ou @, et ne peut pas commencer par l'expression « azureSearch » (avec respect de la casse).|  
|`text`|Contient la propriété weights.|  
|`weights`|facultatif. Contient des paires nom-valeur spécifiant chacune un nom de champ et une pondération relative. La pondération relative doit être un entier positif ou un nombre à virgule flottante.<br /><br /> Les pondérations permettent d’indiquer l’importance d’un champ pouvant faire l’objet d’une recherche par rapport à d’autres.|  
|`functions`|facultatif. Vous pouvez appliquer une fonction de calcul de score uniquement à des champs filtrables.|  
|`type`|Obligatoire pour les fonctions de calcul de score. Indique le type de fonction à utiliser. Les valeurs autorisées sont magnitude, freshness, distance et tag. Vous pouvez inclure plusieurs fonctions dans chaque profil de calcul de score. Le nom de fonction doit être en lettres minuscules.|  
|`boost`|Obligatoire pour les fonctions de calcul de score. Nombre positif utilisé comme multiplicateur pour le score brut. Il ne peut pas être égal à 1.|  
|`fieldname`|Obligatoire pour les fonctions de calcul de score. Vous pouvez appliquer une fonction de calcul de score uniquement à des champs faisant partie de la collection de champs de l'index, et qui sont filtrables. De plus, chaque type de fonction fait l'objet de restrictions supplémentaires (la valeur freshness est utilisée avec des champs datetime, la valeur amplitude avec des champs integer ou double, et la valeur distance avec des champs location). Vous pouvez spécifier un seul champ par définition de fonction. Par exemple, pour utiliser une valeur magnitude deux fois dans le même profil, vous devez inclure deux définitions de magnitude, une pour chaque champ.|  
|`interpolation`|Obligatoire pour les fonctions de calcul de score. Définit la pente pour laquelle la valorisation du score augmente, du début à la fin de la plage. Les valeurs autorisées sont Linear (par défaut), Constant, Quadratic et Logarithmic. Consultez la rubrique [Définition d’interpolations](#bkmk_interpolation) pour plus d’informations.|  
|`magnitude`|La fonction de calcul de score magnitude est utilisée pour modifier des classements en fonction de la plage de valeurs pour un champ numérique. Voici quelques exemples d’utilisation courants :<br /><br /> -   **Évaluations :** modifiez le calcul de score sur la base de la valeur figurant dans le champ « Évaluation ». Lorsque deux éléments sont pertinents, l’élément associé à l’évaluation la plus élevée s’affiche en premier.<br />-   **Marge :** lorsque deux documents sont pertinents, un détaillant peut souhaiter privilégier le document présentant les marges les plus élevées.<br />-   **Nombres de clics :** pour les applications qui suivent les actions de clic sur des produits ou des pages, vous pouvez utiliser la fonction magnitude pour favoriser les éléments qui drainent le plus de trafic.<br />-   **Nombres de téléchargements :** pour les applications qui suivent les téléchargements, la fonction magnitude permet de favoriser les éléments les plus téléchargés.|  
|`magnitude` &#124; `boostingRangeStart`|Définit la valeur de début de la plage sur la base de laquelle les scores de magnitude sont calculés. La valeur doit être un entier ou un nombre à virgule flottante. Pour des évaluations de 1 à 4, il s'agit de 1. Pour des marges de plus de 50 %, il s'agit de 50.|  
|`magnitude` &#124; `boostingRangeEnd`|Définit la valeur de fin de la plage sur laquelle les scores de magnitude sont calculés. La valeur doit être un entier ou un nombre à virgule flottante. Pour les évaluations de 1 à 4, il s'agit de 4.|  
|`magnitude` &#124; `constantBoostBeyondRange`|Les valeurs autorisées sont true ou false (par défaut). Quand la valeur est true, la valorisation complète continue de s'appliquer aux documents dont la valeur pour le champ cible est supérieure à la limite supérieure de la plage. Quand la valeur est false, la valorisation de cette fonction ne s'applique pas aux documents dont la valeur pour le champ cible se situe en dehors de la plage.|  
|`freshness`|La fonction de calcul de score freshness permet de modifier les scores de classement d'éléments en fonction des valeurs des champs `DateTimeOffset`. Par exemple, un élément dont la date est plus récente peut être classé plus haut que des éléments plus anciens.<br /><br /> Il est également possible de classer les éléments tels que les événements de calendrier comportant des dates ultérieures afin que les éléments plus proches de la date du jour soient classés plus haut que les éléments plus éloignés dans le temps.<br /><br /> Dans la version de service actuelle, une extrémité de la plage doit être définie sur l'heure réelle. L’autre extrémité est une heure dans le passé selon l’attribut `boostingDuration`. Pour privilégier une plage d’heures à venir, utilisez un attribut `boostingDuration` négatif.<br /><br /> La vitesse à laquelle la valorisation passe d'une plage maximale à une plage minimale est déterminée par l'interpolation appliquée au profil de calcul de score (voir la figure ci-dessous). Pour inverser le facteur de valorisation appliqué, choisissez un facteur inférieur à 1.|  
|`freshness` &#124; `boostingDuration`|Définit une période d'expiration après laquelle la valorisation s'arrête pour un document spécifique. Pour plus d’informations sur la syntaxe et des exemples, consultez [Set boostingDuration](#bkmk_boostdur) dans la section suivante.|  
|`distance`|La fonction de calcul de score à distance est utilisée pour affecter le score de documents sur la base de leur proximité ou de l'éloignement par rapport à un emplacement géographique de référence. L’emplacement de référence est indiqué comme partie intégrante de la requête dans un paramètre (à l’aide de l'option de chaîne `scoringParameterquery` ) en tant qu’argument lon,lat.|  
|`distance` &#124; `referencePointParameter`|Paramètre à transmettre dans des requêtes, à utiliser comme emplacement de référence. `scoringParameter` est un paramètre de requête. Les paramètres de requête sont décrits dans [Recherche dans des documents &#40;API REST de Recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents).|  
|`distance` &#124; `boostingDistance`|Nombre indiquant la distance en kilomètres par rapport à l'emplacement de référence où la valorisation se termine.|  
|`tag`|La fonction de calcul de score de balises est utilisée pour affecter le score de documents sur la base de balises dans des documents et des requêtes de recherche. Les documents contenant des balises communes avec la requête de recherche seront privilégiés. Les balises pour la requête de recherche sont fournies en tant que paramètre de calcul de score dans chaque requête de recherche (à l’aide de l'option de chaîne `scoringParameterquery`).|  
|`tag` &#124; `tagsParameter`|Paramètre à transmettre dans des requêtes pour spécifier des balises pour une requête spécifique. `scoringParameter` est un paramètre de requête. Les paramètres de requête sont décrits dans [Recherche dans des documents &#40;API REST de Recherche cognitive Azure&#41;](/rest/api/searchservice/Search-Documents).|  
|`functionAggregation`|facultatif. S'applique uniquement quand des fonctions sont spécifiées. Les valeurs autorisées sont les suivantes : sum (par défaut), average, minimum, maximum et firstMatching. Un score de recherche est une valeur unique calculée à partir de plusieurs variables, notamment plusieurs fonctions. Cet attribut indique comment les valorisations de toutes les fonctions sont combinées en une valorisation agrégée qui est ensuite appliquée au score du document de base. Le score de base dépend de la valeur [tf-idf](http://www.tfidf.com/) calculée à partir du document et de la requête de recherche.|  
|`defaultScoringProfile`|Lors de l'exécution d'une requête de recherche, le calcul de score par défaut est utilisé ([tf-idf](http://www.tfidf.com/) uniquement) si aucun profil de calcul de score n'est spécifié.<br /><br /> Un nom de profil de calcul de score par défaut peut être défini ici de façon à ce que Recherche cognitive Azure utilise ce profil quand aucun profil spécifique n'est fourni dans la requête de recherche.|  

##  <a name="set-interpolations"></a><a name="bkmk_interpolation"></a>Définition d'interpolations  
 Les interpolations vous permettent de définir la forme de la pente utilisée pour calculer les scores. Le score allant d'élevé à faible, la pente est toujours décroissante, mais l’interpolation détermine la courbe de la pente descendante. Les interpolations utilisables sont les suivantes :  

| Interpolation | Description |  
|-|-|  
|`linear`|pour les éléments qui s’inscrivent dans la plage de max à min, la valorisation appliquée décroît de manière constante. L'interpolation de type Linear est l'interpolation par défaut pour un profil de calcul de score.|  
|`constant`|Pour les éléments qui s'inscrivent dans la plage du début à la fin, une valorisation constante est appliquée aux résultats du classement.|  
|`quadratic`|Par rapport à une interpolation de type Linear dont la valorisation décroît de façon constante, une interpolation de type Quadratic décroît initialement plus lentement, puis, lorsqu’elle approche de la plage de fin, beaucoup plus rapidement. Cette option d'interpolation n'est pas autorisée dans les fonctions de calcul de score de balises.|  
|`logarithmic`|Par rapport à une interpolation de type Linear dont la valorisation décroît de façon constante, une interpolation de type Logarithmic décroît initialement plus rapidement, puis, lorsqu’elle approche de la plage de fin, beaucoup plus lentement. Cette option d'interpolation n'est pas autorisée dans les fonctions de calcul de score de balises.|  

 ![Lignes constantes, linéaires, quadratiques, log10 sur le graphique](media/scoring-profiles/azuresearch_scorefunctioninterpolationgrapht.png "AzureSearch_ScoreFunctionInterpolationGrapht")  

##  <a name="set-boostingduration"></a><a name="bkmk_boostdur"></a> Définition de boostingDuration  
 `boostingDuration` est un attribut de la fonction `freshness`. Il permet de définir une période d'expiration après laquelle la valorisation s'arrête pour un document spécifique. Par exemple, pour valoriser une ligne de produits ou une marque pendant une période promotionnelle de 10 jours, vous spécifiez la période de 10 jours en tant que « P10D » pour les documents correspondants.  

 `boostingDuration` doit être au format « dayTimeDuration » XSD (sous-ensemble limité d'une valeur de durée ISO 8601). Le modèle est le suivant : « P[nD][T[nH][nM][nS]] ».  

 Le tableau suivant fournit plusieurs exemples.  

|Duration|boostingDuration|  
|--------------|----------------------|  
|1 jour|« P1D »|  
|2 jours et 12 heures|« P2DT12H »|  
|15 minutes|« PT15M »|  
|30 jours, 5 heures 10 minutes, et 6 334 secondes|« P30DT5H10M6.334S »|  

 Pour plus d’exemples, consultez [Schéma XML : types de données (site web W3.org)](https://www.w3.org/TR/xmlschema11-2/#dayTimeDuration).  

## <a name="see-also"></a>Voir aussi  

+ [Référence d’API REST](/rest/api/searchservice/)
+ [Créer une API d’index](/rest/api/searchservice/create-index)
+ [Kit de développement logiciel (SDK) de Recherche cognitive Azure .NET](/dotnet/api/overview/azure/search?)