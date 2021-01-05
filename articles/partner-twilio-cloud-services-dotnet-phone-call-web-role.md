---
title: Appel téléphonique à partir de Twilio (.NET) | Microsoft Docs
description: Découvrez comment passer un appel téléphonique avec le service d’API Twilio sur Azure. Exemples de code écrits en .NET.
services: ''
documentationcenter: .net
author: mimckitt
editor: ''
ms.assetid: 789185ad-69dc-4e9e-a936-42e0a25315c8
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 05/04/2016
ms.author: mimckitt
ms.custom: devx-track-dotnet
ms.openlocfilehash: 22e0a19c0e2f5b29b35ebf43a3f250959c9b636d
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95521119"
---
# <a name="how-to-make-a-phone-call-using-twilio-in-a-web-role-on-azure"></a>Exécution d'un appel téléphonique à l'aide de Twilio dans un rôle Web sur Azure
Ce guide montre comment utiliser Twilio pour passer un appel à partir d'une page Web hébergée dans Azure. L’application obtenue invite l’utilisateur à passer un appel avec le numéro et le message donnés, comme illustré dans la capture d’écran suivante.

![Formulaire d'appel Azure utilisant Twilio et ASP.NET][twilio_dotnet_basic_form]

## <a name="prerequisites"></a><a name="twilio-prereqs"></a>Configuration requise
Pour pouvoir utiliser le code de cette rubrique, vous devez préalablement :

1. Acquérir un compte et un jeton d’authentification Twilio à partir de la [console Twilio][twilio_console]. Pour prendre en main Twilio, inscrivez-vous sur [https://www.twilio.com/try-twilio][try_twilio]. Vous pouvez évaluer les tarifs sur [https://www.twilio.com/pricing][twilio_pricing]. Pour plus d’informations sur l’API fournie par Twilio, consultez [https://www.twilio.com/voice/api][twilio_api].
2. Ajouter la *bibliothèque .NET Twilio* à votre rôle web. Voir **Pour ajouter les bibliothèques Twilio à votre projet de rôle Web**, plus loin dans cette rubrique.

Vous devriez également savoir comment créer un [rôle web de base sur Azure][azure_webroles_get_started].

## <a name="how-to-create-a-web-form-for-making-a-call"></a><a name="howtocreateform"></a>Procédure : création d’un formulaire web pour établir un appel
<a id="use_nuget"></a>Pour ajouter les bibliothèques Twilio à votre projet de rôle web :

1. Ouvrez votre solution dans Visual Studio.
2. Cliquez avec le bouton droit sur **Références**.
3. Cliquez sur **Gérer les packages NuGet...**
4. Cliquez sur **En ligne**.
5. Dans la zone de recherche en ligne, tapez *twilio*.
6. Cliquez sur le bouton **Installer** du package Twilio.

Le code qui suit présente la conception d'un formulaire Web qui extrait les données des utilisateurs pour passer un appel. Dans cet exemple, un rôle web ASP.NET appelé **TwilioCloud** est créé.

```aspx
<%@ Page Title="Home Page" Language="C#" MasterPageFile="~/Site.master"
    AutoEventWireup="true" CodeBehind="Default.aspx.cs"
    Inherits="WebRole1._Default" %>

<asp:Content ID="HeaderContent" runat="server" ContentPlaceHolderID="HeadContent">
</asp:Content>
<asp:Content ID="BodyContent" runat="server" ContentPlaceHolderID="MainContent">
    <div>
        <asp:BulletedList ID="varDisplay" runat="server" BulletStyle="NotSet">
        </asp:BulletedList>
    </div>
    <div>
        <p>Fill in all fields and click <b>Make this call</b>.</p>
        <div>
            To:<br /><asp:TextBox ID="toNumber" runat="server" /><br /><br />
            Message:<br /><asp:TextBox ID="message" runat="server" /><br /><br />
            <asp:Button ID="callpage" runat="server" Text="Make this call"
                onclick="callpage_Click" />
        </div>
    </div>
</asp:Content>
```

## <a name="how-to-create-the-code-to-make-the-call"></a><a id="howtocreatecode"></a>Procédure : création du code pour établir l’appel
Le code suivant, qui est appelé lorsque l'utilisateur remplit le formulaire, crée un message d'appel et génère l'appel. Dans cet exemple, le code est exécuté dans le gestionnaire d'événements sur clic du bouton du formulaire. (Utilisez votre compte Twilio et votre jeton d’authentification plutôt que les valeurs d’espace réservé affectées à `accountSID` et à `authToken` dans le code ci-dessous.)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using Twilio;
using Twilio.Http;
using Twilio.Types;
using Twilio.Rest.Api.V2010;

namespace WebRole1
{
    public partial class _Default : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {

        }

        protected void callpage_Click(object sender, EventArgs e)
        {
            // Call processing happens here.

            // Use your account SID and authentication token instead of
            // the placeholders shown here.
            var accountSID = "ACNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN";
            var authToken =  "NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN";

            // Instantiate an instance of the Twilio client.
            TwilioClient.Init(accountSID, authToken);

            // Retrieve the account, used later to retrieve the
            var account = AccountResource.Fetch(accountSID);

            this.varDisplay.Items.Clear();

            if (this.toNumber.Text == "" || this.message.Text == "")
            {
                this.varDisplay.Items.Add(
                        "You must enter a phone number and a message.");
            }
            else
            {
                // Retrieve the values entered by the user.
                var to = PhoneNumber(this.toNumber.Text);
                var from = new PhoneNumber("+14155992671");
                var myMessage = this.message.Text;

                // Create a URL using the Twilio message and the user-entered
                // text. You must replace spaces in the user's text with '%20'
                // to make the text suitable for a URL.
                var url = $"https://twimlets.com/message?Message%5B0%5D={myMessage.Replace(" ", "%20")}";
                var twimlUri = new Uri(url);

                // Display the endpoint, API version, and the URL for the message.
                this.varDisplay.Items.Add($"Using Twilio endpoint {
                }");
                this.varDisplay.Items.Add($"Twilioclient API Version is {apiVersion}");
                this.varDisplay.Items.Add($"The URL is {url}");

                // Place the call.
                var call = CallResource.create(to, from, url: twimlUri);
                this.varDisplay.Items.Add("Call status: " + call.Status);
            }
        }
    }
}
```

L'appel est passé, et le point de terminaison Twilio, la version de l'API et l'état de l'appel sont affichés. La capture d'écran ci-dessous présente la sortie d'un exemple d'exécution.

![Réponse Azure à un appel à l'aide de Twilio et ASP.NET][twilio_dotnet_basic_form_output]

Des informations supplémentaires sur TwiML sont disponibles sur [https://www.twilio.com/docs/api/twiml][twiml]. Pour plus d’informations sur &lt;Dire (Say)&gt; et d’autres verbes Twilio, consultez [https://www.twilio.com/docs/api/twiml/say][twilio_say].

## <a name="next-steps"></a><a id="nextsteps"></a>Étapes suivantes
Ce code vous est fourni afin de vous présenter les fonctions de base de l'utilisation de Twilio dans un rôle Web ASP.NET sur Azure. Avant d’effectuer le déploiement de production sur Azure, vous pouvez ajouter d’autres fonctionnalités telles que la gestion des erreurs. Par exemple :

* Au lieu d’utiliser un formulaire web, vous pouvez utiliser un stockage Blob Azure ou une instance Azure SQL Database pour stocker les numéros de téléphone et le texte d’appel. Pour plus d’informations sur l’utilisation des objets blob Azure, consultez la page [Utilisation du service de stockage d’objets blob Azure dans .NET][howto_blob_storage_dotnet]. Pour plus d’informations sur l’utilisation de SQL Database, consultez la page [Utilisation d’Azure SQL Database dans des applications .NET][howto_sql_azure_dotnet].
* Au lieu de coder en dur les valeurs de votre formulaire, vous pouvez utiliser `RoleEnvironment.getConfigurationSettings` pour récupérer l’ID du compte Twilio et le jeton d’authentification à partir des paramètres de configuration de votre déploiement. Pour plus d’informations sur la `RoleEnvironment` classe , consultez la page [Espace de noms Microsoft.WindowsAzure.ServiceRuntime][azure_runtime_ref_dotnet].
* Lisez les instructions de sécurité Twilio disponibles à l’adresse [https://www.twilio.com/docs/security][twilio_docs_security].
* Découvrez-en davantage sur Twilio sur [https://www.twilio.com/docs][twilio_docs].

## <a name="see-also"></a><a name="seealso"></a>Voir aussi
* [Utilisation de Twilio pour les fonctionnalités vocales et de SMS depuis Azure](twilio-dotnet-how-to-use-for-voice-sms.md)

[twilio_console]: https://www.twilio.com/console
[twilio_pricing]: https://www.twilio.com/pricing
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_api]: https://www.twilio.com/voice/api
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified

[twilio_dotnet_basic_form]: ./media/partner-twilio-cloud-services-dotnet-phone-call-web-role/WA_twilio_dotnet_basic_form.png
[twilio_dotnet_basic_form_output]: ./media/partner-twilio-cloud-services-dotnet-phone-call-web-role/WA_twilio_dotnet_basic_form_output.png

[twiml]: https://www.twilio.com/docs/api/twiml



[howto_twilio_voice_sms_dotnet]: /develop/net/how-to-guides/twilio/

[howto_blob_storage_dotnet]: https://www.windowsazure.com/develop/net/how-to-guides/blob-storage/

[howto_sql_azure_dotnet]: https://www.windowsazure.com/develop/net/how-to-guides/sql-database/


[twilio_docs_security]: https://www.twilio.com/docs/security
[twilio_docs]: https://www.twilio.com/docs
[twilio_say]: https://www.twilio.com/docs/api/twiml/say


[azure_runtime_ref_dotnet]: /previous-versions/azure/reference/ee741722(v=azure.100)
[azure_webroles_get_started]: ./cloud-services/cloud-services-dotnet-get-started.md