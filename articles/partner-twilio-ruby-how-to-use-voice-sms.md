---
title: Utilisation de Twilio pour les fonctionnalités vocales et de SMS (Ruby) | Microsoft Docs
description: Découvrez comment passer un appel téléphonique et envoyer un SMS avec le service d'API Twilio sur Azure. Exemples de code écrits en Ruby.
services: ''
documentationcenter: ruby
author: georgewallace
ms.assetid: 60e512f6-fa47-47c0-aedc-f19bb72a1158
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: ruby
ms.topic: article
ms.date: 11/25/2014
ms.author: gwallace
ms.openlocfilehash: 49203195bf7746d0bff1b9543d1641f69ab23359
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95542675"
---
# <a name="how-to-use-twilio-for-voice-and-sms-capabilities-in-ruby"></a>Utilisation de Twilio pour les fonctionnalités vocales et de SMS dans Ruby
Ce guide présente l'exécution de tâches de programmation courantes avec le service API Twilio sur Azure. Les scénarios abordés comprennent notamment les appels téléphoniques et l'envoi de SMS. Pour plus d'informations sur Twilio et sur l'utilisation des fonctionnalités vocales et de SMS de vos applications, consultez la section [Étapes suivantes](#NextSteps) .

## <a name="what-is-twilio"></a><a id="WhatIs"></a>Présentation de Twilio
Twilio est une API de service Web de téléphonie vous permettant d'utiliser vos aptitudes et langages Web existants pour générer des applications vocales et de SMS. Twilio est un service tiers, et non une fonctionnalité Azure ni un produit Microsoft.

**Twilio Voice** permet à vos applications de passer et de recevoir des appels téléphoniques. **Twilio SMS** permet à vos applications de créer et de recevoir des SMS. **Twilio Client** permet à vos applications d'activer les communications vocales au moyen de connexions Internet existantes, y compris des connexions mobiles.

## <a name="twilio-pricing-and-special-offers"></a><a id="Pricing"></a>Tarification de Twilio et offres spéciales
Des informations sur les prix de Twilio sont disponibles sur la page [Tarification de Twilio][twilio_pricing]. Les clients Azure ont accès à une [offre spéciale][special_offer] : un crédit gratuit de 1 000 SMS ou de 1 000 minutes d'appels entrants. Pour bénéficier de cette offre ou pour obtenir des informations supplémentaires, visitez la page [https://ahoy.twilio.com/azure][special_offer].  

## <a name="concepts"></a><a id="Concepts"></a>Concepts
L'API Twilio est une API RESTful qui offre des fonctionnalités vocales et de SMS aux applications. Les bibliothèques clientes sont disponibles dans plusieurs langages. Pour obtenir la liste, consultez la page [Bibliothèques de l'API Twilio][twilio_libraries].

### <a name="twiml"></a><a id="TwiML"></a>TwiML
TwiML est un jeu d'instructions XML qui informent Twilio sur la façon de traiter un appel ou un SMS.

À titre d'exemple, le code TwiML suivant convertit le texte **Hello World** en parole.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<Response>
    <Say>Hello World</Say>
