---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: 8a442b5de7a256dd8bcf47da741bd41894709c3a
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95561335"
---
Les problèmes de diagnostic avec un service cloud Microsoft Azure nécessitent la collecte des fichiers journaux du service sur les machines virtuelles au fur et à mesure que les problèmes surviennent. Vous pouvez utiliser l’extension AzureLogCollector à la demande pour exécuter une collecte unique de journaux d’activité à partir d’une ou de plusieurs machines virtuelles de service cloud (à partir des rôles web et de travail) et transférer les fichiers collectés dans un compte de stockage Azure, le tout sans connexion à distance à l’une des machines virtuelles.

> [!NOTE]
> Vous trouverez des descriptions pour la plupart des informations journalisées dans https://blogs.msdn.microsoft.com/kwill/2013/08/09/windows-azure-paas-compute-diagnostics-data/
> 
> 

Il existe deux modes de collecte dépendant des types de fichiers à recueillir.

* **Journaux d’activité des agents invités Azure uniquement (GA)** . Ce mode de collecte inclut tous les journaux d’activité liés aux agents invités d’Azure et d’autres composants Azure.
* **Tous les journaux d’activité (Complet)** . Ce mode de collecte recueille tous les fichiers en mode GA, plus les suivants :
  
  * journaux des événements système et d’application
  * Journaux d’activité d’erreurs HTTP
  * Journaux d’activité IIS
  * Fichiers journaux d’activité d’installation
  * autres journaux d’activité système.

Dans les deux modes de collecte, les dossiers de collecte de données supplémentaires peuvent être spécifiés à l’aide d’une collection de la structure suivante :

* **Nom** : nom de la collection, utilisé comme nom du sous-dossier dans le fichier zip contenant les fichiers collectés.
* **Emplacement** : chemin d’accès au dossier sur la machine virtuelle où se trouvent les fichiers à collecter.
* **Modèle de recherche**: modèle des noms de fichiers à collecter. La valeur par défaut est « \* ».
* **Récursif** : si les fichiers à collecter se trouvent de manière récursive à l’emplacement spécifié.

## <a name="prerequisites"></a>Conditions préalables requises

[!INCLUDE [updated-for-az](./updated-for-az.md)]

* Disposer d’un compte de stockage comme extension pour enregistrer les fichiers zip générés
* Azure PowerShell. Pour obtenir des instructions d’installation, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="add-the-extension"></a>Ajouter l’extension
Vous pouvez utiliser les applets de commande [Microsoft Azure PowerShell](/previous-versions/azure/dn495240(v=azure.100)) ou les [API REST Gestion des services](/previous-versions/azure/ee460799(v=azure.100)) pour ajouter l’extension AzureLogCollector.

Pour les services cloud, l’applet de commande Powershell de Azure existante **Set-AzureServiceExtension**, peut être utilisée pour activer l’extension sur les instances de rôle de service cloud. Chaque fois que cette extension est activée via cette applet de commande, la collecte des journaux est déclenchée sur les instances de rôle choisies pour les rôles sélectionnés.

Pour les machines virtuelles, l’applet de commande Azure Powershell existante **Set-AzureVMExtension** peut être utilisée pour activer l’extension sur des machines virtuelles. À chaque fois que cette extension est activée via les applets de commande, la collecte des journaux est déclenchée sur chaque instance.

En interne, cette extension utilise les configurations PublicConfiguration et PrivateConfiguration basées sur JSON. Voici la disposition d’un exemple de JSON pour la configuration publique et privée.

### <a name="publicconfiguration"></a>PublicConfiguration

```json
{
    "Instances":  "*",
    "Mode":  "Full",
    "SasUri":  "SasUri to your storage account with sp=wl",
    "AdditionalData":
    [
      {
              "Name":  "StorageData",
              "Location":  "%roleroot%storage",
              "SearchPattern":  "*.*",
              "Recursive":  "true"
      },
      {
            "Name":  "CustomDataFolder2",
            "Location":  "c:\customFolder",
            "SearchPattern":  "*.log",
            "Recursive":  "false"
      },
    ]
}
```

### <a name="privateconfiguration"></a>PrivateConfiguration

```json
{

}
```

