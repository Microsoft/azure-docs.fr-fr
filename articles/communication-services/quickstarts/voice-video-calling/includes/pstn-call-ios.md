---
author: nikuklic
ms.service: azure-communication-services
ms.topic: include
ms.date: 9/11/2020
ms.author: nikuklic
ms.openlocfilehash: 63fd0af819fde7d78df289a1b8f5cefa2e415101
ms.sourcegitcommit: d9ba60f15aa6eafc3c5ae8d592bacaf21d97a871
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/06/2020
ms.locfileid: "91779667"
---
[!INCLUDE [Emergency Calling Notice](../../../includes/emergency-calling-notice-include.md)]
## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Une ressource Communication Services déployée. [Créer une ressource Communication Services](../../create-communication-resource.md)
- Numéro de téléphone acquis dans la ressource Communication Services. [Comment obtenir un numéro de téléphone](../../telephony-sms/get-phone-number.md).
- Un `User Access Token` pour activer le client d’appel. Pour en savoir plus, consultez [Comment obtenir un `User Access Token`](../../access-tokens.md)
- Suivez le guide de démarrage rapide pour [prendre en main l’ajout des appels à votre application](../getting-started-with-calling.md)

### <a name="prerequisite-check"></a>Vérification du prérequis

- Pour voir les numéros de téléphone associés à votre ressource Communication Services, connectez-vous au [portail Azure](https://portal.azure.com/), localisez votre ressource Communication Services, puis ouvrez l’onglet **Phone Numbers** (Numéros de téléphone) dans le volet de navigation gauche.
- Vous pouvez créer et exécuter votre application avec la bibliothèque de client Azure Communication Services Calling pour iOS :

## <a name="setting-up"></a>Configuration

## <a name="start-a-call-to-phone"></a>Démarrer un appel téléphonique

Spécifiez le numéro de téléphone que vous avez acquis dans la ressource Communication Services, qui sera utilisé pour démarrer l’appel :
> [!WARNING]
> Notez que les numéros de téléphone doivent être fournis au format standard international E.164. (par exemple +12223334444)

Modifiez le gestionnaire d’événements `startCall` qui sera exécuté quand l’utilisateur clique sur le bouton *Démarrer l’appel* :

```swift
func startCall() {
    // Ask permissions
    AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
        if granted {
            let startCallOptions = ACSStartCallOptions()
            startCallOptions!.alternateCallerID = PhoneNumber(phoneNumber: "+12223334444")
            self.call = self.callAgent!.call([PhoneNumber(phoneNumber: self.callee)], options: startCallOptions)
            self.callDelegate = CallDelegate(self)
            self.call!.delegate = self.callDelegate
        }
    }
}
```

## <a name="run-the-code"></a>Exécuter le code

Vous pouvez générer et exécuter votre application sur un simulateur iOS en sélectionnant **Product** > **Run** ou en utilisant le raccourci clavier (&#8984;-R).

![Apparence finale de l’application du guide de démarrage rapide](../media/ios/quick-start-make-call.png)

Vous pouvez passer un appel téléphonique en fournissant un numéro de téléphone dans le champ texte ajouté et en cliquant sur le bouton **Démarrer l’appel**.
> [!WARNING]
> Notez que les numéros de téléphone doivent être fournis au format standard international E.164. (par exemple +12223334444)

> [!NOTE]
> La première fois que vous effectuez un appel, le système vous invite à accorder l’accès au microphone. Dans une application de production, vous devez utiliser l’API `AVAudioSession` [vérifier l’état de l’autorisation](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/requesting_access_to_protected_resources) et mettre à jour le comportement de votre application de manière appropriée lorsque l’autorisation n’est pas accordée.
