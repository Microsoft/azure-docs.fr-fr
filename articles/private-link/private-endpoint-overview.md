---
title: Qu’est-ce qu’Azure Private Endpoint ?
description: En savoir plus sur Azure Private Endpoint
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: allensu
ms.openlocfilehash: 8021d659c144bfb68c2714f1680b6ad27a51b56a
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95522343"
---
# <a name="what-is-azure-private-endpoint"></a>Qu’est-ce qu’Azure Private Endpoint ?

Azure Private Endpoint est une interface réseau qui vous permet de vous connecter de façon privée et sécurisée à un service basé sur Azure Private Link. Private Endpoint utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Le service peut être un service Azure, tel qu’Azure Storage, Azure Cosmos DB, SQL, etc. ou votre propre [service Private Link](private-link-service-overview.md).
  
## <a name="private-endpoint-properties"></a>Propriétés de Private Endpoint 
 Private Endpoint spécifie les propriétés suivantes : 


|Propriété  |Description |
|---------|---------|
|Nom    |    Nom unique dans le groupe de ressources.      |
|Subnet    |  Le sous-réseau pour déployer et allouer des adresses IP privées à partir d’un réseau virtuel. Pour connaître la configuration requise pour le sous-réseau, consultez la section Limitations de cet article.         |
|Ressource Private Link    |   Ressource Private Link à connecter avec l’ID de ressource ou l’alias dans la liste des types disponibles. Un identificateur réseau unique sera généré pour tout le trafic envoyé à cette ressource.       |
|Sous-ressource cible   |      Sous-ressource à connecter. Chaque type de ressource Private Link a différentes options à sélectionner en fonction de vos préférences.    |
|Méthode d’approbation de la connexion    |  Automatique ou manuel. Selon les autorisations de contrôle d’accès Azure en fonction du rôle (Azure RBAC), votre point de terminaison privé peut être approuvé automatiquement. Si vous essayez de vous connecter à une ressource Private Link sans Azure RBAC, utilisez la méthode manuelle pour permettre au propriétaire de la ressource d’approuver la connexion.        |
|Message de requête     |  Vous pouvez spécifier un message pour que les connexions demandées soient approuvées manuellement. Ce message peut être utilisé pour identifier une demande spécifique.        |
|État de la connexion   |   Propriété en lecture seule qui spécifie si Private Endpoint est actif. Seules les instances Private Endpoint dans un état approuvé peuvent être utilisées pour envoyer du trafic. États supplémentaires disponibles : <br>-**Approuvé** : La connexion a été approuvée automatiquement ou manuellement et est prête à être utilisée.</br><br>-**En attente** : La connexion a été créée manuellement et est en attente d’approbation par le propriétaire de la ressource Private Link.</br><br>-**Rejeté** : La connexion a été rejetée par le propriétaire de la ressource Private Link.</br><br>-**Déconnecté** : La connexion a été supprimée par le propriétaire de la ressource Private Link. Private Endpoint devient informatif et doit être supprimé pour le nettoyage. </br>|