> [!NOTE]
> Cette extension n’a pas besoin de **privateConfiguration**. Vous pouvez simplement fournir une structure vide pour l’argument **–PrivateConfiguration** .
> 
> 

Vous pouvez exécuter l’une des deux étapes suivantes pour ajouter AzureLogCollector à une ou plusieurs instances d’un Service Cloud ou d’une machine virtuelle des rôles sélectionnés, ce qui déclenche les collectes sur chaque machine virtuelle à exécuter et envoyer aux fichiers recueillis à un compte Azure spécifié.

## <a name="adding-as-a-service-extension"></a>Ajout d’une Extension de service
1. Suivez les instructions pour connecter Azure PowerShell à votre abonnement.
2. Spécifiez le nom de service, l’emplacement, les rôles et les instances de rôle auxquels vous souhaitez ajouter et activer l’extension AzureLogCollector.

   ```powershell
   #Specify your cloud service name
   $ServiceName = 'extensiontest2'

   #Specify the slot. 'Production' or 'Staging'
   $slot = 'Production'

   #Specified the roles on which the extension will be installed and enabled
   $roles = @("WorkerRole1","WebRole1")

   #Specify the instances on which extension will be installed and enabled.  Use wildcard * for all instances
   $instances = @("*")

   #Specify the collection mode, "Full" or "GA"
   $mode = "GA"
   ```

3. Spécifiez le dossier de données supplémentaires pour lequel les fichiers seront collectés (cette étape est facultative).

   ```powershell
   #add one location
   $a1 = New-Object PSObject

   $a1 | Add-Member -MemberType NoteProperty -Name "Name" -Value "StorageData"
   $a1 | Add-Member -MemberType NoteProperty -Name "SearchPattern" -Value "*"
   $a1 | Add-Member -MemberType NoteProperty -Name "Location" -Value "%roleroot%storage"  #%roleroot% is normally E: or F: drive
   $a1 | Add-Member -MemberType NoteProperty -Name "Recursive" -Value "true"

   $AdditionalDataList+= $a1
   #more locations can be added....
   ```

   > [!NOTE]
   > Vous pouvez utiliser le jeton `%roleroot%` pour spécifier le lecteur racine de rôle, car il n’utilise pas un lecteur fixe.
   > 
   > 
4. Fournissez le nom du compte de stockage Azure et la clé vers laquelle les fichiers recueillis seront téléchargés.

   ```powershell
   $StorageAccountName = 'YourStorageAccountName'
   $StorageAccountKey  = 'YourStorageAccountKey'
   ```

5. Appelez le SetAzureServiceLogCollector.ps1 (inclus à la fin de l’article) comme suit pour activer l’extension AzureLogCollector pour un Service Cloud. Une fois l’exécution achevée, vous trouverez le fichier téléchargé sous `https://YourStorageAccountName.blob.core.windows.net/vmlogs`

   ```powershell
   .\SetAzureServiceLogCollector.ps1 -ServiceName YourCloudServiceName  -Roles $roles  -Instances $instances –Mode $mode -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey -AdditionDataLocationList $AdditionalDataList
   ```

Voici la définition des paramètres transmis au script. (Également copié ci-dessous.)

```powershell
[CmdletBinding(SupportsShouldProcess = $true)]

param (
  [Parameter(Mandatory=$true)]
  [string]   $ServiceName,

  [Parameter(Mandatory=$false)]
  [string[]] $Roles ,

  [Parameter(Mandatory=$false)]
  [string[]] $Instances,

  [Parameter(Mandatory=$false)]
  [string]   $Slot = 'Production',

  [Parameter(Mandatory=$false)]
  [string]   $Mode = 'Full',

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountName,

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountKey,

  [Parameter(Mandatory=$false)]
  [PSObject[]] $AdditionDataLocationList = $null
)
```

* **ServiceName**: nom de votre service cloud.
* **Rôles**: une liste de rôles, tels que « WebRole1 » ou « WorkerRole1 ».
* **Instances**: liste de noms des instances de rôle séparés par des virgules, utilisez la chaîne de caractères génériques (« * ») pour toutes les instances de rôle.
* **Emplacement**: nom de l’emplacement. « Production » ou « Intermédiaire ».
* **Mode**: mode de collecte. « Complet » ou « GA ».
* **StorageAccountName**: nom du compte de stockage Azure pour le stockage des données recueillies.
* **StorageAccountKey**: nom de clé de compte de stockage Azure.
* **AdditionalDataLocationList**: liste obéissant à la structure suivante :

  ```powershell
  {
    String Name,
    String Location,
    String SearchPattern,
    Bool   Recursive
  }
  ```

