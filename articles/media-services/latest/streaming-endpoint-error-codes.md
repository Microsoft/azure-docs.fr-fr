---
title: Erreurs d’origine et de packaging Azure Media Services | Microsoft Docs
description: Cette rubrique décrit les erreurs que vous pouvez recevoir du service (d’origine) de point de terminaison de streaming Azure Media Services.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: error-reference
ms.date: 05/07/2019
ms.author: inhenkel
ms.openlocfilehash: 8442c52052a2016da002d392c6fe0834b3813229
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89295323"
---
# <a name="streaming-endpoint-origin-errors"></a>Erreurs (d’origine) du point de terminaison de streaming

Cette rubrique décrit les erreurs que vous pouvez recevoir du [service de point de terminaison de streaming](streaming-endpoint-concept.md) Azure Media Services.

## <a name="400-bad-request"></a>400 Demande incorrecte

La demande contient des informations non valides et est rejetée avec ces codes d’erreur pour l’une des raisons suivantes :

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_BAD_URL_SYNTAX |0x80890201|Une erreur de format ou de syntaxe d’URL. Les exemples incluent des demandes pour un type non valide, un fragment non valide ou une piste non valide. |
|MPE_ENC_ENCRYPTION_NOT_SPECIFIED_IN_URL |0x8088024C|La demande n’a pas de balise de chiffrement dans l’URL. Les demandes CMAF exigent une balise de chiffrement dans l’URL. D’autres protocoles configurés avec plusieurs types de chiffrement nécessitent également la balise de chiffrement pour éviter les ambiguïtés. |
|MPE_STORAGE_BAD_URL_SYNTAX |0x808900E9|La demande de stockage pour répondre à la demande a échoué avec une erreur de type Demande incorrecte. |

## <a name="403-forbidden"></a>403 Interdit

La demande n’est pas autorisée pour l’une des raisons suivantes :

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_STORAGE_AUTHENTICATION_FAILED |0x808900EA|La demande de stockage pour répondre à la demande a échoué avec une erreur de type Authentification. Cela peut se produire si les clés de stockage ont été permutées et que le service n’a pas pu synchroniser les clés de stockage. <br/><br/>Contactez le support Azure en accédant à [Aide + Support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) dans le portail Azure.|
|MPE_STORAGE_INSUFFICIENT_ACCOUNT_PERMISSIONS |0x808900EB |Erreur d’opération de stockage, l’accès a échoué en raison d’autorisations de compte insuffisantes. |
|MPE_STORAGE_ACCOUNT_IS_DISABLED |0x808900EC |La demande de stockage pour répondre à la demande a échoué, car le compte de stockage est désactivé. |
|MPE_STORAGE_AUTHENTICATION_FAILURE |0x808900F3 |Erreur d’opération de stockage, l’accès a échoué en raison d’erreurs génériques. |
|MPE_OUTPUT_FORMAT_BLOCKED |0x80890207 |Le format de sortie est bloqué en raison de la configuration dans l’élément StreamingPolicy. |
|MPE_ENC_ENCRYPTION_REQUIRED |0x8088021E |Le chiffrement est requis pour le contenu, la stratégie de remise est requise pour le format de sortie. |
|MPE_ENC_ENCRYPTION_NOT_SET_IN_DELIVERY_POLICY |0x8088024D |Le chiffrement n’est pas défini dans les paramètres de stratégie de remise. |

## <a name="404-not-found"></a>404 Introuvable

