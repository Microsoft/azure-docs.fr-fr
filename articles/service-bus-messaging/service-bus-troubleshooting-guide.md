---
title: Guide de dépannage pour Azure Service Bus | Microsoft Docs
description: Découvrez des conseils et recommandations pour résoudre certains problèmes que vous pourriez rencontrer lors de l’utilisation d’Azure Service Bus.
ms.topic: article
ms.date: 09/16/2020
ms.openlocfilehash: aab7fa53b4af309c68cd91fdb1d25c5771f89828
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91261122"
---
# <a name="troubleshooting-guide-for-azure-service-bus"></a>Guide de dépannage pour Azure Service Bus
Cet article fournit des conseils et des recommandations pour résoudre certains problèmes que vous pourriez rencontrer lors de l’utilisation d’Azure Service Bus. 

## <a name="connectivity-certificate-or-timeout-issues"></a>Problèmes de connectivité, de certificat ou de délai d’expiration
Aidez-vous des étapes suivantes pour résoudre les problèmes de connectivité, de certificat ou de délai d’expiration pour tous les services sous *.servicebus.windows.net. 

- Accédez à `https://<yournamespace>.servicebus.windows.net/` ou utilisez [wget](https://www.gnu.org/software/wget/). Cet outil facilite les vérifications quand vous rencontrez des problèmes avec le filtrage des adresses IP, le réseau virtuel ou les chaînes de certificats, qui sont courants lors de l’utilisation du Kit de développement logiciel (SDK) Java.

    Voici un exemple de message de réussite :
    
    ```xml
    <feed xmlns="http://www.w3.org/2005/Atom"><title type="text">Publicly Listed Services</title><subtitle type="text">This is the list of publicly-listed services currently available.</subtitle><id>uuid:27fcd1e2-3a99-44b1-8f1e-3e92b52f0171;id=30</id><updated>2019-12-27T13:11:47Z</updated><generator>Service Bus 1.1</generator></feed>
    ```
    
    Voici un exemple de message d’échec :

    ```json
    <Error>
        <Code>400</Code>
        <Detail>
            Bad Request. To know more visit https://aka.ms/sbResourceMgrExceptions. . TrackingId:b786d4d1-cbaf-47a8-a3d1-be689cda2a98_G22, SystemTracker:NoSystemTracker, Timestamp:2019-12-27T13:12:40
        </Detail>
    </Error>
    ```
- Exécutez la commande suivante pour vérifier si un port est bloqué sur le pare-feu. Les ports utilisés sont les suivants : 443 (HTTPS), 5671 (AMQP) et 9354 (Net Messaging/SBMP). Selon la bibliothèque que vous utilisez, d’autres ports peuvent également être utilisés. Voici l’exemple de commande qui vérifie si le port 5671 est bloqué. 

    ```powershell
    tnc <yournamespacename>.servicebus.windows.net -port 5671
    ```

    Sur Linux :

    ```shell
    telnet <yournamespacename>.servicebus.windows.net 5671
    ```
- Si vous constatez des problèmes de connectivité intermittents, exécutez la commande suivante pour détecter les paquets supprimés. Cette commande essaiera d’établir 25 connexions TCP différentes toutes les 1 seconde avec le service. Ensuite, vous pouvez vérifier le nombre d’entre elles ayant réussi ou échoué, ainsi que la latence de connexion TCP. Vous pouvez télécharger l’outil `psping` à partir d’[ici](/sysinternals/downloads/psping).

    ```shell
    .\psping.exe -n 25 -i 1 -q <yournamespace>.servicebus.windows.net:5671 -nobanner     
    ```
    Vous pouvez utiliser des commandes équivalentes dans d’autres outils, par exemple `tnc`, `ping`, etc. 
- Si les étapes précédentes n’ont pas résolu le problème, obtenez une trace réseau et analysez-la à l’aide d’un outil tel que [Wireshark](https://www.wireshark.org/). Contactez le [support Microsoft](https://support.microsoft.com/) si nécessaire. 

## <a name="issues-that-may-occur-with-service-upgradesrestarts"></a>Problèmes qui peuvent se produire avec les mises à niveau/redémarrages du service

### <a name="symptoms"></a>Symptômes
- Les demandes peuvent être momentanément limitées.
- Il peut y avoir une chute des messages/demandes entrants.
- Le fichier journal peut contenir des messages d’erreur.
- Les applications peuvent être déconnectées du service pendant quelques secondes.

### <a name="cause"></a>Cause
Les mises à niveau et redémarrages du service principal peuvent causer ces problèmes dans vos applications.

### <a name="resolution"></a>Résolution
Si le code d’application utilise le Kit de développement logiciel (SDK), la stratégie de nouvelles tentatives est déjà intégrée et active. L’application se reconnectera sans que cela ait un impact significatif sur l’application/le flux de travail.

## <a name="unauthorized-access-send-claims-are-required"></a>Accès non autorisé : Envoyer les revendications requises

### <a name="symptoms"></a>Symptômes 
Vous pouvez voir cette erreur lorsque vous tentez d’accéder à une rubrique Service Bus à partir de Visual Studio sur un ordinateur local à l’aide d’une identité gérée affectée par l’utilisateur avec des autorisations d’envoi.

```bash
Service Bus Error: Unauthorized access. 'Send' claim\(s\) are required to perform this operation.
```

### <a name="cause"></a>Cause
L’identité ne dispose pas des autorisations nécessaires pour accéder à la rubrique Service Bus. 

### <a name="resolution"></a>Résolution
Pour résoudre cette erreur, installez la bibliothèque [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication/).  Pour plus d’informations, consultez [Authentification du développement local](../key-vault/general/service-to-service-authentication.md#local-development-authentication). 

Pour savoir comment affecter des autorisations à des rôles, consultez [Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Azure Service Bus](service-bus-managed-service-identity.md).

## <a name="service-bus-exception-put-token-failed"></a>Exception Service Bus : Échec du jeton PUT

### <a name="symptoms"></a>Symptômes
Lorsque vous essayez d’envoyer plus de 1 000 messages à l’aide de la même connexion Service Bus, le message d’erreur suivant s’affiche : 

`Microsoft.Azure.ServiceBus.ServiceBusException: Put token failed. status-code: 403, status-description: The maximum number of '1000' tokens per connection has been reached.` 

### <a name="cause"></a>Cause
Il existe une limite au nombre de jetons utilisés pour envoyer et recevoir des messages en utilisant une seule connexion à un espace de noms Service Bus. C’est 1 000. 

### <a name="resolution"></a>Résolution
Ouvrez une nouvelle connexion à l’espace de noms Service Bus pour envoyer plus de messages.

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants : 

- [Exceptions Azure Resource Manager](service-bus-resource-manager-exceptions.md). Répertorie les exceptions générées lors de l’interaction avec Azure Service Bus à l’aide d’Azure Resource Manager (via des modèles ou des appels directs).
- [Exceptions de messagerie](service-bus-messaging-exceptions.md). Fournit une liste d’exceptions générées par .NET Framework pour Azure Service Bus.