---
title: Contrôles de sécurité pour Azure Relay
description: Cet article fournit une liste de contrôle des contrôles de sécurité intégrés pour l’évaluation de Azure Relay.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 64abee031bb20e2bdb10bf1cc3cd77e135713550
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85316623"
---
# <a name="security-controls-for-azure-relay"></a>Contrôles de sécurité pour Azure Relay

Cet article décrit les contrôles de sécurité intégrés à Azure Relay.

[!INCLUDE [Security controls Header](../../includes/security-controls-header.md)]

## <a name="network"></a>Réseau

| Contrôle de sécurité | Oui/Non | Notes | Documentation |
|---|---|--|--|
| Prise en charge du point de terminaison de service| Non |  |   |
| Prise en charge de l’isolement réseau et de l’installation de pare-feu| Non |  |   |
| Prise en charge du tunneling forcé| N/A | Le relais est le tunnel TLS  |   |

## <a name="monitoring--logging"></a>Supervision et journalisation

| Contrôle de sécurité | Oui/Non | Notes| Documentation |
|---|---|--|--|
| Prise en charge de la supervision Azure (Log analytics, App insights, etc.)| Oui | |   |
| Journalisation et audit du plan de gestion et de contrôle| Oui | Via [Azure Resource Manager](../azure-resource-manager/index.yml). |   |
| Journalisation et audit du plan de données| Oui | La réussite, l’échec et les erreurs de la connexion sont journalisés.  |   |

## <a name="identity"></a>Identité

| Contrôle de sécurité | Oui/Non | Notes| Documentation |
|---|---|--|--|
| Authentification| Oui | Via SAS. | [Authentification et autorisation Azure Relay](relay-authentication-and-authorization.md) |
| Autorisation|  Oui | Via SAS. | [Authentification et autorisation Azure Relay](relay-authentication-and-authorization.md) |

## <a name="data-protection"></a>Protection de données

| Contrôle de sécurité | Oui/Non | Notes | Documentation |
|---|---|--|--|
| Chiffrement côté serveur au repos : Clés managées par Microsoft |  N/A | Le relais est un socket web et ne persiste pas dans les données. |   |
| Chiffrement côté serveur au repos : clés gérées par le client (BYOK) | Non | Utilise uniquement les certificats Microsoft TLS.  |   |
| Chiffrement au niveau des colonnes (Azure Data Services)| N/A | |   |
| Chiffrement en transit (comme ExpressRoute, de réseau virtuel et de réseau virtuel à réseau virtuel)| Oui | Le service nécessite TLS. |   |
| Appels d’API chiffrés| Oui | HTTPS. |


## <a name="configuration-management"></a>Gestion des configurations

| Contrôle de sécurité | Oui/Non | Notes| Documentation |
|---|---|--|--|
| Prise en charge de la gestion de la configuration (gestion de version de la configuration, etc.)| Oui | Via [Azure Resource Manager](../azure-resource-manager/index.yml).|   |

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en plus sur les [contrôles de sécurité intégrés des services Azure](../security/fundamentals/security-controls.md).