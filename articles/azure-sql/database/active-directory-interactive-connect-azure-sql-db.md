---
title: Se connecter à SQL en mode ActiveDirectoryInteractive
description: Exemple de code C# accompagné d’explications, pour établir une connexion à Azure SQL Database en utilisant le mode SqlAuthenticationMethod.ActiveDirectoryInteractive.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: active directory, has-adal-ref, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: GithubMirek
ms.author: MirekS
ms.reviewer: vanto
ms.date: 04/23/2020
ms.openlocfilehash: 93831ec4c1dc3e34c2ea144e71b67dae711ee870
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94841646"
---
# <a name="connect-to-azure-sql-database-with-azure-ad-multi-factor-authentication"></a>Se connecter à Azure SQL Database avec Azure AD Multi-Factor Authentication
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article contient un programme C# qui vous connecte à Azure SQL Database . Le programme utilise l'authentification en mode interactif, qui prend en charge [Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md).

Pour plus d’informations sur la prise en charge de Microsoft Azure Multi-Factor Authentication pour les outils SQL, consultez [Prise en charge d’Azure Active Directory dans SQL Server Data Tools (SSDT)](/sql/ssdt/azure-active-directory).

## <a name="multi-factor-authentication-for-azure-sql-database"></a>Multi-Factor Authentication pour Azure SQL Database

À compter de la version 4.7.2 du .NET Framework, l’énumération [`SqlAuthenticationMethod`](/dotnet/api/system.data.sqlclient.sqlauthenticationmethod) a une nouvelle valeur : `ActiveDirectoryInteractive`. Dans un programme client C#, cette valeur d’énumération indique au système d’utiliser le mode interactif d’Azure Active Directory (Azure AD) prenant en charge Multi-Factor Authentication pour se connecter à Azure SQL Database. L’utilisateur qui exécute le programme voit les boîtes de dialogue suivantes :

* Une boîte de dialogue qui affiche un nom d’utilisateur Azure AD et qui demande le mot de passe de l’utilisateur.

   Si le domaine de l’utilisateur est fédéré avec Azure AD, cette boîte de dialogue n’apparaît pas, car aucun mot de passe n’est nécessaire.

   Si la stratégie Azure AD impose Multi-Factor Authentication pour l’utilisateur, les deux boîtes de dialogue suivantes sont affichées.

* La première fois qu’un utilisateur utilise Multi-Factor Authentication, le système affiche une boîte de dialogue demandant un numéro de téléphone mobile pour l’envoi de SMS. Chaque SMS contient le *code de vérification* que l’utilisateur doit entrer dans la boîte de dialogue suivante.

* Une autre boîte de dialogue qui demande le code de vérification Multi-Factor Authentication, que le système a envoyé sur un téléphone mobile.

Pour plus d'informations sur la configuration d'Azure AD afin d'exiger une authentification multifacteur, consultez [Bien démarrer avec Azure AD Multi-Factor Authentication dans le cloud](../../active-directory/authentication/howto-mfa-getstarted.md).

Pour consulter les captures d’écran de ces boîtes de dialogue, consultez [Configure multi-factor authentication for SQL Server Management Studio and Azure AD](authentication-mfa-ssms-configure.md) (Configurer l’authentification multifacteur pour SQL Server Management Studio et Azure AD).

> [!TIP]
> Vous pouvez rechercher dans les API .NET Framework avec notre [page d’outil Explorateur d’API .NET](/dotnet/api/).
>
> Vous pouvez également rechercher directement avec le [paramètre facultatif ?term=&lt;valeur de recherche&gt;](/dotnet/api/?term=SqlAuthenticationMethod).

## <a name="configure-your-c-application-in-the-azure-portal"></a>Configurer votre application C# dans le portail Azure

Avant de commencer, vous devez avoir créé un [serveur SQL logique](logical-servers.md) qui doit être disponible.

### <a name="register-your-app-and-set-permissions"></a>Inscrire votre application et définir des autorisations

