---
title: Règles personnalisées du pare-feu d’applications web Azure (WAF) v2 sur Application Gateway
description: Cet article fournit une vue d’ensemble des règles personnalisées de la deuxième version (v2) du pare-feu d’applications web Azure (WAF) intégré à la passerelle Azure Application Gateway.
services: web-application-firewall
ms.topic: article
author: vhorne
ms.service: web-application-firewall
ms.date: 04/14/2020
ms.author: victorh
ms.openlocfilehash: c0f802f5113e38e811c110ee913099e76fa7be0b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81383810"
---
# <a name="custom-rules-for-web-application-firewall-v2-on-azure-application-gateway"></a>Règles personnalisées du pare-feu d’applications web v2 sur Azure Application Gateway

Le pare-feu d’applications web (WAF) v2 Azure Application Gateway est fourni avec un ensemble de règles préconfigurées, géré une plateforme, qui offre une protection contre de nombreux types d’attaques. Ces attaques comprennent l’exécution de scripts de site à site, l’injection de code SQL etc. Si vous êtes un administrateur de pare-feu d’applications web, vous pouvez écrire vos propres règles pour étoffer l’ensemble de règles de base (CRS). Vos règles peuvent bloquer ou autoriser le trafic demandé en fonction de critères de correspondance.

Les règles personnalisées vous permettent de créer vos propres règles évaluées pour chaque requête passant par le pare-feu d’applications Web (WAF). Ces règles ont une priorité plus élevée que les autres règles des ensembles de règles gérés. Les règles personnalisées contiennent un nom de règle, une priorité de règle et un tableau des conditions de correspondance. Si ces conditions sont remplies, une action est entreprise (pour autoriser ou bloquer).

Par exemple, vous pouvez bloquer toutes les demandes à partir d’une adresse IP de la plage 192.168.5.4/24. Dans cette règle, l’opérateur est *IPMatch*, matchValues correspond à la plage d’adresses IP (192.168.5.4/24) et l’action consiste à bloquer le trafic. Vous définissez également le nom et la priorité de la règle.

Les règles personnalisées prennent en charge l’utilisation de la logique de composition pour établir des règles plus avancées répondant à vos besoins de sécurité. Par exemple, (Condition 1 **et** Condition 2) **ou** Condition 3). Cet exemple signifie que si la Condition 1 **et** la Condition 2 sont remplies, **ou** si la Condition 3 est remplie, le pare-feu d’applications web doit effectuer l’action spécifiée dans la règle personnalisée.

Différentes conditions de correspondance au sein de la même règle sont toujours composées à l’aide de **et**. Par exemple, bloquer le trafic à partir d’une adresse IP spécifique, et seulement si un certain navigateur est utilisé.

Si vous souhaitez **ou** deux conditions différentes, les deux conditions doivent se trouver dans des règles différentes. Par exemple, bloquer le trafic à partir d’une adresse IP spécifique, ou seulement si un certain navigateur est utilisé.

