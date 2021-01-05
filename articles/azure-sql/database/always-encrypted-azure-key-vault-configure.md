---
title: Configurer Always Encrypted à l’aide d’Azure Key Vault
description: Ce didacticiel montre comment sécuriser les données sensibles d’une base de données dans Azure SQL Database avec le chiffrement des données à l’aide de l’Assistant Always Encrypted dans SQL Server Management Studio (SSMS).
keywords: chiffrement des données, clé de chiffrement, chiffrement cloud
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: VanMSFT
ms.author: vanto
ms.reviewer: ''
ms.date: 11/02/2020
ms.openlocfilehash: 257abf03994c7006b1c3789174f550515dcd309a
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94841459"
---
# <a name="configure-always-encrypted-by-using-azure-key-vault"></a>Configurer Always Encrypted à l’aide d’Azure Key Vault 

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb-sqlmi.md)]

Cet article montre comment sécuriser les données sensibles d’une base de données dans Azure SQL Database avec le chiffrement des données à l’aide de l’[Assistant Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-wizard) dans [SQL Server Management Studio (SSMS)](/sql/ssms/sql-server-management-studio-ssms). Il inclut également des instructions concernant le stockage de chaque clé de chiffrement dans Azure Key Vault.

Always Encrypted est une technologie de chiffrement de données qui permet de protéger des données sensibles au repos sur le serveur, lors de leur déplacement entre client et serveur, et lors de leur utilisation. Cette technologie garantit que les données sensibles n’apparaissent jamais en tant que texte en clair à l’intérieur du système de base de données. Une fois le chiffrement des données configuré, seules des applications clientes ou des serveurs d’applications ayant accès aux clés peuvent accéder aux données texte en clair. Pour plus d’informations, consultez [Chiffrement intégral (moteur de base de données)](/sql/relational-databases/security/encryption/always-encrypted-database-engine).

Après avoir configuré la base de données pour le chiffrement intégral, nous allons créer une application cliente en C# avec Visual Studio afin de l’utiliser avec les données chiffrées.

Suivez les étapes de cet article et découvrez comment configurer Always Encrypted pour votre base de données dans Azure SQL Database ou SQL Managed Instance. Dans cet article, vous allez apprendre à effectuer les tâches suivantes :

