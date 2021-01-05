---
title: Nouveautés Notes de publication - Azure Blockchain Service
description: Découvrez les nouveautés Azure Blockchain Service, comme les dernières notes de publication, les versions, les problèmes connus et les modifications à venir.
ms.date: 06/30/2020
ms.topic: conceptual
ms.reviewer: ravastra
ms.openlocfilehash: 6a3113a2d28e704b188d701da13493ecd8263cab
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94335024"
---
# <a name="whats-new-in-azure-blockchain-service"></a>Nouveautés Azure Blockchain Service

> Soyez notifié de la disponibilité des mises à jour sur cette page en faisant un copier-coller de cette URL : `https://docs.microsoft.com/api/search/rss?search=%22Release+notes+-+Azure+Blockchain+Service%22&locale=en-us`dans votre lecteur de flux RSS [![Icône du lecteur de flux RSS](./media/whats-new/feed-icon-16x16.png)](/api/search/rss?locale=en-us&search=%2522Release%2bnotes%2b-%2bAzure%2bBlockchain%2bService%2522).

Azure Blockchain Service bénéficie d’améliorations en continu. Pour vous informer des développements les plus récents, cet article détaille les thèmes suivants :

- Nouvelles fonctionnalités
- Mises à niveau de la version
- Problèmes connus

---

## <a name="june-2020"></a>Juin 2020

### <a name="version-upgrades"></a>Mises à niveau de la version

