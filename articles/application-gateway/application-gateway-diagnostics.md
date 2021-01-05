---
title: Journaux de diagnostic et de l’intégrité back-end
titleSuffix: Azure Application Gateway
description: Découvrez comment activer et gérer les journaux d’activité d’accès et les journaux d’activité des performances pour Azure Application Gateway
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 11/22/2019
ms.author: victorh
ms.openlocfilehash: 9d2fba48762e11a92f6f3925c969b01aebbe3630
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347589"
---
# <a name="back-end-health-and-diagnostic-logs-for-application-gateway"></a>Intégrité du serveur principal et journaux de diagnostic pour la passerelle Application Gateway

Vous pouvez superviser les ressources Azure Application Gateway des manières suivantes :

* [Intégrité du serveur principal](#back-end-health) : Application Gateway permet de superviser l’intégrité des serveurs dans les pools de back-ends au moyen du portail Azure et de PowerShell. Vous pouvez également accéder à l’intégrité des pools principaux via les journaux de diagnostic des performances.

* [Journaux d’activité](#diagnostic-logging) : les journaux d’activité permettent d’enregistrer ou d’utiliser les performances, les accès et les autres données à partir d’une ressource à des fins de supervision.

* [Métriques](application-gateway-metrics.md) : Application Gateway a plusieurs métriques qui vous permettent de vérifier que votre système fonctionne comme prévu.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="back-end-health"></a>Intégrité du serveur principal

La passerelle Application Gateway permet de surveiller l’intégrité des membres individuels des pools principaux au moyen du portail, de PowerShell et de l’interface de ligne de commande (CLI). Une synthèse de l’intégrité agrégée des pools principaux est également accessible via les journaux de diagnostic des performances.

Le rapport d’intégrité du serveur principal reflète les résultats de la sonde d’intégrité de la passerelle Application Gateway sur les instances de serveur principal. Si la détection réussit et que le trafic peut être orienté vers le serveur principal, ce dernier est considéré comme intègre. Sinon, il est considéré comme défaillant.

> [!IMPORTANT]
> Si le sous-réseau Application Gateway comporte un groupe de sécurité réseau (NSG), ouvrez les plages de ports 65503-65534 pour les références SKU v1 et 65200-65535 pour les références SKU v2 sur ce sous-réseau pour permettre l’arrivée du trafic entrant. Cette plage de ports est nécessaire pour la communication avec l’infrastructure Azure. Ils sont protégés (verrouillés) par des certificats Azure. Sans les certificats appropriés, les entités externes (notamment les clients de ces passerelles) ne pourront initier aucun changement sur ces points de terminaison.


### <a name="view-back-end-health-through-the-portal"></a>Affichage de l’intégrité du serveur principal via le portail

Dans le portail, l’intégrité du serveur principal est indiquée automatiquement. Dans une passerelle Application Gateway existante, sélectionnez **Analyse** > **Intégrité du serveur principal**.

Chaque membre du pool principal est répertorié sur cette page (qu’il s’agisse d’une carte réseau, d’un IP ou d’un nom de domaine complet). Le nom du pool principal, le port, le nom et l’état d’intégrité des paramètres HTTP principaux sont affichés. Les valeurs valides de l’état d’intégrité sont **Healthy** (Intègre), **Unhealthy** (Défaillant) et **Unknown** (Inconnu).

> [!NOTE]
> Si l’état d’intégrité du serveur principal affiché est **Unknown** (Inconnu), assurez-vous que l’accès au serveur principal n’est pas bloqué par une règle de groupe de sécurité réseau, un itinéraire défini par l’utilisateur ou un DNS personnalisé dans le réseau virtuel.

![Intégrité du serveur principal][10]

### <a name="view-back-end-health-through-powershell"></a>Affichage de l’intégrité du serveur principal via PowerShell

Le code PowerShell suivant montre comment afficher l’intégrité du serveur principal en utilisant l’applet de commande `Get-AzApplicationGatewayBackendHealth` :

```powershell
Get-AzApplicationGatewayBackendHealth -Name ApplicationGateway1 -ResourceGroupName Contoso
```

### <a name="view-back-end-health-through-azure-cli"></a>Affichage de l’intégrité du serveur principal via Azure CLI

```azurecli
az network application-gateway show-backend-health --resource-group AdatumAppGatewayRG --name AdatumAppGateway
```

### <a name="results"></a>Résultats

L’extrait de code suivant montre un exemple de la réponse :

```json
{
"BackendAddressPool": {
    "Id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/appGatewayBackendPool"
},
"BackendHttpSettingsCollection": [
    {
    "BackendHttpSettings": {
        "Id": "/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings"
    },
    "Servers": [
        {
        "Address": "hostname.westus.cloudapp.azure.com",
        "Health": "Healthy"
        },
        {
        "Address": "hostname.westus.cloudapp.azure.com",
        "Health": "Healthy"
        }
    ]
    }
]
}
```

## <a name="diagnostic-logs"></a><a name="diagnostic-logging"></a>Journaux de diagnostic

Vous pouvez utiliser différents types de journaux d’activité dans Azure pour gérer les passerelles Application Gateway et résoudre les problèmes associés. Vous pouvez accéder à certains de ces journaux d’activité via le portail. Tous les journaux d’activité peuvent être extraits à partir d’un stockage Blob Azure et affichés dans différents outils, comme les [journaux d’activité Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md), Excel et Power BI. Pour en savoir plus sur les différents types de journaux d’activité, consultez la liste suivante :

* **Journal d’activité** : vous pouvez utiliser le [Journal d’activité Azure](../azure-resource-manager/management/view-activity-logs.md) (anciennement journaux d’activité des opérations et journaux d’audit) pour voir toutes les opérations soumises à votre abonnement Azure, ainsi que leur état. Les entrées du journal d’activité sont recueillies par défaut et vous pouvez les afficher dans le Portail Azure.
* **Journal d’accès** : Vous pouvez utiliser ce journal pour voir les modèles d’accès Application Gateway et analyser les informations importantes. Ceci comprend l’adresse IP de l’appelant, l’URL demandée, la latence de réponse, le code de retour et les octets d’entrée et de sortie. Un journal d’accès est collecté toutes les 60 secondes. Ce journal contient un enregistrement par instance Application Gateway. L’instance de la passerelle Application Gateway est identifiée par la propriété instanceId.
* **Journaux de performances** : vous pouvez utiliser ce journal pour afficher les performances des instances de la passerelle Application Gateway. Ce journal capture des informations sur les performances de chaque instance, notamment le nombre total de requêtes traitées, le débit en octets, le nombre total de requêtes présentées, le nombre de requêtes ayant échoué, le nombre d’instances du serveur principal intègres et défectueuses. Le journal des performances est collecté toutes les 60 secondes. Le journal des performances est uniquement disponible pour la référence SKU v1. S’il s’agit de la référence SKU v2, utilisez les [Métriques](application-gateway-metrics.md) pour les données de performances.
* **Journal du pare-feu** : vous pouvez utiliser ce journal pour afficher les requêtes consignées via le mode de détection ou de prévention d’une passerelle Application Gateway configuré avec un pare-feu d’applications web. Les journaux du pare-feu sont collectés toutes les 60 secondes. 

> [!NOTE]
> Les journaux d’activité ne sont disponibles que pour les ressources déployées dans le modèle de déploiement Azure Resource Manager. Vous ne pouvez pas les utiliser pour les ressources utilisant le modèle de déploiement classique. Pour mieux comprendre ces deux modèles, consultez l’article [Présentation du déploiement de Resource Manager et du déploiement classique](../azure-resource-manager/management/deployment-models.md) .

Pour stocker vos journaux d’activité, vous disposez de trois options :

* **Compte de stockage** : les comptes de stockage conviennent parfaitement aux journaux d’activité quand ils sont stockés pour une durée plus longue et consultés quand cela est nécessaire.
* **Hubs d’événements** : les hubs d’événements constituent une excellente solution pour l’intégration à d’autres outils SIEM (Security Information and Event Management) afin de recevoir des alertes sur vos ressources.
* **Journaux d’activité Azure Monitor** : Les journaux d’activité Azure Monitor conviennent parfaitement pour la supervision en temps réel générale de votre application ou la recherche de tendances.

### <a name="enable-logging-through-powershell"></a>Activation de la journalisation avec PowerShell

La journalisation d’activité est automatiquement activée pour chaque ressource Resource Manager. Vous devez activer la journalisation de l’accès et des performances pour commencer à collecter les données disponibles dans ces journaux d’activité. Pour activer la journalisation, utilisez les étapes suivantes :

1. Notez l’ID de ressource de votre compte de stockage, où les données de journalisation sont stockées. Cette valeur se présente sous la forme : /subscriptions/\<subscriptionId\>/resourceGroups/\<resource group name\>/providers/Microsoft.Storage/storageAccounts/\<storage account name\>. Vous pouvez utiliser n’importe quel compte de stockage dans votre abonnement. Vous pouvez utiliser le portail Azure pour rechercher ces informations.

    ![Portail : ID de ressource du compte de stockage](./media/application-gateway-diagnostics/diagnostics1.png)

2. Notez l’ID de ressource de votre passerelle Application Gateway pour laquelle la journalisation est activée. Cette valeur se présente sous la forme : /subscriptions/\<subscriptionId\>/resourceGroups/\<resource group name\>/providers/Microsoft.Network/applicationGateways/\<application gateway name\>. Vous pouvez utiliser le portail pour rechercher ces informations.

    ![Portail : ID de ressource de la passerelle Application Gateway](./media/application-gateway-diagnostics/diagnostics2.png)

3. Activez la journalisation des diagnostics à l’aide de l’applet de commande PowerShell suivante :

    ```powershell
    Set-AzDiagnosticSetting  -ResourceId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name> -StorageAccountId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name> -Enabled $true     
    ```

> [!TIP]
>Les journaux d’activité ne nécessitent pas de compte de stockage distinct. L’utilisation du stockage pour la journalisation de l’accès et des performances occasionne des frais de service.

### <a name="enable-logging-through-the-azure-portal"></a>Activation de la journalisation avec le portail Azure

1. Sur le portail Azure, recherchez votre ressource, puis sélectionnez **Paramètres de diagnostic**.

   Pour Application Gateway, trois journaux d’activité d’audit sont disponibles :

   * Journal d’accès
   * Journal des performances
   * Journal du pare-feu

2. Sélectionnez **Activer les diagnostics** pour démarrer la collecte de données.

   ![Activation des diagnostics][1]

3. La page **Paramètres de diagnostic** contient les paramètres des journaux de diagnostic. Dans cet exemple, Log Analytics stocke les journaux d’activité. Vous pouvez également utiliser des concentrateurs d’événements et un compte de stockage pour enregistrer les journaux de diagnostic.

   ![Démarrage du processus de configuration][2]

5. Tapez un nom pour les paramètres, confirmez les paramètres, puis sélectionnez **Enregistrer**.

### <a name="activity-log"></a>Journal d’activité

Par défaut, Azure génère le journal d’activité. Les journaux d’activité sont conservés pendant 90 jours dans la banque de journaux d’événements d’Azure. Pour en savoir plus sur ces journaux d’activité, lisez l’article [Affichage des événements et du journal d’activité](../azure-resource-manager/management/view-activity-logs.md).

### <a name="access-log"></a>Journal d’accès

Le journal d’accès n’est généré que si vous l’avez activé sur chaque instance Application Gateway, comme détaillé dans les étapes précédentes. Les données sont stockées dans le compte de stockage spécifié lors de l’activation de la journalisation. Chaque accès à Application Gateway est journalisé au format JSON, comme indiqué ci-dessous. 

#### <a name="for-application-gateway-standard-and-waf-sku-v1"></a>Pour le niveau tarifaire Application Gateway Standard et WAF (v1)

|Valeur  |Description  |
|---------|---------|
|instanceId     | Instance Application Gateway ayant traité la requête.        |
|clientIP     | Adresse IP d’origine de la requête.        |
|clientPort     | Port d’origine de la requête.       |
|httpMethod     | Méthode HTTP utilisée par la requête.       |
|requestUri     | URI de la requête reçue.        |
|RequestQuery     | **Acheminée par le serveur** : instance de pool principal à laquelle la requête a été envoyée.</br>**X-AzureApplicationGateway-LOG-ID** : ID de corrélation utilisé pour la requête. Peut être utilisée pour résoudre les problèmes de trafic sur les serveurs principaux. </br>**ÉTAT DU SERVEUR** : code de réponse HTTP reçu par Application Gateway à partir du back-end.       |
|UserAgent     | Agent utilisateur de l’en-tête de requête HTTP.        |
|httpStatus     | Code d’état HTTP renvoyé au client à partir de d’Application Gateway.       |
|httpVersion     | Version HTTP de la requête.        |
|receivedBytes     | Taille du paquet reçu, en octets.        |
|sentBytes| Taille du paquet envoyé, en octets.|
|timeTaken| Durée (en millisecondes) nécessaire pour le traitement d’une requête et l’envoi de la réponse. Elle est calculée en fonction de l’intervalle entre le moment où Application Gateway reçoit le premier octet d’une requête HTTP et le moment où l’opération d’envoi d’une réponse se termine. Il est important de noter que le champ Time-Taken inclut généralement l’heure à laquelle la requête et les paquets de réponse circulent sur le réseau. |
|sslEnabled| Détermine si la communication avec les pools principaux a utilisé TLS/SSL. Les valeurs valides sont On (Activé) et Off (Désactivé).|
|host| Nom d’hôte avec lequel la requête a été envoyée au serveur back-end. Si le nom d’hôte du serveur principal est remplacé, ce nom le reflète.|
|originalHost| Nom d’hôte avec lequel la requête a été reçue par Application Gateway à partir du client.|
```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/PEERINGTEST/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayAccess",
    "time": "2017-04-26T19:27:38Z",
    "category": "ApplicationGatewayAccessLog",
    "properties": {
        "instanceId": "ApplicationGatewayRole_IN_0",
        "clientIP": "191.96.249.97",
        "clientPort": 46886,
        "httpMethod": "GET",
        "requestUri": "/phpmyadmin/scripts/setup.php",
        "requestQuery": "X-AzureApplicationGateway-CACHE-HIT=0&SERVER-ROUTED=10.4.0.4&X-AzureApplicationGateway-LOG-ID=874f1f0f-6807-41c9-b7bc-f3cfa74aa0b1&SERVER-STATUS=404",
        "userAgent": "-",
        "httpStatus": 404,
        "httpVersion": "HTTP/1.0",
        "receivedBytes": 65,
        "sentBytes": 553,
        "timeTaken": 205,
        "sslEnabled": "off",
        "host": "www.contoso.com",
        "originalHost": "www.contoso.com"
    }
}
```
#### <a name="for-application-gateway-and-waf-v2-sku"></a>Pour le niveau tarifaire Application Gateway et WAF v2

|Valeur  |Description  |
|---------|---------|
|instanceId     | Instance Application Gateway ayant traité la requête.        |
|clientIP     | Adresse IP d’origine de la requête.        |
|httpMethod     | Méthode HTTP utilisée par la requête.       |
|requestUri     | URI de la requête reçue.        |
|UserAgent     | Agent utilisateur de l’en-tête de requête HTTP.        |
|httpStatus     | Code d’état HTTP renvoyé au client à partir de d’Application Gateway.       |
|httpVersion     | Version HTTP de la requête.        |
|receivedBytes     | Taille du paquet reçu, en octets.        |
|sentBytes| Taille du paquet envoyé, en octets.|
|timeTaken| Durée (en **secondes**) nécessaire pour le traitement d’une requête et l’envoi de la réponse. Elle est calculée en fonction de l’intervalle entre le moment où Application Gateway reçoit le premier octet d’une requête HTTP et le moment où l’opération d’envoi d’une réponse se termine. Il est important de noter que le champ Time-Taken inclut généralement l’heure à laquelle la requête et les paquets de réponse circulent sur le réseau. |
|sslEnabled| Détermine si la communication avec les pools principaux utilisait TLS. Les valeurs valides sont On (Activé) et Off (Désactivé).|
|sslCipher| Suite de chiffrement utilisée pour la communication TLS (si TLS est activé).|
|sslProtocol| Protocole SSL/TLS utilisé (si TLS est activé).|
|serverRouted| Serveur back-end vers lequel la passerelle d’application route les demandes.|
|serverStatus| Code d’état HTTP du serveur back-end.|
|serverResponseLatency| Latence de la réponse du serveur back-end.|
|host| Adresse figurant dans l’en-tête d’hôte de la demande. En cas de réécriture, ce champ contient le nom d’hôte mis à jour.|
|originalRequestUriWithArgs| Ce champ contient l’URL de la requête HTTP d’origine. |
|requestUri| Ce champ contient l’URL après l’opération de réécriture sur Application Gateway. |
|originalHost| Ce champ contient le nom d’hôte de la requête HTTP d’origine.
```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/PEERINGTEST/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayAccess",
    "time": "2017-04-26T19:27:38Z",
    "category": "ApplicationGatewayAccessLog",
    "properties": {
        "instanceId": "appgw_1",
        "clientIP": "191.96.249.97",
        "httpMethod": "GET",
        "requestUri": "/phpmyadmin/scripts/setup.php",
        "userAgent": "-",
        "httpStatus": 404,
        "httpVersion": "HTTP/1.0",
        "receivedBytes": 65,
        "sentBytes": 553,
        "timeTaken": 205,
        "sslEnabled": "off",
        "sslCipher": "",
        "sslProtocol": "",
        "serverRouted": "104.41.114.59:80",
        "serverStatus": "200",
        "serverResponseLatency": "0.023",
        "host": "www.contoso.com",
    }
}
```

### <a name="performance-log"></a>Journal des performances

Le journal des performances n’est généré que si vous l’avez activé sur chaque instance Application Gateway, comme détaillé dans les étapes précédentes. Les données sont stockées dans le compte de stockage spécifié lors de l’activation de la journalisation. Les données du journal des performances sont générées par intervalles d’1 minute. Il est disponible uniquement pour la référence SKU v1. S’il s’agit de la référence SKU v2, utilisez les [Métriques](application-gateway-metrics.md) pour les données de performances. Les données suivantes sont enregistrées :


|Valeur  |Description  |
|---------|---------|
|instanceId     |  Instance Application Gateway pour laquelle les données des performances sont générées. Pour une passerelle Application Gateway à plusieurs instances, il y a une ligne par instance.        |
|healthyHostCount     | Nombre d’hôtes intègres dans le pool principal.        |
|unHealthyHostCount     | Nombre d’hôtes défaillants dans le pool principal.        |
|requestCount     | Nombre de requêtes traitées.        |
|latency | Latence moyenne (en millisecondes) des requêtes à partir de l’instance vers le serveur principal qui traite les requêtes. |
|failedRequestCount| Nombre d’échecs de requêtes.|
|throughput| Débit moyen depuis le dernier journal, mesuré en octets par seconde.|

```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayPerformance",
    "time": "2016-04-09T00:00:00Z",
    "category": "ApplicationGatewayPerformanceLog",
    "properties":
    {
        "instanceId":"ApplicationGatewayRole_IN_1",
        "healthyHostCount":"4",
        "unHealthyHostCount":"0",
        "requestCount":"185",
        "latency":"0",
        "failedRequestCount":"0",
        "throughput":"119427"
    }
}
```

> [!NOTE]
> La latence est calculée à partir de l’heure de la réception du premier octet de la requête HTTP jusqu’à l’heure de l'envoi du dernier octet de la réponse HTTP. Il s’agit de la somme du temps de traitement d’Application Gateway et du coût du réseau pour le serveur principal, ainsi que le temps pris par le serveur principal pour traiter la requête.

### <a name="firewall-log"></a>Journal du pare-feu

Le journal du pare-feu n’est généré que si vous l’avez activé sur chaque passerelle Application Gateway, comme détaillé dans les étapes précédentes. Ce fichier journal nécessite également que ce pare-feu d’applications web soit configuré sur une passerelle Application Gateway. Les données sont stockées dans le compte de stockage spécifié lors de l’activation de la journalisation. Les données suivantes sont enregistrées :


|Valeur  |Description  |
|---------|---------|
|instanceId     | Instance Application Gateway pour laquelle les données du pare-feu sont générées. Pour une passerelle Application Gateway à plusieurs instances, il y a une ligne par instance.         |
|clientIp     |   Adresse IP d’origine de la requête.      |
|clientPort     |  Port d’origine de la requête.       |
|requestUri     | URL de la requête reçue.       |
|ruleSetType     | Type d’ensemble de règles. La valeur disponible est OWASP.        |
|ruleSetVersion     | Version d’ensemble de règles utilisée. Les valeurs disponibles sont 2.2.9 et 3.0.     |
|ruleId     | ID de règle de l’événement de déclenchement.        |
|message     | Message convivial pour l’événement de déclenchement. La section Détails vous fournit plus d’informations.        |
|action     |  Action effectuée sur la requête. Les valeurs disponibles sont Correspondance et Bloqué.      |
|site     | Site pour lequel le journal a été généré. Actuellement, seul Global est répertorié car les règles sont globales.|
|details     | Détails de l’événement de déclenchement.        |
|details.message     | Description de la règle.        |
|details.data     | Données spécifiques trouvées dans la requête correspondant à la règle.         |
|details.file     | Fichier de configuration qui contenait la règle.        |
|details.line     | Numéro de ligne dans le fichier de configuration ayant déclenché l’événement.       |
|hostname   | Nom d’hôte ou adresse IP de la passerelle Application Gateway.    |
|transactionId  | ID unique d’une transaction donnée qui permet de regrouper plusieurs violations de règle qui se sont produites au cours de la même demande.   |

```json
{
  "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
  "operationName": "ApplicationGatewayFirewall",
  "time": "2017-03-20T15:52:09.1494499Z",
  "category": "ApplicationGatewayFirewallLog",
  "properties": {
    "instanceId": "ApplicationGatewayRole_IN_0",
    "clientIp": "104.210.252.3",
    "clientPort": "4835",
    "requestUri": "/?a=%3Cscript%3Ealert(%22Hello%22);%3C/script%3E",
    "ruleSetType": "OWASP",
    "ruleSetVersion": "3.0",
    "ruleId": "941320",
    "message": "Possible XSS Attack Detected - HTML Tag Handler",
    "action": "Blocked",
    "site": "Global",
    "details": {
      "message": "Warning. Pattern match \"<(a|abbr|acronym|address|applet|area|audioscope|b|base|basefront|bdo|bgsound|big|blackface|blink|blockquote|body|bq|br|button|caption|center|cite|code|col|colgroup|comment|dd|del|dfn|dir|div|dl|dt|em|embed|fieldset|fn|font|form|frame|frameset|h1|head|h ...\" at ARGS:a.",
      "data": "Matched Data: <script> found within ARGS:a: <script>alert(\\x22hello\\x22);</script>",
      "file": "rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf",
      "line": "865"
    }
    "hostname": "40.90.218.100",
    "transactionId": "AYAcUqAcAcAcAcAcASAcAcAc"
  }
}

```

### <a name="view-and-analyze-the-activity-log"></a>Afficher et analyser le journal d’activité

Vous pouvez afficher et analyser les données du journal d’activité en utilisant l’une des méthodes suivantes :

* **Outils Azure** : récupérez les informations du journal d’activité en utilisant Azure PowerShell, Azure CLI, l’API REST Azure ou le portail Azure. Des instructions pas à pas pour chaque méthode sont détaillées dans l’article [Opérations d’activité avec Resource Manager](../azure-resource-manager/management/view-activity-logs.md).
* **Power BI** : si vous n’avez pas encore de compte [Power BI](https://powerbi.microsoft.com/pricing), vous pouvez l’essayer gratuitement. À l’aide des [applications du modèle Power BI](/power-bi/service-template-apps-overview), vous pouvez analyser vos données.

### <a name="view-and-analyze-the-access-performance-and-firewall-logs"></a>Affichage et analyse des journaux d’activité d’accès, des performances et du pare-feu

Les [journaux d’activité Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md) peuvent collecter les fichiers du compteur et du journal d’événements à partir de votre compte de stockage d’objets Blob. Il inclut des visualisations et des fonctionnalités puissantes de recherche pour analyser vos journaux d’activité.

Vous pouvez également vous connecter à votre compte de stockage et récupérer les entrées de journal d’activité JSON pour les journaux d’activité d’accès et des performances. Après avoir téléchargé les fichiers JSON, vous pouvez les convertir en CSV et les afficher dans Excel, PowerBI ou tout autre outil de visualisation de données.

> [!TIP]
> Si vous savez utiliser Visual Studio et les concepts de base de la modification des valeurs de constantes et variables en C#, vous pouvez utiliser les [outils de convertisseur de journaux](https://github.com/Azure-Samples/networking-dotnet-log-converter) disponibles dans GitHub.
>
>

#### <a name="analyzing-access-logs-through-goaccess"></a>Analyse des journaux d’activité d’accès via GoAccess

Nous avons publié un modèle Resource Manager qui installe et exécute le célèbre analyseur de journal d’activité [GoAccess](https://goaccess.io/) pour les journaux d’activité d’accès Application Gateway. GoAccess fournit des statistiques de trafic HTTP précieuses telles que les visiteurs uniques, les fichiers demandés, les hôtes, les systèmes d’exploitation, les navigateurs ou les codes d’état HTTP. Pour plus d’informations, consultez le [fichier Lisez-moi dans le dossier de modèles Resource Manager dans GitHub](https://aka.ms/appgwgoaccessreadme).

## <a name="next-steps"></a>Étapes suivantes

* Visualisez le compteur et les journaux des événements à l’aide des [journaux d’activité Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md).
* Billet de blog sur la [visualisation de votre journal d’activité Azure avec Power BI](https://powerbi.microsoft.com/blog/monitor-azure-audit-logs-with-power-bi/).
* Billet de blog sur [l’affichage et l’analyse des journaux d’activité Azure dans Power BI](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/).

[1]: ./media/application-gateway-diagnostics/figure1.png
[2]: ./media/application-gateway-diagnostics/figure2.png
[3]: ./media/application-gateway-diagnostics/figure3.png
[4]: ./media/application-gateway-diagnostics/figure4.png
[5]: ./media/application-gateway-diagnostics/figure5.png
[6]: ./media/application-gateway-diagnostics/figure6.png
[7]: ./media/application-gateway-diagnostics/figure7.png
[8]: ./media/application-gateway-diagnostics/figure8.png
[9]: ./media/application-gateway-diagnostics/figure9.png
[10]: ./media/application-gateway-diagnostics/figure10.png
