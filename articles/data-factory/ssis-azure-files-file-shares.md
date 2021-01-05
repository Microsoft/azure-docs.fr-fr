---
title: Ouvrir et enregistrer des fichiers avec des packages SSIS déployés dans Azure
description: Découvrez comment ouvrir et enregistrer des fichiers localement et dans Azure quand effectuez un lift-and-shift de packages SSIS qui utilisent des systèmes de fichiers locaux dans SSIS dans Azure.
ms.date: 06/27/2018
ms.topic: conceptual
ms.prod: sql
ms.technology: integration-services
author: swinarko
ms.author: sawinark
ms.reviewer: maghan
ms.openlocfilehash: a2b95e0de642dc17865643ec569e7220fe5581b2
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637494"
---
# <a name="open-and-save-files-on-premises-and-in-azure-with-ssis-packages-deployed-in-azure"></a>Ouvrir et enregistrer des fichiers localement et dans Azure avec des packages SSIS déployés dans Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article explique ouvrir et enregistrer des fichiers localement et dans Azure quand effectuez un lift-and-shift de packages SSIS qui utilisent des systèmes de fichiers locaux dans SSIS dans Azure.

## <a name="save-temporary-files"></a>Enregistrer des fichiers temporaires

Si vous devez stocker et traiter des fichiers temporaires au cours d’une même exécution d’un package, les packages peuvent utiliser le répertoire de travail actuel (`.`) ou le répertoire temporaire (`%TEMP%`) de vos nœuds Azure SSIS Integration Runtime.

## <a name="use-on-premises-file-shares"></a>Utiliser des partages de fichiers locaux

Pour continuer à utiliser les **partages de fichiers locaux** quand vous faites un lift-and-shift des packages basés sur des systèmes de fichiers locaux dans SSIS au sein d’Azure, effectuez les actions suivantes :

1. Transférez les fichiers des systèmes de fichiers locaux vers des partages de fichiers locaux.

2. Joignez les partages de fichiers locaux à un réseau virtuel Azure.

3. Joignez votre runtime d’intégration Azure-SSIS au même réseau virtuel. Pour plus d’information, voir [Joindre un runtime d’intégration Azure-SSIS à un réseau virtuel](./join-azure-ssis-integration-runtime-virtual-network.md).

4. Connectez votre runtime d’intégration Azure-SSIS aux partages de fichiers locaux dans le même réseau virtuel en configurant des informations d’identification d’accès qui utilisent l’authentification Windows. Pour plus d’informations, consultez [Se connecter à des données et des partages de fichiers avec l’authentification Windows](ssis-azure-connect-with-windows-auth.md).

5. Mettez à jour les chemins de fichiers locaux de vos packages en fonction des chemins UNC qui pointent vers les partages de fichiers locaux. Par exemple, mettez à jour `C:\abc.txt` vers `\\<on-prem-server-name>\<share-name>\abc.txt`.

## <a name="use-azure-file-shares"></a>Utiliser les partages de fichiers Azure

Pour utiliser **Azure Files** quand vous faites un lift-and-shift des packages basés sur des systèmes de fichiers locaux dans SSIS au sein d’Azure, effectuez les actions suivantes :

1. Transférez les fichiers des systèmes de fichiers locaux vers Azure Files. Pour plus d’informations, consultez [Azure Files](https://azure.microsoft.com/services/storage/files/).

2. Connectez votre runtime d’intégration Azure SSIS à Azure Files en configurant les informations d’identification d’accès qui utilisent l’authentification Windows. Pour plus d’informations, consultez [Se connecter à des données et des partages de fichiers avec l’authentification Windows](ssis-azure-connect-with-windows-auth.md).

3. Mettez à jour les chemins de fichiers locaux de vos packages en fonction des chemins UNC qui pointent vers Azure Files. Par exemple, mettez à jour `C:\abc.txt` vers `\\<storage-account-name>.file.core.windows.net\<share-name>\abc.txt`.

## <a name="next-steps"></a>Étapes suivantes

- Déployez vos packages. Pour plus d’informations, voir [Déployer un projet SSIS dans Azure avec SSMS](/sql/integration-services/ssis-quickstart-deploy-ssms).
- Exécutez vos packages. Pour plus d’informations, voir [Exécuter des packages SSIS sur Azure avec SSMS](/sql/integration-services/ssis-quickstart-run-ssms).
- Planifiez vos packages. Pour plus d’informations, consultez [Planifier des packages SSIS dans Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms?view=sql-server-ver15).