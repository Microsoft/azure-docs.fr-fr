---
title: Configurer des stratégies de protection du contenu à l’aide du portail Azure | Microsoft Docs
description: Cet article explique comment utiliser le portail Azure pour configurer des stratégies de protection du contenu. L’article montre également comment activer le chiffrement dynamique à vos ressources.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 270b3272-7411-40a9-ad42-5acdbba31154
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 061d66f08e79f935c778eb84dd47f71cab1a62d7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89266627"
---
# <a name="configure-content-protection-policies-by-using-the-azure-portal"></a>Configurer des stratégies de protection du contenu à l’aide du portail Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com/pricing/free-trial/).   > Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).
>

 Avec Azure Media Services, vous pouvez sécuriser votre contenu multimédia du moment où il quitte votre ordinateur jusqu’à la remise, en passant par le stockage et le traitement. Vous pouvez utiliser Media Services pour délivrer du contenu chiffré de manière dynamique avec la norme AES (Advanced Encryption Standard) à l’aide de clés de chiffrement 128 bits. Vous pouvez aussi l’utiliser avec le chiffrement commun (CENC) à l’aide de la gestion des droits numériques (DRM) PlayReady et/ou Widevine et Apple FairPlay. 

Media Services fournit un service de remise de licences DRM et de clés en clair AES aux clients autorisés. Vous pouvez utiliser le portail Azure pour créer une seule stratégie d’autorisation de clé/licence pour tous les types de chiffrement.

Cet article explique comment configurer une stratégie de protection du contenu à l’aide du portail. L’article explique également comment appliquer le chiffrement dynamique à vos ressources.

## <a name="start-to-configure-content-protection"></a>Commencer par configurer la protection du contenu
Pour utiliser le portail pour configurer la protection globale du contenu à l’aide de votre compte Media Services, effectuez les étapes suivantes :