Pour utiliser l’authentification Azure AD, votre programme C# doit s’inscrire en tant qu’application Azure AD. Pour inscrire une application, vous devez être administrateur Azure AD ou utilisateur avec le rôle Azure AD *Développeur d’applications*. Pour plus d’informations sur l’attribution de rôles, consultez [Attribuer des rôles administrateur et non-administrateur aux utilisateurs avec Azure Active Directory](../../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md).

L’inscription d’une application génère et affiche un **ID d’application**. Votre programme doit utiliser cet ID pour se connecter.

Pour inscrire votre application et définir les autorisations nécessaires :

1. Dans le portail Azure, sélectionnez **Azure Active Directory** > **Inscriptions d’applications** > **Nouvelle inscription**.

    ![Inscription d'application](./media/active-directory-interactive-connect-azure-sql-db/image1.png)

    Une fois l’inscription d’application créée, la valeur de **l’ID d’application** est générée et affichée.

    ![ID d’application affiché](./media/active-directory-interactive-connect-azure-sql-db/image2.png)

2. Sélectionnez **Autorisations des API** > **Ajouter une autorisation**.

    ![Paramètres des autorisations de l’application inscrite](./media/active-directory-interactive-connect-azure-sql-db/sshot-registered-app-settings-required-permissions-add-api-access-c32.png)

3. Sélectionnez le type **API que mon organisation utilise** > **Azure SQL Database** dans la recherche > puis sélectionnez **Azure SQL Database**.

    ![Ajouter un accès à l’API pour Azure SQL Database](./media/active-directory-interactive-connect-azure-sql-db/sshot-registered-app-settings-required-permissions-add-api-access-Azure-sql-db-d11.png)

4. Sélectionnez **Autorisations déléguées** > **user_impersonation** > **Ajouter des autorisations**.

    ![Déléguer des autorisations à l’API pour Azure SQL Database](./media/active-directory-interactive-connect-azure-sql-db/sshot-add-api-access-azure-sql-db-delegated-permissions-checkbox-e14.png)

### <a name="set-an-azure-ad-admin-for-your-server"></a>Définir un administrateur Azure AD pour votre serveur

Pour que votre programme C# s’exécute, un administrateur de [serveur SQL logique](logical-servers.md) doit attribuer un administrateur Azure AD à votre serveur .

Sur la page **Serveur SQL**, sélectionnez **Administrateur Active Directory** > **Définir l’administrateur**.

Pour plus d’informations sur les administrateurs Azure AD et les utilisateurs Azure SQL Database, consultez les captures d’écran de l’article [Configure and manage Azure Active Directory authentication with SQL](authentication-aad-configure.md#provision-azure-ad-admin-sql-database) (Configurer et gérer l’authentification Azure Active Directory avec SQL).

### <a name="add-a-non-admin-user-to-a-specific-database-optional"></a>Ajouter un utilisateur non-administrateur à une base de données spécifique (facultatif)

L’administrateur Azure AD d’un [serveur SQL logique](logical-servers.md) peut exécuter l’exemple de programme C#. Un utilisateur Azure AD peut exécuter le programme s’il se trouve dans la base de données. Un administrateur SQL Azure AD ou un utilisateur Azure AD qui existe déjà dans la base de données et a l’autorisation `ALTER ANY USER` sur la base de données peut ajouter un utilisateur.

Vous pouvez ajouter un utilisateur à la base de données avec la commande SQL [`Create User`](/sql/t-sql/statements/create-user-transact-sql). par exemple `CREATE USER [<username>] FROM EXTERNAL PROVIDER`.

Pour plus d’informations, consultez [Utiliser l’authentification Azure Active Directory pour l’authentification auprès de SQL Database, Managed Instance ou Azure Synapse Analytics](authentication-aad-overview.md).

## <a name="new-authentication-enum-value"></a>Nouvelle valeur d’énumération d’authentification

L’exemple C# repose sur l’espace de noms [`System.Data.SqlClient`](/dotnet/api/system.data.sqlclient). L’énumération `SqlAuthenticationMethod` est particulièrement intéressante pour Multi-Factor Authentication. Elle a les valeurs suivantes :

* `SqlAuthenticationMethod.ActiveDirectoryInteractive`

   Utilisez cette valeur avec un nom d’utilisateur Azure AD pour implémenter Multi-Factor Authentication. Cet article traite essentiellement de cette valeur. Cette valeur génère une expérience interactive en affichant des boîtes de dialogue pour le mot de passe de l’utilisateur, puis pour la validation de Multi-Factor Authentication si ce service est appliqué pour cet utilisateur. Cette valeur est disponible depuis .NET Framework version 4.7.2.

* `SqlAuthenticationMethod.ActiveDirectoryIntegrated`

  Utilisez cette valeur pour un compte *fédéré*. Pour un compte fédéré, le nom d’utilisateur est connu du domaine Windows. Cette méthode d’authentification ne prend pas en charge Multi-Factor Authentication.

* `SqlAuthenticationMethod.ActiveDirectoryPassword`

  Utilisez cette valeur pour une authentification qui exige un nom d’utilisateur et un mot de passe Azure AD. Azure SQL Database effectue l’authentification. Cette méthode ne prend pas en charge Multi-Factor Authentication.

> [!NOTE]
> Si vous utilisez .NET Core, vous pouvez utiliser l’espace de noms [Microsoft.Data.SqlClient](/dotnet/api/microsoft.data.sqlclient?view=sqlclient-dotnet-core-1.1). Pour plus d’informations, consultez le billet de [blog](https://devblogs.microsoft.com/dotnet/introducing-the-new-microsoftdatasqlclient/) suivant.

## <a name="set-c-parameter-values-from-the-azure-portal"></a>Définir les valeurs des paramètres C# à partir du portail Azure

Pour que le programme C# puisse s’exécuter, vous devez attribuer les valeurs appropriées à des champs statiques. Voici des champs avec des exemples de valeurs. Les emplacements dans le portail Azure où vous pouvez obtenir les valeurs nécessaires sont également affichés.

| Nom de champ statique | Valeur d'exemple | Emplacement dans le portail Azure |
| :---------------- | :------------ | :-------------------- |
| Az_SQLDB_svrName | « my-sqldb-svr.database.windows.net » | **Serveurs SQL** > **Filtrer par nom** |
| AzureAD_UserID | « auser\@abc.onmicrosoft.com » | **Azure Active Directory** > **Utilisateur** > **Nouvel utilisateur invité** |
| Initial_DatabaseName | « myDatabase » | **Serveurs SQL** > **Bases de données SQL** |
| ClientApplicationID | « a94f9c62-97fe-4d19-b06d-111111111111 » | **Azure Active Directory** > **Inscriptions d’applications** > **Rechercher par nom** > **ID d’application** |
| RedirectUri | new Uri(« https://mywebserver.com/  ») | **Azure Active Directory** > **Inscriptions d’applications** > **Rechercher par nom** >  *[Votre-inscription-application]*  > **Paramètres** > **RedirectURIs**<br /><br />Pour cet article, n’importe quelle valeur valide convient à RedirectUri, car il n’est pas utilisé ici. |
| &nbsp; | &nbsp; | &nbsp; |

## <a name="verify-with-sql-server-management-studio"></a>Vérifier avec SQL Server Management Studio

Avant d’exécuter le programme C#, vérifiez que vos configurations sont correctes dans SQL Server Management Studio (SSMS). Tout échec du programme C# peut ensuite être limité au code source.

### <a name="verify-server-level-firewall-ip-addresses"></a>Vérifier les adresses IP de pare-feu au niveau du serveur

Exécutez SSMS sur l’ordinateur sur lequel vous envisagez d’exécuter le programme C#. Pour ce test, vous pouvez choisir n’importe quel mode **d’authentification**. Si un message indique que le serveur n’accepte pas votre adresse IP, consultez [Règles de pare-feu au niveau du serveur et de la base de données](firewall-configure.md) pour obtenir de l’aide.

### <a name="verify-azure-active-directory-multi-factor-authentication"></a>Vérifier Azure Active Directory Multi-Factor Authentication

Réexécutez SSMS, en définissant cette fois **Authentification** sur **Azure Active Directory - Authentification universelle avec MFA**. Cette option nécessite SSMS version 17.5 ou ultérieure.

Pour plus d’informations, consultez [Configure multi-factor authentication for SQL Server Management Studio and Azure AD](authentication-mfa-ssms-configure.md) (Configurer l’authentification multifacteur pour SQL Server Management Studio et Azure AD).

> [!NOTE]
> Si vous êtes un utilisateur invité dans la base de données, vous devez également fournir le nom de domaine Azure AD pour la base de données : sélectionnez **Options** > **Nom de domaine AD ou ID de locataire**. Pour rechercher le nom de domaine dans le portail Azure, sélectionnez **Azure Active Directory** > **Noms de domaine personnalisés**. Dans l’exemple de programme C#, vous n’avez pas besoin de fournir de nom de domaine.

## <a name="c-code-example"></a>Exemple de code C#

> [!NOTE]
> Si vous utilisez .NET Core, vous pouvez utiliser l’espace de noms [Microsoft.Data.SqlClient](/dotnet/api/microsoft.data.sqlclient?view=sqlclient-dotnet-core-1.1). Pour plus d’informations, consultez le billet de [blog](https://devblogs.microsoft.com/dotnet/introducing-the-new-microsoftdatasqlclient/) suivant.

L’exemple de programme C# repose sur l’assembly DLL [*Microsoft.IdentityModel.Clients.ActiveDirectory*](/dotnet/api/microsoft.identitymodel.clients.activedirectory).

Pour installer ce package, dans Visual Studio, sélectionnez **Projet** > **Gérer les packages NuGet**. Recherchez **Microsoft.IdentityModel.Clients.ActiveDirectory** et installez-le.

Voici un exemple de code source C#.

```csharp

using System;

// Reference to Azure AD authentication assembly
using Microsoft.IdentityModel.Clients.ActiveDirectory;

using DA = System.Data;
using SC = System.Data.SqlClient;
using AD = Microsoft.IdentityModel.Clients.ActiveDirectory;
using TX = System.Text;
using TT = System.Threading.Tasks;

namespace ADInteractive5
{
    class Program
    {
        // ASSIGN YOUR VALUES TO THESE STATIC FIELDS !!
        static public string Az_SQLDB_svrName = "<Your server>";
        static public string AzureAD_UserID = "<Your User ID>";
        static public string Initial_DatabaseName = "<Your Database>";
        // Some scenarios do not need values for the following two fields:
        static public readonly string ClientApplicationID = "<Your App ID>";
        static public readonly Uri RedirectUri = new Uri("<Your URI>");

        public static void Main(string[] args)
        {
            var provider = new ActiveDirectoryAuthProvider();

            SC.SqlAuthenticationProvider.SetProvider(
                SC.SqlAuthenticationMethod.ActiveDirectoryInteractive,
                //SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated,  // Alternatives.
                //SC.SqlAuthenticationMethod.ActiveDirectoryPassword,
                provider);

            Program.Connection();
        }

        public static void Connection()
        {
            SC.SqlConnectionStringBuilder builder = new SC.SqlConnectionStringBuilder();

            // Program._  static values that you set earlier.
            builder["Data Source"] = Program.Az_SQLDB_svrName;
            builder.UserID = Program.AzureAD_UserID;
            builder["Initial Catalog"] = Program.Initial_DatabaseName;

            // This "Password" is not used with .ActiveDirectoryInteractive.
            //builder["Password"] = "<YOUR PASSWORD HERE>";

            builder["Connect Timeout"] = 15;
            builder["TrustServerCertificate"] = true;
            builder.Pooling = false;

            // Assigned enum value must match the enum given to .SetProvider().
            builder.Authentication = SC.SqlAuthenticationMethod.ActiveDirectoryInteractive;
            SC.SqlConnection sqlConnection = new SC.SqlConnection(builder.ConnectionString);

            SC.SqlCommand cmd = new SC.SqlCommand(
                "SELECT '******** MY QUERY RAN SUCCESSFULLY!! ********';",
                sqlConnection);

            try
            {
                sqlConnection.Open();
                if (sqlConnection.State == DA.ConnectionState.Open)
                {
                    var rdr = cmd.ExecuteReader();
                    var msg = new TX.StringBuilder();
                    while (rdr.Read())
                    {
                        msg.AppendLine(rdr.GetString(0));
                    }
                    Console.WriteLine(msg.ToString());
                    Console.WriteLine(":Success");
                }
                else
                {
                    Console.WriteLine(":Failed");
                }
                sqlConnection.Close();
            }
            catch (Exception ex)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Connection failed with the following exception...");
                Console.WriteLine(ex.ToString());
                Console.ResetColor();
            }
        }
    } // EOClass Program.

    /// <summary>
    /// SqlAuthenticationProvider - Is a public class that defines 3 different Azure AD
    /// authentication methods.  The methods are supported in the new .NET 4.7.2.
    ///  .
    /// 1. Interactive,  2. Integrated,  3. Password
    ///  .
    /// All 3 authentication methods are based on the Azure
    /// Active Directory Authentication Library (ADAL) managed library.
    /// </summary>
    public class ActiveDirectoryAuthProvider : SC.SqlAuthenticationProvider
    {
        // Program._ more static values that you set!
        private readonly string _clientId = Program.ClientApplicationID;
        private readonly Uri _redirectUri = Program.RedirectUri;

        public override async TT.Task<SC.SqlAuthenticationToken>
            AcquireTokenAsync(SC.SqlAuthenticationParameters parameters)
        {
            AD.AuthenticationContext authContext =
                new AD.AuthenticationContext(parameters.Authority);
            authContext.CorrelationId = parameters.ConnectionId;
            AD.AuthenticationResult result;

            switch (parameters.AuthenticationMethod)
            {
                case SC.SqlAuthenticationMethod.ActiveDirectoryInteractive:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_0 == '.ActiveDirectoryInteractive'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,  // "https://database.windows.net/"
                        _clientId,
                        _redirectUri,
                        new AD.PlatformParameters(AD.PromptBehavior.Auto),
                        new AD.UserIdentifier(
                            parameters.UserId,
                            AD.UserIdentifierType.RequiredDisplayableId));
                    break;

                case SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_1 == '.ActiveDirectoryIntegrated'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,
                        _clientId,
                        new AD.UserCredential());
                    break;

                case SC.SqlAuthenticationMethod.ActiveDirectoryPassword:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_2 == '.ActiveDirectoryPassword'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,
                        _clientId,
                        new AD.UserPasswordCredential(
                            parameters.UserId,
                            parameters.Password));
                    break;

                default: throw new InvalidOperationException();
            }
            return new SC.SqlAuthenticationToken(result.AccessToken, result.ExpiresOn);
        }

        public override bool IsSupported(SC.SqlAuthenticationMethod authenticationMethod)
        {
            return authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated
                || authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryInteractive
                || authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryPassword;
        }
    } // EOClass ActiveDirectoryAuthProvider.
} // EONamespace.  End of entire program source code.

```

&nbsp;

Voici un exemple de la sortie de test C#.

```C#
[C:\Test\VSProj\ADInteractive5\ADInteractive5\bin\Debug\]
>> ADInteractive5.exe
In method 'AcquireTokenAsync', case_0 == '.ActiveDirectoryInteractive'.
******** MY QUERY RAN SUCCESSFULLY!! ********

:Success

[C:\Test\VSProj\ADInteractive5\ADInteractive5\bin\Debug\]
>>
```

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

& [Get-AzSqlServerActiveDirectoryAdministrator](/powershell/module/az.sql/get-azsqlserveractivedirectoryadministrator)