## <a name="adding-as-a-vm-extension"></a>Ajout d’une extension de machine virtuelle
Suivez les instructions pour connecter Azure PowerShell à votre abonnement.

1. Spécifiez le nom du service, la machine virtuelle et le mode de collecte.

   ```powershell
   #Specify your cloud service name
   $ServiceName = 'YourCloudServiceName'

   #Specify the VM name
   $VMName = "'YourVMName'"

   #Specify the collection mode, "Full" or "GA"
   $mode = "GA"

   Specify the additional data folder for which files will be collected (this step is optional).

   #add one location
   $a1 = New-Object PSObject

   $a1 | Add-Member -MemberType NoteProperty -Name "Name" -Value "StorageData"
   $a1 | Add-Member -MemberType NoteProperty -Name "SearchPattern" -Value "*"
   $a1 | Add-Member -MemberType NoteProperty -Name "Location" -Value "%roleroot%storage"  #%roleroot% is normally E: or F: drive
   $a1 | Add-Member -MemberType NoteProperty -Name "Recursive" -Value "true"

   $AdditionalDataList+= $a1
        #more locations can be added....
   ```
  
2. Fournissez le nom du compte de stockage Azure et la clé vers laquelle les fichiers recueillis seront téléchargés.

   ```powershell
   $StorageAccountName = 'YourStorageAccountName'
   $StorageAccountKey  = 'YourStorageAccountKey'
   ```

3. Appelez le SetAzureVMLogCollector.ps1 (inclus à la fin de l’article) comme suit pour activer l’extension AzureLogCollector pour Service Cloud. Une fois l’exécution achevée, vous trouverez le fichier téléchargé sous `https://YourStorageAccountName.blob.core.windows.net/vmlogs`

Voici la définition des paramètres transmis au script. (Également copié ci-dessous.)

```powershell
[CmdletBinding(SupportsShouldProcess = $true)]

param (
  [Parameter(Mandatory=$true)]
  [string]   $ServiceName,

  [Parameter(Mandatory=$false)]
  [string] $VMName ,

  [Parameter(Mandatory=$false)]
  [string]   $Mode = 'Full',

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountName,

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountKey,

  [Parameter(Mandatory=$false)]
  [PSObject[]] $AdditionDataLocationList = $null
)
```

* **ServiceName**: nom de votre service cloud.
* **VMName** : nom de la machine virtuelle.
* **Mode**: mode de collecte. « Complet » ou « GA ».
* **StorageAccountName**: nom du compte de stockage Azure pour le stockage des données recueillies.
* **StorageAccountKey**: nom de clé de compte de stockage Azure.
* **AdditionalDataLocationList**: liste obéissant à la structure suivante :

  ```
  {
    String Name,
    String Location,
    String SearchPattern,
    Bool   Recursive
  }
  ```

## <a name="extention-powershell-script-files"></a>Fichiers script PowerShell d’extension
### <a name="setazureservicelogcollectorps1"></a>SetAzureServiceLogCollector.ps1

