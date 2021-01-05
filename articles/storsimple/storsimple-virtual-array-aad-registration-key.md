---
title: Nouvelle authentification pour StorSimple Virtual Arrays
description: Découvrez l’authentification AAD et la nouvelle clé d’inscription du service associé, ainsi que les modifications apportées aux règles de pare-feu applicables aux appareils StorSimple.
author: alkohli
ms.service: storsimple
ms.topic: conceptual
ms.date: 07/25/2019
ms.author: alkohli
ms.openlocfilehash: d750b3a85741bb4761b640848f01e5917760b5aa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91261088"
---
# <a name="use-the-new-authentication-for-your-storsimple"></a>Utiliser la nouvelle authentification pour votre StorSimple

## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

Le service StorSimple Device Manager s’exécute dans Microsoft Azure et se connecte à plusieurs StorSimple Virtual Arrays. Actuellement, le service Gestionnaire de périphériques StorSimple utilise un service de contrôle d’accès (ACS) pour authentifier le service sur votre appareil StorSimple. Ce mécanisme d’ACS sera bientôt mis hors service et remplacé par une authentification Azure Active Directory (AAD).

Les informations contenues dans cet article s’appliquent uniquement à la gamme StorSimple 1200 Virtual Arrays. Cet article décrit en détail l’authentification AAD et la nouvelle clé d’inscription du service associé, ainsi que les modifications apportées aux règles de pare-feu applicables aux appareils StorSimple.

L’authentification AAD s’effectue dans StorSimple Virtual Arrays (modèle 1200) exécutant Update 1 ou version ultérieure.

L’introduction de l’authentification AAD a des répercussions sur les éléments suivants :

- Modèles d’URL pour règles de pare-feu.
- Clé d’inscription du service.

Ces modifications sont décrites en détail dans les sections suivantes.

## <a name="url-changes-for-aad-authentication"></a>Modifications d’URL pour l’authentification AAD

Pour vérifier que le service utilise l’authentification AAD, tous les utilisateurs doivent inclure les nouvelles URL d’authentification dans leurs règles de pare-feu.

Si vous utilisez StorSimple Virtual Array, assurez-vous que l’URL suivante est incluse dans les règles de pare-feu :

| Modèle d’URL                         | Cloud | Composant/Fonctionnalité         |
|------------------------------------|-------|---------------------------------|
| `https://login.windows.net`        | Azure (public) |Service d’authentification ADD      |
| `https://login.microsoftonline.us` | Gouvernement américain |Service d’authentification ADD      |

