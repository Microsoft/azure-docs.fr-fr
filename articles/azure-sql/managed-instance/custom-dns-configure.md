---
title: Système DNS personnalisé
titleSuffix: Azure SQL Managed Instance
description: Cette rubrique détaille les options de configuration d’un DNS personnalisé avec Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 07/17/2019
ms.openlocfilehash: 017698975706aa8501cd059351cf9a9d88594f77
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92779898"
---
# <a name="configure-a-custom-dns-for-azure-sql-managed-instance"></a>Configurer un DNS personnalisé pour Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance doit être déployé au sein d’un [réseau virtuel](../../virtual-network/virtual-networks-overview.md) Azure. Quelques scénarios (comme le courrier de base de données, les serveurs liés à d’autres instances SQL dans votre cloud ou environnement hybride) nécessitent des noms d’hôte privés pour être résolus depuis SQL Managed Instance. Si tel est le cas, vous devez configurer un DNS personnalisé dans Azure. 

Étant donné que l’instance gérée SQL utilise le même DNS pour ses opérations internes, configurez le serveur DNS personnalisé pour qu’il puisse résoudre les noms de domaine public.

> [!IMPORTANT]
> Utilisez toujours un nom de domaine complet (FQDN) pour les serveurs de messagerie, l’instance SQL Server et les autres services, même s’ils se trouvent dans votre zone DNS privée. Par exemple, utilisez `smtp.contoso.com` comme serveur de messagerie, car `smtp` ne sera pas résolu correctement. La création d’un serveur lié ou d’une réplication qui référence des machines virtuelles SQL Server sur le même réseau virtuel nécessite également un nom de domaine complet et un suffixe DNS par défaut. Par exemple : `SQLVM.internal.cloudapp.net`. Pour plus d’informations, consultez [Résolution de noms utilisant votre propre serveur DNS](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

> [!IMPORTANT]
> La mise à jour des serveurs DNS du réseau virtuel n’affectera pas SQL Managed Instance immédiatement. La configuration DNS de SQL Managed Instance est mise à jour après l’expiration du bail DHCP ou après la mise à niveau de la plateforme, selon l’événement qui se produit en premier. **Les utilisateurs sont invités à définir la configuration DNS de leur réseau virtuel avant de créer leur première instance Managed Instance.**

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir une vue d’ensemble, consultez [Présentation d’Azure SQL Managed Instance](sql-managed-instance-paas-overview.md).
- Pour suivre un didacticiel vous expliquant comment créer une nouvelle instance gérée, consultez [Créer une instance gérée](instance-create-quickstart.md).
- Pour plus d’informations sur la configuration d’un réseau virtuel pour une option Managed Instance, consultez [Configurer un réseau virtuel pour Azure SQL Database Managed Instance](connectivity-architecture-overview.md).