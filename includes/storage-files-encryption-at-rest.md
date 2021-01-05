---
title: Fichier Include
description: Fichier Include
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 12/27/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 814bf085ca910ef07a13df779eef25e0e6510e57
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011736"
---
Toutes les données stockées dans Azure Files sont chiffrées au repos avec Azure Storage Service Encryption (SSE). Cette fonctionnalité fonctionne de la même façon que BitLocker sur Windows : les données sont chiffrées sous le niveau du système de fichiers. Comme les données sont chiffrées sous le système de fichiers du partage de fichiers Azure, car codées sur le disque, il n’est pas nécessaire d’avoir accès à la clé sous-jacente sur le client pour lire ou écrire sur le partage de fichiers Azure. Le chiffrement au repos s’applique aux protocoles SMB et NFS.

Par défaut, les données stockées dans Azure Files sont chiffrées avec des clés managées par Microsoft. Microsoft détient ainsi les clés permettant de chiffrer et de déchiffrer les données et se charge de les renouveler régulièrement. Vous pouvez également choisir de gérer vos propres clés, ce qui vous permet de contrôler le processus de renouvellement. Si vous choisissez de chiffrer vos partages de fichiers avec des clés gérées par le client, Azure Files est autorisé à accéder à vos clés pour traiter les demandes de lecture et d’écriture de vos clients. Vous pouvez révoquer cette autorisation à tout moment, mais dans ce cas votre partage de fichiers Azure ne sera plus accessible via SMB ou l’API FileREST.

Azure Files utilise le même schéma de chiffrement que les autres services de stockage Azure, comme le Stockage Blob Azure. Pour plus d’informations sur Azure Storage Service Encryption (SSE), consultez [Chiffrement du stockage Azure pour les données au repos](../articles/storage/common/storage-service-encryption.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).