```powershell
[CmdletBinding(SupportsShouldProcess = $true)]

param (
  [Parameter(Mandatory=$true)]
  [string]   $ServiceName,

  [Parameter(Mandatory=$false)]
  [string[]] $Roles ,

  [Parameter(Mandatory=$false)]
  [string[]] $Instances = '*',

  [Parameter(Mandatory=$false)]
  [string]   $Slot = 'Production',

  [Parameter(Mandatory=$false)]
  [string]   $Mode = 'Full',

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountName,

  [Parameter(Mandatory=$false)]
  [string]   $StorageAccountKey,

  [Parameter(Mandatory=$false)]
  [PSObject[]] $AdditionDataLocationList = $null
)

$publicConfig = New-Object PSObject

if ($Instances -ne $null -and $Instances.Count -gt 0)  #Instances should be separated by ,
{
  $instanceText = $Instances[0]
  for ($i = 1;$i -lt $Instances.Count;$i++)
  {
    $instanceText = $instanceText+ "," + $Instances[$i]
  }
  $publicConfig | Add-Member -MemberType NoteProperty -Name "Instances" -Value $instanceText
}
else  #For all instances if not specified.  The value should be a space or *
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "Instances" -Value " "
}

if ($Mode -ne $null )
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "Mode" -Value $Mode
}
else
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "Mode" -Value "Full"
}

#
#we need to get the Sasuri from StorageAccount and containers
#
$context = New-AzStorageContext -Protocol https -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

$ContainerName = "azurelogcollectordata"
$existingContainer = Get-AzStorageContainer -Context $context |  Where-Object { $_.Name -like $ContainerName}
if ($existingContainer -eq $null)
{
  "Container ($ContainerName) doesn't exist. Creating it now.."
  New-AzStorageContainer -Context $context -Name $ContainerName -Permission off
}

$ExpiryTime =  [DateTime]::Now.AddMinutes(120).ToString("o")
$SasUri = New-AzStorageContainerSASToken -ExpiryTime $ExpiryTime -FullUri -Name $ContainerName -Permission rwl -Context $context
$publicConfig | Add-Member -MemberType NoteProperty -Name "SasUri" -Value $SasUri

#
#Add AdditionalData to collect data from additional folders
#
if ($AdditionDataLocationList -ne $null )
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "AdditionalData" -Value $AdditionDataLocationList
}

#
# Convert it to JSON format
#
$publicConfigJSON = $publicConfig | ConvertTo-Json
"publicConfig is:  $publicConfigJSON"

#we just provide an empty privateConfig object
$privateconfig = "{
}"

if ($Roles -ne $null)
{
  Set-AzureServiceExtension -Service $ServiceName -Slot $Slot -Role $Roles -ExtensionName 'AzureLogCollector' -ProviderNamespace Microsoft.WindowsAzure.Compute -PublicConfiguration $publicConfigJSON -PrivateConfiguration $privateconfig -Version 1.0 -Verbose
}
else
{
  Set-AzureServiceExtension -Service $ServiceName -Slot $Slot  -ExtensionName 'AzureLogCollector' -ProviderNamespace Microsoft.WindowsAzure.Compute -PublicConfiguration $publicConfigJSON -PrivateConfiguration $privateconfig -Version 1.0 -Verbose
}

#
#This is an optional step: generate a sasUri to the container so it can be shared with other people if needed.
#
$SasExpireTime = [DateTime]::Now.AddMinutes(120).ToString("o")
$SasUri = New-AzStorageContainerSASToken -ExpiryTime $ExpiryTime -FullUri -Name $ContainerName -Permission rl -Context $context
$SasUri = $SasUri + "&restype=container&comp=list"
Write-Output "The container for uploaded file can be accessed using this link:`r`n$sasuri"
```

### <a name="setazurevmlogcollectorps1"></a>SetAzureVMLogCollector.ps1

