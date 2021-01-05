---
title: Résoudre les problèmes de connectivité entre Synapse Studio et Stockage
description: Résoudre les problèmes de connectivité entre Synapse Studio et Stockage
author: saveenr
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: xujiang1
ms.reviewer: jrasnick
ms.openlocfilehash: cee6d030a9639a7203a32a3c0957733cecb1f8b6
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445321"
---
# <a name="troubleshoot-connectivity-between-azure-synapse-analytics-synapse-studio-and-storage"></a>Résoudre les problèmes de connectivité entre Azure Synapse Analytics Synapse Studio et Stockage

Dans Synapse Studio, vous pouvez explorer les ressources de données qui se trouvent dans votre stockage lié. Ce guide vous aidera à résoudre les problèmes de connectivité lorsque vous essayez d’accéder à vos ressources de données. 

## <a name="case-1-storage-account-lacks-proper-permissions"></a>1er cas : Le compte de stockage ne dispose pas des autorisations appropriées

Si votre compte de stockage ne dispose pas des autorisations appropriées, vous ne pouvez pas développer la structure de stockage par le biais de « Données » --> « Lié » dans Synapse Studio. Voir la capture d’écran des symptômes du problème ci-dessous. 

Le message d’erreur détaillé peut varier, mais la signification générale du message d’erreur est celle-ci : « Cette requête n’est pas autorisée à effectuer cette opération. »

![Problème de connectivité de stockage 1](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue.1.png)

**SOLUTION** : Pour affecter votre compte au rôle approprié, consultez [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données blob et de file d’attente](../../storage/common/storage-auth-aad-rbac-portal.md).


## <a name="case-2-failed-to-send-the-request-to-storage-server"></a>2e cas : Échec de l’envoi de la requête au serveur de stockage

Quand vous sélectionnez la flèche pour développer la structure de stockage dans « Données » --> « Lié » dans Synapse Studio, le problème « REQUEST_SEND_ERROR » peut s’afficher dans le volet gauche. Voir la capture d’écran des symptômes du problème ci-dessous :

![Problème de connectivité de stockage 2](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue.2.png)

Plusieurs raisons peuvent être à l’origine de ce problème :

### <a name="the-storage-resource-is-behind-a-vnet-and-a-storage-private-endpoint-needs-to-configure"></a>La ressource de stockage se trouve derrière un réseau virtuel et un point de terminaison privé de stockage doit être configuré

**SOLUTION** : Dans ce cas, vous devez configurer le point de terminaison privé de stockage pour votre compte de stockage. Pour savoir comment configurer le point de terminaison privé de stockage pour le réseau virtuel, consultez [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux objets blob et données de file d’attente](../security/how-to-connect-to-workspace-from-restricted-network.md).

Vous pouvez utiliser la commande « nslookup \<storage-account-name\>.dfs.core.windows.net » pour vérifier la connectivité après la configuration du point de terminaison privé de stockage. Elle doit retourner une chaîne semblable à celle-ci : « \<storage-account-name\>.privatelink.dfs.core.windows.net ».

### <a name="the-storage-resource-is-not-behind-a-vnet-but-the-blob-service-azure-ad-endpoint-is-not-accessible-due-to-firewall-configured"></a>La ressource de stockage n’est pas derrière un réseau virtuel, mais le point de terminaison du service BLOB (Azure AD) n’est pas accessible en raison de la configuration du pare-feu

**SOLUTION** : Dans ce cas, vous devez ouvrir votre compte de stockage dans le portail Azure. Dans le volet de navigation de gauche, faites défiler la liste jusqu’à **Support + résolution des problèmes** et sélectionnez **Vérification de la connectivité** pour vérifier l’état de connectivité de **Service BLOB (Azure AD)** . S’il n’est pas accessible, suivez le guide promu pour vérifier la configuration **Pare-feu et réseaux virtuels** dans la page de votre compte de stockage. Pour plus d’informations sur les pare-feu de stockage, consultez [Configurer des pare-feu et des réseaux virtuels dans Stockage Azure](../../storage/common/storage-network-security.md).

### <a name="other-issues-to-check"></a>Autres problèmes à vérifier 

* La ressource de stockage à laquelle vous accédez est Azure Data Lake Storage Gen2 et se trouve derrière un pare-feu et un réseau virtuel (avec un point de terminaison privé de stockage configuré) en même temps.
* La ressource conteneur à laquelle vous accédez a été supprimée ou n’existe pas.


## <a name="next-steps"></a>Étapes suivantes
Si les étapes précédentes ne permettent pas de résoudre le problème, [créez un ticket de support](../../sql-data-warehouse/sql-data-warehouse-get-started-create-support-ticket.md).