L’opération tente d’agir sur une ressource qui n’existe plus. Par exemple, la ressource peut déjà avoir été supprimée.

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_EGRESS_TRACK_NOT_FOUND |0x80890209 |La piste demandée est introuvable. |
|MPE_RESOURCE_NOT_FOUND |0x808901F9 |La ressource demandée est introuvable. |
|MPE_UNAUTHORIZED |0x80890244 |L’accès n’est pas autorisé. |
|MPE_EGRESS_TIMESTAMP_NOT_FOUND |0x8089020A |Le timestamp est introuvable. |
|MPE_EGRESS_FILTER_NOT_FOUND |0x8089020C |Le filtre de manifeste dynamique demandé est introuvable. |
|MPE_FRAGMENT_BY_INDEX_NOT_FOUND |0x80890252 |L’index de fragment demandé est au-delà de la plage valide. |
|MPE_LIVE_MEDIA_ENTRIES_NOT_FOUND |0x80890254 |Les entrées de média en direct sont introuvables pour obtenir le tampon moov. |
|MPE_FRAGMENT_TIMESTAMP_NOT_FOUND |0x80890255 |Impossible de trouver le fragment au moment demandé pour une piste particulière.<br/><br/>Le fragment n’est peut-être pas dans le stockage. Essayez une couche différente de la présentation qui peut avoir un fragment. |
|MPE_MANIFEST_MEDIA_ENTRY_NOT_FOUND |0x80890256 |Impossible de trouver l’entrée de média pour la vitesse de transmission demandée dans le manifeste. <br/><br/>Peut-être que le lecteur a demandé une piste vidéo avec une certaine vitesse de transmission qui n’était pas dans le manifeste.|
|MPE_METADATA_NOT_FOUND |0x80890257 |Impossible de trouver certaines métadonnées dans le manifeste ou impossible de trouver le rebasage dans le stockage. |
|MPE_STORAGE_RESOURCE_NOT_FOUND |0x808900ED |Erreur d’opération de stockage, ressource introuvable. |

## <a name="409-conflict"></a>409 Conflit

L’ID fourni pour une ressource sur une opération `PUT` ou `POST` a été prise par une ressource existante. Pour résoudre ce problème, utilisez un autre ID pour la ressource.

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_STORAGE_CONFLICT  |0x808900EE  |Erreur d’opération de stockage, erreur de conflit.  |

## <a name="410"></a>410

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_FILTER_FORCE_END_LEFT_EDGE_CROSSED_DVR_WINDOW|0x80890263|Pour le streaming en direct, lorsque le filtre dont l’élément forceEndTimestamp est défini sur true, le timestamp de départ ou de fin est en dehors de la fenêtre du magnétoscope numérique actuelle.|

## <a name="412-precondition-failure"></a>412 Échec de la précondition

L’opération spécifiait un eTag différent de la version disponible sur le serveur, c’est-à-dire une erreur d’accès concurrentiel optimiste. Effectuez une nouvelle tentative de requête après avoir lu la dernière version de la ressource et mis à jour l’eTag sur la requête.

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_FRAGMENT_NOT_READY |0x80890200 |Le fragment demandé n’est pas prêt.|
|MPE_STORAGE_PRECONDITION_FAILED| 0x808900EF|Erreur d’opération de stockage, un échec de la précondition.|

## <a name="415-unsupported-media-type"></a>415 Type de support non pris en charge

