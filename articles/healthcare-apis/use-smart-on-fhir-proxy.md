---
title: Proxy SMART on FHIR Azure Active Directory
description: Ce tutoriel explique comment utiliser un proxy pour activer les applications SMART on FHIR avec l’API Azure pour FHIR.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 04/02/2019
ms.openlocfilehash: 2e13a9fc32964781dda07e5534e5cab79868ddf0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995537"
---
# <a name="tutorial-azure-active-directory-smart-on-fhir-proxy"></a>Tutoriel : Proxy SMART on FHIR Azure Active Directory

[SMART on FHIR](https://docs.smarthealthit.org/) est un ensemble de spécifications ouvertes qui visent à intégrer des applications partenaires aux serveurs FHIR et aux systèmes de dossiers médicaux électroniques dotés d’interfaces FHIR. L’un des principaux objectifs des spécifications est de décrire la façon dont une application doit détecter les points de terminaison d’authentification pour un serveur FHIR et de démarrer une séquence d’authentification. 

L’authentification est basée sur OAuth2. Mais comme SMART on FHIR utilise des conventions de nommage des paramètres qui ne sont pas immédiatement compatibles avec Azure Active Directory (Azure AD), l’API Azure pour FHIR dispose d’un proxy SMART on FHIR Azure AD intégré qui autorise un sous-ensemble des séquences de lancement SMART on FHIR. Plus précisément, le proxy autorise la [séquence de lancement des dossiers médicaux électroniques](https://hl7.org/fhir/smart-app-launch/#ehr-launch-sequence).

Ce tutoriel explique comment utiliser le proxy pour activer les applications SMART on FHIR avec l’API Azure pour FHIR.

## <a name="prerequisites"></a>Prérequis

- Une instance de l’API Azure pour FHIR
- [.NET Core 2.2](https://dotnet.microsoft.com/download/dotnet-core/2.2)

## <a name="configure-azure-ad-registrations"></a>Configurer des inscriptions Azure AD

SMART on FHIR exige que `Audience` possède un URI d’identificateur égal à l’URI du service FHIR. La configuration standard de l’API Azure pour FHIR utilise la valeur `https://azurehealthcareapis.com` pour `Audience`. Cependant, vous pouvez aussi définir une valeur correspondant à l’URL spécifique de votre service FHIR (par exemple `https://MYFHIRAPI.azurehealthcareapis.com`). Cela est nécessaire quand le proxy SMART on FHIR est utilisé.

Vous avez aussi besoin d’une inscription d’application cliente. La plupart des applications SMART on FHIR sont des applications JavaScript monopages. Vous devez donc suivre les instructions de configuration d’une [application cliente Azure AD publique](register-public-azure-ad-client-app.md).

Une fois ces étapes terminées, vous devez disposer des éléments suivants :

- Un serveur FHIR avec l’audience RGE définie sur `https://MYFHIRAPI.azurehealthcareapis.com`, où `MYFHIRAPI` est le nom de votre instance Azure API pour FHIR.
- L’inscription d’une application cliente publique. Notez l’ID d’application de cette application cliente.

## <a name="enable-the-smart-on-fhir-proxy"></a>Activer le proxy SMART on FHIR

Activez le proxy SMART on FHIR dans les paramètres d’**authentification** de votre instance API Azure pour FHIR en cochant la case **SMART on FHIR proxy** (Proxy SMART on FHIR) :

![Sélections pour activer le proxy SMART on FHIR](media/tutorial-smart-on-fhir/enable-smart-on-fhir-proxy.png)

## <a name="enable-cors"></a>Activer CORS

Étant donné que la plupart des applications SMART on FHIR sont des applications JavaScript monopages, vous devez [activer le partage des ressources cross-origin (CORS)](configure-cross-origin-resource-sharing.md) pour l’API Azure pour FHIR :

![Sélections pour activer CORS](media/tutorial-smart-on-fhir/enable-cors.png)

## <a name="configure-the-reply-url"></a>Configurer l’URL de réponse

Le proxy SMART on FHIR joue le rôle d’intermédiaire entre l’application SMART on FHIR et Azure AD. La réponse d’authentification (code d’authentification) doit accéder au proxy SMART on FHIR à la place de l’application proprement dite. Le proxy transmet ensuite la réponse à l’application. 

Du fait de ce relais en deux étapes du code d’authentification, vous devez définir l’URL de réponse (rappel) de votre application cliente Azure AD sur une URL qui combine l’URL de réponse du proxy SMART on FHIR et l’URL de réponse de l’application SMART on FHIR. L’URL de réponse combinée prend la forme suivante :

```http
https://MYFHIRAPI.azurehealthcareapis.com/AadSmartOnFhirProxy/callback/aHR0cHM6Ly9sb2NhbGhvc3Q6NTAwMS9zYW1wbGVhcHAvaW5kZXguaHRtbA
```

Dans cette réponse, `aHR0cHM6Ly9sb2NhbGhvc3Q6NTAwMS9zYW1wbGVhcHAvaW5kZXguaHRtbA` est une version sécurisée, encodée en base64 de l’URL de réponse de l’application SMART on FHIR. Pour le lanceur d’application SMART on FHIR, quand l’application s’exécute localement, l’URL de réponse est `https://localhost:5001/sampleapp/index.html`. 

Vous pouvez générer l’URL de réponse combinée en utilisant un script de ce type :

```PowerShell
$replyUrl = "https://localhost:5001/sampleapp/index.html"
$fhirServerUrl = "https://MYFHIRAPI.azurewebsites.net"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($ReplyUrl)
$encodedText = [Convert]::ToBase64String($bytes)
$encodedText = $encodedText.TrimEnd('=');
$encodedText = $encodedText.Replace('/','_');
$encodedText = $encodedText.Replace('+','-');

$newReplyUrl = $FhirServerUrl.TrimEnd('/') + "/AadSmartOnFhirProxy/callback/" + $encodedText
```

Ajoutez l’URL de réponse à l’application cliente publique que vous avez créée précédemment pour Azure AD :

![URL de réponse configurée pour le client public](media/tutorial-smart-on-fhir/configure-reply-url.png)

## <a name="get-a-test-patient"></a>Obtenir un patient de test

Pour tester l’API Azure pour FHIR et le proxy SMART on FHIR, la base de données doit contenir au moins un patient. Si vous n’avez pas encore interagi avec l’API et que vous n’avez pas de données dans la base de données, suivez le [tutoriel API FHIR Postman](access-fhir-postman-tutorial.md) pour charger un patient. Notez l’ID d’un patient spécifique.

## <a name="download-the-smart-on-fhir-app-launcher"></a>Télécharger le lanceur d’application SMART on FHIR

Le [dépôt FHIR Server pour Azure](https://github.com/Microsoft/fhir-server) open source comporte un lanceur d’application SMART on FHIR simple et un exemple d’application SMART on FHIR. Dans ce tutoriel, utilisez ce lanceur SMART on FHIR localement pour tester la configuration.

Vous pouvez cloner le dépôt GitHub et accéder à l’application à l’aide de ces commandes :

```PowerShell
git clone https://github.com/Microsoft/fhir-server
cd fhir-server/samples/apps/SmartLauncher
```

L’application a besoin de quelques paramètres de configuration, que vous pouvez définir dans `appsettings.json` :

```json
{
    "FhirServerUrl": "https://MYFHIRAPI.azurehealthcareapis.com",
    "ClientId": "APP-ID",
    "DefaultSmartAppUrl": "/sampleapp/launch.html"
}
```

Nous vous recommandons d’utiliser la fonctionnalité `dotnet user-secrets` :

```PowerShell
dotnet user-secrets set FhirServerUrl https://MYFHIRAPI.azurehealthcareapis.com
dotnet user-secrets set ClientId <APP-ID>
```

Utilisez cette commande pour exécuter l’application :

```PowerShell
dotnet run
```

## <a name="test-the-smart-on-fhir-proxy"></a>Tester le proxy SMART on FHIR

Après avoir démarré le lanceur d’application SMART on FHIR, vous pouvez faire pointer votre navigateur vers `https://localhost:5001`, où l’écran suivant doit s’afficher :

![Lanceur d’application SMART on FHIR](media/tutorial-smart-on-fhir/smart-on-fhir-app-launcher.png)

Quand vous renseignez les champs **Patient** (Patient), **Encounter** (Consultation) ou **Practitioner** (Médecin), vous constatez que le contenu de **Launch context** (Contexte de lancement) est mis à jour. Quand vous utilisez l’API Azure pour FHIR, le contexte de lancement est simplement un document JSON qui contient des informations notamment sur le patient et le médecin. Ce contexte de lancement est encodé en base64 et transmis à l’application SMART on FHIR en tant que paramètre de requête `launch`. Conformément à la spécification SMART on FHIR, cette variable est inintelligible pour l’application SMART on FHIR et est transmise au fournisseur d’identité. 

Le proxy SMART on FHIR se sert de ces informations pour compléter les champs de la réponse de jeton. L’application SMART on FHIR *peut* utiliser ces champs pour contrôler sur quel patient porte la demande de données et sur la façon dont l’application s’affiche à l’écran. Le proxy SMART on FHIR prend en charge les champs suivants :

* `patient`
* `encounter`
* `practitioner`
* `need_patient_banner`
* `smart_style_url`

Ces champs visent à fournir des indications à l’application, mais ils ne véhiculent pas d’information de sécurité. Une application SMART on FHIR peut les ignorer.

Notez que le lanceur d’application SMART on FHIR met à jour l’information **Launch URL** (URL de lancement) au bas de la page. Sélectionnez **Launch** (Lancer) pour démarrer l’exemple d’application ; vous obtenez alors quelque chose qui ressemble à cet exemple :

![Application SMART on FHIR](media/tutorial-smart-on-fhir/smart-on-fhir-app.png)

Inspectez la réponse du jeton pour voir comment les champs de contexte de lancement sont transmis à l’application.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré le proxy SMART on FHIR Azure Active Directory. Pour explorer l’utilisation des applications SMART on FHIR avec l’API Azure pour FHIR et le serveur FHIR pour Azure open source, accédez au dépôt d’exemples de serveur FHIR sur GitHub :

>[!div class="nextstepaction"]
>[Exemples de serveur FHIR](https://github.com/Microsoft/fhir-server-samples)
