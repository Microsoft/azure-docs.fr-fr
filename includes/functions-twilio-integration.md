---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/04/2018
ms.author: glenga
ms.openlocfilehash: b4a1891eadf2e36bcb94b9f605d91f227fa83a3f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96027556"
---
Cet exemple implique l’utilisation du service [Twilio](https://www.twilio.com/) pour envoyer des SMS à un téléphone mobile. Azure Functions prend déjà ce service en charge, via la [liaison Twilio](../articles/azure-functions/functions-bindings-twilio.md). L’exemple utilise cette fonctionnalité.

Pour commencer, vous devez disposer d’un compte Twilio. Vous pouvez en créer un gratuitement à l’adresse https://www.twilio.com/try-twilio. Une fois ce compte créé, ajoutez les trois **paramètres d’application** suivants à votre application de fonction.

| Nom du paramètre d’application | Description de la valeur |
| - | - |
| **TwilioAccountSid**  | Il s’agit du SID de votre compte Twilio |
| **TwilioAuthToken**   | Il s’agit du jeton d’authentification de votre compte Twilio |
| **TwilioPhoneNumber** | Numéro de téléphone associé à votre compte Twilio. Il est utilisé pour envoyer des SMS. |