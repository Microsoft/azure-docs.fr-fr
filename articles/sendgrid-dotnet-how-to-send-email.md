---
title: Utilisation du service de messagerie électronique SendGrid (.NET) | Microsoft Docs
description: Découvrez comment envoyer un e-mail avec le service e-mail SendGrid dans Azure. Code samples written in C# and use the .NET API.
services: ''
documentationcenter: .net
author: georgewallace
ms.author: gwallace
editor: ''
ms.assetid: 21bf4028-9046-476b-9799-3d3082a0f84c
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 02/15/2017
ms.reviewer: dx@sendgrid.com
ms.custom: devx-track-dotnet
ms.openlocfilehash: e605e162e4639a876cc5b6d763e81fdd7ff93aa8
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97674546"
---
# <a name="how-to-send-email-using-sendgrid-with-azure"></a>Envoi de courriers électroniques à l'aide de SendGrid avec Azure
## <a name="overview"></a>Vue d’ensemble
Ce guide présente l'exécution de tâches de programmation courantes avec le service de messagerie SendGrid dans Azure. Les exemples sont écrits en C\# et prennent en charge .NET Standard 1.3. Les scénarios traités incluent le développement d’une messagerie électronique, l’envoi de courriers électroniques, l’ajout de pièces jointes et l’activation de différents paramètres de messagerie et de suivi. Pour plus d'informations sur SendGrid et l'envoi de courriers électroniques, consultez la section [Étapes suivantes][Next steps] .

## <a name="what-is-the-sendgrid-email-service"></a>Définition du service de messagerie SendGrid
SendGrid est un [service de messagerie dans le cloud] qui fournit des fonctionnalités fiables en matière de [remise de courrier électronique transactionnelle], d'extensibilité et d'analyse en temps réel, ainsi que des API flexibles qui facilitent l'intégration personnalisée. Voici quelques cas d’utilisation courants de SendGrid :

* Envoi automatique de reçus ou de confirmations d’achat aux clients
* Gestion de listes de distribution pour l’envoi mensuel de brochures et de promotions aux clients
* Collecte de mesures en temps réel, notamment en ce qui concerne les e-mails bloqués et l’engagement des clients
* transfert des demandes de renseignements des clients.
* traitement des messages électroniques entrants.