Le format de charge utile envoyé par le client est dans un format non pris en charge.

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_ENC_ALREADY_ENCRYPTED| 0x8088021F| Ne doit pas appliquer le chiffrement sur le contenu déjà chiffré.|
|MPE_ENC_INVALID_INPUT_ENCRYPTION_FORMAT|0x8088021D |Le chiffrement est non valide pour le format d’entrée.|
|MPE_INVALID_ASSET_DELIVERY_POLICY_TYPE|0x8088021C| Type de stratégie de remise non valide.|
|MPE_ENC_MULTIPLE_SAME_DELIVERY_TYPE|0x8088024E |Les paramètres d’origine peuvent être partagés par plusieurs formats de sortie.|
|MPE_FORMAT_NOT_SUPPORTED|0x80890205|Le type ou format de média n’est pas pris en charge. Par exemple, Media Services ne prend pas en charge un nombre de niveaux de qualité supérieur à 64. Dans la balise vidéo FLV, Media Services ne prend pas en charge une trame vidéo avec plusieurs SPS et PPS.|
|MPE_INPUT_FORMAT_NOT_SUPPORTED|0x80890218| Le format d’entrée de ressource demandée n’est pas pris en charge. Media Services prend en charge les formats de téléchargement progressif, MP4 (VoD) et Smooth (direct).|
|MPE_OUTPUT_FORMAT_NOT_SUPPORTED|0x8089020D|Le format de sortie demandé n’est pas pris en charge. Media Services prend en charge les formats de téléchargement progressif, HLS (v3, v4, CMAF), DASH (CSF, CMAF) et Smooth.|
|MPE_ENCRYPTION_NOT_SUPPORTED|0x80890208|Type de chiffrement non pris en charge rencontré.|
|MPE_MEDIA_TYPE_NOT_SUPPORTED|0x8089020E|Le type de média demandé n’est pas pris en charge par le format de sortie. Les types pris en charge sont : vidéo, audio ou sous-titre « SUBT ».|
|MPE_MEDIA_ENCODING_NOT_SUPPORTED|0x8089020F|Le média de ressource source était encodé avec un format de média non compatible avec le format de sortie.|
|MPE_VIDEO_ENCODING_NOT_SUPPORTED|0x80890210|La ressource source était encodée avec un format de vidéo non compatible avec le format de sortie. H.264, AVC, H.265 (HEVC, hev1 ou hvc1) sont pris en charge.|
|MPE_AUDIO_ENCODING_NOT_SUPPORTED|0x80890211|La ressource source était encodée avec un format audio non compatible avec le format de sortie. Les formats audio pris en charge sont AAC, E-AC3 (DD+), Dolby DTS.|
|MPE_SOURCE_PROTECTION_CONVERSION_NOT_SUPPORTED|0x80890212|La ressource protégée source ne peut pas être convertie au format de sortie.|
|MPE_OUTPUT_PROTECTION_FORMAT_NOT_SUPPORTED|0x80890213|Le format de protection n’est pas pris en charge par le format de sortie.|
|MPE_INPUT_PROTECTION_FORMAT_NOT_SUPPORTED|0x80890219|Le format de protection n’est pas pris en charge par le format d’entrée.|
|MPE_INVALID_VIDEO_NAL_UNIT|0x80890231|Unité NAL de vidéo non valide, par exemple, seul la première NAL dans l’exemple peut être de type AUD.|
|MPE_INVALID_NALU_SIZE|0x80890260|Taille d’unité NAL non valide.|
|MPE_INVALID_NALU_LENGTH_FIELD|0x80890261|Valeur de longueur d’unité NAL non valide.|
|MPE_FILTER_INVALID|0x80890236|Filtres de manifeste dynamique non valides.|
|MPE_FILTER_VERSION_INVALID|0x80890237|Versions de filtre non valides ou non prises en charge.|
|MPE_FILTER_TYPE_INVALID|0x80890238|Type de filtre non valide.|
|MPE_FILTER_RANGE_ATTRIBUTE_INVALID|0x80890239|Une plage non valide est spécifiée par le filtre.|
|MPE_FILTER_TRACK_ATTRIBUTE_INVALID|0x8089023A|Un attribut de piste non valide est spécifié par le filtre.|
|MPE_FILTER_PRESENTATION_WINDOW_INVALID|0x8089023B|Une longueur de fenêtre de présentation non valide est spécifiée par le filtre.|
|MPE_FILTER_LIVE_BACKOFF_INVALID|0x8089023C|Une interruption en direct non valide est spécifiée par le filtre.|
|MPE_FILTER_MULTIPLE_SAME_TYPE_FILTERS|0x8089023D|Seul un élément absTimeInHNS est pris en charge dans les filtres hérités.|
|MPE_FILTER_REMOVED_ALL_STREAMS|0x8089023E|Il n’y a plus aucun flux de données après avoir appliqué les filtres.|
|MPE_FILTER_LIVE_BACKOFF_OVER_DVRWINDOW|0x8089023F|L’interruption en direct est au-delà de la fenêtre du magnétoscope numérique.|
|MPE_FILTER_LIVE_BACKOFF_OVER_PRESENTATION_WINDOW|0x80890262|L’interruption en direct est supérieure à la fenêtre de présentation.|
|MPE_FILTER_COMPOSITION_FILTER_COUNT_OVER_LIMIT|0x80890246|Nombre de filtres autorisés maximum par défaut de dix (10) dépassé.|
|MPE_FILTER_COMPOSITION_MULTIPLE_FIRST_QUALITY_OPERATOR_NOT_ALLOWED|0x80890248|Un opérateur de première qualité de vidéo multiple n’est pas autorisé dans les filtres de demandes combinés.|
|MPE_FILTER_FIRST_QUALITY_ATTRIBUTE_INVALID|0x80890249|Le nombre d’attributs de vitesse de transmission de première qualité doit être un (1).|
|MPE_HLS_SEGMENT_TOO_LARGE|0x80890243|La durée du segment HLS doit être inférieure à un tiers de la fenêtre du magnétoscope numérique et de l’interruption HLS.|
|MPE_KEY_FRAME_INTERVAL_TOO_LARGE|0x808901FE|Les durées de fragments doivent être inférieures ou égales à environ 20 secondes, ou les niveaux de qualité d’entrée ne sont pas alignés en termes de temps.|
|MPE_DTS_RESERVEDBOX_EXPECTED|0x80890105|Erreur DTS spécifique, impossible de trouver l’élément ReservedBox quand il doit se trouver dans l’élément DTSSpecficBox pendant l’analyse DTS box.|
|MPE_DTS_INVALID_CHANNEL_COUNT|0x80890106|Erreur DTS spécifique, aucune chaîne trouvée dans l’élément DTSSpecficBox lors de l’analyse DTS box.|
|MPE_DTS_SAMPLETYPE_MISMATCH|0x80890107|Erreur DTS spécifique, exemple d’incompatibilité de type dans l’élément DTSSpecficBox.|
|MPE_DTS_MULTIASSET_DTSH_MISMATCH|0x80890108|Erreur DTS spécifique, « multi-ressource » est défini mais exemple DTSH d’incompatibilité de type.|
|MPE_DTS_INVALID_CORESTREAM_SIZE|0x80890109|Erreur DTS spécifique, taille du flux principal non valide.|
|MPE_DTS_INVALID_SAMPLE_RESOLUTION|0x8089010A|Erreur DTS spécifique, exemple de résolution non valide.|
|MPE_DTS_INVALID_SUBSTREAM_INDEX|0x8089010B|Erreur DTS spécifique, index d’extension de sous-flux non valide.|
|MPE_DTS_INVALID_BLOCK_NUM|0x8089010C|Erreur DTS spécifique, nombre de blocs de sous-flux non valide|
|MPE_DTS_INVALID_SAMPLING_FREQUENCE|0x8089010D|Erreur DTS spécifique, fréquence d’échantillonnage non valide.|
|MPE_DTS_INVALID_REFCLOCKCODE|0x8089010E|Erreur DTS spécifique, le code de l’horloge de référence dans l’extension du sous-flux est non valide.|
|MPE_DTS_INVALID_SPEAKERS_REMAP|0x8089010F|Erreur DTS spécifique, le nombre de haut-parleurs remappés défini est non valide.|