```powershell
[CmdletBinding(SupportsShouldProcess = $true)]

param (
              [Parameter(Mandatory=$true)]
              [string]   $ServiceName,

              [Parameter(Mandatory=$false)]
              [string] $VMName ,

              [Parameter(Mandatory=$false)]
              [string]   $Mode = 'Full',

              [Parameter(Mandatory=$false)]
              [string]   $StorageAccountName,

              [Parameter(Mandatory=$false)]
              [string]   $StorageAccountKey,

              [Parameter(Mandatory=$false)]
              [PSObject[]] $AdditionDataLocationList = $null
        )

$publicConfig = New-Object PSObject
$publicConfig | Add-Member -MemberType NoteProperty -Name "Instances" -Value "*"

if ($Mode -ne $null )
{
    $publicConfig | Add-Member -MemberType NoteProperty -Name "Mode" -Value $Mode
}
else
{
    $publicConfig | Add-Member -MemberType NoteProperty -Name "Mode" -Value "Full"
}

#
#we need to get the Sasuri from StorageAccount and containers
#
$context = New-AzStorageContext -Protocol https -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

$ContainerName = "azurelogcollectordata"
$existingContainer = Get-AzStorageContainer -Context $context |  Where-Object { $_.Name -like $ContainerName}
if ($existingContainer -eq $null)
{
    "Container ($ContainerName) doesn't exist. Creating it now.."
    New-AzStorageContainer -Context $context -Name $ContainerName -Permission off
}

$ExpiryTime =  [DateTime]::Now.AddMinutes(90).ToString("o")
$SasUri = New-AzStorageContainerSASToken -ExpiryTime $ExpiryTime -FullUri -Name $ContainerName -Permission rwl -Context $context
$publicConfig | Add-Member -MemberType NoteProperty -Name "SasUri" -Value $SasUri

#
#Add AdditionalData to collect data from additional folders
#
if ($AdditionDataLocationList -ne $null )
{
  $publicConfig | Add-Member -MemberType NoteProperty -Name "AdditionalData" -Value $AdditionDataLocationList
}

#
# Convert it to JSON format
#
$publicConfigJSON = $publicConfig | ConvertTo-Json

Write-Output "PublicConfiguration is: \r\n$publicConfigJSON"

#
#we just provide an empty privateConfig object
#
$privateconfig = "{
}"

if ($VMName -ne $null )
{
      $VM = Get-AzureVM -ServiceName $ServiceName -Name $VMName
      $VM.VM.OSVirtualHardDisk.OS

      if ($VM.VM.OSVirtualHardDisk.OS -like '*Windows*')
      {
            Set-AzureVMExtension -VM $VM -ExtensionName "AzureLogCollector" -Publisher Microsoft.WindowsAzure.Compute -PublicConfiguration $publicConfigJSON -PrivateConfiguration $privateconfig -Version 1.* | Update-AzureVM -Verbose

            #
            #We will check the VM status to find if operation by extension has been completed or not. The completion of the operation,either succeed or fail, can be indicated by
            #the presence of SubstatusList field.
            #
            $Completed = $false
            while ($Completed -ne $true)
            {
                    $VM = Get-AzureVM -ServiceName $ServiceName -Name $VMName
                    $status = $VM.ResourceExtensionStatusList | Where-Object {$_.HandlerName -eq "Microsoft.WindowsAzure.Compute.AzureLogCollector"}

                    if ( ($status.Code -ne 0) -and ($status.Status -like '*error*'))
                    {
                        Write-Output "Error status is returned: $($Status.ExtensionSettingStatus.FormattedMessage.Message)."
                          $Completed = $true
                    }
                    elseif (($status.ExtensionSettingStatus.SubstatusList -eq $null -or $status.ExtensionSettingStatus.SubstatusList.Count -lt 1))
                    {
                          $Completed = $false
                          Write-Output "Waiting for operation to complete..."
                    }
                    else
                    {
                          $Completed = $true
                          Write-Output "Operation completed."

                    $UploadedFileUri = $Status.ExtensionSettingStatus.SubStatusList[0].FormattedMessage.Message
                          $blob = New-Object Microsoft.WindowsAzure.Storage.Blob.CloudBlockBlob($UploadedFileUri)

                  #
                        # This is an optional step:  For easier access to the file, we can generate a read-only SasUri directly to the file
                          #
                          $ExpiryTimeRead =  [DateTime]::Now.AddMinutes(120).ToString("o")
                          $ReadSasUri = New-AzStorageBlobSASToken -ExpiryTime $ExpiryTimeRead  -FullUri  -Blob  $blob.name -Container $blob.Container.Name -Permission r -Context $context

                        Write-Output "The uploaded file can be accessed using this link: $ReadSasUri"

                          #
                          #This is an optional step:  Remove the extension after we are done
                          #
                          Get-AzureVM -ServiceName $ServiceName -Name $VMName | Set-AzureVMExtension -Publisher Microsoft.WindowsAzure.Compute -ExtensionName "AzureLogCollector" -Version 1.* -Uninstall | Update-AzureVM -Verbose

                    }
                    Start-Sleep -s 5
            }
      }
      else
      {
          Write-Output "VM OS Type is not Windows, the extension cannot be enabled"
      }

}
else
{
  Write-Output "VM name is not specified, the extension cannot be enabled"
}
```

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez maintenant examiner ou copier vos journaux d’activité à partir d’un emplacement simple.