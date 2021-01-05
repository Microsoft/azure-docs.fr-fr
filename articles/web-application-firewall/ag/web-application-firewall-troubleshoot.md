---
title: Détecter un problème – Pare-feu d’applications web Azure
description: Cet article fournit des informations sur la résolution des problèmes liés au pare-feu d’applications web (WAF) pour Azure Application Gateway
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 11/14/2019
ms.author: ant
ms.topic: conceptual
ms.openlocfilehash: 483d261a8cc107d01cfb7a405eac43667d7efcc6
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92131834"
---
# <a name="troubleshoot-web-application-firewall-waf-for-azure-application-gateway"></a>Résoudre les problèmes liés au pare-feu d’applications web (WAF) pour Azure Application Gateway

Vous pouvez prendre plusieurs mesures si les demandes censées transiter par votre pare-feu d’applications web (WAF) sont bloquées.

Tout d’abord, veillez à lire les documents de [présentation du WAF](ag-overview.md) et de [configuration du WAF](application-gateway-waf-configuration.md). De même, vérifiez que vous avez activé la [supervision du WAF](../../application-gateway/application-gateway-diagnostics.md). Ces articles expliquent comment fonctionne le WAF et ses ensembles de règles et comment accéder aux journaux du WAF.

Les ensembles de règles OWASP sont conçues de manière à être très strictes et adaptées aux besoins spécifiques de l’application ou de l’organisation qui utilise WAF. Il est tout à fait normal, et même attendu dans de nombreux cas, de créer des exclusions, des règles personnalisées, et même de désactiver des règles qui peuvent causer des problèmes ou des faux positifs. Les stratégies par site et par URI permettent que ces changements n’affectent que des sites/URI spécifiques de sorte que tout changement ne devrait pas affecter d’autres sites qui pourraient ne pas être confrontés aux mêmes problèmes. 

## <a name="understanding-waf-logs"></a>Compréhension des journaux du WAF

L’objectif des journaux du WAF est de répertorier chaque demande qui est mise en correspondance ou bloquée par le WAF. Il s’agit d’un registre de toutes les demandes évaluées qui sont mises en correspondance ou bloquées. Si vous remarquez que le WAF bloque une demande à tort (faux positif), vous pouvez agir de différentes manières. Tout d’abord, recherchez la demande en question par élimination. Parcourez les journaux pour trouver l’URI, l’horodatage ou l’ID de transaction de la demande qui vous intéresse. Une fois que vous avez trouvé les entrées de journal associées, vous pouvez commencer à agir sur les faux positifs.

Par exemple, supposez que avez un trafic légitime contenant la chaîne *1=1* que vous voulez faire transiter par votre WAF. Si vous tentez la demande, le WAF bloque le trafic qui contient votre chaîne *1=1* dans n’importe quel paramètre ou champ. Il s’agit d’une chaîne souvent associée à une attaque par injection de code SQL. Vous pouvez parcourir les journaux et trouver l’horodatage de la demande et les règles responsables du blocage/mise en correspondance.

