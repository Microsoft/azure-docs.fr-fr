---
title: Appel téléphonique à partir de Twilio (Java) | Microsoft Docs
description: Apprenez à passer un appel téléphonique à partir d'une page web à l'aide de Twilio dans une application Java sur Azure.
services: ''
documentationcenter: java
author: georgewallace
ms.assetid: 0381789e-e775-41a0-a784-294275192b1d
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 11/25/2014
ms.author: gwallace
ms.custom: devx-track-java
ms.openlocfilehash: cad25990d0ba7e27f4a90adc587f0c8a0a2c551a
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95521068"
---
# <a name="how-to-make-a-phone-call-using-twilio-in-a-java-application-on-azure"></a>Exécution d'un appel téléphonique à l'aide de Twilio dans une application Java sur Azure
L'exemple qui suit montre comment utiliser Twilio pour passer un appel depuis une page Web hébergée sur Azure. L’application finale demande à l'utilisateur les valeurs de l’appel téléphonique, comme illustré dans la capture d’écran qui suit.

![Formulaire d'appel Azure avec Twilio et Java][twilio_java]

Pour utiliser le code de cette rubrique, vous devrez effectuer les opérations suivantes :

1. Obtenir un compte Twilio et un jeton d'authentification. Pour bien démarrer avec Twilio, évaluez les tarifs sur le site [https://www.twilio.com/pricing][twilio_pricing]. Vous pouvez vous inscrire sur [https://www.twilio.com/try-twilio][try_twilio]. Pour plus d’informations sur l’API fournie par Twilio, consultez [https://www.twilio.com/api][twilio_api].
2. Obtenir le JAR Twilio. Sur [https://github.com/twilio/twilio-java][twilio_java_github], vous pouvez télécharger les sources GitHub et créer votre propre fichier JAR, ou télécharger un fichier JAR prédéfini (avec ou sans dépendances).
   Le code de cette rubrique a été écrit avec un JAR préconstruit TwilioJava-3.3.8-with-dependencies.
3. Ajouter le JAR à votre chemin de build Java.
4. Si vous utilisez Eclipse pour créer cette application Java, ajoutez le JAR Twilio à votre fichier WAR de déploiement d'application à l'aide de la fonction d'assembly de déploiement d'Eclipse. Si vous n'utilisez pas Eclipse pour créer cette application Java, assurez-vous que le JAR Twilio est inclus dans le même rôle Azure que votre application Java et qu'il est ajouté au chemin de classe de votre application.
5. Vérifiez que le keystore cacerts de votre JDK contient le certificat ESCA (Equifax Secure Certificate Authority) avec l'empreinte MD5 67:CB:9D:C0:13:24:8A:82:9B:B2:17:1E:D1:1B:EC:D4 (le numéro de série est 35:DE:F4:CF et que l'empreinte de SHA1 est D2:32:09:AD:23:D3:14:23:21:74:E4:0D:7F:9D:62:13:97:86:63:3A). Il s'agit du certificat d'autorité de certification du service [https://api.twilio.com][twilio_api_service], qui est appelé lorsque vous utilisez les API Twilio. Pour plus d’informations sur l’ajout de ce certificat d’autorité de certification au magasin cacert de votre JDK, consultez la page [Ajout d’un certificat au magasin de certificats d’autorité de certification Java][add_ca_cert].

En outre, il est vivement recommandé de bien connaître les informations de la page [Création d’une application Hello World à l’aide d’Azure Toolkit for Eclipse][azure_java_eclipse_hello_world] ou d’autres techniques d’hébergement pour les applications Java dans Azure si vous n’utilisez pas Eclipse.

## <a name="create-a-web-form-for-making-a-call"></a>Création d'un formulaire web pour passer un appel
Le code qui suit présente la conception d'un formulaire Web qui extrait les données des utilisateurs pour passer un appel. Pour cet exemple, un projet Web dynamique **TwilioCloud** a été créé et le fichier JSP **callform.jsp** a été ajouté.

```jsp
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "https://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
        <title>Automated call form</title>
    </head>
    <body>
        <p>Fill in all fields and click <b>Make this call</b>.</p>
        <br/>
        <form action="makecall.jsp" method="post">
            <table>
                <tr>
                    <td>To:</td>
                    <td><input type="text" size=50 name="callTo" value="" />
                    </td>
                </tr>
                <tr>
                    <td>From:</td>
                    <td><input type="text" size=50 name="callFrom" value="" />
                    </td>
                </tr>
                <tr>
                    <td>Call message:</td>
                    <td><input type="text" size=400 name="callText" value="Hello. This is the call text. Good bye." />
                    </td>
                </tr>
                <tr>
                    <td colspan=2><input type="submit" value="Make this call" />
                    </td>
                </tr>
            </table>
        </form>
        <br/>
    </body>
</html>
```

## <a name="create-the-code-to-make-the-call"></a>Création du code pour passer l'appel
Le code qui suit, qui est appelé une fois que l'utilisateur remplit le formulaire affiché par callform.jsp, crée le message d'appel et lance l'appel. Dans cet exemple, le fichier JSP s'appelle **makecall.jsp** et il a été ajouté au projet **TwilioCloud**. (Utilisez votre compte Twilio et votre jeton d'authentification plutôt que les valeurs par défaut utilisées dans **accountSID** et **authToken** dans le code qui suit.)

```jsp
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
import="java.util.*"
import="com.twilio.*"
import="com.twilio.sdk.*"
import="com.twilio.sdk.resource.factory.*"
import="com.twilio.sdk.resource.instance.*"
pageEncoding="ISO-8859-1" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "https://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
        <title>Call processing happens here</title>
    </head>
    <body>
        <b>This is my make call page.</b><p/>
<%
try 
{
    // Use your account SID and authentication token instead
    // of the placeholders shown here.
    String accountSID = "your_twilio_account";
    String authToken = "your_twilio_authentication_token";

    // Instantiate an instance of the Twilio client.     
    TwilioRestClient client;
    client = new TwilioRestClient(accountSID, authToken);

    // Retrieve the account, used later to retrieve the CallFactory.
    Account account = client.getAccount();

    // Display the client endpoint. 
    out.println("<p>Using Twilio endpoint " + client.getEndpoint() + ".</p>");

    // Display the API version.
    String APIVERSION = TwilioRestClient.DEFAULT_VERSION;
    out.println("<p>Twilio client API version is " + APIVERSION + ".</p>");

    // Retrieve the values entered by the user.
    String callTo = request.getParameter("callTo");  
    // The Outgoing Caller ID, used for the From parameter,
    // must have previously been verified with Twilio.
    String callFrom = request.getParameter("callFrom");
    String userText = request.getParameter("callText");

    // Replace spaces in the user's text with '%20', 
    // to make the text suitable for a URL.
    userText = userText.replace(" ", "%20");

    // Create a URL using the Twilio message and the user-entered text.
    String Url="https://twimlets.com/message";
    Url = Url + "?Message%5B0%5D=" + userText;

    // Display the message URL.
    out.println("<p>");
    out.println("The URL is " + Url);
    out.println("</p>");

    // Place the call From, To and URL values into a hash map. 
    HashMap<String, String> params = new HashMap<String, String>();
    params.put("From", callFrom);
    params.put("To", callTo);
    params.put("Url", Url);

    CallFactory callFactory = account.getCallFactory();
    Call call = callFactory.create(params);
    out.println("<p>Call status: " + call.getStatus()  + "</p>"); 
} 
catch (TwilioRestException e) 
{
    out.println("<p>TwilioRestException encountered: " + e.getMessage() + "</p>");
    out.println("<p>StackTrace: " + e.getStackTrace().toString() + "</p>");
}
catch (Exception e) 
{
    out.println("<p>Exception encountered: " + e.getMessage() + "");
    out.println("<p>StackTrace: " + e.getStackTrace().toString() + "</p>");
}
%>
    </body>
</html>
```

En plus de passer l'appel, makecall.jsp affiche le point de terminaison Twilio, la version de l'API et l'état de l'appel. Voici un exemple dans la capture d’écran suivante :

![Réponse d'appel Azure avec Twilio et Java][twilio_java_response]

## <a name="run-the-application"></a>Exécution de l'application
Cette section décrit la procédure générale d’exécution de votre application. Ces étapes sont détaillées sur la page [Création d’une application Hello World à l’aide d’Azure Toolkit for Eclipse][azure_java_eclipse_hello_world].

1. Exportez votre WAR TwilioCloud dans le dossier Azure **approot** . 
2. Modifiez **startup.cmd** pour décompresser votre WAR TwilioCloud.
3. Compilez votre application pour l'émulateur de calcul.
4. Démarrez votre déploiement dans l'émulateur de calcul.
5. Ouvrez un navigateur, puis exécutez `http://localhost:8080/TwilioCloud/callform.jsp`.
6. Entrez les valeurs dans le formulaire, cliquez sur **Make this call**, puis consultez les résultats dans makecall.jsp.

Dès que vous êtes prêt pour le déploiement sur Azure, effectuez une recompilation pour déploiement dans le cloud, déployez sur Azure et accédez à l'adresse http://*votre_nom_hébergé*.cloudapp.net/TwilioCloud/callform.jsp dans le navigateur (remplacez *votre_nom_hébergé* par votre valeur).

## <a name="next-steps"></a>Étapes suivantes
Ce code vous est fourni afin de vous présenter les fonctions de base de l'utilisation de Twilio dans Java sur Azure. Avant d’effectuer le déploiement de production sur Azure, vous pouvez ajouter d’autres fonctionnalités telles que la gestion des erreurs. Par exemple :

* Au lieu d'utiliser un formulaire web, vous pouvez utiliser des objets blob de stockage Azure ou SQL Database pour stocker les numéros de téléphone et le texte des appels. Pour plus d'informations sur l'utilisation d'objets blob de stockage Azure dans Java, consultez la page [Utilisation du service de stockage d'objets blob de Java][howto_blob_storage_java]. 
* Vous pouvez utiliser **RoleEnvironment.getConfigurationSettings** pour récupérer l’ID du compte Twilio et le jeton d’authentification à partir des paramètres de configuration de votre déploiement, au lieu de coder les valeurs en dur dans makecall.jsp. Pour plus d’informations sur la classe **RoleEnvironment**, voir [Utiliser la bibliothèque Azure Service Runtime dans JSP][azure_runtime_jsp].
* Le code makecall.jsp attribue une URL fournie par Twilio, [https://twimlets.com/message][twimlet_message_url] à la variable **URL**. Cette URL fournit une réponse TwiML (Twilio Markup Language) qui précise à Twilio comment traiter l'appel. Par exemple, le code TwiML renvoyé peut contenir un verbe **&lt;Say&gt;** qui fait que le texte est lu au destinataire de l'appel. Au lieu d’utiliser l’URL fournie par Twilio, vous pouvez concevoir votre propre service pour répondre à la demande de Twilio. Pour plus d’informations, consultez l’article [Utilisation de Twilio pour les fonctionnalités vocales et de SMS dans Java][howto_twilio_voice_sms_java]. Des informations supplémentaires sur TwiML sont disponibles sur [https://www.twilio.com/docs/api/twiml][twiml]. Des informations supplémentaires sur **&lt; Say&gt;** et sur d’autres verbes Twilio sont disponibles à l’adresse [https://www.twilio.com/docs/api/twiml/say][twilio_say].
* Lisez les instructions de sécurité Twilio disponibles à l’adresse [https://www.twilio.com/docs/security][twilio_docs_security].

Pour obtenir des informations supplémentaires sur Twilio, consultez [https://www.twilio.com/docs][twilio_docs].

## <a name="see-also"></a>Voir aussi
* [Utilisation de Twilio pour les fonctionnalités vocales et de SMS dans Java][howto_twilio_voice_sms_java]
* [Ajout d’un certificat au magasin de certificats d’autorité de certification Java][add_ca_cert]

[twilio_pricing]: https://www.twilio.com/pricing
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_api]: https://www.twilio.com/api
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
[twilio_java_github]: https://github.com/twilio/twilio-java
[twimlet_message_url]: https://twimlets.com/message
[twiml]: https://www.twilio.com/docs/api/twiml
[twilio_api_service]: https://api.twilio.com
[add_ca_cert]: /azure/developer/java/sdk/java-sdk-add-certificate-ca-store
[azure_java_eclipse_hello_world]: /java/azure/eclipse/azure-toolkit-for-eclipse-create-hello-world-web-app
[howto_twilio_voice_sms_java]: partner-twilio-java-how-to-use-voice-sms.md
[howto_blob_storage_java]: https://www.windowsazure.com/develop/java/how-to-guides/blob-storage/
[howto_sql_azure_java]: https://msdn.microsoft.com/library/windowsazure/hh749029.aspx
[azure_runtime_jsp]: /previous-versions/azure/hh690948(v=azure.100)
[twilio_docs_security]: https://www.twilio.com/docs/security
[twilio_docs]: https://www.twilio.com/docs
[twilio_say]: https://www.twilio.com/docs/api/twiml/say
[twilio_java]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaCallForm.jpg
[twilio_java_response]: ./media/partner-twilio-java-phone-call-example/WA_TwilioJavaMakeCall.jpg