---
title: Utilisation de PowerShell pour gérer Traffic Manager dans Azure
description: Avec ce parcours d’apprentissage, commencez à utiliser Azure PowerShell pour Traffic Manager.
services: traffic-manager
documentationcenter: na
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: duau
ms.openlocfilehash: 21076fe23301c189d9987f78706cc81691ce7a4f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89400532"
---
# <a name="using-powershell-to-manage-traffic-manager"></a>Utilisation de PowerShell pour gérer Traffic Manager

Azure Resource Manager est l’interface de gestion par défaut des services Azure. Les outils et API d’Azure Resource Manager permettent de gérer les profils Azure Traffic Manager.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="resource-model"></a>Modèle de ressource

Azure Traffic Manager est configuré à l'aide d'un ensemble de paramètres appelé profil Traffic Manager. Ce profil contient les paramètres DNS, les paramètres de routage du trafic, les paramètres de surveillance des points de terminaison et la liste des points de terminaison de service vers lesquels le trafic est acheminé.

Chaque profil Traffic Manager est représenté par une ressource de type « TrafficManagerProfiles ». Au niveau de l'API REST, l'URI de chaque profil est la suivante :

`https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/trafficManagerProfiles/{profile-name}?api-version={api-version}`

## <a name="setting-up-azure-powershell"></a>Configuration d’Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Ces instructions utilisent Microsoft Azure PowerShell. L’article suivant explique comment installer et configurer Azure PowerShell.

* [Installation et configuration d’Azure PowerShell](/powershell/azure/)

Les exemples de cet article supposent que vous disposez déjà d’un groupe de ressources. Vous pouvez en créer un à l’aide de la commande suivante :

```powershell
New-AzResourceGroup -Name MyRG -Location "West US"
```

> [!NOTE]
> Azure Resource Manager exige des groupes de ressources qu’ils disposent tous d’un emplacement. Celui-ci est utilisé comme emplacement par défaut pour les ressources créées dans ce groupe de ressources. Cependant, étant donné que les ressources de profil Traffic Manager sont mondiales et non régionales, le choix de l’emplacement du groupe de ressources n’a aucune incidence sur Azure Traffic Manager.

## <a name="create-a-traffic-manager-profile"></a>Créer un profil Traffic Manager

Pour créer un profil Traffic Manager, utilisez l’applet de commande `New-AzTrafficManagerProfile` :

```powershell
$TmProfile = New-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName contoso -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
```

Le tableau suivant décrit les paramètres :

| Paramètre | Description |
| --- | --- |
| Nom |Nom de la ressource de profil Traffic Manager. Les profils d’un même groupe de ressources doivent avoir un nom unique. Ce nom est différent du nom DNS utilisé pour les requêtes DNS. |
| ResourceGroupName |Nom du groupe de ressources contenant la ressource de profil. |
| TrafficRoutingMethod |Spécifie la méthode de routage du trafic utilisée pour identifier le point de terminaison qui est retourné en réponse à une requête DNS. Les valeurs possibles sont « Performance », « Weighted » ou « Priority ». |
| RelativeDnsName |Spécifie la partie nom d’hôte du nom DNS fourni par ce profil Traffic Manager. Cette valeur est combinée au nom de domaine DNS utilisé par Azure Traffic Manager pour former le nom de domaine complet (FQDN) du profil. Par exemple, la valeur définie « contoso » devient « contoso.trafficmanager.net ». |
| TTL |Spécifie la durée de vie (TTL) du DNS en secondes. Ce paramètre indique aux programmes de résolution DNS locaux et aux clients DNS la durée de mise en cache des réponses DNS pour ce profil Traffic Manager. |
| MonitorProtocol |Spécifie le protocole à utiliser pour surveiller l’intégrité des points de terminaison. Les valeurs possibles sont « HTTP » et « HTTPS ». |
| MonitorPort |Spécifie le port TCP utilisé pour surveiller l’intégrité des points de terminaison. |
| MonitorPort |Spécifie le chemin relatif du nom de domaine de point de terminaison utilisé pour évaluer l’intégrité des points de terminaison. |

L’applet de commande crée un profil Traffic Manager dans Azure et retourne un objet de profil correspondant à PowerShell. À ce stade, le profil ne contient aucun point de terminaison. Pour plus d’informations sur l’ajout de points de terminaison à un profil Traffic Manager, consultez Ajout de points de terminaison Traffic Manager.

## <a name="get-a-traffic-manager-profile"></a>Récupérer un profil Traffic Manager

