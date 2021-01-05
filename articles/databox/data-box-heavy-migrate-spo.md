---
title: Utiliser Azure Data Box Heavy pour déplacer le contenu d’un partage de fichiers vers SharePoint Online
description: Utilisez ce tutoriel pour apprendre à migrer le contenu d’un partage de fichiers vers SharePoint Online à l’aide d’Azure Data Box Heavy
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: how-to
ms.date: 07/18/2019
ms.author: alkohli
ms.openlocfilehash: b00a944b69b8492d4aa00258154bf00f1c66ad33
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92127112"
---
# <a name="use-the-azure-data-box-heavy-to-migrate-your-file-share-content-to-sharepoint-online"></a>Utiliser Azure Data Box Heavy pour migrer le contenu de votre partage de fichiers vers SharePoint Online

Utilisez Azure Data Box Heavy et l’Outil de migration SharePoint (SPMT) pour migrer facilement le contenu de votre partage de fichiers vers SharePoint Online et OneDrive. Data Box Heavy vous permet de ne plus dépendre de votre liaison WAN pour transférer les données.

Microsoft Azure Data Box est un service qui vous permet de commander un appareil à partir du portail Microsoft Azure. Vous pouvez ensuite copier des téraoctets de données à partir de vos serveurs sur l’appareil. Une fois l’appareil réexpédié à Microsoft, vos données sont copiées dans Azure. Selon la taille des données que vous projetez de transférer, vous pouvez choisir différents appareils :

- [Data Box Disk](./data-box-disk-overview.md) avec 35 To de capacité utilisable par commande pour des jeux de données petits à moyens.
- [Data Box](./data-box-overview.md) avec 80 To de capacité utilisable par appareil pour des jeux de données moyens à grands.
- [Data Box Heavy](./data-box-heavy-overview.md) avec 770 To de capacité utilisable par appareil pour des grands jeux de données.

Cet article couvre en particulier l’utilisation de Data Box Heavy pour migrer le contenu de votre partage de fichiers vers SharePoint Online.

## <a name="requirements-and-costs"></a>Exigences et coûts

### <a name="for-data-box-heavy"></a>Pour Data Box Heavy

