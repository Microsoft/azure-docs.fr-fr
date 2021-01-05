---
title: Résoudre les problèmes de latence avec les journaux Storage Analytics
description: Identifiez et résolvez les problèmes de latence avec les journaux Azure Storage Analytics, et optimisez l’application cliente.
author: v-miegge
ms.topic: troubleshooting
ms.author: kartup
manager: dcscontentpm
ms.date: 10/21/2019
ms.service: storage
ms.subservice: common
services: storage
tags: ''
ms.openlocfilehash: 5f04a20b347e2672d9699551885f5dd16ceaa99c
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92785593"
---
# <a name="troubleshoot-latency-using-storage-analytics-logs"></a>Résoudre les problèmes de latence avec les journaux Storage Analytics

Les compétences en diagnostic et en résolution des problèmes sont essentielles pour créer et prendre en charge des applications clientes avec le Stockage Azure.

En raison de la nature distribuée d’une application Azure, diagnostic et résolution des erreurs et des problèmes de performances peuvent être plus complexes que dans les environnements traditionnels.

Les étapes suivantes montrent comment identifier et résoudre les problèmes de latence avec les journaux Azure Storage Analytics, et optimiser l’application cliente.

## <a name="recommended-steps"></a>Étapes recommandées

1. Téléchargez les [journaux Storage Analytics](./storage-analytics-logging.md#download-storage-logging-log-data).

2. Utilisez le script PowerShell suivant pour convertir les journaux du format brut au format tabulaire :

   ```Powershell
   $Columns = 
        (   "version-number",
            "request-start-time",
            "operation-type",
            "request-status",
            "http-status-code",
            "end-to-end-latency-in-ms",
            "server-latency-in-ms",
            "authentication-type",
            "requester-account-name",
            "owner-account-name",
            "service-type",
            "request-url",
            "requested-object-key",
            "request-id-header",
            "operation-count",
            "requester-ip-address",
            "request-version-header",
            "request-header-size",
            "request-packet-size",
            "response-header-size",
            "response-packet-size",
            "request-content-length",
            "request-md5",
            "server-md5",
            "etag-identifier",
            "last-modified-time",
            "conditions-used",
            "user-agent-header",
            "referrer-header",
            "client-request-id"
        )

   $logs = Import-Csv “REPLACE THIS WITH FILE PATH” -Delimiter ";" -Header $Columns

   $logs | Out-GridView -Title "Storage Analytic Log Parser"
   ```

3. Le script lance une fenêtre GUI dans laquelle les informations peuvent être filtrées par colonnes, comme ci-dessous.

   ![Fenêtre de l’analyseur de journaux Storage Analytics](media/troubleshoot-latency-storage-analytics-logs/storage-analytic-log-parser-window.png)
 
4. Affinez les entrées du journal en fonction de « operation-type », puis recherchez celle qui a été créée pendant le laps de temps du problème.

   ![Entrées du journal operation-type](media/troubleshoot-latency-storage-analytics-logs/operation-type.png)

5. Au moment où le problème s’est produit, les valeurs suivantes sont importantes :

   * Operation-type = GetBlob
   * request-status = SASNetworkError
   * End-to-End-Latency-In-Ms = 8453
   * Server-Latency-In-Ms = 391

   La latence de bout en bout se calcule à partir de l’équation suivante :

   * Latence de bout en bout = Latence du serveur + Latence du client

   Calculez la latence du client à l’aide de l’entrée du journal :

   * Latence du client = Latence de bout en bout - Latence du serveur

        Exemple : 8453 – 391 = 8062 ms

   Le tableau suivant fournit des informations sur les résultats OperationType et RequestStatus à latence élevée :

   | Type d’objet blob |RequestStatus =<br>Succès|RequestStatus =<br>(SAS)NetworkError|Recommandation|
   |---|---|---|---|
   |GetBlob|Oui|Non|[**Opération GetBlob :** RequestStatus = Success](#getblob-operation-requeststatus--success)|
   |GetBlob|Non|Oui|[**Opération GetBlob :** RequestStatus = (SAS)NetworkError](#getblob-operation-requeststatus--sasnetworkerror)|
   |PutBlob|Oui|Non|[**Opération Put :** RequestStatus = Success](#put-operation-requeststatus--success)|
   |PutBlob|Non|Oui|[**Opération Put :** RequestStatus = (SAS)NetworkError](#put-operation-requeststatus--sasnetworkerror)|

## <a name="status-results"></a>Résultats de l’état

### <a name="getblob-operation-requeststatus--success"></a>Opération GetBlob : RequestStatus = Success

Vérifiez les valeurs suivantes, comme indiqué à l’étape 5 de la section « Étapes recommandées » :

* Latence de bout en bout
* Latence du serveur
* Latence du client

Dans une **opération GetBlob** avec **RequestStatus = Success** , si le **Temps maximal** est consacré à la **Latence du client** , c’est le signe que le Stockage Azure passe beaucoup de temps à écrire des données sur le client. Ce délai indique un problème côté client.

**Recommandation :**

* Examinez le code de votre client.
* Utilisez Wireshark, l’analyseur de message Microsoft ou TCPing pour enquêter sur les problèmes de connectivité réseau provenant du client. 

### <a name="getblob-operation-requeststatus--sasnetworkerror"></a>Opération GetBlob : RequestStatus = (SAS)NetworkError

Vérifiez les valeurs suivantes, comme indiqué à l’étape 5 de la section « Étapes recommandées » :

* Latence de bout en bout
* Latence du serveur
* Latence du client

Dans une **opération GetBlob** avec **RequestStatus = (SAS)NetworkError** , si le **Temps maximal** est consacré à la **Latence du client** , le problème le plus courant est que le client se déconnecte avant l’expiration du délai dans le service de stockage.

**Recommandation :**

* Examinez le code dans votre client afin de comprendre pourquoi et quand le client se déconnecte du service de stockage.
* Utilisez Wireshark, l’analyseur de message Microsoft ou TCPing pour enquêter sur les problèmes de connectivité réseau provenant du client. 

### <a name="put-operation-requeststatus--success"></a>Opération Put : RequestStatus = Success

Vérifiez les valeurs suivantes, comme indiqué à l’étape 5 de la section « Étapes recommandées » :

* Latence de bout en bout
* Latence du serveur
* Latence du client

Dans une **opération Put** avec **RequestStatus = Success** , si le **Temps maximal** est consacré à la **Latence du client** , c’est le signe que le client prend plus de temps à envoyer des données au Stockage Azure. Ce délai indique un problème côté client.

**Recommandation :**

* Examinez le code de votre client.
* Utilisez Wireshark, l’analyseur de message Microsoft ou TCPing pour enquêter sur les problèmes de connectivité réseau provenant du client. 

### <a name="put-operation-requeststatus--sasnetworkerror"></a>Opération Put : RequestStatus = (SAS)NetworkError

Vérifiez les valeurs suivantes, comme indiqué à l’étape 5 de la section « Étapes recommandées » :

* Latence de bout en bout
* Latence du serveur
* Latence du client

Dans une **opération PutBlob** avec **RequestStatus = (SAS)NetworkError** , si le **Temps maximal** est consacré à la **Latence du client** , le problème le plus courant est que le client se déconnecte avant l’expiration du délai dans le service de stockage.

**Recommandation :**

* Examinez le code dans votre client afin de comprendre pourquoi et quand le client se déconnecte du service de stockage.
* Utilisez Wireshark, l’analyseur de message Microsoft ou TCPing pour enquêter sur les problèmes de connectivité réseau provenant du client.