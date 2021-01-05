---
title: Codes d’erreur d’Azure Media Services | Microsoft Docs
description: Il se peut que vous receviez du service des codes d’erreur HTTP liés à des problèmes tels que des jetons d’authentification qui expirent, pour des actions qui ne sont pas prises en charge dans Media Services. Cet article fournit une vue d’ensemble des codes d’erreur de l’API Azure Media Services v2.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: d3a62a64-7608-4b17-8667-479b26ba0d6c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: c30808a46b5cecfaf2e761b3f7a611020cd2cdfb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89263669"
---
# <a name="azure-media-services-error-codes"></a>Codes d’erreur d’Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

lors de l’utilisation de Microsoft Azure Media Services, il se peut que vous receviez du service des codes d’erreur HTTP liés à problèmes tels que des jetons d’authentification qui expirent pour des actions qui ne sont pas prises en charge dans Media Services. Vous trouverez ci-dessous la liste des **codes d’erreur HTTP** qui peuvent être retournés par Media Services, ainsi que leurs causes possibles.  

## <a name="400-bad-request"></a>400 Demande incorrecte
La demande contient des informations non valides et est rejetée pour l’une des raisons suivantes :

* La version spécifiée de l’API n’est pas prise en charge. Pour la version la plus récente, voir [Configuration pour le développement de l’API REST Media Services](media-services-rest-how-to-use.md).
* La version de l’API de Media Services n’est pas spécifiée. Pour en savoir plus sur la façon de spécifier la version de l’API, voir [Référence de l’API REST de Media Services Operations](/rest/api/media/operations/azure-media-services-rest-api-reference).
  
  > [!NOTE]
  > Si vous utilisez les Kits de développement logiciel (SDK) .NET ou Java pour vous connecter à Media Services, la version de l’API est automatiquement spécifiée chaque fois que vous tentez d’effectuer une action sur Media Services.
  > 
  > 
* Une propriété non définie a été spécifiée. Le nom de la propriété figure dans le message d’erreur. Vous ne pouvez spécifier que des propriétés membres d’une entité donnée. Pour obtenir la liste des entités et de leurs propriétés, voir [Référence de l’API REST d’Azure Media Services](/rest/api/media/operations/azure-media-services-rest-api-reference).
* Une valeur de propriété non valide a été spécifiée. Le nom de la propriété figure dans le message d’erreur. Pour connaître les types de propriété valides et leurs valeurs, voir le lien précédent.
* Une valeur de propriété requise est manquante.
* Une partie de l’URL spécifiée contient une valeur incorrecte.
* Une tentative de mise à jour d’une propriété WriteOnce a été effectuée.
* Une tentative a été faite de créer un travail comportant une ressource d’entrée avec un fichier de ressources (AssetFile) non spécifié ou qui n’a pas pu être déterminé.
* Une tentative de mise à jour d’un localisateur de SAP a été effectuée. Des localisateurs de SAP peuvent uniquement être créés ou supprimés. Des localisateurs de streaming peuvent être mis à jour. Pour plus d’informations, voir [Localisateurs](/rest/api/media/operations/locator).
* Une opération ou demande non prises en charge ont été envoyées.

## <a name="401-unauthorized"></a>401 Non autorisé
La demande n’a pas pu être authentifiée (avant d’être autorisée) pour l’une des raisons suivantes :

* En-tête d’authentification absent.
* Valeur d’en-tête d’authentification incorrecte.
  * Le jeton a expiré. 
  * Le jeton contient une signature non valide.

## <a name="403-forbidden"></a>403 Interdit
La demande n’est pas autorisée pour l’une des raisons suivantes :

* Le compte Media Services est introuvable ou a été supprimé.
* Le compte Media Services est désactivé et le type de la demande n’est pas HTTP GET. Les opérations de service renvoient également une réponse 403.
* Le jeton d'authentification ne contient pas les informations d'identification de l'utilisateur : AccountName et/ou SubscriptionId. Vous pouvez trouver ces informations dans l’extension interface utilisateur de Media Services pour votre compte Media Services dans le portail de gestion.
* La ressource est inaccessible.
  
  * Une tentative a été effectuée d’utiliser un processeur multimédia (MediaProcessor) non disponible pour votre compte Media Services.
  * Une tentative a été effectuée de mettre à jour un modèle de travail (JobTemplate) défini par Media Services.
  * Une tentative a été effectuée de remplacer un autre localisateur du compte Media Services.
  * Une tentative a été effectuée de remplacer un autre clé de contenu (ContentKey) du compte Media Services.
