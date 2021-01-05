---
title: Syntaxe d’action SQL pour les règles d’abonnement Azure Service Bus | Microsoft Docs
description: Cet article fournit une référence pour la syntaxe d’action des règles SQL. Les actions sont écrites dans une syntaxe basée sur le langage SQL exécutée sur un message.
ms.topic: article
ms.date: 11/24/2020
ms.openlocfilehash: 7ce3332fb1a2025e89135e5e42e72d4afe1e7a5e
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489392"
---
# <a name="subscription-rule-sql-action-syntax"></a>Syntaxe d’action SQL pour les règles d’abonnement

Une *action SQL* permet de manipuler les métadonnées d’un message après qu’un message a été sélectionné par un filtre d’une règle d’abonnement. Il s’agit d’une expression textuelle qui s’appuie sur un sous-ensemble de la norme SQL-92. Les expressions d’action sont utilisées avec l’élément `sqlExpression` de la propriété « action » d’un objet `Rule` Service Bus dans un [modèle Azure Resource Manager](service-bus-resource-manager-namespace-topic-with-rule.md), avec l’argument [`--action-sql-expression`](/cli/azure/servicebus/topic/subscription/rule?preserve-view=true&view=azure-cli-latest#az_servicebus_topic_subscription_rule_create) de la commande Azure CLI `az servicebus topic subscription rule create` et avec plusieurs fonctions de Kit de développement logiciel (SDK) qui autorisent la gestion des règles d’abonnement.
  
  
```  
<statements> ::=
    <statement> [, ...n]  
  
```  
  
```  
<statement> ::=
    <action> [;]
    Remarks
    -------
    Semicolon is optional.  
  
```  
  
```  
<action> ::=
    SET <property> = <expression>
    REMOVE <property>  
```

```
<expression> ::=
    <constant>
    | <function>
    | <property>
    | <expression> { + | - | * | / | % } <expression>
    | { + | - } <expression>
    | ( <expression> )
``` 

```
<property> := 
    [<scope> .] <property_name>
``` 
  
## <a name="arguments"></a>Arguments  
  
-   `<scope>` est une chaîne facultative qui indique la portée de `<property_name>`. Les valeurs valides sont `sys` ou `user`. La valeur `sys` indique la portée du système où `<property_name>` est un nom de propriété publique de la [classe BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage). `user` indique la portée de l’utilisateur où `<property_name>` est une clé du dictionnaire de la [classe BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage). La portée de l’`user` est l’étendue par défaut si `<scope>` n’est pas défini.  
  
### <a name="remarks"></a>Remarques  

Une tentative d’accès à une propriété inexistante du système est une erreur, tandis qu’une tentative d’accès à une propriété d’utilisateur inexistante n’est pas une erreur. Au lieu de cela, une propriété d’utilisateur inexistante est évaluée en interne en tant que valeur inconnue. Une valeur inconnue est traitée spécialement lors de l’évaluation de l’opérateur.  
  
## <a name="property_name"></a>property_name  
  
```  
<property_name> ::=  
     <identifier>  
     | <delimited_identifier>  
  
<identifier> ::=  
     <regular_identifier> | <quoted_identifier> | <delimited_identifier>  
  
```  
  
### <a name="arguments"></a>Arguments  
 `<regular_identifier>` est une chaîne est représentée par l’expression régulière suivante :  
  
```  
[[:IsLetter:]][_[:IsLetter:][:IsDigit:]]*  
```  
  
 Cela signifie toute chaîne commençant par une lettre et suivie par un(e) ou plusieurs traits de soulignement/lettres/chiffres.  
  
 `[:IsLetter:]` signifie tout caractère Unicode classé en tant que lettre Unicode. `System.Char.IsLetter(c)` renvoie `true` si `c` est une lettre Unicode.  
  
 `[:IsDigit:]` signifie tout caractère Unicode classé en tant que chiffre décimal. `System.Char.IsDigit(c)` renvoie `true` si `c` est un chiffre Unicode.  
  
 Un `<regular_identifier>` ne peut pas être un mot-clé réservé.  
  
 `<delimited_identifier>` correspond à toute chaîne placée entre crochets ([]). Un crochet droit est représenté par deux crochets droits. Voici quelques exemples de `<delimited_identifier>` :  
  
```  
[Property With Space]  
[HR-EmployeeID]  
  
```  
  
 `<quoted_identifier>` correspond à toute chaîne placée entre guillemets doubles. Un guillemet double dans l’identificateur est représenté par deux guillemets doubles. Il est déconseillé d’utiliser des identificateurs entre guillemets, qui peuvent être facilement confondus avec une constante de chaîne. Utilisez si possible un identificateur délimité. Vous trouverez ci-dessous un exemple de `<quoted_identifier>` :  
  
```  
"Contoso & Northwind"  
```  
  
## <a name="pattern"></a>modèle  
  
```  
<pattern> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>Remarques
  
 `<pattern>` doit être une expression évaluée comme chaîne. Il est utilisé comme modèle pour l’opérateur LIKE.      Il peut contenir les caractères génériques suivants :  
  
-   `%` : toute chaîne de zéro caractère ou plus.  
  
-   `_` : n’importe quel caractère unique.  
  
## <a name="escape_char"></a>escape_char  
  
```  
<escape_char> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>Remarques
  
 `<escape_char>` doit être une expression évaluée comme chaîne dont la longueur est 1. Il est utilisé comme caractère d’échappement pour l’opérateur LIKE.  
  
 Par exemple, `property LIKE 'ABC\%' ESCAPE '\'` correspond à `ABC%` au lieu d’une chaîne qui commence par `ABC`.  
  
## <a name="constant"></a>constant  
  
```  
<constant> ::=  
      <integer_constant> | <decimal_constant> | <approximate_number_constant> | <boolean_constant> | NULL  
```  
  
### <a name="arguments"></a>Arguments  
  
-   `<integer_constant>` est une chaîne de nombres qui n’est pas entourée de guillemets et ne contient pas de décimales. Les valeurs sont stockées en tant que `System.Int64` en interne et suivent la même plage.  
  
     Voici quelques exemples de constantes longues :  
  
    ```  
    1894  
    2  
    ```  
  
-   `<decimal_constant>` est une chaîne de nombres qui n’est pas entourée de guillemets et qui contient une décimale. Les valeurs sont stockées en tant que `System.Double` en interne et suivent la même plage/précision.  
  
     Dans une version ultérieure, ce nombre pourrait être stocké dans un autre type de données pour prendre en charge la sémantique de nombre exacte. Vous n’aurez donc pas à compter sur le fait que le type de données sous-jacent soit `System.Double` pour `<decimal_constant>`.  
  
     Voici quelques exemples de constantes décimales :  
  
    ```  
    1894.1204  
    2.0  
    ```  
  
-   `<approximate_number_constant>` est un nombre écrit de manière scientifique. Les valeurs sont stockées en tant que `System.Double` en interne et suivent la même plage/précision. Voici des exemples de constantes numériques approximatives :  
  
    ```  
    101.5E5  
    0.5E-2  
    ```  
  
## <a name="boolean_constant"></a>boolean_constant  
  
```  
<boolean_constant> :=  
      TRUE | FALSE  
```  
  
### <a name="remarks"></a>Remarques
  
Les constantes booléennes sont représentées par les mots clés `TRUE` ou `FALSE`. Les valeurs sont stockées en tant que `System.Boolean`.  
  
## <a name="string_constant"></a>string_constant  
  
```  
<string_constant>  
```  
  
### <a name="remarks"></a>Remarques
  
Les constantes de chaîne sont placées entre guillemets simples et incluent tout caractère Unicode valide. Un guillemet simple intégré à une constante de chaîne est représenté par deux guillemets simples.  
  
## <a name="function"></a>function  
  
```  
<function> :=  
      newid() |  
      property(name) | p(name)  
```  
  
### <a name="remarks"></a>Remarques  

La fonction `newid()` renvoie un **System.Guid** généré par la méthode `System.Guid.NewGuid()`.  
  
La fonction `property(name)` renvoie la valeur de la propriété référencée par `name`. La valeur `name` peut être toute expression valide renvoyant une valeur de chaîne.  
  
## <a name="considerations"></a>Considérations

- SET est utilisé pour créer une nouvelle propriété ou mettre à jour la valeur d’une propriété existante.
- REMOVE permet de supprimer une propriété.
- SET effectue si possible une conversion implicite lorsque le type d’expression et le type de propriété existant sont différents.
- L’action échoue s’il est fait référence à des propriétés de système inexistantes.
- L’action réussit s’il est fait référence à des propriétés d’utilisateur inexistantes.
- Une propriété d’utilisateur inexistante est évaluée comme « Inconnue » en interne, suivant la même sémantique que [SQLFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter) lors de l’évaluation des opérateurs.

## <a name="next-steps"></a>Étapes suivantes

- [Classe SQLRuleAction (.NET Framework)](/dotnet/api/microsoft.servicebus.messaging.sqlruleaction)
- [Classe SQLRuleAction (.NET Standard)](/dotnet/api/microsoft.azure.servicebus.sqlruleaction)
- [Classe SqlRuleAction (Java)](/java/api/com.microsoft.azure.servicebus.rules.sqlruleaction)
- [SqlRuleAction (JavaScript)](/javascript/api/@azure/service-bus/sqlruleaction)
- [az servicebus topic subscription rule](/cli/azure/servicebus/topic/subscription/rule)
- [New-AzServiceBusRule](/powershell/module/az.servicebus/new-azservicebusrule)