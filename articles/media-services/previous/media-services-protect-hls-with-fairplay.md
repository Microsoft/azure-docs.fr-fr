---
title: Protéger le contenu HLS avec Microsoft PlayReady ou Apple FairPlay - Azure | Microsoft Docs
description: Cette rubrique fournit une vue d’ensemble et montre comment utiliser Azure Media Services pour chiffrer dynamiquement votre contenu HTTP Live Streaming (HLS) avec Apple FairPlay. Elle montre également comment utiliser le service de distribution de licences Media Services pour fournir des licences FairPlay aux clients.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 2f9b3cdd0b2080a26a9b1948263a7638dc66f2b3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89259742"
---
# <a name="protect-your-hls-content-with-apple-fairplay-or-microsoft-playready"></a>Protéger votre contenu HLS avec Apple FairPlay ou Microsoft PlayReady

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com/pricing/free-trial/).   > Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).
>

Azure Media Services vous permet de chiffrer dynamiquement votre contenu HTTP Live Streaming (HLS) à l’aide des formats suivants :  

* **Clé en clair de l’enveloppe AES-128**

    Le segment entier est chiffré à l’aide du mode **AES-128 CBC**. Le déchiffrement du flux est pris en charge par iOS et le lecteur OS X en mode natif. Pour plus d’informations, consultez la page [Utilisation du chiffrement dynamique AES-128 et du service de distribution des clés](media-services-protect-with-aes128.md).
* **Apple FairPlay**

    Les échantillons audio et vidéo individuels sont chiffrés à l’aide du mode **AES-128 CBC**. **FairPlay Streaming** (FPS) est intégré dans les systèmes d'exploitation de l’appareil, avec prise en charge native sur iOS et Apple TV. Safari sur OS X active FPS à l’aide de la prise en charge de l’interface EME (Encrypted Media Extensions).
* **Microsoft PlayReady**

L’image suivante présente le flux de travail **HLS + FairPlay ou chiffrement dynamique PlayReady**.

![Diagramme de flux de travail de chiffrement dynamique](./media/media-services-content-protection-overview/media-services-content-protection-with-FairPlay.png)

Cet article montre comment utiliser Media Services pour chiffrer dynamiquement votre contenu HLS avec Apple FairPlay. Elle montre également comment utiliser le service de distribution de licences Media Services pour fournir des licences FairPlay aux clients.

> [!NOTE]
> Si vous souhaitez également chiffrer votre contenu HLS avec PlayReady, vous devez créer une clé de contenu commune et l’associer à votre élément multimédia. Vous devez également configurer la stratégie d’autorisation de la clé de contenu, comme décrit dans la rubrique [Using PlayReady dynamic common encryption](media-services-protect-with-playready-widevine.md) (Utilisation du chiffrement commun dynamique PlayReady).
>
>

## <a name="requirements-and-considerations"></a>Conditions requises et éléments à prendre en compte

