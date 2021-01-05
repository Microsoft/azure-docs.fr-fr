---
title: FairPlay Streaming hors connexion pour iOS avec Azure Media Services v3
description: Cette rubrique fournit une vue d’ensemble et montre comment utiliser Azure Media Services v3 pour chiffrer dynamiquement votre contenu HTTP Live Streaming (HLS) avec Apple FairPlay en mode hors connexion.
services: media-services
keywords: HLS, DRM, FairPlay Streaming (FPS), hors connexion, iOS 10
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.assetid: 7c3b35d9-1269-4c83-8c91-490ae65b0817
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: willzhan
ms.custom: devx-track-csharp
ms.openlocfilehash: 886a8aa1a6f062b5fe01476e387742f34efc0c56
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90532237"
---
# <a name="offline-fairplay-streaming-for-ios-with-media-services-v3"></a>FairPlay Streaming hors connexion pour iOS avec Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

 Azure Media Services fournit un ensemble bien conçu de [services de protection de contenu](https://azure.microsoft.com/services/media-services/content-protection/), couvrant :

- Microsoft PlayReady
- Google Widevine
    
    Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.
- Apple FairPlay
- Chiffrement AES-128

Le chiffrement de contenu Digital rights management (DRM)/Advanced Encryption Standard (AES) est exécuté dynamiquement sur demande pour divers protocoles de diffusion en continu. Des services de remise de clés de chiffrement AES/licence DRM sont également fournis par Media Services.

Outre la protection du contenu pour la diffusion en continu en ligne sur plusieurs protocoles de diffusion en continu, le mode hors connexion pour le contenu protégé est également une fonctionnalité fréquemment demandée. La prise en charge du mode hors connexion est nécessaire pour les scénarios suivants :

* La lecture avec une connexion Internet n’est pas disponible, par exemple en voyage ;
* Certains fournisseurs de contenu peuvent interdire la remise de licence DRM au-delà de la frontière d’un pays/d’une région. Si des utilisateurs souhaitent consulter du contenu pendant qu’ils sont en déplacement, un téléchargement hors connexion est nécessaire.
* Dans certains pays/régions, la disponibilité d’Internet et/ou de la bande passante est encore limitée. Les utilisateurs peuvent choisir de télécharger d’abord afin de pouvoir regarder le contenu dans une résolution suffisamment élevée pour une expérience d’affichage satisfaisante. Dans ce cas, le problème n’est généralement pas lié à la disponibilité du réseau mais à la bande passante réseau limitée. Les fournisseurs Over-the-top (OTT) ou de plateforme de vidéos en ligne (OVP) nécessitent la prise en charge du mode hors connexion.

Cet article couvre la prise en charge du mode hors connexion FairPlay Streaming (FPS) ciblant les appareils exécutant iOS 10 ou version ultérieure. Cette fonctionnalité n’est pas prise en charge pour d’autres plateformes Apple, telles que watchOS, tvOS ou Safari sur macOS.

> [!NOTE]
> DRM hors connexion est uniquement facturé pour une requête unique de licence lorsque vous téléchargez le contenu. Les erreurs ne sont pas facturées.

## <a name="prerequisites"></a>Prérequis

Avant de mettre en œuvre DRM hors connexion pour FairPlay sur un appareil iOS 10+, vous devez :

* Familiarisez-vous avec la protection du contenu en ligne pour FairPlay : 

    - [Configuration et conditions de licence Apple FairPlay](fairplay-license-overview.md)
    - [Utilisation du chiffrement dynamique DRM et du service de remise des licences](protect-with-drm.md)
    - Exemple .NET qui comprend la configuration du streaming FPS en ligne : [ConfigureFairPlayPolicyOptions](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L505)
* Obtenez le Kit SDK FPS d’Apple Developer Network. Le Kit SDK FPS contient deux composants :

    - le Kit SDK du serveur FPS, qui contient le module de sécurité de clés (KSM), des exemples clients, une spécification et un ensemble de vecteurs de test ;
    - le pack de déploiement FPS, qui contient la fonction D, une spécification avec des instructions sur le mode de génération du certificat FPS, une clé privée spécifique au client et la clé secrète d’application. Apple publie le pack de déploiement FPS uniquement aux fournisseurs de contenu sous licence.
* Clonez https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git. 

    Pour ajouter des configurations FairPlay, vous devrez modifier le code fourni dans [Encrypt with DRM using .NET](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM).  

## <a name="configure-content-protection-in-azure-media-services"></a>Configurer la protection du contenu dans Azure Media Services

Dans la méthode [GetOrCreateContentKeyPolicyAsync](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L189), effectuez les étapes suivantes :

Décommentez le code qui configure l’option de stratégie FairPlay :

```csharp
ContentKeyPolicyFairPlayConfiguration fairplayConfig = ConfigureFairPlayPolicyOptions();
```

En outre, décommentez le code qui ajoute CBCS ContentKeyPolicyOption à la liste des ContentKeyPolicyOptions.

```csharp
options.Add(
    new ContentKeyPolicyOption()
    {
        Configuration = fairplayConfig,
        Restriction = restriction,
        Name = "ContentKeyPolicyOption_CBCS"
    });
```

## <a name="enable-offline-mode"></a>Activer le mode hors connexion

Pour activer le mode hors connexion, créez un StreamingPolicy personnalisé et utiliser son nom lors de la création d’un StreamingLocator dans [CreateStreamingLocatorAsync](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L561).
 
```csharp
CommonEncryptionCbcs objStreamingPolicyInput= new CommonEncryptionCbcs()
{
    Drm = new CbcsDrmConfiguration()
    {
        FairPlay = new StreamingPolicyFairPlayConfiguration()
        {
            AllowPersistentLicense = true // This enables offline mode
        }
    },
    EnabledProtocols = new EnabledProtocols()
    {
        Hls = true,
        Dash = true // Even though DASH under CBCS is not supported for either CSF or CMAF, HLS-CMAF-CBCS uses DASH-CBCS fragments in its HLS playlist
    },

    ContentKeys = new StreamingPolicyContentKeys()
    {
        // Default key must be specified if keyToTrackMappings is present
        DefaultKey = new DefaultKey()
        {
            Label = "CBCS_DefaultKeyLabel"
        }
    }
}

```

Désormais votre compte Media Services est configuré pour fournir des licences FairPlay hors connexion.

## <a name="sample-ios-player"></a>Exemple de lecteur iOS

La prise en charge en mode hors connexion FPS est disponible uniquement sur iOS 10 et version ultérieure. Le Kit SDK 3.0 ou version ultérieure du serveur FPS contient le document ou l’exemple pour le mode hors connexion FPS. En particulier, le Kit SDK du serveur FPS (version 3.0 ou ultérieure) contient les deux éléments suivants associés au mode hors connexion :

* Document : « Lecture hors connexion avec FairPlay Streaming et HTTP Live Streaming. » Apple, 14 septembre 2016. Dans le Kit SDK du serveur FPS version 4.0, ce document est fusionné avec le document FPS principal.
* Exemple de code : exemple HLSCatalog (fait partie du SDK FPS Server d’Apple) pour le mode hors connexion FPS dans \FairPlay Streaming Server SDK version 3.1\Development\Client\HLSCatalog_With_FPS\HLSCatalog\. Dans l’application en exemple HLSCatalog, les fichiers de code suivants servent à mettre en œuvre des fonctionnalités en mode hors connexion :

    - Fichier de code AssetPersistenceManager.swift : AssetPersistenceManager est la principale classe de cet exemple qui démontre comment :

        - gérer le téléchargement des flux HLS, telles que les API pour démarrer et annuler les téléchargements et pour supprimer des éléments multimédia existants des appareils ;
        - contrôler la progression du téléchargement.
    - Fichiers de code AssetListTableViewController.swift et AssetListTableViewCell.swift : AssetListTableViewController est l’interface principale de cet exemple. Elle fournit une liste d’éléments multimédia permettant à l’exemple de lire, télécharger, supprimer ou annuler un téléchargement. 

Ces étapes indiquent comment configurer un lecteur iOS en cours d’exécution. En supposant un démarrage à partir de l’exemple HLSCatalog dans la version 4.0.1 du Kit SDK du serveur FPS, apportez les modifications suivantes au code :

Dans HLSCatalog\Shared\Managers\ContentKeyDelegate.swift,mettez en œuvre la méthode `requestContentKeyFromKeySecurityModule(spcData: Data, assetID: String)` en utilisant le code suivant. Faites de « drmUr » une variable affectée à l’URL HLS.

```swift
    var ckcData: Data? = nil
    
    let semaphore = DispatchSemaphore(value: 0)
    let postString = "spc=\(spcData.base64EncodedString())&assetId=\(assetIDString)"
    
    if let postData = postString.data(using: .ascii, allowLossyConversion: true), let drmServerUrl = URL(string: self.drmUrl) {
        var request = URLRequest(url: drmServerUrl)
        request.httpMethod = "POST"
        request.setValue(String(postData.count), forHTTPHeaderField: "Content-Length")
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        request.httpBody = postData
        
        URLSession.shared.dataTask(with: request) { (data, _, error) in
            if let data = data, var responseString = String(data: data, encoding: .utf8) {
                responseString = responseString.replacingOccurrences(of: "<ckc>", with: "").replacingOccurrences(of: "</ckc>", with: "")
                ckcData = Data(base64Encoded: responseString)
            } else {
                print("Error encountered while fetching FairPlay license for URL: \(self.drmUrl), \(error?.localizedDescription ?? "Unknown error")")
            }
            
            semaphore.signal()
            }.resume()
    } else {
        fatalError("Invalid post data")
    }
    
    semaphore.wait()
    return ckcData
```

Dans HLSCatalog\Shared\Managers\ContentKeyDelegate.swift,mettez en œuvre la méthode `requestApplicationCertificate()`. Cette mise en œuvre dépend de l’incorporation du certificat (clé publique uniquement) à l’appareil ou de l’hébergement du certificat sur le Web. La mise en œuvre suivante utilise le certificat d’application hébergé utilisé dans les exemples de tests. Faites de « certUrl » une variable contenant l’URL du certificat d’application.

```swift
func requestApplicationCertificate() throws -> Data {

        var applicationCertificate: Data? = nil
        do {
            applicationCertificate = try Data(contentsOf: URL(string: certUrl)!)
        } catch {
            print("Error loading FairPlay application certificate: \(error)")
        }
        
        return applicationCertificate
    }
```

Pour le test intégré final, l’URL de la vidéo et l’URL du certificat d’application sont fournies dans la section « Test intégré ».

Dans HLSCatalog\Shared\Resources\Streams.plist, ajoutez l’URL de votre vidéo de test. Pour l’ID de clé de contenu, utilisez l’URL d’acquisition de licence FairPlay avec le protocole skd comme valeur unique.

![Flux d’applications iOS FairPlay hors connexion](media/offline-fairplay-for-ios/offline-fairplay-ios-app-streams.png)

Utilisez vos propres URL de vidéo de test, l’URL d’acquisition de licence FairPlay et l’URL de certificat d’application, si vous les avez configurées. Ou vous pouvez passer à la section suivante, qui fournit des exemples de test.

## <a name="integrated-test"></a>Test intégré

Trois exemples de tests dans Media Services couvrent les trois scénarios suivants :

* FPS protégé, avec vidéo, audio et autre piste audio ;
* FPS protégé, avec vidéo et audio, mais pas d’autre piste audio ;
* FPS protégé, avec vidéo seulement et pas d’audio.

Vous trouverez ces exemples sur ce [site de démo](https://aka.ms/poc#22), avec le certificat d’application correspondant hébergé dans une application Web Azure.
Avec la version 3 ou la version 4 de l’exemple du Kit SDK du serveur FPS, si une liste de lectures principale contient une autre audio, en mode hors connexion, elle ne lit que l’audio. Vous devez donc supprimer l’autre audio. En d’autres termes, les deuxième et troisième exemples mentionnés précédemment fonctionnent en mode en ligne et hors connexion. Le premier exemple ne lit l’audio qu’en mode hors connexion tandis que la diffusion en continu en ligne fonctionne correctement.

## <a name="faq"></a>Questions fréquentes (FAQ)

Consultez les [Questions fréquemment posées sur la résolution des problèmes](frequently-asked-questions.md#why-does-only-audio-play-but-not-video-during-offline-mode).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [protéger avec AES-128](protect-with-aes128.md)
