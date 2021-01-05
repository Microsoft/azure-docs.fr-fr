---
title: Utilisation de comptes de stockage gérés par le client dans Azure Monitor Log Analytics
description: Utilisez votre propre compte de stockage pour les scénarios de Log Analytics
ms.subservice: logs
ms.topic: conceptual
author: noakup
ms.author: noakuper
ms.date: 09/03/2020
ms.openlocfilehash: f221237bee441ec78d726dabf476d1085a27071d
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095302"
---
# <a name="using-customer-managed-storage-accounts-in-azure-monitor-log-analytics"></a>Utilisation de comptes de stockage gérés par le client dans Azure Monitor Log Analytics

Log Analytics s’appuie sur Stockage Azure dans différents scénarios. Cette utilisation est généralement gérée automatiquement. Toutefois, dans certains cas, vous devez fournir et gérer votre propre compte de stockage, également appelé compte de stockage géré par le client. Ce document détaille l’utilisation du stockage géré par le client pour l’ingestion des journaux WAD/LAD, des scénarios Azure Private Link spécifiques et le chiffrement à l’aide d’une clé gérée par le client (CMK). 

> [!NOTE]
> Nous vous recommandons de ne pas dépendre du contenu que Log Analytics charge sur le stockage géré par le client, étant donné que le formatage et le contenu peuvent changer.