</Response>
```

Tous les documents TwiML ont un élément racine `<Response>` . Vous pouvez donc utiliser des verbes Twilio pour définir le comportement de votre application.

### <a name="twiml-verbs"></a><a id="Verbs"></a>Verbes TwiML
Les verbes Twilio sont des balises XML qui indiquent à Twilio les **actions** à effectuer. Par exemple, le verbe **&lt;Say&gt;** indique à Twilio de délivrer un message audible lors d'un appel. 

La liste suivante présente les verbes Twilio.

* **&lt;Composer&gt;**  : connecte l’appelant à un autre téléphone.
* **&lt;Rassembler&gt;**  : collecte les chiffres numériques entrés sur le clavier du téléphone.
* **&lt;Raccrocher&gt;**  : met fin à un appel.
* **&lt;Lire&gt;**  : lit un fichier audio.
* **&lt;Suspendre&gt;**  : patiente silencieusement pendant un nombre déterminé de secondes.
* **&lt;Enregistrer&gt;**  : enregistre la voix de l’appelant et retourne l’URL d’un fichier contenant l’enregistrement.
* **&lt;Rediriger&gt;**  : transfère le contrôle d’un appel ou d’un SMS au TwiML à une autre URL.
* **&lt;Rejeter&gt;**  : refuse un appel entrant sur votre numéro Twilio sans vous facturer
* **&lt;Dire&gt;**  : convertit le texte d’un appel par synthèse vocale.
* **&lt;Sms&gt;**  : envoie un SMS.

Pour plus d'informations sur les verbes Twilio, leurs attributs et le langage TwiML, consultez la page [TwiML][twiml]. Pour plus d'informations sur l'API Twilio, consultez la page [API Twilio][twilio_api].

## <a name="create-a-twilio-account"></a><a id="CreateAccount"></a>Créer un compte Twilio
Lorsque vous êtes prêt à créer votre compte Twilio, inscrivez-vous sur la page [Essayer Twilio][try_twilio]. Vous pouvez commencer avec un compte gratuit, avant de le mettre à niveau ultérieurement.

Lors de la création d'un compte Twilio, vous obtenez un numéro de téléphone gratuit pour votre application. Vous recevez également un SID de compte et un jeton d'authentification. Les deux sont nécessaires pour passer des appels d'API Twilio. Pour éviter qu'une personne non autorisée n'accède à votre compte, conservez votre jeton d'authentification en lieu sûr. Vous pouvez consulter vos ID de compte et jeton d’authentification sur la [page du compte Twilio][twilio_account], dans les champs intitulés **ACCOUNT SID** et **AUTH TOKEN**, respectivement.

### <a name="verify-phone-numbers"></a><a id="VerifyPhoneNumbers"></a>Vérifier les numéros de téléphone
Outre le numéro donné par Twilio, vous pouvez également vérifier les numéros que vous contrôlez (votre numéro de portable ou de téléphone fixe) pour les utiliser dans vos applications. 

Pour en savoir plus sur la vérification d’un numéro de téléphone, consultez la page [Gestion des numéros][verify_phone].

## <a name="create-a-ruby-application"></a><a id="create_app"></a>Créer une application Ruby
Une application Ruby qui utilise le service Twilio et qui s'exécute dans Azure est identique aux autres applications Ruby qui utilisent le service Twilio. Bien que les services Twilio soient basés sur RESTful et puissent être appelés de différentes manières depuis Ruby, cet article met l’accent sur l’utilisation des services Twilio avec la [bibliothèque d’aide Twilio pour Ruby][twilio_ruby].

Tout d’abord, [configurez une nouvelle machine virtuelle Linux Azure][azure_vm_setup] afin qu’elle joue le rôle d’hôte pour votre nouvelle application web Ruby. Ignorez les étapes concernant la création de l'application Rails. Il vous suffit de configurer la machine virtuelle. Veillez à créer un point de terminaison avec un port externe défini sur 80 et un port interne défini sur 5000.

Dans les exemples ci-dessous, nous utilisons [Sinatra][sinatra], une infrastructure web très simple pour Ruby. Vous pouvez certainement utiliser la bibliothèque d'aide Twilio pour Ruby avec une autre infrastructure Web, dont Ruby on Rails.

Utilisez SSH dans votre nouvelle machine virtuelle et créez un répertoire pour votre nouvelle application. Dans ce répertoire, créez un fichier nommé Gemfile et copiez-y le code suivant :

```bash
source 'https://rubygems.org'
gem 'sinatra'
gem 'thin'
```

Sur la ligne de commande, exécutez `bundle install`. Ceci permet d'installer les dépendances ci-dessus. Ensuite, créez un fichier appelé `web.rb`. C'est là que va résider le code de votre application Web. Collez-y le code suivant :

```ruby
require 'sinatra'

get '/' do
    "Hello Monkey!"
end
```

À ce stade, vous devez pouvoir exécuter la commande `ruby web.rb -p 5000`. Ceci met en place un petit serveur web sur le port 5000. Vous devez pouvoir accéder à cette application dans votre navigateur en vous rendant à l'URL définie pour la création de votre machine virtuelle Azure. Lorsque vous accédez à votre application Web dans le navigateur, vous êtes prêt à créer une application Twilio.

## <a name="configure-your-application-to-use-twilio"></a><a id="configure_app"></a>Configurer l’application pour utiliser Twilio
Vous pouvez configurer votre application web afin qu’elle utilise la bibliothèque Twilio en mettant à jour votre fichier `Gemfile` pour y inclure la ligne suivante :

```bash
gem 'twilio-ruby'
```

Sur la ligne de commande, exécutez `bundle install`. Ensuite, ouvrez `web.rb` et ajoutez cette ligne au début du code :

```ruby
require 'twilio-ruby'
```

Vous pouvez maintenant utiliser la bibliothèque d'aide Twilio dans votre application web.

## <a name="how-to-make-an-outgoing-call"></a><a id="howto_make_call"></a>Procédure : passer un appel sortant
Le code suivant permet d'exécuter un appel sortant. Les concepts principaux incluent l'utilisation de la bibliothèque d'aide Twilio pour Ruby afin de passer des appels d'API REST et d'assurer le rendu TwiML. Remplacez vos valeurs pour les numéros de téléphone **From** (De) et **To** (À), puis assurez-vous de vérifier le numéro de téléphone **From** de votre compte Twilio avant d'exécuter le code.

Ajoutez cette fonction à `web.md`:

```ruby
# Set your account ID and authentication token.
sid = "your_twilio_account_sid";
token = "your_twilio_authentication_token";

