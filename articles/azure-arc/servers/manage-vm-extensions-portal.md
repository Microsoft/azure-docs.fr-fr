---
title: Activer l’extension de machine virtuelle à partir du portail Azure
description: Cet article explique comment déployer des extensions de machine virtuelle sur des serveurs Azure Arc exécutés dans des environnements cloud hybrides à partir du portail Azure.
ms.date: 11/06/2020
ms.topic: conceptual
ms.openlocfilehash: 48d7d4085dce893d94436fe0c6be32cfeea9cda3
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359083"
---
# <a name="enable-azure-vm-extensions-from-the-azure-portal"></a>Activer les extensions de machine virtuelle Azure à partir du portail Azure

Cet article explique comment déployer et désinstaller des extensions de machine virtuelle Azure, prises en charge par les serveurs Azure Arc, sur une machine hybride Linux ou Windows via le portail Azure.

> [!NOTE]
> L’extension de la machine virtuelle Key Vault (préversion) ne prend pas en charge le déploiement à partir du portail Azure, uniquement à l’aide de l’interface de ligne de commande Azure, d’Azure PowerShell ou d’un modèle Azure Resource Manager.

## <a name="enable-extensions-from-the-portal"></a>Activer les extensions à partir du portail

Les extensions de machine virtuelle peuvent être appliquées pour une machine managée par serveur via le portail Azure.

1. À partir de votre navigateur, accédez au [portail Azure](https://portal.azure.com).

2. Sur le portail, accédez à **Serveurs – Azure Arc**, puis sélectionnez votre machine hybride dans la liste.

3. Choisissez **Extensions**, puis sélectionnez **Ajouter**. Choisissez l’extension souhaitée dans la liste des extensions disponibles, puis suivez les instructions de l’Assistant. Dans cet exemple, nous allons déployer l’extension de machine virtuelle Log Analytics.

    ![Sélectionner l’extension de machine virtuelle pour la machine sélectionnée](./media/manage-vm-extensions/add-vm-extensions.png)

    L’exemple suivant illustre l’installation de l’extension de machine virtuelle Log Analytics à partir du portail Azure :

    ![Installer l’extension de machine virtuelle Log Analytics](./media/manage-vm-extensions/mma-extension-config.png)

    Pour mener à bien l’installation, il vous est demandé de fournir l’ID et la clé primaire de l’espace de travail. Si vous ne savez pas comment vous procurer ces informations, consultez [Obtenir l’ID et la clé d’espace de travail](../../azure-monitor/platform/log-analytics-agent.md#workspace-id-and-key).

4. Après avoir confirmé les informations nécessaires que vous avez fournies, sélectionnez **Créer**. Dans le résumé du déploiement qui s’affiche, examinez l’état du déploiement.

>[!NOTE]
>Même si plusieurs extensions peuvent être regroupées et traitées ensemble, elles sont installées en série. Une fois l’installation de la première extension terminée, l’installation de la suivante est entreprise.

## <a name="uninstall-extension"></a>Désinstaller une extension

Vous pouvez supprimer une ou plusieurs extensions d’un serveur Arc à partir du portail Azure. Ensuite, effectuez les opérations suivantes pour supprimer une extension.

1. À partir de votre navigateur, accédez au [portail Azure](https://portal.azure.com).

2. Sur le portail, accédez à **Serveurs – Azure Arc**, puis sélectionnez votre machine hybride dans la liste.

3. Choisissez **Extensions**, puis sélectionnez une extension dans la liste des extensions installées.

4. Sélectionnez **Désinstaller**, puis, lorsque vous êtes invité à vérifier, sélectionnez **Oui** pour continuer.

## <a name="next-steps"></a>Étapes suivantes

- Vous pouvez déployer, gérer et supprimer des extensions de machine virtuelle à l’aide d’[Azure CLI](manage-vm-extensions-cli.md), de [PowerShell](manage-vm-extensions-powershell.md) ou de [modèles Azure Resource Manager](manage-vm-extensions-template.md).

- Des informations de dépannage sont accessibles dans le [guide Résoudre les problèmes liés aux extensions de machine virtuelle](troubleshoot-vm-extensions.md).