Pour plus d’informations, consultez le site [https://sendgrid.com](https://sendgrid.com) ou le dépôt GitHub [Bibliothèque C#][sendgrid-csharp] de SendGrid.

## <a name="create-a-sendgrid-account"></a>Création d'un compte SendGrid
[!INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="reference-the-sendgrid-net-class-library"></a>Référencement de la bibliothèque de classes .NET de SendGrid
Le [package NuGet SendGrid](https://www.nuget.org/packages/Sendgrid) permet facilement d'obtenir l'API SendGrid et de configurer votre application avec toutes les dépendances associées. NuGet est une extension Visual Studio incluse dans Microsoft Visual Studio 2015 et ses versions ultérieures qui facilite l’installation et la mise à jour des bibliothèques et des outils.

> [!NOTE]
> Pour installer NuGet si vous exécutez une version de Visual Studio antérieure à Visual Studio 2015, consultez la page [https://www.nuget.org](https://www.nuget.org), puis cliquez sur le bouton **Install NuGet**.
>
>

Pour installer le package NuGet SendGrid dans votre application, procédez comme suit :

1. Cliquez sur **Nouveau projet**, puis sélectionnez un **Modèle**.

   ![Création d'un projet][create-new-project]
2. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur **References**, puis cliquez sur **Manage NuGet Packages**.

   ![package NuGet SendGrid][SendGrid-NuGet-package]
3. Recherchez **SendGrid**, puis sélectionnez l'élément **SendGrid** dans la liste de résultats.
4. Sélectionnez la dernière version stable du package Nuget dans la liste déroulante de versions pour être en mesure d’utiliser le modèle objet et les API décrits dans cet article.

   ![Package SendGrid][sendgrid-package]
5. Cliquez sur **Install** pour terminer l'installation, puis fermez cette boîte de dialogue.

La bibliothèque de classes .NET de SendGrid est appelée **SendGrid**. Elle contient les espaces de noms suivants :

* **SendGrid** pour la communication avec l’API de SendGrid
* **SendGrid.Helpers.Mail** pour les méthodes d’assistance destinées à faciliter la création d’objets SendGridMessage qui spécifient le mode d’envoi des e-mails

Ajoutez les déclarations d'espace de noms de code suivantes en haut de chaque fichier C# dans lequel vous souhaitez accéder au service de messagerie SendGrid par programme.

```csharp
using SendGrid;
using SendGrid.Helpers.Mail;
```

## <a name="how-to-create-an-email"></a>Procédure : créer un e-mail
Pour créer un message électronique, utilisez l'objet **SendGridMessage** . Une fois que l'objet du message est créé, vous pouvez définir les propriétés et les méthodes, notamment l'expéditeur, le destinataire, l'objet et le corps du message électronique.

L'exemple suivant montre comment créer un objet de message entièrement renseigné :

```csharp
var msg = new SendGridMessage();

msg.SetFrom(new EmailAddress("dx@example.com", "SendGrid DX Team"));

var recipients = new List<EmailAddress>
{
    new EmailAddress("jeff@example.com", "Jeff Smith"),
    new EmailAddress("anna@example.com", "Anna Lidman"),
    new EmailAddress("peter@example.com", "Peter Saddow")
};
msg.AddTos(recipients);

msg.SetSubject("Testing the SendGrid C# Library");

msg.AddContent(MimeType.Text, "Hello World plain text!");
msg.AddContent(MimeType.Html, "<p>Hello World!</p>");
```

Pour plus d'informations sur les propriétés et méthodes prises en charge par le type **SendGrid** , consultez la page [sendgrid-csharp][sendgrid-csharp] sur GitHub.

## <a name="how-to-send-an-email"></a>Procédure : envoyer un e-mail
Après avoir créé un e-mail, vous pouvez l’envoyer à l’aide de l’API de SendGrid. Une autre possibilité consiste à utiliser la [bibliothèque intégrée de .NET][NET-library].

Pour envoyer des e-mails, vous devez fournir votre clé API SendGrid. Pour plus d’informations sur la configuration des clés API, consultez la [documentation][documentation] des clés API de SendGrid.

Vous pouvez stocker ces informations d’identification par le biais de votre portail Azure en cliquant sur Paramètres de l’application et en ajoutant les paires clé/valeur sous Paramètres de l’application.

![Paramètres de l’application Azure][azure_app_settings]

Vous pouvez ensuite y accéder comme suit :

```csharp
var apiKey = System.Environment.GetEnvironmentVariable("SENDGRID_APIKEY");
var client = new SendGridClient(apiKey);
```

Les exemples suivants montrent comment envoyer un e-mail en utilisant l’API web SendGrid avec une application console.

```csharp
using System;
using System.Threading.Tasks;
using SendGrid;
using SendGrid.Helpers.Mail;

namespace Example
{
    internal class Example
    {
        private static void Main()
        {
            Execute().Wait();
        }

        static async Task Execute()
        {
            var apiKey = System.Environment.GetEnvironmentVariable("SENDGRID_APIKEY");
            var client = new SendGridClient(apiKey);
            var msg = new SendGridMessage()
            {
                From = new EmailAddress("test@example.com", "DX Team"),
                Subject = "Hello World from the SendGrid CSharp SDK!",
                PlainTextContent = "Hello, Email!",
                HtmlContent = "<strong>Hello, Email!</strong>"
            };
            msg.AddTo(new EmailAddress("test@example.com", "Test User"));
            var response = await client.SendEmailAsync(msg);
        }
    }
}
```

## <a name="how-to-send-email-from-asp-net-core-api-using-mailhelper-class"></a>Procédure : envoyer un e-mail à partir de l’API ASP .NET Core avec la classe MailHelper

L’exemple ci-dessous peut être utilisé pour envoyer un même e-mail à plusieurs personnes à partir de l’API ASP .NET Core avec la classe `MailHelper` de l’espace de noms `SendGrid.Helpers.Mail`. Pour cet exemple, nous utilisons ASP .NET Core 1.0. 

Dans cet exemple, la clé d’API a été stockée dans le fichier `appsettings.json`, qui peut être remplacé à partir du portail Azure, comme illustré dans les exemples ci-dessus.

Le contenu du fichier `appsettings.json` doit se présenter ainsi :

```json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    }
  },
  "SENDGRID_API_KEY": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}
```

Nous devons d’abord ajouter le code ci-dessous dans le fichier `Startup.cs` du projet d’API .NET Core. Ceci est nécessaire pour pouvoir accéder à `SENDGRID_API_KEY` à partir du fichier `appsettings.json` en utilisant l’injection de dépendances dans le contrôleur d’API. L’interface `IConfiguration` peut être injectée dans le constructeur du contrôleur après son ajout dans la méthode `ConfigureServices` ci-dessous. Le contenu du fichier `Startup.cs` se présente comme ceci après l’ajout du code nécessaire :

```csharp
    public IConfigurationRoot Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        // Add mvc here
        services.AddMvc();
        services.AddSingleton<IConfiguration>(Configuration);
    }
```

Au niveau du contrôleur, après l’injection de l’interface `IConfiguration`, nous pouvons utiliser la méthode `CreateSingleEmailToMultipleRecipients` de la classe `MailHelper` pour envoyer un même e-mail à plusieurs destinataires. La méthode accepte un paramètre booléen supplémentaire nommé `showAllRecipients`. Ce paramètre peut être utilisé pour contrôler si les destinataires de l’e-mail pourront voir l’adresse e-mail des autres dans la section À de l’en-tête de l’e-mail. L’exemple de code pour le contrôleur doit se présenter comme ci-dessous : 

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using SendGrid;
using SendGrid.Helpers.Mail;
using Microsoft.Extensions.Configuration;

namespace SendgridMailApp.Controllers
{
    [Route("api/[controller]")]
    public class NotificationController : Controller
    {
       private readonly IConfiguration _configuration;

       public NotificationController(IConfiguration configuration)
       {
         _configuration = configuration;
       }

       [Route("SendNotification")]
       public async Task PostMessage()
       {
          var apiKey = _configuration.GetSection("SENDGRID_API_KEY").Value;
          var client = new SendGridClient(apiKey);
          var from = new EmailAddress("test1@example.com", "Example User 1");
          List<EmailAddress> tos = new List<EmailAddress>
          {
              new EmailAddress("test2@example.com", "Example User 2"),
              new EmailAddress("test3@example.com", "Example User 3"),
              new EmailAddress("test4@example.com","Example User 4")
          };

          var subject = "Hello world email from Sendgrid ";
          var htmlContent = "<strong>Hello world with HTML content</strong>";
          var displayRecipients = false; // set this to true if you want recipients to see each others mail id 
          var msg = MailHelper.CreateSingleEmailToMultipleRecipients(from, tos, subject, "", htmlContent, false);
          var response = await client.SendEmailAsync(msg);
      }
   }
}
```

## <a name="how-to-add-an-attachment"></a>Procédure : ajouter une pièce jointe
Vous pouvez ajouter des pièces jointes à un message en appelant la méthode **AddAttachment** et en spécifiant au minimum le nom du fichier et le contenu encodé en Base64 que vous souhaitez joindre au message. Si vous souhaitez inclure plusieurs pièces jointes, appelez cette méthode pour chacun des fichiers que vous voulez joindre ou utilisez la méthode **AddAttachments**. L'exemple suivant montre comment ajouter une pièce jointe à un message :

```csharp
var banner2 = new Attachment()
{
    Content = Convert.ToBase64String(raw_content),
    Type = "image/png",
    Filename = "banner2.png",
    Disposition = "inline",
    ContentId = "Banner 2"
};
msg.AddAttachment(banner2);
```

## <a name="how-to-use-mail-settings-to-enable-footers-tracking-and-analytics"></a>Procédure : utiliser des paramètres de messagerie pour activer les pieds de page, le suivi et l’analytique
SendGrid offre d’autres fonctionnalités de messagerie grâce à l’utilisation de paramètres de messagerie et de suivi. Vous pouvez ajouter ces paramètres à un message électronique pour activer des fonctionnalités spécifiques, telles que le suivi des clics, Google Analytics, le suivi des abonnements, etc. Pour obtenir la liste complète des applications, consultez la [documentation relative aux paramètres][settings-documentation].

Les applications peuvent être appliquées aux messages électroniques **SendGrid** à l’aide des méthodes implémentées dans le cadre de la classe **SendGridMessage**. Les exemples suivants montrent les filtres de pied de page et de suivi des clics :

Les exemples suivants montrent les filtres de pied de page et de suivi des clics :

### <a name="footer-settings"></a>Paramètres de pied de page

```csharp
    msg.SetFooterSetting(
        true,
        "Some Footer HTML",
        "<strong>Some Footer Text</strong>");
```

### <a name="click-tracking"></a>Suivi des clics

```csharp
msg.SetClickTracking(true);
```

## <a name="how-to-use-additional-sendgrid-services"></a>Procédure : utiliser des services SendGrid supplémentaires
SendGrid offre plusieurs API et webhooks que vous pouvez utiliser pour tirer profit de fonctionnalités supplémentaires dans votre application Azure. Pour plus d’informations, consultez la [documentation de référence sur l’API SendGrid][SendGrid API documentation].

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris les bases du service de messagerie SendGrid, consultez ces liens pour en savoir plus.

* Dépôt Bibliothèque C\# de SendGrid : [sendgrid-csharp][sendgrid-csharp]
* Documentation de l’API SendGrid : <https://sendgrid.com/docs>

[Next steps]: #next-steps
[What is the SendGrid Email Service?]: #whatis
[Create a SendGrid Account]: #createaccount
[Reference the SendGrid .NET Class Library]: #reference
[How to: Create an Email]: #createemail
[How to: Send an Email]: #sendemail
[How to: Add an Attachment]: #addattachment
[How to: Use Filters to Enable Footers, Tracking, and Analytics]: #usefilters
[How to: Use Additional SendGrid Services]: #useservices

[create-new-project]: ./media/sendgrid-dotnet-how-to-send-email/new-project.png
[SendGrid-NuGet-package]: ./media/sendgrid-dotnet-how-to-send-email/reference.png
[sendgrid-package]: ./media/sendgrid-dotnet-how-to-send-email/sendgrid-package.png
[azure_app_settings]: ./media/sendgrid-dotnet-how-to-send-email/azure-app-settings.png
[sendgrid-csharp]: https://github.com/sendgrid/sendgrid-csharp
[SMTP vs. Web API]: https://sendgrid.com/docs/Integrate/index.html
[App Settings]: https://sendgrid.com/docs/API_Reference/SMTP_API/apps.html
[SendGrid API documentation]: https://sendgrid.com/docs/API_Reference/api_v3.html
[NET-library]: https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html#-Using-NETs-Builtin-SMTP-Library
[documentation]: https://sendgrid.com/docs/Classroom/Send/api_keys.html
[settings-documentation]: https://sendgrid.com/docs/API_Reference/SMTP_API/apps.html

[service de messagerie dans le cloud]: https://sendgrid.com/solutions
[remise de courrier électronique transactionnelle]: https://sendgrid.com/use-cases/transactional-email

