---
title: Protection contre l’exfiltration de données pour les espaces de travail Azure Synapse Analytics
description: Cet article explique la protection contre l’exfiltration de données dans Azure Synapse Analytics
author: NanditaV
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: security
ms.date: 12/01/2020
ms.author: NanditaV
ms.reviewer: jrasnick
ms.openlocfilehash: 71210cdcc2b3758a59a1b41816e6468556e94808
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96518258"
---
# <a name="data-exfiltration-protection-for-azure-synapse-analytics-workspaces"></a>Protection contre l’exfiltration de données pour les espaces de travail Azure Synapse Analytics
Cet article explique la protection contre l’exfiltration de données dans Azure Synapse Analytics

## <a name="securing-data-egress-from-synapse-workspaces"></a>Sécurisation de la sortie de données à partir d’espaces de travail Synapse
Les espaces de travail Azure Synapse Analytics prennent en charge l’activation de la protection contre l’exfiltration de données pour les espaces de travail. Avec la protection contre l’exfiltration, vous pouvez vous protéger contre les personnes malveillantes qui accèdent à vos ressources Azure et exfiltrent des données sensibles vers des emplacements situés en dehors de l’étendue de votre organisation. Au moment de la création de l’espace de travail, vous pouvez choisir de configurer l’espace de travail avec un réseau virtuel managé et une protection supplémentaire contre l’exfiltration de données. Lorsqu’un espace de travail est créé avec un [réseau virtuel managé](./synapse-workspace-managed-vnet.md), l’intégration des données et les ressources Spark sont déployées dans le réseau virtuel managé. Les pools SQL dédiés et les pools SQL serverless de l’espace de travail ont des fonctionnalités multi-locataire et, par conséquent, doivent exister en dehors du réseau virtuel managé. Pour les espaces de travail avec la protection contre l’exfiltration de données, les ressources au sein du réseau virtuel managé communiquent toujours sur des [points de terminaison privés managés](./synapse-workspace-managed-private-endpoints.md) et les ressources SQL Synapse peuvent uniquement se connecter aux ressources Azure autorisées (cibles des connexions de point de terminaison privé managé approuvées à partir de l’espace de travail). 

>[!Note]
>Vous ne pouvez pas modifier la configuration de l’espace de travail pour le réseau virtuel managé et la protection contre l’exfiltration de données après la création de l’espace de travail.

## <a name="managing-synapse-workspace-data-egress-to-approved-targets"></a>Gestion de la sortie des données de l’espace de travail Synapse vers les cibles approuvées
Une fois l’espace de travail créé avec l’exfiltration de données activée, les propriétaires de la ressource d’espace de travail peuvent gérer la liste des locataires Azure AD approuvés pour l’espace de travail. Les utilisateurs disposant des [autorisations appropriées](./synapse-workspace-access-control-overview.md) sur l’espace de travail peuvent utiliser Synapse Studio pour créer des demandes de connexion de point de terminaison privé managé aux ressources dans les locataires Azure AD approuvés de l’espace de travail. La création d’un point de terminaison privé managé sera bloquée si l’utilisateur tente de créer une connexion de point de terminaison privé à une ressource dans un locataire non approuvé.

## <a name="sample-workspace-with-data-exfiltration-protection-enabled"></a>Exemple d’espace de travail avec la protection contre l’exfiltration de données activée
Utilisons un exemple pour illustrer la protection contre l’exfiltration de données pour les espaces de travail Synapse. Contoso dispose de ressources Azure dans le locataire A et le locataire B et il est nécessaire que ces ressources se connectent en toute sécurité. Un espace de travail Synapse a été créé dans le locataire A avec le locataire B ajouté en tant que locataire Azure AD approuvé. Le diagramme montre les connexions de point de terminaison privé aux comptes Stockage Azure dans le locataire A et le locataire B qui ont été approuvées par les propriétaires du compte de stockage. Le diagramme montre également la création bloquée d’un point de terminaison privé. La création de ce point de terminaison privé a été bloquée, car il ciblait un compte Stockage Azure dans le locataire Fabrikam Azure AD, qui n’est pas un locataire Azure AD approuvé pour l’espace de travail de Contoso. 
:::image type="content" source="media/workspace-data-exfiltration-protection/workspace-data-exfiltration-protection-diagram.png" alt-text="Ce diagramme montre comment la protection contre l’exfiltration de données est implémentée pour les espaces de travail Synapse":::

>[!IMPORTANT]
>Les ressources dans les locataires autres que le locataire de l’espace de travail ne doivent pas avoir de règles de pare-feu bloquantes pour que les pools SQL puissent s’y connecter. Les ressources au sein du réseau virtuel managé de l’espace de travail, comme les clusters Spark, peuvent se connecter via des liaisons privées managées aux ressources protégées par un pare-feu.
## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [créer un espace de travail avec la protection contre l’exfiltration de données activée](./how-to-create-a-workspace-with-data-exfiltration-protection.md)

En savoir plus sur le [Réseau virtuel d’espace de travail managé](./synapse-workspace-managed-vnet.md)

En savoir plus sur les [Points de terminaison privés managés](./synapse-workspace-managed-private-endpoints.md)

[Créer des points de terminaison privés managés à vos sources de données](./how-to-create-managed-private-endpoints.md)