Voici quelques détails clés sur Private Endpoint : 
- Private Endpoint permet la connectivité entre consommateurs à partir du même réseau virtuel, des réseaux virtuels avec homologation globale et locaux l’aide d’un [VPN](https://azure.microsoft.com/services/vpn-gateway/) ou [d’Express Route](https://azure.microsoft.com/services/expressroute/) et des services basés sur Private Link.
 
- Les connexions réseau ne peuvent être initiées que par des clients se connectant au point de terminaison privé ; les fournisseurs de services n’ont pas de configuration de routage pour initier des connexions aux consommateurs du service. Les connexions peuvent uniquement être établies dans une seule direction.

- Lors de la création d’un point de terminaison privé, une interface réseau en lecture seule est également créée pour le cycle de vie de la ressource. Des adresses IP privées sont attribuées dynamiquement à l’interface à partir du sous-réseau qui est mappé à la ressource de liaison privée. La valeur de l’adresse IP privée reste inchangée pour l’intégralité du cycle de vie du point de terminaison privé.
 
- Private Endpoint doit être déployé dans la même région que le réseau virtuel. 
 
- La ressource Private Link peut être déployée dans une autre région que le réseau virtuel et Private Endpoint.
 
- Vous pouvez créer plusieurs instances Private Endpoint à l’aide de la même ressource Private Link. Pour un réseau unique qui utilise une configuration de serveur DNS commune, il est recommandé d’utiliser Private Endpoint pour une ressource Private Link donnée afin d’éviter les entrées en double ou les conflits dans la résolution DNS. 
 
- Plusieurs instances Private Endpoint privées peuvent être créées sur des sous-réseaux identiques ou différents au sein du même réseau virtuel. Il existe des limites au nombre d’instances Private Endpoint que vous pouvez créer dans un abonnement. Pour plus d’informations, consultez  [Limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

- L’abonnement de la ressource de lien privé doit également être inscrit auprès du fournisseur de ressources Microsoft Network. Pour plus d’informations, consultez  [Fournisseurs de ressources Azure](../azure-resource-manager/management/resource-providers-and-types.md).

 
## <a name="private-link-resource"></a>Ressource Private Link 
Une ressource Private Link est la cible de destination d’une instance Private Endpoint donnée. La liste suivante répertorie les types de ressources Private Link disponibles : 
 
|Nom de la ressource Private Link  |Type de ressource   |Sous-ressources  |
|---------|---------|---------|
|**Service Private Link** (votre propre service)   |  Microsoft.Network/privateLinkServices       | empty |
|**Azure Automation** |  Microsoft.Automation/automationAccounts | Webhook, DSCAndHybridWorker |
|**Azure SQL Database** | Microsoft.Sql/servers    |  Sql Server (sqlServer)        |
|**Azure Synapse Analytics** | Microsoft.Sql/servers    |  Sql Server (sqlServer)        | 
|**Stockage Azure**  | Microsoft.Storage/storageAccounts    |  Blob (blob, blob_secondary)<BR> Table (table, table_secondary)<BR> File d'attente (queue, queue_secondary)<BR> Fichier (file, file_secondary)<BR> Web (web, web_secondary)        |
|**Azure Data Lake Storage Gen2**  | Microsoft.Storage/storageAccounts    |  Blob (blob, blob_secondary)<BR> Système de fichiers Data Lake Gen2 (dfs, dfs_secondary)       |
|**Azure Cosmos DB** | Microsoft.AzureCosmosDB/databaseAccounts    | Sql, MongoDB, Cassandra, Gremlin, Table|
|**Azure Database pour PostgreSQL – Serveur unique** | Microsoft.DBforPostgreSQL/servers    | postgresqlServer |
|**Azure Database pour MySQL** | Microsoft.DBforMySQL/servers    | mysqlServer |
|**Azure Database for MariaDB** | Microsoft.DBforMariaDB/servers    | mariadbServer |
|**Azure IoT Hub** | Microsoft.Devices/IotHubs    | iotHub |
|**Azure Key Vault** | Microsoft.KeyVault/vaults    | coffre |
|**Azure Kubernetes Service – API Kubernetes** | Microsoft.ContainerService/managedClusters    | gestion |
|**Azure Search** | Microsoft.Search/searchService| searchService|  
|**Azure Container Registry** | Microsoft.ContainerRegistry/registries    | Registre |
|**Azure App Configuration** | Microsoft.Appconfiguration/configurationStores    | configurationStores |
|**Azure Backup** | Microsoft.RecoveryServices/vaults    | coffre |
|**Azure Event Hub** | Microsoft.EventHub/namespaces    | espace de noms |
|**Azure Service Bus** | Microsoft.ServiceBus/namespaces | espace de noms |
|**Azure Relay** | Microsoft.Relay/namespaces | espace de noms |
|**Azure Event Grid** | Microsoft.EventGrid/topics    | topic |
|**Azure Event Grid** | Microsoft.EventGrid/domains    | domaine |
|**Azure App Service** | Microsoft.Web/sites    | sites |
|**Azure Machine Learning** | Microsoft.MachineLearningServices/workspaces    | espace de travail |
|**SignalR** | Microsoft.SignalRService/SignalR    | signalR |
|**Azure Monitor** | Microsoft.Insights/privateLinkScopes    | azuremonitor |
|**Cognitive Services** | (Microsoft.CognitiveServices/accounts    | account |
|**Azure File Sync** | Microsoft.StorageSync/storageSyncServices    | Afs |
    
  

  
 
## <a name="network-security-of-private-endpoints"></a>Sécurité réseau de Private Endpoint 
Lorsque vous utilisez Private Endpoint pour les services Azure, le trafic est sécurisé vers une ressource Private Link spécifique. La plateforme effectue un contrôle d’accès pour valider les connexions réseau qui atteignent uniquement la ressource Private Link spécifiée. Pour accéder à des ressources supplémentaires au sein du même service Azure, des instances Private Endpoint supplémentaires sont requises. 
 
Vous pouvez verrouiller complètement vos charges de travail d’accès aux points de terminaison publics pour vous connecter à un service Azure pris en charge. Ce contrôle fournit une couche de sécurité réseau supplémentaire à vos ressources en fournissant une protection d’exfiltration intégrée qui empêche l’accès aux autres ressources hébergées sur le même service Azure. 
 
## <a name="access-to-a-private-link-resource-using-approval-workflow"></a>Accès à une ressource Private Link à l’aide du flux de travail d’approbation 
Vous pouvez vous connecter à une ressource Private Link à l’aide des méthodes d’approbation de connexion suivantes :
- Approuvé **automatiquement** lorsque vous êtes propriétaire de la ressource Private Link spécifique ou que vous en avez l’autorisation. L’autorisation requise est basée sur le type de ressource Private Link au format suivant : Microsoft.\<Provider>/<resource_type>/privateEndpointConnectionApproval/action
- Demande **manuelle** lorsque vous n’avez pas l’autorisation requise et que vous souhaitez demander l’accès. Un workflow d’approbation est lancé. Private Endpoint et la connexion Private Endpoint correspondante sont créés dans un état « En attente ». Le propriétaire de la ressource Private Link est responsable de l’approbation de la connexion. Une fois approuvé, Private Endpoint est activé pour envoyer le trafic normalement, comme indiqué dans le diagramme de flux de travail d’approbation suivant.  

![workflow d’approbation](media/private-endpoint-overview/private-link-paas-workflow.png)
 
Le propriétaire de la ressource Private Link peut effectuer les actions suivantes sur une connexion Private Endpoint : 
- Passez en revue tous les détails des connexions Private Endpoint. 
- Approuvez une connexion Private Endpoint. Le Private Endpoint correspondant sera activé pour envoyer le trafic vers la ressource Private Link. 
- Rejetez une connexion Private Endpoint. L’instance Private Endpoint correspondante sera mise à jour pour refléter cet état.
- Supprimez une connexion Private Endpoint dans n’importe quel état. L’instance Private Endpoint privée correspondante sera mise à jour avec un état Déconnecté pour refléter l’action ; le propriétaire de Private Endpoint peut uniquement supprimer la ressource à ce stade. 
 
> [!NOTE]
> Seule une instance Private Endpoint dans un état approuvé peut envoyer le trafic vers une ressource Private Link donnée. 

### <a name="connecting-using-alias"></a>Connexion à l’aide de l’alias
Alias est un moniker unique qui est généré lorsque le propriétaire du service crée le service Private Link derrière un Standard Load Balancer. Le propriétaire du service peut partager cet alias avec ses consommateurs hors connexion. Les consommateurs peuvent demander une connexion au service Private Link à l’aide de l’URI de ressource ou de l’alias. Si vous souhaitez vous connecter à l’aide de l’alias, vous devez créer une instance Private Endpoint à l’aide de la méthode d’approbation de connexion manuelle. Pour utiliser la méthode d’approbation de connexion manuelle, affectez la valeur true au paramètre de demande manuelle au cours de la création de votre Private Endpoint. Étudiez [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint?view=azps-2.6.0) et [az network private-endpoint create](/cli/azure/network/private-endpoint?view=azure-cli-latest#az-network-private-endpoint-create) pour plus de détails. 

## <a name="dns-configuration"></a>Configuration DNS 
Lors de la connexion à une ressource Private Link à l’aide d’un nom de domaine complet (FQDN) dans la chaîne de connexion, il est important de configurer correctement vos paramètres DNS pour résoudre l’adresse IP privée allouée. Les services Azure existants peuvent déjà avoir une configuration DNS à utiliser lors de la connexion à un point de terminaison public. Cette valeur doit être remplacée pour la connexion à l’aide de votre instance Private Endpoint. 
 
L’interface réseau associée à l’instance Private Endpoint contient l’ensemble complet des informations requises pour configurer votre DNS, y compris le nom de domaine complet et les adresses IP privées allouées pour une ressource Private Link donnée. 

Pour obtenir des informations détaillées et complètes sur les bonnes pratiques et des recommandations relatives à la configuration du système DNS pour les points de terminaison privés, consultez l’[article sur la configuration DNS des points de terminaison privés](private-endpoint-dns.md).



 
## <a name="limitations"></a>Limites
 
Le tableau suivant répertorie les limitations connues lors de l’utilisation de Private Endpoint : 


|Limitation |Description |Limitation des risques  |
|---------|---------|---------|
|Les règles de groupe de sécurité réseau et les routes définies par l’utilisateur ne s’appliquent pas au point de terminaison privé.    |Le groupe de sécurité réseau n’est pas pris en charge sur Private Endpoint. Si les sous-réseaux contenant Private Endpoint peuvent être associés à un groupe de sécurité réseau, les règles ne sont pas effectives sur le trafic traité par Private Endpoint. Vous devez [désactiver l’application des stratégies réseau](disable-private-endpoint-network-policy.md) pour déployer Private Endpoint dans un sous-réseau. Le groupe de sécurité réseau est toujours appliqué sur les autres charges de travail hébergées sur le même sous-réseau. Les routes sur un sous-réseau client, quel qu’il soit, utiliseront un préfixe /32. La modification du comportement de routage par défaut nécessite une UDR similaire.  | Contrôlez le trafic à l’aide de règles de groupe de sécurité réseau pour le trafic sortant sur les clients source. Déployez des routes individuelles avec le préfixe /32 pour remplacer les routes de points de terminaison privés. Les journaux de trafic NSG et les informations de supervision pour les connexions sortantes sont toujours pris en charge et peuvent être utilisés.        |


## <a name="next-steps"></a>Étapes suivantes
- [Créer un point de terminaison privé pour SQL Database à l’aide du portail](create-private-endpoint-portal.md)
- [Créer un point de terminaison privé pour SQL Database à l’aide de PowerShell](create-private-endpoint-powershell.md)
- [Créer un point de terminaison privé pour SQL Database à l’aide de l’interface CLI](create-private-endpoint-cli.md)
- [Créer un point de terminaison privé pour un compte de stockage à l’aide du portail](./tutorial-private-endpoint-storage-portal.md)
- [Créer un point de terminaison privé pour un compte Cosmos Azure à l’aide du portail](../cosmos-db/how-to-configure-private-endpoints.md)
- [Créer votre propre service Liaison privée à l’aide d’Azure PowerShell](create-private-link-service-powershell.md)
- [Créer votre propre liaison privée pour Azure Database pour PostgreSQL – Serveur unique à l’aide du portail](../postgresql/howto-configure-privatelink-portal.md)
- [Créer votre propre liaison privée pour Azure Database pour PostgreSQL – Serveur unique à l’aide de l’interface CLI](../postgresql/howto-configure-privatelink-cli.md)
- [Créer votre propre liaison privée pour Azure Database pour MySQL à l’aide du portail](../mysql/howto-configure-privatelink-portal.md)
- [Créer votre propre liaison privée pour Azure Database pour MySQL à l’aide de l’interface CLI](../mysql/howto-configure-privatelink-cli.md)
- [Créer votre propre liaison privée pour Azure Database for MariaDB à l’aide du portail](../mariadb/howto-configure-privatelink-portal.md)
- [Créer votre propre liaison privée pour Azure Database for MariaDB à l’aide de l’interface CLI](../mariadb/howto-configure-privatelink-cli.md)
- [Créer votre propre liaison privée pour Azure Key Vault à l’aide du portail et de l’interface CLI](../key-vault/general/private-link-service.md)