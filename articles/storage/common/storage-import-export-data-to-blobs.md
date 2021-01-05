---
title: Utilisation du service Azure Import/Export pour transférer des données dans des objets blob Azure | Microsoft Docs
description: Découvrez comment créer des tâches d’importation et d’exportation dans le portail Azure pour transférer des données vers et à partir d’objets blob Azure.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 10/29/2020
ms.author: alkohli
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 07f1a6ff5d15ee552680c59c86a194aeabe5b866
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96326384"
---
# <a name="use-the-azure-importexport-service-to-import-data-to-azure-blob-storage"></a>Utiliser le service Azure Import/Export pour transférer des données dans le Stockage Blob Azure

Cet article fournit des instructions détaillées sur l’utilisation du service Azure Import/Export pour importer de manière sécurisée de grandes quantités de données dans le Stockage Blob Azure. Pour importer des données dans des objets blob Azure, le service vous demande d’expédier à un centre de données Azure des lecteurs de disque chiffrés contenant vos données.

## <a name="prerequisites"></a>Prérequis

Avant de créer une tâche d’importation pour transférer des données dans le Stockage Blob Azure, passez soigneusement en revue et complétez la liste suivante des prérequis pour ce service.
Vous devez respecter les consignes suivantes :

* Avoir un abonnement Azure actif utilisable avec le service Import/Export
* Avoir au moins un compte de stockage Azure avec un conteneur de stockage. Consultez la liste des [Comptes de stockage et types de stockage pris en charge pour le service Import/Export](storage-import-export-requirements.md).
  * Pour plus d'informations sur la création d'un compte de stockage, consultez la page [Création d'un compte de stockage](storage-account-create.md).
  * Pour plus d’informations sur le conteneur de stockage, accédez à [Créer un conteneur de stockage](../blobs/storage-quickstart-blobs-portal.md#create-a-container).
* Avoir un nombre suffisant de disques correspondant aux [types pris en charge](storage-import-export-requirements.md#supported-disks).
* Avoir un système Windows exécutant une [Version de système d’exploitation prise en charge](storage-import-export-requirements.md#supported-operating-systems).
* Activez BitLocker sur le système Windows. Consultez [Comment activer BitLocker](https://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/).
* [Téléchargez la dernière version 1 de WAImportExport](https://www.microsoft.com/download/details.aspx?id=42659) sur le système Windows. La dernière version de l’outil apporte des mises à jour de sécurité pour autoriser un protecteur externe pour la clé BitLocker, et une nouvelle version de la fonctionnalité du mode de déverrouillage.

  * Décompressez le package dans le dossier par défaut : `waimportexportv1`. Par exemple : `C:\WaImportExportV1`.
* Dotez-vous d’un compte FedEx/DHL. Si vous souhaitez faire appel à un autre transporteur que FedEx/DHL, contactez l’équipe des opérations Azure Data Box à l’adresse `adbops@microsoft.com`.
  * Le compte doit être valide, doit avoir un solde et doit offrir des fonctionnalités de réexpédition.
  * Générez un numéro de suivi pour le travail d’exportation.
  * Chaque travail doit avoir un numéro de suivi distinct. Plusieurs travaux portant le même numéro de suivi ne sont pas pris en charge.
  * Si vous n’avez pas de compte de transporteur, accédez à :
    * [Créer un compte FedEx](https://www.fedex.com/en-us/create-account.html), ou
    * [Créer un compte DHL](http://www.dhl-usa.com/en/express/shipping/open_account.html).

## <a name="step-1-prepare-the-drives"></a>Étape 1 : Préparer les lecteurs

Cette étape génère un fichier journal. Le fichier journal stocke les informations de base comme le numéro de série du lecteur, la clé de chiffrement et les détails du compte de stockage.

Effectuez les étapes suivantes pour préparer les lecteurs.

1. Connectez vos lecteurs de disque au système Windows via des connecteurs SATA.
2. Créez un seul volume NTFS sur chaque lecteur. Attribuez une lettre de lecteur au volume. N’utilisez pas de points de montage.
3. Activez le chiffrement BitLocker sur le volume NTFS. Si vous utilisez un système Windows Server, suivez les instructions de [How to enable BitLocker sur Windows Server 2012 R2](https://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/) (Comment activer BitLocker sur Windows Server 2012 R2).
4. Copiez des données sur le volume chiffré. Utilisez la fonction glisser-déplacer, Robocopy ou n’importe quel outil de copie de ce type. Un fichier journal ( *.jrn*) est créé dans le même dossier où vous avez exécuté l’outil.

   Si le lecteur est verrouillé et que vous devez le déverrouiller, les étapes de déverrouillage peuvent varier en fonction de votre cas d’utilisation.

   * Si vous avez ajouté des données à un lecteur pré-chiffré (l’outil WAImportExport n’a pas été utilisé pour le chiffrement), utilisez la clé BitLocker (un mot de passe numérique que vous spécifiez) dans la fenêtre contextuelle pour déverrouiller le lecteur.

   * Si vous avez ajouté des données à un lecteur qui a été chiffré avec l’outil WAImportExport, utilisez la commande suivante pour déverrouiller le lecteur :

        `WAImportExport Unlock /externalKey:<BitLocker key (base 64 string) copied from journal (*.jrn*) file>`

5. Ouvrez une fenêtre PowerShell ou de ligne de commande avec des privilèges d’administrateur. Pour accéder au répertoire du dossier décompressé, exécutez la commande suivante :

    `cd C:\WaImportExportV1`
6. Pour obtenir la clé BitLocker du lecteur, exécutez la commande suivante :

    `manage-bde -protectors -get <DriveLetter>:`
7. Pour préparer le disque, exécutez la commande suivante. **Selon la taille des données, l’opération peut durer plusieurs heures, voire plusieurs jours.**

    ```powershell
    ./WAImportExport.exe PrepImport /j:<journal file name> /id:session<session number> /t:<Drive letter> /bk:<BitLocker key> /srcdir:<Drive letter>:\ /dstdir:<Container name>/ /blobtype:<BlockBlob or PageBlob> /skipwrite
    ```

    Un fichier journal est créé dans le même dossier où vous avez exécuté l’outil. Deux autres fichiers sont également créés : un fichier *.xml* (dossier où vous exécutez l’outil) et un fichier *drive-manifest.xml* (dossier où se trouvent les données).

    Les paramètres utilisés sont décrits dans le tableau suivant :

    |Option  |Description  |
    |---------|---------|
    |/j:     |Nom du fichier journal, avec l’extension .jrn. Un fichier journal est généré par lecteur. Nous vous recommandons d’utiliser le numéro de série du disque comme nom de fichier journal.         |
    |/id:     |ID de la session. Utilisez un numéro de session unique pour chaque instance de la commande.      |
    |/t:     |Lettre de lecteur du disque à expédier. Exemple : lecteur `D`.         |
    |/bk:     |Clé BitLocker du lecteur. Son mot de passe numérique à partir de la sortie de `manage-bde -protectors -get D:`      |
    |/srcdir:     |Lettre de lecteur du disque à expédier suivie de `:\`. Par exemple : `D:\`.         |
    |/dstdir:     |Nom du conteneur de destination dans le Stockage Azure.         |
    |/blobtype:     |Cette option spécifie le type d’objets blob vers lequel vous souhaitez importer les données. Pour des objets blob de blocs, il s’agit de `BlockBlob`, et pour des objet blob de pages, `PageBlob`.         |
    |/skipwrite:     |Option qui spécifie qu’aucune nouvelle donnée ne doit être copiée et que les données existantes sur le disque doivent être préparées.          |
    |/enablecontentmd5:     |Lorsqu’elle est activée, cette option garantit que MD5 est calculté et défini comme propriété `Content-md5` sur chaque objet blob. Utilisez cette option uniquement si vous souhaitez utiliser le champ `Content-md5` une fois que les données sont téléchargées vers Azure. <br> Cette option n’affecte pas la vérification d’intégrité des données (ce qui se produit par défaut). Le paramètre augmente le temps nécessaire au chargement des données dans le cloud.          |
8. Répétez l’étape précédente pour chaque disque à expédier. Un fichier journal avec le nom fourni est créé pour chaque exécution de la ligne de commande.

    > [!IMPORTANT]
    > * En plus du fichier journal, un fichier `<Journal file name>_DriveInfo_<Drive serial ID>.xml` est également créé dans le même dossier où se trouve l’outil. Le fichier .xml est utilisé à la place du fichier journal quand vous créez une tâche si le fichier journal est trop volumineux.

## <a name="step-2-create-an-import-job"></a>Étape 2 : Créer une tâche d’importation

### <a name="portal"></a>[Portail](#tab/azure-portal)

Effectuez les étapes suivantes pour créer une tâche d’importation dans le portail Azure.

1. Connectez-vous sur https://portal.azure.com/.
2. Accédez à **Tous les services > Stockage > Tâches d’importation/exportation**.

    ![Accéder à Tâches d’importation/exportation](./media/storage-import-export-data-to-blobs/import-to-blob1.png)

3. Cliquez sur **Créer une tâche d’importation/exportation**.

    ![Cliquez sur Créer une tâche d’importation/exportation](./media/storage-import-export-data-to-blobs/import-to-blob2.png)

4. Dans **De base** :

   * Sélectionnez **Importer dans Azure**.
   * Indiquez un nom décrivant le travail d’importation. Utilisez le nom pour suivre la progression de vos tâches.
       * Le nom ne peut contenir que des lettres minuscules, des chiffres et des traits d’union.
       * Le nom doit commencer par une lettre et ne doit pas contenir d’espaces.
   * Sélectionnez un abonnement.
   * Entrez ou sélectionnez un groupe de ressources.

     ![Créer une tâche d’importation - Étape 1](./media/storage-import-export-data-to-blobs/import-to-blob3.png)

5. Dans **Détails de la tâche** :

   * Chargez les fichiers journaux du lecteur que vous avez obtenus à l’étape de préparation de lecteur. Si vous avez utilisé `waimportexport.exe version1`, chargez un fichier pour chaque lecteur préparé. Si la taille du fichier journal dépasse 2 Mo, vous pouvez utiliser `<Journal file name>_DriveInfo_<Drive serial ID>.xml` également créé avec le fichier journal.
   * Sélectionnez le compte de stockage de destination des données.
   * L’emplacement de remise est automatiquement rempli en fonction de la région du compte de stockage sélectionné.

   ![Créer une tâche d’importation - Étape 2](./media/storage-import-export-data-to-blobs/import-to-blob4.png)

6. Dans **Informations de réexpédition** :

   * Sélectionnez le transporteur dans la liste déroulante. Si vous souhaitez utiliser un autre transporteur que FedEx/DHL, choisissez une option existante dans la liste déroulante. Contactez l’équipe des opérations Azure Data Box à l’adresse `adbops@microsoft.com` pour lui indiquer le nom du transporteur auquel vous envisagez de faire appel.
   * Entrez un numéro de compte de transporteur valide que vous avez créé pour ce transporteur. Microsoft utilise ce compte pour renvoyer les lecteurs une fois la tâche d’importation terminée. Si vous n’avez pas de numéro de compte, créez un compte de transporteur [FedEx](https://www.fedex.com/us/oadr/) ou [DHL](https://www.dhl.com/).
   * Indiquez le nom d’un contact, le numéro de téléphone, l’e-mail, l’adresse, la ville, le code postal, l’état/la province et le pays/la région, puis vérifiez que ces informations sont complètes et valides.

       > [!TIP]
       > Au lieu de spécifier une adresse de messagerie pour un seul utilisateur, fournissez une adresse de groupe. Cela garantit que vous recevrez des notifications même si un administrateur s’en va.

     ![Créer une tâche d'importation - Étape 3](./media/storage-import-export-data-to-blobs/import-to-blob5.png)

7. Dans le **Récapitulatif** :

   * Passez en revue les informations de tâche dans le récapitulatif. Notez le nom de la tâche et l’adresse du centre de données Azure pour réexpédier les disques à Azure. Ces informations sont utilisées par la suite sur l’étiquette d’expédition.
   * Cliquez sur **OK** pour créer la tâche d’importation.

     ![Créer une tâche d’importation - Étape 4](./media/storage-import-export-data-to-blobs/import-to-blob6.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Exécutez les étapes suivantes pour créer une tâche d’importation dans Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="create-a-job"></a>Créer un travail

1. Utilisez la commande [az extension add](/cli/azure/extension#az_extension_add) pour ajouter l’extension [az import-export](/cli/azure/ext/import-export/import-export) :

    ```azurecli
    az extension add --name import-export
    ```

1. Vous pouvez utiliser un groupe de ressources existant ou en créer un. Exécutez la commande [az group create](/cli/azure/group#az_group_create) pour créer un groupe de ressources :

    ```azurecli
    az group create --name myierg --location "West US"
    ```

1. Vous pouvez utiliser un compte de stockage existant ou en créer un nouveau. Pour créer un compte de stockage, exécutez la commande [az storage account create](/cli/azure/storage/account#az_storage_account_create) :

    ```azurecli
    az storage account create --resource-group myierg --name myssdocsstorage --https-only
    ```

1. Pour obtenir la liste des emplacements vers lesquels vous pouvez expédier des disques, utilisez la commande [az import-export location list](/cli/azure/ext/import-export/import-export/location#ext_import_export_az_import_export_location_list) :

    ```azurecli
    az import-export location list
    ```

1. Pour obtenir les emplacements de votre région, utilisez la commande [az import-export location show](/cli/azure/ext/import-export/import-export/location#ext_import_export_az_import_export_location_show) :

    ```azurecli
    az import-export location show --location "West US"
    ```

1. Exécutez la commande [az import-export create](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_create) suivante pour créer un travail d’importation :

    ```azurecli
    az import-export create \
        --resource-group myierg \
        --name MyIEjob1 \
        --location "West US" \
        --backup-drive-manifest true \
        --diagnostics-path waimportexport \
        --drive-list bit-locker-key=439675-460165-128202-905124-487224-524332-851649-442187 \
            drive-header-hash= drive-id=AZ31BGB1 manifest-file=\\DriveManifest.xml \
            manifest-hash=69512026C1E8D4401816A2E5B8D7420D \
        --type Import \
        --log-level Verbose \
        --shipping-information recipient-name="Microsoft Azure Import/Export Service" \
            street-address1="3020 Coronado" city="Santa Clara" state-or-province=CA postal-code=98054 \
            country-or-region=USA phone=4083527600 \
        --return-address recipient-name="Gus Poland" street-address1="1020 Enterprise way" \
            city=Sunnyvale country-or-region=USA state-or-province=CA postal-code=94089 \
            email=gus@contoso.com phone=4085555555" \
        --return-shipping carrier-name=FedEx carrier-account-number=123456789 \
        --storage-account myssdocsstorage
    ```

   > [!TIP]
   > Au lieu de spécifier une adresse de messagerie pour un seul utilisateur, fournissez une adresse de groupe. Cela garantit que vous recevrez des notifications même si un administrateur s’en va.

1. Utilisez la commande [az import-export list](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_list) pour afficher toutes les tâches du groupe de ressources myierg :

    ```azurecli
    az import-export list --resource-group myierg
    ```

1. Pour mettre à jour votre travail ou l’annuler, exécutez la commande [az import-export update](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_update) :

    ```azurecli
    az import-export update --resource-group myierg --name MyIEjob1 --cancel-requested true
    ```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Exécutez les étapes suivantes pour créer une tâche d’importation dans Azure PowerShell.

[!INCLUDE [azure-powershell-requirements-h3.md](../../../includes/azure-powershell-requirements-h3.md)]

> [!IMPORTANT]
> Tant que le module PowerShell **Az.ImportExport** est en préversion, vous devez l’installer séparément à l’aide de la cmdlet `Install-Module`. Une fois que ce module PowerShell sera en disponibilité générale, il fera partie intégrante des versions futures du module Az PowerShell et sera disponible par défaut dans Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.ImportExport
```

### <a name="create-a-job"></a>Créer un travail

1. Vous pouvez utiliser un groupe de ressources existant ou en créer un. Pour créer un groupe de ressources, exécutez la cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) :

   ```azurepowershell-interactive
   New-AzResourceGroup -Name myierg -Location westus
   ```

1. Vous pouvez utiliser un compte de stockage existant ou en créer un nouveau. Pour créer un compte de stockage, exécutez la cmdlet [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) :

   ```azurepowershell-interactive
   New-AzStorageAccount -ResourceGroupName myierg -AccountName myssdocsstorage -SkuName Standard_RAGRS -Location westus -EnableHttpsTrafficOnly $true
   ```

1. Pour obtenir la liste des emplacements vers lesquels vous pouvez expédier des disques, utilisez la cmdlet [Get-AzImportExportLocation](/powershell/module/az.importexport/get-azimportexportlocation) :

   ```azurepowershell-interactive
   Get-AzImportExportLocation
   ```

1. Utilisez la cmdlet `Get-AzImportExportLocation` avec le paramètre `Name` pour obtenir les emplacements de votre région :

   ```azurepowershell-interactive
   Get-AzImportExportLocation -Name westus
   ```

1. Exécutez l’exemple [New-AzImportExport](/powershell/module/az.importexport/new-azimportexport) suivant pour créer une tâche d’importation :

   ```azurepowershell-interactive
   $driveList = @(@{
     DriveId = '9CA995BA'
     BitLockerKey = '439675-460165-128202-905124-487224-524332-851649-442187'
     ManifestFile = '\\DriveManifest.xml'
     ManifestHash = '69512026C1E8D4401816A2E5B8D7420D'
     DriveHeaderHash = 'AZ31BGB1'
   })

   $Params = @{
      ResourceGroupName = 'myierg'
      Name = 'MyIEjob1'
      Location = 'westus'
      BackupDriveManifest = $true
      DiagnosticsPath = 'waimportexport'
      DriveList = $driveList
      JobType = 'Import'
      LogLevel = 'Verbose'
      ShippingInformationRecipientName = 'Microsoft Azure Import/Export Service'
      ShippingInformationStreetAddress1 = '3020 Coronado'
      ShippingInformationCity = 'Santa Clara'
      ShippingInformationStateOrProvince = 'CA'
      ShippingInformationPostalCode = '98054'
      ShippingInformationCountryOrRegion = 'USA'
      ShippingInformationPhone = '4083527600'
      ReturnAddressRecipientName = 'Gus Poland'
      ReturnAddressStreetAddress1 = '1020 Enterprise way'
      ReturnAddressCity = 'Sunnyvale'
      ReturnAddressStateOrProvince = 'CA'
      ReturnAddressPostalCode = '94089'
      ReturnAddressCountryOrRegion = 'USA'
      ReturnAddressPhone = '4085555555'
      ReturnAddressEmail = 'gus@contoso.com'
      ReturnShippingCarrierName = 'FedEx'
      ReturnShippingCarrierAccountNumber = '123456789'
      StorageAccountId = '/subscriptions/<SubscriptionId>/resourceGroups/myierg/providers/Microsoft.Storage/storageAccounts/myssdocsstorage'
   }
   New-AzImportExport @Params
   ```

   > [!TIP]
   > Au lieu de spécifier une adresse de messagerie pour un seul utilisateur, fournissez une adresse de groupe. Cela garantit que vous recevrez des notifications même si un administrateur s’en va.

1. Utilisez la cmdlet [Get-AzImportExport](/powershell/module/az.importexport/get-azimportexport) pour afficher toutes les tâches du groupe de ressources myierg :

   ```azurepowershell-interactive
   Get-AzImportExport -ResourceGroupName myierg
   ```

1. Pour mettre à jour votre travail ou l’annuler, exécutez la cmdlet [Update-AzImportExport](/powershell/module/az.importexport/update-azimportexport) :

   ```azurepowershell-interactive
   Update-AzImportExport -Name MyIEjob1 -ResourceGroupName myierg -CancelRequested
   ```

---

## <a name="step-3-optional-configure-customer-managed-key"></a>Étape 3 (facultative) : Configurer une clé gérée par le client

Ignorez cette étape et passez à l’étape suivante si vous souhaitez utiliser la clé gérée par Microsoft pour protéger vos clés BitLocker pour les lecteurs. Pour configurer votre propre clé afin de protéger la clé BitLocker, suivez les instructions de [Configurer les clés gérées par le client avec Azure Key Vault pour Azure Import/Export dans le portail Azure](storage-import-export-encryption-key-portal.md)

## <a name="step-4-ship-the-drives"></a>Étape 4 : Expédier les disques

[!INCLUDE [storage-import-export-ship-drives](../../../includes/storage-import-export-ship-drives.md)]

## <a name="step-5-update-the-job-with-tracking-information"></a>Étape 5 : Mettre à jour la tâche avec les informations de suivi

[!INCLUDE [storage-import-export-update-job-tracking](../../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-6-verify-data-upload-to-azure"></a>Étape 6 : Vérifier le chargement des données dans Azure

Surveillez le travail jusqu’à son achèvement. Une fois le travail terminé, vérifiez que vos données ont été chargées sur Azure. Ne supprimez les données locales qu’après avoir vérifié que le chargement a réussi.

## <a name="next-steps"></a>Étapes suivantes

* [Voir l’état de la tâche et des disques](storage-import-export-view-drive-status.md)
* [Passer en revue les exigences d’importation/exportation](storage-import-export-requirements.md)
