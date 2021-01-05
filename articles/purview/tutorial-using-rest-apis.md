---
title: 'Tutoriel : Utiliser les API REST'
description: Ce tutoriel explique comment utiliser les API REST Azure Purview pour accéder au contenu de votre catalogue.
author: hophanms
ms.author: hophan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/03/2020
ms.openlocfilehash: 7a2e029fbcc1a26da9c8c8cbff5ee3b9bbb849e0
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928271"
---
# <a name="tutorial-use-the-rest-apis"></a>Tutoriel : Utiliser les API REST

Ce tutoriel montre comment utiliser les API REST Azure Purview. Toute personne qui souhaite envoyer des données dans un catalogue Azure Purview, inclure le catalogue dans le cadre d’un processus automatisé ou créer sa propre expérience utilisateur sur le catalogue peut utiliser les API REST pour le faire.

Dans ce tutoriel, vous apprenez à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Créer un principal de service (application).
> * Configurer votre catalogue pour approuver le principal de service (application).
> * Consulter la documentation sur les API REST.
> * Collecter les valeurs nécessaires pour utiliser les API REST.
> * Utiliser le client Postman pour appeler les API REST.
> * Générer un client .NET pour appeler les API REST.

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Pour commencer, vous devez disposer d’un compte Azure Purview existant. Si vous n’avez pas de catalogue, consultez le [guide de démarrage rapide pour créer un compte Azure Purview](create-catalog-portal.md).