- Utiliser l’Assistant Chiffrement intégral dans SSMS pour créer des [clés intégralement chiffrées](/sql/relational-databases/security/encryption/always-encrypted-database-engine#Anchor_3).
  - Créer une [clé principale de colonne (CMK)](/sql/t-sql/statements/create-column-master-key-transact-sql).
  - Créer une [clé de chiffrement de colonne (CEK)](/sql/t-sql/statements/create-column-encryption-key-transact-sql).
- Créer une table de base de données et chiffrer des colonnes.
- Créer une application qui insère, sélectionne et affiche les données des colonnes chiffrées.

## <a name="prerequisites"></a>Prérequis


- Un compte et un abonnement Azure. Si vous n’en avez pas, inscrivez-vous pour un [essai gratuit](https://azure.microsoft.com/pricing/free-trial/).
- Une base de données dans [Azure SQL Database](single-database-create-quickstart.md) ou [Azure SQL Managed Instance](../managed-instance/instance-create-quickstart.md).
- [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) , version 13.0.700.242 ou ultérieure.
- [.NET Framework 4.6](/dotnet/framework/) ou version ultérieure (sur l’ordinateur client).
- [Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx).
- [Azure PowerShell](/powershell/azure/) ou [Azure CLI](/cli/azure/install-azure-cli)

## <a name="enable-client-application-access"></a>Activer l’accès à l’application cliente

Vous devez autoriser votre application cliente à accéder à votre base de données dans SQL Database en configurant une application Azure Active Directory (Azure AD) et en copiant l’*ID d’application* et la *clé* dont vous aurez besoin pour authentifier votre application.

Pour obtenir l’*ID d’application* et la *clé*, suivez la procédure dans [Utiliser le portail pour créer une application et un principal du service Azure Active Directory pouvant accéder aux ressources](../../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="create-a-key-vault-to-store-your-keys"></a>Créer un coffre de clés pour stocker vos clés

À présent que votre application cliente est configurée et que vous avez votre ID d’application, vous devez créer un coffre de clés et configurer sa stratégie d’accès pour que votre application et vous-même puissiez accéder aux clés secrètes contenues dans le coffre (clés Always Encrypted). Les autorisations *create*, *get*, *list*, *sign*, *verify*, *wrapKey* et *unwrapKey* sont nécessaires pour créer une clé principale de colonne et configurer le chiffrement avec SQL Server Management Studio.

Vous pouvez rapidement créer un coffre de clés en exécutant le script suivant. Pour obtenir une description détaillée de ces commandes et des informations complémentaires sur la création et la configuration d’un coffre de clés, consultez [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager (RM) est toujours pris en charge par Azure SQL Database, mais tous les développements à venir sont destinés au module Az.Sql. Le module AzureRM continue à recevoir des résolutions de bogues jusqu’à au moins décembre 2020.  Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Pour en savoir plus sur leur compatibilité, consultez [Présentation du nouveau module Az Azure PowerShell](/powershell/azure/new-azureps-module-az).

```powershell
$subscriptionName = '<subscriptionName>'
$userPrincipalName = '<username@domain.com>'
$applicationId = '<applicationId from AAD application>'
$resourceGroupName = '<resourceGroupName>' # use the same resource group name when creating your SQL Database below
$location = '<datacenterLocation>'
$vaultName = '<vaultName>'

Connect-AzAccount
$subscriptionId = (Get-AzSubscription -SubscriptionName $subscriptionName).Id
Set-AzContext -SubscriptionId $subscriptionId

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzKeyVault -VaultName $vaultName -ResourceGroupName $resourceGroupName -Location $location

Set-AzKeyVaultAccessPolicy -VaultName $vaultName -ResourceGroupName $resourceGroupName -PermissionsToKeys create,get,wrapKey,unwrapKey,sign,verify,list -UserPrincipalName $userPrincipalName
Set-AzKeyVaultAccessPolicy  -VaultName $vaultName  -ResourceGroupName $resourceGroupName -ServicePrincipalName $applicationId -PermissionsToKeys get,wrapKey,unwrapKey,sign,verify,list
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
$subscriptionName = '<subscriptionName>'
$userPrincipalName = '<username@domain.com>'
$applicationId = '<applicationId from AAD application>'
$resourceGroupName = '<resourceGroupName>' # use the same resource group name when creating your database in Azure SQL Database below
$location = '<datacenterLocation>'
$vaultName = '<vaultName>'

az login
az account set --subscription $subscriptionName

az group create --location $location --name $resourceGroupName

az keyvault create --name $vaultName --resource-group $resourceGroupName --location $location

az keyvault set-policy --name $vaultName --key-permissions create get list sign unwrapKey verify wrapKey --resource-group $resourceGroupName --upn $userPrincipalName
az keyvault set-policy --name $vaultName --key-permissions get list sign unwrapKey verify wrapKey --resource-group $resourceGroupName --spn $applicationId
```

---

## <a name="connect-with-ssms"></a>Se connecter à SSMS

Ouvrez SQL Server Management Studio (SSMS) et connectez-vous au serveur ou à l’identité gérée avec votre base de données.

1. Ouvrez SSMS. (Accédez à **Connexion** > **Moteur de base de données** pour ouvrir la fenêtre **Connexion au serveur** si elle n’est pas déjà ouverte.)

2. Entrez le nom du serveur ou de l’instance et vos informations d’identification. 

    ![Copier la chaîne de connexion](./media/always-encrypted-azure-key-vault-configure/ssms-connect.png)

Si la fenêtre **Nouvelle règle de pare-feu** s’ouvre, connectez-vous à Azure et laissez SSMS créer une règle de pare-feu pour vous.

## <a name="create-a-table"></a>Créer une table

Dans cette section, vous allez créer une table pour stocker les données des patients. Initialement, elle n’est pas chiffrée, vous allez configurer le chiffrement dans la section suivante.

1. Développez **Bases de données**.
2. Cliquez avec le bouton droit sur la base de données, puis cliquez sur **Nouvelle requête**.
3. Collez l’élément Transact-SQL (T-SQL) suivant dans la fenêtre de la nouvelle requête, puis **exécutez** celle-ci.

```sql
CREATE TABLE [dbo].[Patients](
         [PatientId] [int] IDENTITY(1,1),
         [SSN] [char](11) NOT NULL,
         [FirstName] [nvarchar](50) NULL,
         [LastName] [nvarchar](50) NULL,
         [MiddleName] [nvarchar](50) NULL,
         [StreetAddress] [nvarchar](50) NULL,
         [City] [nvarchar](50) NULL,
         [ZipCode] [char](5) NULL,
         [State] [char](2) NULL,
         [BirthDate] [date] NOT NULL
         PRIMARY KEY CLUSTERED ([PatientId] ASC) ON [PRIMARY] );
GO
```

## <a name="encrypt-columns-configure-always-encrypted"></a>Chiffrer des colonnes (configurer le chiffrement intégral)

SSMS intègre un Assistant pour vous aider à configurer facilement le chiffrement intégral en définissant pour vous la clé principale de colonne, la clé de chiffrement de colonne et les colonnes chiffrées.

1. Développez **Bases de données** > **Clinique** > **Tables**.
2. Cliquez avec le bouton droit sur la table **Patients**, puis sélectionnez **Chiffrer les colonnes** pour ouvrir l’Assistant Always Encrypted :

    ![Capture d’écran sur laquelle l’option de menu Chiffrer les colonnes... est encadrée en rouge.](./media/always-encrypted-azure-key-vault-configure/encrypt-columns.png)

L’Assistant Always Encrypted comprend les sections suivantes : **Sélection de la colonne**, **Configuration de la clé principale**, **Validation** et **Résumé**.

### <a name="column-selection"></a>Sélection de colonnes

Dans la page **Introduction**, cliquez sur **Suivant** pour ouvrir la page **Sélection de colonne**. Dans cette page, sélectionnez les colonnes à chiffrer, [le type de chiffrement et la clé de chiffrement de colonne (CEK)](/sql/relational-databases/security/encryption/always-encrypted-wizard#Anchor_2) à utiliser.

Chiffrez les informations **SSN** et **BirthDate** pour chaque patient. La colonne SSN utilise un chiffrement déterministe qui prend en charge les recherches d’égalité, les jointures et les regroupements. La colonne BirthDate utilisera le chiffrement aléatoire, qui ne prend pas en charge ces opérations.

Définissez le **Type de chiffrement** de la colonne SSN sur **Déterministe**, et celui de la colonne BirthDate sur **Aléatoire**. Cliquez sur **Suivant**.

![Chiffrer les colonnes](./media/always-encrypted-azure-key-vault-configure/column-selection.png)

### <a name="master-key-configuration"></a>Configuration de la clé principale

La page **Configuration de la clé principale** vous permet de définir votre clé principale de colonne (CMK) et de sélectionner le fournisseur de magasin de clés dans lequel la CMK sera stockée. Actuellement, vous pouvez stocker une CMK dans le magasin de certificats Windows, dans Azure Key Vault ou dans un module de sécurité matériel (HSM).

Ce didacticiel montre comment stocker vos clés dans Azure Key Vault.

1. Sélectionnez **Azure Key Vault**.
2. Sélectionnez le coffre de clés de votre choix dans la liste déroulante.
3. Cliquez sur **Suivant**.

![Configuration de la clé principale](./media/always-encrypted-azure-key-vault-configure/master-key-configuration.png)

### <a name="validation"></a>Validation

Vous pouvez chiffrer les colonnes maintenant ou enregistrer un script PowerShell à exécuter ultérieurement. Pour ce didacticiel, sélectionnez **Continuer pour terminer maintenant**, puis cliquez sur **Suivant**.

### <a name="summary"></a>Résumé

Vérifiez que les paramètres sont corrects, puis cliquez sur **Terminer** afin d’achever la configuration du Chiffrement intégral.

![Capture d’écran montrant la page de résultats avec des tâches marquées comme ayant réussi.](./media/always-encrypted-azure-key-vault-configure/summary.png)

### <a name="verify-the-wizards-actions"></a>Vérifier les actions de l’Assistant

Une fois l’exécution de l’Assistant terminée, votre base de données est configurée pour le chiffrement intégral. L’Assistant a effectué les actions suivantes :

- Création d’une clé principale de colonne et stockage de celle-ci dans Azure Key Vault.
- Création d’une clé de chiffrement de colonne et stockage de celle-ci dans Azure Key Vault.
- Configuration des colonnes sélectionnées pour le chiffrement. La table Patients ne contient actuellement aucune donnée, mais toutes les données existantes dans les colonnes sélectionnées sont chiffrées.

Vous pouvez vérifier la création des clés dans SSMS en développant **Clinique** > **Sécurité** > **Clés Always Encrypted**.

## <a name="create-a-client-application-that-works-with-the-encrypted-data"></a>Création d’une application cliente compatible avec les données chiffrées

Maintenant qu’Always Encrypted est configuré, nous allons créer une application qui effectue des *insertions* et des *sélections* sur les colonnes chiffrées.  

> [!IMPORTANT]
> Votre application doit utiliser des objets [SqlParameter](/dotnet/api/system.data.sqlclient.sqlparameter) lors de la transmission de données en clair vers le serveur avec des colonnes intégralement chiffrées. La transmission de valeurs littérales sans objets SqlParameter entraînera une exception.

1. Ouvrez Visual Studio et créez une **application console** C# (Visual Studio 2015 et versions antérieures) ou une **application console (.NET Framework)** (Visual Studio 2017 et versions ultérieures). Assurez-vous que votre projet est défini sur **.NET Framework 4.6** ou version ultérieure.
2. Nommez le projet **AlwaysEncryptedConsoleAKVApp**, puis cliquez sur **OK**.
3. Installez les packages NuGet suivants en accédant à **Outils** > **Gestionnaire de package NuGet** > **Console du Gestionnaire de package**.

Dans la Console du Gestionnaire de package, exécutez ces deux lignes de code :

   ```powershell
   Install-Package Microsoft.SqlServer.Management.AlwaysEncrypted.AzureKeyVaultProvider
   Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
   ```

## <a name="modify-your-connection-string-to-enable-always-encrypted"></a>Modification de votre chaîne de connexion pour activer le chiffrement intégral

Cette section explique simplement comment activer le chiffrement intégral dans votre chaîne de connexion de base de données.

Pour activer Always Encrypted, vous devez ajouter le mot clé **Paramètre de chiffrement de colonne** à votre chaîne de connexion et le définir sur **Activé**.

Vous pouvez définir cette option directement dans la chaîne de connexion, ou la définir à l’aide du paramètre [SqlConnectionStringBuilder](/dotnet/api/system.data.sqlclient.sqlconnectionstringbuilder). L’exemple d’application de la section suivante montre comment utiliser le paramètre **SqlConnectionStringBuilder**.

### <a name="enable-always-encrypted-in-the-connection-string"></a>Activation du chiffrement intégral dans la chaîne de connexion

Ajoutez le mot clé suivant à votre chaîne de connexion.

   `Column Encryption Setting=Enabled`

### <a name="enable-always-encrypted-with-sqlconnectionstringbuilder"></a>Activer le chiffrement intégral avec le paramètre SqlConnectionStringBuilder

Le code suivant montre comment activer le chiffrement intégral en définissant le paramètre [SqlConnectionStringBuilder.ColumnEncryptionSetting](/dotnet/api/system.data.sqlclient.sqlconnectionstringbuilder.columnencryptionsetting) sur [Activé](/dotnet/api/system.data.sqlclient.sqlconnectioncolumnencryptionsetting).

```csharp
// Instantiate a SqlConnectionStringBuilder.
SqlConnectionStringBuilder connStringBuilder = new SqlConnectionStringBuilder("replace with your connection string");

// Enable Always Encrypted.
connStringBuilder.ColumnEncryptionSetting = SqlConnectionColumnEncryptionSetting.Enabled;
```

## <a name="register-the-azure-key-vault-provider"></a>Inscrire le fournisseur du coffre de clés Azure (Azure Key Vault)
Le code suivant montre comment inscrire le fournisseur d’Azure Key Vault auprès du pilote ADO.NET.

```csharp
private static ClientCredential _clientCredential;

static void InitializeAzureKeyVaultProvider() {
    _clientCredential = new ClientCredential(applicationId, clientKey);

    SqlColumnEncryptionAzureKeyVaultProvider azureKeyVaultProvider = new SqlColumnEncryptionAzureKeyVaultProvider(GetToken);

    Dictionary<string, SqlColumnEncryptionKeyStoreProvider> providers = new Dictionary<string, SqlColumnEncryptionKeyStoreProvider>();

    providers.Add(SqlColumnEncryptionAzureKeyVaultProvider.ProviderName, azureKeyVaultProvider);
    SqlConnection.RegisterColumnEncryptionKeyStoreProviders(providers);
}
```

## <a name="always-encrypted-sample-console-application"></a>Exemple d’application console intégralement chiffrée

Cet exemple montre comment :

- Modifier votre chaîne de connexion pour activer le chiffrement intégral.
- Inscrire Azure Key Vault en tant que fournisseur de magasin de clés de l’application.  
- Insérer des données dans les colonnes chiffrées.
- Sélectionner un enregistrement en filtrant une valeur spécifique dans une colonne chiffrée.

Remplacez le contenu de *Program.cs* par le code suivant. Remplacez la chaîne de connexion pour la variable constringente globale dans la ligne juste au-dessus de la méthode Main par votre chaîne de connexion valide provenant du portail Azure. Il s'agit de la seule modification que vous devez apporter à ce code.

Exécutez l’application pour voir le chiffrement intégral en action.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Data;
using System.Data.SqlClient;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.SqlServer.Management.AlwaysEncrypted.AzureKeyVaultProvider;

namespace AlwaysEncryptedConsoleAKVApp {
    class Program {
        // Update this line with your Clinic database connection string from the Azure portal.
        static string connectionString = @"<connection string from the portal>";
        static string applicationId = @"<application ID from your AAD application>";
        static string clientKey = "<key from your AAD application>";

        static void Main(string[] args) {
            InitializeAzureKeyVaultProvider();

            Console.WriteLine("Signed in as: " + _clientCredential.ClientId);

            Console.WriteLine("Original connection string copied from the Azure portal:");
            Console.WriteLine(connectionString);

            // Create a SqlConnectionStringBuilder.
            SqlConnectionStringBuilder connStringBuilder =
                new SqlConnectionStringBuilder(connectionString);

            // Enable Always Encrypted for the connection.
            // This is the only change specific to Always Encrypted
            connStringBuilder.ColumnEncryptionSetting =
                SqlConnectionColumnEncryptionSetting.Enabled;

            Console.WriteLine(Environment.NewLine + "Updated connection string with Always Encrypted enabled:");
            Console.WriteLine(connStringBuilder.ConnectionString);

            // Update the connection string with a password supplied at runtime.
            Console.WriteLine(Environment.NewLine + "Enter server password:");
            connStringBuilder.Password = Console.ReadLine();

            // Assign the updated connection string to our global variable.
            connectionString = connStringBuilder.ConnectionString;

            // Delete all records to restart this demo app.
            ResetPatientsTable();

            // Add sample data to the Patients table.
            Console.Write(Environment.NewLine + "Adding sample patient data to the database...");

            InsertPatient(new Patient() {
                SSN = "999-99-0001",
                FirstName = "Orlando",
                LastName = "Gee",
                BirthDate = DateTime.Parse("01/04/1964")
            });
            InsertPatient(new Patient() {
                SSN = "999-99-0002",
                FirstName = "Keith",
                LastName = "Harris",
                BirthDate = DateTime.Parse("06/20/1977")
            });
            InsertPatient(new Patient() {
                SSN = "999-99-0003",
                FirstName = "Donna",
                LastName = "Carreras",
                BirthDate = DateTime.Parse("02/09/1973")
            });
            InsertPatient(new Patient() {
                SSN = "999-99-0004",
                FirstName = "Janet",
                LastName = "Gates",
                BirthDate = DateTime.Parse("08/31/1985")
            });
            InsertPatient(new Patient() {
                SSN = "999-99-0005",
                FirstName = "Lucy",
                LastName = "Harrington",
                BirthDate = DateTime.Parse("05/06/1993")
            });

            // Fetch and display all patients.
            Console.WriteLine(Environment.NewLine + "All the records currently in the Patients table:");

            foreach (Patient patient in SelectAllPatients()) {
                Console.WriteLine(patient.FirstName + " " + patient.LastName + "\tSSN: " + patient.SSN + "\tBirthdate: " + patient.BirthDate);
            }

            // Get patients by SSN.
            Console.WriteLine(Environment.NewLine + "Now lets locate records by searching the encrypted SSN column.");

            string ssn;

            // This very simple validation only checks that the user entered 11 characters.
            // In production be sure to check all user input and use the best validation for your specific application.
            do {
                Console.WriteLine("Please enter a valid SSN (ex. 999-99-0003):");
                ssn = Console.ReadLine();
            } while (ssn.Length != 11);

            // The example allows duplicate SSN entries so we will return all records
            // that match the provided value and store the results in selectedPatients.
            Patient selectedPatient = SelectPatientBySSN(ssn);

            // Check if any records were returned and display our query results.
            if (selectedPatient != null) {
                Console.WriteLine("Patient found with SSN = " + ssn);
                Console.WriteLine(selectedPatient.FirstName + " " + selectedPatient.LastName + "\tSSN: "
                    + selectedPatient.SSN + "\tBirthdate: " + selectedPatient.BirthDate);
            }
            else {
                Console.WriteLine("No patients found with SSN = " + ssn);
            }

            Console.WriteLine("Press Enter to exit...");
            Console.ReadLine();
        }

        private static ClientCredential _clientCredential;

        static void InitializeAzureKeyVaultProvider() {
            _clientCredential = new ClientCredential(applicationId, clientKey);

            SqlColumnEncryptionAzureKeyVaultProvider azureKeyVaultProvider =
              new SqlColumnEncryptionAzureKeyVaultProvider(GetToken);

            Dictionary<string, SqlColumnEncryptionKeyStoreProvider> providers =
              new Dictionary<string, SqlColumnEncryptionKeyStoreProvider>();

            providers.Add(SqlColumnEncryptionAzureKeyVaultProvider.ProviderName, azureKeyVaultProvider);
            SqlConnection.RegisterColumnEncryptionKeyStoreProviders(providers);
        }

        public async static Task<string> GetToken(string authority, string resource, string scope) {
            var authContext = new AuthenticationContext(authority);
            AuthenticationResult result = await authContext.AcquireTokenAsync(resource, _clientCredential);

            if (result == null)
                throw new InvalidOperationException("Failed to obtain the access token");
            return result.AccessToken;
        }

        static int InsertPatient(Patient newPatient) {
            int returnValue = 0;

            string sqlCmdText = @"INSERT INTO [dbo].[Patients] ([SSN], [FirstName], [LastName], [BirthDate])
     VALUES (@SSN, @FirstName, @LastName, @BirthDate);";

            SqlCommand sqlCmd = new SqlCommand(sqlCmdText);

            SqlParameter paramSSN = new SqlParameter(@"@SSN", newPatient.SSN);
            paramSSN.DbType = DbType.AnsiStringFixedLength;
            paramSSN.Direction = ParameterDirection.Input;
            paramSSN.Size = 11;

            SqlParameter paramFirstName = new SqlParameter(@"@FirstName", newPatient.FirstName);
            paramFirstName.DbType = DbType.String;
            paramFirstName.Direction = ParameterDirection.Input;

            SqlParameter paramLastName = new SqlParameter(@"@LastName", newPatient.LastName);
            paramLastName.DbType = DbType.String;
            paramLastName.Direction = ParameterDirection.Input;

            SqlParameter paramBirthDate = new SqlParameter(@"@BirthDate", newPatient.BirthDate);
            paramBirthDate.SqlDbType = SqlDbType.Date;
            paramBirthDate.Direction = ParameterDirection.Input;

            sqlCmd.Parameters.Add(paramSSN);
            sqlCmd.Parameters.Add(paramFirstName);
            sqlCmd.Parameters.Add(paramLastName);
            sqlCmd.Parameters.Add(paramBirthDate);

            using (sqlCmd.Connection = new SqlConnection(connectionString)) {
                try {
                    sqlCmd.Connection.Open();
                    sqlCmd.ExecuteNonQuery();
                }
                catch (Exception ex) {
                    returnValue = 1;
                    Console.WriteLine("The following error was encountered: ");
                    Console.WriteLine(ex.Message);
                    Console.WriteLine(Environment.NewLine + "Press Enter key to exit");
                    Console.ReadLine();
                    Environment.Exit(0);
                }
            }
            return returnValue;
        }


        static List<Patient> SelectAllPatients() {
            List<Patient> patients = new List<Patient>();

            SqlCommand sqlCmd = new SqlCommand(
              "SELECT [SSN], [FirstName], [LastName], [BirthDate] FROM [dbo].[Patients]",
                new SqlConnection(connectionString));

            using (sqlCmd.Connection = new SqlConnection(connectionString))

            using (sqlCmd.Connection = new SqlConnection(connectionString)) {
                try {
                    sqlCmd.Connection.Open();
                    SqlDataReader reader = sqlCmd.ExecuteReader();

                    if (reader.HasRows) {
                        while (reader.Read()) {
                            patients.Add(new Patient() {
                                SSN = reader[0].ToString(),
                                FirstName = reader[1].ToString(),
                                LastName = reader["LastName"].ToString(),
                                BirthDate = (DateTime)reader["BirthDate"]
                            });
                        }
                    }
                }
                catch (Exception ex) {
                    throw;
                }
            }

            return patients;
        }

        static Patient SelectPatientBySSN(string ssn) {
            Patient patient = new Patient();

            SqlCommand sqlCmd = new SqlCommand(
                "SELECT [SSN], [FirstName], [LastName], [BirthDate] FROM [dbo].[Patients] WHERE [SSN]=@SSN",
                new SqlConnection(connectionString));

            SqlParameter paramSSN = new SqlParameter(@"@SSN", ssn);
            paramSSN.DbType = DbType.AnsiStringFixedLength;
            paramSSN.Direction = ParameterDirection.Input;
            paramSSN.Size = 11;

            sqlCmd.Parameters.Add(paramSSN);

            using (sqlCmd.Connection = new SqlConnection(connectionString)) {
                try {
                    sqlCmd.Connection.Open();
                    SqlDataReader reader = sqlCmd.ExecuteReader();

                    if (reader.HasRows) {
                        while (reader.Read()) {
                            patient = new Patient() {
                                SSN = reader[0].ToString(),
                                FirstName = reader[1].ToString(),
                                LastName = reader["LastName"].ToString(),
                                BirthDate = (DateTime)reader["BirthDate"]
                            };
                        }
                    }
                    else {
                        patient = null;
                    }
                }
                catch (Exception ex) {
                    throw;
                }
            }
            return patient;
        }

        // This method simply deletes all records in the Patients table to reset our demo.
        static int ResetPatientsTable() {
            int returnValue = 0;

            SqlCommand sqlCmd = new SqlCommand("DELETE FROM Patients");
            using (sqlCmd.Connection = new SqlConnection(connectionString)) {
                try {
                    sqlCmd.Connection.Open();
                    sqlCmd.ExecuteNonQuery();

                }
                catch (Exception ex) {
                    returnValue = 1;
                }
            }
            return returnValue;
        }
    }

    class Patient {
        public string SSN { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime BirthDate { get; set; }
    }
}
```

## <a name="verify-that-the-data-is-encrypted"></a>Vérifier que les données sont chiffrées.

Vous pouvez vérifier rapidement que les données réelles sur le serveur sont chiffrées en interrogeant les données Patients avec SSMS (en utilisant votre connexion actuelle où le **Paramètre de chiffrement de colonne** n’est pas encore activé).

Exécutez la requête suivante sur la base de données Clinique.

```sql
SELECT FirstName, LastName, SSN, BirthDate FROM Patients;
```

Vous pouvez voir que les colonnes chiffrées ne contiennent pas de donnée en clair.

   ![Capture d’écran qui montre que les colonnes chiffrées ne contiennent pas de donnée en clair.](./media/always-encrypted-azure-key-vault-configure/ssms-encrypted.png)

Pour utiliser SSMS dans le but d’accéder aux données texte en clair, vous devez tout d’abord vérifier que l’utilisateur dispose des autorisations appropriées pour Azure Key Vault : *get*, *unwrapKey* et *verify*. Pour plus d’informations, consultez [Créer et stocker des clés principales de colonne (Always Encrypted)](/sql/relational-databases/security/encryption/create-and-store-column-master-keys-always-encrypted).

Ajoutez ensuite le paramètre *Column Encryption Setting=enabled* lors de votre connexion.

1. Dans SSMS, cliquez avec le bouton droit sur votre serveur dans **l’Explorateur d’objets**, puis choisissez **Déconnecter**.
2. Cliquez sur **Se connecter** > **Moteur de base de données** pour ouvrir la fenêtre **Connexion au serveur**, puis cliquez sur **Options**.
3. Cliquez sur **Paramètres de connexion supplémentaires** et tapez **Column Encryption Setting=enabled**.

    ![Capture d’écran qui montre l’onglet Additional Correction Parameters (Paramètres de correction supplémentaires) sélectionné.](./media/always-encrypted-azure-key-vault-configure/ssms-connection-parameter.png)

4. Exécutez la requête suivante sur la base de données Clinique.

   ```sql
   SELECT FirstName, LastName, SSN, BirthDate FROM Patients;
   ```

   Vous pouvez maintenant afficher les données en clair dans les colonnes chiffrées.
   
   ![Nouvelle application de console](./media/always-encrypted-azure-key-vault-configure/ssms-plaintext.png)

## <a name="next-steps"></a>Étapes suivantes

Une fois que votre base de données est configurée pour utiliser Always Encrypted, vous pouvez effectuer les opérations suivantes :

- [Faire pivoter et nettoyer vos clés](/sql/relational-databases/security/encryption/configure-always-encrypted-using-sql-server-management-studio).
- [Migrer des données déjà chiffrées avec le chiffrement intégral](/sql/relational-databases/security/encryption/migrate-sensitive-data-protected-by-always-encrypted).

## <a name="related-information"></a>Informations connexes

- [Chiffrement intégral (développement client)](/sql/relational-databases/security/encryption/always-encrypted-client-development)
- [Chiffrement transparent des données](/sql/relational-databases/security/encryption/transparent-data-encryption)
- [Chiffrement SQL Server](/sql/relational-databases/security/encryption/sql-server-encryption)
- [Assistant Chiffrement intégral](/sql/relational-databases/security/encryption/always-encrypted-wizard)
- [Blog Chiffrement intégral](/archive/blogs/sqlsecurity/always-encrypted-key-metadata)