> [!NOTE]
> Le nombre maximal de règles personnalisées WAF est de 100. Pour en savoir plus sur les limites de la passerelle Application Gateway, consultez [Limites, quotas et contraintes applicables aux services et abonnements Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md#application-gateway-limits).

Les expressions régulières sont également prises en charge dans les règles personnalisées, comme dans les ensembles de règles CRS. Pour plus de détails, voir les exemples 3 et 5 dans [Créer et utiliser des règles de pare-feu d’applications web personnalisées](create-custom-waf-rules.md).

## <a name="allowing-vs-blocking"></a>Autoriser et bloquer

Autoriser et bloquer le trafic est simple avec des règles personnalisées. Par exemple, vous pouvez bloquer tout le trafic entrant à partir d’une plage d’adresses IP. Vous pouvez créer une autre règle autorisant le trafic si la requête provient d’un navigateur spécifique.

Pour autoriser quelque chose, vérifiez que le paramètre `-Action` est défini sur **Autoriser**. Pour bloquer quelque chose, vérifiez que le paramètre `-Action` est défini sur **Bloquer**.

```azurepowershell
$AllowRule = New-AzApplicationGatewayFirewallCustomRule `
   -Name example1 `
   -Priority 2 `
   -RuleType MatchRule `
   -MatchCondition $condition `
   -Action Allow

$BlockRule = New-AzApplicationGatewayFirewallCustomRule `
   -Name example2 `
   -Priority 2 `
   -RuleType MatchRule `
   -MatchCondition $condition `
   -Action Block
```

Le `$BlockRule` précédent est mis en correspondance avec la règle personnalisée suivante dans Azure Resource Manager :

```json
"customRules": [
      {
        "name": "blockEvilBot",
        "priority": 2,
        "ruleType": "MatchRule",
        "action": "Block",
        "matchConditions": [
          {
            "matchVariables": [
              {
                "variableName": "RequestHeaders",
                "selector": "User-Agent"
              }
            ],
            "operator": "Contains",
            "negationConditon": false,
            "matchValues": [
              "evilbot"
            ],
            "transforms": [
              "Lowercase"
            ]
          }
        ]
      }
    ], 
```

Cette règle personnalisée contient un nom, une priorité, une action et le tableau des conditions de correspondance qui doivent être remplies pour que l’action puisse être effectuée. Pour en savoir plus sur ces champs, consultez les descriptions suivantes. Pour voir des exemples de règles personnalisées, consultez [Créer et utiliser des règles de pare-feu d’applications web personnalisées](create-custom-waf-rules.md).

## <a name="fields-for-custom-rules"></a>Champs de règles personnalisées

### <a name="name-optional"></a>Name [facultatif]

nom de la règle.  Il apparaît dans les journaux.

### <a name="priority-required"></a>Priorité [requise]

- Détermine l’ordre d’évaluation de la règle. Plus la valeur est faible, plus l’évaluation de la règle est récente. La plage autorisée est comprise entre 1 et 100. 
- Doit être unique parmi toutes les règles personnalisées. Une règle dont la priorité est égale à 40 est évaluée avant une règle dont la priorité est égale à 80.

### <a name="rule-type-required"></a>Type de règle [obligatoire]

Actuellement, ce doit être **MatchRule**.

### <a name="match-variable-required"></a>Variable de correspondance [obligatoire]

Doit être l’une des variables suivantes :

- Remaddr - Adresse IP/nom d’hôte de la connexion de l’ordinateur distant
- RequestMethod - Méthode de requête HTTP (GET, POST, PUT, DELETE etc.)
- Chaîne de requête - Variable de l’URI
- PostArgs – Arguments envoyés dans le corps POST. Les règles personnalisées utilisant cette variable de correspondance ne sont appliquées que si l’en-tête « Content-Type » est défini sur « application/x-www-form-urlencoded » et « multipart/form-data ».
- RequestUri - URI de la requête
- RequestHeaders - En-têtes de la requête
- RequestBody : contient le corps de la demande en tant que tout. Les règles personnalisées utilisant cette variable de correspondance ne sont appliquées que si l’en-tête « Content-Type » est défini sur « application/x-www-form-urlencoded ». 
- RequestCookies - Cookies de la requête

### <a name="selector-optional"></a>Sélecteur [facultatif]

Décrit le champ de la collection matchVariable. Par exemple, si matchVariable est RequestHeaders, le sélecteur peut être sur l’en-tête *User-Agent*.

### <a name="operator-required"></a>Opérateur [obligatoire]

Il doit s’agir de l’un des opérateurs suivants :

- IPMatch - Utilisé uniquement lorsque matchVariable est *RemoteAddr*
- Equal : indique que la saisie est identique à MatchValue
- Contient
- LessThan
- GreaterThan
- LessThanOrEqual
- GreaterThanOrEqual
- BeginsWith
- EndsWith
- Expression régulière
- Geomatch (préversion)

### <a name="negate-condition-optional"></a>Negate condition [facultatif]

Annule la condition actuelle.

### <a name="transform-optional"></a>Transform [facultatif]

Liste de chaînes avec les noms des transformations à effectuer avant la tentative de la mise en correspondance. Il peut s’agir des transformations suivantes :

- Minuscules
- SupprEspace
- UrlDecode
- UrlEncode 
- RemoveNulls
- HtmlEntityDecode

### <a name="match-values-required"></a>Match values [obligatoire]

Liste des valeurs selon lesquelles effectuer la mise en correspondance, ce qui peut être considéré comme étant soumis à l’opérateur *OR*. Par exemple, il peut s’agir d’adresses IP ou d’autres chaînes. Le format de la valeur dépend de l’opérateur précédent.

### <a name="action-required"></a>Action [obligatoire]

- Autoriser - Autorise la transaction, en ignorant toutes les autres règles. La demande spécifiée est ajoutée à la liste verte et, une fois la mise en correspondance effectuée, elle interrompt l’évaluation et est envoyée au pool principal. Les règles qui se trouvent sur la liste verte ne sont pas évaluées selon d’autres règles personnalisées ou managées.
- Bloquer - Bloque la transaction selon *SecDefaultAction* (mode de détection/prévention). Tout comme l’action d’autorisation, une fois la requête évaluée et ajoutée à la liste rouge, l’évaluation est interrompue et la requête bloquée. Toute demande subséquente répondant aux mêmes conditions n’est pas évaluée, mais simplement bloquée. 
- Journal - Autorise la règle à écrire dans le journal, mais autorise l’exécution du reste des règles pour évaluation. Les autres règles personnalisées consécutives sont évaluées par ordre de priorité, suivies des règles managées.

## <a name="geomatch-custom-rules-preview"></a>Règles personnalisées Geomatch (préversion)

Vous pouvez créer des règles personnalisées pour répondre aux besoins exacts de vos applications et stratégies de sécurité. Vous pouvez restreindre l’accès à vos applications web par pays/région. Pour plus d’informations, voir [Règles personnalisées de géocorrespondance (préversion)](geomatch-custom-rules.md).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous en savez plus sur les règles personnalisées, [créez vos propres règles personnalisées](create-custom-waf-rules.md).
