---
title: Fichier Include
description: Fichier Include
services: batch
author: laurenhughes
ms.service: batch
ms.topic: include
ms.date: 05/04/2018
ms.author: lahugh
ms.custom: include file
ms.openlocfilehash: 51b687dc2a6264eb12362616429746c9b182c3b1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68323888"
---
> [!NOTE]
> Lorsque vous créez un compte Batch, vous pouvez choisir entre deux modes de *répartition de pool* : **abonnement utilisateur** et **service Batch**. Dans la plupart des cas, vous devez utiliser le mode de service Batch par défaut. Les pools sont alloués en arrière-plan dans des abonnements gérés par Azure. Dans l’autre mode d’abonnement utilisateur, les machines virtuelles Batch et les autres ressources sont créées directement dans l’abonnement lors de la création d’un pool. Le mode d’abonnement utilisateur est nécessaire si vous voulez créer des pools Batch à l’aide d’[Azure Reserved VM Instances](https://azure.microsoft.com/pricing/reserved-vm-instances/). Pour créer un compte Batch dans le mode abonnement utilisateur, vous devez également inscrire votre abonnement auprès d’Azure Batch et associer le compte avec une Azure Key Vault.