---
title: Logique de traitement des règles d’Azure Firewall Manager
description: En savoir plus sur la logique de traitement des règles du service Pare-feu Azure
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: conceptual
ms.date: 06/30/2020
ms.author: victorh
ms.openlocfilehash: 9184bf7baa85420e067edb4c0aafccb7e6711225
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86512178"
---
# <a name="azure-firewall-rule-processing-logic"></a>Logique de traitement des règles du service Pare-feu Azure

Le service Pare-feu Azure comporte des règles de traduction d’adresses réseau (NAT), des règles de réseau et des règles d’application. Les règles sont traitées selon leur type.

## <a name="network-rules-and-applications-rules"></a>Règles de réseau et règles d’application

Le service applique d’abord les règles de réseau, puis les règles d’application. Ce processus prend fin dès qu’une règle est exécutée. Par conséquent, si une correspondance est trouvée dans les règles de réseau, les règles d’application ne sont pas traitées.  Si aucune règle de réseau ne correspond, et si le protocole du paquet est HTTP/HTTPS, le paquet est évalué par les règles d’application. Si aucune correspondance n’est trouvée, le paquet est évalué par rapport à la collection de règles de l’infrastructure. S’il n’existe toujours pas de correspondance, le paquet est refusé par défaut.

## <a name="nat-rules"></a>Règles NAT

Vous pouvez activer la connectivité entrante en configurant le mode DNAT (Destination Network Address Translation), comme décrit dans l’article [Tutoriel : Filtrer le trafic entrant avec pare-feu Azure DNAT via le portail Azure](../firewall/tutorial-firewall-dnat.md). Les règles DNAT sont appliquées en premier. Si une correspondance est trouvée, une règle de réseau correspondante implicite autorisant le trafic traduit est ajoutée. Vous pouvez remplacer ce comportement en ajoutant explicitement une collection de règles de réseau avec des règles de refus correspondant au trafic traduit. Aucune règle d’application n’est appliquée pour ces connexions.

## <a name="inherited-rules"></a>Règles héritées

Les collections de règles de réseau héritées d’une stratégie parent sont toujours prioritaires sur les collections de règles de réseau définies dans le cadre de votre nouvelle stratégie. La même logique s’applique également aux collections de règles d’application. Cependant, les collections de règles de réseau sont toujours traitées avant les collections de règles d’application quel que soit l’héritage.

Par défaut, votre stratégie hérite du mode de renseignement sur les menaces de la stratégie parente. Vous pouvez la remplacer en définissant le mode d’intelligence des menaces sur une valeur différente dans la page des paramètres de la stratégie. Il est uniquement possible de remplacer cette valeur par une valeur plus stricte. Par exemple, si vous définissez la stratégie parente sur *Alerte uniquement*, vous pouvez configurer cette stratégie locale sur *Alerter et refuser*, mais vous ne pouvez pas la désactiver.

## <a name="next-steps"></a>Étapes suivantes

- [Découvrir Azure Firewall Manager](overview.md)