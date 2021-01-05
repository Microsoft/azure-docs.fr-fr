---
title: Mise à jour du schéma Azure Traffic Analytics – Mars 2020 | Microsoft Docs
description: Exemples de requêtes avec de nouveaux champs dans le schéma Traffic Analytics.
services: network-watcher
documentationcenter: na
author: vinigam
manager: agummadi
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/06/2020
ms.author: vinigam
ms.openlocfilehash: be69e7fc0c836257b92aa8573826fe1e8a36d770
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94948440"
---
# <a name="sample-queries-with-new-fields-in-the-traffic-analytics-schema-august-2019-schema-update"></a>Exemples de requêtes avec de nouveaux champs dans le schéma Traffic Analytics (mise à jour du schéma d’août 2019)

Le [schéma de journal Traffic Analytics](./traffic-analytics-schema.md) inclut les nouveaux champs suivants : **SrcPublicIPs_s**, **DestPublicIPs_s**, **NSGRule_s**. Les nouveaux champs fournissent des informations sur les adresses IP source et de destination et simplifient les requêtes.

Dans les prochains mois, les anciens champs suivants seront déconseillés : **VMIP_s**, **Subscription_g**, **Region_s**, **NSGRules_s**, **Subnet_s**, **VM_s**, **NIC_s**, **PublicIPs_s** et **FlowCount_d**.

Les trois exemples suivants montrent comment remplacer les anciens champs par les nouveaux.

## <a name="example-1-vmip_s-subscription_g-region_s-subnet_s-vm_s-nic_s-and-publicips_s-fields"></a>Exemple 1 : Champs VMIP_s, Subscription_g, Region_s, Subnet_s, VM_s, NIC_s et PublicIPs_s

Nous n’avons pas à déduire les cas source et de destination à partir du champ **FlowDirection_s** pour les flux AzurePublic et ExternalPublic. Il peut également être inapproprié d’utiliser le champ **FlowDirection_s** pour une appliance virtuelle réseau.

```Old Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "1"
| extend isAzureOrExternalPublicFlows = FlowType_s in ("AzurePublic", "ExternalPublic")
| extend SourceAzureVM = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', VM_s, "N/A"), VM1_s),
SourceAzureVMIP = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', VM_s, "N/A"), SrcIP_s),
SourceAzureVMSubscription = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', Subscription_g, "N/A"), Subscription1_g),
SourceAzureRegion = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', Region_s, "N/A"), Region1_s),
SourceAzureSubnet = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', Subnet_s, "N/A"), Subnet1_s),
SourceAzureNIC = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'O', NIC_s, "N/A"), NIC1_s),
DestAzureVM = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', VM_s, "N/A"), VM2_s),
DestAzureVMIP = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', VM_s, "N/A"), DestIP_s),
DestAzureVMSubscription = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', Subscription_g, "N/A"), Subscription2_g),
DestAzureRegion = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', Region_s, "N/A"), Region2_s),
DestAzureSubnet = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', Subnet_s, "N/A"), Subnet2_s),
DestAzureNIC = iif(isAzureOrExternalPublicFlows, iif(FlowDirection_s == 'I', NIC_s, "N/A"), NIC2_s),
SourcePublicIPsAggregated = iif(isAzureOrExternalPublicFlows and FlowDirection_s == 'I', PublicIPs_s, "N/A"),
DestPublicIPsAggregated = iif(isAzureOrExternalPublicFlows and FlowDirection_s == 'O', PublicIPs_s, "N/A")
```


