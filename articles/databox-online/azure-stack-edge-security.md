---
title: Sécurité Azure Stack Edge Pro | Microsoft Docs
description: Décrit les fonctionnalités de sécurité et de confidentialité qui protègent votre appareil, votre service et vos données Azure Stack Edge Pro en local et dans le cloud.
services: Data Box Edge
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 08/21/2019
ms.author: alkohli
ms.openlocfilehash: b5c5143c7bc92487aeabc3cdc1afea5061466027
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90903158"
---
# <a name="azure-stack-edge-pro-security-and-data-protection"></a>Protection des données et sécurité Azure Stack Edge Pro

La sécurité est une préoccupation majeure pour toute nouvelle technologie, surtout si cette technologie est utilisée avec des données confidentielles ou propriétaires. Azure Stack Edge Pro vous permet de vous assurer que seules des entités autorisées peuvent consulter, modifier ou supprimer vos données.

Cet article décrit les fonctionnalités de sécurité d’Azure Stack Edge Pro qui permettent de protéger chaque composant de la solution et les données qui y sont stockées.

Azure Stack Edge Pro repose sur l’interaction de quatre composants principaux :

- **Service Azure Stack Edge, hébergé dans Azure**. la ressource de gestion qui vous permet de créer la commande de l’appareil, de configurer l’appareil et de suivre la commande jusqu’à son achèvement.
- **Appareil Azure Stack Edge Pro** : appareil de transfert qui vous est livré pour que vous puissiez importer vos données locales dans Azure.
- **Clients/hôtes connectés à l’appareil** : clients dans votre infrastructure qui se connectent à l’appareil Azure Stack Edge Pro et contiennent des données à protéger.
- **Stockage cloud** : emplacement dans la plateforme cloud Azure où les données sont stockées. Il s’agit généralement du compte de stockage lié à la ressource Azure Stack Edge que vous créez.

## <a name="azure-stack-edge-service-protection"></a>Protection du service Azure Stack Edge

Le service Data Box Edge est un service de gestion hébergé dans Azure. Ce service est utilisé pour configurer et gérer l’appareil.

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-service-protection.md)]

## <a name="azure-stack-edge-pro-device-protection"></a>Protection des appareils Azure Stack Edge Pro

L’appareil Azure Stack Edge Pro est un appareil local qui vous permet de transformer vos données en les traitant localement avant de les envoyer à Azure. Votre appareil :

- a besoin d’une clé d’activation pour accéder au service Azure Stack Edge.
- Est protégé à tout moment par un mot de passe d’appareil.
- Est un appareil verrouillé. L’appareil BMC et BIOS sont protégés par mot de passe. Le BIOS est protégé par accès utilisateur limité.
- Le démarrage sécurisé est activé.
- Exécute Windows Defender Device Guard. Device Guard permet d’exécuter uniquement les applications de confiance que vous définissez dans vos stratégies d’intégrité du code.

### <a name="protect-the-device-via-activation-key"></a>Protéger l’appareil via une clé d’activation

Seul un appareil Azure Stack Edge Pro autorisé est autorisé à rejoindre le service Azure Stack Edge que vous créez dans votre abonnement Azure. Pour autoriser un appareil, vous devez utiliser une clé d’activation pour activer l’appareil auprès du service Azure Stack Edge.

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-activation-key.md)]

Pour plus d’informations, consultez [Obtenir une clé d’activation](azure-stack-edge-deploy-prep.md#get-the-activation-key).

### <a name="protect-the-device-via-password"></a>Protéger l’appareil via un mot de passe

Les mots de passe garantissent que seuls les utilisateurs autorisés accèdent à vos données. Les appareils Azure Stack Edge Pro démarrent en état verrouillé.

Vous pouvez :

- Vous connecter à l’interface utilisateur web locale de l’appareil via un navigateur, puis indiquer un mot de passe pour vous connecter à l’appareil.
- Vous connecter à distance à l’interface PowerShell de l’appareil via HTTP. La gestion à distance est désactivée par défaut. Vous pouvez ensuite fournir le mot de passe pour vous connecter à l’appareil. Pour plus d’informations, consultez [Connexion à distance à votre appareil Azure Stack Edge Pro](azure-stack-edge-connect-powershell-interface.md#connect-to-the-powershell-interface).

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-password-best-practices.md)]
- Utiliser l’interface utilisateur web locale pour [modifier le mot de passe](azure-stack-edge-manage-access-power-connectivity-mode.md#manage-device-access). Si vous modifiez le mot de passe, veillez à informer tous les utilisateurs à distance afin qu’ils ne connaissent pas de problème de connexion.

## <a name="protect-your-data"></a>Protéger vos données

Cette section décrit les fonctionnalités de sécurité d’Azure Stack Edge Pro, qui protègent les données en transit et stockées.

### <a name="protect-data-at-rest"></a>Protection des données au repos

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-data-rest.md)]
- Le chiffrement BitLocker XTS-AES de 256 bits est utilisé pour protéger les données locales.


### <a name="protect-data-in-flight"></a>Protection des données à la volée

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-data-flight.md)]

### <a name="protect-data-via-storage-accounts"></a>Protection des données par les comptes de stockage

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-protect-data-storage-accounts.md)]
- Faites tourner puis [synchronisez vos clés de compte de stockage](azure-stack-edge-manage-shares.md#sync-storage-keys) régulièrement pour mieux protéger votre compte de stockage des utilisateurs non autorisés.

## <a name="manage-personal-information"></a>Gestion des informations personnelles

Le service Azure Stack Edge collecte des informations personnelles dans les scénarios suivants :

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-manage-personal-data.md)]

Pour voir la liste des utilisateurs qui ont accès à un partage ou qui peuvent supprimer un partage, suivez les étapes décrites dans [Gérer les partages sur Azure Stack Edge Pro](azure-stack-edge-manage-shares.md).

Pour plus d’informations, consultez la Politique de confidentialité Microsoft dans le [Centre de gestion de la confidentialité](https://www.microsoft.com/trustcenter).

## <a name="next-steps"></a>Étapes suivantes

[Déployer votre appareil Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md)