1. Dans le [portail](https://portal.azure.com/), sélectionnez votre compte Media Services.

1. Sélectionnez **Paramètres** > **Protection du contenu**.

    ![Protection du contenu](./media/media-services-portal-content-protection/media-services-content-protection001.png)

## <a name="keylicense-authorization-policy"></a>stratégie d’autorisation de clé/licence
Media Services prend en charge plusieurs méthodes d’authentification des utilisateurs effectuant des demandes de clé ou de licence. Vous devez configurer la stratégie d’autorisation de clé de contenu. Votre client doit ensuite satisfaire à la stratégie pour que la licence/clé lui soit remise. La stratégie d’autorisation de la clé de contenu peut comporter une ou plusieurs restrictions d’autorisation, ouvertes ou à jeton.

Vous pouvez utiliser le portail pour créer une seule stratégie d’autorisation de clé/licence pour tous les types de chiffrement.

### <a name="open-authorization"></a>Autorisation ouverte
La restriction ouverte signifie que le système fournit la clé à toute personne effectuant une demande de clé. Cette restriction peut être utile à des fins de test. 

### <a name="token-authorization"></a>Autorisation de jeton
La stratégie de restriction à jeton doit être accompagnée d’un jeton émis par un service d’émission de jeton de sécurité (STS). Media Services prend en charge les jetons aux formats SWT (Simple Web Tokens) et JWT (JSON Web Token). Media Services ne fournit pas de service STS. Vous pouvez créer un service STS personnalisé ou utiliser Azure Access Control Service pour émettre des jetons. Le STS doit être configuré pour créer un jeton signé avec la clé spécifiée et émettre les revendications spécifiées dans la configuration de restriction de jeton. Si le jeton est valide et que les revendications du jeton correspondent à celles configurées pour la clé (ou licence), le service de remise de clé Media Services retourne la clé (ou licence) demandée au client.

Quand vous configurez la stratégie de restriction par jeton, vous devez définir les paramètres de clé de vérification, émetteur et audience principaux. La clé de vérification principale contient le jeton avec lequel la clé a été signée. L’émetteur est le service STS qui émet le jeton. Le public (parfois appelé l’étendue) décrit l’objectif du jeton ou la ressource à laquelle le jeton autorise l’accès. Le service de remise de clé Media Services valide le fait que les valeurs du jeton correspondent aux valeurs du modèle.

![stratégie d’autorisation de clé/licence](./media/media-services-portal-content-protection/media-services-content-protection002.png)

## <a name="playready-license-template"></a>Modèle de licence PlayReady
Le modèle de licence PlayReady définit la fonctionnalité activée sur votre licence PlayReady. Pour plus d’informations sur le modèle de licence PlayReady, consultez la [vue d’ensemble du modèle de licence PlayReady de Media Services](media-services-playready-license-template-overview.md).

### <a name="nonpersistent"></a>Non persistante
Si vous configurez une licence comme étant non persistante, elle est conservée en mémoire uniquement lors de son utilisation par le lecteur.  

![Protection du contenu non persistant](./media/media-services-portal-content-protection/media-services-content-protection003.png)

### <a name="persistent"></a>Persistante
Si vous configurez une licence comme étant persistante, elle est enregistrée dans un stockage persistant sur le client.

![Protection du contenu persistant](./media/media-services-portal-content-protection/media-services-content-protection004.png)

## <a name="widevine-license-template"></a>Modèle de licence Widevine
Le modèle de licence Widevine définit la fonctionnalité activée sur vos licences Widevine.

### <a name="basic"></a>De base
Quand vous sélectionnez **De base**, le modèle est créé avec toutes les valeurs par défaut.

### <a name="advanced"></a>Avancé
Pour plus d’informations sur le modèle de droits Widevine, consultez la [vue d’ensemble du modèle de licence Widevine](media-services-widevine-license-template-overview.md).

![Protection avancée du contenu](./media/media-services-portal-content-protection/media-services-content-protection005.png)

## <a name="fairplay-configuration"></a>Configuration de FairPlay
Pour activer le chiffrement FairPlay, sélectionnez **Configuration FairPlay**. Ensuite, sélectionnez le **Certificat d’application** et entrez la **Clé secrète d’application**. Pour plus d’informations sur la configuration et les spécifications de FairPlay, consultez [Protéger votre contenu HLS avec Apple FairPlay ou Microsoft PlayReady](media-services-protect-hls-with-FairPlay.md).

![Configuration de FairPlay](./media/media-services-portal-content-protection/media-services-content-protection006.png)

## <a name="apply-dynamic-encryption-to-your-asset"></a>Appliquer le chiffrement dynamique à votre ressource
Pour tirer parti du chiffrement dynamique, encodez votre fichier source en un ensemble de fichiers MP4 à débit adaptatif.

### <a name="select-an-asset-that-you-want-to-encrypt"></a>Sélectionner une ressource que vous souhaitez chiffrer
Pour afficher touts vos éléments multimédias, sélectionnez **Paramètres** > **Éléments multimédias**.

![Option de ressources](./media/media-services-portal-content-protection/media-services-content-protection007.png)

### <a name="encrypt-with-aes-or-drm"></a>Chiffrer avec AES ou DRM
Quand vous sélectionnez **Chiffrer** pour une ressource, deux choix s’offrent à vous : **AES** et **DRM**. 

#### <a name="aes"></a>AES
Le chiffrement de clé en clair AES est activé sur tous les protocoles de streaming : Smooth Streaming, HLS et MPEG-DASH.

![Configuration du chiffrement](./media/media-services-portal-content-protection/media-services-content-protection008.png)

#### <a name="drm"></a>DRM
1. Une fois que vous avez sélectionné **DRM**, vous voyez différentes stratégies de protection du contenu (qui doivent être configurées à ce stade) et un ensemble de protocoles de streaming :

    a. **PlayReady et Widevine avec MPEG-DASH** : chiffre dynamiquement votre flux MPEG-DASH avec les DRM PlayReady et Widevine.

    b. **PlayReady et Widevine avec MPEG-DASH+ FairPlay avec HLS** : chiffre dynamiquement votre flux MPEG-DASH avec les DRM PlayReady et Widevine. Cette option chiffre également vos flux HLS avec FairPlay.

    c. **PlayReady uniquement avec Smooth Streaming, HLS et MPEG-DASH** : chiffre dynamiquement les flux Smooth Streaming, HLS et MPEG-DASH avec la DRM PlayReady.

    d. **Widevine uniquement avec MPEG-DASH** : chiffre dynamiquement votre flux MPEG-DASH avec la DRM Widevine.
    
    e. **FairPlay uniquement avec HLS** : chiffre dynamiquement votre flux HLS avec FairPlay.

1. Pour activer le chiffrement FairPlay, dans le panneau **Paramètres globaux de protection du contenu**, sélectionnez **Configuration FairPlay**. Ensuite, sélectionnez le **Certificat d’application** et entrez la **Clé secrète d’application**.

    ![Type de chiffrement](./media/media-services-portal-content-protection/media-services-content-protection009.png)

1. Une fois le chiffrement sélectionné, sélectionnez **Appliquer**.

>[!NOTE] 
>Si vous envisagez de lire un flux HLS chiffré par AES dans Safari, consultez le billet de blog [HLS chiffré dans Safari](https://azure.microsoft.com/blog/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/).

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="next-steps"></a>Étapes suivantes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
