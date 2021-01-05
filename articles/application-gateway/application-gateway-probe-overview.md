---
title: Vue d’ensemble de l’analyse de l’intégrité pour Azure Application Gateway
description: Azure Application Gateway analyse l’intégrité de toutes les ressources de son pool principal et supprime automatiquement du pool les ressources considérées comme défectueuses.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 07/09/2020
ms.author: victorh
ms.openlocfilehash: f0e5a153efe26640e54f386600f07c7b3d4711d0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89649058"
---
# <a name="application-gateway-health-monitoring-overview"></a>Vue d’ensemble de l’analyse d’intégrité Application Gateway

Azure Application Gateway analyse par défaut l’intégrité de toutes les ressources de son pool principal et supprime automatiquement du pool les ressources considérées comme défectueuses. Application Gateway continue de surveiller les instances défaillantes et les réintroduit dans le pool principal intègre une fois qu’elles redeviennent disponibles et répondent aux sondes d’intégrité. Par défaut, la passerelle d’application envoie les sondes d’intégrité avec le même port que celui défini dans les paramètres HTTP du serveur principal. Un port de sonde personnalisé peut être configuré à l’aide d’une sonde d’intégrité personnalisée.

L’adresse IP source qu’Application Gateway utilise pour les sondes d’intégrité dépend du pool de back-ends :
 
- Si l’adresse du serveur dans le pool de back-ends est un point de terminaison public, l’adresse source est l’adresse IP publique front-end de la passerelle d’application.
- Si l’adresse du serveur dans le pool de back-ends est un point de terminaison privé, l’adresse IP source vient de l’espace d’adressage IP privé du sous-réseau de la passerelle d’application.

![exemple de sonde application gateway][1]

En plus d’utiliser la surveillance par sonde d’intégrité par défaut, vous pouvez aussi personnaliser la sonde d’intégrité pour répondre aux exigences de votre application. Dans cet article, nous nous intéressons aux sondes d’intégrité par défaut et personnalisées.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="default-health-probe"></a>Sonde d’intégrité par défaut

Une passerelle d’application configure automatiquement une sonde d’intégrité par défaut lorsque vous ne définissez pas de configuration de sonde personnalisée. Le comportement d’analyse par défaut consiste à lancer une requête HTTP GET aux adresses IP configurées dans le pool de back-ends. En ce qui concerne les sondes par défaut, si les paramètres HTTP du serveur principal sont configurés pour HTTPS, la sonde utilise HTTPS pour tester l’intégrité des serveurs principaux.

