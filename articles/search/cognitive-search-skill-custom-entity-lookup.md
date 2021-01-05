---
title: Compétence de recherche cognitive Recherche d'entité personnalisée
titleSuffix: Azure Cognitive Search
description: Extrayez différentes entités personnalisées du texte dans un pipeline de recherche cognitive Recherche cognitive Azure. Cette compétence est actuellement en préversion publique.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 5511551f240fe4fdd2f2aa3bc8a3a2615505f35f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88936110"
---
#     <a name="custom-entity-lookup-cognitive-skill-preview"></a>Compétence cognitive Recherche d'entité personnalisée (préversion)

> [!IMPORTANT] 
> Cette compétence est actuellement en préversion publique. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Il n’y a actuellement pas de prise en charge du portail ou du SDK .NET.

La compétence **Recherche d'entité personnalisée** recherche du texte dans une liste de mots et d'expressions personnalisée et définie par l'utilisateur. À l'aide de cette liste, elle étiquète tous les documents contenant des entités correspondantes. La compétence prend également en charge un degré de correspondance approximative qui peut être appliqué pour rechercher des correspondances similaires sans être rigoureusement exactes.  

Cette compétence n'est pas liée à une API Cognitive Services et peut être utilisée gratuitement pendant toute la durée de la préversion. Vous devez néanmoins [joindre une ressource Cognitive Services](./cognitive-search-attach-cognitive-services.md) pour passer outre la limite d'enrichissement quotidienne. La limite quotidienne s'applique à l'accès gratuit à Cognitive Services via Recherche cognitive Azure.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.CustomEntityLookupSkill 

## <a name="data-limits"></a>Limites de données
+ La taille maximale prise en charge pour les enregistrements d'entrée est de 256 Mo. Si vous devez subdiviser vos données avant de les envoyer à la compétence de recherche d'entité personnalisée, utilisez la [compétence Fractionnement de texte](cognitive-search-skill-textsplit.md).
+ La taille maximale prise en charge pour la table de définition des entités est de 10 Mo si elle est fournie à l’aide du paramètre *entitiesDefinitionUri*. 
+ Si les entités sont définies au format inline, à l'aide du paramètre *inlineEntitiesDefinition*, la taille maximale prise en charge est de 10 Ko.

## <a name="skill-parameters"></a>Paramètres de la compétence

Les paramètres respectent la casse.