Pour des exemples et articles dédiés au chiffrement, consultez :

- [Concept : protection du contenu](content-protection-overview.md)
- [Concept : stratégies de clé de contenu](content-key-policy-concept.md)
- [Concept : stratégies de diffusion en continu](streaming-policy-concept.md)
- [Exemple : protéger avec le chiffrement AES](protect-with-aes128.md)
- [Exemple : protéger avec DRM](protect-with-drm.md)

Pour obtenir des conseils sur les filtres, consultez :

- [Concept : manifestes dynamiques](filters-dynamic-manifest-overview.md)
- [Concept : filtres](filters-concept.md)
- [Exemple : créer des filtres avec les API REST](filters-dynamic-manifest-rest-howto.md)
- [Exemple : créer des filtres avec .NET](filters-dynamic-manifest-dotnet-howto.md)
- [Exemple : créer des filtres avec CLI](filters-dynamic-manifest-cli-howto.md)

Pour les exemples et articles dédiés au direct, consultez :

- [Concept : vue d’ensemble du streaming en direct](live-streaming-overview.md)
- [Concept : événements en direct et sorties en direct](live-events-outputs-concept.md)
- [Exemple : didacticiel sur le streaming en direct](stream-live-tutorial-with-api.md)

## <a name="416-range-not-satisfiable"></a>416 Plage non satisfaite

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_STORAGE_INVALID_RANGE|0x808900F1|Erreur d’opération de stockage, erreur http 416 renvoyée, plage non valide.|