Les éléments suivants sont nécessaires pour utiliser Media Services afin de fournir du contenu HLS chiffré avec FairPlay et des licences FairPlay :

  * Un compte Azure. Pour plus d’informations, consultez la page [Version d’évaluation gratuite d’Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F).
  * Un compte Media Services. Pour en créer un, consultez l’article [Création d’un compte Azure Media Services à l’aide du portail Azure](media-services-portal-create-account.md).
  * S’inscrire au programme [Apple Developer Program](https://developer.apple.com/).
  * Apple exige que le propriétaire du contenu se procure le [package de déploiement](https://developer.apple.com/contact/fps/). Indiquez que vous avez déjà implémenté le module de sécurité des clés (KSM) avec Media Services et que vous avez besoin du package FPS final. Voici les instructions figurant dans le package FPS final pour générer la certification et obtenir la clé secrète d’application (ASK). Utilisez la clé ASK pour configurer FairPlay.
  * Kit de développement logiciel (SDK) .NET Azure Media Services version **3.6.0** ou ultérieure.

Les éléments suivants doivent être définis du côté de la remise de clé Media Services :

  * **Certificat d’application** : fichier .pfx qui contient la clé privée. Vous devez créer ce fichier et le chiffrer avec un mot de passe.

       Lorsque vous configurez une stratégie de remise de clé, vous devez fournir ce mot de passe et le fichier .pfx au format Base64.

      Les étapes suivantes décrivent la génération d’un fichier de certificat .pfx pour FairPlay :

    1. Installez OpenSSL à partir de https://slproweb.com/products/Win32OpenSSL.html.

        Accédez au dossier dans lequel se situent le certificat FairPlay et les autres fichiers fournis par Apple.
    2. Exécutez la commande suivante à partir de la ligne de commande. Elle permet de convertir le fichier .cer en fichier .pem.

        « C:\OpenSSL-Win32\bin\openssl.exe" x509 -inform der -in FairPlay.cer -out FairPlay-out.pem
    3. Exécutez la commande suivante à partir de la ligne de commande. Elle permet de convertir le fichier .pem en fichier .pfx avec la clé privée. Le mot de passe du fichier .pfx est ensuite demandé par OpenSSL.

        « C:\OpenSSL-Win32\bin\openssl.exe" pkcs12 -export -out FairPlay-out.pfx -inkey privatekey.pem -in FairPlay-out.pem -passin file:privatekey-pem-pass.txt
  * **Mot de passe du certificat d’application** : mot de passe pour créer le fichier .pfx.
  * **ID de mot de passe du certificat d'application** : vous devez charger le mot de passe comme vous chargez les autres clés Media Services. Utilisez la valeur d’énumération **ContentKeyType.FairPlayPfxPassword** pour obtenir l’ID Media Services. C’est ce que vous devez utiliser dans l’option de stratégie de remise de clé.
  * **iv** : valeur aléatoire de 16 octets. Elle doit correspondre au vecteur d’initialisation de la stratégie de distribution d’éléments multimédias. Vous générez le vecteur d’initialisation et le placez aux deux endroits : la stratégie de distribution d’éléments multimédias et l’option de stratégie de remise de clé.
  * **ASK** : cette clé est reçue quand vous générez la certification à l’aide du portail des développeurs Apple. Chaque équipe de développement reçoit une ASK unique. Enregistrez une copie de la clé ASK et stockez-la dans un endroit sûr. Vous devrez configurer la clé ASK comme FairPlayAsk pour Media Services plus tard.
  * **ID de clé ASK** : cet ID est obtenu lorsque vous chargez la clé ASK dans Media Services. Vous devez charger la clé ASK à l’aide de la valeur d’énumération **ContentKeyType.FairPlayAsk**. L’ID Media Services est retourné dans le résultat. C’est cet ID qui doit être utilisé lors de la définition de l’option de stratégie de remise de clé.

Les éléments suivants doivent être définis par FPS côté client :

  * **Certificat d’application** : fichier .cer/.der qui contient la clé publique utilisée par le système d’exploitation pour chiffrer une charge utile. Media Services doit le connaître, car il est requis par le lecteur. Le service de remise de clé le déchiffre à l'aide de la clé privée correspondante.

Pour lire un flux chiffré FairPlay, vous devez obtenir une clé ASK réelle en premier lieu, puis générer un certificat réel. Ce processus crée les trois composants suivants :

  * Fichier .der
  * Fichier .pfx
  * Mot de passe du fichier .pfx

Les clients suivants prennent en charge HLS avec le chiffrement **AES-128 CBC** : Safari sur OS X, Apple TV, iOS.

## <a name="configure-fairplay-dynamic-encryption-and-license-delivery-services"></a>Configurer le chiffrement dynamique FairPlay et des services de remise de licences
Voici les étapes générales que vous devez suivre pour protéger vos éléments multimédias avec FairPlay, en utilisant le service de remise de licences Media Services, ainsi que le chiffrement dynamique.

1. Créer un élément multimédia et y charger des fichiers.
2. Encoder l’élément multimédia qui contient le fichier au débit adaptatif MP4 défini.
3. Créer une clé de contenu et l’associer à l’élément multimédia encodé.  
4. Configurer la stratégie d'autorisation de la clé de contenu. Spécifiez les éléments suivants :

   * La méthode de remise (dans ce cas, FairPlay).
   * la configuration des options de stratégie FairPlay. Pour plus d’informations sur la configuration de FairPlay, consultez la méthode **ConfigureFairPlayPolicyOptions()** dans l’exemple ci-dessous.

     > [!NOTE]
     > Généralement, vous n’aurez à configurer les options de stratégie FairPlay qu’une seule fois, car vous n’aurez qu’un seul jeu de certification et de clé ASK.
     >
     >
   * Les restrictions (d’ouverture ou jeton).
   * Les informations propres au type de remise de clé qui définissent la façon dont la clé est fournie au client.
5. Configurer la stratégie de distribution d’éléments multimédias. La configuration de la stratégie de distribution inclut :

   * le protocole de distribution (HLS) ;
   * le type de chiffrement dynamique (Common CBC Encryption) ;
   * l’URL d’acquisition de licence.

     > [!NOTE]
     > Si vous souhaitez fournir un flux chiffré avec FairPlay et un autre système de gestion des droits numériques (DRM), vous devez configurer des stratégies de remise distinctes :
     >
     > * Une stratégie IAssetDeliveryPolicy pour configurer Dynamic Adaptive Streaming via HTTP (DASH) avec Common Encryption (CENC) (PlayReady + Widevine), et Smooth avec PlayReady
     > * Une autre stratégie IAssetDeliveryPolicy pour configurer FairPlay pour HLS
     >
     >
6. Créez un localisateur à la demande afin d’obtenir une URL de diffusion en continu.

## <a name="use-fairplay-key-delivery-by-player-apps"></a>Utiliser la remise de clé FairPlay des applications de lecteur
Vous pouvez développer des applications de lecteur à l’aide du Kit de développement logiciel (SDK) iOS. Pour pouvoir lire le contenu FairPlay, vous devez implémenter le protocole d’échange de licence. Ce protocole n’est pas spécifié par Apple. Chaque application doit décider comment envoyer des requêtes de distribution de clés. Le service de remise de clé Media Services FairPlay s’attend à ce que le certificat SPC soit fourni en tant que message encodé www-form-url au format suivant :

`spc=<Base64 encoded SPC>`

> [!NOTE]
> Azure Media Player prend en charge la lecture de FairPlay. Consultez la [documentation Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) pour obtenir plus d’informations.
>
>

## <a name="streaming-urls"></a>URL de diffusion
Si votre ressource a été chiffrée avec plusieurs DRM, vous devez utiliser une balise de chiffrement dans l’URL de streaming : (format=’m3u8-aapl’, encryption=’xxx’).

Les considérations suivantes s'appliquent :

* Seul zéro ou un type de chiffrement peut être spécifié.
* Le type de chiffrement ne doit pas être spécifié dans l’URL si un seul chiffrement a été appliqué à l’élément multimédia.
* Le type de chiffrement ne tient pas compte de la casse.
* Les types de chiffrement suivants peuvent être spécifiés :  
  * **cenc** :  chiffrement commun (Playready ou Widevine)
  * **cbcs-aapl** : FairPlay
  * **cbc** : chiffrement de l'enveloppe AES

## <a name="create-and-configure-a-visual-studio-project"></a>Créer et configurer un projet Visual Studio

1. Configurez votre environnement de développement et ajoutez des informations de connexion au fichier app.config selon la procédure décrite dans l’article [Développement Media Services avec .NET](media-services-dotnet-how-to-use.md). 
2. Ajoutez les éléments suivants aux **appSettings** définis dans votre fichier app.config :

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```

## <a name="example"></a>Exemple

L’exemple suivant illustre la possibilité d’utiliser Media Services pour distribuer votre contenu chiffré avec FairPlay. Cette fonctionnalité a été introduite dans le Kit de développement logiciel (SDK) Azure Media Services pour .NET version 3.6.0. 

Remplacez le code dans votre fichier Program.cs par le code présenté dans cette section.

>[!NOTE]
>Un nombre limite de 1 000 000 a été défini pour les différentes stratégies AMS (par exemple, pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy). Vous devez utiliser le même ID de stratégie si vous utilisez toujours les mêmes jours / autorisations d’accès, par exemple, les stratégies pour les localisateurs destinées à demeurer en place pendant une longue période (stratégies sans chargement). Pour plus d’informations, consultez [cet](media-services-dotnet-manage-entities.md#limit-access-policies) article.

Veillez à mettre à jour les variables pour pointer vers les dossiers où se trouvent vos fichiers d'entrée.

```csharp
using System;
using System.Collections.Generic;
using System.Configuration;
using System.IO;
using System.Linq;
using System.Threading;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
using Microsoft.WindowsAzure.MediaServices.Client.FairPlay;
using Newtonsoft.Json;
using System.Security.Cryptography.X509Certificates;

namespace DynamicEncryptionWithFairPlay
{
    class Program
    {
        // Read values from the App.config file.
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        private static readonly Uri _sampleIssuer =
            new Uri(ConfigurationManager.AppSettings["Issuer"]);
        private static readonly Uri _sampleAudience =
            new Uri(ConfigurationManager.AppSettings["Audience"]);

        // Field for service context.
        private static CloudMediaContext _context = null;

        private static readonly string _mediaFiles =
            Path.GetFullPath(@"../..\Media");

        private static readonly string _singleMP4File =
            Path.Combine(_mediaFiles, @"BigBuckBunny.mp4");

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            bool tokenRestriction = false;
            string tokenTemplateString = null;

            IAsset asset = UploadFileAndCreateAsset(_singleMP4File);
            Console.WriteLine("Uploaded asset: {0}", asset.Id);

            IAsset encodedAsset = EncodeToAdaptiveBitrateMP4Set(asset);
            Console.WriteLine("Encoded asset: {0}", encodedAsset.Id);

            IContentKey key = CreateCommonCBCTypeContentKey(encodedAsset);
            Console.WriteLine("Created key {0} for the asset {1} ", key.Id, encodedAsset.Id);
            Console.WriteLine("FairPlay License Key delivery URL: {0}", key.GetKeyDeliveryUrl(ContentKeyDeliveryType.FairPlay));
            Console.WriteLine();

            if (tokenRestriction)
                tokenTemplateString = AddTokenRestrictedAuthorizationPolicy(key);
            else
                AddOpenAuthorizationPolicy(key);

            Console.WriteLine("Added authorization policy: {0}", key.AuthorizationPolicyId);
            Console.WriteLine();

            CreateAssetDeliveryPolicy(encodedAsset, key);
            Console.WriteLine("Created asset delivery policy. \n");
            Console.WriteLine();

            if (tokenRestriction && !String.IsNullOrEmpty(tokenTemplateString))
            {
                // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
                // back into a TokenRestrictionTemplate class instance.
                TokenRestrictionTemplate tokenTemplate =
                    TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

                // Generate a test token based on the data in the given TokenRestrictionTemplate.
                // Note, you need to pass the key id Guid because we specified
                // TokenClaim.ContentKeyIdentifierClaim in during the creation of TokenRestrictionTemplate.
                Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);
                string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey,
                                            DateTime.UtcNow.AddDays(365));
                Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
                Console.WriteLine();
            }

            string url = GetStreamingOriginLocator(encodedAsset);
            Console.WriteLine("Encrypted HLS URL: {0}/manifest(format=m3u8-aapl)", url);

            Console.ReadLine();
        }

        static public IAsset UploadFileAndCreateAsset(string singleFilePath)
        {
            if (!File.Exists(singleFilePath))
            {
                Console.WriteLine("File does not exist.");
                return null;
            }

            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
            IAsset inputAsset = _context.Assets.Create(assetName, AssetCreationOptions.None);

            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return inputAsset;
        }

        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset inputAsset)
        {
            var encodingPreset = "Adaptive Streaming";

            IJob job = _context.Jobs.Create(String.Format("Encoding {0}", inputAsset.Name));

            var mediaProcessors =
            _context.MediaProcessors.Where(p => p.Name.Contains("Media Encoder Standard")).ToList();

            var latestMediaProcessor =
            mediaProcessors.OrderBy(mp => new Version(mp.Version)).LastOrDefault();

            ITask encodeTask = job.Tasks.AddNew("Encoding", latestMediaProcessor, encodingPreset, TaskOptions.None);
            encodeTask.InputAssets.Add(inputAsset);
            encodeTask.OutputAssets.AddNew(String.Format("{0} as {1}", inputAsset.Name, encodingPreset), AssetCreationOptions.StorageEncrypted);

            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
            job.Submit();
            job.GetExecutionProgressTask(CancellationToken.None).Wait();

            return job.OutputMediaAssets[0];
        }

        static public IContentKey CreateCommonCBCTypeContentKey(IAsset asset)
        {
            // Create HLS SAMPLE AES encryption content key
            Guid keyId = Guid.NewGuid();
            byte[] contentKey = GetRandomBuffer(16);

            IContentKey key = _context.ContentKeys.Create(
                        keyId,
                        contentKey,
                        "ContentKey",
                        ContentKeyType.CommonEncryptionCbcs);

            // Associate the key with the asset.
            asset.ContentKeys.Add(key);

            return key;
        }


        static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
        {
            // Create ContentKeyAuthorizationPolicy with Open restrictions
            // and create authorization policy          

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
                    {
                    new ContentKeyAuthorizationPolicyRestriction
                    {
                        Name = "Open",
                        KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
                        Requirements = null
                    }
                    };


            // Configure FairPlay policy option.
            string FairPlayConfiguration = ConfigureFairPlayPolicyOptions();

            IContentKeyAuthorizationPolicyOption FairPlayPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("",
            ContentKeyDeliveryType.FairPlay,
            restrictions,
            FairPlayConfiguration);


            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common CBC Content Key with no restrictions").
                Result;

            contentKeyAuthorizationPolicy.Options.Add(FairPlayPolicy);

            // Associate the content key authorization policy with the content key.
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;
        }

        public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
        {
            string tokenTemplateString = GenerateTokenRequirements();

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
                    {
                    new ContentKeyAuthorizationPolicyRestriction
                    {
                        Name = "Token Authorization Policy",
                        KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                        Requirements = tokenTemplateString,
                    }
                    };

            // Configure FairPlay policy option.
            string FairPlayConfiguration = ConfigureFairPlayPolicyOptions();


            IContentKeyAuthorizationPolicyOption FairPlayPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                   ContentKeyDeliveryType.FairPlay,
                   restrictions,
                   FairPlayConfiguration);

            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common CBC Content Key with token restrictions").
                Result;

            contentKeyAuthorizationPolicy.Options.Add(FairPlayPolicy);

            // Associate the content key authorization policy with the content key
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;

            return tokenTemplateString;
        }

        private static string ConfigureFairPlayPolicyOptions()
        {
            // For testing you can provide all zeroes for ASK bytes together with the cert from Apple FPS SDK.
            // However, for production you must use a real ASK from Apple bound to a real prod certificate.
            byte[] askBytes = Guid.NewGuid().ToByteArray();
            var askId = Guid.NewGuid();
            // Key delivery retrieves askKey by askId and uses this key to generate the response.
            IContentKey askKey = _context.ContentKeys.Create(
                        askId,
                        askBytes,
                        "askKey",
                        ContentKeyType.FairPlayASk);

            //Customer password for creating the .pfx file.
            string pfxPassword = "<customer password for creating the .pfx file>";
            // Key delivery retrieves pfxPasswordKey by pfxPasswordId and uses this key to generate the response.
            var pfxPasswordId = Guid.NewGuid();
            byte[] pfxPasswordBytes = System.Text.Encoding.UTF8.GetBytes(pfxPassword);
            IContentKey pfxPasswordKey = _context.ContentKeys.Create(
                        pfxPasswordId,
                        pfxPasswordBytes,
                        "pfxPasswordKey",
                        ContentKeyType.FairPlayPfxPassword);

            // iv - 16 bytes random value, must match the iv in the asset delivery policy.
            byte[] iv = Guid.NewGuid().ToByteArray();

            //Specify the .pfx file created by the customer.
            var appCert = new X509Certificate2("path to the .pfx file created by the customer", pfxPassword, X509KeyStorageFlags.Exportable);

            string FairPlayConfiguration =
            Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
                appCert,
                pfxPassword,
                pfxPasswordId,
                askId,
                iv);

            return FairPlayConfiguration;
        }

        static private string GenerateTokenRequirements()
        {
            TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);

            template.PrimaryVerificationKey = new SymmetricVerificationKey();
            template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();
            template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);

            return TokenRestrictionTemplateSerializer.Serialize(template);
        }

        static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
        {
            var kdPolicy = _context.ContentKeyAuthorizationPolicies.Where(p => p.Id == key.AuthorizationPolicyId).Single();

            var kdOption = kdPolicy.Options.Single(o => o.KeyDeliveryType == ContentKeyDeliveryType.FairPlay);

            FairPlayConfiguration configFP = JsonConvert.DeserializeObject<FairPlayConfiguration>(kdOption.KeyDeliveryConfiguration);

            // Get the FairPlay license service URL.
            Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.FairPlay);

            // The reason the below code replaces "https://" with "skd://" is because
            // in the IOS player sample code which you obtained in Apple developer account,
            // the player only recognizes a Key URL that starts with skd://.
            // However, if you are using a customized player,
            // you can choose whatever protocol you want.
            // For example, "https".

            Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
            {
                    {AssetDeliveryPolicyConfigurationKey.FairPlayLicenseAcquisitionUrl, acquisitionUrl.ToString().Replace("https://", "skd://")},
                    {AssetDeliveryPolicyConfigurationKey.CommonEncryptionIVForCbcs, configFP.ContentEncryptionIV}
            };

            var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryptionCbcs,
            AssetDeliveryProtocol.HLS,
            assetDeliveryPolicyConfiguration);

            // Add AssetDelivery Policy to the asset
            asset.DeliveryPolicies.Add(assetDeliveryPolicy);

        }


        /// <summary>
        /// Gets the streaming origin locator.
        /// </summary>
        /// <param name="assets"></param>
        /// <returns></returns>
        static public string GetStreamingOriginLocator(IAsset asset)
        {

            // Get a reference to the streaming manifest file from the  
            // collection of files in the asset.

            var assetFile = asset.AssetFiles.LoList().Where(f => f.Name.ToLower().
                         EndsWith(".ism")).
                         FirstOrDefault();

            // Create a 30-day readonly access policy.
            IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

            // Create a locator to the streaming content on an origin.
            ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

            // Create a URL to the manifest file.
            return originLocator.Path + assetFile.Name;
        }

        static private void JobStateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine(string.Format("{0}\n  State: {1}\n  Time: {2}\n\n",
            ((IJob)sender).Name,
            e.CurrentState,
            DateTime.UtcNow.ToString(@"yyyy_M_d__hh_mm_ss")));
        }

        static private byte[] GetRandomBuffer(int length)
        {
            var returnValue = new byte[length];

            using (var rng =
            new System.Security.Cryptography.RNGCryptoServiceProvider())
            {
                rng.GetBytes(returnValue);
            }

            return returnValue;
        }
    }
}
```

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="next-steps-media-services-learning-paths"></a>Étapes suivantes : Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
