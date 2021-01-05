---
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/28/2020
ms.author: normesta
ms.openlocfilehash: 9b18687c0a6f3e48d94431e88be8ae8137c9dcdb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011134"
---
| Propriété | Description |
|:--- |:---|
|**accountName** | Nom du compte de stockage. Par exemple : `mystorageaccount`.  |
|**requestUrl** | URL demandée. |
|**userAgentHeader** | Valeur d’**en-tête User-Agent**, entre guillemets. Par exemple : `WA-Storage/6.2.0 (.NET CLR 4.0.30319.42000; Win32NT 6.2.9200.0)`.|
|**referrerHeader** | Valeur d’en-tête du **référent**. Par exemple : `http://contoso.com/about.html`.|
|**clientRequestId** | Valeur d’en-tête **x-ms-client-request-id** de la demande. Par exemple : `360b66a6-ad4f-4c4a-84a4-0ad7cb44f7a6`. |
|**etag** | Identificateur de l’ETag pour l’objet retourné, entre guillemets. Par exemple : `0x8D101F7E4B662C4`.  |
|**serverLatencyMs** | Durée totale, en millisecondes, pour exécuter l’opération demandée. Cette valeur n’inclut pas la latence réseau (le temps nécessaire à la lecture de la demande entrante et à l’envoi de la réponse au demandeur). Par exemple : `22`. |
|**serviceType** | Service associé à cette demande. Par exemple : `blob`, `table`, `files` ou `queue`. |
|**operationCount** | Nombre de chaque opération journalisée qui est impliquée dans la demande. Ce nombre commence par un index de `0`. Certaines requêtes nécessitent plusieurs opérations. La plupart des demandes effectuent une seule opération. Par exemple : `1`. |
|**requestHeaderSize** | Taille de l’en-tête de demande, en octets. Par exemple : `578`. <br>Si une demande n’aboutit pas, cette valeur peut être vide. |
|**requestBodySize** | Taille des paquets de demande, en octets, lus par le service de stockage. <br> Par exemple : `0`. <br>Si une demande n’aboutit pas, cette valeur peut être vide.  |
|**responseHeaderSize** | Taille de l’en-tête de réponse, en octets. Par exemple : `216`. <br>Si une demande n’aboutit pas, cette valeur peut être vide.  |
|**responseBodySize** | Taille des paquets de réponse écrits par le service de stockage, en octets. Si une demande n’aboutit pas, cette valeur peut être vide. Par exemple : `216`.  |
|**requestMd5** | Valeur de l’en-tête **Content-MD5** ou de l’en-tête **x-ms-content-md5** dans la demande. La valeur de hachage MD5 spécifiée dans ce champ représente le contenu dans la demande. Par exemple : `788815fd0198be0d275ad329cafd1830`. <br>Ce champ peut être vide.  |
|**serverMd5** | Valeur du hachage MD5 calculée par le service de stockage. Par exemple : `3228b3cf1069a5489b298446321f8521`. <br>Ce champ peut être vide.  |
|**lastModifiedTime** | Heure de dernière modification (LMT) de l’objet retourné.  Par exemple : `Tuesday, 09-Aug-11 21:13:26 GMT`. <br>Ce champ est vide pour les opérations qui peuvent retourner plusieurs objets. |
|**conditionsUsed** | Liste de paires clé-valeur séparées par des points-virgules qui représentent des conditions. Les conditions peuvent correspondre aux éléments suivants : <li> If-Modified-Since <li> If-Unmodified-Since <li> If-Match <li> If-None-Match  <br> Par exemple : `If-Modified-Since=Friday, 05-Aug-11 19:11:54 GMT`. |
|**contentLengthHeader** | Valeur de l’en-tête Content-Length de la demande envoyée au service de stockage. Si la demande a réussi, cette valeur est égale à requestBodySize. Si une demande n’aboutit pas, cette valeur peut ne pas être égale à requestBodySize, ou être vide. |
|**tlsVersion** | Version TLS utilisée dans la connexion de la demande. Par exemple : `TLS 1.2`. |
|**smbTreeConnectID** | Identificateur SMB (Server Message Block) **treeConnectId** établi au moment de la connexion de l’arborescence. Par exemple : `0x3` |
|**smbPersistentHandleID** | ID de handle volatile issu d’une demande SMB2 CREATE qui survit aux reconnexions réseau.  Référencé dans [MS-SMB2](/openspecs/windows_protocols/ms-smb2/f1d9b40d-e335-45fc-9d0b-199a31ede4c3) 2.2.14.1 comme **SMB2_FILEID.Persistent**. Par exemple : `0x6003f` |
|**smbVolatileHandleID** | ID de handle volatile issu d’une demande SMB2 CREATE qui est recyclée sur les reconnexions réseau.  Référencé dans [MS-SMB2](/openspecs/windows_protocols/ms-smb2/f1d9b40d-e335-45fc-9d0b-199a31ede4c3) 2.2.14.1 comme **SMB2_FILEID.Volatile**. Par exemple : `0xFFFFFFFF00000065` |
|**smbMessageID** | **MessageId** relatif à la connexion. Par exemple : `0x3b165` |
|**smbCreditsConsumed** | Entrée ou sortie consommée par la demande, en unités de 64 000. Par exemple : `0x3` |
|**smbCommandDetail** | Plus d’informations sur cette demande spécifique plutôt que sur le type général de demande. Par exemple : `0x2000 bytes at offset 0xf2000` |
|**smbFileId** | **FileId** associé au fichier ou au répertoire.  À peu près similaire à un identificateur NTFS FileId. Par exemple : `0x9223442405598953` |
|**smbSessionID** | Identificateur SMB2 **SessionId** établi au moment de la configuration de la session. Par exemple : `0x8530280128000049` |
|**smbCommandMajor  uint32** | Valeur dans **SMB2_HEADER.Command**. Actuellement, il s’agit d’un nombre compris entre 0 et 18 inclus. Par exemple : `0x6` |
|**smbCommandMinor** | Sous-classe de **SmbCommandMajor**, le cas échéant. Par exemple : `DirectoryCloseAndDelete` |