* Si vous devez ajouter des données dans votre catalogue, consultez le [tutoriel pour exécuter le starter kit et analyser des données](tutorial-scan-data.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="create-a-service-principal-application"></a>Créer un principal de service (application)

Pour qu’un client d’API REST accède au catalogue, le client doit avoir un principal de service (application) et une identité que le catalogue reconnaît et approuve conformément à sa configuration. Lorsque vous adressez au catalogue des appels d’API REST, ils utilisent l’identité du principal de service.

Les clients qui ont utilisé des principaux de service existants (ID d’application) ont connu un taux élevé d’échecs. Par conséquent, nous vous recommandons de créer un nouveau principal de service pour appeler les API.

Pour créer un principal de service :

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Azure Active Directory**.
1. Dans la page **Azure Active Directory**, sélectionnez **Inscriptions des applications** dans le volet de gauche.
1. Sélectionnez **Nouvelle inscription**.
1. Dans la page **Inscrire une application** :
    1. Entrez un **nom** pour l’application (nom du principal de service).
    1. Sélectionnez **Comptes dans cet annuaire d’organisation uniquement ( _&lt;nom de votre locataire&gt;_ uniquement - Locataire unique)** .
    1. Pour **URI de redirection (facultatif)** , sélectionnez **Web** et entrez une valeur. Cette valeur n’a pas besoin d’être un URI valide.
    1. Sélectionnez **Inscription**.
1. Dans la page du nouveau principal de service, copiez les valeurs des paramètres **Nom d’affichage** et **ID d’application (client)** à enregistrer pour plus tard.

   L’ID d’application est la valeur `client_id` dans l’exemple de code.

Pour utiliser le principal de service (application), vous devez obtenir son mot de passe. Voici comment faire :

1. Dans le portail Azure, recherchez et sélectionnez **Azure Active Directory**, puis sélectionnez **Inscriptions d’applications** dans le volet de gauche.
1. Sélectionnez votre principal de service (application) dans la liste.
1. Sélectionnez **Certificats et secrets** dans le volet de gauche.
1. Sélectionnez **Nouveau secret client**.
1. Dans la page **Ajouter un secret client**, entrez une **description**, sélectionnez un délai d’expiration sous **Expire**, puis sélectionnez **Ajouter**.

   Dans la page **Secrets client**, la chaîne dans la colonne **Valeur** de votre nouveau secret est votre mot de passe. Enregistrez cette valeur.

   :::image type="content" source="./media/tutorial-using-rest-apis/client-secret.png" alt-text="Capture d’écran montrant un secret client":::

## <a name="configure-your-catalog-to-trust-the-service-principal-application"></a>Configurer votre catalogue pour approuver le principal de service (application)

Pour configurer Azure Purview afin qu’il approuve votre nouveau principal de service :

1. Accédez à votre compte Purview.

1. Dans la page **Compte Purview**, sélectionnez l’onglet **Contrôle d’accès (IAM)** .

1. Cliquez sur **+ Ajouter**

1. Sélectionnez **Ajouter une attribution de rôle**.

1. Pour Rôle, entrez **Conservateur de données Purview**.

    > [!Note]
    > Pour plus d’informations sur les autorisations du catalogue de données Azure Purview, consultez [Autorisations du catalogue](catalog-permissions.md). Par exemple, si vous avez besoin d’une autorisation pour déclencher l’analyse, le type de rôle doit être **Administrateur de sources de données Purview**.

1. Pour **Attribuer l’accès à**, laissez la valeur par défaut **Utilisateur, groupe ou principal de service**.

1. Pour **Sélectionner**, entrez le nom de l’utilisateur, du groupe Azure Active Directory ou du principal de service que vous souhaitez affecter, puis cliquez sur son nom dans le volet des résultats.

1. Dans le menu Paramètres, cliquez sur **Enregistrer**

Vous avez maintenant configuré le principal de service en tant qu’administrateur d’application, ce qui lui permet d’envoyer du contenu dans le catalogue.

## <a name="view-the-rest-apis-documentation"></a>Consulter la documentation sur les API REST

Pour afficher la documentation Swagger d’API, téléchargez [PurviewCatalogAPISwagger.zip](https://github.com/Azure/Purview-Samples/raw/master/rest-api/PurviewCatalogAPISwagger.zip), extrayez ses fichiers, puis ouvrez index.html.

Si vous souhaitez en savoir plus sur l’API de recherche avancée/suggestion fournie par Azure Purview, consultez la documentation sur les filtres de recherche d’API REST. Le client généré par AutoRest ne prend actuellement pas en charge les paramètres de recherche personnalisés. Pour contourner ce problème, suivez le document de filtre de recherche pour définir des classes de filtre dans le code sous la forme de paramètres d’appel d’API. Le document index.html contient des exemples de ces API.

## <a name="collect-the-necessary-values-to-use-the-rest-apis"></a>Collecter les valeurs nécessaires pour utiliser les API REST

Recherchez et enregistrez les valeurs suivantes :

* ID de locataire :
  * Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Azure Active Directory**.
  * Dans la section **Gérer** dans le volet gauche, sélectionnez **Propriétés**, recherchez l’**ID de locataire**, puis sélectionnez l’icône **Copier dans le Presse-papiers** pour enregistrer sa valeur.
* Point de terminaison Atlas :
  * Dans la [page des comptes Azure Purview](https://aka.ms/purviewportal) dans le portail Azure, recherchez et sélectionnez votre compte Azure Purview dans la liste.
  * Sélectionnez **Vue d’ensemble**, recherchez **Point de terminaison Atlas**, puis sélectionnez l’icône **Copier dans le Presse-papiers** pour enregistrer sa valeur. Supprimez la partie *https://* de la chaîne quand vous l’utilisez ultérieurement.
* Nom du compte :
  * Extrayez le nom de votre catalogue à partir de la chaîne du point de terminaison Atlas. Par exemple, si votre point de terminaison Atlas est `https://ThisIsMyCatalog.catalog.purview.azure.com`, le nom de votre compte est `ThisIsMyCatalog`.

## <a name="use-the-postman-client-to-call-the-rest-apis"></a>Utiliser le client Postman pour appeler les API REST

1. Installer le [client Postman](https://www.getpostman.com/).
1. À partir du client, sélectionnez **Importer** et utilisez [Test.postman_collection.json](https://raw.githubusercontent.com/Azure/Purview-Samples/master/rest-api/Test.postman_collection.json).
1. Sélectionnez **Collections**, puis **Test**.
1. Sélectionnez **Obtenir un jeton** :
    1. Dans l’URL en regard de POST, remplacez *&lt;your-tenant-id&gt;* par l’ID de locataire que vous avez copié dans la section précédente.
    1. Sélectionnez l’onglet **Corps** et remplacez les espaces réservés dans le chemin d’accès et le corps de l’étape précédente.

       Une fois que vous avez sélectionné **Envoyer**, le corps de la réponse contient une structure JSON incluant le nom *access_token* et une valeur de chaîne entre guillemets. 
    1. Copiez la valeur du jeton du porteur (sans les guillemets) en vue de l’utiliser à l’étape suivante.

1. Sélectionnez **/v2/types/typedefs** :
    1. Remplacez l’espace réservé dans le chemin par la valeur de votre point de terminaison Atlas. Cette valeur peut être obtenue en accédant à l’instance de catalogue sur le portail Ibiza et en cliquant sur Vue d’ensemble. 

       Le jeton du porteur est défini à partir de la première étape (ou vous pouvez le copier manuellement dans l’onglet « Autorisation »).

    1. Sélectionnez **Envoyer** pour obtenir la réponse.

## <a name="generate-a-net-client-to-call-the-rest-apis"></a>Générer un client .NET pour appeler les API REST

### <a name="install-autorest"></a>Installez AutoRest.



1. [Installez Node.js](https://github.com/Azure/autorest/blob/master/docs/installing-autorest.md).
1. Ouvrez PowerShell et exécutez la commande suivante :

   ```powershell
   npm install -g autorest@V2
   ```

### <a name="download-rest-api-specszip-and-create-the-client"></a>Télécharger rest-api-specs.zip et créer le client

1. Téléchargez [rest-api-specs.zip](https://github.com/Azure/Purview-Samples/raw/master/rest-api/rest-api-specs.zip) et extrayez ses fichiers.
1. Exécutez la commande suivante dans PowerShell à partir du dossier rest-api-specs extrait :

   ```powershell
   autorest --input-file=data-plane/preview/purviewcatalog.json --csharp --output-folder=PurviewCatalogClient/CSharp --namespace=PurviewCatalog --add-credentials
   ```

   La sortie de ce processus crée un dossier PurviewCatalogClient et un sous-dossier CSharp dans le dossier rest-api-specs.

### <a name="create-a-sample-net-console-application"></a>Créer un exemple d’application console .NET

1. Ouvrez Visual Studio 2019. Ces instructions ont été testées avec l’édition Community gratuite.
1. À partir de la page **Créer un nouveau projet**, créez un projet **Application console (.NET Core)** dans C#.
1. Effectuez un copier-coller de l’[exemple de code](#sample-code-for-the-console-application) fourni.
1. Remplacez *accountName*, *servicePrincipalId*, *servicePrincipalKey* et *tenantId* par les valeurs que vous avez collectées précédemment.
1. Utilisez l’**Explorateur de solutions** pour ajouter un dossier nommé **Generated** dans le projet Visual Studio.
1. Copiez le dossier rest-api-specs\PurviewCatalogClient\CSharp que vous avez créé précédemment et collez-le dans le dossier \Generated. Utilisez l’Explorateur de fichiers ou l’invite de ligne de commande pour l’opération de copie, qui déclenchera l’ajout automatique par Visual Studio des fichiers au projet.
1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet et sélectionnez **Gérer les packages NuGet**.
1. Sélectionnez l’onglet **Parcourir**. Recherchez et sélectionnez **Microsoft.Rest.ClientRuntime**.
1. Assurez-vous que la version est au moins 2.3.21, puis sélectionnez **Installer**.
1. Générez et exécutez l’application.

L’exemple de code retourne le nombre de typedefs figurant dans le catalogue et montre comment gérer les attributions de rôles. Pour plus d’informations, consultez `DoRoleAssignmentOperations()` dans l’exemple de code. Pour plus d’informations sur le projet, consultez [Configuration du projet](https://github.com/Azure/autorest/blob/master/docs/client/proj-setup.md).

### <a name="sample-code-for-the-console-application"></a>Exemple de code pour l’application console

```csharp
using Microsoft.Rest;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Http;

namespace PurviewCatalogSdkTest
{
    class Program
    {
        private static string accountName = "{account-name}";
        private static string servicePrincipalId = "{service-principal-id}";
        private static string servicePrincipalKey = "{service-principal-key}";
        private static string tenantId = "{tenant-id}";

        static void Main(string[] args)
        {
            Console.WriteLine("Azure Purview client");

            string baseUri = string.Format("https://{0}.catalog.purview.azure.com/api", accountName);

            // Get token and set auth
            var svcClientCreds = new TokenCredentials(getToken(), "Bearer");
            var client = new PurviewCatalog.PurviewCatalogServiceRESTAPIDocument(svcClientCreds);
            client.BaseUri = new System.Uri(baseUri);

            // /v2/types/typedefs
            var task = client.TypesREST.GetAllTypeDefsWithHttpMessagesAsync();
            Console.WriteLine("\nURI:\n" + task.Result.Request.RequestUri);
            Console.WriteLine("\nStatus Code:\n" + task.Result.Response.StatusCode);
            Console.WriteLine("\nResult:\n" + JsonConvert.SerializeObject(task.Result.Body, Formatting.Indented));
        }

        // Replace client_id and client_secret with application ID and password value from service principal
        private static string getToken()
        {
            var values = new Dictionary<string, string>

            // The "resource" could be "https://purview.azure.net" or "73c2949e-da2d-457a-9607-fcc665198967"
            {
                { "grant_type", "client_credentials" },
                { "client_id", servicePrincipalId },
                { "client_secret", servicePrincipalKey },
                { "resource", "73c2949e-da2d-457a-9607-fcc665198967" }
            };

            string authUrl = string.Format("https://login.windows.net/{0}/oauth2/token", tenantId);
            var content = new FormUrlEncodedContent(values);

            HttpClient authClient = new HttpClient();
            var bearerResult = authClient.PostAsync(authUrl, content);
            bearerResult.Wait();
            var resultContent = bearerResult.Result.Content.ReadAsStringAsync();
            resultContent.Wait();
            var bearerToken = JObject.Parse(resultContent.Result)["access_token"].ToString();
            var svcClientCreds = new TokenCredentials(bearerToken, "Bearer");

            return bearerToken;
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Gérer les sources de données](manage-data-sources.md)
