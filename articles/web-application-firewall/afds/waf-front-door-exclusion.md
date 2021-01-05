---
title: Listes d’exclusions du pare-feu d’applications web dans Azure Front Door – Portail Azure
description: Cet article fournit des informations sur la configuration des listes d’exclusions dans Azure Front Door avec le Portail Azure.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 11/10/2020
ms.author: victorh
ms.topic: conceptual
ms.openlocfilehash: c2c84b508ee86ebdd82dbcc7040106142187c506
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563458"
---
# <a name="web-application-firewall-waf-with-front-door-service-exclusion-lists"></a>Pare-feu d’applications web (WAF) avec des listes d’exclusions Azure Front Door Service 

Parfois, le pare-feu d’applications web (WAF) peut bloquer une requête que vous souhaitez autoriser pour votre application. Par exemple, Active Directory insère des jetons qui sont utilisés pour l’authentification. Ces jetons peuvent contenir des caractères spéciaux susceptibles de déclencher un faux positif dans les règles du pare-feu d’applications web. Les listes d’exclusion du WAF vous permettent d’omettre certains attributs de la demande dans une évaluation par le WAF.  Une liste d’exclusions peut être configurée à l’aide de [PowerShell](/powershell/module/az.frontdoor/New-AzFrontDoorWafManagedRuleExclusionObject?view=azps-3.5.0), d’[Azure CLI](/cli/azure/ext/front-door/network/front-door/waf-policy/managed-rules/exclusion?view=azure-cli-latest#ext-front-door-az-network-front-door-waf-policy-managed-rules-exclusion-add), de l’[API REST](/rest/api/frontdoorservice/webapplicationfirewall/policies/createorupdate) ou du portail Azure. L’exemple suivant illustre la configuration avec le Portail Azure. 
## <a name="configure-exclusion-lists-using-the-azure-portal"></a>Configurer des listes d’exclusions à l’aide du Portail Azure
**Gérer les exclusions** est accessible à partir du portail WAF sous **Règles managées**

![Manage exclusion](../media/waf-front-door-exclusion/exclusion1.png)
![Manage exclusion_add](../media/waf-front-door-exclusion/exclusion2.png)

 Exemple de liste d’exclusions : ![Manage exclusion_define](../media/waf-front-door-exclusion/exclusion3.png)

Cet exemple exclut la valeur dans le champ d’en-tête *user*. Une requête valide peut inclure le champ *user* contenant une chaîne qui déclenche une règle d’injection SQL. Vous pouvez exclure le paramètre *user* dans ce cas, afin que la règle WAF n’évalue rien qui appartienne à ce champ.

Les attributs qui suivent peuvent être ajoutés aux listes d’exclusion par nom. Les valeurs des champs que vous utilisez ne sont pas évaluées par rapport aux règles WAF, mais leurs noms sont évalués. Les listes d’exclusion suppriment l’inspection de la valeur du champ.

* Nom de l’en-tête de la demande
* Nom de cookie de la demande
* Nom des arguments de chaîne de la demande
* Nom des arguments de publication du corps de la demande

Vous pouvez spécifier une correspondance exacte avec l'en-tête ou le corps d'une requête, un cookie ou un attribut de chaîne de requête  ou spécifier des correspondances partielles. Les opérateurs suivants sont les critères de correspondance pris en charge :

- **Est égal à** :  cet opérateur est utilisé pour une correspondance exacte. Par exemple, pour sélectionner l’en-tête **bearerToken**, utilisez l’opérateur d’égalité avec le sélecteur défini sur **bearerToken**.
- **Commence par** : cet opérateur correspond à tous les champs qui commencent par la valeur de sélecteur spécifiée.
- **Se termine par** :  cet opérateur correspond à tous les champs de demande qui se terminent par la valeur de sélecteur spécifiée.
- **Contient** : cet opérateur correspond à tous les champs de demande qui se contiennent la valeur de sélecteur spécifiée.
- **Est égal à** : Cet opérateur correspond à tous les champs de la requête. * est la valeur du sélecteur.

Les noms d’en-tête et de cookie se sont pas sensibles à la casse.

Si une valeur d’en-tête, de cookie, d’argument de publication ou d’argument de requête produit des faux positifs pour certaines règles, vous pouvez exclure cette partie de la requête de la prise en compte par la règle :


|matchVariableName des journaux WAF  |Exclusion de la règle dans le portail  |
|---------|---------|
|CookieValue:SOME_NAME        |Nom de cookie de la demande Égal à SOME_NAME|
|HeaderValue:SOME_NAME        |Nom de l’en-tête de la demande Égal à SOME_NAME|
|PostParamValue:SOME_NAME     |Nom des arguments de publication du corps de la demande Égal à SOME_NAME|
|QueryParamValue:SOME_NAME    |Nom des arguments de chaîne de requête Égal à SOME_NAME|


Actuellement, nous prenons uniquement en charge les exclusions de règle pour matchVariableNames ci-dessus dans les journaux WAF. Pour tout autre matchVariableNames, vous devez soit désactiver les règles qui donnent des faux positifs, soit créer une règle personnalisée qui autorise explicitement ces requêtes. En particulier, lorsque matchVariableName est CookieName, HeaderName, PostParamName ou QueryParamName, cela signifie que le nom lui-même déclenche la règle. L’exclusion de la règle ne propose pas de prise en charge pour ces matchVariableNames pour l’instant.


Si vous excluez les arguments de publication nommés *FOO* du corps de la demande, aucune règle ne doit afficher PostParamValue:FOO comme matchVariableName dans vos journaux WAF. Toutefois, vous pouvez toujours voir une règle où matchVariableName InitialBodyContents correspond à la valeur du paramètre de publication FOO, car les valeurs du paramètre de publication font partie d’InitialBodyContents.

Vous pouvez appliquer des listes d’exclusions à toutes les règles de l’ensemble de règles managées, aux règles d’un groupe de règles spécifique ou à une règle unique, comme indiqué dans l’exemple précédent.

## <a name="define-exclusion-based-on-web-application-firewall-logs"></a>Définir l’exclusion basée sur les journaux Web Application Firewall
 [Surveillance et journalisation du pare-feu d’applications web Azure](waf-front-door-monitor.md) affiche les détails correspondants d’une demande bloquée. Si une valeur d’en-tête, une valeur de cookie, une valeur d’argument de publication ou une valeur d’argument de requête produit un faux positif pour certaines règles, vous pouvez exclure cette partie de la requête de la règle. Le tableau suivant présente des exemples de valeurs de journaux WAF et les conditions d’exclusion correspondantes.

|matchVariableName des journaux WAF    |Exclusion de la règle dans le portail|
|--------|------|
|CookieValue:SOME_NAME  |Nom de cookie de la demande Égal à SOME_NAME|
|HeaderValue:SOME_NAME  |Nom de l’en-tête de la demande Égal à SOME_NAME|
|PostParamValue:SOME_NAME|  Nom des arguments de publication du corps de la demande Égal à SOME_NAME|
|QueryParamValue:SOME_NAME| Nom des arguments de chaîne de requête Égal à SOME_NAME|


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré vos paramètres du pare-feu d’applications web, découvrez comment afficher vos journaux d’activité WAF. Pour plus d’informations, consultez [Diagnostics Azure Front Door](../afds/waf-front-door-monitor.md).