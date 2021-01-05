---
title: Meilleures pratiques d’utilisation du service Recherche Azure Maps | Microsoft Azure Maps
description: Découvrez comment appliquer les meilleures pratiques du service de recherche de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/02/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 11c1938c3c1ccba533f52336fad81ebeaae53b24
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895475"
---
# <a name="best-practices-for-azure-maps-search-service"></a>Meilleures pratiques d’utilisation du service Recherche Azure Maps

Le [service Search](/rest/api/maps/search) d’Azure Maps inclut des API offrant plusieurs fonctionnalités destinées aux développeurs souhaitant rechercher des adresses, des lieux, des listes d’entreprises par nom ou catégorie et d’autres informations d’ordre géographique. Par exemple, [l’API Fuzzy Search](/rest/api/maps/search/getsearchfuzzy) permet aux utilisateurs de rechercher une adresse ou un point d’intérêt (POI).

Cet article explique comment appliquer des pratiques saines lorsque vous appelez des données à partir du service Recherche Azure Maps. Vous découvrirez comment effectuer les actions suivantes :
> [!div class="checklist"]
> * Générer des requêtes destinées à renvoyer les correspondances pertinentes
> * Limiter les résultats de la recherche
> * Découvrir les différences entre les types de résultats
> * Lire la structure recherche-réponse de l’adresse

## <a name="prerequisites"></a>Prérequis

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.

