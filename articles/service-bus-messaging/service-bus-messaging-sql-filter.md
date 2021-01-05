---
title: Syntaxe de filtre SQL pour les règles d’abonnement Azure Service Bus | Microsoft Docs
description: Cet article fournit des détails sur la grammaire du filtre SQL. Un filtre SQL prend en charge un sous-ensemble de la norme SQL-92.
ms.topic: article
ms.date: 11/24/2020
ms.openlocfilehash: 9bff18b2161e419d728c360c9ed950ac2867fea8
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498674"
---
# <a name="subscription-rule-sql-filter-syntax"></a>Syntaxe de filtre SQL pour les règles d’abonnement

Un *filtre SQL* est l’un des types de filtres disponibles pour les abonnements aux rubriques Service Bus. Il s’agit d’une expression textuelle qui s’appuie sur un sous-ensemble de la norme SQL-92. Les expressions de filtre sont utilisées avec l’élément `sqlExpression` de la propriété « sqlFilter » d’un objet `Rule` Service Bus dans un [modèle Azure Resource Manager](service-bus-resource-manager-namespace-topic-with-rule.md), avec l’argument [`--filter-sql-expression`](/cli/azure/servicebus/topic/subscription/rule?preserve-view=true&view=azure-cli-latest#az_servicebus_topic_subscription_rule_create) de la commande Azure CLI `az servicebus topic subscription rule create` et avec plusieurs fonctions de Kit de développement logiciel (SDK) qui autorisent la gestion des règles d’abonnement.

Service Bus Premium prend également en charge la [syntaxe du sélecteur de messages SQL JMS](https://docs.oracle.com/javaee/7/api/javax/jms/Message.html) via l’API JMS 2.0.

  
``` 
<predicate ::=  
      { NOT <predicate> }  
      | <predicate> AND <predicate>  
      | <predicate> OR <predicate>  
      | <expression> { = | <> | != | > | >= | < | <= } <expression>  
      | <property> IS [NOT] NULL  
      | <expression> [NOT] IN ( <expression> [, ...n] )  
      | <expression> [NOT] LIKE <pattern> [ESCAPE <escape_char>]  
      | EXISTS ( <property> )  
      | ( <predicate> )  
  
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
  
-   `<scope>` est une chaîne facultative qui indique la portée de `<property_name>`. Les valeurs valides sont `sys` ou `user`. La valeur `sys` indique la portée du système où `<property_name>` est un nom de propriété publique de la [classe BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage). `user` indique la portée de l’utilisateur où `<property_name>` est une clé du dictionnaire de la [classe BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage). L’étendue de l’objet `user` est l’étendue par défaut si `<scope>` n’est pas défini.  
  
## <a name="remarks"></a>Remarques

Une tentative d’accès à une propriété système inexistante est une erreur, tandis qu’une tentative d’accès à une propriété utilisateur inexistante n’en est pas une. Au lieu de cela, une propriété d’utilisateur inexistante est évaluée en interne en tant que valeur inconnue. Une valeur inconnue est traitée spécialement lors de l’évaluation de l’opérateur.  
  
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
  
Un `<regular_identifier>` ne peut pas être un mot clé réservé.  
  
`<delimited_identifier>` correspond à toute chaîne placée entre crochets ([]). Un crochet droit est représenté par deux crochets droits. Voici quelques exemples de `<delimited_identifier>` :  
  
```  
[Property With Space]  
[HR-EmployeeID]  
  
```  
  
`<quoted_identifier>` correspond à toute chaîne placée entre guillemets doubles. Un guillemet double dans l’identificateur est représenté par deux guillemets doubles. Il est déconseillé d’utiliser des identificateurs entre guillemets, qui peuvent être facilement confondus avec une constante de chaîne. Utilisez si possible un identificateur délimité. Voici un exemple de `<quoted_identifier>` :  
  
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
  
     Dans une version ultérieure, ce nombre pourrait être stocké dans un autre type de données pour prendre en charge la sémantique exacte des nombres. Vous n’aurez donc pas à compter sur le fait que le type de données sous-jacent soit `System.Double` pour `<decimal_constant>`.  
  
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

Les constantes booléennes sont représentées par les mots-clés **TRUE** ou **FALSE**. Les valeurs sont stockées en tant que `System.Boolean`.  
  
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
  
La fonction `newid()` renvoie un `System.Guid` généré par la méthode `System.Guid.NewGuid()`.  
  
La fonction `property(name)` renvoie la valeur de la propriété référencée par `name`. La valeur `name` peut être toute expression valide renvoyant une valeur de chaîne.  
  
## <a name="considerations"></a>Considérations
  
Tenez compte de la sémantique [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter) suivante :  
  
-   Les noms de propriété respectent la casse.  
  
-   Les opérateurs suivent autant que possible la sémantique de conversion implicite C#.  
  
-   Les propriétés système sont des propriétés publiques exposées dans les instances [BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage).  
  
    Examinez la sémantique `IS [NOT] NULL` suivante :  
  
    -   `property IS NULL` est évalué comme `true` Si la propriété n’existe pas ou si la valeur de la propriété est `null`.  
  
### <a name="property-evaluation-semantics"></a>Sémantique d’évaluation de la propriété  
  
- Toute tentative pour évaluer une propriété de système qui n’existe pas lève une exception [FilterException](/dotnet/api/microsoft.servicebus.messaging.filterexception).  
  
- Une propriété qui n’existe pas est évaluée en interne comme **inconnue**.  
  
  Évaluation « inconnue » dans les opérateurs arithmétiques :  
  
- Pour les opérateurs binaires, si le côté gauche ou droit des opérandes est évalué comme **inconnu**, le résultat est **inconnu**.  
  
- Pour les opérateurs unaires, si un opérande est évalué comme **inconnu**, le résultat est **inconnu**.  
  
  Évaluation « inconnue » dans les opérateurs de comparaison binaires :  
  
- Si le côté gauche ou droit des opérandes est évalué comme **inconnu**, le résultat est **inconnu**.  
  
  Évaluation « inconnue » dans `[NOT] LIKE` :  
  
- Si un opérande est évalué comme **inconnu**, le résultat est **inconnu**.  
  
  Évaluation « inconnue » dans `[NOT] IN` :  
  
- Si l’opérande gauche est évalué comme **inconnu**, le résultat est **inconnu**.  
  
  Évaluation « inconnue » dans l’opérateur **AND** :  
  
```  
+---+---+---+---+  
|AND| T | F | U |  
+---+---+---+---+  
| T | T | F | U |  
+---+---+---+---+  
| F | F | F | F |  
+---+---+---+---+  
| U | U | F | U |  
+---+---+---+---+  
```  
  
 Évaluation « inconnue » dans l’opérateur **OR** :  
  
```  
+---+---+---+---+  
|OR | T | F | U |  
+---+---+---+---+  
| T | T | T | T |  
+---+---+---+---+  
| F | T | F | U |  
+---+---+---+---+  
| U | T | U | U |  
+---+---+---+---+  
```  
  
### <a name="operator-binding-semantics"></a>Sémantique de liaison d’opérateur
  
-   Les opérateurs de comparaison, tels que `>`, `>=`, `<`, `<=`, `!=`, et `=` suivent la même sémantique que la liaison d’opérateur C# dans les promotions de type de données et les conversions implicites.  
  
-   Les opérateurs arithmétiques, tels que `+`, `-`, `*`, `/`, et `%` suivent la même sémantique que la liaison d’opérateur C# dans les promotions de type de données et les conversions implicites.


## <a name="examples"></a>Exemples

### <a name="set-rule-action-for-a-sql-filter"></a>Définir l’action de la règle pour un filtre SQL

```csharp
// instantiate the ManagementClient
this.mgmtClient = new ManagementClient(connectionString);

// create the SQL filter
var sqlFilter = new SqlFilter("source = @stringParam");

// assign value for the parameter
sqlFilter.Parameters.Add("@stringParam", "orders");

// instantiate the Rule = Filter + Action
var filterActionRule = new RuleDescription
{
    Name = "filterActionRule",
    Filter = sqlFilter,
    Action = new SqlRuleAction("SET source='routedOrders'")
};

// create the rule on Service Bus
await this.mgmtClient.CreateRuleAsync(topicName, subscriptionName, filterActionRule);
```

### <a name="sql-filter-on-a-system-property"></a>Filtre SQL sur une propriété système

```csharp
sys.Label LIKE '%bus%'`
```

### <a name="using-or"></a>Utilisation de la clause OR 

```csharp
 sys.Label LIKE '%bus%'` OR `user.tag IN ('queue', 'topic', 'subscription')
```

### <a name="using-in-and-not-in"></a>Utilisation des clauses IN et NOT IN

```csharp
StoreId IN('Store1', 'Store2', 'Store3')"

sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'

sys.To NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8')
```

Pour un exemple C#, consultez [Exemple de filtres de rubrique sur GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Azure.Messaging.ServiceBus/BasicSendReceiveTutorialwithFilters).


## <a name="next-steps"></a>Étapes suivantes

- [Classe SQLFilter (.NET Framework)](/dotnet/api/microsoft.servicebus.messaging.sqlfilter)
- [Classe SQLFilter (.NET Standard)](/dotnet/api/microsoft.azure.servicebus.sqlfilter)
- [Classe SqlFilter (Java)](/java/api/com.microsoft.azure.servicebus.rules.SqlFilter)
- [SqlRuleFilter (JavaScript)](/javascript/api/@azure/service-bus/sqlrulefilter)
- [az servicebus topic subscription rule](/cli/azure/servicebus/topic/subscription/rule)
- [New-AzServiceBusRule](/powershell/module/az.servicebus/new-azservicebusrule)