---
title: Utilisation de certificats – Azure Batch
description: Utiliser des certificats pour activer l’authentification d’applications
services: batch
documentationcenter: .net
author: LauraBrenner
manager: evansma
editor: ''
ms.assetid: 63d9d4f1-8521-4bbb-b95a-c4cad73692d3
ms.service: batch
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 02/17/2020
ms.author: labrenne
ms.custom: seodec18
ms.openlocfilehash: 5f7d547ad549b2747ae41182ee4058b001d9e78a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89146383"
---
# <a name="using-certificates-with-batch"></a>Utilisation de certificats avec Batch

Actuellement, la principale raison d’utiliser des certificats avec Batch est que vous avez des applications s’exécutant dans des pools qui doivent s’authentifier auprès d’un point de terminaison. 

Si vous n’avez pas encore de certificat, vous pouvez créer un certificat autosigné à l’aide de l’outil en ligne de commande `makecert`.

## <a name="upload-certificates-manually-through-the-azure-portal"></a>Charger des certificats manuellement via le portail Azure

1. À partir du compte Batch sur lequel vous souhaitez charger un certificat, sélectionnez **Certificats**, puis sélectionnez **Ajouter**. 

2. Chargez le certificat avec une extension .pfx ou .cer. 

Une fois chargé, le certificat est ajouté à une liste de certificats et vous pouvez vérifier l’empreinte.

Désormais, quand vous créez un pool Batch, vous pouvez accéder à Certificats au sein du pool et attribuer à ce dernier le certificat que vous avez chargé.

## <a name="next-steps"></a>Étapes suivantes

Batch a une API de certificat, [AZ batch certificate create](/cli/azure/batch/certificate)

Pour plus d’informations sur l’utilisation de Key Vault, consultez [Accéder de façon sécurisée à Key Vault avec Batch](credential-access-key-vault.md).