Cet article utilise l’[application Postman](https://www.postman.com/downloads/) pour générer des appels REST, mais vous pouvez choisir l’environnement de développement d’API de votre choix.

## <a name="best-practices-to-geocode-addresses"></a>Meilleures pratiques pour les adresses géocode

Lorsque vous recherchez une adresse complète ou partielle à l’aide du service Recherche Azure Maps, l’API lit les mots clés de votre requête de recherche. Elle retourne ensuite les coordonnées de longitude et de latitude de l’adresse. Ce processus est appelé *géocodage* .

La possibilité de géocoder dans un pays/une région dépend de la disponibilité des données de route et de la précision du service de géocodage. Pour plus d’informations sur les fonctionnalités de géocodage Azure Maps par pays ou région, voir [Couverture du géocodage](./geocoding-coverage.md).

### <a name="limit-search-results"></a>Limiter les résultats de la recherche

 L’API Recherche Azure Maps peut vous aider à limiter les résultats de recherche de manière appropriée. Vous limitez les résultats afin de pouvoir afficher des données pertinentes à vos utilisateurs.

> [!NOTE]
> Les API de recherche prennent en charge davantage de paramètres que ceux abordés dans cet article.

#### <a name="geobiased-search-results"></a>Résultats de recherche adaptés à la zone géographique

Pour adapter des résultats en fonction de la zone pertinente pour votre utilisateur, ajoutez toujours un maximum de détails sur l’emplacement. Vous pouvez restreindre les résultats de la recherche en spécifiant des types d’entrée :

* Définissez le paramètre `countrySet`. Vous pouvez le définir sur `US,FR`, par exemple. Par défaut, l’API recherche dans le monde entier, de sorte qu’elle peut retourner des résultats inutiles. Si votre requête n’a pas de paramètre `countrySet`, la recherche peut retourner des résultats inexacts. Par exemple, la recherche d’une ville nommée *Bellevue* retourne des résultats pour les États-Unis et la France, car les deux pays/régions contiennent une ville nommée *Bellevue* .

* Vous pouvez utiliser les paramètres `btmRight` et `topleft` pour définir le cadre englobant. Ces paramètres limitent la recherche à une zone spécifique sur la carte.

* Pour influencer la zone de pertinence des résultats, définissez les paramètres de coordonnées `lat` et `lon`. Utilisez le paramètre `radius` pour définir le rayon de la zone de recherche.


#### <a name="fuzzy-search-parameters"></a>Paramètres de recherche approximative

Nous vous suggérons d’utiliser l’[API de recherche approximative](/rest/api/maps/search/getsearchfuzzy) Azure Maps lorsque vous ne connaissez pas vos entrées utilisateur pour une requête de recherche. Par exemple, l’entrée de l’utilisateur peut être une adresse ou le type de point d’intérêt (POI), par exemple *centre commercial* . L’API combine la recherche de points d’intérêt et le géocodage dans une *recherche de ligne unique* canonique : 

* Les paramètres `minFuzzyLevel` et `maxFuzzyLevel` permettent de renvoyer des correspondances pertinentes même lorsque les paramètres de requête ne correspondent pas exactement aux informations que l’utilisateur souhaite. Pour optimiser les performances et réduire les résultats inhabituels, définissez les requêtes de recherche sur les valeurs par défaut `minFuzzyLevel=1` et `maxFuzzyLevel=2`. 

    Par exemple, lorsque le paramètre `maxFuzzyLevel` a la valeur 2, le terme de recherche *restrant* est mis en correspondance avec *restaurant* . Vous pouvez remplacer les niveaux approximatifs par défaut quand vous en avez besoin. 

* Utilisez le paramètre `idxSet` pour hiérarchiser le jeu exact de types de résultats. Pour hiérarchiser un jeu exact de résultats, vous pouvez envoyer une liste d’index séparés par des virgules. L’ordre des éléments dans votre liste n’a pas d’importance. Azure Maps prend en charge les index suivants :

* `Addr` - **Plages d’adresses** : Points d’adresse interpolés à partir du début et de la fin de la rue. Ces points sont représentés en tant que plages d’adresses.
* `Geo` - **Zones géographiques** : Divisions administratives de territoire. Une zone géographique peut être, par exemple, un pays/une région, un État ou une ville.
* `PAD` - **Adresses exactes**  : Adresses incluant un nom et un numéro de rue. Des adresses exactes peuvent figurer dans un index. Exemple : *Soquel Dr 2501* . Une adresse exacte fournit le niveau de précision le plus élevé disponible pour des adresses.  
* `POI` - **Points d’intérêt** : Points sur une carte qui sont considérés comme dignes d’attention ou susceptibles d’être intéressants. L’[API de recherche d’adresse](/rest/api/maps/search/getsearchaddress) ne retourne pas POI.  
* `Str` - **Rues** : Rues sur la carte.
* `XStr` - **Croisements ou intersections** : Jonctions ou emplacements où deux rues se croisent.


#### <a name="usage-examples"></a>Exemples d'utilisation

* `idxSet=POI` - Rechercher uniquement des POI. 

* `idxSet=PAD,Addr` - Rechercher uniquement des adresses. `PAD` indique l’adresse exacte et `Addr` indique la plage d’adresses.

### <a name="reverse-geocode-and-filter-for-a-geography-entity-type"></a>Géocode inversé et filtre pour un type d’entité géographique

Lorsque vous effectuez une recherche de géocode inversé dans l’[API de recherche d’adresse inverse](/rest/api/maps/search/getsearchaddressreverse), le service peut retourner des polygones pour des zones administratives. Par exemple, vous souhaitez peut-être extraire le polygone de zone pour une ville.  Pour limiter la recherche à des types d’entités géographiques spécifiques, incluez le paramètre `entityType` dans vos demandes. 

La réponse obtenue contient l’ID de zone géographique et le type d’entité mis en correspondance. Si vous fournissez plusieurs entités, le point de terminaison renvoie la *plus petite entité disponible* . Vous pouvez utiliser l’ID de géométrie renvoyé pour récupérer la géométrie de la zone géographique par le biais du [service de recherche de polygone](/rest/api/maps/search/getsearchpolygon).

#### <a name="sample-request"></a>Exemple de requête

```HTTP
https://atlas.microsoft.com/search/address/reverse/json?api-version=1.0&subscription-key={subscription-key}&query=47.6394532,-122.1304551&language=en-US&entityType=Municipality
```

#### <a name="response"></a>response

```JSON
{
    "summary": {
        "queryTime": 14,
        "numResults": 1
    },
    "addresses": [
        {
            "address": {
                "routeNumbers": [],
                "countryCode": "US",
                "countrySubdivision": "WA",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "municipality": "Redmond",
                "country": "United States",
                "countryCodeISO3": "USA",
                "countrySubdivisionName": "Washington"
            },
            "position": "47.639454,-122.130455",
            "dataSources": {
                "geometry": {
                    "id": "00005557-4100-3c00-0000-0000596ae571"
                }
            },
            "entityType": "Municipality"
        }
    ]
}
```

### <a name="set-the-results-language"></a>Définir la langue des résultats

Utilisez le paramètre `language` pour définir la langue des résultats de recherche retournés. Si la demande ne définit pas la langue, par défaut, le service Recherche utilise la langue la plus courante dans le pays ou la région. Si aucune donnée n’est disponible dans la langue spécifiée, la langue par défaut est utilisée. 

Pour plus d’informations, voir [Langues prises en charge dans Azure Maps](./supported-languages.md).


### <a name="use-predictive-mode-automatic-suggestions"></a>Utiliser le mode prédictif (suggestions automatiques)

Pour trouver d’autres correspondances pour des requêtes partielles, définissez le paramètre `typeahead` sur `true`. Cette requête est interprétée comme une entrée partielle et la recherche passe en mode prédictif. Si vous ne définissez pas le paramètre `typeahead` sur `true`, le service suppose que toutes les informations pertinentes ont été transmises.

Dans l’exemple de requête suivant, le service de recherche d’adresse est interrogé pour *Microsoft* . Ici, le paramètre `typeahead` est défini sur `true`. La réponse indique que le service de recherche a interprété la requête comme une requête partielle. La réponse contient des résultats pour une requête suggérée automatiquement.

#### <a name="sample-query"></a>Exemple de requête

```HTTP
https://atlas.microsoft.com/search/address/json?subscription-key={subscription-key}&api-version=1.0&typeahead=true&countrySet=US&lat=47.6370891183&lon=-122.123736172&query=Microsoft
```

#### <a name="response"></a>response

```JSON
{
    "summary": {
        "query": "microsoft",
        "queryType": "NON_NEAR",
        "queryTime": 18,
        "numResults": 7,
        "offset": 0,
        "totalResults": 7,
        "fuzzyLevel": 1,
        "geoBias": {
            "lat": 47.6370891183,
            "lon": -122.123736172
        }
    },
    "results": [
        {
            "type": "Street",
            "id": "US/STR/p0/9438784",
            "score": 2.594099998474121,
            "dist": 314.0590106663596,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
            },
            "position": {
                "lat": 47.63988,
                "lon": -122.12438
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.64223,
                    "lon": -122.1256,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63748,
                    "lon": -122.12309,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/1756074",
            "score": 2.592679977416992,
            "dist": 876.0272035824189,
            "address": {
                "streetName": "Microsoft Road",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98052",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft Road, Redmond, WA 98052"
            },
            "position": {
                "lat": 47.64032,
                "lon": -122.1344
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.64253,
                    "lon": -122.13535,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63816,
                    "lon": -122.13305,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/1470668",
            "score": 2.5290400981903076,
            "dist": 2735.4883918101486,
            "address": {
                "streetName": "Microsoft West Campus Road",
                "municipalitySubdivision": "Redmond",
                "municipality": "Bellevue",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98007",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft West Campus Road, Bellevue, WA 98007"
            },
            "position": {
                "lat": 47.65784,
                "lon": -122.14335
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.65785,
                    "lon": -122.14335,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.65784,
                    "lon": -122.14325,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/12812615",
            "score": 2.527509927749634,
            "dist": 2870.9579016916873,
            "address": {
                "streetName": "Microsoft West Campus Road",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98052",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft West Campus Road, Redmond, WA 98052"
            },
            "position": {
                "lat": 47.66034,
                "lon": -122.1404
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.66039,
                    "lon": -122.14325,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.65778,
                    "lon": -122.13749,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/197588",
            "score": 2.4630401134490967,
            "dist": 878.1404663812472,
            "address": {
                "streetName": "157th Avenue Northeast",
                "municipalitySubdivision": "Redmond",
                "municipality": "Redmond",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98052",
                "extendedPostalCode": "980525344, 980525398, 980525399",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "157th Avenue Northeast, Redmond, WA 98052"
            },
            "position": {
                "lat": 47.64351,
                "lon": -122.13056
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.64473,
                    "lon": -122.13058,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.6425,
                    "lon": -122.13016,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/3033991",
            "score": 2.0754499435424805,
            "dist": 3655467.8844475765,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Yorkmount, Charlotte",
            },
            "position": {
                "lat": 35.14267,
                "lon": -80.91824
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 35.14287,
                    "lon": -80.91839,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 35.14267,
                    "lon": -80.91814,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/8395877",
            "score": 2.0754499435424805,
            "dist": 3655437.0037482483,
            "address": {
                "streetName": "Microsoft Way",
                "municipalitySubdivision": "Charlotte",
                "municipality": "Charlotte",
                "countrySecondarySubdivision": "Mecklenburg",
                "countryTertiarySubdivision": "Township 1 Charlotte",
                "countrySubdivision": "NC",
                "countrySubdivisionName": "North Carolina",
                "postalCode": "28273",
                "extendedPostalCode": "282738105, 282738106, 282738108, 2827382, 282738200",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Microsoft Way, Charlotte, NC 28273"
            },
            "position": {
                "lat": 35.14134,
                "lon": -80.9198
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 35.14274,
                    "lon": -80.92159,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 35.14002,
                    "lon": -80.91824,
                    "valid": true
                }
            }
        }
    ]
}
```


### <a name="encode-a-uri-to-handle-special-characters"></a>Encoder un URI pour gérer les caractères spéciaux 

Pour rechercher des adresses de croisement, vous devez encoder l’URI pour gérer des caractères spéciaux dans l’adresse. Prenons cet exemple d’adresse : *1st Avenue & Union Street, Seattle* . Ici, encodez le caractère esperluette (`&`) avant d’envoyer la demande. 

Nous vous recommandons d’encoder les données de caractères dans un URI. Dans un URI, vous encodez tous les caractères à l’aide d’un signe de pourcentage (`%`) et d’une valeur hexadécimale de deux caractères correspondant au code UTF-8 des caractères.

#### <a name="usage-examples"></a>Exemples d'utilisation

Commencer par cette adresse :

```
query=1st Avenue & E 111th St, New York
```

Encoder l’adresse :

```
query=1st%20Avenue%20%26%20E%20111th%20St%2C%20New%20York
```

Vous pouvez utiliser les méthodes suivantes.

JavaScript ou TypeScript :
```javascript
encodeURIComponent(query)
```

C# ou Visual Basic :
```csharp
Uri.EscapeDataString(query)
```

Java :
```Java
URLEncoder.encode(query, "UTF-8") 
```

Python :
```Python
import urllib.parse 
urllib.parse.quote(query)
```

C++ :
```C++
#include <curl/curl.h>
curl_easy_escape(query)
```

PHP :
```PHP
urlencode(query)
```

Ruby :
```Ruby
CGI::escape(query) 
```

Swift :
```Swift
query.stringByAddingPercentEncodingWithAllowedCharacters(.URLHostAllowedCharacterSet()) 
```

Go :
```Go
import ("net/url") 
url.QueryEscape(query)
```


## <a name="best-practices-for-poi-searching"></a>Meilleures pratiques pour la recherche de points d’intérêt

Dans une recherche de POI, vous pouvez demander des résultats de recherche par nom. Par exemple, vous pouvez rechercher une entreprise par son nom. 

Nous vous recommandons vivement d’utiliser le paramètre `countrySet` pour spécifier les pays/régions où votre application a besoin d’une couverture. Le comportement par défaut consiste à effectuer des recherches dans le monde entier. Une recherche aussi large peut retourner des résultats superflus et prendre beaucoup de temps.

### <a name="brand-search"></a>Recherche de marque

Pour améliorer la pertinence des résultats et des informations contenues dans la réponse, une réponse à une recherche de POI inclut des informations de marque. Vous pouvez utiliser celles-ci pour poursuivre l’analyse de la réponse.

Dans une demande, vous pouvez envoyer une liste de noms de marques séparés par des virgules. Vous pouvez utiliser cette liste pour limiter les résultats à des marques spécifiques à l’aide du paramètre `brandSet`. L’ordre des éléments dans votre liste est sans importance. Lorsque vous fournissez plusieurs listes de marques, les résultats retournés doivent appartenir à au moins une de vos listes.

Pour explorer la recherche de marque, nous allons effectuer une demande de [recherche de catégorie de POI](/rest/api/maps/search/getsearchpoicategory). Dans l’exemple suivant, nous recherchons des stations-service à proximité du campus Microsoft à Redmond, Washington. La réponse affiche des informations de marque pour chaque POI retourné.

#### <a name="sample-query"></a>Exemple de requête

```HTTP
https://atlas.microsoft.com/search/poi/json?subscription-key={subscription-key}&api-version=1.0&query=gas%20station&limit=3&lat=47.6413362&lon=-122.1327968
```

#### <a name="response"></a>response

```JSON
{
    "summary": {
        "query": "gas station",
        "queryType": "NON_NEAR",
        "queryTime": 276,
        "numResults": 3,
        "offset": 0,
        "totalResults": 762680,
        "fuzzyLevel": 1,
        "geoBias": {
            "lat": 47.6413362,
            "lon": -122.1327968
        }
    },
    "results": [
        {
            "type": "POI",
            "id": "US/POI/p0/8831765",
            "score": 5.6631999015808105,
            "dist": 1037.0280221303253,
            "info": "search:ta:840531000004190-US",
            "poi": {
                "name": "Chevron",
                "phone": "+(1)-(425)-6532200",
                "brands": [
                    {
                        "name": "Chevron"
                    }
                ],
                "categorySet": [
                    {
                        "id": 7311
                    }
                ],
                "url": "www.chevron.com",
                "categories": [
                    "petrol station"
                ],
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "2444",
                "streetName": "Bel Red Rd",
                "municipalitySubdivision": "Northeast Bellevue, Bellevue",
            },
            "position": {
                "lat": 47.63201,
                "lon": -122.13281
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.63291,
                    "lon": -122.13414,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63111,
                    "lon": -122.13148,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.63222,
                        "lon": -122.13312,
                        "valid": true
                    }
                }
            ]
        },
        {
            "type": "POI",
            "id": "US/POI/p0/8831752",
            "score": 5.662710189819336,
            "dist": 1330.1278248163273,
            "info": "search:ta:840539001100326-US",
            "poi": {
                "name": "76",
                "phone": "+(1)-(425)-7472126",
                "brands": [
                    {
                        "name": "76"
                    }
                ],
                "categorySet": [
                    {
                        "id": 7311
                    }
                ],
                "url": "www.76.com",
                "categories": [
                    "petrol station"
                ],
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "2421",
                "streetName": "148Th Ave Ne",
                "municipalitySubdivision": "Redmond, Bridle Trails, Bellevue",
                "municipality": "Redmond, Bellevue",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle East",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98007",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "2421 148Th Ave Ne, Bellevue, WA 98007",
                "localName": "Bellevue"
            },
            "position": {
                "lat": 47.63187,
                "lon": -122.14365
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.63277,
                    "lon": -122.14498,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.63097,
                    "lon": -122.14232,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "minor",
                    "position": {
                        "lat": 47.63187,
                        "lon": -122.14374,
                        "valid": true
                    }
                },
                {
                    "type": "main",
                    "position": {
                        "lat": 47.63186,
                        "lon": -122.14313,
                        "valid": true
                    }
                }
            ]
        },
        {
            "type": "POI",
            "id": "US/POI/p0/8831764",
            "score": 5.662449836730957,
            "dist": 1458.645407416307,
            "info": "search:ta:840539000488527-US",
            "poi": {
                "name": "BROWN BEAR CAR WASH",
                "phone": "+(1)-(425)-6442868",
                "brands": [
                    {
                        "name": "Texaco"
                    }
                ],
                "categorySet": [
                    {
                        "id": 7311
                    }
                ],
                "url": "www.texaco.com/",
                "categories": [
                    "petrol station"
                ],
                "classifications": [
                    {
                        "code": "PETROL_STATION",
                        "names": [
                            {
                                "nameLocale": "en-US",
                                "name": "petrol station"
                            }
                        ]
                    }
                ]
            },
            "address": {
                "streetNumber": "15248",
                "streetName": "Bel Red Rd",
                "municipalitySubdivision": "Redmond",
            },
            "position": {
                "lat": 47.62843,
                "lon": -122.13628
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62933,
                    "lon": -122.13761,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.62753,
                    "lon": -122.13495,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.62827,
                        "lon": -122.13628,
                        "valid": true
                    }
                }
            ]
        }
    ]
}
```


### <a name="airport-search"></a>Recherche d’aéroports

L’API de recherche de POI vous permet de rechercher des aéroports en utilisant leur code officiel. Par exemple, vous pouvez utiliser *SEA* pour trouver l’aéroport international de Seattle-Tacoma : 

```HTTP
https://atlas.microsoft.com/search/poi/json?subscription-key={subscription-key}&api-version=1.0&query=SEA 
```

### <a name="nearby-search"></a>Recherche à proximité

Pour récupérer des résultats de POI autour d’un emplacement spécifique, vous pouvez essayer d’utiliser l’[API de recherche à proximité](/rest/api/maps/search/getsearchnearby). Le point de terminaison retourne uniquement des résultats de POI. Il ne prend pas de paramètre de requête de recherche. 

Pour limiter les résultats, nous vous suggérons de définir le rayon.

## <a name="understanding-the-responses"></a>Compréhension des réponses

Cherchons une adresse à Seattle en adressant une demande de recherche d’adresse au service Recherche Azure Maps. Dans l’URL de demande suivante, nous définissons le paramètre `countrySet` sur `US` pour rechercher l’adresse aux États-Unis.

### <a name="sample-query"></a>Exemple de requête

```HTTP
https://atlas.microsoft.com/search/address/json?subscription-key={subscription-key}&api-version=1&query=400%20Broad%20Street%2C%20Seattle%2C%20WA&countrySet=US
```

### <a name="supported-types-of-results"></a>Types de résultats pris en charge

* **Adresse exacte**  : Points sur une carte correspondant à une adresse spécifique comportant un nom et un numéro de rue. L’adresse du point fournit le niveau de précision le plus élevé pour des adresses. 

* **Plage d’adresses**  : Plage de points d’adresse interpolés à partir du début et de la fin de la rue.  

* **Geography**  : zones sur une carte représentant des divisions administratives d’un territoire, telles qu’un pays/une région, un État ou une ville. 

* **POI**  : points sur une carte qui méritent une attention particulière et pourraient être intéressants.

* **Rue**  : Rues sur la carte. Les adresses sont converties en coordonnées de latitude et longitude dans la rue correspondante. Il se peut que le numéro de l’immeuble ne soit pas être traité. 

* **Croisement**  : intersections. Les croisements représentent des jonctions où deux rues se croisent.

### <a name="response"></a>response

Examinons la structure de la réponse. Dans la réponse qui suit, les types des objets obtenus sont différents. En regardant attentivement, vous voyez trois types d’objets obtenus :

* Adresse exacte
* Rue
* Croisement

Notez que la recherche d’adresse ne renvoie pas de points d’intérêt.  

Le paramètre `Score` de chaque objet de réponse indique le score de correspondance de cet objet par rapport aux autres objets qui figurent dans la même réponse. Pour plus d’informations sur les paramètres d’objet de réponse, voir [Get Search Address](/rest/api/maps/search/getsearchaddress).

```JSON
{
    "summary": {
        "query": "400 broad street seattle wa",
        "queryType": "NON_NEAR",
        "queryTime": 146,
        "numResults": 6,
        "offset": 0,
        "totalResults": 7,
        "fuzzyLevel": 1
    },
    "results": [
        {
            "type": "Point Address",
            "id": "US/PAD/p0/28725082",
            "score": 9.893799781799316,
            "address": {
                "streetNumber": "400",
                "streetName": "Broad Street",
            },
            "position": {
                "lat": 47.62039,
                "lon": -122.34928
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62129,
                    "lon": -122.35061,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61949,
                    "lon": -122.34795,
                    "valid": true
                }
            },
            "entryPoints": [
                {
                    "type": "main",
                    "position": {
                        "lat": 47.61982,
                        "lon": -122.34886,
                        "valid": true
                    }
                }
            ]
        },
        {
            "type": "Street",
            "id": "US/STR/p0/6700384",
            "score": 8.129190444946289,
            "address": {
                "streetName": "Broad Street",
            },
            "position": {
                "lat": 47.61724,
                "lon": -122.35207
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.61825,
                    "lon": -122.35336,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61626,
                    "lon": -122.35078,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/9701953",
            "score": 8.129190444946289,
            "address": {
                "streetName": "Broad Street",
            },
            "position": {
                "lat": 47.61965,
                "lon": -122.349
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62066,
                    "lon": -122.35041,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61857,
                    "lon": -122.34761,
                    "valid": true
                }
            }
        },
        {
            "type": "Street",
            "id": "US/STR/p0/11721297",
            "score": 8.129190444946289,
            "address": {
                "streetName": "Broad Street",
                "municipalitySubdivision": "Seattle, Downtown Seattle, Denny Regrade, Belltown",
                "municipality": "Seattle",
                "countrySecondarySubdivision": "King",
                "countryTertiarySubdivision": "Seattle",
                "countrySubdivision": "WA",
                "countrySubdivisionName": "Washington",
                "postalCode": "98121",
                "extendedPostalCode": "981211237",
                "countryCode": "US",
                "country": "United States",
                "countryCodeISO3": "USA",
                "freeformAddress": "Broad Street, Seattle, WA 98121"
            },
            "position": {
                "lat": 47.61825,
                "lon": -122.35078
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.61857,
                    "lon": -122.35078,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.61825,
                    "lon": -122.35041,
                    "valid": true
                }
            }
        },
        {
            "type": "Cross Street",
            "id": "US/XSTR/p1/232144",
            "score": 6.754479885101318,
            "address": {
                "streetName": "Broad Street & Valley Street",
                "municipalitySubdivision": "South Lake Union, Seattle",
            },
            "position": {
                "lat": 47.62545,
                "lon": -122.33974
            },
            "viewport": {
                "topLeftPoint": {
                    "lat": 47.62635,
                    "lon": -122.34107,
                    "valid": true
                },
                "btmRightPoint": {
                    "lat": 47.62455,
                    "lon": -122.33841,
                    "valid": true
                }
            }
        }
    ]
}
```

### <a name="geometry"></a>Géométrie

Un type de réponse de *Géométrie* peut inclure l’ID de géométrie retourné dans l’objet `dataSources` sous `geometry` et `id`. Par exemple, vous pouvez utiliser le [service de recherche de polygone](/rest/api/maps/search/getsearchpolygon) pour demander les données de géométrie dans un format GeoJSON. Ce format vous permet d’obtenir un plan de ville ou d’aéroport pour un ensemble d’entités. Vous pouvez ensuite utiliser ces données de limite pour [Définir une limite géographique](./tutorial-geofence.md) ou [Rechercher des points d’intérêt à l’intérieur de la géométrie](/rest/api/maps/search/postsearchinsidegeometry).


Les réponses pour l’API de [recherche d’adresse](/rest/api/maps/search/getsearchaddress) ou l’API de [recherche approximative](/rest/api/maps/search/getsearchfuzzy) peuvent inclure l’ID de géométrie retourné dans l’objet `dataSources` sous `geometry` et `id` :


```JSON 
"dataSources": { 
        "geometry": { 
            "id": "00005557-4100-3c00-0000-000059690938" // The geometry ID is returned in the dataSources object under "geometry" and "id".
        }
} 
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez :

> [!div class="nextstepaction"]
> [Comment générer des demandes requêtes destinées au service Recherche Azure Maps](./how-to-search-for-address.md)

> [!div class="nextstepaction"]
> [Documentation de l’API du service de recherche](/rest/api/maps/search)