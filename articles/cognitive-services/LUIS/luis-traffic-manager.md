---
title: Augmenter le quota de points de terminaison – LUIS
titleSuffix: Azure Cognitive Services
description: Language Understanding (LUIS) offre la possibilité d’augmenter le quota de demandes des points de terminaison au-delà du quota d’une seule clé. Il suffit de créer plusieurs clés pour LUIS et de les ajouter à l’application LUIS dans la section **Ressources et clés** de la page **Publier**.
manager: nitinme
ms.custom: seodec18, devx-track-js, devx-track-azurepowershell
services: cognitive-services
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 08/20/2019
ms.openlocfilehash: 6fc5bea71909d0e17b4ef0256ab0cad644dacbb3
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95993820"
---
# <a name="use-microsoft-azure-traffic-manager-to-manage-endpoint-quota-across-keys"></a>Utiliser Microsoft Azure Traffic Manager pour gérer le quota de points de terminaison entre les clés
Language Understanding (LUIS) offre la possibilité d’augmenter le quota de demandes des points de terminaison au-delà du quota d’une seule clé. Il suffit de créer plusieurs clés pour LUIS et de les ajouter à l’application LUIS dans la section **Ressources et clés** de la page **Publier**.

L’application cliente doit gérer le trafic entre les clés. LUIS n’effectue pas ce travail.

