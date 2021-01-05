---
title: Connexion avec Managed Identity – Azure Database pour MySQL
description: En savoir plus sur la connexion et l’authentification à l’aide de Managed Identity pour l’authentification avec Azure Database pour MySQL
author: lfittl-msft
ms.author: lufittl
ms.service: mysql
ms.topic: how-to
ms.date: 05/19/2020
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 74530b861b4dc7a35fa8b192ecc0e79ce5fa59c2
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92742995"
---
# <a name="connect-with-managed-identity-to-azure-database-for-mysql"></a>Se connecter avec Managed Identity à Azure Database pour MySQL

Cet article explique comment utiliser une identité affectée par l’utilisateur à une machine virtuelle Azure pour accéder à un serveur Azure Database pour MySQL. Les identités MSI sont gérées automatiquement par Azure et vous permettent de vous authentifier auprès des services prenant en charge l’authentification Azure AD sans avoir à insérer des informations d’identification dans votre code. 

Vous allez apprendre à effectuer les actions suivantes :

- Accorder à votre machine virtuelle l’accès à un serveur Azure Database pour MySQL
- Créer un utilisateur dans la base de données représentant l’identité affectée par l’utilisateur de la machine virtuelle
- Obtenir un jeton d’accès à l’aide de l’identité de la machine virtuelle et l’utiliser pour interroger un serveur Azure Database pour MySQL
- Implémenter la récupération de jetons dans un exemple d’application C#