```New Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "2"
| extend SourceAzureVM = iif(isnotempty(VM1_s), VM1_s, "N/A"),
SourceAzureVMIP = iif(isnotempty(SrcIP_s), SrcIP_s, "N/A"),
SourceAzureVMSubscription = iif(isnotempty(Subscription1_g), Subscription1_g, "N/A"),
SourceAzureRegion = iif(isnotempty(Region1_s), Region1_s, "N/A"),
SourceAzureSubnet = iif(isnotempty(Subnet1_s), Subnet1_s, "N/A"),
SourceAzureNIC = iif(isnotempty(NIC1_s), NIC1_s, "N/A"),
DestAzureVM = iif(isnotempty(VM2_s), VM2_s, "N/A"),
DestAzureVMIP = iif(isnotempty(DestIP_s), DestIP_s, "N/A"),
DestAzureVMSubscription = iif(isnotempty(Subscription2_g), Subscription2_g, "N/A"),
DestAzureRegion = iif(isnotempty(Region2_s), Region2_s, "N/A"),
DestAzureSubnet = iif(isnotempty(Subnet2_s), Subnet2_s, "N/A"),
DestAzureNIC = iif(isnotempty(NIC2_s), NIC2_s, "N/A"),
SourcePublicIPsAggregated = iif(isnotempty(SrcPublicIPs_s), SrcPublicIPs_s, "N/A"),
DestPublicIPsAggregated = iif(isnotempty(DestPublicIPs_s), DestPublicIPs_s, "N/A")
```

## <a name="example-2-nsgrules_s-field"></a>Exemple 2 : Champ NSGRules_s

L’ancien champ utilise le format :

`<Index value 0)>|<NSG_ RuleName>|<Flow Direction>|<Flow Status>|<FlowCount ProcessedByRule>`

Nous ne regroupons plus de données dans un groupe de sécurité réseau. Dans le schéma mis à jour, **NSGList_s** contient un seul groupe de sécurité réseau. En outre, **NSGRules** contient une seule règle. Nous avons supprimé la mise en forme compliquée ici et dans d’autres champs, comme indiqué dans l’exemple.

```Old Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "1"
| extend NSGRuleComponents = split(NSGRules_s, "|")
| extend NSGName = NSGList_s // remains same
| extend NSGRuleName = NSGRuleComponents[1],
         FlowDirection = NSGRuleComponents[2],
         FlowStatus = NSGRuleComponents[3],
         FlowCountProcessedByRule = NSGRuleComponents[4]
| project NSGName, NSGRuleName, FlowDirection, FlowStatus, FlowCountProcessedByRule
```

```New Kusto query
AzureNetworkAnalytics_CL
| where SubType_s == "FlowLog" and FASchemaVersion_s == "2"
| extend NSGRuleComponents = split(NSGRules_s, "|")
| project NSGName = NSGList_s,
NSGRuleName = NSGRule_s ,
FlowDirection = FlowDirection_s,
FlowStatus = FlowStatus_s,
FlowCountProcessedByRule = AllowedInFlows_d + DeniedInFlows_d + AllowedOutFlows_d + DeniedOutFlows_d
```

## <a name="example-3-flowcount_d-field"></a>Exemple 3 : Champ FlowCount_d

Étant donné que nous ne recherchons pas de données sur le groupe de sécurité réseau, le **FlowCount_d** est simplement :

**AllowedInFlows_d** + **DeniedInFlows_d** + **AllowedOutFlows_d** + **DeniedOutFlows_d**

Seul l’un des quatre champs sera différent de zéro. Les trois autres champs seront des valeurs nulles. Les champs sont renseignés pour indiquer l’état et le nombre dans la carte réseau où le flux a été capturé.

Pour illustrer ces conditions :

- Si le flux est autorisé, l’un des champs préfixés « autorisés » est rempli.
- Si le flux est refusé, l’un des champs préfixés « refusés » est rempli.
- Si le flux est entrant, l’un des champs préfixés « InFlows_d » est rempli.
- Si le flux est sortant, l’un des champs préfixés « OutFlows_d » est rempli.

Selon les conditions, nous savons que l’un des quatre champs est rempli.

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir des réponses aux questions fréquemment posées, consultez [Traffic Analytics frequently asked questions](traffic-analytics-faq.md) (Forum aux questions Traffic Analytics).
- Pour plus d’informations sur les fonctionnalités, consultez la [documentation sur Traffic Analytics](traffic-analytics.md).