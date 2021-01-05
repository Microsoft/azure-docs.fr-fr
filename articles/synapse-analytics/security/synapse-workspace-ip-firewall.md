---
title: Configurer des règles de pare-feu IP
description: Article qui explique comment configurer des règles de pare-feu IP dans Azure Synapse Analytics.
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: security
ms.date: 04/15/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: 3cbd9fd697bbae8779f5b2b9fa4ab817efce30ac
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454644"
---
# <a name="azure-synapse-analytics-ip-firewall-rules"></a>Règles de pare-feu IP Azure Synapse Analytics

Cet article explique ce que sont les règles de pare-feu IP et montre comment les configurer dans Azure Synapse Analytics.

## <a name="ip-firewall-rules"></a>Règles de pare-feu IP

Les règles de pare-feu IP octroient ou refusent l’accès à votre espace de travail Synapse en fonction de l’adresse IP d’origine de chaque requête. Vous pouvez configurer des règles de pare-feu IP pour votre espace de travail. Les règles de pare-feu IP configurées au niveau de l’espace de travail s’appliquent à tous les points de terminaison publics de l’espace de travail (pools SQL dédiés, pool SQL serverless et développement).

## <a name="create-and-manage-ip-firewall-rules"></a>Créer et gérer des règles de pare-feu IP

Il existe deux façons d’ajouter des règles de pare-feu IP à un espace de travail Synapse. Pour ajouter un pare-feu IP à votre espace de travail, sélectionnez **Sécurité + réseau** et cochez **Autoriser les connexions de toutes les adresses IP** lors de la création de l’espace de travail.

![Capture d’écran avec mise en évidence du bouton Sécurité + réseau.](./media/synpase-workspace-ip-firewall/ip-firewall-1.png)

![Configuration IP de l’espace de travail Synapse dans le portail Azure.](./media/synpase-workspace-ip-firewall/ip-firewall-2.png)

Vous pouvez également ajouter des règles de pare-feu IP à un espace de travail Synapse après la création de l’espace de travail. Sélectionnez **Pare-feu** sous **Sécurité** dans le portail Azure. Pour ajouter une nouvelle règle de pare-feu IP, attribuez-lui un nom, une adresse IP de début et une adresse IP de fin. Sélectionnez **Enregistrer** lorsque vous avez terminé.

![Configuration IP de l’espace de travail Azure Synapse dans le portail Azure](./media/synpase-workspace-ip-firewall/ip-firewall-3.png)

## <a name="connect-to-synapse-from-your-own-network"></a>Se connecter à Synapse à partir de votre propre réseau

Vous pouvez vous connecter à votre espace de travail Synapse à l’aide de Synapse Studio. Vous pouvez également utiliser SQL Server Management Studio (SSMS) pour vous connecter aux ressources SQL (pools SQL dédiés et pool SQL serverless) dans votre espace de travail.

Vérifiez que le pare-feu sur votre réseau et ordinateur local autorise les communications sortantes sur les ports TCP 80, 443 et 1443 pour Synapse Studio.

Vous devez aussi autoriser les communications sortantes sur le port UDP 53 pour Synapse Studio. Pour vous connecter à l’aide d’outils tels que SSMS et Power BI, vous devez autoriser la communication sortante sur le port TCP 1433.

Si vous utilisez le paramètre de stratégie Rediriger la connexion par défaut, vous devrez peut-être aussi autoriser la communication sortante sur d’autres ports. Vous pouvez en savoir plus sur les stratégies de connexion [ici](https://docs.microsoft.com/azure/sql-database/sql-database-connectivity-architecture#connection-policy).

## <a name="next-steps"></a>Étapes suivantes

Créer un [espace de travail Azure Synapse](../quickstart-create-workspace.md)

Créer un espace de travail Azure Synapse avec un [réseau virtuel d’espace de travail managé](./synapse-workspace-managed-vnet.md)