---
title: Paramétrage du pare-feu d’applications web (WAF) pour Azure Front Door
description: Dans cet article, vous allez découvrir comment paramétrer le WAF pour Front Door.
services: web-application-firewall
author: mohitkusecurity
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 12/11/2020
ms.author: mohitku
ms.reviewer: tyao
ms.openlocfilehash: 4c710792dd7966fad76b33954fdf7c2253cf18f0
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488236"
---
# <a name="tuning-web-application-firewall-waf-for-azure-front-door"></a>Paramétrage du pare-feu d’applications web (WAF) pour Azure Front Door
 
L’ensemble de règles par défaut gérées par Azure s’appuie sur [l’ensemble CRS (Core Rule Set) de OWASP](https://github.com/SpiderLabs/owasp-modsecurity-crs/tree/v3.1/dev), et est conçu pour être strict dès sa première utilisation. Il est souvent nécessaire de paramétrer les règles du WAF pour répondre aux besoins particuliers de l’application ou de l’organisation qui utilise le pare-feu d’applications web. Pour cela, il suffit en général de définir des exclusions de règles, de créer des règles personnalisées et même de désactiver des règles pouvant être à l’origine de problèmes ou de faux positifs. Vous pouvez prendre plusieurs mesures si les demandes censées transiter par votre pare-feu d’applications web (WAF) sont bloquées.

Avant toute chose, prenez soin de lire les documents [Vue d’ensemble du WAF de Front Door](afds-overview.md) et [Stratégie WAF pour Front Door](waf-front-door-create-portal.md) si cela n’est pas déjà fait. Assurez-vous également que vous avez activé la [journalisation et la supervision du WAF](waf-front-door-monitor.md). Ces articles expliquent le fonctionnement du WAF ainsi que ses ensembles de règles, et l’accès aux journaux du WAF.
 
## <a name="understanding-waf-logs"></a>Compréhension des journaux du WAF
 
L’objectif des journaux du WAF est de répertorier chaque demande qui est mise en correspondance ou bloquée par le WAF. Il s’agit d’une collecte de toutes les requêtes évaluées qui sont mises en correspondance ou bloquées. Si vous remarquez que le WAF bloque une demande à tort (faux positif), vous pouvez agir de différentes manières. Tout d’abord, recherchez la demande en question par élimination. Afin d’identifier facilement l’événement et d’exécuter une requête de journal sur cette valeur spécifique, vous pouvez si vous le souhaitez [configurer un message de réponse personnalisé](./waf-front-door-configure-custom-response-code.md) pour inclure le champ `trackingReference`. Parcourez les journaux pour trouver l’URI, l’horodatage ou l’adresse IP du client de la requête. Lorsque vous avez trouvé les entrées de journal associées, vous pouvez commencer à agir sur les faux positifs. 
 
Par exemple, supposez que avez un trafic légitime contenant la chaîne `1=1` que vous voulez faire transiter par votre WAF. La requête se présente ainsi :

```
POST http://afdwafdemosite.azurefd.net/api/Feedbacks HTTP/1.1
Host: afdwafdemosite.azurefd.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 55

UserId=20&captchaId=7&captchaId=15&comment="1=1"&rating=3
```

Si vous tentez la demande, le WAF bloque le trafic qui contient votre chaîne *1=1* dans n’importe quel paramètre ou champ. Il s’agit d’une chaîne souvent associée à une attaque par injection de code SQL. Vous pouvez parcourir les journaux et trouver l’horodatage de la demande et les règles responsables du blocage/mise en correspondance.
 
Dans l’exemple suivant, nous explorons un journal `FrontdoorWebApplicationFirewallLog` créé en raison d’une correspondance de règle.
 
Dans le champ « requestUri », vous pouvez voir que la requête a été élaborée précisément pour `/api/Feedbacks/`. En poussant plus loin notre exploration, nous trouvons l’ID de règle `942110` dans le champ « ruleName ». Avec l’ID de règle, vous pouvez accéder au [référentiel officiel de OWASP ModSecurity Core Rule Set](https://github.com/coreruleset/coreruleset) et effectuer une recherche sur cet [ID de règle](https://github.com/coreruleset/coreruleset/blob/v3.1/dev/rules/REQUEST-942-APPLICATION-ATTACK-SQLI.conf), afin d’examiner son code et de comprendre exactement la mise en correspondance de cette règle. 
 
Ensuite, en vérifiant le champ `action`, nous nous apercevons que cette règle est définie pour bloquer les requêtes en cas de correspondance, et nous constatons que la requête a en fait été bloquée par le WAF, car `policyMode` est défini sur `prevention`. 
 
À présent, contrôlons les informations dans le champ `details`. C’est ici que vous pouvez voir les informations `matchVariableName` et `matchVariableValue`. Nous découvrons que cette règle a été déclenchée à cause de l’entrée *1=1* dans le champ `comment` de l’application web.
 
```json
{
    "time": "2020-09-24T16:43:04.5422943Z",
    "resourceId": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.NETWORK/FRONTDOORS/AFDWAFDEMOSITE",
    "category": "FrontdoorWebApplicationFirewallLog",
    "operationName": "Microsoft.Network/FrontDoor/WebApplicationFirewallLog/Write",
    "properties": {
        "clientIP": "1.1.1.1",
        "clientPort": "53566",
        "socketIP": "1.1.1.1",
        "requestUri": "http://afdwafdemosite.azurefd.net:80/api/Feedbacks/",
        "ruleName": "DefaultRuleSet-1.0-SQLI-942110",
        "policy": "AFDWAFDemoPolicy",
        "action": "Block",
        "host": "afdwafdemosite.azurefd.net",
        "trackingReference": "0mMxsXwAAAABEalekYeI4S55qpi5R7R0/V1NURURHRTA4MTIAZGI4NGQzZDgtNWQ5Ny00ZWRkLTg2ZGYtZDJjNThlMzI2N2I4",
        "policyMode": "prevention",
        "details": {
            "matches": [
                {
                    "matchVariableName": "PostParamValue:comment",
                    "matchVariableValue": "\"1=1\""
                }
            ],
            "msg": "SQL Injection Attack: Common Injection Testing Detected",
            "data": "Matched Data: \"1=1\" found within PostParamValue:comment: \"1=1\""
        }
    }
}
```
 
Il est également intéressant de vérifier les journaux d’accès pour approfondir sa compréhension d’un événement WAF donné. Ci-dessous, nous consultons le journal `FrontdoorAccessLog` qui a été généré en réponse à l’événement ci-dessus.
 
Vous pouvez voir qu’il s’agit de journaux liés, en raison de la valeur `trackingReference` qui est la même. Parmi les différents champs qui fournissent des informations générales, comme `userAgent` et `clientIP`, nous nous attardons sur les champs `httpStatusCode` et `httpStatusDetails`. Là, nous pouvons vérifier que le client a reçu une réponse HTTP 403, ce qui confirme formellement que cette requête a été refusée et bloquée. 
 
```json
{
    "time": "2020-09-24T16:43:04.5430764Z",
    "resourceId": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.NETWORK/FRONTDOORS/AFDWAFDEMOSITE",
    "category": "FrontdoorAccessLog",
    "operationName": "Microsoft.Network/FrontDoor/AccessLog/Write",
    "properties": {
        "trackingReference": "0mMxsXwAAAABEalekYeI4S55qpi5R7R0/V1NURURHRTA4MTIAZGI4NGQzZDgtNWQ5Ny00ZWRkLTg2ZGYtZDJjNThlMzI2N2I4",
        "httpMethod": "POST",
        "httpVersion": "1.1",
        "requestUri": "http://afdwafdemosite.azurefd.net:80/api/Feedbacks/",
        "requestBytes": "2160",
        "responseBytes": "324",
        "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36",
        "clientIp": "1.1.1.1",
        "socketIp": "1.1.1.1",
        "clientPort": "53566",
        "timeToFirstByte": "0.01",
        "timeTaken": "0.011",
        "securityProtocol": "",
        "routingRuleName": "DemoBERoutingRule",
        "rulesEngineMatchNames": [],
        "backendHostname": "13.88.65.130:3000",
        "isReceivedFromClient": true,
        "httpStatusCode": "403",
        "httpStatusDetails": "403",
        "pop": "WST",
        "cacheStatus": "CONFIG_NOCACHE"
    }
}
```

## <a name="resolving-false-positives"></a>Résolution des faux-positifs
 
Pour prendre une décision avisée sur le traitement d’un faux positif, il est important de se familiariser avec les technologies qu’utilise votre application. Par exemple, supposons que votre pile de technologies ne comporte pas de serveur SQL Server et que vous obtenez des faux positifs liés à ces règles. La désactivation de ces règles n’affaiblit pas nécessairement votre sécurité. 

Sachant cela, et compte tenu que la règle 942110 est celle qui a mis en correspondance la chaîne `1=1` dans notre exemple, nous pouvons prendre quelques mesures pour éviter le blocage de cette requête légitime :
 
* Utiliser les listes d’exclusions
  * Pour plus d’informations sur les listes d’exclusion, consultez [Pare-feu d’applications web (WAF) avec des listes d’exclusions Azure Front Door](waf-front-door-exclusion.md). 
* Changer les actions WAF
  * Pour plus d’informations sur les actions qui peuvent être effectuées lorsqu’une requête correspond aux conditions d’une règle, consultez [Actions WAF](afds-overview.md#waf-actions).
* Utiliser des règles personnalisées
  * Pour plus d’informations sur les règles personnalisées, consultez [Règles personnalisées pour un pare-feu d’applications web avec Azure Front Door](waf-front-door-custom-rules.md).
* Désactiver les règles 

> [!TIP]
> Lorsque vous choisissez une approche visant à autoriser les requêtes légitimes via le WAF, essayez de rendre celle-ci aussi restrictive que possible. Par exemple, il est préférable d’utiliser une liste d’exclusions plutôt que de désactiver une règle entièrement.

### <a name="using-exclusion-lists"></a>Utilisation de listes d’exclusions

Un des avantages de l’utilisation d’une liste d’exclusion réside dans le fait que seule la variable de correspondance que vous sélectionnez pour l’exclusion n’est plus inspectée pour cette requête donnée. Autrement dit, vous pouvez choisir entre des éléments spécifiques à exclure, comme les en-têtes de requête, les cookies de requête, les arguments de chaîne de requête ou les arguments de publication de corps de requête, si une certaine condition est remplie, plutôt que d’exclure la requête entière de l’inspection. Les autres variables non spécifiées de la requête seront toujours inspectées normalement.
 
Il est important de considérer une exclusion comme un paramètre global. Cela signifie que l’exclusion configurée s’appliquera à tout le trafic transitant par votre WAF, et pas seulement à une application web ou un URI particulier. Un problème peut se poser, par exemple si *1=1* est une requête valide dans le corps pour une application web donnée, mais pas pour les autres subissant la même stratégie WAF. S’il est judicieux d’utiliser différentes listes d’exclusion pour différentes applications, prévoyez d’utiliser des stratégies WAF différentes pour chaque application, et de les appliquer au serveur front-end de chaque application.
 
Lors de la configuration des listes d’exclusions pour les règles gérées, vous pouvez choisir d’exclure toutes les règles d’un ensemble de règles, toutes les règles d’un groupe de règles ou une règle seule. Une liste d’exclusions peut être configurée à l’aide de [PowerShell](/powershell/module/az.frontdoor/New-AzFrontDoorWafManagedRuleExclusionObject?view=azps-4.7.0&viewFallbackFrom=azps-3.5.0), de l’interface [Azure CLI](/cli/azure/ext/front-door/network/front-door/waf-policy/managed-rules/exclusion?view=azure-cli-latest#ext_front_door_az_network_front_door_waf_policy_managed_rules_exclusion_add), de l’[API REST](/rest/api/frontdoorservice/webapplicationfirewall/policies/createorupdate) ou du portail Azure.

* Exclusions au niveau de la règle
  * L’application d’exclusions au niveau de la règle signifie que les exclusions spécifiées ne seront pas analysées par rapport à cette seule règle uniquement, mais qu’elles seront toujours analysées par toutes les autres règles de l’ensemble de règles. En matière d’exclusions, il s’agit du niveau le plus granulaire, et il peut être utilisé pour affiner le paramétrage de l’ensemble de règles gérées, en fonction des informations que vous trouvez dans les journaux WAF lors de la résolution d’un problème d’événement.
* Exclusions au niveau du groupe de règles
  * L’application d’exclusions au niveau du groupe de règles signifie que les exclusions spécifiées ne seront pas analysées par rapport à cet ensemble précis de types de règles. Par exemple, si vous sélectionnez *SQLI* en tant que groupe de règles exclues, les exclusions de requête définies ne seront inspectées par aucune des règles propres à SQLI, mais elles continueront d’être inspectées par les règles d’autres groupes, tels que *PHP*, *RFI* ou *XSS*. Ce type d’exclusion peut être utile lorsque nous sommes sûrs que l’application n’est pas sujette à des types particuliers d’attaques. Par exemple, une application ne disposant pas de base de données SQL peut voir toutes les règles *SQLI* exclues sans que cela nuise à son niveau de sécurité.
* Exclusions au niveau de l’ensemble de règles 
  * L’application d’exclusions au niveau d’un ensemble de règles signifie que les exclusions spécifiées ne seront pas analysées par rapport aux règles de sécurité disponibles dans cet ensemble de règles. Il s’agit d’une exclusion complète, vous devez donc l’utiliser avec précaution.

Dans cet exemple, nous allons procéder à une exclusion au niveau le plus granulaire (en appliquant une exclusion à une règle seule) : nous cherchons à exclure la variable de correspondance **Nom des arguments de publication du corps de la requête** contenant `comment`. Le résultat est visible dans les journaux du pare-feu où figurent les détails de la variable de correspondance : `"matchVariableName": "PostParamValue:comment"`. L'attribut est `comment`. Il existe d’autres façons de rechercher ce nom d’attribut ; consultez [Rechercher les noms d’attributs d’une demande](#finding-request-attribute-names).

![Règles d’exclusion](../media/waf-front-door-tuning/exclusion-rules.png)

![Exclusion de règle pour une règle particulière](../media/waf-front-door-tuning/exclusion-rule.png)

À certaines occasions, il peut arriver que certains paramètres spécifiques soient transmis au WAF de manière peu intuitive. Par exemple, un jeton est transmis quand l’authentification repose sur Azure Active Directory. Ce jeton, `__RequestVerificationToken`, est généralement passé sous la forme d’un cookie de requête. Cependant, dans certains cas où les cookies sont désactivés, ce jeton est également transmis comme argument de publication de la requête. Ainsi, pour traiter les faux-positifs du jeton Azure AD, vous devez vous assurer que `__RequestVerificationToken` est ajouté à la liste d’exclusions pour `RequestCookieNames` et `RequestBodyPostArgsNames`.

Exclure sur un nom de champ (*sélecteur*) signifie que la valeur n’est plus évaluée par le WAF. Toutefois, le nom du champ lui-même continue d’être évalué ; dans de rares cas, il peut correspondre à une règle WAF et déclencher une action.

![Exclusion de règle pour un ensemble de règles](../media/waf-front-door-tuning/exclusion-rule-selector.png)

### <a name="changing-waf-actions"></a>Changement d’actions WAF

Une autre façon de gérer le comportement des règles WAF consiste à choisir l’action que le pare-feu effectuera lorsqu’une requête correspondra aux conditions d’une règle. Les actions disponibles sont : [Autoriser, Bloquer, Journaliser et Rediriger](afds-overview.md#waf-actions).

Dans cet exemple, nous avons remplacé l’action par défaut *Bloquer* par l’action *Journaliser* dans la règle 942110. Ce changement entraînera la journalisation de la requête par le WAF, et la poursuite de l’évaluation de cette même requête par rapport aux règles de priorité inférieure restantes.

![Actions WAF](../media/waf-front-door-tuning/actions.png)

Après avoir effectué la même requête, nous pouvons nous référer aux journaux et remarquer que cette requête correspondait à l’ID de règle 942110, et que le champ `action_s` indique désormais *Journaliser* au lieu de *Bloquer*. Nous avons ensuite développé la requête de journal pour inclure les informations `trackingReference_s` et voir ce qui s’est produit d’autre avec cette requête.

![Journal présentant plusieurs correspondances de règle](../media/waf-front-door-tuning/actions-log.png)

Il est intéressant de constater qu’une autre correspondance de règle SQLI se produit quelques millisecondes après le traitement de l’ID de règle 942110. La même requête a correspondu à l’ID de règle 942310, et cette fois-ci, l’action par défaut *Bloquer* a été déclenchée.

Pouvoir déterminer si plusieurs règles au sein d’un groupe de règles particulier correspondent et bloquent une requête donnée constitue un autre avantage de l’utilisation de l’action *Bloquer* lors du paramétrage ou du dépannage du WAF. Vous pouvez ensuite créer vos exclusions au niveau approprié, c’est-à-dire à celui de la règle ou du groupe de règles. 

### <a name="using-custom-rules"></a>Utilisation des règles personnalisées

Lorsque vous avez identifié ce qui est à l’origine d’une correspondance de règle WAF, vous pouvez utiliser des règles personnalisées pour ajuster la manière dont le WAF répond à l’événement. Les règles personnalisées sont traitées avant les règles gérées ; elles peuvent contenir plusieurs conditions, et [Autoriser, Refuser, Journaliser ou Rediriger](afds-overview.md#waf-actions) constituent leurs actions possibles. En cas de correspondance de règle, le moteur WAF arrête le traitement. Cela signifie que d’autres règles personnalisées avec une priorité inférieure, et des règles gérées ne sont plus exécutées.

Dans l’exemple ci-dessous, nous avons créé une règle personnalisée comportant deux conditions. La première condition recherche la valeur `comment` dans le corps de la requête. La deuxième condition recherche la valeur `/api/Feedbacks/` dans l’URI de la requête.

L’utilisation d’une règle personnalisée vous permet d’être extrêmement précis quand vous affinez le paramétrage des règles de votre WAF et que vous traitez des faux positifs. Dans ce cas, nous n’effectuons aucune action qui se baseraient uniquement sur la valeur du corps de la requête `comment`, ce qui peut exister sur plusieurs sites ou applications appliquant la même stratégie WAF. En incluant une autre condition pour qu’elle corresponde également à un URI de requête particulier `/api/Feedbacks/`, nous nous assurons que cette règle personnalisée s’applique véritablement à ce cas d’usage explicite que nous avons pris en compte. Ainsi, la même attaque, si elle est effectuée par rapport à des conditions différentes, sera toujours inspectée et empêchée par le moteur WAF.

![Journal](../media/waf-front-door-tuning/custom-rule.png)

Lorsque vous explorez le journal, vous pouvez remarquer que le champ `ruleName_s` contient le nom donné à la règle personnalisée que nous avons créée : `redirectcomment`. Dans le champ `action_s`, vous pouvez constater que l’action *Rediriger* a été effectuée pour cet événement. Dans le champ `details_matches_s`, nous pouvons voir que les détails des deux conditions ont été mises en correspondance.

### <a name="disabling-rules"></a>Désactiver les règles

Une autre façon de contourner un faux positif est de désactiver la règle qui a mis en correspondance l’entrée considérée malveillante par le WAF. Dans la mesure où vous avez analysé les journaux du WAF et identifié la règle 942110, vous pouvez la désactiver sur le portail Azure. Consultez [Personnaliser les règles du pare-feu d’applications web à l’aide du portail Azure](../ag/application-gateway-customize-waf-rules-portal.md#disable-rule-groups-and-rules).
 
La désactivation d’une règle présente un avantage lorsque vous êtes sûr que toutes les requêtes répondant à cette condition particulière sont en fait des requêtes légitimes, ou lorsque vous êtes sûr que la règle ne s’applique tout simplement pas à votre environnement (par exemple, la désactivation d’une règle d’injection SQL lorsque vous disposez de back-ends non-SQL). 
 
Par contre, la désactivation d’une règle est un paramètre global qui s’applique à tous les hôtes front-end associés à la stratégie WAF. Lorsque vous choisissez de désactiver une règle, vous risquez de laisser des vulnérabilités exposées sans protection ni détection pour d’autres hôtes front-end associés à la stratégie WAF.
 
Si vous souhaitez utiliser Azure PowerShell pour désactiver une règle gérée, consultez la documentation de l’objet [`PSAzureManagedRuleOverride`](/powershell/module/az.frontdoor/new-azfrontdoorwafmanagedruleoverrideobject?preserve-view=true&view=azps-4.7.0). Si vous souhaitez utiliser l’interface Azure CLI, consultez la documentation [`az network front-door waf-policy managed-rules override`](/cli/azure/ext/front-door/network/front-door/waf-policy/managed-rules/override?preserve-view=true&view=azure-cli-latest).

![Règles WAF](../media/waf-front-door-tuning/waf-rules.png)

## <a name="finding-request-fields"></a>Recherche de champs de requête

À l’aide d’un proxy de navigateur tel que [Fiddler](https://www.telerik.com/fiddler), vous pouvez inspecter les différentes requêtes et identifier quels champs spécifiques d’une page web sont appelés. C’est un moyen pratique lorsqu’il nous faut exclure certains champs de l’inspection à l’aide de listes d’exclusions dans le WAF.

### <a name="finding-request-attribute-names"></a>Rechercher les noms d’attributs d’une demande
 
Dans cet exemple, vous pouvez voir que le champ dans lequel la chaîne `1=1` a été entrée s’appelle `comment`. Ces données ont été transmises dans le corps d’une requête POST.

![Fiddler - Requête affichant le corps](../media/waf-front-door-tuning/fiddler-request-attribute-name.png)

Il s’agit d’un champ que vous pouvez exclure. Pour en savoir plus sur les listes d’exclusions, consultez [Listes d’exclusions du pare-feu d’applications web](./waf-front-door-exclusion.md). Vous pouvez exclure l’évaluation dans ce cas en configurant l’exclusion suivante :

![Règle d’exclusion](../media/waf-front-door-tuning/fiddler-request-attribute-name-exclusion.png)

L’examen des journaux du pare-feu peut vous renseigner sur les éléments que vous devez ajouter à la liste d’exclusion. Pour activer la journalisation, consultez [Supervision des journaux et des métriques dans Azure Front Door](./waf-front-door-monitor.md).

Examinez le journal du pare-feu dans le fichier `PT1H.json` pendant l’heure à laquelle s’est produite la requête que vous voulez inspecter. Les fichiers `PT1H.json` sont disponibles dans les conteneurs du compte de stockage où sont stockés les journaux de diagnostic `FrontDoorWebApplicationFirewallLog` et `FrontDoorAccessLog`.

Dans cet exemple, vous pouvez voir la règle qui a bloqué la requête (avec la même référence de transaction) et qui s’est produite exactement au même moment :

```json
{
    "time": "2020-09-24T16:43:04.5422943Z",
    "resourceId": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/<Resource Group Name>/PROVIDERS/MICROSOFT.NETWORK/FRONTDOORS/AFDWAFDEMOSITE",
    "category": "FrontdoorWebApplicationFirewallLog",
    "operationName": "Microsoft.Network/FrontDoor/WebApplicationFirewallLog/Write",
    "properties": {
        "clientIP": "1.1.1.1",
        "clientPort": "53566",
        "socketIP": "1.1.1.1",
        "requestUri": "http://afdwafdemosite.azurefd.net:80/api/Feedbacks/",
        "ruleName": "DefaultRuleSet-1.0-SQLI-942110",
        "policy": "AFDWAFDemoPolicy",
        "action": "Block",
        "host": "afdwafdemosite.azurefd.net",
        "trackingReference": "0mMxsXwAAAABEalekYeI4S55qpi5R7R0/V1NURURHRTA4MTIAZGI4NGQzZDgtNWQ5Ny00ZWRkLTg2ZGYtZDJjNThlMzI2N2I4",
        "policyMode": "prevention",
        "details": {
            "matches": [
                {
                    "matchVariableName": "PostParamValue:comment",
                    "matchVariableValue": "\"1=1\""
                }
            ],
            "msg": "SQL Injection Attack: Common Injection Testing Detected",
            "data": "Matched Data: \"1=1\" found within PostParamValue:comment: \"1=1\""
        }
    }
}
```

Du fait de vos connaissances sur le fonctionnement des ensembles de règles gérées par Azure (voir [Pare-feu d’applications web sur Azure Front Door](afds-overview.md)), vous savez que la règle dotée de la propriété *action: Block* bloque en fonction des données qui ont été mises en correspondance dans le corps de la requête. En y regardant de plus près, vous pouvez voir qu’elles ont correspondu à un modèle (`1=1`) et que le champ est nommé `comment`. Suivez les mêmes étapes que précédemment pour exclure le nom des arguments de publication du corps de la requête qui contient `comment`.

### <a name="finding-request-header-names"></a>Rechercher les noms d’en-tête d’une demande

Encore une fois, Fiddler est un outil utile quand il s’agit de rechercher les noms d’en-tête d’une demande. Dans la capture d’écran suivante figurent les en-têtes de la demande GET, à savoir Content-Type, User-Agent, etc. Vous pouvez également utiliser des en-têtes de requête pour créer des exclusions et des règles personnalisées dans le WAF.

![Fiddler - Requête affichant un en-tête](../media/waf-front-door-tuning/fiddler-request-header-name.png)

Une autre façon d’examiner les en-têtes de requête et de réponse consiste à regarder dans les outils de développement de votre navigateur, tel que Edge ou Chrome. Vous pouvez appuyer sur F12 ou cliquer avec le bouton droit sur -> **Inspecter** -> **Outils de développement**, puis sélectionner l’onglet **Network**. Chargez une page web, puis cliquez sur la demande que vous voulez inspecter.

![Network - Inspecteur de requête](../media/waf-front-door-tuning/network-inspector-request.png)

### <a name="finding-request-cookie-names"></a>Recherche les noms de cookies d’une demande

Si la demande contient des cookies, l’onglet Cookies peut être sélectionné pour les examiner dans Fiddler. Les informations sur les cookies peuvent également être utilisées pour créer des exclusions ou des règles personnalisées dans le WAF.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [pare-feu d’application web Azure](../overview.md).
- Découvrez comment [créer une porte d’entrée](../../frontdoor/quickstart-create-front-door.md).