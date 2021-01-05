---
title: Utilisation d’Azure Import/Export pour transférer des données vers Azure Files | Microsoft Docs
description: Découvrez comment créer des tâches d’importation dans le portail Azure pour transférer des données vers Azure Files.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 10/29/2020
ms.author: alkohli
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 9d1d0f4b615bbf4cc7faf82d70a4de0b0157ed82
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96326350"
---
# <a name="use-azure-importexport-service-to-import-data-to-azure-files"></a>Utiliser le service Azure Import/Export pour importer des données dans Azure Files

Cet article fournit des instructions détaillées sur l’utilisation du service Azure Import/Export pour importer de manière sécurisée de grandes quantités de données dans Azure Files. Pour importer des données, le service vous demande d’expédier à un centre de données Azure des lecteurs de disque pris en charge contenant vos données.

Le service Import/Export prend en charge uniquement l’importation de fichiers Azure dans le Stockage Azure. L’exportation de fichiers Azure n’est pas prise en charge.

## <a name="prerequisites"></a>Prérequis

Avant de créer une tâche d’importation pour transférer des données dans Azure Files, examinez soigneusement la liste de prérequis suivante et complétez-la. Vous devez respecter les consignes suivantes :

- Avoir un abonnement Azure actif à utiliser avec le service Import/Export.
- Avoir au moins un compte de stockage Azure. Consultez la liste des [Comptes de stockage et types de stockage pris en charge pour le service Import/Export](storage-import-export-requirements.md). Pour plus d'informations sur la création d'un compte de stockage, consultez la page [Création d'un compte de stockage](storage-account-create.md).
- Avoir un nombre suffisant de disques correspondant aux [types pris en charge](storage-import-export-requirements.md#supported-disks).
- Avoir un système Windows exécutant une [Version de système d’exploitation prise en charge](storage-import-export-requirements.md#supported-operating-systems).
- [Téléchargez la version 2 de WAImportExport](https://aka.ms/waiev2) sur le système Windows. Décompressez le package dans le dossier par défaut : `waimportexport`. Par exemple : `C:\WaImportExport`.
- Dotez-vous d’un compte FedEx/DHL. Si vous souhaitez faire appel à un autre transporteur que FedEx/DHL, contactez l’équipe des opérations Azure Data Box à l’adresse `adbops@microsoft.com`.
    - Le compte doit être valide, doit avoir un solde et doit offrir des fonctionnalités de réexpédition.
    - Générez un numéro de suivi pour le travail d’exportation.
    - Chaque travail doit avoir un numéro de suivi distinct. Plusieurs travaux portant le même numéro de suivi ne sont pas pris en charge.
    - Si vous n’avez pas de compte de transporteur, accédez à :
        - [Créer un compte FedEx](https://www.fedex.com/en-us/create-account.html), ou
        - [Créer un compte DHL](http://www.dhl-usa.com/en/express/shipping/open_account.html).



## <a name="step-1-prepare-the-drives"></a>Étape 1 : Préparer les lecteurs

Cette étape génère un fichier journal. Le fichier journal stocke les informations de base comme le numéro de série du lecteur, la clé de chiffrement et les détails du compte de stockage.

Effectuez les étapes suivantes pour préparer les lecteurs.

1. Connectez vos lecteurs de disque au système Windows via des connecteurs SATA.
2. Créez un seul volume NTFS sur chaque lecteur. Attribuez une lettre de lecteur au volume. N’utilisez pas de points de montage.
3. Modifiez le fichier *dataset.csv* dans le dossier racine où réside l’outil. Selon que vous importez un fichier ou un dossier, ou les deux, ajoutez des entrées dans le fichier *dataset.csv* comme dans les exemples suivants.

   - **Pour importer un fichier** : Dans l’exemple suivant, les données à copier se trouvent sur le lecteur F:. Votre fichier *MyFile1.txt* est copié à la racine de *MyAzureFileshare1*. Si *MyAzureFileshare1* n’existe pas, il est créé dans le compte de stockage Azure. La structure de dossiers est conservée.

       ```
           BasePath,DstItemPathOrPrefix,ItemType,Disposition,MetadataFile,PropertiesFile
           "F:\MyFolder1\MyFile1.txt","MyAzureFileshare1/MyFile1.txt",file,rename,"None",None

       ```
   - **Pour importer un dossier** : Tous les fichiers et dossiers sous *MyFolder2* sont copiés de manière récursive dans le partage de fichiers. La structure de dossiers est conservée.

       ```
           "F:\MyFolder2\","MyAzureFileshare1/",file,rename,"None",None

       ```
     Plusieurs entrées correspondant aux dossiers ou fichiers importés peuvent être créées dans le même fichier.

       ```
           "F:\MyFolder1\MyFile1.txt","MyAzureFileshare1/MyFile1.txt",file,rename,"None",None
           "F:\MyFolder2\","MyAzureFileshare1/",file,rename,"None",None

       ```
     Vous trouverez plus d’informations sur la [préparation du fichier CSV du jeu de données ici](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import).


4. Modifiez le fichier *driveset.csv* dans le dossier racine où réside l’outil. Ajoutez des entrées dans le fichier *driveset.csv* comme dans les exemples suivants. Le fichier driveset contient la liste des disques et des lettres de lecteur correspondantes pour que l’outil puisse choisir correctement les disques à préparer.

    Cet exemple suppose que deux disques sont attachés et que les volumes NTFS de base G:\ et H:\ sont créés. H:\ n’est pas chiffré, contrairement à G: qui est déjà chiffré. L’outil formate et chiffre le disque qui héberge H:\ uniquement (et non G:\).

   - **Pour un disque qui n’est pas chiffré** : Spécifiez *Encrypt* pour activer le chiffrement BitLocker sur le disque.

       ```
       DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
       H,Format,SilentMode,Encrypt,
       ```

   - **Pour un disque qui est déjà chiffré** : Spécifiez *AlreadyEncrypted* et fournissez la clé BitLocker.

       ```
       DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
       G,AlreadyFormatted,SilentMode,AlreadyEncrypted,060456-014509-132033-080300-252615-584177-672089-411631
       ```

     Plusieurs entrées correspondant aux différents lecteurs peuvent être créées dans le même fichier. Vous trouverez plus d’informations sur la [préparation du fichier CSV du jeu de disques ici](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import).

5. Utilisez l’option `PrepImport` pour copier et préparer des données sur le lecteur de disque. Pendant la première session de copie, pour copier des répertoires et/ou des fichiers dans une nouvelle session de copie, exécutez la commande suivante :

    ```cmd
    .\WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>]/DataSet:<dataset.csv>
    ```

   Voici un exemple d’importation.

    ```cmd
    .\WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:************* /InitialDriveSet:driveset.csv /DataSet:dataset.csv /logdir:C:\logs
    ```

6. Un fichier journal avec le nom fourni incluant le paramètre `/j:` est créé pour chaque exécution de la ligne de commande. Chaque lecteur que vous préparez a un fichier journal qui doit être chargé au moment de la création de la tâche d’importation. Les lecteurs sans fichier journal ne sont pas traités.

    > [!IMPORTANT]
    > - Ne modifiez ni les données sur les lecteurs de disque ni le fichier journal après la préparation du disque.

Pour plus d’exemples, accédez à [Exemples de fichiers journaux](#samples-for-journal-files).

## <a name="step-2-create-an-import-job"></a>Étape 2 : Créer une tâche d’importation

### <a name="portal"></a>[Portail](#tab/azure-portal)

Effectuez les étapes suivantes pour créer une tâche d’importation dans le portail Azure.
1. Connectez-vous sur https://portal.azure.com/.
2. Accédez à **Tous les services > Stockage > Tâches d’importation/exportation**.

    ![Accédez à Importation/exportation](./media/storage-import-export-data-to-blobs/import-to-blob1.png)

3. Cliquez sur **Créer une tâche d’importation/exportation**.

    ![Cliquer sur Tâche d’importation/exportation](./media/storage-import-export-data-to-blobs/import-to-blob2.png)

4. Dans **De base** :

    - Sélectionnez **Importer dans Azure**.
    - Indiquez un nom décrivant le travail d’importation. Utilisez ce nom pour suivre les tâches en cours et effectuées.
        -  Le nom peut contenir uniquement des lettres minuscules, des chiffres, des tirets et des traits de soulignement.
        -  Le nom doit commencer par une lettre et ne doit pas contenir d’espaces.
    - Sélectionnez un abonnement.
    - Sélectionnez un groupe de ressources.

        ![Créer une tâche d’importation - Étape 1](./media/storage-import-export-data-to-blobs/import-to-blob3.png)

3. Dans **Détails de la tâche** :

    - Chargez les fichiers journaux que vous avez créés précédemment dans [Étape 1 : Préparer les lecteurs](#step-1-prepare-the-drives).
    - Sélectionnez le compte de stockage dans lequel les données doivent être importées.
    - L’emplacement de remise est automatiquement rempli en fonction de la région du compte de stockage sélectionné.

       ![Créer une tâche d’importation - Étape 2](./media/storage-import-export-data-to-blobs/import-to-blob4.png)

4. Dans **Informations de réexpédition** :

    - Sélectionnez le transporteur dans la liste déroulante. Si vous souhaitez utiliser un autre transporteur que FedEx/DHL, choisissez une option existante dans la liste déroulante. Contactez l’équipe des opérations Azure Data Box à l’adresse `adbops@microsoft.com` pour lui indiquer le nom du transporteur auquel vous envisagez de faire appel.
    - Entrez un numéro de compte de transporteur valide que vous avez créé pour ce transporteur. Microsoft utilise ce compte pour renvoyer les lecteurs une fois la tâche d’importation terminée.
    - Indiquez le nom d’un contact, le numéro de téléphone, l’e-mail, l’adresse, la ville, le code postal, l’état/la province et le pays/la région, puis vérifiez que ces informations sont complètes et valides.

        > [!TIP]
        > Au lieu de spécifier une adresse de messagerie pour un seul utilisateur, fournissez une adresse de groupe. Cela garantit que vous recevrez des notifications même si un administrateur s’en va.

       ![Créer une tâche d'importation - Étape 3](./media/storage-import-export-data-to-blobs/import-to-blob5.png)


5. Dans le **Récapitulatif** :

    - Indiquez l’adresse du centre de données Azure pour réexpédier les disques à Azure. Assurez-vous que le nom du travail et l’adresse complète sont mentionnés sur l’étiquette d’expédition.
    - Cliquez sur **OK** pour créer la tâche d’exportation.

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
    az storage account create -resource-group myierg -name myssdocsstorage --https-only
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

## <a name="step-3-ship-the-drives-to-the-azure-datacenter"></a>Étape 3 : Expédier les lecteurs au centre de données Azure

[!INCLUDE [storage-import-export-ship-drives](../../../includes/storage-import-export-ship-drives.md)]

## <a name="step-4-update-the-job-with-tracking-information"></a>Étape 4 : Mettre à jour la tâche avec les informations de suivi

[!INCLUDE [storage-import-export-update-job-tracking](../../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-5-verify-data-upload-to-azure"></a>Étape 5 : vérifier le chargement des données sur Azure

Surveillez le travail jusqu’à son achèvement. Une fois le travail terminé, vérifiez que vos données ont été chargées sur Azure. Ne supprimez les données locales qu’après avoir vérifié que le chargement a réussi.

## <a name="samples-for-journal-files"></a>Exemples de fichiers journaux

Pour **ajouter des disques**, créez un fichier driveset et exécutez la commande indiquée ci-dessous.

Pour les sessions de copie suivantes sur d’autres disques non spécifiés dans le fichier *InitialDriveSet.csv*, spécifiez un nouveau fichier driveset *.csv* et indiquez-le comme valeur du paramètre `AdditionalDriveSet`. Utilisez le **même nom de fichier journal** et fournissez un **nouvel ID de session**. Le format du fichier CSV du paramètre AdditionalDriveSet est identique au format de celui du paramètre InitialDriveSet.

```cmd
WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /AdditionalDriveSet:<driveset.csv>
```

Voici un exemple d’importation.

```cmd
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#3  /AdditionalDriveSet:driveset-2.csv
```


Pour ajouter des données au même ficher driveset, utilisez la commande PrepImport pour les sessions de copie suivantes de fichiers/répertoires supplémentaires.

Pour les sessions de copie suivantes sur les mêmes disques durs spécifiés dans le fichier *InitialDriveset.csv*, spécifiez le **même nom de fichier journal** et indiquez un **nouvel ID de session**. Il n’est pas nécessaire d’indiquer la clé du compte de stockage.

```cmd
WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] DataSet:<dataset.csv>
```

Voici un exemple d’importation.

```cmd
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv
```

## <a name="next-steps"></a>Étapes suivantes

* [Voir l’état de la tâche et des disques](storage-import-export-view-drive-status.md)
* [Passer en revue les exigences d’importation/exportation](storage-import-export-requirements.md)