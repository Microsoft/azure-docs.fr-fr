---
title: Processeurs de télémétrie (préversion) – Azure Monitor Application Insights pour Java
description: Comment configurer des processeurs de télémétrie dans Azure Monitor Application Insights pour Java
ms.topic: conceptual
ms.date: 10/29/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 7fd53c77b64e028ffad25c8fa7a9eefd95439513
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387154"
---
# <a name="telemetry-processors-preview---azure-monitor-application-insights-for-java"></a>Processeurs de télémétrie (préversion) – Azure Monitor Application Insights pour Java

> [!NOTE]
> Cette fonctionnalité est encore en préversion.

L’agent Java 3.0 pour Application Insights offre désormais les fonctionnalités nécessaires pour le traitement des données de télémétrie avant leur exportation.

### <a name="some-use-cases"></a>Voici quelques cas d’utilisation :
 * Masquer des données sensibles
 * Ajouter de manière conditionnelle des dimensions personnalisées
 * Mettre à jour le nom de télémétrie utilisé pour l’agrégation et l’affichage

### <a name="supported-processors"></a>Processeurs pris en charge :
 * Processeur d’attribut
 * Processeur d’étendue

## <a name="to-get-started"></a>Pour commencer

Créez un fichier de configuration nommé `applicationinsights.json`, puis placez-le dans le même répertoire que `applicationinsights-agent-***.jar`, avec le modèle suivant.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        ...
      },
      {
        "type": "attribute",
        ...
      },
      {
        "type": "span",
        ...
      }
    ]
  }
}
```

## <a name="includeexclude-spans"></a>Inclure/exclure des étendues

Les processeurs d’attribut et d’étendue exposent l’option permettant de fournir un ensemble de propriétés d’une étendue par rapport à laquelle établir une correspondance, afin de déterminer si l’étendue doit être incluse dans le processeur ou exclue de celui-ci. Pour configurer cette option, sous `include` et/ou `exclude`, au moins un `matchType` et l’un des `spanNames` ou `attributes` sont requis. Plusieurs conditions peuvent être spécifiées pour la configuration d’inclusion/exclusion. Pour qu’une correspondance soit établie, toutes les conditions spécifiées doivent prendre la valeur true. 

**Champ obligatoire** : 
* `matchType` contrôle la façon dont les éléments des tableaux `spanNames` et `attributes` sont interprétés. Les valeurs possibles sont `regexp` ou `strict`. 

**Champs facultatifs** : 
* `spanNames` doit correspondre au moins à l’un des éléments. 
* `attributes` spécifie la liste des attributs par rapport auxquels établir une correspondance. Tous ces attributs doivent correspondre exactement pour qu’une correspondance soit établie.

> [!NOTE]
> Si les attributs `include` et `exclude` sont tous deux spécifiés, les propriétés de l’attribut `include` sont vérifiées avant celles ce l’attribut `exclude`.

#### <a name="sample-usage"></a>Exemple d’utilisation

L’exemple suivant montre comment spécifier le jeu de propriétés d’étendue pour indiquer les étendues auxquelles ce processeur doit être appliqué. Les propriétés `include` indiquent les étendues à inclure, et les propriétés `exclude` filtrent les plages qui ne doivent pas être traitées.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "include": {
          "matchType": "strict",
          "spanNames": [
            "svcA",
            "svcB"
          ]
        },
        "exclude": {
          "matchType": "strict",
          "attributes": [
            {
              "key": "redact_trace",
              "value": "false"
            }
          ]
        },
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          },
          {
            "key": "duplicate_key",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

Avec la configuration ci-dessus, les étendues suivantes correspondent aux propriétés et les actions de processeur sont appliquées :

* Span1 Name: 'svcB' Attributes: {env: production, test_request: 123, credit_card: 1234, redact_trace: "false"}

* Span2 Name: 'svcA' Attributes: {env: staging, test_request: false, redact_trace: true}

Les étendues suivantes ne correspondent pas aux propriétés include et les actions de processeur ne sont pas appliquées :

* Span3 Name: 'svcB' Attributes: {env: production, test_request: true, credit_card: 1234, redact_trace: false}

* Span4 Name: 'svcC' Attributes: {env: dev, test_request: false}

## <a name="attribute-processor"></a>Processeur d’attributs 

Le processeur d’attributs modifie les attributs d’une étendue. Il prend éventuellement en charge la possibilité d’inclure ou d’exclure des étendues.
Il prend une liste d’actions qui sont exécutées dans l’ordre spécifié dans le fichier de configuration. Les actions prises en charge sont les suivantes :

* `insert` : insère un nouvel attribut dans les étendues où la clé n’existe pas encore.
* `update` : met à jour un attribut dans les étendues où la clé existe.
* `delete` : supprime un attribut d’une étendue.
* `hash` : hache (SHA1) une valeur d’attribut existante.

Pour les actions `insert` et `update` :
* `key` est obligatoire.
* `value` ou `fromAttribute` est obligatoire.
* `action` est obligatoire.

Pour l’action `delete` :
* `key` est obligatoire.
* `action`: `delete` est obligatoire.

Pour l’action `hash` :
* `key` est obligatoire.
* `action` : `hash` est obligatoire.

Vous pouvez composer la liste d’actions pour créer des scénarios riches, tels que le remplissage d’attribut en retour, la copie de valeurs vers une nouvelle clé et la rédaction d’informations sensibles.

#### <a name="sample-usage"></a>Exemple d’utilisation

L’exemple suivant montre l’insertion de clés/valeurs dans des étendues :

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "attribute1",
            "value": "value1",
            "action": "insert"
          },
          {
            "key": "key1",
            "fromAttribute": "anotherkey",
            "action": "insert"
          }
        ]
      }
    ]
  }
}
```

