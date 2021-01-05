---
title: Charger des fichiers dans un compte Azure Media Services à partir d’Azure StorSimple | Microsoft Docs
description: Cet article donne une brève vue d’ensemble d’Azure StorSimple Data Manager. L’article contient également des liens vers des didacticiels qui expliquent comment extraire des données de StorSimple et les charger en tant que ressources dans un compte Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 1dd09328-262b-43ef-8099-73241b49a925
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 26a3f3ff600de4418ccf6f6b09dea4b091678065
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89265726"
---
# <a name="upload-files-into-an-azure-media-services-account-from-azure-storsimple"></a>Charger des fichiers dans un compte Azure Media Services à partir d’Azure StorSimple 

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).
>
> 
> Azure StorSimple Data Manager est actuellement disponible en version préliminaire privée. 
> 

## <a name="overview"></a>Vue d’ensemble

Dans Media Services, vous téléchargez vos fichiers numériques dans une ressource. L’élément multimédia peut contenir des fichiers vidéo, des fichiers audio, des images, des collections de miniatures, des pistes textuelles et des légendes (ainsi que les métadonnées concernant ces fichiers). Une fois les fichiers téléchargés, votre contenu est stocké en toute sécurité dans le cloud et peut faire l’objet d’un traitement et d’une diffusion en continu.

[Azure StorSimple](../../storsimple/index.yml) utilise le stockage cloud pour étendre la solution sur site et hiérarchise automatiquement les données sur le stockage local et le stockage cloud. L’appareil StorSimple déduplique et compresse les données avant de les envoyer vers le cloud. Il améliore considérablement l’efficacité d’envoi de fichiers volumineux vers le cloud. Le service [StorSimple Data Manager](../../storsimple/storsimple-data-manager-overview.md) fournit des API vous permettant d’extraire des données à partir de StorSimple et de les présenter sous la forme de ressources AMS.

## <a name="get-started"></a>Bien démarrer

1. [Créer un compte Media Services](media-services-portal-create-account.md) dans lequel vous souhaitez transférer les ressources.
2. Inscrivez-vous à la version préliminaire de Data Manager, comme décrit dans l’article [StorSimple Data Manager](../../storsimple/storsimple-data-manager-overview.md).
3. Créez un compte StorSimple Data Manager.
4. Créez un travail de transformation de données qui extrait des données d’un appareil StorSimple et les transfère vers un compte AMS en tant que ressources. 

    Une file d’attente de stockage est créée dès le début d’exécution du travail. Cette file d’attente est renseignée avec des messages sur les objets blob transformés, au fur et à mesure de leur mise à disposition. Le nom de cette file d’attente est identique à celui de la définition du travail. Vous pouvez utiliser cette file d’attente pour déterminer quand une ressource est prête et appeler l’opération Media Services souhaitée pour l’exécuter sur celle-ci. Par exemple, vous pouvez utiliser cette file d’attente pour déclencher une fonction Azure qui contient le code Media Services nécessaire.

## <a name="see-also"></a>Voir aussi

[Utiliser le Kit SDK .NET pour déclencher des travaux dans le Data Manager](../../storsimple/storsimple-data-manager-dotnet-jobs.md)

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez désormais encoder vos éléments multimédias téléchargés. Pour plus d'informations, consultez [Encode an asset using Media Encoder Standard with the Azure portal (Encoder un élément multimédia à l’aide de Media Encoder Standard avec le portail Azure)](media-services-portal-encode.md).
