---
title: Versions de registre, mise à jour corrective et mise à niveau d’Azure Blockchain Service
description: Vue d’ensemble des versions de registres prises en charge dans Azure Blockchain Service. Y compris les stratégies de mise à jour corrective et de mise à niveau des systèmes.
ms.date: 06/30/2020
ms.topic: conceptual
ms.reviewer: ravastra
ms.openlocfilehash: ea7c54e229178320329204a3199ab2b4c44058fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85807738"
---
# <a name="supported-azure-blockchain-service-ledger-versions"></a>Versions de registre d’Azure Blockchain Service prises en charge

Azure Blockchain Service utilise le registre [Quorum](https://www.goquorum.com/developers) basé sur Ethereum et conçu pour le traitement des transactions privées au sein d’un groupe de participants connus, identifié en tant que consortium dans Azure Blockchain Service.

Azure Blockchain Service prend en charge [Quorum version 2.6.0](https://github.com/jpmorganchase/quorum/releases/tag/v2.6.0) et le [gestionnaire de transactions Tessera](https://github.com/jpmorganchase/tessera).

## <a name="managing-updates-and-upgrades"></a>Gestion des mises à jour et des mises à niveau

La gestion de versions dans Quorum s’effectue via les versions majeure, mineure et de correctifs. Par exemple, si la version de Quorum est 2.0.1, le type de version est catégorisé comme suit :

|Majeure | Secondaire  | Correctif  |
| :--- | :----- | :----- |
| 2 | 0 | 1 | 

Azure Blockchain Service met à jour automatiquement les versions de correctifs de Quorum sur les membres en cours d’exécution dans les 30 jours de leur mise à disposition à partir de Quorum.

## <a name="availability-of-new-ledger-versions"></a>Disponibilité des nouvelles versions de registre

Azure Blockchain Service fournit les dernières versions majeure et mineure du registre Quorum dans les 60 jours suivant leur mise à disposition par le fabricant de Quorum. Un maximum de quatre versions mineures sont fournies au choix aux consortiums lors du provisionnement d’un nouveau membre et d’un consortium. La mise à niveau de/vers une version majeure ou mineure n’est actuellement pas prise en charge. Par exemple, si vous exécutez une version 2.x, il n’est pas possible de la mettre à niveau vers une version 3.x pour le moment. De même, si vous exécutez la version 2.2, il n’est pas possible de la mettre à niveau vers la version 2.3 pour le moment.

## <a name="how-to-check-quorum-ledger-version"></a>Vérifier la version du registre Quorum

Vous pouvez vérifier la version de Quorum de votre membre Azure Blockchain Service en créant une liaison avec votre nœud à l’aide de geth ou en consultant les journaux de diagnostic.

### <a name="using-geth"></a>Utilisation de geth

Créez une liaison avec votre nœud Azure Blockchain Service à l’aide de geth. Par exemple : `geth attach https://myblockchainmember.blockchain.azure.com:3200/<Access key>`.

Une fois votre nœud connecté, geth signale la version de Quorum, comme dans la sortie suivante :

``` text
instance: Geth/v1.9.7-stable-9339be03(quorum-v2.6.0)/linux-amd64/go1.13.12
```

Pour plus d’informations sur l’utilisation de geth, consultez [Démarrage rapide : Utiliser Geth pour effectuer une jonction à un nœud de transaction Azure Blockchain Service](connect-geth.md).

### <a name="using-diagnostic-logs"></a>Utilisation des journaux de diagnostic

Si vous activez les journaux de diagnostic, la version de Quorum est indiquée pour les nœuds de transaction. Par exemple, le message du journal d’information du nœud suivant comprend la version de Quorum.

``` text 
{"NodeName":"transaction-node","Message":"INFO [06-22|05:31:45.156] Starting peer-to-peer node instance=Geth/v1.9.7-stable-9339be03(quorum-v2.6.0)/linux-amd64/go1.13.12\n"}
{"NodeName":"transaction-node","Message":"[*] Starting Quorum node with QUORUM_VERSION=2.6.0, TESSERA_VERSION=0.10.5 and PRIVATE_CONFIG=/working-dir/c/tm.ipc\n"}
111
```

Pour plus d’informations sur les journaux de diagnostic, consultez [Superviser Azure Blockchain Service avec Azure Monitor](monitor-azure-blockchain-service.md#diagnostic-settings).

## <a name="how-to-check-genesis-file-content"></a>Vérifier le contenu d’un fichier Genesis

Pour vérifier le contenu du fichier Genesis de votre nœud blockchain, vous pouvez utiliser l’API JavaScript Ethereum suivante :

``` bash
admin.nodeInfo.protocols
```
Vous pouvez appeler l’API à l’aide d’une console geth ou d’une bibliothèque web3. Pour plus d’informations sur l’utilisation de geth, consultez [Démarrage rapide : Utiliser Geth pour effectuer une jonction à un nœud de transaction Azure Blockchain Service](connect-geth.md).

## <a name="next-steps"></a>Étapes suivantes

[Limites dans Azure Blockchain Service](limits.md)