Cet article explique comment gérer le trafic entre les clés avec Azure [Traffic Manager][traffic-manager-marketing]. Vous devez déjà disposer d’une application LUIS entraînée et publiée. Si ce n’est pas le cas, suivez le [guide de démarrage rapide](luis-get-started-create-app.md) du domaine prédéfini.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="connect-to-powershell-in-the-azure-portal"></a>Se connecter à PowerShell sur le Portail Azure
Sur le Portail [Microsoft Azure][azure-portal], ouvrez la fenêtre PowerShell. L’icône de la fenêtre PowerShell est le signe **>_** dans la barre de navigation supérieure. La version de PowerShell qui se trouve sur le Portail est la dernière version ; par ailleurs, elle vous authentifie automatiquement. Elle requiert un compte [Stockage Azure](https://azure.microsoft.com/services/storage/).

![Capture d’écran du Portail Azure avec la fenêtre PowerShell ouverte](./media/traffic-manager/azure-portal-powershell.png)

Les sections suivantes utilisent les [cmdlets PowerShell Traffic Manager](/powershell/module/az.trafficmanager/#traffic_manager).

## <a name="create-azure-resource-group-with-powershell"></a>Créer un groupe de ressources Azure avec PowerShell
Avant de créer les ressources Azure, créez le groupe de ressources qui les contiendra. Nommez-le `luis-traffic-manager` et utilisez la région `West US`. La région du groupe de ressources stocke des métadonnées sur le groupe. Si vos ressources se trouvent dans une autre région, cela ne les ralentira pas.

Créez un groupe de ressources avec l'applet de commande **[New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)** :

```powerShell
New-AzResourceGroup -Name luis-traffic-manager -Location "West US"
```

## <a name="create-luis-keys-to-increase-total-endpoint-quota"></a>Créer des clés LUIS pour augmenter le quota total de points de terminaison
1. Sur le Portail Azure, créez deux clés **Language Understanding**, l’une dans `West US` et l’autre dans `East US`. Utilisez le groupe de ressources créé dans la section précédente, nommé `luis-traffic-manager`.

    ![Capture d’écran du Portail Azure avec deux clés LUIS dans le groupe de ressources luis-traffic-manager](./media/traffic-manager/luis-keys.png)

2. Sur le site web [LUIS][LUIS], dans la section **Manage**, dans la page **Ressources Azure**, attribuez des clés à l’application, puis republiez l’application en sélectionnant le bouton **Publier** dans le menu en haut à droite.

    L’exemple d’URL dans la colonne **Point de terminaison** utilise une demande GET avec la clé de point de terminaison comme paramètre de requête. Copiez l’URL de point de terminaison des deux nouvelles clés. Elles seront utilisées pour la configuration de Traffic Manager dans la suite de cet article.

## <a name="manage-luis-endpoint-requests-across-keys-with-traffic-manager"></a>Gérer les demandes du point de terminaison LUIS entre les clés avec Traffic Manager
Traffic Manager crée un nouveau point d’accès DNS pour les points de terminaison. Il ne fonctionne pas comme une passerelle ou un proxy, mais uniquement au niveau du DNS. Cet exemple ne modifie pas les enregistrements DNS. Il utilise une bibliothèque DNS pour communiquer avec Traffic Manager afin de récupérer le bon point de terminaison pour cette demande en particulier. _Chaque_ demande destinée à LUIS implique au préalable qu’une demande Traffic Manager détermine quel point de terminaison LUIS utiliser.

### <a name="polling-uses-luis-endpoint"></a>L’interrogation utilise le point de terminaison LUIS
Traffic Manager interroge périodiquement les points de terminaison pour vérifier qu’ils sont toujours disponibles. L’URL de Traffic Manager interrogée doit être accessible avec une demande GET et retourner une réponse 200. L’URL du point de terminaison sur la page **Publier** effectue cette opération. Une clé de point de terminaison a son propre itinéraire et ses propres paramètres de chaîne de requête ; elle a donc besoin d’un chemin d’interrogation qui lui soit propre. Chaque interrogation de Traffic Manager coûte une demande de quota. Le paramètre de chaîne de requête **q** du point de terminaison LUIS est l’énoncé envoyé à LUIS. Ce paramètre, au lieu d’envoyer un énoncé, permet d’ajouter l’interrogation de Traffic Manager au journal du point de terminaison LUIS comme technique de débogage tout en configurant Traffic Manager.

Chaque point de terminaison LUIS a besoin de son propre chemin d’accès et donc de son propre profil Traffic Manager. Pour gérer les différents profils, créez une architecture [Traffic Manager _imbriquée_](../../traffic-manager/traffic-manager-nested-profiles.md). Un seul profil parent pointe vers les profils enfants et gère le trafic entre eux.

Une fois Traffic Manager configuré, n’oubliez pas de modifier le chemin d’accès pour utiliser le paramètre de chaîne de requête logging=false afin d’éviter de remplir votre journal d’interrogations.

## <a name="configure-traffic-manager-with-nested-profiles"></a>Configurer Traffic Manager avec des profils imbriqués
Les sections suivantes créent deux profils enfants, l’un pour la clé LUIS Est et l’autre pour la clé LUIS Ouest. Ensuite, un profil parent est créé et les profils enfants y sont ajoutés.

### <a name="create-the-east-us-traffic-manager-profile-with-powershell"></a>Créer le profil Traffic Manager USA Est avec PowerShell
Pour créer le profil Traffic Manager USA Est, il y a plusieurs étapes à suivre : créer le profil, ajouter le point de terminaison et définir le point de terminaison. Un profil Traffic Manager peut comporter plusieurs points de terminaison, mais tous ont le même chemin de validation. Les URL des points de terminaison LUIS sont différentes pour les abonnements Est et Ouest en raison de la région et de la clé de point de terminaison ; chaque point de terminaison LUIS doit donc être unique dans le profil.

1. Créer le profil avec l'applet de commande **[New-AzTrafficManagerProfile](/powershell/module/az.trafficmanager/new-aztrafficmanagerprofile)**

    Utilisez la cmdlet suivante pour créer le profil. Veillez à modifier `appIdLuis` et `subscriptionKeyLuis`. La clé d’abonnement correspond à la clé LUIS USA Est. Si le chemin d’accès n’est pas correct, y compris la clé de point de terminaison et l’ID de l’application LUIS, l’interrogation de Traffic Manager a le statut `degraded`, car Traffic Manager ne parvient pas à demander le point de terminaison LUIS. Vérifiez que la valeur de `q` est `traffic-manager-east` afin de pouvoir la voir dans les journaux d’activité du point de terminaison LUIS.

    ```powerShell
    $eastprofile = New-AzTrafficManagerProfile -Name luis-profile-eastus -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-eastus -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/luis/v2.0/apps/<appID>?subscription-key=<subscriptionKey>&q=traffic-manager-east"
    ```

    Ce tableau décrit les différentes variables de la cmdlet :

    |Paramètre de configuration|Valeur ou nom de variable|Objectif|
    |--|--|--|
    |-Name|luis-profile-eastus|Nom de Traffic Manager sur le Portail Azure.|
    |-ResourceGroupName|luis-traffic-manager|Créé dans la section précédente.|
    |-TrafficRoutingMethod|Performances|Pour plus d’informations, voir [Méthodes de routage de Traffic Manager][routing-methods]. En cas de performances, la demande d’URL à Traffic Manager doit provenir de la région de l’utilisateur. En cas d’utilisation d’un chatbot ou d’une autre application, le chatbot a la charge de reproduire la région dans l’appel à Traffic Manager. |
    |-RelativeDnsName|luis-dns-eastus|Sous-domaine du service : luis-dns-eastus.trafficmanager.net.|
    |-Ttl|30|Intervalle d'interrogation, 30 secondes.|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|Le port et le protocole de LUIS sont HTTPS/443.|
    |-MonitorPath|`/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-east`|Remplacez `<appIdLuis>` et `<subscriptionKeyLuis>` par vos propres valeurs.|

    En cas de succès, la demande ne reçoit aucune réponse.

2. Ajouter le point de terminaison USA Est avec l'applet de commande **[Add-AzTrafficManagerEndpointConfig](/powershell/module/az.trafficmanager/add-aztrafficmanagerendpointconfig)**

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName luis-east-endpoint -TrafficManagerProfile $eastprofile -Type ExternalEndpoints -Target eastus.api.cognitive.microsoft.com -EndpointLocation "eastus" -EndpointStatus Enabled
    ```
    Ce tableau décrit les différentes variables de la cmdlet :

    |Paramètre de configuration|Valeur ou nom de variable|Objectif|
    |--|--|--|
    |-EndpointName|luis-east-endpoint|Nom du point de terminaison qui s’affiche sous le profil.|
    |-TrafficManagerProfile|$eastprofile|Utilisez l’objet de profil créé à l’étape 1.|
    |-Type|ExternalEndpoints|Pour en savoir plus, voir [Points de terminaison Traffic Manager][traffic-manager-endpoints]. |
    |-Target|eastus.api.cognitive.microsoft.com|Domaine du point de terminaison LUIS.|
    |-EndpointLocation|"eastus"|Région du point de terminaison.|
    |-EndpointStatus|activé|Active le point de terminaison lors de sa création.|

    En cas de succès, la réponse se présente ainsi :

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-eastus
    Name                             : luis-profile-eastus
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-eastus
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /luis/v2.0/apps/<luis-app-id>?subscription-key=f0517d185bcf467cba5147d6260bb868&q=traffic-manager-east
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {luis-east-endpoint}
    ```

3. Définir le point de terminaison USA Est avec l'applet de commande **[Set-AzTrafficManagerProfile](/powershell/module/az.trafficmanager/set-aztrafficmanagerprofile)**

    ```powerShell
    Set-AzTrafficManagerProfile -TrafficManagerProfile $eastprofile
    ```

    En cas de succès, la réponse sera la même qu’à l’étape 2.

### <a name="create-the-west-us-traffic-manager-profile-with-powershell"></a>Créer le profil Traffic Manager USA Ouest avec PowerShell
Pour créer le profil Traffic Manager USA Ouest, suivez les mêmes étapes : créer le profil, ajouter le point de terminaison et définir le point de terminaison.

1. Créer le profil avec l'applet de commande **[New-AzTrafficManagerProfile](/powershell/module/az.TrafficManager/New-azTrafficManagerProfile)**

    Utilisez la cmdlet suivante pour créer le profil. Veillez à modifier `appIdLuis` et `subscriptionKeyLuis`. La clé d’abonnement correspond à la clé LUIS USA Est. Si le chemin d’accès n’est pas correct, y compris la clé de point de terminaison et l’ID de l’application LUIS, l’interrogation de Traffic Manager a le statut `degraded`, car Traffic Manager ne parvient pas à demander le point de terminaison LUIS. Vérifiez que la valeur de `q` est `traffic-manager-west` afin de pouvoir la voir dans les journaux d’activité du point de terminaison LUIS.

    ```powerShell
    $westprofile = New-AzTrafficManagerProfile -Name luis-profile-westus -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-westus -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-west"
    ```

    Ce tableau décrit les différentes variables de la cmdlet :

    |Paramètre de configuration|Valeur ou nom de variable|Objectif|
    |--|--|--|
    |-Name|luis-profile-westus|Nom de Traffic Manager sur le Portail Azure.|
    |-ResourceGroupName|luis-traffic-manager|Créé dans la section précédente.|
    |-TrafficRoutingMethod|Performances|Pour plus d’informations, voir [Méthodes de routage de Traffic Manager][routing-methods]. En cas de performances, la demande d’URL à Traffic Manager doit provenir de la région de l’utilisateur. En cas d’utilisation d’un chatbot ou d’une autre application, le chatbot a la charge de reproduire la région dans l’appel à Traffic Manager. |
    |-RelativeDnsName|luis-dns-westus|Sous-domaine du service : luis-dns-westus.trafficmanager.net.|
    |-Ttl|30|Intervalle d'interrogation, 30 secondes.|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|Le port et le protocole de LUIS sont HTTPS/443.|
    |-MonitorPath|`/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-west`|Remplacez `<appId>` et `<subscriptionKey>` par vos propres valeurs. N’oubliez pas que cette clé de point de terminaison est différente de celle de l’Est|

    En cas de succès, la demande ne reçoit aucune réponse.

2. Ajouter le point de terminaison USA Ouest avec l'applet de commande **[Add-AzTrafficManagerEndpointConfig](/powershell/module/az.TrafficManager/Add-azTrafficManagerEndpointConfig)**

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName luis-west-endpoint -TrafficManagerProfile $westprofile -Type ExternalEndpoints -Target westus.api.cognitive.microsoft.com -EndpointLocation "westus" -EndpointStatus Enabled
    ```

    Ce tableau décrit les différentes variables de la cmdlet :

    |Paramètre de configuration|Valeur ou nom de variable|Objectif|
    |--|--|--|
    |-EndpointName|luis-west-endpoint|Nom du point de terminaison qui s’affiche sous le profil.|
    |-TrafficManagerProfile|$westprofile|Utilisez l’objet de profil créé à l’étape 1.|
    |-Type|ExternalEndpoints|Pour en savoir plus, voir [Points de terminaison Traffic Manager][traffic-manager-endpoints]. |
    |-Target|westus.api.cognitive.microsoft.com|Domaine du point de terminaison LUIS.|
    |-EndpointLocation|"westus"|Région du point de terminaison.|
    |-EndpointStatus|activé|Active le point de terminaison lors de sa création.|

    En cas de succès, la réponse se présente ainsi :

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-westus
    Name                             : luis-profile-westus
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-westus
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /luis/v2.0/apps/c3fc5d1e-5187-40cc-af0f-fbde328aa16b?subscription-key=e3605f07e3cc4bedb7e02698a54c19cc&q=traffic-manager-west
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {luis-west-endpoint}
    ```

3. Définir le point de terminaison USA Ouest avec l'applet de commande **[Set-AzTrafficManagerProfile](/powershell/module/az.TrafficManager/Set-azTrafficManagerProfile)**

    ```powerShell
    Set-AzTrafficManagerProfile -TrafficManagerProfile $westprofile
    ```

    En cas de succès, la réponse sera la même qu’à l’étape 2.

### <a name="create-parent-traffic-manager-profile"></a>Créer un profil Traffic Manager parent
Créez le profil Traffic Manager parent et liez les deux profils Traffic Manager enfants au parent.

1. Créer le profil parent avec l'applet de commande **[New-AzTrafficManagerProfile](/powershell/module/az.TrafficManager/New-azTrafficManagerProfile)**

    ```powerShell
    $parentprofile = New-AzTrafficManagerProfile -Name luis-profile-parent -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-parent -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/"
    ```

    Ce tableau décrit les différentes variables de la cmdlet :

    |Paramètre de configuration|Valeur ou nom de variable|Objectif|
    |--|--|--|
    |-Name|luis-profile-parent|Nom de Traffic Manager sur le Portail Azure.|
    |-ResourceGroupName|luis-traffic-manager|Créé dans la section précédente.|
    |-TrafficRoutingMethod|Performances|Pour plus d’informations, voir [Méthodes de routage de Traffic Manager][routing-methods]. En cas de performances, la demande d’URL à Traffic Manager doit provenir de la région de l’utilisateur. En cas d’utilisation d’un chatbot ou d’une autre application, le chatbot a la charge de reproduire la région dans l’appel à Traffic Manager. |
    |-RelativeDnsName|luis-dns-parent|Sous-domaine du service : luis-dns-parent.trafficmanager.net.|
    |-Ttl|30|Intervalle d'interrogation, 30 secondes.|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|Le port et le protocole de LUIS sont HTTPS/443.|
    |-MonitorPath|`/`|Ce chemin d’accès n’a pas d’importance, car ce sont les chemins d’accès du point de terminaison enfant qui sont utilisés à la place.|

    En cas de succès, la demande ne reçoit aucune réponse.

2. Ajouter le profil enfant USA Est au parent avec **[Add-AzTrafficManagerEndpointConfig](/powershell/module/az.TrafficManager/Add-azTrafficManagerEndpointConfig)** et le type **NestedEndpoints**

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName child-endpoint-useast -TrafficManagerProfile $parentprofile -Type NestedEndpoints -TargetResourceId $eastprofile.Id -EndpointStatus Enabled -EndpointLocation "eastus" -MinChildEndpoints 1
    ```

    Ce tableau décrit les différentes variables de la cmdlet :

    |Paramètre de configuration|Valeur ou nom de variable|Objectif|
    |--|--|--|
    |-EndpointName|child-endpoint-useast|Profil Est.|
    |-TrafficManagerProfile|$parentprofile|Profil à attribuer à ce point de terminaison.|
    |-Type|NestedEndpoints|Pour plus d’informations, consultez [Add-AzTrafficManagerEndpointConfig](/powershell/module/az.trafficmanager/Add-azTrafficManagerEndpointConfig). |
    |-TargetResourceId|$eastprofile.Id|ID du profil enfant.|
    |-EndpointStatus|activé|État du point de terminaison après ajout au parent.|
    |-EndpointLocation|"eastus"|[Nom de la région Azure](https://azure.microsoft.com/global-infrastructure/regions/) de la ressource.|
    |-MinChildEndpoints|1|Nombre minimal de points de terminaison enfants.|

    En cas de succès, la réponse, qui comporte le nouveau point de terminaison `child-endpoint-useast`, se présente ainsi :

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-parent
    Name                             : luis-profile-parent
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-parent
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {child-endpoint-useast}
    ```

3. Ajouter le profil enfant USA Ouest au parent avec l'applet de commande **[Add-AzTrafficManagerEndpointConfig](/powershell/module/az.TrafficManager/Add-azTrafficManagerEndpointConfig)** et le type **NestedEndpoints**

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName child-endpoint-uswest -TrafficManagerProfile $parentprofile -Type NestedEndpoints -TargetResourceId $westprofile.Id -EndpointStatus Enabled -EndpointLocation "westus" -MinChildEndpoints 1
    ```

    Ce tableau décrit les différentes variables de la cmdlet :

    |Paramètre de configuration|Valeur ou nom de variable|Objectif|
    |--|--|--|
    |-EndpointName|child-endpoint-uswest|Profil Ouest.|
    |-TrafficManagerProfile|$parentprofile|Profil à attribuer à ce point de terminaison.|
    |-Type|NestedEndpoints|Pour plus d’informations, consultez [Add-AzTrafficManagerEndpointConfig](/powershell/module/az.trafficmanager/Add-azTrafficManagerEndpointConfig). |
    |-TargetResourceId|$westprofile.Id|ID du profil enfant.|
    |-EndpointStatus|activé|État du point de terminaison après ajout au parent.|
    |-EndpointLocation|"westus"|[Nom de la région Azure](https://azure.microsoft.com/global-infrastructure/regions/) de la ressource.|
    |-MinChildEndpoints|1|Nombre minimal de points de terminaison enfants.|

    En cas de succès, la réponse, qui comporte à la fois le précédent point de terminaison `child-endpoint-useast` et le nouveau point de terminaison `child-endpoint-uswest`, se présente ainsi :

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-parent
    Name                             : luis-profile-parent
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-parent
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {child-endpoint-useast, child-endpoint-uswest}
    ```

4. Définir des points de terminaison avec l'applet de commande **[Set-AzTrafficManagerProfile](/powershell/module/az.TrafficManager/Set-azTrafficManagerProfile)**

    ```powerShell
    Set-AzTrafficManagerProfile -TrafficManagerProfile $parentprofile
    ```

    En cas de succès, la réponse sera la même qu’à l’étape 3.

### <a name="powershell-variables"></a>Variables PowerShell
Dans les sections précédentes, trois variables PowerShell ont été créées : `$eastprofile`, `$westprofile` et `$parentprofile`. Elles sont utilisées vers la fin de la configuration de Traffic Manager. Si vous choisissez de ne pas créer les variables, si vous avez oublié de le faire ou si la fenêtre PowerShell expire, vous pouvez utiliser la cmdlet PowerShell **[Get-AzTrafficManagerProfile](/powershell/module/az.TrafficManager/Get-azTrafficManagerProfile)** pour récupérer le profil et l’affecter à une variable.

Remplacez les éléments figurant entre crochets, `<>`, par les valeurs correctes de chacun des trois profils dont vous avez besoin.

```powerShell
$<variable-name> = Get-AzTrafficManagerProfile -Name <profile-name> -ResourceGroupName luis-traffic-manager
```

## <a name="verify-traffic-manager-works"></a>Vérifier le fonctionnement de Traffic Manager
Les profils Traffic Manager doivent avoir l’état `Online`, qui dépend du chemin d’interrogation du point de terminaison, pour fonctionner.

### <a name="view-new-profiles-in-the-azure-portal"></a>Afficher les nouveaux profils sur le Portail Azure
Vous pouvez vérifier que les trois profils sont créés en examinant les ressources du groupe de ressources `luis-traffic-manager`.

![Capture d’écran du groupe de ressources Azure luis-traffic-manager](./media/traffic-manager/traffic-manager-profiles.png)

### <a name="verify-the-profile-status-is-online"></a>Vérifier que l’état du profil est En ligne
Traffic Manager interroge le chemin de chaque point de terminaison pour vérifier qu’il est en ligne. Si c’est le cas, l’état des profils enfants est `Online`, ce que l’on peut voir sur la **Vue d’ensemble** de chaque profil.

![Capture d’écran de la Vue d’ensemble du profil Azure Traffic Manager indiquant un état de monitoring En ligne](./media/traffic-manager/profile-status-online.png)

### <a name="validate-traffic-manager-polling-works"></a>Valider le fonctionnement de l’interrogation de Traffic Manager
Il existe un autre moyen de valider le fonctionnement de l’interrogation de Traffic Manager : les journaux d’activité du point de terminaison LUIS. Sur la page de la liste des applications du site web [LUIS][LUIS], exportez le journal du point de terminaison de l’application. Traffic Manager interroge souvent les deux points de terminaison, ce qui fait qu’il y a des entrées dans les journaux d’activité même s’ils n’ont été en ligne que quelques minutes. N’oubliez pas de rechercher les entrées pour lesquelles la requête commence par `traffic-manager-`.

```console
traffic-manager-west    6/7/2018 19:19  {"query":"traffic-manager-west","intents":[{"intent":"None","score":0.944767}],"entities":[]}
traffic-manager-east    6/7/2018 19:20  {"query":"traffic-manager-east","intents":[{"intent":"None","score":0.944767}],"entities":[]}
```

### <a name="validate-dns-response-from-traffic-manager-works"></a>Valider le fonctionnement de la réponse DNS de Traffic Manager
Pour vérifier que la réponse DNS retourne un point de terminaison LUIS, demandez le DNS du profil parent Traffic Manager à l’aide d’une bibliothèque de client DNS. Le nom DNS du profil parent est `luis-dns-parent.trafficmanager.net`.

Le Node.js suivant effectue une demande auprès du profil parent et retourne un point de terminaison LUIS :

```javascript
const dns = require('dns');

dns.resolveAny('luis-dns-parent.trafficmanager.net', (err, ret) => {
  console.log('ret', ret);
});
```

En cas de succès, la réponse avec le point de terminaison LUIS est la suivante :

```json
[
    {
        value: 'westus.api.cognitive.microsoft.com',
        type: 'CNAME'
    }
]
```

## <a name="use-the-traffic-manager-parent-profile"></a>Utiliser le profil parent Traffic Manager
Pour gérer le trafic entre les points de terminaison, vous devez insérer un appel au DNS Traffic Manager pour rechercher le point de terminaison LUIS. Cet appel, effectué à chaque demande du point de terminaison LUIS, doit simuler l’emplacement géographique de l’utilisateur de l’application cliente LUIS. Ajoutez le code de réponse DNS entre votre application cliente LUIS et la demande envoyée à LUIS pour la prédiction du point de terminaison.

## <a name="resolving-a-degraded-state"></a>Résolution d’un état détérioré

Activez [journaux de diagnostic](../../traffic-manager/traffic-manager-diagnostic-logs.md) pour Traffic Manager afin de connaître la cause de la détérioration de l’état du point de terminaison.

## <a name="clean-up"></a>Nettoyer
Supprimez les deux clés de point de terminaison LUIS, les trois profils Traffic Manager et le groupe de ressources contenant ces cinq ressources. Utilisez pour cela le Portail Azure. Supprimez les cinq ressources de la liste de ressources, puis supprimez le groupe de ressources.

## <a name="next-steps"></a>Étapes suivantes

Examinez les options [intergiciel (middleware)](/azure/bot-service/bot-builder-create-middleware?tabs=csaddmiddleware%252ccsetagoverwrite%252ccsmiddlewareshortcircuit%252ccsfallback%252ccsactivityhandler&view=azure-bot-service-4.0) de BotFramework v4 pour comprendre comment ajouter ce code de gestion du trafic à un bot BotFramework.

[traffic-manager-marketing]: https://azure.microsoft.com/services/traffic-manager/
[traffic-manager-docs]: ../../traffic-manager/index.yml
[LUIS]: ./luis-reference-regions.md#luis-website
[azure-portal]: https://portal.azure.com/
[azure-storage]: https://azure.microsoft.com/services/storage/
[routing-methods]: ../../traffic-manager/traffic-manager-routing-methods.md
[traffic-manager-endpoints]: ../../traffic-manager/traffic-manager-endpoint-types.md