Par exemple : Vous configurez votre passerelle d’application de manière à utiliser les serveurs principaux A, B et C, qui recevront le trafic réseau HTTP sur le port 80. Les contrôles de défaillance par défaut testent les trois serveurs toutes les 30 secondes pour obtenir une réponse HTTP correcte, avec un délai d’attente de 30 secondes pour chaque requête. Le [code d’état](https://msdn.microsoft.com/library/aa287675.aspx) d’une réponse HTTP correcte est compris entre 200 et 399. Dans ce cas, la requête HTTP GET pour la sonde d’intégrité se présente comme suit : http://127.0.0.1/.

Si la vérification de la sonde par défaut échoue pour le serveur A, la passerelle d’application arrête de transférer des requêtes vers ce serveur. La sonde par défaut continue de contrôler le serveur A toutes les 30 secondes. Lorsque le serveur A répond correctement à une requête de sonde d’intégrité par défaut, la passerelle d’application recommence à transférer les requêtes vers le serveur.

### <a name="default-health-probe-settings"></a>Paramètres de sonde d’intégrité par défaut

| Propriétés de la sonde | Valeur | Description |
| --- | --- | --- |
| URL de sonde |\<protocol\>://127.0.0.1:\<port\>/ |Le protocole et le port sont hérités des paramètres HTTP principaux auxquels la sonde est associée |
| Intervalle |30 |Durée de l’attente, en secondes, avant l’envoi de la sonde d’intégrité suivante.|
| Délai d’attente |30 |Durée de l’attente, en secondes, de la passerelle d’application pour une réponse de la sonde avant que la sonde ne soit déclarée comme défectueuse. Si une sonde renvoie un état intègre, le serveur principal correspondant est immédiatement marqué comme étant intègre.|
| Seuil de défaillance sur le plan de l’intégrité |3 |Détermine le nombre de sondes à envoyer en cas d’échec de la sonde d’intégrité standard. Dans la référence SKU v1, ces sondes d’intégrité supplémentaires sont envoyées de façon rapprochée pour déterminer rapidement l’intégrité du serveur principal et ne tiennent pas compte de l’intervalle d’analyse. Dans le cas du SKU v2, les sondes d’intégrité attendent l’intervalle. Le serveur principal est marqué comme étant défectueux après que le nombre d’échecs consécutifs a atteint le seuil de défaillance. |

La sonde par défaut examine uniquement \<protocol\>:\//127.0.0.1:\<port\> pour déterminer l’état d’intégrité. Si vous devez configurer la sonde d’intégrité de sorte qu’elle accède à une URL personnalisée ou modifier d’autres paramètres, vous devez utiliser des sondes personnalisées. Pour plus d’informations sur les sondes HTTPS, consultez [Présentation de la terminaison TLS et du TLS de bout en bout sur la passerelle Application Gateway](ssl-overview.md#for-probe-traffic).

### <a name="probe-intervals"></a>Intervalles d'analyse

Toutes les instances d’Application Gateway sondent le serveur principal indépendamment des autres. La même configuration de sonde s’applique à chaque instance d’Application Gateway. Par exemple, si la configuration de sonde consiste à envoyer des sondes d’intégrité toutes les 30 secondes, et si Application Gateway a deux instances, les deux instances envoient la sonde d’intégrité toutes les 30 secondes.

S’il existe plusieurs processus d’écoute, chacun d’entre eux analyse le serveur principal indépendamment des autres. Par exemple, s’il existe deux processus d’écoute pointant vers le même pool principal sur deux ports distincts (configurés par deux paramètres HTTP du serveur principal), chaque processus d’écoute analyse le même serveur principal indépendamment. Dans ce cas, il existe deux sondes provenant de chaque instance d’Application Gateway pour les deux processus d’écoute. S’il existe deux instances d’Application Gateway dans ce scénario, la machine virtuelle principale voit quatre sondes par intervalle d’analyse configuré.

## <a name="custom-health-probe"></a>Sonde d’intégrité personnalisée

Les sondes personnalisées vous permettent d’avoir un contrôle plus précis de l’analyse de l’intégrité. En utilisant des sondes personnalisées, vous pouvez notamment configurer un nom d’hôte personnalisé, un intervalle d’analyse et le nombre de réponses en échec autorisé avant que l’instance de pool principal soit marquée comme non saine.

### <a name="custom-health-probe-settings"></a>Paramètres de sonde d’intégrité personnalisée

Le tableau suivant fournit des définitions pour les propriétés d’une sonde d’intégrité personnalisée.

| Propriétés de la sonde | Description |
| --- | --- |
| Nom |Nom de la sonde. Ce nom est utilisé pour identifier et désigner la sonde dans les paramètres HTTP du serveur principal. |
| Protocol |Protocole utilisé pour envoyer la sonde. Il doit correspondre au protocole défini dans les paramètres HTTP principaux auxquels il est associé.|
| Host |Nom d’hôte pour l’envoi de la sonde. Dans la référence SKU v1, cette valeur est uniquement utilisée pour l’en-tête d’hôte de la requête de sonde. Dans la référence SKU v2, elle est utilisée en tant qu’en-tête d’hôte et SNI. |
| Path |Chemin relatif de la sonde. Un chemin d’accès valide commence par « / » |
| Port |S’il est défini, il est utilisé en tant que port de destination. Sinon, il utilise le même port que les paramètres HTTP auxquels il est associé. Cette propriété est uniquement disponible dans la référence SKU v2
| Intervalle |Intervalle d’analyse en secondes. Cette valeur correspond à l’intervalle de temps qui s’écoule entre deux analyses consécutives. |
| Délai d’attente |Délai d’expiration de l’analyse en secondes. Si aucune réponse valide n’est reçue dans le délai imparti, la sonde est marquée comme étant en échec.  |
| Seuil de défaillance sur le plan de l’intégrité |Nombre de tentatives d’analyse Le serveur principal est marqué comme étant défectueux après que le nombre d’échecs consécutifs a atteint le seuil de défaillance. |

### <a name="probe-matching"></a>Correspondance des sondes

Par défaut, une réponse HTTP(S) avec un code d’état compris entre 200 et 399 est considérée comme intègre. Les sondes d’intégrité personnalisées prennent également en charge deux critères de correspondance. Des critères de correspondance peuvent être utilisés pour éventuellement modifier l’interprétation par défaut de ce qui rend une réponse intègre.

Les éléments suivants sont des critères de correspondance : 

- **Correspondance de code d’état de la réponse HTTP** : le critère de correspondance de la sonde pour l’acceptation du code de réponse http spécifié par l’utilisateur ou des plages de codes de réponse. Les codes d’état de réponse séparées par des virgules individuelles ou une plage de codes d’état sont pris en charge.
- **Correspondance du corps de la réponse HTTP** : critère de correspondance de la sonde qui examine le corps de la réponse HTTP et correspond à une chaîne spécifiée par l’utilisateur. La correspondance ne tient compte que de la présence d’une chaîne spécifiée par l’utilisateur dans le corps de la réponse et n’est pas une correspondance d’expression régulière complète.

Les critères de correspondance peuvent être spécifiés à l’aide de la cmdlet `New-AzApplicationGatewayProbeHealthResponseMatch`.

Par exemple :

```azurepowershell
$match = New-AzApplicationGatewayProbeHealthResponseMatch -StatusCode 200-399
$match = New-AzApplicationGatewayProbeHealthResponseMatch -Body "Healthy"
```
Une fois les critères de correspondance spécifiés, ils peuvent être joints à la configuration de la sonde à l’aide d’un `-Match` paramètre dans PowerShell.

## <a name="nsg-considerations"></a>Considérations pour un groupe de sécurité réseau

Vous devez autoriser le trafic Internet entrant sur les ports TCP 65503-65534 pour la référence (SKU) Application Gateway v1, et sur les ports TCP 65200-65535 pour la référence (SKU) v2 avec le sous-réseau de destination en tant que **Any** et la source en tant que balise de service **GatewayManager**. Cette plage de ports est nécessaire pour la communication avec l’infrastructure Azure.

En outre, la connectivité Internet sortante ne peut pas être bloquée, et le trafic entrant provenant de la balise **AzureLoadBalancer** doit être autorisé.

Pour plus d’informations, consultez [Présentation de la configuration d’Application Gateway](configuration-infrastructure.md#network-security-groups).

## <a name="next-steps"></a>Étapes suivantes
Après vous être familiarisé avec l’analyse d’intégrité Application Gateway, vous pouvez configurer une [sonde d’intégrité personnalisée](application-gateway-create-probe-portal.md) dans le portail Azure ou une [sonde d’intégrité personnalisée](application-gateway-create-probe-ps.md) à l’aide de PowerShell et du modèle de déploiement Azure Resource Manager.

[1]: ./media/application-gateway-probe-overview/appgatewayprobe.png
