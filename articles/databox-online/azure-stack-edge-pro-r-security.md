---
title: Sécurité Azure Stack Edge Pro R | Microsoft Docs
description: Décrit les fonctionnalités de sécurité et de confidentialité qui protègent vos appareils Azure Stack Edge Pro R et Azure Stack Edge Mini R, votre service et vos données Azure Stack Edge Pro en local et dans le cloud.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 10/14/2020
ms.author: alkohli
ms.openlocfilehash: f7d81d14ca561e6d4d897994088b2fc01b2c7701
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465501"
---
# <a name="security-and-data-protection-for-azure-stack-edge-pro-r-and-azure-stack-edge-mini-r"></a>Sécurité et protection des données pour Azure Stack Edge Pro R et Azure Stack Edge Mini R

[!INCLUDE [applies-to-r-skus](../../includes/azure-stack-edge-applies-to-r-sku.md)]

La sécurité est une préoccupation majeure pour toute nouvelle technologie, surtout si cette technologie est utilisée avec des données confidentielles ou propriétaires. Azure Stack Edge Pro R et Azure Stack Edge Mini R vous permettent de vous assurer que seules des entités autorisées peuvent consulter, modifier ou supprimer vos données.

Cet article décrit les fonctionnalités de sécurité d’Azure Stack Edge Pro R et Azure Stack Edge Mini R qui permettent de protéger chaque composant de la solution et les données qui y sont stockées.

La solution se compose de quatre éléments principaux qui interagissent les uns avec les autres :

- **Service Azure Stack Edge, hébergé dans Azure (public) ou Azure Government Cloud** : la ressource de gestion qui vous permet de créer la commande de l’appareil, de configurer l’appareil et de suivre la commande jusqu’à son achèvement.
- **Appareil robuste Azure Stack Edge** : l’appareil physique et robuste qui vous est livré pour que vous puissiez importer vos données locales dans Azure (public) ou Azure Government Cloud. L’appareil peut être Azure Stack Edge Pro R ou Azure Stack Edge Mini R.
- **Clients/hôtes connectés à l’appareil** : les clients de votre infrastructure se connectant à l’appareil et contenant des données devant être protégées.
- **Stockage cloud** : emplacement dans la plateforme cloud Azure où les données sont stockées. Il s’agit généralement du compte de stockage lié à la ressource Azure Stack Edge que vous créez.

## <a name="service-protection"></a>Protection des services

Le service Data Box Edge est un service de gestion hébergé dans Azure. Ce service est utilisé pour configurer et gérer l’appareil.

[!INCLUDE [azure-stack-edge-gateway-data-rest](../../includes/azure-stack-edge-gateway-service-protection.md)]

## <a name="device-protection"></a>Protection d’appareil

L’appareil robuste est un appareil local qui vous permet de transformer vos données en les traitant localement, avant de les envoyer vers Azure. Votre appareil :

- a besoin d’une clé d’activation pour accéder au service Azure Stack Edge.
- Est protégé à tout moment par un mot de passe d’appareil.
- Est un appareil verrouillé. Le contrôleur de gestion de la carte de base (BMC) et le BIOS sont protégés par mot de passe. Le BMC est protégé par accès utilisateur limité.
- Active le démarrage sécurisé, qui garantit que l’appareil démarre uniquement à l’aide des logiciels approuvés fournis par Microsoft.
- Exécute le contrôle d’application Windows Defender (WDAC). WDAC vous permet d’exécuter uniquement les applications approuvées que vous définissez dans vos stratégies d’intégrité du code.
- Dispose d’un Module de plateforme sécurisée (TPM) qui effectue des fonctions matérielles et liées à la sécurité. Plus précisément, le TPM gère et protège les secrets et les données qui doivent être rendus persistants sur l’appareil.
- Seuls les ports requis sont ouverts sur l’appareil et tous les autres ports sont bloqués. Pour plus d’informations, consultez la liste des [exigences relatives aux ports pour l’appareil](azure-stack-edge-pro-r-system-requirements.md).
- L’ensemble de l’accès au matériel de l’appareil, ainsi qu’au logiciel, est consigné. 
    - Pour le logiciel de l’appareil, les journaux de pare-feu par défaut sont collectés pour le trafic entrant et sortant à partir de l’appareil. Ces journaux sont regroupés dans le package de support.
    - Pour le matériel de l’appareil, tous les événements de châssis de périphérique, tels que l’ouverture et la fermeture du châssis de l’appareil, sont enregistrés dans l’appareil.

    Pour plus d’informations sur les journaux spécifiques qui contiennent les événements d’intrusion matérielle et logicielle et sur la façon d’obtenir les journaux, accédez à [Collecter des journaux de sécurité avancés](azure-stack-edge-gpu-troubleshoot.md).


