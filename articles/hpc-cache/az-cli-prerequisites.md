---
title: Prérequis Azure CLI à l’utilisation d’Azure HPC Cache
description: Étapes de configuration avant de pouvoir utiliser Azure CLI pour créer ou modifier un cache Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 07/08/2020
ms.author: v-erkel
ms.openlocfilehash: 13f45c96a830110bd0f4a2d4a2b422921d7a2e31
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94654454"
---
# <a name="set-up-azure-cli-for-azure-hpc-cache"></a>Configurer Azure CLI pour Azure HPC Cache

Suivez ces étapes pour préparer votre environnement avant d’utiliser Azure CLI pour créer ou gérer un cache Azure HPC Cache.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Azure HPC Cache nécessite la version 2.7 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="set-default-resource-group-optional"></a>Définir le groupe de ressources par défaut (facultatif)

La plupart des commandes hpc-cache nécessitent que vous transmettiez le groupe de ressources du cache. Vous pouvez définir le groupe de ressources par défaut en utilisant [az configure](/cli/azure/reference-index#az-configure).

## <a name="next-steps"></a>Étapes suivantes

Après avoir installé l’extension Azure CLI et vous être connecté, vous pouvez utiliser Azure CLI pour créer et gérer des systèmes Azure HPC Cache.

* [Créer un cache Azure HPC Cache](hpc-cache-create.md)
* [Documentation Azure CLI sur hpc-cache](/cli/azure/ext/hpc-cache/hpc-cache)