Pour obtenir la liste complète des modèles d’URL pour StorSimple Virtual Arrays, consultez [Modèles d’URL pour règles de pare-feu](storsimple-ova-system-requirements.md#url-patterns-for-firewall-rules).

Si l’URL d’authentification ne figure pas dans les règles de pare-feu après la date d’obsolescence, les utilisateurs voient une alerte critique les avertissant que leur appareil StorSimple ne peut pas s’authentifier auprès du service. Le service ne sera pas en mesure de communiquer avec l’appareil. Si les utilisateurs voient cette alerte, ils doivent intégrer la nouvelle URL d’authentification. Pour plus d’informations sur l’alerte, consultez [Utiliser des alertes pour surveiller votre appareil StorSimple](storsimple-virtual-array-manage-alerts.md#networking-alerts).

## <a name="device-version-and-authentication-changes"></a>Modifications de la version et de l’authentification de l’appareil

Avec un StorSimple Virtual Array, utilisez le tableau suivant pour déterminer l’action à effectuer en fonction de la version du logiciel d’appareil que vous exécutez.

| Si votre appareil exécute  | Procédez comme suit                                    |
|----------------------------|--------------------------------------------------------------|
| Update 1.0 ou une version ultérieure et que l’appareil est hors connexion. <br> Vous voyez une alerte indiquant que l’URL n’est pas dans la liste verte.| 1. Modifiez les règles de pare-feu et incluez-y l’URL d’authentification. Consultez la section [URL d’authentification](#url-changes-for-aad-authentication). <br> 2. [Obtenez la clé d’inscription AAD auprès du service](#aad-based-registration-keys). <br> 3. Suivez les étapes 1 à 5 pour [vous connecter à l’interface Windows PowerShell du tableau virtuel](storsimple-virtual-array-deploy2-provision-hyperv.md#step-2-provision-a-virtual-array-in-hypervisor).<br> 4. Utilisez la cmdlet `Invoke-HcsReRegister` pour inscrire l’appareil via Windows PowerShell. Indiquez la clé que vous avez obtenue à l’étape précédente.|
| Update 1.0 ou une version ultérieure et que l’appareil est en ligne.| Aucune action n'est requise.                                       |
| Update 0.6 ou une version antérieure et que l’appareil est hors ligne. | 1. [Téléchargez Update 1.0 via le serveur de catalogue](storsimple-virtual-array-install-update-1.md#download-the-update-or-the-hotfix).<br>2. [Appliquez Update 1.0 via l’interface utilisateur web locale](storsimple-virtual-array-install-update-1.md#install-the-update-or-the-hotfix).<br>3. [Obtenez la clé d’inscription AAD auprès du service](#aad-based-registration-keys). <br>4. Suivez les étapes 1 à 5 pour [vous connecter à l’interface Windows PowerShell du tableau virtuel](storsimple-virtual-array-deploy2-provision-hyperv.md#step-2-provision-a-virtual-array-in-hypervisor).<br>5. Utilisez la cmdlet `Invoke-HcsReRegister` pour inscrire l’appareil via Windows PowerShell. Indiquez la clé que vous avez obtenue à l’étape précédente.|
| Update 0.6 ou une version antérieure et que l’appareil est en ligne | Modifiez les règles de pare-feu et incluez-y l’URL d’authentification.<br> Installez Update 1.0 à l’aide du portail Azure. |

## <a name="aad-based-registration-keys"></a>Clés d’inscription AAD

À partir d’Update 1.0 pour StorSimple Virtual Arrays, les nouvelles clés d’inscription AAD sont utilisées. Vous utilisez les clés d’inscription pour inscrire votre service Gestionnaire de périphériques Microsoft Azure StorSimple à l’appareil.

Vous ne pouvez pas utiliser les nouvelles clés d’inscription au service AAD si vous utilisez StorSimple Virtual Arrays exécutant Update 0.6 ou une version antérieure. Vous devez régénérer la clé d’inscription au service. Lorsque vous régénérez la clé, celle-ci permet d’inscrire tous les appareils suivants. L’ancienne clé n’est plus valide.

- La nouvelle clé d’inscription AAD expire au bout de 3 jours.
- Les clés d’inscription AAD ne fonctionnent qu’avec les tableaux virtuels de la gamme StorSimple 1200 exécutant Update 1 ou une version ultérieure. La clé d’inscription AAD d’un appareil de la gamme StorSimple 8000 ne fonctionnera pas.
- Les clés d’inscription AAD sont plus longues que les clés d’inscription ACS correspondantes.

Procédez comme suit pour générer une clé d’inscription au service AAD.

#### <a name="to-generate-the-aad-service-registration-key"></a>Pour régénérer la clé d’inscription du service AAD

1. Dans **Gestionnaire de périphériques StorSimple**, accédez à **Gestion &gt;** **Clés**.
    
    ![Accéder aux clés](./media/storsimple-virtual-array-aad-registration-key/aad-registration-key1.png)

2. Cliquez sur **Générer une clé**.

    ![Cliquer sur Régénérer](./media/storsimple-virtual-array-aad-registration-key/aad-click-generate-registration-key.png)

3. Copiez la nouvelle clé. L’ancienne clé ne fonctionnera plus.

    ![Confirmer la régénération](./media/storsimple-virtual-array-aad-registration-key/aad-registration-key2.png)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le déploiement de [StorSimple Virtual Array](storsimple-virtual-array-deploy1-portal-prep.md)