Pour récupérer un objet de profil Traffic Manager existant, utilisez l’applet de commande `Get-AzTrafficManagerProfle` :

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
```

Cette applet de commande renvoie un objet de profil Traffic Manager.

## <a name="update-a-traffic-manager-profile"></a>Mettre à jour un profil Traffic Manager

La modification des profils Traffic Manager est un processus en 3 étapes :

1. Récupérez le profil à l’aide de `Get-AzTrafficManagerProfile` ou utilisez le profil retourné par `New-AzTrafficManagerProfile`.
2. Modifiez le profil. Vous pouvez ajouter et supprimer des points de terminaison ou modifier les paramètres des points de terminaison ou du profil. Ces modifications sont des opérations hors connexion. Vous modifiez seulement l’objet local en mémoire qui représente le profil.
3. Validez vos modifications à l’aide de l’applet de commande `Set-AzTrafficManagerProfile`.

Toutes les propriétés de profil peuvent être modifiées à l’exception de RelativeDnsName. Pour modifier la propriété RelativeDnsName, vous devez supprimer le profil et en créer un sous un nouveau nom.

L’exemple suivant montre comment modifier la durée de vie (TTL) du profil :

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
$TmProfile.Ttl = 300
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```

Il existe trois types de points de terminaison Traffic Manager :

1. Les **points de terminaison Azure** sont des services hébergés dans Azure.
2. Les **points de terminaison externes** sont des services hébergés en dehors d’Azure.
3. Les **points de terminaison imbriqués** servent à construire des hiérarchies imbriquées de profils Traffic Manager. Ils permettent de définir des configurations de routage du trafic avancées pour les applications complexes.

Dans ces trois cas, les points de terminaison peuvent être ajoutés de deux manières :

1. En suivant le processus en 3 étapes décrit précédemment. L’avantage de cette méthode est que plusieurs modifications de point de terminaison peuvent être apportées en une seule mise à jour.
2. En utilisant la cmdlet New-AzTrafficManagerEndpoint. Celle-ci permet d’ajouter un point de terminaison à un profil Traffic Manager existant en une seule opération.

## <a name="adding-azure-endpoints"></a>Ajout de points de terminaison Azure

Les points de terminaison Azure font référence à des services hébergés dans Azure. Les types de point de terminaison Azure pris en charge sont au nombre de deux :

1. Azure App Service
2. Ressources PublicIpAddress Azure (peuvent être attachées à un équilibreur de charge ou à une carte réseau de machine virtuelle). PublicIpAddress doit se voir affecter un nom DNS pour être utilisé dans Traffic Manager.

Dans chaque cas :

* Le service est spécifié à l’aide du paramètre « targetResourceId » de `Add-AzTrafficManagerEndpointConfig` ou `New-AzTrafficManagerEndpoint`.
* « Target » et « EndpointLocation » sont définis implicitement selon TargetResourceId.
* La spécification du paramètre « Pondération » est facultative. Les pondérations ne sont utilisées que si le profil est configuré pour utiliser la méthode de routage du trafic « Weighted ». Autrement, elles sont ignorées. S’il est spécifié, le paramètre doit avoir une valeur numérique comprise entre 1 et 1 000. La valeur par défaut est « 1 ».
* La spécification du paramètre « Priorité » est facultative. Les priorités ne sont utilisées que si le profil est configuré pour utiliser la méthode d’acheminement de trafic « Priority ». Autrement, elles sont ignorées. Les valeurs valides vont de 1 à 1 000, les valeurs basses indiquant une priorité élevée. Si elles sont spécifiées pour un point de terminaison, elles doivent l’être pour tous les points de terminaison. En cas d’omission, les valeurs par défaut à partir de « 1 » sont appliquées dans l’ordre où les points de terminaison sont répertoriés.

### <a name="example-1-adding-app-service-endpoints-using-add-aztrafficmanagerendpointconfig"></a>Exemple 1 : Ajout de points de terminaison App Service à l’aide de `Add-AzTrafficManagerEndpointConfig`

Dans cet exemple, nous créons un profil Traffic Manager et ajoutons deux points de terminaison App Service à l’aide de l’applet de commande `Add-AzTrafficManagerEndpointConfig`.