## <a name="ingesting-azure-diagnostics-extension-logs-wadlad"></a>Ingestion des journaux de l’extension Azure Diagnostics (WAD/LAD)
Les agents de l’extension Azure Diagnostics (également appelés WAD et LAD pour les agents Windows et Linux, respectivement) recueillent différents journaux du système d’exploitation et les stockent sur un compte de stockage géré par le client. Vous pouvez ensuite ingérer ces journaux dans Log Analytics pour les examiner et les analyser.
Pour collecter les journaux de l’extension Azure Diagnostics à partir de votre compte de stockage, connectez le compte de stockage à votre espace de travail Log Analytics en tant que source de données de stockage à l’aide du [portail Azure](./diagnostics-extension-logs.md#collect-logs-from-azure-storage) ou en appelant l’[API Storage Insights](/rest/api/loganalytics/storage%20insights/createorupdate).

Types de données pris en charge :
* syslog
* Événements Windows
* Service Fabric
* Événements ETW
* Journaux d’activité IIS

## <a name="using-private-links"></a>Utilisation de liaisons privées
Les comptes de stockage gérés par le client sont requis dans certains cas d’usage, notamment lorsque des liaisons privées sont utilisées pour se connecter à des ressources Azure Monitor. L’un de ces cas est l’ingestion de journaux personnalisés ou de journaux IIS. Ces types de données sont d’abord chargés sous forme de blobs sur un compte Stockage Azure intermédiaire, et seulement ensuite ils sont ingérés dans un espace de travail. De même, certaines solutions Azure Monitor peuvent utiliser des comptes de stockage pour stocker des fichiers volumineux, tels que des fichiers de sauvegarde Watson, qui sont utilisés par la solution Azure Security Center. 

##### <a name="private-link-scenarios-that-require-a-customer-managed-storage"></a>Scénarios Azure Private Link qui requièrent un stockage géré par le client
* Ingestion de journaux personnalisés et de journaux IIS
* Solution ASC autorisée à collecter des fichiers de sauvegarde Watson

### <a name="how-to-use-a-customer-managed-storage-account-over-a-private-link"></a>Comment utiliser un compte de stockage géré par le client via une liaison privée
##### <a name="workspace-requirements"></a>Conditions requises pour l’espace de travail
Lors de la connexion à Azure Monitor via une liaison privée, les agents Log Analytics peuvent uniquement envoyer des journaux à des espaces de travail qui sont liés à votre réseau via une liaison privée. Cette règle nécessite que vous configuriez correctement un objet Étendue de liaison privée Azure Monitor (AMPLS), que vous le connectiez à vos espaces de travail, puis que vous le connectiez à votre réseau via une liaison privée. Pour plus d’informations sur la procédure de configuration de l’AMPLS, consultez [Utiliser Azure Private Link pour connecter en toute sécurité des réseaux à Azure Monitor](./private-link-security.md). 
##### <a name="storage-account-requirements"></a>Conditions requises pour le compte de stockage
Pour que le compte de stockage se connecte correctement à votre liaison privée, il doit :
* se trouver sur votre réseau virtuel ou sur un réseau appairé et être connecté à votre réseau virtuel via une liaison privée. Cela permet aux agents de votre réseau virtuel d’envoyer des journaux au compte de stockage ;
* se trouver dans la même région que l’espace de travail auquel il est lié ;
* autoriser Azure Monitor à accéder au compte de stockage. Si vous avez choisi de n’autoriser que certains réseaux à accéder à votre compte de stockage, veillez à autoriser cette exception : « Autoriser les services Microsoft approuvés à accéder à ce compte de stockage ». Cela permet à Log Analytics de lire les journaux ingérés dans ce compte de stockage ;
* si votre espace de travail traite également du trafic provenant d’autres réseaux, vous devez configurer le compte de stockage de manière à autoriser le trafic entrant provenant des réseaux concernés ou d’Internet.

##### <a name="link-your-storage-account-to-a-log-analytics-workspace"></a>Lier votre compte de stockage à un espace de travail Log Analytics
Vous pouvez lier votre compte de stockage à l’espace de travail via l’[interface de ligne de commande Azure](/cli/azure/monitor/log-analytics/workspace/linked-storage) ou l’[API REST](/rest/api/loganalytics/linkedstorageaccounts). Valeurs dataSourceType applicables :
* CustomLogs : utilise le stockage pour les journaux personnalisés et les journaux IIS pendant l’ingestion.
* AzureWatson : utilise le stockage pour les fichiers de sauvegarde Watson chargés par la solution ASC (Azure Security Center). Pour plus d’informations sur la gestion de la conservation des données, le remplacement d’un compte de stockage lié et la surveillance de l’activité sur votre compte de stockage, consultez la section [Gestion des comptes de stockage liés](#managing-linked-storage-accounts). 

## <a name="encrypting-data-with-cmk"></a>Chiffrement des données avec une clé gérée par le client
Stockage Azure chiffre toutes les données au repos dans un compte de stockage. Par défaut, le service chiffre les données avec des clés gérées par Microsoft (MMK). Toutefois, Stockage Azure vous permet d’utiliser une clé gérée par le client (CMK) à partir d’Azure Key Vault pour chiffrer vos données de stockage. Vous pouvez importer vos propres clés dans Azure Key Vault ou utiliser les API d’Azure Key Vault pour générer des clés.
##### <a name="cmk-scenarios-that-require-a-customer-managed-storage-account"></a>Scénarios CMK qui requièrent un compte de stockage géré par le client
* Chiffrement des requêtes d’alerte de journal avec une clé gérée par le client
* Chiffrement des requêtes enregistrées avec une clé gérée par le client

### <a name="how-to-apply-cmk-to-customer-managed-storage-accounts"></a>Comment appliquer une clé gérée par le client à des comptes de stockage gérés par le client
##### <a name="storage-account-requirements"></a>Conditions requises pour le compte de stockage
Le compte de stockage et le coffre de clés doivent se trouver dans la même région, mais ils peuvent appartenir à des abonnements différents. Pour plus d’informations sur le chiffrement et la gestion des clés dans le stockage Azure, consultez [Chiffrement du stockage Azure pour les données au repos](../../storage/common/storage-service-encryption.md).

##### <a name="apply-cmk-to-your-storage-accounts"></a>Appliquer une clé gérée par le client à vos comptes de stockage
Pour configurer votre compte Stockage Azure de façon à utiliser des clés gérées par le client avec Azure Key Vault, utilisez le [portail Azure](../../storage/common/customer-managed-keys-configure-key-vault.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json), [PowerShell](../../storage/common/customer-managed-keys-configure-key-vault.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) ou l’[interface CLI](../../storage/common/customer-managed-keys-configure-key-vault.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json). 

## <a name="managing-linked-storage-accounts"></a>Gestion des comptes de stockage liés

Pour lier des comptes de stockage à votre espace de travail ou les en dissocier, utilisez l’[interface de ligne de commande Azure](/cli/azure/monitor/log-analytics/workspace/linked-storage) ou l’[API REST](/rest/api/loganalytics/linkedstorageaccounts).

##### <a name="create-or-modify-a-link"></a>Créer ou modifier une liaison
Lorsque vous liez un compte de stockage à un espace de travail, Log Analytics commence à l’utiliser au lieu du compte de stockage détenu par le service. Vous pouvez : 
* inscrire plusieurs comptes de stockage pour répartir la charge des journaux entre eux ;
* réutiliser le même compte de stockage pour plusieurs espaces de travail.

##### <a name="unlink-a-storage-account"></a>Dissocier un compte de stockage
Pour cesser d’utiliser un compte de stockage, dissociez le stockage de l’espace de travail. La dissociation de tous les comptes de stockage d’un espace de travail signifie que Log Analytics tentera de se tourner vers les comptes de stockage gérés par le service. Si votre réseau dispose d’un accès limité à Internet, ces stockages peuvent être indisponibles et tout scénario qui repose sur le stockage échouera.

##### <a name="replace-a-storage-account"></a>Remplacer un compte de stockage
Pour remplacer un compte de stockage utilisé pour l’ingestion :
1.  **Créez une liaison vers un nouveau compte de stockage.** Les agents de journalisation recevront la configuration mise à jour et commenceront également à envoyer des données vers le nouveau stockage. Le processus peut prendre quelques minutes.
2.  **Ensuite, dissociez l’ancien compte de stockage pour que les agents cessent d’écrire dans le compte supprimé.** Le processus d’ingestion continue de lire les données de ce compte jusqu’à ce qu’elles soient toutes ingérées. Ne supprimez pas le compte de stockage tant que tous les journaux n’ont pas été ingérés.

### <a name="maintaining-storage-accounts"></a>Conservation des comptes de stockage
##### <a name="manage-log-retention"></a>Gérer la rétention des journaux
Lorsque vous utilisez votre propre compte de stockage, la rétention est laissée à votre discrétion. En d’autres termes, Log Analytics ne supprime pas les journaux stockés sur votre stockage privé. Au lieu de cela, vous devez configurer une stratégie pour gérer la charge en fonction de vos préférences.

##### <a name="consider-load"></a>Prendre en compte la charge
Les comptes de stockage peuvent gérer une certaine charge de requêtes de lecture et d’écriture avant de commencer à limiter les requêtes (pour plus d’informations, consultez [Cibles de scalabilité et de performances pour les comptes de stockage standard](../../storage/common/scalability-targets-standard-account.md)). La limitation a une incidence sur le temps nécessaire à l’ingestion des journaux. Si votre compte de stockage est surchargé, inscrivez un compte de stockage supplémentaire afin de répartir la charge. Pour surveiller la capacité et le niveau de performance de votre compte de stockage, consultez ses [insights dans le portail Azure]( https://docs.microsoft.com/azure/azure-monitor/insights/storage-insights-overview).

### <a name="related-charges"></a>Frais connexes
Les comptes de stockage sont facturés en fonction du volume de données stockées, du type de stockage et du type de redondance. Pour plus d’informations, consultez [Tarification des objets blob de blocs](https://azure.microsoft.com/pricing/details/storage/blobs) et [Tarification de Stockage Table](https://azure.microsoft.com/pricing/details/storage/tables).


## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’[utilisation d’Azure Private Link pour connecter en toute sécurité des réseaux à Azure Monitor](private-link-security.md)
- En savoir plus sur les [clés gérées par le client dans Azure Monitor](customer-managed-keys.md)
