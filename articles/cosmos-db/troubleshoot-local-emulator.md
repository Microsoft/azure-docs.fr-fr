---
title: Résoudre les problèmes liés à l’utilisation de l’émulateur Azure Cosmos DB
description: Découvrez comment résoudre les problèmes de service non disponible, de certificat, de chiffrement et de gestion de versions liés à l’utilisation de l’émulateur Azure Cosmos DB.
ms.service: cosmos-db
ms.topic: troubleshooting
author: markjbrown
ms.author: mjbrown
ms.date: 09/17/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: bb07adafbc68ff1e9b7d3ee49bb7631dc4395d77
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033457"
---
# <a name="troubleshoot-issues-when-using-the-azure-cosmos-db-emulator"></a>Résoudre les problèmes liés à l’utilisation de l’émulateur Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

L’émulateur Azure Cosmos DB fournit un environnement local qui émule le service Azure Cosmos DB à des fins de développement. Suivez les conseils fournis dans cet article pour résoudre les problèmes rencontrés durant l’installation ou l’utilisation de l’émulateur Azure Cosmos DB. 

Si vous avez installé une nouvelle version de l’émulateur et si vous rencontrez des erreurs, réinitialisez vos données. Pour réinitialiser vos données, cliquez avec le bouton droit sur l’icône de l’émulateur Azure Cosmos DB dans la zone de notification, puis cliquez sur Réinitialiser les données. Si les erreurs persistent, désinstallez l'émulateur et toute autre version ancienne de celui-ci, le cas échéant, puis supprimez le répertoire *C:\Program files\Azure Cosmos DB Emulator* et réinstallez l'émulateur. Pour obtenir des instructions, consultez [Désinstaller l’émulateur local](local-emulator.md#uninstall). Sinon, si la réinitialisation des données ne fonctionne pas, accédez à l'emplacement `%LOCALAPPDATA%\CosmosDBEmulator` et supprimez le dossier.

## <a name="troubleshoot-corrupted-windows-performance-counters"></a>Résoudre les problèmes liés aux compteurs de performances Windows endommagés

* Si vous êtes confronté à un plantage de l’émulateur Azure Cosmos DB, collectez les fichiers d’image mémoire du dossier `%LOCALAPPDATA%\CrashDumps`, compressez-les, puis ouvrez un ticket de support à partir du [portail Azure](https://portal.azure.com).

* Si des problèmes de plantage se produisent dans `Microsoft.Azure.Cosmos.ComputeServiceStartupEntryPoint.exe`, cela peut être le symptôme d’un état endommagé d’un compteur de performances. Généralement, l’exécution de la commande suivante à partir d’une invite de commandes administrateur résout le problème :

  ```cmd
  lodctr /R
   ```

## <a name="troubleshoot-connectivity-issues"></a>Résoudre les problèmes de connectivité

* Si vous rencontrez un problème de connectivité, [collectez les fichiers de trace](#trace-files), compressez-les et ouvrez un ticket de support à partir du [portail Azure](https://portal.azure.com).

* Si vous recevez un message **Service indisponible**, il se peut que l’émulateur n’arrive pas à initialiser la pile réseau. Vérifiez si les clients Pulse Secure ou Juniper Networks sont installés, car leurs pilotes de filtre réseau peuvent être à l’origine du problème. La désinstallation des pilotes de filtre de réseau tiers permet généralement de résoudre le problème. Vous pouvez également démarrer l’émulateur avec l’option /DisableRIO, pour passer la communication réseau de l’émulateur à Winsock standard. 

* Si vous rencontrez des problèmes de connectivité indiquant **« Interdit »,« message »:« La requête est effectuée avec un chiffrement ou un protocole de chiffrement en transit interdit. Vérifiez le paramètre de protocole minimum autorisé SSL/TLS… »** , cela peut être dû à des changements globaux dans le système d’exploitation (par exemple, Insider Preview Build 20170) ou dans les paramètres du navigateur qui activent TLS 1.3 par défaut. Une erreur similaire peut se produire lors de l’utilisation du Kit de développement logiciel (SDK) pour exécuter une requête sur l’émulateur Cosmos, par exemple **Microsoft.Azure.Documents.DocumentClientException: La requête est effectuée avec un chiffrement ou un protocole de chiffrement en transit interdit. Vérifiez le paramètre de protocole minimum autorisé SSL/TLS**. Ceci est en attente pour le moment, car l’émulateur Cosmos fonctionne uniquement avec le protocole TLS 1.2. La solution recommandée consiste à modifier les paramètres et à utiliser par défaut TLS 1.2 ; par exemple, dans IIS Manager, accédez à « Sites » -> « Sites web par défaut » et recherchez les « Liaisons de site » pour le port 8081, puis modifiez-les pour désactiver TLS 1.3. Une opération similaire peut être effectuée pour le navigateur web via les options « Paramètres ».

* Lorsque l’émulateur est en cours d’exécution, si votre ordinateur se met en mode veille ou exécute une mise à jour du système d’exploitation, le message **Le service est actuellement indisponible** peut s’afficher. Réinitialisez les données de l’émulateur en cliquant avec le bouton droit sur l’icône qui apparaît dans la zone de notification Windows, puis en sélectionnant **Réinitialiser les données**.

## <a name="collect-trace-files"></a><a id="trace-files"></a>Collecter les fichiers de trace

Pour collecter des traces de débogage, exécutez les commandes suivantes à partir d’une invite de commande d’administration :

1. Accédez à l'emplacement où l'émulateur est installé :

   ```bash
   cd /d "%ProgramFiles%\Azure Cosmos DB Emulator"
   ```

1. Arrêtez l'émulateur et jetez un coup d'œil à la barre d'état système pour vérifier que le programme s'est arrêté. L'arrêt peut prendre une minute. Vous pouvez également sélectionner **Quitter** dans l’interface utilisateur de l’émulateur Azure Cosmos DB.

   ```bash
   Microsoft.Azure.Cosmos.Emulator.exe /shutdown
   ```

1. Lancez la journalisation à l'aide de la commande suivante :

   ```bash
   Microsoft.Azure.Cosmos.Emulator.exe /startwprtraces
   ```

1. Lancez l'émulateur.

   ```bash
   Microsoft.Azure.Cosmos.Emulator.exe
   ```

1. Reproduisez le problème. Si l'explorateur de données ne fonctionne pas, patientez quelques secondes jusqu'à ce que le navigateur s'ouvre afin d'intercepter l'erreur.

1. Arrêtez la journalisation à l'aide de la commande suivante :

   ```bash
   Microsoft.Azure.Cosmos.Emulator.exe /stopwprtraces
   ```
   
1. Accédez à l'emplacement `%ProgramFiles%\Azure Cosmos DB Emulator` et recherchez le fichier *docdbemulator_000001.etl*.

1. Ouvrez un ticket de support dans le [portail Azure](https://portal.azure.com) et incluez le fichier. etl avec les étapes de reproduction.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à déboguer les problèmes liés à l'émulateur local. Vous pouvez maintenant passer aux articles suivants :

* [Exporter les certificats de l’émulateur Azure Cosmos DB pour une utilisation avec des applications Java, Python et Node.js](local-emulator-export-ssl-certificates.md)
* [Utiliser des paramètres de ligne de commande et des commandes PowerShell pour contrôler l'émulateur](emulator-command-line-parameters.md)