### <a name="protect-the-device-via-activation-key"></a>Protéger l’appareil via une clé d’activation

Seul un appareil Azure Stack Edge Pro R et Azure Stack Edge Mini R autorisé est autorisé à rejoindre le service Azure Stack Edge que vous créez dans votre abonnement Azure. Pour autoriser un appareil, vous devez utiliser une clé d’activation pour activer l’appareil auprès du service Azure Stack Edge.

[!INCLUDE [azure-stack-edge-gateway-data-rest](../../includes/azure-stack-edge-gateway-activation-key.md)]

Pour plus d’informations, consultez [Obtenir une clé d’activation](azure-stack-edge-pro-r-deploy-prep.md#get-the-activation-key).

### <a name="protect-the-device-via-password"></a>Protéger l’appareil via un mot de passe

Les mots de passe garantissent que seuls les utilisateurs autorisés accèdent à vos données. Les appareils Azure Stack Edge Pro R démarrent en état verrouillé.

Vous pouvez :

- Vous connecter à l’interface utilisateur web locale de l’appareil via un navigateur, puis indiquer un mot de passe pour vous connecter à l’appareil.
- Vous connecter à distance à l’interface PowerShell de l’appareil via HTTP. La gestion à distance est désactivée par défaut. La gestion à distance est également configurée pour utiliser JEA (Just Enough Administration) afin de limiter ce que les utilisateurs peuvent faire. Vous pouvez ensuite fournir le mot de passe pour vous connecter à l’appareil. Pour plus d’informations, consultez [Connexion à distance à votre appareil](azure-stack-edge-gpu-connect-powershell-interface.md).
- L’utilisateur Edge local sur l’appareil dispose d’un accès limité à l’appareil pour la configuration initiale et la résolution des problèmes. Les charges de travail de calcul qui s’exécutent sur l’appareil, le transfert de données et le stockage sont tous accessibles à partir du portail public ou gouvernemental Azure pour la ressource dans le cloud.

[!INCLUDE [azure-stack-edge-gateway-data-rest](../../includes/azure-stack-edge-gateway-password-best-practices.md)]
- Utilisez l’interface utilisateur web locale pour [Modifier le mot de passe](azure-stack-edge-gpu-manage-access-power-connectivity-mode.md#change-device-password). Si vous modifiez le mot de passe, veillez à informer tous les utilisateurs à distance afin qu’ils ne connaissent pas de problème de connexion.

### <a name="establish-trust-with-the-device-via-certificates"></a>Établir une relation de confiance avec l’appareil via des certificats

L’appareil robuste Azure Stack Edge vous permet d’apporter vos propres certificats et d’installer ceux qui seront utilisés pour tous les points de terminaison publics. Pour plus d’informations, consultez la page [Charger votre certificat](azure-stack-edge-j-series-manage-certificates.md#upload-certificates). Pour obtenir la liste de tous les certificats qui peuvent être installés sur votre appareil, accédez à [Gérer des certificats sur votre appareil](azure-stack-edge-j-series-manage-certificates.md).

- Lorsque vous configurez le calcul sur votre appareil, un appareil IoT et un appareil IoT Edge sont créés. Des clés d’accès symétriques sont affectées automatiquement à ces appareils. Conformément aux meilleures pratiques de sécurité, le service IoT Hub veille à une rotation régulière de ces clés.

## <a name="protect-your-data"></a>Protéger vos données

Cette section décrit les fonctionnalités de sécurité visant à protéger les données transmises et stockées.

### <a name="protect-data-at-rest"></a>Protection des données au repos

Toutes les données au repos sur l’appareil sont chiffrées doublement, l’accès aux données est contrôlé et une fois que l’appareil est désactivé, les données sont effacées de façon sécurisée des disques de données.

#### <a name="double-encryption-of-data"></a>Double-chiffrement des données

Les données de vos disques sont protégées par deux couches de chiffrement :

- La première couche de chiffrement est le chiffrement BitLocker XTS-AES 256 bits sur les volumes de données.
- La deuxième couche est celle des disques durs dotés d’un chiffrement intégré.
- Le volume du système d’exploitation a BitLocker comme couche unique de chiffrement.

> [!NOTE]
> Le disque du système d’exploitation possède un chiffrement de logiciel BitLocker XTS-AES-256 à couche unique.

Lorsque l’appareil est activé, vous êtes invité à enregistrer un fichier de clé qui contient des clés de récupération permettant de récupérer les données sur l’appareil si ce dernier ne démarre pas. Le fichier comporte deux clés :

- Une clé récupère la configuration de l’appareil sur les volumes du système d’exploitation.
<!-- - Second key is to unlock the BitLocker on the data disks. -->
- La deuxième clé déverrouille le chiffrement matériel dans les disques de données.

> [!IMPORTANT]
> Enregistrez le fichier de clé dans un emplacement sécurisé en dehors de l’appareil. Si l’appareil ne démarre pas et que vous n’avez pas la clé, cela peut entraîner une perte de données.

- Certains scénarios de récupération vous inviteront à saisir le fichier de clé que vous avez enregistré. 
<!--- If a node isn't booting up, you will need to perform a node replacement. You will have the option to swap the data disks from the failed node to the new node. For a 4-node device, you won't need a key file. For a 1-node device, you will be prompted to provide a key file.-->

#### <a name="restricted-access-to-data"></a>Accès restreint aux données

L’accès aux données stockées dans les partages et les comptes de stockage est limité.

- Les clients SMB qui accèdent aux données partagées ont besoin des informations d’identification utilisateur associées au partage. Ces informations d’identification sont définies lorsque le partage est créé.
- Les clients NFS qui accèdent à un partage doivent avoir leur adresse IP ajoutée explicitement lors de la création du partage.
- Les comptes de stockage Edge créés sur l’appareil sont locaux et sont protégés par le chiffrement sur les disques de données. Les comptes de stockage Azure auxquels ces comptes de stockage Edge sont mappés sont protégés par l’abonnement et deux clés d’accès de stockage 512 bits associées au compte de stockage Edge (ces clés sont différentes de celles associées à vos comptes de stockage Azure). Pour plus d’informations, consultez [Protection des données dans les comptes de stockage](#protect-data-in-storage-accounts).
- Le chiffrement BitLocker XTS-AES de 256 bits est utilisé pour protéger les données locales.

#### <a name="secure-data-erasure"></a>Sécuriser l’effacement des données

Lorsque l’appareil subit une réinitialisation matérielle, une réinitialisation sécurisée est effectuée sur l’appareil. La réinitialisation sécurisée effectue l’effacement des données sur les disques à l’aide de la purge NIST SP 800-88r1.

### <a name="protect-data-in-flight"></a>Protection des données à la volée

[!INCLUDE [azure-stack-edge-gateway-data-rest](../../includes/azure-stack-edge-gateway-data-flight.md)]

### <a name="protect-data-in-storage-accounts"></a>Protection des données dans les comptes de stockage

[!INCLUDE [azure-stack-edge-gateway-data-rest](../../includes/azure-stack-edge-gateway-protect-data-storage-accounts.md)]

- Faites tourner puis [synchronisez vos clés de compte de stockage](azure-stack-edge-j-series-manage-storage-accounts.md) régulièrement pour mieux protéger votre compte de stockage des utilisateurs non autorisés.

## <a name="manage-personal-information"></a>Gestion des informations personnelles

Le service Azure Stack Edge collecte des informations personnelles dans les scénarios suivants :

[!INCLUDE [azure-stack-edge-gateway-data-rest](../../includes/azure-stack-edge-gateway-manage-personal-data.md)]

Pour afficher la liste des utilisateurs qui ont accès à un partage ou qui peuvent supprimer un partage, suivez les étapes décrites dans [Gérer les partages sur Azure Stack Edge](azure-stack-edge-j-series-manage-shares.md).

Pour plus d’informations, consultez la Politique de confidentialité Microsoft dans le [Centre de gestion de la confidentialité](https://www.microsoft.com/trustcenter).

## <a name="next-steps"></a>Étapes suivantes

[Déployer votre appareil Azure Stack Edge Pro R](azure-stack-edge-gpu-deploy-prep.md)