- Mise à niveau de Quorum vers la version 2.6.0. Avec la version 2.6.0, vous pouvez envoyer des transactions privées signées. Pour plus d’informations sur l’envoi de transactions privées, consultez la [documentation relative à l’API Quorum](https://docs.goquorum.consensys.net/en/latest/Reference/APIs/ContractExtensionAPIs/#apis).
- Mise à niveau de Tessera vers la version 0.10.5.

### <a name="contract-size-and-transaction-size-increased-to-128-kb"></a>Taille du contrat et taille de la transaction augmentée à 128 Ko

Tapez : Modification de la configuration

La taille du contrat (MaxCodeSize) a été augmentée à 128 Ko, ce qui vous permet de déployer des contrats intelligents de plus grande taille. En outre, la taille de la transaction (txnSizeLimit) a été augmentée à 128 Ko. Les modifications de configuration s’appliquent aux nouveaux consortiums créés sur Azure Blockchain Service après le 19 juin 2020.

### <a name="trietimeout-value-reduced"></a>Valeur TrieTimeout réduite

Tapez : Modification de la configuration

La valeur TrieTimeout a été réduite afin que l’état en mémoire soit écrit plus fréquemment sur le disque. La valeur inférieure garantit une récupération plus rapide d’un nœud dans le cas rare d’un incident de nœud.

## <a name="may-2020"></a>Mai 2020

### <a name="version-upgrades"></a>Mises à niveau de la version

- Mise à niveau d’Ubuntu vers la version 18.04
- Mise à niveau de Quorum vers la version 2.5.0
- Mise à niveau de Tessera vers la version 0.10.4

### <a name="azure-blockchain-service-supports-sending-rawprivate-transactions"></a>Azure Blockchain Service prend en charge l’envoi de transactions rawPrivate

Tapez : Fonctionnalité

Les clients peuvent signer des transactions privées en dehors du compte, sur le nœud.

### <a name="two-phase-member-provisioning"></a>Approvisionnement des membres en deux phases

Tapez : Amélioration

Les deux phases permettent d’optimiser les scénarios dans lesquels un membre est créé dans un long consortium existant. L’infrastructure de membre est approvisionnée lors de la première phase. Pendant la deuxième phase, le membre est synchronisé avec blockchain. L’approvisionnement en deux phases permet d’éviter que la création d’un membre n’échoue en raison du délai d’expiration.

## <a name="known-issues"></a>Problèmes connus

### <a name="ethestimategas-function-throws-exception-in-quorum-v260"></a>La fonction eth.estimateGas lève une exception dans Quorum v2.6.0

Dans Quorum v2.6.0, appeler la fonction *eth.estimateGas* sans fournir le paramètre de *valeur* supplémentaire entraîne une exception de type *Gestionnaire de méthode en panne*. L’équipe de Quorum a été notifiée et un correctif est attendu à la fin du mois de juillet 2020. Vous pouvez utiliser les solutions de contournement suivantes en attendant qu’un correctif soit disponible :

- Évitez d’utiliser *eth.estimateGas* , car cela peut affecter les performances. Pour plus d’informations sur les problèmes de performances liés à eth.estimateGas, consultez [L’appel de la fonction eth.estimateGas diminue les performances](#calling-ethestimategas-function-reduces-performance). Incluez une valeur de gaz pour chaque transaction. La plupart des bibliothèques appellent la fonction eth.estimateGas si aucune valeur de gaz n’est fournie, ce qui entraîne un indicent sur Quorum 2.6.0.
- Si vous devez appeler *eth.estimateGas* , l’équipe de Quorum suggère de définir le paramètre supplémentaire *valeur* sur *0* comme solution de contournement.

### <a name="mining-stops-if-fewer-than-four-validator-nodes"></a>L’exploration s’arrête s’il y a moins de quatre nœuds validateurs

Les réseaux de production doivent avoir au moins quatre nœuds validateurs. Quorum recommande un minimum de quatre nœuds validateurs pour se conformer à la tolérance de panne d’incident IBFT (3F+1). Vous devez disposer d’au moins deux nœuds Azure Blockchain Service de niveau *Standard* pour obtenir quatre nœuds validateurs. Un nœud standard est approvisionné de deux nœuds validateurs.  

Si le réseau Blockchain sur Azure Blockchain Service ne comporte pas quatre nœuds validateurs, il se peut que l’exploration s’arrête sur le réseau. Il est possible de détecter l’arrêt de l’exploration en définissant une alerte sur les blocs traités. Dans un réseau sain, le bloc traité comportera 60 blocs par nœud toutes les cinq minutes.

Pour réduire les risques, l’équipe Azure Blockchain Service doit redémarrer le nœud. Les clients doivent ouvrir une demande de support pour redémarrer le nœud. L’équipe Azure Blockchain Service travaille à la détection et à la résolution automatiques des problèmes d’exploration.

Utilisez le niveau *Standard* pour les déploiements en production. Utilisez le niveau *De base* pour le développement, les tests et les preuves de concept. Le passage du niveau tarifaire de base au niveau standard (ou inversement) après la création du membre n'est pas pris en charge.

### <a name="blockchain-data-manager-requires-standard-tier-node"></a>Blockchain Data Manager requiert un nœud de niveau standard

Utilisez le niveau *Standard* si vous utilisez Blockchain Data Manager. Le niveau de *base* n’a que 4 Go de mémoire. Par conséquent, il n’est pas possible de mettre à l’échelle jusqu’à l’utilisation requise par Blockchain Data Manager et d’autres services exécutés sur ce dernier.

Utilisez le niveau *De base* pour le développement, les tests et les preuves de concept. Le passage du niveau tarifaire de base au niveau standard (ou inversement) après la création du membre n'est pas pris en charge.

### <a name="large-volume-of-unlock-account-calls-causes-geth-to-crash"></a>Les appels de déverrouillage de compte en masse entraînent un incident sur Geth

Des appels de déverrouillage de compte en masse lors de la soumission d’une transaction peuvent provoquer un incident sur Geth sur un nœud de transaction. Par conséquent, vous devez arrêter l’ingestion des transactions. Dans le cas contraire, la file d’attente des transactions en attente augmente.

Geth redémarre automatiquement en moins d’une minute. Selon le nœud, la synchronisation peut prendre beaucoup de temps. L’équipe Azure Blockchain Service travaille à l’activation d’une fonctionnalité d’archivage qui réduit le temps de synchronisation.

Pour détecter les incidents sur Geth, vous pouvez consulter les journaux pour accéder aux messages d’erreur dans les messages Blockchain au sein des journaux des applications. Vous pouvez également vérifier si les blocs traités diminuent alors que les transactions en attente augmentent.

Pour résoudre ce problème, envoyez des transactions signées au lieu d’envoyer des transactions non signées avec une commande permettant de déverrouiller le compte. Pour les transactions déjà signées en externe, le déverrouillage du compte n’est pas nécessaire.

Si vous souhaitez envoyer des transactions non signées, déverrouillez le compte pour une durée infinie en envoyant 0 en tant que paramètre de temps dans la commande de déverrouillage. Vous pouvez verrouiller à nouveau le compte après l’envoi de toutes les transactions.  

Voici les paramètres de Geth utilisés par Azure Blockchain Service. Ces paramètres peuvent être ajustés.

- Période de blocage d’Istanbul : 5 secondes
- Limite de gaz plancher : 700 000 000
- Limite de gaz plafond : 800 000 000

### <a name="large-volume-of-private-transactions-reduces-performance"></a>Un volume important de transactions privées réduit les performances

Si vous utilisez le niveau de base d’Azure Blockchain Service et les transactions privées d’Azure Blockchain Service, cela peut entraîner un incident sur Tessera.

Vous pouvez détecter l’incident sur Tessera en examinant les journaux des applications Blockchain et en recherchant le message `Tessera crashed. Restarting Tessera...`.

Azure Blockchain Service redémarre Tessera lorsqu’il y a un incident. Le redémarrage prend environ une minute.

Utilisez le niveau *Standard* si vous envoyez un volume élevé de transactions privées. Utilisez le niveau *De base* pour le développement, les tests et les preuves de concept. Le passage du niveau tarifaire de base au niveau standard (ou inversement) après la création du membre n'est pas pris en charge.

### <a name="calling-ethestimategas-function-reduces-performance"></a>L’appel de la fonction eth.estimateGas réduit les performances

Le fait d’appeler la fonction *eth.estimateGas* plusieurs fois réduit considérablement le nombre de transactions par seconde. N’utilisez pas la fonction *eth.estimateGas* à chaque soumission de transaction. La fonction *eth.estimateGas* utilise beaucoup de mémoire.

Si possible, utilisez une valeur de gaz prudente pour soumettre des transactions et n’abusez pas de l’utilisation de *eth.estimateGas*.

### <a name="unbounded-loops-in-smart-contracts-reduces-performance"></a>Les boucles non liées dans les contrats intelligents réduisent les performances

Évitez les boucles non liées dans les contrats intelligents car elles peuvent diminuer les performances. Pour plus d’informations, consultez les ressources suivantes :

- [Éviter les boucles non liées](https://blog.b9lab.com/getting-loopy-with-solidity-1d51794622ad )
- [Meilleures pratiques de sécurité pour les contrats intelligents](https://github.com/ConsenSys/smart-contract-best-practices)
- [Instructions relatives aux contrats intelligents fournies par Quorum](https://docs.goquorum.consensys.net/en/stable/Concepts/Security/Framework/DecentralizedApplication/SmartContractsSecurity/)
- [Instructions concernant les boucles et les limites de gaz fournies par Solidity](https://solidity.readthedocs.io/en/develop/security-considerations.html#gas-limit-and-loops)