```powershell
$TmProfile = New-AzTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
$webapp1 = Get-AzWebApp -Name webapp1
Add-AzTrafficManagerEndpointConfig -EndpointName webapp1ep -TrafficManagerProfile $TmProfile -Type AzureEndpoints -TargetResourceId $webapp1.Id -EndpointStatus Enabled
$webapp2 = Get-AzWebApp -Name webapp2
Add-AzTrafficManagerEndpointConfig -EndpointName webapp2ep -TrafficManagerProfile $TmProfile -Type AzureEndpoints -TargetResourceId $webapp2.Id -EndpointStatus Enabled
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```
### <a name="example-2-adding-a-publicipaddress-endpoint-using-new-aztrafficmanagerendpoint"></a>Exemple 2 : Ajout d’un point de terminaison publicIpAddress à l’aide de `New-AzTrafficManagerEndpoint`

Dans cet exemple, une ressource d’adresse IP publique est ajoutée au profil Traffic Manager. L’adresse IP publique doit avoir un nom DNS configuré et peut être liée à la carte réseau d’une machine virtuelle ou à un équilibreur de charge.

```powershell
$ip = Get-AzPublicIpAddress -Name MyPublicIP -ResourceGroupName MyRG
New-AzTrafficManagerEndpoint -Name MyIpEndpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled
```

## <a name="adding-external-endpoints"></a>Ajout de points de terminaison externes

Traffic Manager utilise les points de terminaison externes pour diriger le trafic vers les services hébergés en dehors d’Azure. Comme les points de terminaison Azure, les points de terminaison externes peuvent être ajoutés à l’aide de `Add-AzTrafficManagerEndpointConfig` suivi de `Set-AzTrafficManagerProfile`, ou avec `New-AzTrafficManagerEndpoint`.

Lors de la spécification de points de terminaison externes :