# The number of the phone initiating the call.
# This should either be a Twilio number or a number that you've verified
from = "NNNNNNNNNNN";

# The number of the phone receiving call.
to = "NNNNNNNNNNN";

# Use the Twilio-provided site for the TwiML response.
url = "http://yourdomain.cloudapp.net/voice_url";

get '/make_call' do
    # Create the call client.
    client = Twilio::REST::Client.new(sid, token);

    # Make the call
    client.account.calls.create(to: to, from: from, url: url)
end

post '/voice_url' do
    "<Response>
        <Say>Hello Monkey!</Say>
    </Response>"
end
```

Si vous ouvrez l’adresse `http://yourdomain.cloudapp.net/make_call` dans un navigateur, cela déclenche l’appel de l’API Twilio qui passe alors l’appel téléphonique. Les deux premiers paramètres dans `client.account.calls.create` sont relativement explicites : le numéro d’où provient l’appel (`from`) et le numéro appelé (`to`). 

Le troisième paramètre (`url`) est l’URL demandée par Twilio pour obtenir les instructions à suivre une fois l’appel connecté. Dans notre exemple, nous avons défini une URL (`http://yourdomain.cloudapp.net`) qui renvoie un document TwiML simple utilisant le verbe `<Say>` pour convertir du texte en parole et dire « Hello Monkey » à la personne qui reçoit l’appel.

## <a name="how-to-receive-an-sms-message"></a><a id="howto_receive_sms"></a>Procédure : Recevoir un SMS
Dans l'exemple précédent, nous avons émis un appel **sortant** . Cette fois, nous allons utiliser le numéro de téléphone fourni par Twilio lors de l'inscription pour traiter un SMS **entrant** .

Commencez par vous connecter à votre [tableau de bord Twilio][twilio_account]. Cliquez sur « Numbers » dans la zone de navigation supérieure, puis sur le numéro fourni par Twilio. Deux URL sont disponibles pour configuration : une URL de requête vocale et une URL de requête SMS. Il s'agit des URL que Twilio appelle lors de toute émission d'appel ou toute réception de SMS. On parle de « raccordements Web » pour désigner ces URL.

Nous voulons traiter les messages SMS entrants ; nous allons donc mettre à jour l’URL avec l’adresse `http://yourdomain.cloudapp.net/sms_url`. Cliquez sur Save Changes en bas de la page pour poursuivre. Revenons maintenant dans `web.rb` pour programmer notre application afin qu’elle exécute l’action suivante :

```ruby
post '/sms_url' do
    "<Response>
        <Message>Hey, thanks for the ping! Twilio and Azure rock!</Message>
    </Response>"
end
```

Veillez à redémarrer l'application Web après avoir effectué cette modification. Maintenant, à l'aide de votre téléphone, envoyez un SMS à votre numéro Twilio. Vous devriez rapidement recevoir une réponse par SMS avec le message « Hey, merci pour le ping ! Twilio et Azure sont les meilleurs ! »

## <a name="how-to-use-additional-twilio-services"></a><a id="additional_services"></a>Procédure : Utiliser des services Twilio supplémentaires
En plus des exemples présentés ici, Twilio offre des API Web que vous pouvez utiliser pour tirer profit d'autres fonctionnalités de Twilio à partir de votre application Azure. Pour plus d'informations, consultez la [documentation de l'API Twilio][twilio_api_documentation].

### <a name="next-steps"></a><a id="NextSteps"></a>Étapes suivantes
Maintenant que vous avez appris les bases du service Twilio, consultez ces liens pour en savoir plus :

* [Conseils de sécurité Twilio][twilio_security_guidelines]
* [Procédures et exemples de code Twilio][twilio_howtos]
* [Didacticiels de démarrage rapide Twilio][twilio_quickstarts] 
* [Twilio sur GitHub][twilio_on_github]
* [Contacter le support Twilio][twilio_support]

[twilio_ruby]: https://www.twilio.com/docs/ruby/install





[twilio_pricing]: https://www.twilio.com/pricing
[special_offer]: https://ahoy.twilio.com/azure
[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: https://www.twilio.com/docs/api/twiml
[twilio_api]: https://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]:  https://www.twilio.com/user/account
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
[twilio_api_documentation]: https://www.twilio.com/api
[twilio_security_guidelines]: https://www.twilio.com/docs/security
[twilio_howtos]: https://www.twilio.com/docs/howto
[twilio_on_github]: https://github.com/twilio
[twilio_support]: https://www.twilio.com/help/contact
[twilio_quickstarts]: https://www.twilio.com/docs/quickstart
[sinatra]: http://www.sinatrarb.com/
[azure_vm_setup]: /previous-versions/azure/virtual-machines/linux/classic/ruby-rails-web-app