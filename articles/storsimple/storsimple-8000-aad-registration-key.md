---
title: Authentification Azure AD pour StorSimple 8000 dans Gestionnaire de périphériques
description: Explique comment utiliser l’authentification AAD pour votre service, générer la nouvelle clé d’inscription et effectuer l’inscription manuelle des appareils.
author: alkohli
ms.service: storsimple
ms.topic: conceptual
ms.date: 01/23/2018
ms.author: alkohli
ms.openlocfilehash: 0e071a20051f047efbd040dfc01a30e3c1381367
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91631322"
---
# <a name="use-azuer-active-directory-ad-authentication-for-your-storsimple"></a>Utiliser l’authentification Azure Active Directory (AD) pour votre StorSimple

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble

Le service StorSimple Device Manager s’exécute dans Microsoft Azure et se connecte à plusieurs appareils StorSimple. Actuellement, le service Gestionnaire de périphériques StorSimple utilise un service de contrôle d’accès (ACS) pour authentifier le service sur votre appareil StorSimple. Ce mécanisme d’ACS sera bientôt mis hors service et remplacé par une authentification Azure Active Directory (AAD). Pour plus d’informations, consultez les annonces suivantes sur la mise hors service d’ACS et l’utilisation de l’authentification AAD.

- [Azure Active Directory est l’avenir d’Azure ACS](https://cloudblogs.microsoft.com/enterprisemobility/2015/02/12/the-future-of-azure-acs-is-azure-active-directory/)
- [Modifications prochaines du service Microsoft Azure Access Control Service](https://azure.microsoft.com/blog/acs-access-control-service-namespace-creation-restriction/)

Cet article décrit en détail l’authentification AAD et la nouvelle clé d’inscription du service associé, ainsi que les modifications apportées aux règles de pare-feu applicables aux appareils StorSimple. Les informations contenues dans cet article ne s’appliquent qu’aux appareils de la gamme StorSimple 8000.

L’authentification AAD s’effectue dans l’appareil StorSimple 8000 exécutant Update 5 ou version ultérieure. L’introduction de l’authentification AAD a des répercussions sur les éléments suivants :

- Modèles d’URL pour règles de pare-feu.
- Clé d’inscription du service.

Ces modifications sont décrites en détail dans les sections suivantes.

## <a name="url-changes-for-aad-authentication"></a>Modifications d’URL pour l’authentification AAD

Pour vérifier que le service utilise l’authentification AAD, tous les utilisateurs doivent inclure les nouvelles URL d’authentification dans leurs règles de pare-feu.

Si vous utilisez StorSimple 8000, assurez-vous que l’URL suivante est incluse dans les règles de pare-feu :

| Modèle d’URL                         | Cloud | Composant/Fonctionnalité         |
|------------------------------------|-------|----------------------------------|
| `https://login.windows.net`        | Azure (public) |Service d’authentification ADD      |
| `https://login.microsoftonline.us` | Gouvernement américain |Service d’authentification ADD      |

Pour obtenir la liste complète des modèles d’URL pour les appareils StorSimple 8000, consultez [Modèles d’URL pour règles de pare-feu](storsimple-8000-system-requirements.md#url-patterns-for-firewall-rules).

Si l’URL d’authentification n’est pas incluse dans les règles de pare-feu au-delà de la date de mise hors service et que l’appareil exécute Update 5, les utilisateurs voient une alerte d’URL. Les utilisateurs doivent inclure la nouvelle URL d’authentification. Si l’appareil exécute une version antérieure à Update 5, les utilisateurs voient une alerte de pulsation. Dans chaque cas, l’appareil StorSimple ne peut pas s’authentifier auprès du service et ce dernier ne peut pas communiquer avec l’appareil.

## <a name="device-version-and-authentication-changes"></a>Modifications de la version et de l’authentification de l’appareil

Avec un appareil StorSimple 8000, utilisez le tableau suivant pour déterminer l’action à effectuer en fonction de la version du logiciel d’appareil que vous exécutez.

| Si votre appareil exécute| Procédez comme suit                                    |
|--------------------------|------------------------|
| Update 5 ou une version ultérieure et que l’appareil est hors connexion. <br> Vous voyez une alerte indiquant que l’URL n’est pas approuvée.|1. Modifiez les règles de pare-feu et incluez-y l’URL d’authentification. Consultez la section [URL d’authentification](#url-changes-for-aad-authentication).<br>2. [Obtenez la clé d’inscription AAD auprès du service](#aad-based-registration-keys).<br>3. [Connectez-vous à l’interface Windows PowerShell de l’appareil StorSimple 8000](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).<br>4. Utilisez la cmdlet `Redo-DeviceRegistration` pour inscrire l’appareil via Windows PowerShell. Indiquez la clé que vous avez obtenue à l’étape précédente.|
| Update 5 ou une version ultérieure et que l’appareil est en ligne.| Aucune action n'est requise.                                       |
| Update 4 ou une version antérieure et que l’appareil est hors ligne. |1. Modifiez les règles de pare-feu et incluez-y l’URL d’authentification.<br>2. [Téléchargez Update 5 via le serveur de catalogue](storsimple-8000-install-update-5.md#download-updates-for-your-device).<br>3. [Appliquer Update 5 par la méthode du correctif logiciel](storsimple-8000-install-update-5.md#install-update-5-as-a-hotfix).<br>4. [Obtenez la clé d’inscription AAD auprès du service](#aad-based-registration-keys).<br>5. [Connectez-vous à l’interface Windows PowerShell de l’appareil StorSimple 8000](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console). <br>6. Utilisez la cmdlet `Redo-DeviceRegistration` pour inscrire l’appareil via Windows PowerShell. Indiquez la clé que vous avez obtenue à l’étape précédente.|
| Update 4 ou une version antérieure et que l’appareil est en ligne. |Modifiez les règles de pare-feu et incluez-y l’URL d’authentification.<br> Installez Update 5 à l’aide du portail Azure.              |
| Effectuez une réinitialisation au paramètres d’usine à une version antérieure à Update 5.      |Le portail affiche une clé d’inscription AAD lorsque l’appareil exécute un ancien logiciel. Suivez les étapes décrites dans le scénario précédent lorsque l’appareil exécute Update 4 ou une version antérieure.              |

## <a name="aad-based-registration-keys"></a>Clés d’inscription AAD

À partir d’Update 5 pour les appareils StorSimple 8000, les nouvelles clés d’inscription AAD sont utilisées. Vous utilisez les clés d’inscription pour inscrire votre service Gestionnaire de périphériques Microsoft Azure StorSimple à l’appareil.

Vous ne pouvez pas utiliser les nouvelles clés d’inscription AAD avec un appareil StorSimple 8000 exécutant Update 4 ou une version antérieure (y compris un appareil plus ancien maintenant activé).
Dans ce cas, vous devez régénérer la clé d’inscription au service. Lorsque vous régénérez la clé, celle-ci permet d’inscrire tous les appareils suivants. L’ancienne clé n’est plus valide.

- La nouvelle clé d’inscription AAD expire au bout de 3 jours.
- Les clés d’inscription AAD ne fonctionnent qu’avec les appareils StorSimple 8000 exécutant Update 5 ou une version ultérieure.
- Les clés d’inscription AAD sont plus longues que les clés d’inscription ACS correspondantes.

Procédez comme suit pour générer une clé d’inscription au service AAD.

#### <a name="to-generate-the-aad-service-registration-key"></a>Pour régénérer la clé d’inscription du service AAD

1. Dans **Gestionnaire de périphériques StorSimple**, accédez à **Gestion &gt;** **Clés**. Vous pouvez également utiliser la barre de recherche pour rechercher _Clés_.
    
2. Cliquez sur **Générer une clé**.

    ![Cliquer sur Régénérer](./media/storsimple-8000-aad-registration-key/aad-click-generate-registration-key.png)

3. Copiez la nouvelle clé. L’ancienne clé ne fonctionnera plus.

    ![Confirmer la régénération](./media/storsimple-8000-aad-registration-key/aad-registration-key2.png)

    > [!NOTE] 
    > Si vous créez un StorSimple Cloud Appliance sur le service inscrit à votre appareil StorSimple 8000, ne générez pas de clé d’inscription lorsque la création est en cours. Attendez que la création se termine, puis générez la clé d’inscription.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le déploiement d’un [appareil StorSimple 8000](storsimple-8000-deployment-walkthrough-u2.md).
