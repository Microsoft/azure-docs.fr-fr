---
title: Utilisation du service de messagerie électronique SendGrid (Java) | Microsoft Docs
description: Découvrez comment envoyer un courrier électronique avec le service de messagerie SendGrid dans Azure. Exemples de code écrits en Java.
services: ''
documentationcenter: java
author: thinkingserious
manager: sendgrid
editor: mollybos
ms.assetid: cc75c43e-ede9-492b-98c2-9147fcb92c21
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 10/30/2014
ms.author: erikre
ms.reviewer: elmer.thomas@sendgrid.com; erika.berkland@sendgrid.com; vibhork
ms.custom: devx-track-java
ms.openlocfilehash: 9ff006b74b6202b02a2767aee4d853b1206ce60d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015449"
---
# <a name="how-to-send-email-using-sendgrid-from-java"></a>Envoi de courriers électroniques à l'aide de SendGrid depuis Java
Ce guide présente l'exécution de tâches de programmation courantes avec le service de messagerie SendGrid dans Azure. Les exemples sont écrits en Java. Les scénarios traités incluent la **construction** et **l'envoi de courriers électroniques**, **l'ajout de pièces jointes**, **l'utilisation de filtres**, et la **mise à jour de propriétés**. Pour plus d'informations sur SendGrid et l'envoi de courriers électroniques, consultez la section [Étapes suivantes](#next-steps) .

## <a name="what-is-the-sendgrid-email-service"></a>Définition du service de messagerie SendGrid
SendGrid est un [service de messagerie dans le cloud] qui fournit des fonctionnalités fiables en matière de [remise de courrier électronique transactionnelle], d'extensibilité et d'analyse en temps réel, ainsi que des API flexibles qui facilitent l'intégration personnalisée. Voici quelques scénarios courants en termes d'utilisation de SendGrid :

* Envoi automatique d'accusés de réception aux clients
* Administration de listes de distribution pour un envoi mensuel de prospectus électroniques et d'offres spéciales aux clients
* Collecte de mesures en temps réel concernant des éléments tels que les messages électroniques bloqués et la réactivité vis-à-vis des clients
* Génération de rapports pour identifier les tendances
* Transfert des demandes des clients
* Notifications par courriers électroniques depuis votre application

Pour plus d’informations, consultez <https://sendgrid.com>.

## <a name="create-a-sendgrid-account"></a>Création d'un compte SendGrid
[!INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="how-to-use-the-javaxmail-libraries"></a>Procédure : Utiliser les bibliothèques javax.mail
Obtenez les bibliothèques javax.mail (à partir de <https://www.oracle.com/technetwork/java/javamail>, par exemple) et importez-les dans votre code. Généralement, le processus d'utilisation des bibliothèques javax.mail pour envoyer des messages électroniques via SMTP se déroule comme suit :

1. Spécifiez les valeurs SMTP, notamment le serveur SMTP qui, dans le cas de SendGrid, est smtp.sendgrid.net.

    ```java
    import java.util.Properties;
    import javax.mail.*;
    import javax.mail.internet.*;

    public class MyEmailer {
        private static final String SMTP_HOST_NAME = "smtp.sendgrid.net";
        private static final String SMTP_AUTH_USER = "your_sendgrid_username";
        private static final String SMTP_AUTH_PWD = "your_sendgrid_password";

        public static void main(String[] args) throws Exception{
            new MyEmailer().SendMail();
        }

        public void SendMail() throws Exception
        {
            Properties properties = new Properties();
                properties.put("mail.transport.protocol", "smtp");
                properties.put("mail.smtp.host", SMTP_HOST_NAME);
                properties.put("mail.smtp.port", 587);
                properties.put("mail.smtp.auth", "true");
                // …
    ```

1. Développez la classe *javax.mail.Authenticator*, puis, dans votre implémentation de la méthode *getPasswordAuthentication*, renvoyez votre nom d'utilisateur et votre mot de passe SendGrid.  

    ```java
    private class SMTPAuthenticator extends javax.mail.Authenticator {
    public PasswordAuthentication getPasswordAuthentication() {
        String username = SMTP_AUTH_USER;
        String password = SMTP_AUTH_PWD;
        return new PasswordAuthentication(username, password);
    }
    ```
2. Créez une session de messagerie électronique authentifiée via un objet *javax.mail.Session* .  

    ```java
    Authenticator auth = new SMTPAuthenticator();
    Session mailSession = Session.getDefaultInstance(properties, auth);
    ```
3. Créez votre message et spécifiez des valeurs pour **À**, **De**, **Objet**. Cela est illustré dans la section [Comment créer un message électronique](#how-to-create-an-email).
4. Envoyez le message via un objet *javax.mail.Transport* . Cela est illustré dans la section [Comment envoyer un message électronique][#comment-envoyer-un-message-électronique].

## <a name="how-to-create-an-email"></a>Procédure : créer un message électronique
Le code suivant permet d'entrer des valeurs pour un message électronique.

```java
MimeMessage message = new MimeMessage(mailSession);
Multipart multipart = new MimeMultipart("alternative");
BodyPart part1 = new MimeBodyPart();
part1.setText("Hello, Your Contoso order has shipped. Thank you, John");
BodyPart part2 = new MimeBodyPart();
part2.setContent(
    "<p>Hello,</p>
    <p>Your Contoso order has <b>shipped</b>.</p>
    <p>Thank you,<br>John</br></p>",
    "text/html");
multipart.addBodyPart(part1);
multipart.addBodyPart(part2);
message.setFrom(new InternetAddress("john@contoso.com"));
message.addRecipient(Message.RecipientType.TO,
    new InternetAddress("someone@example.com"));
message.setSubject("Your recent order");
message.setContent(multipart);
```

## <a name="how-to-send-an-email"></a>Procédure : Envoyer un e-mail
Le code suivant permet d'envoyer un message électronique.

```java
Transport transport = mailSession.getTransport();
// Connect the transport object.
transport.connect();
// Send the message.
transport.sendMessage(message, message.getAllRecipients());
// Close the connection.
transport.close();
```

## <a name="how-to-add-an-attachment"></a>Procédure : ajouter une pièce jointe
Le code suivant permet d'ajouter une pièce jointe.

```java
// Local file name and path.
String attachmentName = "myfile.zip";
String attachmentPath = "c:\\myfiles\\";
MimeBodyPart attachmentPart = new MimeBodyPart();
// Specify the local file to attach.
DataSource source = new FileDataSource(attachmentPath + attachmentName);
attachmentPart.setDataHandler(new DataHandler(source));
// This example uses the local file name as the attachment name.
// They could be different if you prefer.
attachmentPart.setFileName(attachmentName);
multipart.addBodyPart(attachmentPart);
```

## <a name="how-to-use-filters-to-enable-footers-tracking-and-analytics"></a>Procédure : utiliser des filtres pour activer les pieds de page, le suivi et l’analyse
SendGrid offre des fonctionnalités de messagerie électronique supplémentaires grâce à l'utilisation des *filtres*. Il s'agit de paramètres que vous pouvez ajouter à un message électronique pour activer des fonctionnalités spécifiques telles que le suivi des clics, Google Analytics, le suivi d'abonnement, etc. Pour obtenir une liste exhaustive des filtres, consultez la page [Paramètres de filtre][Filter Settings].

* Le code suivant permet d'insérer un filtre de pied de page entraînant l'affichage de texte HTML en bas du message électronique à envoyer.

    ```java
    message.addHeader("X-SMTPAPI",
        "{\"filters\":
        {\"footer\":
        {\"settings\":
        {\"enable\":1,\"text/html\":
        \"<html><b>Thank you</b> for your business.</html>\"}}}}");
    ```
* Parmi les exemples de filtres, on peut également citer le suivi des clics. Imaginons que le texte de votre courrier électronique contient un lien hypertexte comme celui qui suit, et que vous voulez suivre le taux de clics :

    ```java
    messagePart.setContent(
        "Hello,
        <p>This is the body of the message. Visit
        <a href='http://www.contoso.com'>http://www.contoso.com</a>.</p>
        Thank you.",
        "text/html");
    ```
* Pour activer le suivi des clics, utilisez le code suivant :

    ```java
    message.addHeader("X-SMTPAPI",
        "{\"filters\":
        {\"clicktrack\":
        {\"settings\":
        {\"enable\":1}}}}");
    ```

## <a name="how-to-update-email-properties"></a>Procédure : mettre à jour les propriétés du courrier électronique
Vous pouvez remplacer certaines propriétés d’e-mail à l’aide de la **propriété set** ou en ajouter avec la **propriété add**.

Par exemple, pour indiquer des adresses **ReplyTo** , utilisez le code suivant :

```java
InternetAddress addresses[] =
    { new InternetAddress("john@contoso.com"),
        new InternetAddress("wendy@contoso.com") };

message.setReplyTo(addresses);
```

Pour ajouter un destinataire **Cc** , utilisez le code suivant :

```java
message.addRecipient(Message.RecipientType.CC, new
InternetAddress("john@contoso.com"));
```

## <a name="how-to-use-additional-sendgrid-services"></a>Procédure : utiliser des services SendGrid supplémentaires
SendGrid propose des API web qui peuvent vous aider à tirer parti de fonctionnalités SendGrid supplémentaires à partir de votre application Azure. Pour plus d'informations, consultez la [documentation de l'API SendGrid][SendGrid API documentation].

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris les bases du service de messagerie SendGrid, consultez ces liens pour en savoir plus.

* Exemple illustrant l’utilisation de SendGrid dans un déploiement Azure : [Guide pratique pour envoyer du courrier électronique en utilisant SendGrid à partir de Java dans un déploiement Azure](store-sendgrid-java-how-to-send-email-example.md)
* SDK Java SendGrid : <https://sendgrid.com/docs/Code_Examples/java.html>
* Documentation de l’API SendGrid : <https://sendgrid.com/docs/API_Reference/index.html>
* Offre spéciale SendGrid pour les clients Azure : <https://sendgrid.com/windowsazure.html>

[https://sendgrid.com]: https://sendgrid.com
[https://sendgrid.com/pricing.html]: https://sendgrid.com/pricing.html
[http://www.sendgrid.com/azure.html]: https://www.sendgrid.com/windowsazure.html
[https://sendgrid.com/features]: https://sendgrid.com/features
[https://www.oracle.com/technetwork/java/javamail]: https://www.oracle.com/technetwork/java/javamail/index.html
[Filter Settings]: https://sendgrid.com/docs/API_Reference/Web_API/filter_settings.html
[SendGrid API documentation]: https://sendgrid.com/docs/API_Reference/index.html
[https://sendgrid.com/azure.html]: https://sendgrid.com/windowsazure.html
[service de messagerie dans le cloud]: https://sendgrid.com/email-solutions
[remise de courrier électronique transactionnelle]: https://sendgrid.com/transactional-email