L’exemple suivant montre la configuration du processeur pour mettre à jour uniquement les clés existantes dans un attribut :

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "piiattribute",
            "value": "redacted",
            "action": "update"
          },
          {
            "key": "credit_card",
            "action": "delete"
          },
          {
            "key": "user.email",
            "action": "hash"
          }
        ]
      }
    ]
  }
}
```

L’exemple suivant montre comment traiter des étendues dont le nom correspond à des modèles regexps.
Ce processeur supprime l’attribut « token » et obfusque l’attribut « password » dans les étendues dont le nom correspond à « auth.\* » et ne correspond pas à « login.\* ».

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "include": {
          "matchType": "regexp",
          "spanNames": [
            "auth.*"
          ]
        },
        "exclude": {
          "matchType": "regexp",
          "spanNames": [
            "login.*"
          ]
        },
        "actions": [
          {
            "key": "password",
            "value": "obfuscated",
            "action": "update"
          },
          {
            "key": "token",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

## <a name="span-processors"></a>Processeurs d’étendue

Le processeur d’étendue modifie le nom ou les attributs d’une étendue en fonction de son nom. Il prend éventuellement en charge la possibilité d’inclure ou d’exclure des étendues.

### <a name="name-a-span"></a>Nommer une étendue

Le paramètre suivant est requis dans la section du nom :

* `fromAttributes` : la valeur d’attribut pour les clés est utilisée pour créer un nom dans l’ordre spécifié dans la configuration. Toutes les clés d’attribut doivent être spécifiées dans l’étendue pour que le processeur la renomme.

Vous pouvez éventuellement configurer le paramètre suivant :

* `separator` : une chaîne, qui est spécifiée, sera utilisée pour fractionner les valeurs
> [!NOTE]
> Si le changement de nom dépend d’attributs en modifiés par le processeur d’attributs, assurez-vous que le processeur d’étendue est spécifié après le processeur d’attributs dans la spécification du pipeline.

#### <a name="sample-usage"></a>Exemple d’utilisation

L’exemple suivant spécifie que les valeurs d’attribut « db.svc », « operation », et « id » formeront le nouveau nom de l’étendue, dans cet ordre, séparées par la valeur « :: ».
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "span",
        "name": {
          "fromAttributes": [
            "db.svc",
            "operation",
            "id"
          ],
          "separator": "::"
        }
      }
    ]
  }
}
```

### <a name="extract-attributes-from-span-name"></a>Extraire des attributs d’une nom d’étendue

Prend une liste d’expressions régulières à mettre en correspondance avec le nom d’étendue, et en extrait des attributs en fonction de sous-expressions. Doit être spécifié dans la section `toAttributes`.

Les paramètres suivants sont requis :

`rules` : liste de règles pour l’extraction de valeurs d’attribut du nom d’étendue. Les valeurs dans le nom d’étendue sont remplacées par les noms d’attributs extraits. Chaque règle figurant dans la liste est une chaîne de modèle regex. Le nom d’étendue est vérifié par rapport à l’expression régulière. Si l’expression régulière correspond, toutes les sous-expressions nommées de l’expression régulière sont extraites en tant qu’attributs et ajoutées à l’étendue. Chaque nom de sous-expression devient un nom d’attribut, et la portion correspondante de la sous-expression devient la valeur d’attribut. La portion correspondante dans le nom d’étendue est remplacée par le nom d’attribut extrait. Si les attributs existent déjà dans l’étendue, ils sont remplacés. Le processus est répété pour toutes les règles dans l’ordre dans de leur spécification. Chaque règle subséquente fonctionne sur le nom d’étendue qui est la sortie résultant du traitement de la règle précédente.

#### <a name="sample-usage"></a>Exemple d’utilisation

Supposons que le nom d’étendue en entrée est /API/v1/document/12345678/update. L’application des résultats suivants dans le nom d’étendue en sortie /api/v1/document/{documentId}/update ajoute un nouvel attribut "documentId"="12345678" à l’étendue.
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "span",
        "name": {
          "toAttributes": {
            "rules": [
              "^/api/v1/document/(?<documentId>.*)/update$"
            ]
          }
        }
      }
    ]
  }
}
```

L’exemple suivant montre comment renommer le nom d’étendue en « {operation_website} », et ajouter l’attribut {Key: operation_website, Value: oldSpanName } lorsque l’étendue a les propriétés suivantes :
- Le nom d’étendue contient « / » quelque part dans la chaîne.
- Le nom d’étendue n’est pas « donot/change ».
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "span",
        "include": {
          "matchType": "regexp",
          "spanNames": [
            "^(.*?)/(.*?)$"
          ]
        },
        "exclude": {
          "matchType": "strict",
          "spanNames": [
            "donot/change"
          ]
        },
        "name": {
          "toAttributes": {
            "rules": [
              "(?<operation_website>.*?)$"
            ]
          }
        }
      }
    ]
  }
}
```