| Nom du paramètre     | Description |
|--------------------|-------------|
| `entitiesDefinitionUri`    | Chemin d'accès à un fichier JSON ou CSV contenant tout le texte cible à comparer. Cette définition d'entité est lue au début de l'exécution d'un indexeur ; les mises à jour de ce fichier à mi-parcours ne seront pas réalisées avant l'exécution suivante. Cette configuration doit être accessible via HTTPS. Reportez-vous à la section Format de [définition d'entité personnalisée](#custom-entity-definition-format) ci-dessous pour en savoir plus sur le schéma CSV ou JSON attendu.|
|`inlineEntitiesDefinition` | Définitions d'entités JSON au format inline. Ce paramètre remplace le paramètre entitiesDefinitionUri s'il est présent. Un maximum de 10 Ko de configuration peut être fourni au format inline. Reportez-vous à la section [Définition d'entité personnalisée](#custom-entity-definition-format) ci-dessous pour en savoir plus sur le schéma JSON attendu. |
|`defaultLanguageCode` |    (Facultatif) Code langue du texte d'entrée utilisé pour « tokeniser » et délimiter le texte. Langues prises en charge : `da, de, en, es, fi, fr, it, ko, pt`. La langue par défaut est l'anglais (`en`). Si vous utilisez un format codelangue-codepays, seule la partie codelangue du format est utilisée.  |


## <a name="skill-inputs"></a>Entrées de la compétence

| Nom d’entrée      | Description                   |
|---------------|-------------------------------|
| `text`          | Texte à analyser.          |
| `languageCode`    | facultatif. La valeur par défaut est `"en"`.  |


## <a name="skill-outputs"></a>Sorties de la compétence


| Nom de sortie      | Description                   |
|---------------|-------------------------------|
| `entities` | Groupe d'objets contenant des informations sur les correspondances trouvées et les métadonnées associées. Chacune des entités identifiées peut contenir les champs suivants :  <ul> <li> *nom* : entité de niveau supérieur identifiée. L'entité représente la forme « normalisée ». </li> <li> *ID* :  identificateur unique de l'entité, tel que défini par l'utilisateur dans le « Format de définition d'entité personnalisée ».</li> <li> *description* : description de l'entité telle que définie par l'utilisateur dans le « Format de définition d'entité personnalisée ». </li> <li> *type :* type de l'entité tel que défini par l'utilisateur dans le « Format de définition d'entité personnalisée ».</li> <li> *subtype :* sous-type de l'entité, tel que défini par l'utilisateur dans le « Format de définition d'entité personnalisée ».</li>  <li> *matches* : collection qui décrit chacune des correspondances de cette entité sur le texte source. Chaque correspondance comportera les membres suivants : </li> <ul> <li> *text* : correspondance en texte brut issue du document source. </li> <li> *offset* : emplacement où la correspondance a été trouvée dans le texte. </li> <li> *length* :  longueur du texte de la correspondance. </li> <li> *matchDistance* : nombre de caractères séparant cette correspondance du nom ou de l'alias de l'entité d'origine.  </li> </ul> </ul>
  |

## <a name="custom-entity-definition-format"></a>Format de définition d'entité personnalisée

La liste des entités personnalisées peut être fournie à la compétence Recherche d'entité personnalisée de trois façons différentes. Vous pouvez fournir la liste dans un fichier .CSV, un fichier .JSON ou une définition inline dans le cadre de la définition de la compétence.  

Si le fichier de définition est un fichier .CSV ou .JSON, le chemin d'accès à celui-ci doit être fourni dans le paramètre *entitiesDefinitionUri*. Dans ce cas, le fichier est téléchargé une fois au début de chaque exécution de l'indexeur. Le fichier doit être accessible aussi longtemps que l'indexeur est censé s'exécuter. En outre, le fichier doit être encodé en UTF-8.

Si la définition est fournie au format inline, elle doit être fournie au format inline comme contenu du paramètre de compétence *inlineEntitiesDefinition*. 

### <a name="csv-format"></a>Format CSV

Vous pouvez fournir la définition des entités personnalisées à rechercher dans un fichier CSV (Comma-Separated Value) en indiquant le chemin d’accès à celui-ci et en le définissant dans le paramètre de compétence *entitiesDefinitionUri*. Le chemin d'accès doit correspondre à un emplacement https. La taille du fichier de définition ne doit pas dépasser 10 Mo.

Le format CSV est simple. Chaque ligne représente une entité unique, comme illustré ci-dessous :

```
Bill Gates, BillG, William H. Gates
Microsoft, MSFT
Satya Nadella 
```

Trois entités peuvent ici être renvoyées en tant qu'entités trouvées (Bill Gates, Satya Nadella, Microsoft), mais elles seront identifiées si l'un des termes de la ligne (alias) correspond dans le texte. Par exemple, si la chaîne « William H. Gates » est trouvée dans un document, une correspondance de l'entité « Bill Gates » sera renvoyée.

### <a name="json-format"></a>Format JSON

Vous pouvez également fournir la définition des entités personnalisées à rechercher dans un fichier JSON. Le format JSON vous offre un peu plus de flexibilité, car il vous permet de définir des règles de correspondance « par terme ». Par exemple, vous pouvez spécifier la distance de correspondance approximative (distance de Damerau-Levenshtein) pour chaque terme, ou indiquer si la correspondance doit respecter la casse ou non. 

 Comme avec les fichiers CSV, vous devez fournir le chemin d’accès au fichier JSON et le définir dans le paramètre de compétence *entitiesDefinitionUri*. Le chemin d'accès doit correspondre à un emplacement https. La taille du fichier de définition ne doit pas dépasser 10 Mo.

La définition de liste d'entités personnalisées JSON la plus élémentaire peut être une liste d'entités à comparer :

```json
[ 
    { 
        "name" : "Bill Gates"
    }, 
    { 
        "name" : "Microsoft"
    }, 
    { 
        "name" : "Satya Nadella"
    }
]
```

Un exemple plus complexe de définition JSON peut éventuellement fournir l'ID, la description, le type et le sous-type de chaque entité - ainsi que d'autres *alias*. Si une correspondance est trouvée avec un terme d'alias, l'entité est également renvoyée :

```json
[ 
    { 
        "name" : "Bill Gates",
        "description" : "Microsoft founder." ,
        "aliases" : [ 
            { "text" : "William H. Gates", "caseSensitive" : false },
            { "text" : "BillG", "caseSensitive" : true }
        ]
    }, 
    { 
        "name" : "Xbox One", 
        "type": "Harware",
        "subtype" : "Gaming Device",
        "id" : "4e36bf9d-5550-4396-8647-8e43d7564a76",
        "description" : "The Xbox One product"
    }, 
    { 
        "name" : "LinkedIn" , 
        "description" : "The LinkedIn company", 
        "id" : "differentIdentifyingScheme123", 
        "fuzzyEditDistance" : 0 
    }, 
    { 
        "name" : "Microsoft" , 
        "description" : "Microsoft Corporation", 
        "id" : "differentIdentifyingScheme987", 
        "defaultCaseSensitive" : false, 
        "defaultFuzzyEditDistance" : 1, 
        "aliases" : [ 
            { "text" : "MSFT", "caseSensitive" : true }
        ]
    } 
] 
```

Les tableaux ci-dessous décrivent plus en détail les différents paramètres de configuration que vous pouvez définir lors de la définition des entités à comparer :

|  Nom du champ  |        Description  |
|--------------|----------------------|
| `name` | descripteur d'entité de niveau supérieur. Dans la sortie de la compétence, les correspondances seront regroupées en fonction de ce nom. Il s'agit de la forme « normalisée » du texte trouvé.  |
| `description`  | (Facultatif) Ce champ peut être utilisé pour transmettre des métadonnées personnalisées au sujet du(des) texte(s) comparés. La valeur de ce champ apparaîtra avec chaque correspondance de son entité dans la sortie de la compétence. |
| `type` | (Facultatif) Ce champ peut être utilisé pour transmettre des métadonnées personnalisées au sujet du(des) texte(s) comparés. La valeur de ce champ apparaîtra avec chaque correspondance de son entité dans la sortie de la compétence. |
| `subtype` | (Facultatif) Ce champ peut être utilisé pour transmettre des métadonnées personnalisées au sujet du(des) texte(s) comparés. La valeur de ce champ apparaîtra avec chaque correspondance de son entité dans la sortie de la compétence. |
| `id` | (Facultatif) Ce champ peut être utilisé pour transmettre des métadonnées personnalisées au sujet du(des) texte(s) comparés. La valeur de ce champ apparaîtra avec chaque correspondance de son entité dans la sortie de la compétence. |
| `caseSensitive` | (Facultatif) La valeur par défaut est « False ». Valeur booléenne indiquant si les comparaisons avec le nom de l'entité doivent respecter la casse des caractères. Exemples de correspondances qui ne respectent pas la casse de « Microsoft » : microsoft, microSoft, MICROSOFT |
| `fuzzyEditDistance` | (Facultatif) La valeur par défaut est 0. La valeur maximale est 5. Indique le nombre acceptable de caractères divergents qui constitueraient encore une correspondance avec le nom de l'entité. La plus petite approximation possible d'une correspondance donnée est renvoyée.  Par exemple, si la distance de modification est définie sur 3, « Windows 10 » correspondra encore à « Windows », « Windows10 » et « windows 7 ». <br/> Lorsque le respect de la casse est défini sur « False », les différences de casse ne comptent PAS dans le cadre de la tolérance aux approximations, mais sinon elles comptent. |
| `defaultCaseSensitive` | (Facultatif) Modifie la valeur de respect de la casse par défaut pour cette entité. Elle peut être utilisée pour modifier les valeurs caseSensitive par défaut de tous les alias. |
| `defaultFuzzyEditDistance` | (Facultatif) Modifie la valeur par défaut de la distance de modification approximative pour cette entité. Elle peut être utilisée pour modifier les valeurs fuzzyEditDistance par défaut de tous les alias. |
| `aliases` | (Facultatif) Groupe d'objets complexes qui peut être utilisé pour spécifier d'autres orthographes ou des synonymes au nom de l'entité racine. |

| Propriétés des alias | Description |
|------------------|-------------|
| `text`  | Autre orthographe ou représentation d'un nom d'entité cible.  |
| `caseSensitive` | (Facultatif) Agit de la même manière que le paramètre « caseSensitive » de l'entité racine ci-dessus, mais ne s'applique qu'à cet alias. |
| `fuzzyEditDistance` | (Facultatif) Agit de la même manière que le paramètre « fuzzyEditDistance » de l'entité racine ci-dessus, mais ne s'applique qu'à cet alias. |


### <a name="inline-format"></a>Format inline

Dans certains cas, il est plus facile de fournir directement dans la définition de la compétence la liste des entités personnalisées à comparer au format inline. Vous pouvez alors utiliser un format JSON semblable à celui décrit ci-dessus, mais il est « inlined » (inclus) dans la définition de la compétence.
Seules les configurations dont la taille est inférieure à 10 Ko (taille sérialisée) peuvent être définies au format inline. 

##    <a name="sample-definition"></a>Exemple de définition

Voici un exemple de définition de compétence utilisant un format inline :

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.CustomEntityLookupSkill",
    "context": "/document",
    "inlineEntitiesDefinition": 
    [
      { 
        "name" : "Bill Gates",
        "description" : "Microsoft founder." ,
        "aliases" : [ 
            { "text" : "William H. Gates", "caseSensitive" : false },
            { "text" : "BillG", "caseSensitive" : true }
        ]
      }, 
      { 
        "name" : "Xbox One", 
        "type": "Hardware",
        "subtype" : "Gaming Device",
        "id" : "4e36bf9d-5550-4396-8647-8e43d7564a76",
        "description" : "The Xbox One product"
      }
    ],    
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "entities",
        "targetName": "matchedEntities"
      }
    ]
  }
