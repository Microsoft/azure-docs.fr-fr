---
title: Règle personnalisée de pare-feu d’applications web pour Azure Front Door
description: Découvrez comment utiliser des règles personnalisées de pare-feu d’applications web pour protéger vos applications web contre les attaques malveillantes.
author: vhorne
ms.service: web-application-firewall
ms.topic: article
services: web-application-firewall
ms.date: 09/05/2019
ms.author: victorh
ms.openlocfilehash: 158bfe30bf48ee420be8efb9ff32fff0e555d9e7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "79475822"
---
#  <a name="custom-rules-for-web-application-firewall-with-azure-front-door"></a>Règles personnalisées pour le pare-feu d’applications web avec Azure Front Door

Le Pare-feu d’applications web Azure (WAF) doté de Front Door vous permet de contrôler l’accès à vos applications web en fonction des conditions que vous définissez. Une règle WAF personnalisée se compose d’un numéro de priorité, d’un type de règle, de conditions de correspondance et d’une action. Il existe deux types de règles personnalisées : les règles de correspondance et les règles de limite de taux. Une règle de correspondance contrôle un accès en fonction de conditions de correspondance, alors qu’une règle de limite de taux contrôle d’accès basée sur la correspondance des conditions et les taux de requêtes entrantes. Vous pouvez désactiver une règle personnalisée pour l’empêcher d’être évaluée, tout en conservant la configuration. 

## <a name="priority-match-conditions-and-action-types"></a>Priorité, conditions de correspondance et les types d’action

Vous pouvez contrôler l’accès à l’aide d’une règle WAF personnalisée qui définit un numéro de priorité, un type de règle, des conditions de correspondance et une action. 

- **Priorité :** est un entier unique qui décrit l’ordre des évaluations des règles de pare-feu d’applications web (WAF). Les règles avec des valeurs de priorité inférieure sont évaluées avant les règles avec des valeurs plus élevées. Les numéros de priorité doivent être uniques parmi toutes les règles personnalisées.

- **Action :** définit comment acheminer une requête si une règle de pare-feu d'applications web (WAF) est mise en correspondance. Vous pouvez choisir parmi les actions ci-dessous pour qu’elle soit appliquée quand une requête correspond à une règle personnalisée.

    - *Autoriser :*  le pare-feu d'applications web (WAF) transfère la requête au back-end, enregistre une entrée dans ses journaux et s’arrête.
    - *Bloquer :* la requête est bloquée, le pare-feu d'applications web (WAF) envoie la réponse au client sans la transférer au back-end. Le pare-feu d'applications web (WAF) journalise une entrée dans ses journaux.
    - *Journaliser :* le pare-feu d'applications web (WAF) enregistre une entrée dans ses journaux, puis passe à l’évaluation de la règle suivante.
    - *Rediriger :* le pare-feu d'applications web (WAF) redirige la requête vers un URI spécifié, enregistre une entrée dans ses journaux, puis s’arrête.