* Le nom de domaine du point de terminaison doit être spécifié en utilisant le paramètre « Target »
* Si la méthode de routage du trafic « Performance » est utilisée, le paramètre « EndpointLocation » doit être renseigné. Sinon, il est facultatif. La valeur doit être un [nom de région Azure valide](https://azure.microsoft.com/regions/).
* Les paramètres « Weight » et « Priority » sont facultatifs.

### <a name="example-1-adding-external-endpoints-using-add-aztrafficmanagerendpointconfig-and-set-aztrafficmanagerprofile"></a>Exemple 1 : Ajout de points de terminaison externes à l’aide de `Add-AzTrafficManagerEndpointConfig` et `Set-AzTrafficManagerProfile`

Dans cet exemple, nous créons un profil Traffic Manager, ajoutons deux points de terminaison externes et validons les modifications.

```powershell
$TmProfile = New-AzTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
Add-AzTrafficManagerEndpointConfig -EndpointName eu-endpoint -TrafficManagerProfile $TmProfile -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointLocation "North Europe" -EndpointStatus Enabled
Add-AzTrafficManagerEndpointConfig -EndpointName us-endpoint -TrafficManagerProfile $TmProfile -Type ExternalEndpoints -Target app-us.contoso.com -EndpointLocation "Central US" -EndpointStatus Enabled
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```

### <a name="example-2-adding-external-endpoints-using-new-aztrafficmanagerendpoint"></a>Exemple 2 : Ajout de points de terminaison externes à l’aide de `New-AzTrafficManagerEndpoint`

Dans cet exemple, nous ajoutons un point de terminaison externe à un profil existant. Le profil est spécifié en utilisant les noms de profil et de groupe de ressources.

```powershell
New-AzTrafficManagerEndpoint -Name eu-endpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointStatus Enabled
```

## <a name="adding-nested-endpoints"></a>Ajout de points de terminaison « imbriqués »

Chaque profil Traffic Manager spécifie une seule méthode de routage du trafic. Cependant, certains scénarios exigent une méthode de routage du trafic plus sophistiquée que celle proposée par un profil Traffic Manager unique. Vous pouvez imbriquer des profils Traffic Manager pour combiner les avantages de plusieurs méthodes de routage du trafic. Les profils imbriqués permettent de modifier le comportement par défaut de Traffic Manager pour prendre en charge des déploiements d’applications plus importants et plus complexes. Pour obtenir des informations plus détaillées, consultez [Profils Traffic Manager imbriqués](traffic-manager-nested-profiles.md).

Les points de terminaison imbriqués sont configurés sur le profil parent, à l’aide d’un type de point de terminaison spécifique, « NestedEndpoints ». Lors de la spécification de points de terminaison imbriqués :

* Le point de terminaison doit être spécifié à l’aide du paramètre « targetResourceId ».
* Si la méthode de routage du trafic « Performance » est utilisée, le paramètre « EndpointLocation » doit être renseigné. Sinon, il est facultatif. La valeur doit être un [nom de région Azure valide](https://azure.microsoft.com/regions/).
* Les paramètres « Pondération » et « Priorité » sont facultatifs, comme pour les points de terminaison Azure.
* Le paramètre « MinChildEndpoints » est facultatif. La valeur par défaut est « 1 ». Si le nombre de points de terminaison disponibles tombe sous ce seuil, le profil parent considère le profil enfant comme étant « dégradé » et dirige le trafic vers les autres points de terminaison du profil parent.

### <a name="example-1-adding-nested-endpoints-using-add-aztrafficmanagerendpointconfig-and-set-aztrafficmanagerprofile"></a>Exemple 1 : Ajout de points de terminaison imbriqués à l’aide de `Add-AzTrafficManagerEndpointConfig` et `Set-AzTrafficManagerProfile`

Dans cet exemple, nous créons des profils Traffic Manager enfant et parent, ajoutons l’enfant en tant que point de terminaison imbriqué au parent et validons les modifications.

```powershell
$child = New-AzTrafficManagerProfile -Name child -ResourceGroupName MyRG -TrafficRoutingMethod Priority -RelativeDnsName child -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
$parent = New-AzTrafficManagerProfile -Name parent -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName parent -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
Add-AzTrafficManagerEndpointConfig -EndpointName child-endpoint -TrafficManagerProfile $parent -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "North Europe" -MinChildEndpoints 2
Set-AzTrafficManagerProfile -TrafficManagerProfile $parent
```

Par souci de concision, dans cet exemple, nous n’avons pas ajouté d’autres points de terminaison aux profils enfant ou parent.

### <a name="example-2-adding-nested-endpoints-using-new-aztrafficmanagerendpoint"></a>Exemple 2 : Ajout de points de terminaison imbriqués à l’aide de `New-AzTrafficManagerEndpoint`

Dans cet exemple, nous ajoutons un profil enfant existant en tant que point de terminaison imbriqué à un profil parent existant. Le profil est spécifié en utilisant les noms de profil et de groupe de ressources.

```powershell
$child = Get-AzTrafficManagerEndpoint -Name child -ResourceGroupName MyRG
New-AzTrafficManagerEndpoint -Name child-endpoint -ProfileName parent -ResourceGroupName MyRG -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "North Europe" -MinChildEndpoints 2
```

## <a name="adding-endpoints-from-another-subscription"></a>Ajout de points de terminaison à partir d’un autre abonnement

Traffic Manager peut fonctionner avec les points de terminaison à partir d’abonnements différents. Vous devez basculer vers l’abonnement avec le point de terminaison que vous souhaitez ajouter pour récupérer l’entrée nécessaire à Traffic Manager. Vous devez ensuite passer aux abonnements avec le profil Traffic Manager et lui ajouter le point de terminaison. L’exemple ci-dessous montre comment effectuer cette opération avec une adresse IP publique.

```powershell
Set-AzContext -SubscriptionId $EndpointSubscription
$ip = Get-AzPublicIpAddress -Name $IpAddressName -ResourceGroupName $EndpointRG

Set-AzContext -SubscriptionId $trafficmanagerSubscription
New-AzTrafficManagerEndpoint -Name $EndpointName -ProfileName $ProfileName -ResourceGroupName $TrafficManagerRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled
```

## <a name="update-a-traffic-manager-endpoint"></a>Mettre à jour un point de terminaison Traffic Manager

Il existe deux façons de mettre à jour un point de terminaison Traffic Manager existant :

1. Obtenir le profil Traffic Manager à l’aide de `Get-AzTrafficManagerProfile`, mettre à jour les propriétés de point de terminaison dans le profil et valider les modifications apportées à l’aide de `Set-AzTrafficManagerProfile`. Cette méthode présente l’avantage de pouvoir mettre à jour plusieurs points de terminaison en une seule opération.
2. Obtenir le point de terminaison Traffic Manager à l’aide de `Get-AzTrafficManagerEndpoint`, mettre à jour les propriétés de point de terminaison et valider les modifications apportées à l’aide de `Set-AzTrafficManagerEndpoint`. Cette méthode est plus simple, car elle ne nécessite pas d’indexation dans le tableau de points de terminaison dans le profil.

### <a name="example-1-updating-endpoints-using-get-aztrafficmanagerprofile-and-set-aztrafficmanagerprofile"></a>Exemple 1 : Mise à jour des points de terminaison à l’aide de `Get-AzTrafficManagerProfile` et `Set-AzTrafficManagerProfile`

Dans cet exemple, nous modifions l’ordre de priorité de deux points de terminaison dans un profil existant.

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG
$TmProfile.Endpoints[0].Priority = 2
$TmProfile.Endpoints[1].Priority = 1
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```

### <a name="example-2-updating-an-endpoint-using-get-aztrafficmanagerendpoint-and-set-aztrafficmanagerendpoint"></a>Exemple 2 : Mise à jour d’un point de terminaison à l’aide de `Get-AzTrafficManagerEndpoint` et `Set-AzTrafficManagerEndpoint`

Dans cet exemple, nous modifions le poids d’un point de terminaison unique dans un profil existant.

```powershell
$endpoint = Get-AzTrafficManagerEndpoint -Name myendpoint -ProfileName myprofile -ResourceGroupName MyRG -Type ExternalEndpoints
$endpoint.Weight = 20
Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

## <a name="enabling-and-disabling-endpoints-and-profiles"></a>Activation et désactivation des points de terminaison et des profils

Traffic Manager permet d’activer et de désactiver des points de terminaison individuels, tout en permettant l’activation et la désactivation des profils complets.
Ces modifications peuvent être effectuées par obtention/mise à jour/définition du point de terminaison ou de ressources de profil. Pour rationaliser les opérations courantes, elles sont également prises en charge via les applets de commande dédiées.

### <a name="example-1-enabling-and-disabling-a-traffic-manager-profile"></a>Exemple 1 : Activation et désactivation d’un profil Traffic Manager

Pour activer un profil Traffic Manager, utilisez `Enable-AzTrafficManagerProfile`. Le profil peut être spécifié à l’aide d’un objet de profil. L’objet de profil peut être transmis via le pipeline ou en utilisant le paramètre « -TrafficManagerProfile ». Dans cet exemple, nous spécifions le profil avec le nom de profil et le nom de groupe de ressources.

```powershell
Enable-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup
```

Pour désactiver un profil Traffic Manager :

```powershell
Disable-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup
```

La cmdlet Disable-AzTrafficManagerProfile vous demande de confirmer l’opération. Ce message peut être supprimé en utilisant le paramètre « -Force ».

### <a name="example-2-enabling-and-disabling-a-traffic-manager-endpoint"></a>Exemple 2 : Activation et désactivation d’un point de terminaison Traffic Manager

Pour activer un point de terminaison Traffic Manager, utilisez `Enable-AzTrafficManagerEndpoint`. Il existe deux façons de spécifier le point de terminaison :

1. en utilisant un objet TrafficManagerEndpoint, qui est transmis via le pipeline ou en utilisant le paramètre « -TrafficManagerEndpoint » ;
2. en utilisant le nom du point de terminaison, le type du point de terminaison, le nom du profil et le nom du groupe de ressources :

```powershell
Enable-AzTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG
```

De même, pour désactiver un point de terminaison Traffic Manager :

```powershell
Disable-AzTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG -Force
```

Comme `Disable-AzTrafficManagerProfile`, l’applet de commande `Disable-AzTrafficManagerEndpoint` vous demande de confirmer l’opération. Ce message peut être supprimé en utilisant le paramètre « -Force ».

## <a name="delete-a-traffic-manager-endpoint"></a>Supprimer un point de terminaison Traffic Manager

Pour supprimer des points de terminaison individuels, utilisez l’applet de commande `Remove-AzTrafficManagerEndpoint` :

```powershell
Remove-AzTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG
```

Cette applet de commande vous demande de confirmer l’opération. Ce message peut être supprimé en utilisant le paramètre « -Force ».

## <a name="delete-a-traffic-manager-profile"></a>Supprimer un profil Traffic Manager

Pour supprimer un profil Traffic Manager, utilisez l’applet de commande `Remove-AzTrafficManagerProfile` en indiquant les noms de profil et de groupe de ressources :

```powershell
Remove-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG [-Force]
```

Cette applet de commande vous demande de confirmer l’opération. Ce message peut être supprimé en utilisant le paramètre « -Force ».

Le profil à supprimer peut également être spécifié par un objet de profil :

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
Remove-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile [-Force]
```

Cette séquence peut également être canalisée :

```powershell
Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG | Remove-AzTrafficManagerProfile [-Force]
```

## <a name="next-steps"></a>Étapes suivantes

[Surveillance avec Traffic Manager](traffic-manager-monitoring.md)

[Considérations sur les performances de Traffic Manager](traffic-manager-performance-considerations.md)