```
Par ailleurs, si vous décidez de fournir un pointeur vers le fichier de définition des entités, un exemple de définition de compétence utilisant le format `entitiesDefinitionUri` est présenté ci-dessous :

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.CustomEntityLookupSkill",
    "context": "/document",
    "entitiesDefinitionUri": "https://myblobhost.net/keyWordsConfig.csv",    
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "entities",
        "targetName": "matchedEntities"
      }
    ]
  }

```

##    <a name="sample-input"></a>Exemple d’entrée

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "text": "The company, Microsoft, was founded by Bill Gates. Microsoft's gaming console is called Xbox",
             "languageCode": "en"
           }
      }
    ]
}
```

##    <a name="sample-output"></a>Exemple de sortie

```json
  { 
    "values" : 
    [ 
      { 
        "recordId": "1", 
        "data" : { 
          "entities": [
            { 
              "name" : "Microsoft", 
              "description" : "This document refers to Microsoft the company", 
              "id" : "differentIdentifyingScheme987", 
              "matches" : [ 
                { 
                  "text" : "microsoft", 
                  "offset" : 13, 
                  "length" : 9, 
                  "matchDistance" : 0 
                }, 
                { 
                  "text" : "Microsoft",
                  "offset" : 49, 
                  "length" : 9, 
                  "matchDistance" : 0
                }
              ] 
            },
            { 
              "name" : "Bill Gates",
              "description" : "William Henry Gates III, founder of Microsoft.", 
              "matches" : [
                { 
                  "text" : "Bill Gates",
                  "offset" : 37, 
                  "length" : 10,
                  "matchDistance" : 0 
                }
              ]
            }
          ] 
        } 
      } 
    ] 
  } 
```

## <a name="errors-and-warnings"></a>Erreurs et avertissements

### <a name="warning-reached-maximum-capacity-for-matches-skipping-all-further-duplicate-matches"></a>Avertissement : la capacité maximale a été atteinte pour les correspondances, et toutes les correspondances en double ont été ignorées.

Cet avertissement s’affiche si le nombre de correspondances détectées est supérieur au nombre maximal autorisé. Dans ce cas, les correspondances en double ne seront pas incluses. Si cela ne vous contient pas, veuillez soumettre un [ticket de support](https://ms.portal.azure.com/#create/Microsoft.Support) afin que nous puissions vous aider dans votre cas d’utilisation spécifique.

## <a name="see-also"></a>Voir aussi

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Compétence de reconnaissance d'entités (pour rechercher des entités bien connues)](cognitive-search-skill-entity-recognition.md)