- Data Box Heavy est uniquement disponible dans le cadre des offres Accord Entreprise (EA), de fournisseur de solutions cloud (CSP) ou de parrainage Azure. Si votre abonnement ne correspond à aucune des offres ci-dessus, contactez le support Microsoft pour mettre à niveau votre abonnement ou consultez les [Prix des abonnements Azure](https://azure.microsoft.com/pricing/).
- L’utilisation de Data Box Heavy est payante. Consultez les [prix de Data Box Heavy](https://azure.microsoft.com/pricing/details/databox/heavy/).


### <a name="for-sharepoint-online"></a>Pour SharePoint Online

- Passez en revue les [exigences minimales de l’Outil de migration SharePoint](/sharepointmigration/how-to-use-the-sharepoint-migration-tool).

## <a name="workflow-overview"></a>Vue d’ensemble du workflow

Ce workflow nécessite des actions aussi bien sur Azure Data Box Heavy que sur SharePoint Online.
Les étapes suivantes concernent Azure Data Box Heavy.

1. Commandez Azure Data Box Heavy.
2. Recevez et configurez votre appareil.
3. Copiez les données de votre partage de fichiers local dans le dossier Azure Files sur votre appareil.
4. Une fois la copie effectuée, réexpédiez l’appareil en suivant les instructions.
5. Attendez la fin du chargement des données sur Azure.

Les étapes suivantes concernent SharePoint Online.

6. Créez une machine virtuelle dans le portail Azure et montez-y le partage de fichiers Azure.
7. Installez l’outil SPMT sur la machine virtuelle Azure.
8. Exécutez l’outil SPMT en définissant le partage de fichiers Azure comme *source*.
9. Effectuez les étapes finales de l’outil.
10. Vérifiez et confirmez vos données.

## <a name="use-data-box-heavy-to-copy-data"></a>Utiliser Data Box Heavy pour copier les données

Effectuez les étapes suivantes pour copier les données dans votre Data Box Heavy.

1. [Commandez votre Data Box Heavy](data-box-heavy-deploy-ordered.md).
2. Après avoir reçu votre Data Box Heavy, [Configurez Data Box Heavy](data-box-heavy-deploy-set-up.md). Vous câblez et configurez les deux nœuds sur votre appareil.
3. [Copiez les données sur Azure Data Box Disk](data-box-heavy-deploy-copy-data.md). Pendant la copie, vérifiez que vous :

    - Utilisez uniquement le dossier *StorageAccountName_AzFile* de Data Box Heavy pour copier les données. C’est pour que les données se retrouvent dans un partage de fichiers Azure, et non dans des objets blob de blocs ou des objets blob de pages.
    - Copiez les fichiers dans un dossier au sein du dossier *StorageAccountName_AzFile*. Un sous-dossier du dossier *StorageAccountName_AzFile* crée un partage de fichiers. Les fichiers copiés directement dans le dossier *StorageAccountName_AzFile* échouent et sont chargés en tant qu’objets blob de blocs. Il s’agit du partage de fichiers que vous montez sur votre machine virtuelle à l’étape suivante.
    - Copiez des données dans les deux nœuds de votre Data Box Heavy.
3. Exécutez l’étape [Préparer l’expédition](data-box-heavy-deploy-picked-up.md#prepare-to-ship) sur votre appareil. Cette étape est essentielle pour assurer le chargement des fichiers sur Azure.
4. [Renvoyez l’appareil](data-box-heavy-deploy-picked-up.md#ship-data-box-heavy-back).
5. [Vérifiez le chargement des données sur Azure](data-box-heavy-deploy-picked-up.md#verify-data-upload-to-azure).

## <a name="use-spmt-to-migrate-data"></a>Utiliser SPMT pour migrer les données

Après avoir reçu la confirmation de l’équipe de données Azure que votre copie de données est effectuée, effectuez la migration de vos données vers SharePoint Online.

Pour optimiser les performances et la connectivité, nous vous recommandons de créer une machine virtuelle Azure.

1. Connectez-vous au portail Azure, puis [créez une machine virtuelle](../virtual-machines/windows/quick-create-portal.md).
2. [Montez le partage de fichiers Azure sur la machine virtuelle](../storage/files/storage-how-to-use-files-windows.md#mount-the-azure-file-share-with-file-explorer).
3. [Téléchargez l’Outil de migration SharePoint](https://spmtreleasescus.blob.core.windows.net/install/default.htm) et installez-le sur votre machine virtuelle Azure.
4. Démarrez l’Outil de migration SharePoint. Cliquez sur **Se connecter** et entrez le nom d’utilisateur et le mot de passe de votre compte professionnel ou scolaire.
5. Quand une invite vous demande **Où se trouvent vos données ?** , sélectionnez **Partage de fichiers**. Entrez le chemin du partage de fichiers Azure où se trouvent vos données.
6. Suivez les invites restantes, y compris la spécification de votre emplacement cible. Pour plus d’informations, accédez à [Comment utiliser l’Outil de migration SharePoint](/sharepointmigration/how-to-use-the-sharepoint-migration-tool).

> [!IMPORTANT]
> - La vitesse à laquelle les données sont ingérées dans SharePoint Online est affectée par plusieurs facteurs, même si vos données sont déjà prêtes dans Azure. Examinez ces facteurs pour bien planifier et optimiser l’efficacité de votre migration.  Pour plus d’informations, accédez à [Vitesse de migration de SharePoint Online et OneDrive](/sharepointmigration/sharepoint-online-and-onedrive-migration-speed).
> - Vous risquez éventuellement de perdre les autorisations existantes sur les fichiers pendant la migration des données vers SharePoint Online. Vous risquez aussi de perdre certaines métadonnées, comme les informations *Créé par* et *Date de modification*.

## <a name="next-steps"></a>Étapes suivantes

[Commander votre Data Box Heavy](./data-box-heavy-deploy-ordered.md)