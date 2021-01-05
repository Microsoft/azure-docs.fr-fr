---
title: Vue d’ensemble du modèle de licence Widevine | Microsoft Docs
description: Cette rubrique donne un aperçu d’un modèle de licence Widevine utilisé pour configurer des licences Widevine.
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 0e6f1f05-7ed6-4ed6-82a0-0cc2182b075a
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 7bdffa607a1cbe47b940590d19f6140238d31bf0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89266491"
---
# <a name="widevine-license-template-overview"></a>Vue d’ensemble du modèle de licence Widevine

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Vous pouvez utiliser Azure Media Services pour configurer et demander des licences Google Widevine. Quand le lecteur tente de lire votre contenu Widevine protégé, une demande est envoyée au service de remise de licence pour obtenir une licence. Si le service de licence approuve la demande, le service émet la licence. Elle est envoyée au client et utilisée pour déchiffrer et lire le contenu spécifié.

Une demande de licence Widevine se présente sous forme de message JSON.  

>[!NOTE]
> Vous pouvez créer un message vide sans valeur, simplement « {} ». Dans ce cas, un modèle de licence est créé avec les valeurs par défaut. Les valeurs par défaut fonctionnent pour la plupart des cas. Les scénarios de remise de licence Microsoft doivent toujours utiliser les valeurs par défaut. Si vous devez définir les valeurs « provider » et « content_id », un fournisseur doit correspondre aux informations d’identification Widevine.

```json
{  
   "payload": "<license challenge>",
   "content_id": "<content id>" 
   "provider": "<provider>"
   "allowed_track_types": "<types>",
   "content_key_specs": [  
      {  
         "track_type": "<track type 1>"
      },
      {  
         "track_type": "<track type 2>"
      },
      …
   ],
   "policy_overrides": {  
      "can_play": <can play>,
      "can persist": <can persist>,
      "can_renew": <can renew>,
      "rental_duration_seconds": <rental duration>,
      "playback_duration_seconds": <playback duration>,
      "license_duration_seconds": <license duration>,
      "renewal_recovery_duration_seconds": <renewal recovery duration>,
      "renewal_server_url": "<renewal server url>",
      "renewal_delay_seconds": <renewal delay>,
      "renewal_retry_interval_seconds": <renewal retry interval>,
      "renew_with_usage": <renew with usage>
   }
}
```

## <a name="json-message"></a>Message JSON
| Nom | Valeur | Description |
| --- | --- | --- |
| payload |Chaîne codée en Base64 |La demande de licence envoyée par un client. |
| content_id |Chaîne codée en Base64 |Identificateur utilisé afin de dériver l’ID de clé et la clé de contenu pour chaque content_key_specs.track_type. |
| provider |string |Utilisé pour rechercher les stratégies et les clés de contenu. Si la remise de clé Microsoft est utilisée pour la remise de licence Widevine, ce paramètre est ignoré. |
| policy_name |string |Nom d'une stratégie précédemment enregistrée. facultatif. |
| allowed_track_types |enum |SD_ONLY ou SD_HD. Contrôle les clés de contenu incluses dans une licence. |
| content_key_specs |Tableau de structures JSON (voir la section « Spécifications de clé de contenu »).  |Un contrôle plus fin sur les clés de contenu à retourner. Pour plus d’informations, consultez la section « Spécifications de clé de contenu ». Une seule valeur allowed_track_types et content_key_specs peut être spécifiée. |
| use_policy_overrides_exclusively |Booléen, true ou false |Utiliser les attributs de la stratégie spécifiés par policy_overrides, et ignorer toutes les stratégies stockées précédemment. |
| policy_overrides |Structure JSON (voir la section « Remplacements de stratégies »). |Paramètres de stratégie pour cette licence.  Si cette ressource comporte une stratégie prédéfinie, les valeurs spécifiées sont utilisées. |
| session_init |Structure JSON (voir « Initialisation de la session »). |Des données facultatives sont passées à la licence. |
| parse_only |Booléen, true ou false |La demande de licence est analysée, mais aucune licence n’est émise. Toutefois, les valeurs de la demande de licence sont retournées dans la réponse. |