## <a name="500-internal-server-error"></a>500 Erreur interne du serveur

Lors du traitement de la demande, Media Services rencontre une erreur qui empêche la poursuite du traitement.  

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_STORAGE_SOCKET_TIMEOUT|0x808900F4|Réception et traduction depuis le code d’erreur Winhttp de ERROR_WINHTTP_TIMEOUT (0x00002ee2).|
|MPE_STORAGE_SOCKET_CONNECTION_ERROR|0x808900F5|Réception et traduction depuis le code d’erreur Winhttp de ERROR_WINHTTP_CONNECTION_ERROR (0x00002efe).|
|MPE_STORAGE_SOCKET_NAME_NOT_RESOLVED|0x808900F6|Réception et traduction depuis le code d’erreur Winhttp de ERROR_WINHTTP_NAME_NOT_RESOLVED (0x00002ee7).|
|MPE_STORAGE_INTERNAL_ERROR|0x808900E6|Erreur d’opération de stockage, InternalError générale de l’une des erreurs HTTP 500.|
|MPE_STORAGE_OPERATION_TIMED_OUT|0x808900E7|Erreur d’opération de stockage, OperationTimedOut générale de l’une des erreurs HTTP 500.|
|MPE_STORAGE_FAILURE|0x808900F2|Erreur d’opération de stockage, autres erreurs HTTP 500 que InternalError ou OperationTimedOut.|

## <a name="503-service-unavailable"></a>503 Service indisponible

Le serveur est actuellement incapable de recevoir des demandes. Cette erreur peut résulter d’un excès de demandes adressées au service. Le mécanisme de limitation de Media Services restreint l'utilisation des ressources pour les applications qui recourent de manière excessive au service.

> [!NOTE]
> Examinez le message d’erreur et la chaîne de code d’erreur pour obtenir des informations plus détaillées sur la raison pour laquelle vous avez reçu l’erreur 503. Cette erreur n’indique pas toujours une limitation.
> 

|Code d'erreur|Valeur hexadécimale |Description de l'erreur|
|---|---|---|
|MPE_STORAGE_SERVER_BUSY|0x808900E8|Erreur d’opération de stockage, réception d’erreur 503 de serveur occupé.|

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="see-also"></a>Voir aussi

- [Codes d’erreur d’encodage](/rest/api/media/jobs/get#joberrorcode)
- [Concepts Azure Media Services](concepts-overview.md)
- [Quotas et limites](limits-quotas-constraints.md)

## <a name="next-steps"></a>Étapes suivantes

[Exemple : accéder aux éléments ErrorCode et Message à partir de ApiException avec .NET](configure-connect-dotnet-howto.md#connect-to-the-net-client)