- **Condition de correspondance :** définit une variable de correspondance, un opérateur et une valeur de correspondance. Chaque règle peut contenir plusieurs conditions de correspondance. Une condition de correspondance peut être basée sur l’emplacement géographique, les adresses IP du client (CIDR), la taille ou la correspondance de chaîne. La correspondance de chaîne peut correspondre à une liste de variables de correspondance.
  - **Variable de correspondance :**
    - RequestMethod
    - QueryString
    - PostArgs
    - RequestUri
    - RequestHeader
    - RequestBody
    - Cookies
  - **Opérateur :**
    - Any : souvent utilisé pour définir l’action par défaut si aucune règle n’est mise en correspondance. Any est un opérateur qui fait correspondre tous les éléments.
    - Égal à
    - Contient
    - LessThan : contrainte de taille
    - GreaterThan : contrainte de taille
    - LessThanOrEqual : contrainte de taille
    - GreaterThanOrEqual : contrainte de taille
    - BeginsWith
    - EndsWith
    - Expression régulière
  
  - **Regex** ne prend pas en charge les opérations suivantes : 
    - Références arrière et capture de sous-expressions
    - Assertions arbitraires de largeur nulle
    - Références de sous-routines et modèles récursifs
    - Modèles conditionnels
    - Verbes de contrôle de retours sur trace
    - La directive \C mono-octet
    - La directive de correspondance de saut de ligne \R
    - La directive de début de correspondance de réinitialisation \K
    - Légendes et code incorporé
    - Regroupement atomique et quantificateurs de possession

  - **Négation [facultatif] :** Vous pouvez définir la condition *negate* sur la valeur true si le résultat d’une condition doit être inversé.
      
  - **Transformation [facultatif] :** Liste de chaînes avec les noms des transformations à effectuer avant la tentative de la mise en correspondance. Il peut s’agir des transformations suivantes :
     - Majuscules 
     - Minuscules
     - SupprEspace
     - RemoveNulls
     - UrlDecode
     - UrlEncode
     
   - **Valeur de correspondance :** Les valeurs de la méthode de requête HTTP prises en charge incluent :
     - GET
     - POST
     - PUT
     - HEAD
     - Suppression
     - LOCK
     - UNLOCK
     - PROFILE
     - OPTIONS
     - PROPFIND
     - PROPPATCH
     - MKCOL
     - COPY
     - MOVE

## <a name="examples"></a>Exemples

### <a name="waf-custom-rules-example-based-on-http-parameters"></a>Exemple de règles de pare-feu d'applications web (WAF) personnalisées selon les paramètres http

Voici un exemple qui illustre la configuration d’une règle personnalisée avec deux conditions de correspondance. Les requêtes proviennent d’un site spécifié, tel que défini par le point d’accès, et la chaîne de requête ne contient pas le terme « password » (« mot de passe »).

```
# http rules example
{
  "name": "AllowFromTrustedSites",
  "priority": 1,
  "ruleType": "MatchRule",
  "matchConditions": [
    {
      "matchVariable": "RequestHeader",
      "selector": "Referer",
      "operator": "Equal",
      "negateCondition": false,
      "matchValue": [
        "www.mytrustedsites.com/referpage.html"
      ]
    },
    {
      "matchVariable": "QueryString",
      "operator": "Contains",
      "matchValue": [
        "password"
      ],
      "negateCondition": true
    }
  ],
  "action": "Allow",
  "transforms": []
}

```
Un exemple de configuration pour le blocage de la méthode « PUT » est présenté ci-dessous :

``` 
# http Request Method custom rules
{
  "name": "BlockPUT",
  "priority": 2,
  "ruleType": "MatchRule",
  "matchConditions": [
    {
      "matchVariable": "RequestMethod",
      "selector": null,
      "operator": "Equal",
      "negateCondition": false,
      "matchValue": [
        "PUT"
      ]
    }
  ],
  "action": "Block",
  "transforms": []
}
```

### <a name="size-constraint"></a>Contrainte de taille

Vous pouvez générer une règle personnalisée qui spécifie la contrainte de taille de la part d’une requête entrante. Par exemple, la règle suivante bloque une URL qui dépasse les 100 caractères.

```
# http parameters size constraint
{
  "name": "URLOver100",
  "priority": 5,
  "ruleType": "MatchRule",
  "matchConditions": [
    {
      "matchVariable": "RequestUri",
      "selector": null,
      "operator": "GreaterThanOrEqual",
      "negateCondition": false,
      "matchValue": [
        "100"
      ]
    }
  ],
  "action": "Block",
  "transforms": []
}
```

## <a name="next-steps"></a>Étapes suivantes
- [Configurer une stratégie de pare-feu d’applications web à l’aide d’Azure PowerShell](waf-front-door-custom-rules-powershell.md) 
- En savoir plus sur le [pare-feu d’applications web avec Front Door](afds-overview.md)
- Découvrez comment [créer une porte d’entrée](../../frontdoor/quickstart-create-front-door.md).