* La ressource n’a pas pu être créée en raison d’un quota de service atteint pour le compte Media Services. Pour plus d’informations sur les quotas du service, voir [Quotas et limitations](media-services-quotas-and-limitations.md).

## <a name="404-not-found"></a>404 Introuvable
La demande n’est pas autorisée sur une ressource pour l’une des raisons suivantes :

* Une tentative a été effectuée de mettre à jour une entité inexistante.
* Une tentative a été effectuée de supprimer une entité inexistante.
* Une tentative a été effectuée de créer une entité liant à une entité inexistante.
* Une tentative a été effectuée d’obtenir (GET) une entité inexistante.
* Une tentative a été effectuée de spécifier un compte de stockage non associé au compte Media Services.  

## <a name="409-conflict"></a>409 Conflit
La demande n’est pas autorisée pour l’une des raisons suivantes :

* Plusieurs fichiers de ressources (AssetFile) portent le nom spécifié dans la ressource (Asset).
* Une tentative a été effectuée de créer un deuxième fichier de ressources (AssetFile) principal dans la ressource (Asset).
* Une tentative a été effectuée de créer une clé de contenu (ContentKey) avec l’ID spécifié qui est déjà utilisé.
* Une tentative a été effectuée de créer un localisateur avec l’ID spécifié qui est déjà utilisé.
* Plusieurs fichiers IngestManifestFile portent le nom spécifié dans le manifeste IngestManifest.
* Une tentative a été effectuée de lier un deuxième clé de contenu (ContentKey) de chiffrement de stockage à la ressource (Asset) chiffrée pour stockage.
* Une tentative a été effectuée de lier la même clé de contenu (ContentKey) à la ressource (Asset).
* Une tentative a été effectuée de créer un localisateur pour une ressource (Asset) dont conteneur de stockage est manquant ou n’est plus associé à la ressource.
* Une tentative a été effectuée de créer un localisateur pour une ressource (Asset) qui a déjà 5 localisateurs en cours d’utilisation (le Stockage Azure applique une limite de cinq stratégies d’accès partagé par conteneur de stockage).
* Le compte de stockage de liaison d’une ressource (Asset) à une ressource IngestManifestAsset n’est pas le même que le compte de stockage utilisé par le manifeste IngestManifest parent.  

## <a name="500-internal-server-error"></a>500 Erreur interne du serveur
Lors du traitement de la demande, Media Services rencontre une erreur qui empêche la poursuite du traitement. Cela peut être dû à l’une des raisons suivantes :

* La création d’une ressource ou d’une tâche échoue parce que les informations de quota de service du compte Media Services sont temporairement indisponibles.
* La création d’un conteneur de stockage d’objets blob Asset ou IngestManifest échoue parce que les informations de compte de stockage du compte sont temporairement indisponibles.
* Autre erreur inattendue.

## <a name="503-service-unavailable"></a>503 Service indisponible
Le serveur est actuellement incapable de recevoir des demandes. Cette erreur peut résulter d’un excès de demandes adressées au service. Le mécanisme de limitation de Media Services restreint l'utilisation des ressources pour les applications qui recourent de manière excessive au service.

> [!NOTE]
> Examinez le message d’erreur et la chaîne de code d’erreur pour obtenir des informations plus détaillées sur la raison pour laquelle vous avez reçu l’erreur 503. Cette erreur n’indique pas toujours une limitation.
> 
> 

Les descriptions d’état possibles sont les suivantes :

* « Le serveur est occupé. Des exécutions précédentes de ce type de requête ont pris plus de {0} secondes. »
* « Le serveur est occupé. Plus de {0} demandes par seconde peuvent être limitées. »
* « Le serveur est occupé. Plus de {0} demandes toutes les {1} secondes peuvent être limitées. »

Pour gérer cette erreur, nous recommandons d’utiliser une logique de nouvelle tentative d’interruption exponentielle. Cela implique d’utiliser des attentes de plus en plus longues entre nouvelles tentatives suite à des réponses d’erreur consécutives.  Pour plus d’informations, voir [What Does the Transient Fault Handling Application Block Do?](/previous-versions/msp-n-p/hh680905(v=pandp.50)) (Que fait le bloc applicatif de gestion des erreurs temporaires ?).

> [!NOTE]
> Si vous utilisez le [Kit de développement logiciel (SDK) Azure Media Services pour .Net](https://github.com/Azure/azure-sdk-for-media-services/tree/master), la logique de nouvelle tentative pour l’erreur 503 a été implémentée par le Kit de développement logiciel.  
> 
> 

## <a name="see-also"></a>Voir aussi
[Codes d’erreur de gestion de Media Services](/rest/api/media/)

## <a name="next-steps"></a>Étapes suivantes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
