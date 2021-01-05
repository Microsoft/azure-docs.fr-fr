---
title: Migrer un certificat TDE - instance managée
description: Migrer un certificat assurant la protection de la clé de chiffrement d’une base de données à l’aide de Transparent Data Encryption vers Azure SQL Managed Instance
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: MladjoA
ms.author: mlandzic
ms.reviewer: sstein, jovanpop
ms.date: 07/21/2020
ms.openlocfilehash: c465da3d5d812ea7e811cbe59318122700c6e786
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97824668"
---
# <a name="migrate-a-certificate-of-a-tde-protected-database-to-azure-sql-managed-instance"></a>Migrer vers Azure SQL Managed Instance un certificat d’une base de données protégée par TDE
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Quand vous migrez une base de données protégée par [Transparent Data Encryption (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) vers Azure SQL Managed Instance à l’aide de l’option de restauration native, le certificat correspondant de l’instance SQL Server doit être migré avant la restauration de la base de données. Cet article vous guide dans le processus de migration manuelle du certificat vers Azure SQL Managed Instance :

> [!div class="checklist"]
>
> * Exporter le certificat dans un fichier Personal Information Exchange (.pfx)
> * Extraire le certificat d’un fichier vers une chaîne en base 64
> * Le charger à l’aide d’une applet de commande PowerShell

Pour prendre connaissance d’une autre possibilité de migration fluide d’une base de données protégée par TDE et d’un certificat correspondant à l’aide d’un service complètement managé, consultez [Comment migrer votre base de données locale vers Azure SQL Managed Instance à l’aide d’Azure Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).

> [!IMPORTANT]
> Un certificat migré permet de restaurer uniquement la base de données protégée par TDE. Peu de temps après la restauration, le certificat migré est remplacé par un autre protecteur, soit un certificat géré par le service, soit une clé asymétrique du coffre de clés, en fonction du type de TDE défini sur l’instance.

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes indiquées dans cet article, vous avez besoin des éléments requis suivants :

* L’outil en ligne de commande [Pvk2pfx](/windows-hardware/drivers/devtest/pvk2pfx) installé sur le serveur local ou un autre ordinateur avec accès au certificat exporté dans un fichier. Cet outil fait partie du kit [Enterprise Windows Driver Kit](/windows-hardware/drivers/download-the-wdk), un environnement de ligne de commande autonome.
* [Windows PowerShell](/powershell/scripting/install/installing-windows-powershell) version 5.0 ou supérieure installé.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Vérifiez que vous disposez des éléments suivants :

* Module Azure PowerShell [installé et mis à jour](/powershell/azure/install-az-ps).
* [Module Az.Sql](https://www.powershellgallery.com/packages/Az.Sql).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Managed Instance, mais tous les développements à venir sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRM.

Exécutez les commandes suivantes dans PowerShell pour installer/mettre à jour le module :

```azurepowershell
Install-Module -Name Az.Sql
Update-Module -Name Az.Sql
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

* * *

## <a name="export-the-tde-certificate-to-a-pfx-file"></a>Exporter le certificat TDE vers un fichier .pfx

Le certificat peut être exporté directement à partir de l’instance du serveur SQL Server source ou du magasin de certificats s’il y est conservé.

### <a name="export-the-certificate-from-the-source-sql-server-instance"></a>Exporter le certificat à partir de l’instance du serveur SQL Server source

Pour exporter le certificat avec SQL Server Management Studio et le convertir au format .pfx, procédez comme suit. Dans les étapes indiquées ci-dessous, les noms génériques *TDE_Cert* et *full_path* sont utilisés comme noms de certificat et de fichier et comme chemins d’accès. Il convient de les remplacer par les noms réels.

1. Dans SSMS, ouvrez une nouvelle fenêtre de requête et connectez-vous à l’instance du serveur SQL Server source.

1. Utilisez le script suivant pour lister les bases de données protégées par TDE et obtenir le nom du certificat protégeant le chiffrement de la base de données à migrer :

   ```sql
   USE master
   GO
   SELECT db.name as [database_name], cer.name as [certificate_name]
   FROM sys.dm_database_encryption_keys dek
   LEFT JOIN sys.certificates cer
   ON dek.encryptor_thumbprint = cer.thumbprint
   INNER JOIN sys.databases db
   ON dek.database_id = db.database_id
   WHERE dek.encryption_state = 3
   ```

   ![Liste des certificats TDE](./media/tde-certificate-migrate/onprem-certificate-list.png)

1. Exécutez le script suivant pour exporter le certificat vers une paire de fichiers (.cer et .pvk), en conservant les informations de clés publiques et privées :

   ```sql
   USE master
   GO
   BACKUP CERTIFICATE TDE_Cert
   TO FILE = 'c:\full_path\TDE_Cert.cer'
   WITH PRIVATE KEY (
     FILE = 'c:\full_path\TDE_Cert.pvk',
     ENCRYPTION BY PASSWORD = '<SomeStrongPassword>'
   )
   ```

   ![Sauvegarder un certificat TDE](./media/tde-certificate-migrate/backup-onprem-certificate.png)

1. Utilisez la console PowerShell pour copier les informations de certificat d’une paire de fichiers qui viennent d’être créés vers un fichier .pfx à l’aide de l’outil Pvk2Pfx :

   ```cmd
   .\pvk2pfx -pvk c:/full_path/TDE_Cert.pvk  -pi "<SomeStrongPassword>" -spc c:/full_path/TDE_Cert.cer -pfx c:/full_path/TDE_Cert.pfx
   ```

### <a name="export-the-certificate-from-a-certificate-store"></a>Exporter le certificat à partir d’un magasin de certificats

Si le certificat est conservé dans le magasin de certificats de l’ordinateur local SQL Server, vous pouvez l’exporter en procédant comme suit :

1. Ouvrez la console PowerShell et exécutez la commande suivante pour ouvrir le composant logiciel enfichable Certificats de Microsoft Management Console :

   ```cmd
   certlm
   ```

2. Dans le composant logiciel enfichable MMC Certificats, développez le chemin d’accès Personnel > Certificats pour afficher la liste des certificats.

3. Cliquez avec le bouton droit sur le certificat, puis cliquez sur **Exporter**.

4. Suivez l’Assistant pour exporter le certificat et la clé privée dans un format .pfx.

## <a name="upload-the-certificate-to-azure-sql-managed-instance-using-an-azure-powershell-cmdlet"></a>Charger le certificat sur Azure SQL Managed Instance à l’aide d’une applet de commande Azure PowerShell

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Commencez par les étapes de préparation dans PowerShell :

   ```azurepowershell
   # import the module into the PowerShell session
   Import-Module Az
   # connect to Azure with an interactive dialog for sign-in
   Connect-AzAccount
   # list subscriptions available and copy id of the subscription target the managed instance belongs to
   Get-AzSubscription
   # set subscription for the session
   Select-AzSubscription <subscriptionId>
   ```

2. Une fois que toutes les étapes de préparation sont effectuées, exécutez les commandes suivantes pour charger le certificat encodé en base 64 vers l’instance gérée cible :

   ```azurepowershell
   $fileContentBytes = Get-Content 'C:/full_path/TDE_Cert.pfx' -AsByteStream
   $base64EncodedCert = [System.Convert]::ToBase64String($fileContentBytes)
   $securePrivateBlob = $base64EncodedCert  | ConvertTo-SecureString -AsPlainText -Force
   $password = "<password>"
   $securePassword = $password | ConvertTo-SecureString -AsPlainText -Force
   Add-AzSqlManagedInstanceTransparentDataEncryptionCertificate -ResourceGroupName "<resourceGroupName>" `
       -ManagedInstanceName "<managedInstanceName>" -PrivateBlob $securePrivateBlob -Password $securePassword
   ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Vous devez d’abord [configurer un coffre de clés Azure](../../key-vault/general/manage-with-cli2.md) avec votre fichier *.pfx*.

1. Commencez par les étapes de préparation dans PowerShell :

   ```azurecli
   # connect to Azure with an interactive dialog for sign-in
   az login

   # list subscriptions available and copy id of the subscription target the managed instance belongs to
   az account list

   # set subscription for the session
   az account set --subscription <subscriptionId>
   ```

1. Une fois que toutes les étapes de préparation sont effectuées, exécutez les commandes suivantes pour charger le certificat encodé en base 64 sur l’instance managée cible :

   ```azurecli
   az sql mi tde-key set --server-key-type AzureKeyVault --kid "<keyVaultId>" `
       --managed-instance "<managedInstanceName>" --resource-group "<resourceGroupName>"
   ```

* * *

Le certificat est désormais disponible dans l’instance managée spécifiée, et la sauvegarde de la base de données correspondante protégée par TDE peut être restaurée avec succès.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à migrer un certificat qui protège la clé de chiffrement d’une base de données avec TDE d’une instance d’un serveur SQL Server local ou IaaS vers Azure SQL Managed Instance.

Consultez [Restaurer une sauvegarde de base de données dans Azure SQL Managed Instance](restore-sample-database-quickstart.md) pour apprendre à restaurer une sauvegarde de base de données dans Azure SQL Managed Instance.