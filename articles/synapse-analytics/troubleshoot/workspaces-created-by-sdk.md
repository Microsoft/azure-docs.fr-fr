---
title: 'Résoudre les problèmes : Les espaces de travail créés par le SDK ne peuvent pas lancer Synapse Studio'
description: Étapes de résolution des problèmes d’espaces de travail créés par le Kit de développement logiciel (SDK) incapables de lancer Synapse Studio
author: julieMSFT
ms.author: jrasnick
ms.topic: troubleshooting
ms.service: synapse-analytics
ms.subservice: workspace
ms.date: 11/24/2020
ms.openlocfilehash: f5d0aae975d476da47510a1f2fd164cbc5f32945
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465271"
---
# <a name="troubleshoot-azure-synapse-analytics-workspaces-created-using-sdk"></a>Résoudre des problèmes d’espaces de travail Azure Synapse Analytics créés à l’aide du Kit de développement logiciel (SDK)

Cet article décrit les étapes de résolution des problèmes de lancement de Synapse Studio à partir d’un espace de travail Synapse créé avec un Kit de développement logiciel (SDK).


## <a name="prerequisites"></a>Prérequis

- Espace de travail Synapse créé à l’aide d’un Kit de développement logiciel (SDK)

## <a name="workaround"></a>Solution de contournement

Pour lancer Synapse Studio à partir de votre espace de travail créé à l’aide d’un Kit de développement logiciel (SDK), procédez comme suit : 
  1.    Créez un espace de travail en exécutant `az synapse workspace create`.
  2.    Extrayez l’ID de l’identité managée en exécutant `$identity=$(az synapse workspace show --name {workspace name}  --resource-group {resource group name} --query "identity.principalId")`.
  3.    Accordez le rôle contributeur aux données BLOB de stockage au compte de stockage des identités gérées en exécutant ` az role assignment create --role "Storage Blob Data Contributor" --assignee-object-id {identity } --scope {storage account resource id}.`
  4.    Ajoutez une règle de pare-feu en exécutant ` az synapse firewall-rule create --name allowAll --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255 `.

## <a name="next-steps"></a>Étapes suivantes

* [Qu’est-ce qu’Azure Synapse ?](../overview-what-is.md)
* [Bien démarrer](../get-started.md)
