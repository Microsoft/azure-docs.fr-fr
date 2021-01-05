---
title: Protéger le contenu HLS Apple FairPlay - Azure hors connexion | Microsoft Docs
description: Cette rubrique fournit une vue d’ensemble et montre comment utiliser Azure Media Services pour chiffrer dynamiquement votre contenu HTTP Live Streaming (HLS) avec Apple FairPlay en mode hors connexion.
services: media-services
keywords: HLS, DRM, FairPlay Streaming (FPS), hors connexion, iOS 10
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/07/2020
ms.author: willzhan
ms.reviewer: dwgeo
ms.custom: devx-track-csharp
ms.openlocfilehash: 5e003e17490c2e35ef3f8adfdef6de0377cda7d5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89259708"
---
# <a name="offline-fairplay-streaming-for-ios"></a>FairPlay Streaming hors connexion pour iOS

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Sélectionnez la version de Media Services que vous utilisez :"]
> * [Version 3](../latest/offline-fairplay-for-ios.md)
> * [Version 2](media-services-protect-hls-with-offline-fairplay.md)

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Azure Media Services fournit un ensemble bien conçu de [services de protection de contenu](https://azure.microsoft.com/services/media-services/content-protection/), couvrant :

- Microsoft PlayReady
- Google Widevine
- Apple FairPlay
- Chiffrement AES-128

Le chiffrement de contenu Digital rights management (DRM)/Advanced Encryption Standard (AES) est exécuté dynamiquement sur demande pour divers protocoles de diffusion en continu. Des services de remise de clés de chiffrement AES/licence DRM sont également fournis par Media Services.

Outre la protection du contenu pour la diffusion en continu en ligne sur plusieurs protocoles de diffusion en continu, le mode hors connexion pour le contenu protégé est également une fonctionnalité fréquemment demandée. La prise en charge du mode hors connexion est nécessaire pour les scénarios suivants :

* La lecture avec une connexion Internet n’est pas disponible, par exemple en voyage ;
* Certains fournisseurs de contenu peuvent interdire la remise de licence DRM au-delà de la frontière d’un pays/d’une région. Si des utilisateurs souhaitent consulter du contenu pendant qu’ils sont en déplacement, un téléchargement hors connexion est nécessaire.
* Dans certains pays/régions, la disponibilité d’Internet et/ou de la bande passante est encore limitée. Les utilisateurs peuvent choisir de télécharger d’abord afin de pouvoir regarder le contenu dans une résolution suffisamment élevée pour une expérience d’affichage satisfaisante. Dans ce cas, le problème n’est généralement pas lié à la disponibilité du réseau mais à la bande passante réseau limitée. Les fournisseurs Over-the-top (OTT) ou de plateforme de vidéos en ligne (OVP) nécessitent la prise en charge du mode hors connexion.

Cet article couvre la prise en charge du mode hors connexion FairPlay Streaming (FPS) ciblant les appareils exécutant iOS 10 ou version ultérieure. Cette fonctionnalité n’est pas prise en charge pour d’autres plateformes Apple, telles que watchOS, tvOS ou Safari sur macOS.

## <a name="preliminary-steps"></a>Étapes préliminaires
Avant de mettre en œuvre DRM hors connexion pour FairPlay sur un appareil iOS 10+, vous devez :

* Vous familiariser avec la protection du contenu en ligne pour Fairplay. Pour plus d’informations, consultez les articles et exemples suivants :

    - [Apple FairPlay Streaming pour Azure Media Services est généralement disponible](https://azure.microsoft.com/blog/apple-FairPlay-streaming-for-azure-media-services-generally-available/)
    - [Protéger votre contenu HLS avec Apple FairPlay ou Microsoft PlayReady](./media-services-protect-hls-with-fairplay.md)
    - [Un exemple pour une diffusion en contenu FPS en ligne](https://azure.microsoft.com/resources/samples/media-services-dotnet-dynamic-encryption-with-FairPlay/)

* Obtenez le Kit SDK FPS d’Apple Developer Network. Le Kit SDK FPS contient deux composants :

    - le Kit SDK du serveur FPS, qui contient le module de sécurité de clés (KSM), des exemples clients, une spécification et un ensemble de vecteurs de test ;
    - le pack de déploiement FPS, qui contient la fonction D, une spécification avec des instructions sur le mode de génération du certificat FPS, une clé privée spécifique au client et la clé secrète d’application. Apple publie le pack de déploiement FPS uniquement aux fournisseurs de contenu sous licence.

## <a name="configuration-in-media-services"></a>Configuration dans Media Services
Pour la configuration du mode hors connexion FPS via le [Kit SDK .NET de Media Services](https://www.nuget.org/packages/windowsazure.mediaservices), utilisez le Kit SDK .NET de Media Services 4.0.0.4 ou version ultérieure, qui fournit l’API nécessaire pour configurer le mode hors connexion FPS.
Vous avez également besoin du code pour configurer la protection du contenu FPS en mode en ligne. Une fois que vous avez obtenu le code pour configurer la protection de contenu en mode en ligne pour FPS, il vous suffit d’effectuer les deux modifications suivantes.

## <a name="code-change-in-the-fairplay-configuration"></a>Modification du code dans la configuration FairPlay
Vous devez d’abord définir un booléen « activer le mode hors connexion », nommé objDRMSettings.EnableOfflineMode, qui est vrai lors de l’activation du scénario DRM hors connexion. En fonction de cet indicateur, modifiez la configuration FairPlay comme suit :

```csharp
if (objDRMSettings.EnableOfflineMode)
    {
        FairPlayConfiguration = Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
            objX509Certificate2,
            pfxPassword,
            pfxPasswordId,
            askId,
            iv, 
            RentalAndLeaseKeyType.PersistentUnlimited,
            0x9999);
    }
    else
    {
        FairPlayConfiguration = Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
            objX509Certificate2,
            pfxPassword,
            pfxPasswordId,
            askId,
            iv);
    }
```

## <a name="code-change-in-the-asset-delivery-policy-configuration"></a>Modification du code dans la configuration de stratégie de remise des éléments multimédia
La deuxième modification consiste à ajouter la troisième clé dans Dictionary<AssetDeliveryPolicyConfigurationKey, string>.
Ajoutez AssetDeliveryPolicyConfigurationKey comme indiqué ici :
 
```csharp
// FPS offline mode
    if (drmSettings.EnableOfflineMode)
    {
        objDictionary_AssetDeliveryPolicyConfigurationKey.Add(AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense, "true");
        Console.WriteLine("FPS OFFLINE MODE: AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense added into asset delivery policy configuration.");
    }

    // for IAssetDelivery for FPS
    IAssetDeliveryPolicy objIAssetDeliveryPolicy = objCloudMediaContext.AssetDeliveryPolicies.Create(
            drmSettings.AssetDeliveryPolicyName,
            AssetDeliveryPolicyType.DynamicCommonEncryptionCbcs,
            AssetDeliveryProtocol.HLS,
            objDictionary_AssetDeliveryPolicyConfigurationKey);
```

Après cette étape, la chaîne <Dictionary_AssetDeliveryPolicyConfigurationKey> dans la stratégie de remise des éléments multimédia FPS contient les trois entrées suivantes :

* AssetDeliveryPolicyConfigurationKey.FairPlayBaseLicenseAcquisitionUrl ou AssetDeliveryPolicyConfigurationKey.FairPlayLicenseAcquisitionUrl en fonction des facteurs tels que le serveur de clé/KSM FPS utilisé et si nous réutilisons la même stratégie de remise des éléments multimédia dans plusieurs éléments multimédia
* AssetDeliveryPolicyConfigurationKey.CommonEncryptionIVForCbcs
* AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense

Désormais votre compte Media Services est configuré pour fournir des licences FairPlay hors connexion.

## <a name="sample-ios-player"></a>Exemple de lecteur iOS
La prise en charge en mode hors connexion FPS est disponible uniquement sur iOS 10 et version ultérieure. Le Kit SDK 3.0 ou version ultérieure du serveur FPS contient le document ou l’exemple pour le mode hors connexion FPS. En particulier, le Kit SDK du serveur FPS (version 3.0 ou ultérieure) contient les deux éléments suivants associés au mode hors connexion :

* Document : « Lecture hors connexion avec FairPlay Streaming et HTTP Live Streaming. » Apple, 14 septembre 2016. Dans le Kit SDK du serveur FPS version 4.0, ce document est fusionné avec le document FPS principal.
* Exemple de code : Exemple HLSCatalog pour le mode hors connexion FPS dans \FairPlay Streaming Server SDK version 3.1\Development\Client\HLSCatalog_With_FPS\HLSCatalog\. Dans l’application en exemple HLSCatalog, les fichiers de code suivants servent à mettre en œuvre des fonctionnalités en mode hors connexion :

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

![Flux d’applications iOS FairPlay hors connexion](media/media-services-protect-hls-with-offline-FairPlay/media-services-offline-FairPlay-ios-app-streams.png)

Utilisez vos propres URL de vidéo de test, l’URL d’acquisition de licence FairPlay et l’URL de certificat d’application, si vous les avez configurées. Ou vous pouvez passer à la section suivante, qui fournit des exemples de test.

## <a name="integrated-test"></a>Test intégré
Trois exemples de tests dans Media Services couvrent les trois scénarios suivants :

* FPS protégé, avec vidéo, audio et autre piste audio ;
* FPS protégé, avec vidéo et audio, mais pas d’autre piste audio ;
* FPS protégé, avec vidéo seulement et pas d’audio.

Vous trouverez ces exemples sur ce [site de démo](https://aka.ms/poc#22), avec le certificat d’application correspondant hébergé dans une application Web Azure.
Avec la version 3 ou la version 4 de l’exemple du Kit SDK du serveur FPS, si une liste de lectures principale contient une autre audio, en mode hors connexion, elle ne lit que l’audio. Vous devez donc supprimer l’autre audio. En d’autres termes, les deuxième et troisième exemples mentionnés précédemment fonctionnent en mode en ligne et hors connexion. Le premier exemple ne lit l’audio qu’en mode hors connexion tandis que la diffusion en continu en ligne fonctionne correctement.

## <a name="faq"></a>Questions fréquentes (FAQ)
Les questions fréquemment posées suivantes vous aideront à résoudre vos problèmes :

- **Pourquoi seul l’audio mais pas la vidéo est lu en mode hors connexion ?** Ce comportement semble être conçu tel quel dans l’application en exemple. Lorsqu’une autre piste audio est présente (ce qui est le cas pour HLS), en mode hors connexion, iOS 10 et iOS 11 passent par défaut à l’autre piste audio. Pour compenser ce comportement en mode hors connexion FPS, supprimez l’autre piste audio du flux. Pour effectuer cette opération du côté de Media Services, ajoutez le filtre manifeste dynamique « audio-only=false. » En d’autres termes, une URL HLS se termine par .ism/manifest(format=m3u8-aapl,audio-only=false). 
- **Pourquoi l’audio est toujours lu sans vidéo en mode hors connexion après l’ajout de audio-only=false ?** En fonction de la conception de la clé du cache du réseau de distribution de contenu (CDN), le contenu peut être en cache. Videz le cache.
- **Le mode hors connexion FPS est-il également pris en charge sur iOS 11 comme sur iOS 10 ?** Oui. Le mode hors connexion FPS est pris en charge sur iOS 10 et iOS 11.
- **Pourquoi ne puis-je pas trouver le document Lecture hors connexion avec FairPlay Streaming et HTTP Live Streaming dans le Kit SDK du serveur FPS ?** Depuis le Kit SDK du serveur FPS version 4, ce document a été fusionné avec le « Guide de programmation FairPlay Streaming ».
- **À quoi correspond le dernier paramètre dans l’API suivante pour le mode hors connexion FPS ?** 
`Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(objX509Certificate2, pfxPassword, pfxPasswordId, askId, iv, RentalAndLeaseKeyType.PersistentUnlimited, 0x9999);`

    Pour obtenir la documentation sur cette API, consultez [Méthode FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration](/dotnet/api/microsoft.windowsazure.mediaservices.client.fairplay.fairplayconfiguration.createserializedfairplayoptionconfiguration?view=azure-dotnet). Le paramètre représente la durée de location hors connexion avec la seconde pour unité.
- **Quelle est la structure du fichier téléchargé/hors connexion sur les appareils iOS ?** La structure du fichier téléchargé sur un appareil iOS ressemble à la capture d’écran suivante. Le dossier `_keys` stocke les licences FPS téléchargées, avec un fichier de magasin pour chaque hôte de service de licence. Le dossier `.movpkg` stocke le contenu audio et vidéo. Le premier dossier dont le nom se termine par un tiret suivi par un numérique contient un contenu vidéo. La valeur numérique est la PeakBandwidth des rendus de vidéo. Le deuxième dossier dont le nom se termine par un tiret suivi de 0 contient un contenu audio. Le troisième dossier nommé « Données » contient la liste de lectures principale du contenu FPS. Enfin, boot.xml fournit une description complète du contenu du dossier `.movpkg`. 

![Structure de fichier d’application exemple iOS FairPlay hors connexion](media/media-services-protect-hls-with-offline-FairPlay/media-services-offline-FairPlay-file-structure.png)

Un fichier boot.xml en exemple :
```xml
<?xml version="1.0" encoding="UTF-8"?>
<HLSMoviePackage xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://apple.com/IMG/Schemas/HLSMoviePackage" xsi:schemaLocation="http://apple.com/IMG/Schemas/HLSMoviePackage /System/Library/Schemas/HLSMoviePackage.xsd">
  <Version>1.0</Version>
  <HLSMoviePackageType>PersistedStore</HLSMoviePackageType>
  <Streams>
    <Stream ID="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" Path="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(127000)/Manifest(aac_eng_2_127,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
    <Stream ID="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" Path="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(161000)/Manifest(video,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
  </Streams>
  <MasterPlaylist>
    <NetworkURL>https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/manifest(format=m3u8-aapl,audio-only=false)</NetworkURL>
  </MasterPlaylist>
  <DataItems Directory="Data">
    <DataItem>
      <ID>CB50F631-8227-477A-BCEC-365BBF12BCC0</ID>
      <Category>Playlist</Category>
      <Name>master.m3u8</Name>
      <DataPath>Playlist-master.m3u8-CB50F631-8227-477A-BCEC-365BBF12BCC0.data</DataPath>
      <Role>Master</Role>
    </DataItem>
  </DataItems>
</HLSMoviePackage>
```

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="summary"></a>Résumé
Ce document inclut les étapes ci-dessous et des informations que vous pouvez utiliser pour implémenter le mode hors connexion FPS :

* La configuration de la protection de contenu Media Services via l’API .NET de Media Services configure le chiffrement FairPlay dynamique ainsi que la remise des licences FairPlay dans Media Services.
* Un lecteur iOS basé sur l’exemple du Kit SDK du serveur FPS configure un lecteur iOS pouvant lire le contenu FPS en mode de diffusion en continu en ligne ou un mode hors connexion.
* Des exemples de vidéos FPS sont utilisés pour les tests en mode hors connexion et en diffusion continue en ligne.
* Un forum aux questions répond aux questions sur le mode hors connexion FPS.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