## <a name="content-key-specs"></a>Spécifications de clé de contenu
S’il existe une stratégie préexistante, il est inutile de spécifier des valeurs dans la spécification de clé de contenu. La stratégie préexistante associée à ce contenu est utilisée pour déterminer la protection de sortie, telle que HDCP (High-bandwidth Digital Content Protection) et CGMS (Copy General Management System). Si aucune stratégie préexistante n’est inscrite auprès du serveur de licences Widevine, le fournisseur de contenu peut injecter les valeurs dans la demande de licence.   

Chaque valeur content_key_specs doit être spécifiée pour toutes les pistes, quelle que soit l’option use_policy_overrides_exclusively. 

| Nom | Valeur | Description |
| --- | --- | --- |
| content_key_specs. track_type |string |Un nom de type de piste. Si la valeur content_key_specs est spécifiée dans la demande de licence, assurez-vous de spécifier tous les types de pistes de façon explicite. Dans le cas contraire, vous serez confronté à un échec de lecture des 10 dernières secondes. |
| content_key_specs  <br/> security_level |uint32 |Définit la configuration requise de robustesse du client pour la lecture. <br/> - Chiffrement whitebox logiciel requis. <br/> - Chiffrement logiciel et décodeur masqué requis. <br/> - Le matériel de clé et les opérations de chiffrement doivent être effectués dans un environnement d’exécution approuvé soutenu par le matériel. <br/> - Le chiffrement et le décodage du contenu doivent être effectués dans un environnement d’exécution approuvé soutenu par le matériel.  <br/> - Le chiffrement, le décodage et le traitement du support (compressé et décompressé) doivent être gérés dans un environnement d’exécution approuvé soutenu par le matériel. |
| content_key_specs <br/> required_output_protection.hdc |string : HDCP_NONE, HDCP_V1 ou HDCP_V2 |Indique si HDCP est obligatoire. |
| content_key_specs <br/>key |Chaîne<br/>encodée Base64 |Clé de contenu à utiliser pour cette piste. Si spécifié, track_type ou key_id est requis. Le fournisseur de contenu peut utiliser cette option pour injecter la clé de contenu de cette piste, au lieu de laisser le serveur de licences Widevine générer ou rechercher une clé. |
| content_key_specs.key_id |Chaîne binaire codée en Base64, 16 octets |Identificateur unique pour la clé. |

## <a name="policy-overrides"></a>Remplacements de stratégies
| Nom | Valeur | Description |
| --- | --- | --- |
| policy_overrides. can_play |Booléen, true ou false |Indique que la lecture du contenu est autorisée. La valeur par défaut est false. |
| policy_overrides. can_persist |Booléen, true ou false |Indique que la licence peut être rendue persistante dans le stockage non volatile pour une utilisation hors connexion. La valeur par défaut est false. |
| policy_overrides. can_renew |Booléen, true ou false |Indique que le renouvellement de cette licence est autorisé. Si la valeur est true, la durée de la licence peut être étendue par pulsation. La valeur par défaut est false. |
| policy_overrides. license_duration_seconds |int64 |Indique la fenêtre de temps pour cette licence spécifique. La valeur 0 indique qu'il n'existe aucune limite de durée. La valeur par défaut est 0. |
| policy_overrides. rental_duration_seconds |int64 |Indique la fenêtre de temps où la lecture est autorisée. La valeur 0 indique qu'il n'existe aucune limite de durée. La valeur par défaut est 0. |
| policy_overrides. playback_duration_seconds |int64 |Fenêtre d’affichage du temps une fois la lecture démarrée dans la durée de la licence. La valeur 0 indique qu'il n'existe aucune limite de durée. La valeur par défaut est 0. |
| policy_overrides. renewal_server_url |string |Toutes les demandes de pulsation (renouvellement) de cette licence sont dirigées vers l’URL spécifiée. Ce champ est utilisé uniquement si can_renew a la valeur true. |
| policy_overrides. renewal_delay_seconds |int64 |Nombre de secondes après license_start_time avant la première tentative de renouvellement. Ce champ est utilisé uniquement si can_renew a la valeur true. La valeur par défaut est 0. |
| policy_overrides. renewal_retry_interval_seconds |int64 |Spécifie le délai en secondes entre les demandes de renouvellement de licence suivantes, en cas d'échec. Ce champ est utilisé uniquement si can_renew a la valeur true. |
| policy_overrides. renewal_recovery_duration_seconds |int64 |La fenêtre de temps pendant laquelle la lecture peut se poursuivre pendant la tentative de renouvellement, malgré l’échec en raison de problèmes de backend avec le serveur de licences. La valeur 0 indique qu'il n'existe aucune limite de durée. Ce champ est utilisé uniquement si can_renew a la valeur true. |
| policy_overrides. renew_with_usage |Booléen, true ou false |Indique que la licence est envoyée pour renouvellement quand l’utilisation commence. Ce champ est utilisé uniquement si can_renew a la valeur true. |