Dans l’exemple suivant, vous pouvez constater que quatre règles sont déclenchées dans la même demande (voir le champ TransactionId). La première indique qu’elle a été mise en correspondance, car l’utilisateur a utilisé une URL numérique/IP pour la demande, ce qui augmente le score d’anomalie de trois puisqu’il s’agit d’un avertissement. La règle suivante à avoir eu une correspondance est la règle 942130, qui est celle que vous recherchez. Vous pouvez constater que *1=1* figure dans le champ `details.data`. Cela augmente encore le score d’anomalie de trois, car il s’agit aussi d’un avertissement. En règle générale, chaque règle qui contient l’action **Matched** augmente le score d’anomalie. À ce stade le score d’anomalie est égal à six. Pour plus d’informations, consultez [Mode de calcul du score d’anomalie](ag-overview.md#anomaly-scoring-mode).

Les deux dernières entrées de journal montrent que la demande a été bloquée, car le score d’anomalie était suffisamment élevé. Ces entrées présentent une action différente des deux autres. Elles montrent qu’elles ont bel et bien *bloqué* la demande. Ces règles sont obligatoires et ne peuvent pas être désactivées. Elles ne devraient pas être considérées comme des règles, mais plutôt comme l’infrastructure de base des éléments internes du WAF.

```json
{ 
    "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2", 
    "operationName": "ApplicationGatewayFirewall", 
    "category": "ApplicationGatewayFirewallLog", 
    "properties": { 
        "instanceId": "appgw_3", 
        "clientIp": "167.220.2.139", 
        "clientPort": "", 
        "requestUri": "\/", 
        "ruleSetType": "OWASP_CRS", 
        "ruleSetVersion": "3.0.0", 
        "ruleId": "920350", 
        "message": "Host header is a numeric IP address", 
        "action": "Matched", 
        "site": "Global", 
        "details": { 
            "message": "Warning. Pattern match \\\"^[\\\\\\\\d.:]+$\\\" at REQUEST_HEADERS:Host. ", 
            "data": "40.90.218.160", 
            "file": "rules\/REQUEST-920-PROTOCOL-ENFORCEMENT.conf\\\"", 
            "line": "791" 
        }, 
        "hostname": "vm000003", 
        "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt" 
    } 
} 
{ 
    "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2", 
    "operationName": "ApplicationGatewayFirewall", 
    "category": "ApplicationGatewayFirewallLog", 
    "properties": { 
        "instanceId": "appgw_3", 
        "clientIp": "167.220.2.139", 
        "clientPort": "", 
        "requestUri": "\/", 
        "ruleSetType": "OWASP_CRS", 
        "ruleSetVersion": "3.0.0", 
        "ruleId": "942130", 
        "message": "SQL Injection Attack: SQL Tautology Detected.", 
        "action": "Matched", 
        "site": "Global", 
        "details": { 
            "message": "Warning. Pattern match \\\"(?i:([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)([\\\\\\\\d\\\\\\\\w]++)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)(?:(?:=|\\u003c=\\u003e|r?like|sounds\\\\\\\\s+like|regexp)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)\\\\\\\\2|(?:!=|\\u003c=|\\u003e=|\\u003c\\u003e|\\u003c|\\u003e|\\\\\\\\^|is\\\\\\\\s+not|not\\\\\\\\s+like|not\\\\\\\\s+regexp)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)(?!\\\\\\\\2)([\\\\\\\\d\\\\\\\\w]+)))\\\" at ARGS:text1. ", 
            "data": "Matched Data: 1=1 found within ARGS:text1: 1=1", 
            "file": "rules\/REQUEST-942-APPLICATION-ATTACK-SQLI.conf\\\"", 
            "line": "554" 
        }, 
        "hostname": "vm000003", 
        "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt" 
    } 
} 
{ 
    "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2", 
    "operationName": "ApplicationGatewayFirewall", 
    "category": "ApplicationGatewayFirewallLog", 
    "properties": { 
        "instanceId": "appgw_3", 
        "clientIp": "167.220.2.139", 
        "clientPort": "", 
        "requestUri": "\/", 
        "ruleSetType": "", 
        "ruleSetVersion": "", 
        "ruleId": "0", 
        "message": "Mandatory rule. Cannot be disabled. Inbound Anomaly Score Exceeded (Total Score: 8)", 
        "action": "Blocked", 
        "site": "Global", 
        "details": { 
            "message": "Access denied with code 403 (phase 2). Operator GE matched 5 at TX:anomaly_score. ", 
            "data": "", 
            "file": "rules\/REQUEST-949-BLOCKING-EVALUATION.conf\\\"", 
            "line": "57" 
        }, 
        "hostname": "vm000003", 
        "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt" 
    } 
} 
{ 
    "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2", 
    "operationName": "ApplicationGatewayFirewall", 
    "category": "ApplicationGatewayFirewallLog", 
    "properties": { 
        "instanceId": "appgw_3", 
        "clientIp": "167.220.2.139", 
        "clientPort": "", 
        "requestUri": "\/", 
        "ruleSetType": "", 
        "ruleSetVersion": "", 
        "ruleId": "0", 
        "message": "Mandatory rule. Cannot be disabled. Inbound Anomaly Score Exceeded (Total Inbound Score: 8 - SQLI=5,XSS=0,RFI=0,LFI=0,RCE=0,PHPI=0,HTTP=0,SESS=0): SQL Injection Attack: SQL Tautology Detected.", 
        "action": "Blocked", 
        "site": "Global", 
        "details": { 
            "message": "Warning. Operator GE matched 5 at TX:inbound_anomaly_score. ", 
            "data": "", 
            "file": "rules\/RESPONSE-980-CORRELATION.conf\\\"", 
            "line": "73" 
        }, 
        "hostname": "vm000003", 
        "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt" 
    }
}
```

## <a name="fixing-false-positives"></a>Correction des faux-positifs

Compte tenu de ces éléments et sachant que la règle 942130 est celle qui a mis en correspondance la chaîne *1=1* , vous pouvez prendre quelques mesures pour empêcher le blocage du trafic :

- Utiliser une liste d’exclusion.

   Pour plus d’informations sur les listes d’exclusion, consultez [Configuration du pare-feu WAF](application-gateway-waf-configuration.md#waf-exclusion-lists).
- Désactiver la règle.

### <a name="using-an-exclusion-list"></a>Utiliser une liste d’exclusion

Pour prendre une décision avisée sur le traitement d’un faux positif, il est important de se familiariser avec les technologies qu’utilise votre application. Par exemple, supposons que votre pile de technologies ne comporte pas de serveur SQL Server et que vous obtenez des faux positifs liés à ces règles. La désactivation de ces règles n’affaiblit pas nécessairement votre sécurité.

Un des avantages d’utiliser une liste d’exclusion est que seule une partie d’une demande est désactivée. Cependant, cela signifie qu’une exclusion spécifique est applicable à l’ensemble du trafic transitant par votre WAF, car il s’agit d’un paramètre global. Cela peut par exemple causer un problème si *1=1* est une demande valide dans le corps pour une application donnée, mais pas pour les autres. Un autre avantage est que vous pouvez choisir entre exclure le corps, les en-têtes et les cookies si une certaine condition est remplie, au lieu d’exclure la demande dans son intégralité.

À certaines occasions, il peut arriver que certains paramètres spécifiques soient transmis au WAF de manière peu intuitive. Par exemple, un jeton est transmis quand l’authentification repose sur Azure Active Directory. Ce jeton, *__RequestVerificationToken* , est généralement transmis dans le cookie d’une demande. Cependant, dans certains cas où les cookies sont désactivés, ce jeton est aussi transmis comme attribut de demande ou « arg ». Si cela se produit, vous devez veiller à ce que *__RequestVerificationToken* soit également ajouté à la liste d’exclusion comme **nom d’attribut de la demande** .

![Exclusions](../media/web-application-firewall-troubleshoot/exclusion-list.png)

Dans cet exemple, vous souhaitez exclure le **nom d’attribut de la demande** correspondant à *text1* . Cela est visible dans les journaux du pare-feu où figure le nom de l’attribut : **data: Matched Data: 1=1 found within ARGS:text1: 1=1** . L’attribut est **text1** . Il existe d’autres façons de rechercher ce nom d’attribut ; consultez [Rechercher les noms d’attributs d’une demande](#finding-request-attribute-names).

![Listes d’exclusions du WAF](../media/web-application-firewall-troubleshoot/waf-config.png)

### <a name="disabling-rules"></a>Désactiver les règles

Une autre façon de contourner un faux positif est de désactiver la règle qui a mis en correspondance l’entrée considérée malveillante par le WAF. Comme vous avez analysé les journaux du WAF et identifié la règle 942130, vous pouvez le désactiver sur le portail Azure. Consultez [Personnaliser les règles de pare-feu d’applications web via le portail Azure](application-gateway-customize-waf-rules-portal.md).

La désactivation d’une règle présente l’avantage suivant : si vous savez que l’ensemble du trafic contenant une certaine condition susceptible de le bloquer est du trafic valide, vous pouvez désactiver cette règle pour le WAF tout entier. Cependant, si le trafic n’est valide que dans un cas d’usage précis, vous créez une vulnérabilité en désactivant cette règle pour le pare-feu WAF entier, car il s’agit d’un paramètre global.

Si vous voulez utiliser Azure PowerShell, consultez [Personnaliser les règles de pare-feu d’applications web par le biais de PowerShell](application-gateway-customize-waf-rules-powershell.md). Si vous voulez utiliser Azure CLI, consultez [Personnaliser les règles de pare-feu d’applications web par le biais d’Azure CLI](application-gateway-customize-waf-rules-cli.md).

![Règles WAF](../media/web-application-firewall-troubleshoot/waf-rules.png)

## <a name="finding-request-attribute-names"></a>Rechercher les noms d’attributs d’une demande

À l’aide de [Fiddler](https://www.telerik.com/fiddler), inspectez les différentes demandes et identifiez les champs spécifiques d’une page web qui sont appelés. Cela peut vous aider à exclure certains champs de l’inspection avec des listes d’exclusion.

Dans cet exemple, vous pouvez voir que le champ dans lequel la chaîne *1=1* a été entrée s’appelle **text1** .

:::image type="content" source="../media/web-application-firewall-troubleshoot/fiddler-1.png" alt-text="Capture d’écran du débogueur web Progress Telerik Fiddler. Dans l’onglet Raw, 1 = 1 est visible après le nom text1." border="false":::

Il s’agit d’un champ que vous pouvez exclure. Pour en savoir plus sur les listes d’exclusion, consultez [Limites de la taille des demandes adressées au pare-feu d’applications web et listes d’exclusions](application-gateway-waf-configuration.md#waf-exclusion-lists). Vous pouvez exclure l’évaluation dans ce cas en configurant l’exclusion suivante :

![Exclusion du WAF](../media/web-application-firewall-troubleshoot/waf-exclusion-02.png)

L’examen des journaux du pare-feu peut vous renseigner sur les éléments que vous devez ajouter à la liste d’exclusion. Pour activer la journalisation, consultez [Intégrité du serveur principal, journaux de ressources et métriques pour la passerelle Application Gateway](../../application-gateway/application-gateway-diagnostics.md).

Examinez le journal du pare-feu et consultez le fichier PT1H.json correspondant à l’heure à laquelle s’est produite la demande que vous voulez inspecter.

Dans cet exemple, vous pouvez constater la présence de quatre règles avec le même TransactionID et qu’elles sont toutes intervenues exactement au même moment :

```json
-   {
-       "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2",
-       "operationName": "ApplicationGatewayFirewall",
-       "category": "ApplicationGatewayFirewallLog",
-       "properties": {
-           "instanceId": "appgw_3",
-           "clientIp": "167.220.2.139",
-           "clientPort": "",
-           "requestUri": "\/",
-           "ruleSetType": "OWASP_CRS",
-           "ruleSetVersion": "3.0.0",
-           "ruleId": "920350",
-           "message": "Host header is a numeric IP address",
-           "action": "Matched",
-           "site": "Global",
-           "details": {
-               "message": "Warning. Pattern match \\\"^[\\\\\\\\d.:]+$\\\" at REQUEST_HEADERS:Host. ",
-               "data": "40.90.218.160",
-               "file": "rules\/REQUEST-920-PROTOCOL-ENFORCEMENT.conf\\\"",
-               "line": "791"
-           },
-           "hostname": "vm000003",
-           "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt"
-       }
-   }
-   {
-       "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2",
-       "operationName": "ApplicationGatewayFirewall",
-       "category": "ApplicationGatewayFirewallLog",
-       "properties": {
-           "instanceId": "appgw_3",
-           "clientIp": "167.220.2.139",
-           "clientPort": "",
-           "requestUri": "\/",
-           "ruleSetType": "OWASP_CRS",
-           "ruleSetVersion": "3.0.0",
-           "ruleId": "942130",
-           "message": "SQL Injection Attack: SQL Tautology Detected.",
-           "action": "Matched",
-           "site": "Global",
-           "details": {
-               "message": "Warning. Pattern match \\\"(?i:([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)([\\\\\\\\d\\\\\\\\w]++)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)(?:(?:=|\\u003c=\\u003e|r?like|sounds\\\\\\\\s+like|regexp)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)\\\\\\\\2|(?:!=|\\u003c=|\\u003e=|\\u003c\\u003e|\\u003c|\\u003e|\\\\\\\\^|is\\\\\\\\s+not|not\\\\\\\\s+like|not\\\\\\\\s+regexp)([\\\\\\\\s'\\\\\\\"`\\\\\\\\(\\\\\\\\)]*?)(?!\\\\\\\\2)([\\\\\\\\d\\\\\\\\w]+)))\\\" at ARGS:text1. ",
-               "data": "Matched Data: 1=1 found within ARGS:text1: 1=1",
-               "file": "rules\/REQUEST-942-APPLICATION-ATTACK-SQLI.conf\\\"",
-               "line": "554"
-           },
-           "hostname": "vm000003",
-           "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt"
-       }
-   }
-   {
-       "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2",
-       "operationName": "ApplicationGatewayFirewall",
-       "category": "ApplicationGatewayFirewallLog",
-       "properties": {
-           "instanceId": "appgw_3",
-           "clientIp": "167.220.2.139",
-           "clientPort": "",
-           "requestUri": "\/",
-           "ruleSetType": "",
-           "ruleSetVersion": "",
-           "ruleId": "0",
-           "message": "Mandatory rule. Cannot be disabled. Inbound Anomaly Score Exceeded (Total Score: 8)",
-           "action": "Blocked",
-           "site": "Global",
-           "details": {
-               "message": "Access denied with code 403 (phase 2). Operator GE matched 5 at TX:anomaly_score. ",
-               "data": "",
-               "file": "rules\/REQUEST-949-BLOCKING-EVALUATION.conf\\\"",
-               "line": "57"
-           },
-           "hostname": "vm000003",
-           "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt"
-       }
-   }
-   {
-       "resourceId": "/SUBSCRIPTIONS/A6F44B25-259E-4AF5-888A-386FED92C11B/RESOURCEGROUPS/DEMOWAF_V2/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/DEMOWAF-V2",
-       "operationName": "ApplicationGatewayFirewall",
-       "category": "ApplicationGatewayFirewallLog",
-       "properties": {
-           "instanceId": "appgw_3",
-           "clientIp": "167.220.2.139",
-           "clientPort": "",
-           "requestUri": "\/",
-           "ruleSetType": "",
-           "ruleSetVersion": "",
-           "ruleId": "0",
-           "message": "Mandatory rule. Cannot be disabled. Inbound Anomaly Score Exceeded (Total Inbound Score: 8 - SQLI=5,XSS=0,RFI=0,LFI=0,RCE=0,PHPI=0,HTTP=0,SESS=0): SQL Injection Attack: SQL Tautology Detected.",
-           "action": "Blocked",
-           "site": "Global",
-           "details": {
-               "message": "Warning. Operator GE matched 5 at TX:inbound_anomaly_score. ",
-               "data": "",
-               "file": "rules\/RESPONSE-980-CORRELATION.conf\\\"",
-               "line": "73"
-           },
-           "hostname": "vm000003",
-           "transactionId": "AcAcAcAcAKH@AcAcAcAcAyAt"
-       }
-   }
```

Compte tenu du fonctionnement des ensembles de règles CRS et sachant que l’ensemble de règles CRS 3.0 utilise un système de calcul de score d’anomalie (voir [Pare-feu d’applications web pour Azure Application Gateway](ag-overview.md)), vous savez que les deux règles du bas avec la propriété **action: Blocked** bloquent en fonction du score total d’anomalie. Les règles à examiner plus particulièrement sont les deux du haut.

La première entrée est journalisée, car l’utilisateur a utilisé une adresse IP numérique pour accéder à Application Gateway, ce qui peut être ignoré dans ce cas.

La deuxième (règle 942130) est celle qui nous intéresse. En y regardant de plus près, vous pouvez voir qu’elle a été mise en correspondance avec un modèle (1=1) et que le champ est nommé **text1** . Suivez les mêmes étapes que précédemment pour exclure le **Nom d’attribut de la demande** qui **correspond** à **1=1** .

## <a name="finding-request-header-names"></a>Rechercher les noms d’en-tête d’une demande

Encore une fois, Fiddler est un outil utile quand il s’agit de rechercher les noms d’en-tête d’une demande. Dans la capture d’écran suivante figurent les en-têtes de la demande GET, à savoir *Content-Type* , *User-Agent* , etc.

:::image type="content" source="../media/web-application-firewall-troubleshoot/fiddler-2.png" alt-text="Capture d’écran du débogueur web Progress Telerik Fiddler. L’onglet Raw répertorie les détails contenus dans l’en-tête de demande, tels que les variables connection, content-type et user-agent." border="false":::

Une autre façon d’examiner les en-têtes de demande et de réponse est de regarder dans les outils de développement de Chrome. Vous pouvez appuyer sur F12 ou cliquer avec le bouton droit sur -> **Inspecter** -> **Outils de développement** , puis sélectionner l’onglet **Network** . Chargez une page web, puis cliquez sur la demande que vous voulez inspecter.

![Chrome F12](../media/web-application-firewall-troubleshoot/chrome-f12.png)

## <a name="finding-request-cookie-names"></a>Recherche les noms de cookies d’une demande

Si la demande contient des cookies, l’onglet **Cookies** peut être sélectionné pour les examiner dans Fiddler.

## <a name="restrict-global-parameters-to-eliminate-false-positives"></a>Restreindre les paramètres globaux pour éliminer les faux positifs

- Désactiver l’inspection du corps de la demande

   En désactivant **Inspecter le corps de la demande** , votre pare-feu n’évalue pas le corps des demandes pour l’ensemble du trafic. Cela peut être utile si vous savez que le corps des demandes n’est pas malveillant pour votre application.

   En désactivant cette option, seul le corps de la demande n’est pas inspecté. Les en-têtes et les cookies sont inspectés, sauf ceux qui ont été exclus avec la fonctionnalité de liste d’exclusion.

- Limites de taille de fichiers

   En limitant la taille de fichier pour votre pare-feu WAF, vous limitez la possibilité d’une attaque ciblant vos serveurs web. En autorisant le chargement de fichiers volumineux, vous augmentez le risque de saturation de votre back-end. Limiter la taille de fichier à un cas d’usage normal pour votre application est simplement une autre façon de prévenir les attaques.

   > [!NOTE]
   > Si vous savez que votre application n’aura jamais besoin de charger des fichiers au-delà d’une taille donnée, vous pouvez ajouter une restriction en définissant une limite.

## <a name="firewall-metrics-waf_v1-only"></a>Métriques de pare-feu (WAF_v1 uniquement)

Pour les pare-feu d’applications web v1, les métriques suivantes sont désormais disponibles dans le portail : 

1. Nombre de demandes bloquées par le pare-feu d’applications web : nombre de demandes qui ont été bloquées
2. Nombre de règles bloquées par le pare-feu d’applications web : toutes les règles qui correspondaient **et** la demande ont été bloquées
3. Distribution totale des règles de pare-feu d’applications web : toutes les règles qui correspondaient pendant l’évaluation
     
Pour activer les métriques, sélectionnez l’onglet **Métriques** dans le portail, puis sélectionnez l’une des trois métriques.

## <a name="next-steps"></a>Étapes suivantes

Consultez le [Guide pratique pour configurer le pare-feu d’applications web dans Application Gateway](tutorial-restrict-web-traffic-powershell.md).