> [!IMPORTANT]
> La connexion avec une identité gérée est uniquement disponible pour MySQL 5.7 et versions ultérieures.

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé de la fonctionnalité identités managées pour ressources Azure, consultez cette [Vue d’ensemble](../../articles/active-directory/managed-identities-azure-resources/overview.md). Si vous n’avez pas encore de compte Azure, [Inscrivez-vous sur un compte gratuit](https://azure.microsoft.com/free/) avant de continuer.
- Pour effectuer les opérations nécessaires de création de ressources et de gestion de rôles, votre compte doit disposer des autorisations « Propriétaire » au niveau de l’étendue appropriée (votre abonnement ou groupe de ressources). Si vous avez besoin d’aide concernant l’attribution de rôle, consultez [Utiliser un contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour gérer l’accès aux ressources de votre abonnement Azure](../../articles/role-based-access-control/role-assignments-portal.md).
- Vous avez besoin d’une machine virtuelle Azure (exécutant par exemple Ubuntu Linux) que vous souhaitez utiliser pour accéder à votre base de données à l’aide de Managed Identity
- Vous avez besoin d’un serveur de base de données Azure Database pour MySQL sur lequel [Azure AD Authentication](howto-configure-sign-in-azure-ad-authentication.md) est configuré
- Pour suivre cet exemple C#, vous devez d’abord suivre le guide pratique vous permettant de [vous connecter avec C#](connect-csharp.md)

## <a name="creating-a-user-assigned-managed-identity-for-your-vm"></a>Création d’une identité managée affectée par l’utilisateur pour votre machine virtuelle

Créez une identité dans votre abonnement avec la commande [az identity create](/cli/azure/identity#az-identity-create). Vous pouvez utiliser le même groupe de ressources que celui dans lequel votre machine virtuelle s’exécute ou un groupe différent.

```azurecli-interactive
az identity create --resource-group myResourceGroup --name myManagedIdentity
```

Pour configurer l’identité aux étapes suivantes, utilisez la commande [az identity show](/cli/azure/identity#az-identity-show) afin de stocker l’ID de ressource et l’ID client de l’identité dans des variables.

```azurecli
# Get resource ID of the user-assigned identity
resourceID=$(az identity show --resource-group myResourceGroup --name myManagedIdentity --query id --output tsv)

# Get client ID of the user-assigned identity
clientID=$(az identity show --resource-group myResourceGroup --name myManagedIdentity --query clientId --output tsv)
```

Nous pouvons maintenant attribuer l’identité affectée par l’utilisateur à la machine virtuelle à l’aide de la commande [az vm identity assign](/cli/azure/vm/identity#az-vm-identity-assign) :

```azurecli
az vm identity assign --resource-group myResourceGroup --name myVM --identities $resourceID
```

Pour terminer la configuration, affichez la valeur de l’ID client, dont vous aurez besoin dans les prochaines étapes :

```bash
echo $clientID
```

## <a name="creating-a-mysql-user-for-your-managed-identity"></a>Création d’un utilisateur MySQL pour votre identité managée

À présent, connectez-vous en tant qu’administrateur Azure AD à votre base de données MySQL, puis exécutez les instructions SQL suivantes :

```sql
SET aad_auth_validate_oids_in_tenant = OFF;
CREATE AADUSER 'myuser' IDENTIFIED BY 'CLIENT_ID';
```

L’identité managée dispose désormais d’un accès lors de l’authentification avec le nom d’utilisateur `myuser` (remplacez-le par le nom de votre choix).

## <a name="retrieving-the-access-token-from-azure-instance-metadata-service"></a>Récupération du jeton d’accès à partir d’Azure Instance Metadata Service

Votre application peut désormais récupérer un jeton d’accès à partir d’Azure Instance Metadata Service et l’utiliser pour l’authentification auprès de la base de données.

Vous effectuez cette récupération de jeton en envoyant une requête HTTP à `http://169.254.169.254/metadata/identity/oauth2/token` et en transmettant les paramètres suivants :

* `api-version` = `2018-02-01`
* `resource` = `https://ossrdbms-aad.database.windows.net`
* `client_id` = `CLIENT_ID` (que vous avez récupérés précédemment)

Vous obtenez un résultat JSON qui contient un champ `access_token`. Cette valeur de texte longue est le jeton d’accès Managed Identity que vous devez utiliser comme mot de passe lors de la connexion à la base de données.

À des fins de test, vous pouvez exécuter les commandes suivantes dans votre interpréteur de commandes. Notez que `curl`, `jq` et le client `mysql` doivent être installés.

```bash
# Retrieve the access token
accessToken=$(curl -s 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fossrdbms-aad.database.windows.net&client_id=CLIENT_ID' -H Metadata:true | jq -r .access_token)

# Connect to the database
mysql -h SERVER --user USER@SERVER --enable-cleartext-plugin --password=$accessToken
```

Vous êtes maintenant connecté à la base de données que vous avez configurée précédemment.

## <a name="connecting-using-managed-identity-in-c"></a>Connexion à l’aide de Managed Identity dans C#

Cette section montre comment obtenir un jeton d’accès à l’aide de l’identité managée affectée par l’utilisateur de la machine virtuelle et comment l’utiliser pour appeler Azure Database pour MySQL. Azure Database pour MySQL prend en charge Azure AD Authentication en mode natif, et peut donc accepter directement des jetons d’accès obtenus à l’aide d’identités managées pour les ressources Azure. Lors de la création d’une connexion à MySQL, vous transmettez le jeton d’accès dans le champ du mot de passe.

Voici un exemple de code .NET pour l’ouverture d’une connexion à MySQL à l’aide d’un jeton d’accès. Pour permettre l’accès au point de terminaison de l’identité managée affectée par l’utilisateur de la machine virtuelle, ce code doit s’exécuter sur la machine virtuelle. Pour pouvoir utiliser la méthode de jeton d’accès,le logiciel .NET Framework 4.6 ou version ultérieure ou .NET Core 2.2 ou version ultérieure est requis. Remplacez les valeurs de HOST, USER, DATABASE et CLIENT_ID.

```csharp
using System;
using System.Net;
using System.IO;
using System.Collections;
using System.Collections.Generic;
using System.Text.Json;
using System.Text.Json.Serialization;
using System.Threading.Tasks;
using MySql.Data.MySqlClient;

namespace Driver
{
    class Script
    {
        // Obtain connection string information from the portal
        //
        private static string Host = "HOST";
        private static string User = "USER";
        private static string Database = "DATABASE";
        private static string ClientId = "CLIENT_ID";

        static async Task Main(string[] args)
        {
            //
            // Get an access token for MySQL.
            //
            Console.Out.WriteLine("Getting access token from Azure Instance Metadata service...");

            // Azure AD resource ID for Azure Database for MySQL is https://ossrdbms-aad.database.windows.net/
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fossrdbms-aad.database.windows.net&client_id=" + ClientId);
            request.Headers["Metadata"] = "true";
            request.Method = "GET";
            string accessToken = null;

            try
            {
                // Call managed identities for Azure resources endpoint.
                HttpWebResponse response = (HttpWebResponse)request.GetResponse();

                // Pipe response Stream to a StreamReader and extract access token.
                StreamReader streamResponse = new StreamReader(response.GetResponseStream());
                string stringResponse = streamResponse.ReadToEnd();
                var list = JsonSerializer.Deserialize<Dictionary<string, string>>(stringResponse);
                accessToken = list["access_token"];
            }
            catch (Exception e)
            {
                Console.Out.WriteLine("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
                System.Environment.Exit(1);
            }

            //
            // Open a connection to the MySQL server using the access token.
            //
            var builder = new MySqlConnectionStringBuilder
            {
                Server = Host,
                Database = Database,
                UserID = User,
                Password = accessToken,
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.Out.WriteLine("Opening connection using access token...");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "SELECT VERSION()";

                    using (var reader = await command.ExecuteReaderAsync())
                    {
                        while (await reader.ReadAsync())
                        {
                            Console.WriteLine("\nConnected!\n\nMySQL version: {0}", reader.GetString(0));
                        }
                    }
                }
            }
        }
    }
}
```

Quand elle est exécutée, cette commande donne une sortie semblable à celle-ci :

```
Getting access token from Azure Instance Metadata service...
Opening connection using access token...

Connected!

MySQL version: 5.7.27
```

## <a name="next-steps"></a>Étapes suivantes

* Passez en revue les concepts généraux pour [Authentification Azure Active Directory avec Azure Database pour MySQL](concepts-azure-ad-authentication.md)
