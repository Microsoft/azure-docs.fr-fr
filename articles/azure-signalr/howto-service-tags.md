---
title: Utiliser des étiquettes de service
titleSuffix: Azure SignalR Service
description: Utiliser des étiquettes de service pour autoriser le trafic sortant vers votre instance Azure SignalR Service
services: signalr
author: ArchangelSDY
ms.service: signalr
ms.topic: article
ms.date: 05/06/2020
ms.author: dayshen
ms.openlocfilehash: aaa97c2cb062f30b1260ec7f80f85a3caccf932f
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92152315"
---
# <a name="use-service-tags-for-azure-signalr-service"></a>Utiliser des étiquettes de service pour Azure SignalR Service

Vous pouvez utiliser des [étiquettes de service](../virtual-network/network-security-groups-overview.md#service-tags) pour Azure SignalR Service lors de la configuration du [groupe de sécurité réseau](../virtual-network/network-security-groups-overview.md#network-security-groups). Cela vous permet de définir une règle de sécurité réseau sortante pour les points de terminaison Azure SignalR Service, sans devoir coder en dur des adresses IP.

Azure SignalR Service gère ces étiquettes de service. Vous ne pouvez pas créer votre propre étiquette de service ou modifier une étiquette existante. Microsoft gère ces préfixes d'adresse correspondant à l'étiquette de service, et met automatiquement à jour l'étiquette de service lorsque les adresses changent.

## <a name="use-service-tag-on-portal"></a>Utiliser une étiquette de service sur le portail

Vous pouvez autoriser le trafic sortant vers Azure SignalR Service en ajoutant une nouvelle règle de sécurité réseau sortante :

1. Accédez au groupe de sécurité réseau.

1. Cliquez sur le menu des paramètres appelé **Règles de sécurité de trafic sortant**.

1. Cliquez sur le bouton **+ Ajouter** en haut.

1. Sélectionnez **Étiquette de service** sous **Destination**.

1. Sélectionnez **AzureSignalR** sous **Étiquette de service de destination**.

1. Indiquez **443** dans **Plages de ports de destination**.

    ![Créer une règle de sécurité de trafic sortant](media/howto-service-tags/portal-add-outbound-security-rule.png)

1. Ajustez les autres champs selon vos besoins.

1. Cliquez sur **Add**.


## <a name="next-steps"></a>Étapes suivantes

- [Groupes de sécurité réseau - Étiquettes de service](../virtual-network/network-security-groups-overview.md#security-rules)