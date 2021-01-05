---
title: Configuration de la Threat Intelligence du Pare-feu Azure
description: Découvrez comment configurer le filtrage basé sur le renseignement sur les menaces de manière à ce que votre stratégie Pare-feu Azure donne l’alerte et rejette le trafic depuis ou vers des adresses IP et des domaines malveillants connus.
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: article
ms.date: 06/30/2020
ms.author: victorh
ms.openlocfilehash: a663c5f3bcf3492c4a9bc74fe93c6ed6a86137ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90530639"
---
# <a name="azure-firewall-threat-intelligence-configuration"></a>Configuration de la Threat Intelligence du Pare-feu Azure

Le filtrage basé sur Threat Intelligence peut être configuré de manière à ce que votre stratégie de pare-feu Azure donne l’alerte et rejette le trafic depuis ou vers des adresses IP et des domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence. [Intelligent Security Graph](https://www.microsoft.com/security/operations/intelligence) alimente Microsoft Threat Intelligence et est utilisé par de nombreux services, dont Azure Security Center.<br>

Si vous avez configuré le filtrage basé sur la Threat Intelligence, les règles associées sont traitées avant les règles NAT, les règles de réseau ou les règles d’application.

:::image type="content" source="media/threat-intelligence-settings/threat-intelligence-policy.png" alt-text="Stratégie d’informations sur les menaces":::

## <a name="threat-intelligence-mode"></a>Mode de renseignement sur les menaces

Vous pouvez vous contenter d'enregistrer une alerte lorsqu'une règle est déclenchée ou vous pouvez choisir le mode d'alerte et le mode de refus.

Par défaut, le filtrage basé sur la Threat Intelligence est activé en mode d'alerte.

## <a name="allowed-list-addresses"></a>Adresses de la liste autorisée

Vous pouvez configurer une liste d’adresses IP autorisées afin que les informations sur les menaces ne filtrent pas les adresses, les plages ou les sous-réseaux que vous spécifiez.



## <a name="logs"></a>Journaux d’activité

L'extrait de journal suivant montre une règle déclenchée :

```
{
    "category": "AzureFirewallNetworkRule",
    "time": "2018-04-16T23:45:04.8295030Z",
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
    "operationName": "AzureFirewallThreatIntelLog",
    "properties": {
         "msg": "HTTP request from 10.0.0.5:54074 to somemaliciousdomain.com:80. Action: Alert. ThreatIntel: Bot Networks"
    }
}
```

## <a name="testing"></a>Test

- **Test sortant** : les alertes de trafic sortant doivent être exceptionnelles, car elles signifient que votre environnement a été compromis. Pour tester le fonctionnement des alertes de trafic sortant, un nom de domaine complet de test a été créé afin de déclencher une alerte. Utilisez **testmaliciousdomain.eastus.cloudapp.azure.com** pour vos tests sortants.

- **Test entrant** : vous pouvez vous attendre à recevoir des alertes de trafic entrant si des règles DNAT sont configurées sur le pare-feu, même si seules certaines sources sont autorisées sur la règle DNAT et que le trafic est autrement refusé. Le Pare-feu Azure n’alerte pas sur tous les scanneurs de ports connus ; il alerte uniquement sur les scanneurs connus pour leurs activités malveillantes.

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [Rapport des renseignements de sécurité de Microsoft](https://www.microsoft.com/en-us/security/operations/security-intelligence-report)