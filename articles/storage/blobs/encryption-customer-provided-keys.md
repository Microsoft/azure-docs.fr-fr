---
title: Fournir une clé de chiffrement lors d’une requête au stockage d’objets blob
titleSuffix: Azure Storage
description: Les clients effectuant des requêtes auprès du stockage d’objets blob Azure ont la possibilité de fournir une clé de chiffrement lors de chaque requête. L’inclusion de la clé de chiffrement dans la requête offre un contrôle précis des paramètres de chiffrement pour les opérations de stockage d’objets blob.
services: storage
author: tamram
ms.service: storage
ms.date: 09/17/2020
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: abdc83019205fc39e1e85a53da7e49f8a7d4f11c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91618724"
---
# <a name="provide-an-encryption-key-on-a-request-to-blob-storage"></a>Fournir une clé de chiffrement lors d’une requête au stockage d’objets blob

Les clients effectuant des requêtes auprès du stockage d’objets blob Azure ont la possibilité de fournir une clé de chiffrement lors de chaque requête. L’inclusion de la clé de chiffrement dans la requête offre un contrôle précis des paramètres de chiffrement pour les opérations de stockage d’objets blob. Les clés fournies par le client peuvent être stockées dans Azure Key Vault ou dans un autre magasin de clés.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="encrypting-read-and-write-operations"></a>Chiffrement des opérations de lecture et d’écriture

Quand une application cliente fournit une clé de chiffrement dans la requête, Stockage Azure effectue le chiffrement et le déchiffrement de manière transparente lors de la lecture et de l’écriture des données blob. Stockage Azure écrit un hachage SHA-256 de la clé de chiffrement avec le contenu de l’objet blob. Le hachage est utilisé pour vérifier que toutes les opérations suivantes sur l’objet blob utilisent la même clé de chiffrement.

Stockage Azure ne stocke pas et ne gère pas la clé de chiffrement que le client envoie avec la requête. La clé est supprimée de façon sécurisée dès que le processus de chiffrement ou de déchiffrement est terminé.

Quand un client crée ou met à jour un objet blob à l’aide d’une clé fournie par le client lors d’une requête, les requêtes de lecture et d’écriture suivantes pour cet objet blob doivent également fournir la clé. Si la clé n’est pas fournie sur une requête d’objet blob qui a déjà été chiffrée avec une clé fournie par le client, la requête échoue avec le code d’erreur 409 (conflit).

Si l’application cliente envoie une clé de chiffrement dans la requête et que le compte de stockage est également chiffré à l’aide d’une clé managée par Microsoft ou d’une clé managée par le client, Stockage Azure utilise la clé fournie dans la requête pour le chiffrement et le déchiffrement.

Pour envoyer la clé de chiffrement dans le cadre de la requête, un client doit établir une connexion sécurisée à Stockage Azure à l’aide du protocole HTTPS.

Chaque instantané d’objet blob peut avoir sa propre clé de chiffrement.

## <a name="request-headers-for-specifying-customer-provided-keys"></a>En-têtes de requêtes pour spécifier les clés fournies par le client

Pour les appels REST, les clients peuvent utiliser les en-têtes suivants pour transmettre en toute sécurité des informations sur les clés de chiffrement sur une requête au stockage d’objets blob :

|En-tête de requête | Description |
|---------------|-------------|
|`x-ms-encryption-key` |Nécessaire pour les requêtes d’écriture et de lecture. Valeur de clé de chiffrement AES-256 encodée en base64. |
|`x-ms-encryption-key-sha256`| Nécessaire pour les requêtes d’écriture et de lecture. Valeur SHA256 encodée en base64 de la clé de chiffrement. |
|`x-ms-encryption-algorithm` | Nécessaire pour les requêtes d’écriture, facultatif pour les requêtes de lecture. Spécifie l’algorithme à utiliser lors du chiffrement des données à l’aide de la clé donnée. Doit être AES256. |

La spécification de clés de chiffrement dans la requête est facultative. Toutefois, si vous spécifiez l’un des en-têtes listés ci-dessus pour une opération d’écriture, vous devez les spécifier tous.

## <a name="blob-storage-operations-supporting-customer-provided-keys"></a>Opérations de stockage d’objets blob prenant en charge les clés fournies par le client

Les opérations de stockage d’objets blob suivantes prennent en charge l’envoi de clés de chiffrement fournies par le client sur une requête :

- [Put Blob](/rest/api/storageservices/put-blob)
- [Put Block List](/rest/api/storageservices/put-block-list)
- [Put Block](/rest/api/storageservices/put-block)
- [Put Block from URL](/rest/api/storageservices/put-block-from-url)
- [Put Page](/rest/api/storageservices/put-page)
- [Placer la page à partir de l’URL](/rest/api/storageservices/put-page-from-url)
- [Append Block](/rest/api/storageservices/append-block)
- [Set Blob Properties](/rest/api/storageservices/set-blob-properties)
- [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata)
- [Get Blob](/rest/api/storageservices/get-blob)
- [Get Blob Properties](/rest/api/storageservices/get-blob-properties)
- [Get Blob Metadata](/rest/api/storageservices/get-blob-metadata)
- [Snapshot Blob](/rest/api/storageservices/snapshot-blob)

## <a name="rotate-customer-provided-keys"></a>Permuter des clés fournies par le client

Pour permuter une clé de chiffrement qui a été utilisée pour chiffrer un objet blob, téléchargez l’objet blob puis rechargez-le avec la nouvelle clé de chiffrement.

> [!IMPORTANT]
> Vous ne pouvez pas utiliser le portail Azure pour lire ou écrire dans un conteneur ou un objet blob chiffré avec une clé fournie dans la requête.
>
> Veillez à protéger la clé de chiffrement que vous fournissez dans une requête au stockage d’objets blob dans un magasin de clés sécurisé comme Azure Key Vault. Si vous tentez d’effectuer une opération d’écriture sur un conteneur ou un objet blob sans clé de chiffrement, l’opération échoue et vous perdez l’accès à l’objet.

## <a name="next-steps"></a>Étapes suivantes

- [Spécifier une clé fournie par le client sur une demande au stockage Blob avec .NET](storage-blob-customer-provided-key.md)
- [Chiffrement du stockage Azure pour les données au repos](../common/storage-service-encryption.md)