## <a name="session-initialization"></a>Initialisation de la session
| Nom | Valeur | Description |
| --- | --- | --- |
| provider_session_token |Chaîne codée en Base64 |Ce jeton de session est repassé dans la licence et existe dans les renouvellements suivants. Le jeton de session n’est pas persistant au-delà des sessions. |
| provider_client_token |Chaîne codée en Base64 |Jeton client à renvoyer dans la réponse de la licence. Si la demande de licence contient un jeton client, cette valeur est ignorée. Le jeton client persiste au-delà des sessions de la licence. |
| override_provider_client_token |Booléen, true ou false |Si la valeur est false et si la demande de licence contient un jeton client, utilisez le jeton de la demande même si un jeton client a été spécifié dans cette structure. Si la valeur est true, utilisez toujours le jeton spécifié dans cette structure. |

## <a name="configure-your-widevine-licenses-by-using-net-types"></a>Configurer vos licences Widevine à l’aide des types .NET
Media Services propose des API .NET que vous pouvez utiliser pour configurer vos licences Widevine. 

### <a name="classes-as-defined-in-the-media-services-net-sdk"></a>Les classes sont définies dans le Kit de développement logiciel (SDK) .NET de Media Services
Les classes suivantes sont les définitions de ces types :

```dotnetcli
public class WidevineMessage
{
    public WidevineMessage();

    [JsonProperty(NullValueHandling = NullValueHandling.Ignore)]
    public AllowedTrackTypes? allowed_track_types { get; set; }
    [JsonProperty(NullValueHandling = NullValueHandling.Ignore)]
    public ContentKeySpecs[] content_key_specs { get; set; }
    [JsonProperty(NullValueHandling = NullValueHandling.Ignore)]
    public object policy_overrides { get; set; }
}

[JsonConverter(typeof(StringEnumConverter))]
public enum AllowedTrackTypes
{
    SD_ONLY = 0,
    SD_HD = 1
}
public class ContentKeySpecs
{
    public ContentKeySpecs();

    [JsonProperty(NullValueHandling = NullValueHandling.Ignore)]
    public string key_id { get; set; }
    [JsonProperty(NullValueHandling = NullValueHandling.Ignore)]
    public RequiredOutputProtection required_output_protection { get; set; }
    [JsonProperty(NullValueHandling = NullValueHandling.Ignore)]
    public int? security_level { get; set; }
    [JsonProperty(NullValueHandling = NullValueHandling.Ignore)]
    public string track_type { get; set; }
}

public class RequiredOutputProtection
{
    public RequiredOutputProtection();

    public Hdcp hdcp { get; set; }
}

[JsonConverter(typeof(StringEnumConverter))]
public enum Hdcp
{
    HDCP_NONE = 0,
    HDCP_V1 = 1,
    HDCP_V2 = 2
}
```

### <a name="example"></a>Exemple
L’exemple suivant montre comment utiliser les API .NET pour configurer une licence Widevine simple :

```dotnetcli
private static string ConfigureWidevineLicenseTemplate()
{
    var template = new WidevineMessage
    {
        allowed_track_types = AllowedTrackTypes.SD_HD,
        content_key_specs = new[]
        {
            new ContentKeySpecs
            {
                required_output_protection = new RequiredOutputProtection { hdcp = Hdcp.HDCP_NONE},
                security_level = 1,
                track_type = "SD"
            }
        },
        policy_overrides = new
        {
            can_play = true,
            can_persist = true,
            can_renew = false
        }
    };

    string configuration = JsonConvert.SerializeObject(template);
    return configuration;
}
```

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Voir aussi
[Utiliser le chiffrement commun dynamique PlayReady et/ou Widevine](media-services-protect-with-playready-widevine.md)

