---
title: Utilisation d’Axinom pour fournir des licences Widevine à Azure Media Services | Microsoft Docs
description: Cet article décrit comment vous pouvez utiliser Azure Media Services (AMS) pour fournir un flux chiffré dynamiquement par AMS avec des DRM PlayReady et Widevine. La licence PlayReady provient du serveur de licences Media Services PlayReady et la licence Widevine est délivrée par le serveur de licences Axinom.
services: media-services
documentationcenter: ''
author: willzhan
manager: femila
editor: ''
ms.assetid: 9c93fa4e-b4da-4774-ab6d-8b12b371631d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: willzhan
ms.reviewer: Mingfeiy;rajputam;Juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 1f486bfe7cb96e81cef2da64552be5ddfd87990b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89264689"
---
# <a name="using-axinom-to-deliver-widevine-licenses-to-azure-media-services"></a>Utilisation d’Axinom pour fournir des licences Widevine à Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [castLabs](media-services-castlabs-integration.md)
> * [Axinom](media-services-axinom-integration.md)
> 
> 

## <a name="overview"></a>Vue d’ensemble
Azure Media Services (AMS) a ajouté la protection dynamique Google Widevine (voir le [blog de Mingfei](https://azure.microsoft.com/blog/azure-media-services-adds-google-widevine-packaging-for-delivering-multi-drm-stream/) pour plus de détails). En outre, Azure Media Player (AMP) a ajouté la prise en charge de Widevine (voir [Document AMP](https://amp.azure.net/libs/amp/latest/docs/) pour plus d’informations). Il s’agit d’un succès majeur en matière de diffusion en continu de contenu DASH protégé par CENC avec DRM multi-natif (PlayReady et Widevine) sur les navigateurs modernes équipés de MSE et EME.

En commençant par le kit de développement logiciel (SDK) Media Services .NET version 3.5.2, Media Services vous permet de configurer le modèle de licence Widevine et d’obtenir des licences Widevine. Vous pouvez également utiliser les partenaires AMS suivants pour vous aider à distribuer des licences Widevine : [Axinom](https://www.axinom.com), [EZDRM](https://ezdrm.com/), [castLabs](https://castlabs.com/company/partners/azure/).

Cet article décrit comment intégrer et tester le serveur de licences Widevine géré par Axinom. Il aborde plus précisément :  

* la configuration d’un chiffrement commun dynamique avec DRM multiples (PlayReady et Widevine) avec les URL d’acquisition de licences correspondantes ;
* la génération d’un jeton JWT afin de répondre aux exigences du serveur de licences ,
* Le développement d’une application Azure Media Player gérant l’acquisition de licence avec l’authentification de jetons JWT

Le système complet et la clé de flux de contenu, l’ID de clé, l’amorce de clé, le jeton JTW et ses revendications sont illustrés dans le schéma qui suit :

![DASH et CENC](./media/media-services-axinom-integration/media-services-axinom1.png)

## <a name="content-protection"></a>Protection du contenu
Pour configurer la stratégie de protection dynamique et de distribution de clés, consultez le blog de Mingfei : [Configuration de l’association de packaging Widevine avec Azure Media Services](https://mingfeiy.com/how-to-configure-widevine-packaging-with-azure-media-services).

Vous pouvez configurer la protection CENC dynamique avec DRM multiples pour la diffusion DASH présentant les deux éléments suivants :

1. Protection PlayReady pour Microsoft Edge et IE11, qui peut présenter une restriction en matière d’autorisation de jeton. La stratégie de restriction à jeton doit être accompagnée d’un jeton émis par un service de jeton sécurisé (STS) comme Azure Active Directory ;
2. Widevine protection pour Chrome, qui peut exiger l’authentification des jetons avec le jeton émis par un autre STS. 

Consultez la rubrique [Génération de jetons JWT](media-services-axinom-integration.md#jwt-token-generation) pour savoir pourquoi Azure Active Directory ne peut pas servir de service STS pour un serveur de licences Widevine d’Axinom.

### <a name="considerations"></a>Considérations
1. Vous devez utiliser l’amorce de clé Axinom spécifiée (8888000000000000000000000000000000000000) et l’ID de clé généré ou sélectionné pour configurer le service de distribution de clé. Le serveur de licences Axinom émet toutes les licences contenant des clés de contenu sur la base de la même amorce de clé, qui est valide à la fois pour les tests et la production.
2. URL d’acquisition de licence Widevine de test : [https://drm-widevine-licensing.axtest.net/AcquireLicense](https://drm-widevine-licensing.axtest.net/AcquireLicense). HTTP et HTTS sont tous les deux autorisés.

## <a name="azure-media-player-preparation"></a>Préparation d’Azure Media Player
AMP 1.4.0 prend en charge la lecture de contenu AMS qui est empaqueté dynamiquement avec les DRM de PlayReady et Widevine.
Si le serveur de licences Widevine n’exige pas l’authentification de jeton, vous n’avez besoin de rien d’autre pour tester un contenu DASH protégé par Widevine. Par exemple, l’équipe AMP fournit un [exemple](https://amp.azure.net/libs/amp/latest/samples/dynamic_multiDRM_PlayReadyWidevineFairPlay_notoken.html) simple, dans lequel vous pouvez voir qu’il fonctionne sous Microsoft Edge et IE11 avec PlayReady et Chrome avec Widevine.
Le serveur de licences Widevine fourni par Axinom requiert l’authentification des jetons JWT. Le jeton JWT doit être soumis avec la demande de licence par le biais d’un en-tête HTTP « X-AxDRM-Message ». Pour ce faire, vous devez ajouter le code javascript suivant dans la page web d’hébergement AMP avant de définir la source :

```html
<script>AzureHtml5JS.KeySystem.WidevineCustomAuthorizationHeader = "X-AxDRM-Message"</script>
```

Le reste du code AMP est en API AMP standard, comme pour le document AMP [suivant](https://amp.azure.net/libs/amp/latest/docs/).

Le code javascript qui permet de configurer l’en-tête d’autorisation personnalisé obéit toujours à une approche à court terme, et ce, avant que l’approche officielle AMP à long terme soit émise.

## <a name="jwt-token-generation"></a>Génération de jetons JWT
Le serveur de licences Axinom Widevine fourni par Axinom requiert l’authentification de jeton JWT. De plus, une des revendications du jeton JWT est un type d’objet complexe et non un type de données primitif.

Malheureusement, Azure AD ne peut émettre que des jetons JWT avec des types primitifs. De même, API .NET Framework (System.IdentityModel.Tokens.SecurityTokenHandler et JwtPayload) permet d’entrer seulement des revendications avec type d’objet complexe. Toutefois, les revendications sont toujours sérialisées en tant que chaîne. Par conséquent, nous ne pouvons utiliser aucun des deux pour générer le jeton JWT pour la demande de licence Widevine.

Le [package NuGet JWT](https://www.nuget.org/packages/JWT) de John Sheehan répond à ces besoins. Nous allons donc l’utiliser.

Vous trouverez ci-dessous le code permettant de générer un jeton JWT avec les revendications nécessaires, comme l’exige le serveur de licences Axinom Widevine pour le test :

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.IdentityModel.Tokens;
using System.IdentityModel.Protocols.WSTrust;
using System.Security.Claims;

namespace OpenIdConnectWeb.Utils
{
    public class JwtUtils
    {
        //using John Sheehan's NuGet JWT library: https://www.nuget.org/packages/JWT/
        public static string CreateJwtSheehan(string symmetricKeyHex, string key_id)
        {
            byte[] symmetricKey = ConvertHexStringToByteArray(symmetricKeyHex);  //hex string to byte[] Note: Note that the key is a hex string, however it must be treated as a series of bytes not a string when encoding.

            var payload = new Dictionary<string, object>()
            {
                { "version", 1 },
                { "com_key_id", System.Configuration.ConfigurationManager.AppSettings["ax:com_key_id"] },
                { "message", new { type = "entitlement_message", key_ids = new string[] { key_id } }  }
            };

            string token = JWT.JsonWebToken.Encode(payload, symmetricKey, JWT.JwtHashAlgorithm.HS256);

            return token;
        }

        //convert hex string to byte[]
        public static byte[] ConvertHexStringToByteArray(string hexString)
        {
            if (hexString.Length % 2 != 0)
            {
                throw new ArgumentException(String.Format(System.Globalization.CultureInfo.InvariantCulture, "The binary key cannot have an odd number of digits: {0}", hexString));
            }

            byte[] HexAsBytes = new byte[hexString.Length / 2];
            for (int index = 0; index < HexAsBytes.Length; index++)
            {
                string byteValue = hexString.Substring(index * 2, 2);
                HexAsBytes[index] = byte.Parse(byteValue, System.Globalization.NumberStyles.HexNumber, System.Globalization.CultureInfo.InvariantCulture);
            }

            return HexAsBytes;
        }

    }  

}  
```

Serveur de licences Axinom Widevine

```xml
<add key="ax:laurl" value="https://drm-widevine-licensing.axtest.net/AcquireLicense" />
<add key="ax:com_key_id" value="69e54088-e9e0-4530-8c1a-1eb6dcd0d14e" />
<add key="ax:com_key" value="4861292d027e269791093327e62ceefdbea489a4c7e5a4974cc904b840fd7c0f" />
<add key="ax:keyseed" value="8888000000000000000000000000000000000000" />
```

### <a name="considerations"></a>Considérations
1. Bien que le service de remise de licence PlayReady AMS exige que la chaîne de caractères « Bearer= » précède un jeton d’authentification, le serveur de licences Axinom Widevine ne l’utilise pas.
2. C’est la clé de communication Axinom qui est utilisée comme clé de connexion. La clé est une chaîne hexadécimale, mais elle doit être traitée comme une série d’octets et non comme une chaîne lors du codage. Pour ce faire, on utilise la méthode ConvertHexStringToByteArray.

## <a name="retrieving-key-id"></a>Récupération de l’ID de clé
Vous avez peut-être remarqué que dans le code permettant de générer un jeton JWT, un ID de clé est exigé. Étant donné que le jeton JWT doit être prêt avant le chargement du lecteur AMP, l’ID de clé doit être récupéré pour générer le jeton JWT.

Bien sûr, il existe plusieurs manières de se procurer l’ID de clé. Par exemple, il est possible de stocker un ID de clé avec les métadonnées de contenu dans une base de données. Il est également possible d’extraire un ID de clé d’un fichier DASH MPD (Media Presentation Description). Le code ci-dessous s’applique à la seconde possibilité.

```csharp
//get key_id from DASH MPD
public static string GetKeyID(string dashUrl)
{
    if (!dashUrl.EndsWith("(format=mpd-time-csf)"))
    {
        dashUrl += "(format=mpd-time-csf)";
    }

    XPathDocument objXPathDocument = new XPathDocument(dashUrl);
    XPathNavigator objXPathNavigator = objXPathDocument.CreateNavigator();
    XmlNamespaceManager objXmlNamespaceManager = new XmlNamespaceManager(objXPathNavigator.NameTable);
    objXmlNamespaceManager.AddNamespace("",     "urn:mpeg:dash:schema:mpd:2011");
    objXmlNamespaceManager.AddNamespace("ns1",  "urn:mpeg:dash:schema:mpd:2011");
    objXmlNamespaceManager.AddNamespace("cenc", "urn:mpeg:cenc:2013");
    objXmlNamespaceManager.AddNamespace("ms",   "urn:microsoft");
    objXmlNamespaceManager.AddNamespace("mspr", "urn:microsoft:playready");
    objXmlNamespaceManager.AddNamespace("xsi",  "https://www.w3.org/2001/XMLSchema-instance");
    objXmlNamespaceManager.PushScope();

    XPathNodeIterator objXPathNodeIterator;
    objXPathNodeIterator = objXPathNavigator.Select("//ns1:MPD/ns1:Period/ns1:AdaptationSet/ns1:ContentProtection[@value='cenc']", objXmlNamespaceManager);

    string key_id = string.Empty;
    if (objXPathNodeIterator.MoveNext())
    {
        key_id = objXPathNodeIterator.Current.GetAttribute("default_KID", "urn:mpeg:cenc:2013");
    }

    return key_id;
}
```

## <a name="summary"></a>Résumé

Grâce à la récente prise en charge de Widevine dans Protection de contenu Azure Media Services et Azure Media Player, nous sommes en mesure d’implémenter la diffusion en continu de DASH + DRM multi-native (PlayReady + Widevine) avec le service de licence PlayReady dans AMS et le serveur de licences Widevine d’Axinom pour les navigateurs modernes suivants :

* Chrome
* Microsoft Edge sous Windows 10
* IE 11 sous Windows 8.1et Windows 10
* Firefox (Bureau) et Safari sur Mac (non iOS) sont également pris en charge via Silverlight et la même URL avec Azure Media Player

Les paramètres suivants sont requis pour la mini-solution qui utilise serveur de licences Axinom Widevine. À l’exception de l’ID de clé, le reste des paramètres sont fournis par Axinom en fonction du programme d’installation du serveur Widevine.

| Paramètre | Utilisation |
| --- | --- |
| ID de clé de communication |Doit être inclus en tant que valeur de revendication « com_key_id » dans le jeton JWT (consultez [cette](media-services-axinom-integration.md#jwt-token-generation) section). |
| Clé de communication |Doit être utilisé comme clé de signature de jeton JWT (consultez [cette](media-services-axinom-integration.md#jwt-token-generation) section). |
| Amorce de clé |Doit être utilisé pour générer la clé de contenu avec un ID de clé de contenu donné (consultez [cette](media-services-axinom-integration.md#content-protection) section). |
| URL d’acquisition de licence Widevine. |Doit être utilisé dans la configuration de la stratégie de remise de l’élément multimédia pour la diffusion en streaming DASH (consultez [cette](media-services-axinom-integration.md#content-protection) section). |
| ID de clé de contenu |Doit être inclus dans le cadre de la valeur de la revendication Entitlement Message du jeton JWT (consultez [cette](media-services-axinom-integration.md#jwt-token-generation) section). |

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

### <a name="acknowledgments"></a>Remerciements
Nous aimerions remercier les personnes suivantes qui ont contribué à la création de ce document : Kristjan Jõgi of Axinom, Mingfei Yan et Amit Rajput.

