---
title: Identité d’appareil et virtualisation de bureau - Azure Active Directory
description: Découvrez comment les identités d’appareil VDI et Azure AD peuvent être utilisées ensemble
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: c30ad26f079e6353dc4763b9ae968c33882d8ab6
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96029345"
---
# <a name="device-identity-and-desktop-virtualization"></a>Identité d’appareil et virtualisation de bureau

Les administrateurs déploient généralement des plateformes VDI (Virtual Desktop Infrastructure) hébergeant des systèmes d’exploitation Windows dans leurs organisations. Les administrateurs déploient la solution VDI pour :

- Simplifier la gestion.
- Réduire les coûts par le regroupement et la centralisation des ressources.
- Dispenser aux utilisateurs finals la mobilité et la liberté d’accéder aux bureaux virtuels, à tout moment, en tous lieux, sur tout appareil.

Il existe deux types principaux de bureaux virtuels :

- Persistante
- Non persistant

Les versions persistantes utilisent une image de bureau unique pour chaque utilisateur ou pool d’utilisateurs. Ces bureaux uniques peuvent être personnalisés et enregistrés pour une utilisation ultérieure. 

Les versions non persistantes utilisent une collection de bureaux auxquels les utilisateurs peuvent accéder en fonction des besoins. L’état d’origine de ces postes de travail non persistants est rétabli : dans le cas de Windows actuel<sup>1</sup>, cela se produit lorsqu’une machine virtuelle subit un processus d’arrêt/de redémarrage/de réinitialisation du système d’exploitation et, dans le cas de Windows de bas niveau<sup>2</sup>, cela se produit lorsqu’un utilisateur se déconnecte.

On constate une augmentation des déploiements VDI non persistants, le travail à distance continuant d’être la nouvelle norme. À mesure que les clients déploient une VDI non persistante, il est important de s’assurer que vous gérez les évolutions des appareils qui peuvent être causées par une inscription fréquente des appareils sans avoir de stratégie appropriée pour la gestion du cycle de vie des appareils.

> [!IMPORTANT]
> Si vous ne parvenez pas à gérer l’évolution des appareils, vous risquez d’augmenter la pression sur la consommation de votre quota de locataires et d’être confronté à un risque potentiel d’interruption de service si le quota de locataires est insuffisant. Vous devez suivre les instructions indiquées ci-dessous lors du déploiement d’environnements VDI non persistants afin d’éviter cette situation.

Cet article présente les conseils de Microsoft aux administrateurs, en matière de prise en charge de l’identité d’appareil et de la solution VDI. Pour plus d’informations sur l’identité d’appareil, consultez l’article [Présentation de l’identité d’appareil](overview.md).

## <a name="supported-scenarios"></a>Scénarios pris en charge

Avant de configurer les identités d’appareil dans Azure AD pour votre environnement VDI, familiarisez-vous avec les scénarios pris en charge. Le tableau ci-dessous illustre les scénarios de provisionnement pris en charge. Dans ce contexte, le provisionnement implique qu’un administrateur peut configurer des identités d’appareil à grande échelle, sans nécessiter l’intervention d’utilisateurs finals.

| Type d’identité d’appareil | Infrastructure d’identité | Appareils Windows | Version de la plateforme VDI | Prise en charge |
| --- | --- | --- | --- | --- |
| Appareils joints Azure AD hybrides | Fédérée<sup>3</sup> | Windows actuel et Windows de bas niveau | Persistante | Oui |
|   |   | Windows actuel | Non persistante | Oui<sup>5</sup> |
|   |   | Appareils Windows de bas niveau | Non persistante | Oui<sup>6</sup> |
|   | Managée<sup>4</sup> | Windows actuel et Windows de bas niveau | Persistante | Oui |
|   |   | Windows actuel | Non persistante | Non |
|   |   | Appareils Windows de bas niveau | Non persistante | Oui<sup>6</sup> |
| Appareil joints Azure AD | Adresses IP fédérées | Windows actuel | Persistante | Non |
|   |   |   | Non persistante | Non |
|   | Adresses IP gérées | Windows actuel | Persistante | Non |
|   |   |   | Non persistante | Non |
| Appareils inscrits sur Azure AD | Fédérée/managée | Windows actuel/Windows de bas niveau | Persistante/non persistante | Non applicable |

<sup>1</sup> Les appareils **Windows actuels** sont Windows 10, Windows Server 2016 v1803 ou ultérieur et Windows Server 2019.
<sup>2</sup> Les appareils **Windows de bas niveau** sont Windows 7, Windows 8.1, Windows Server 2008 R2, Windows Server 2012 et Windows Server 2012 R2. Pour des informations relatives à la prise en charge de Windows 7, consultez [Fin de la prise en charge de Windows 7](https://www.microsoft.com/microsoft-365/windows/end-of-windows-7-support). Pour obtenir des informations de support sur Windows Server 2008 R2, consultez [Préparez-vous à la fin du support de Windows Server 2008](https://www.microsoft.com/cloud-platform/windows-server-2008).

<sup>3</sup> Un environnement d’infrastructure d’identité **fédérée** représente un environnement avec un fournisseur d’identité, tel qu’AD FS ou d’autres IdP tiers.

<sup>4</sup> Un environnement d’infrastructure d’identité **managée** représente un environnement avec Azure AD en tant que fournisseur d’identité déployé au moyen de la [synchronisation de hachage de mot de passe (PHS)](../hybrid/whatis-phs.md) ou de l’[authentification directe (PTA)](../hybrid/how-to-connect-pta.md) avec l’[authentification unique fluide](../hybrid/how-to-connect-sso.md).

<sup>5</sup> La **prise en charge de la non persistance pour Windows actuel** nécessite une considération supplémentaire, comme indiqué ci-dessous dans la section d’aide. Ce scénario nécessite Windows 10 1803, Windows Server 2019 ou Windows Server (canal semi-annuel) à partir de la version 1803.

<sup>6</sup> La **prise en charge de la non persistance pour Windows de bas niveau** nécessite une considération supplémentaire, comme indiqué ci-dessous dans la section d’aide.


## <a name="microsofts-guidance"></a>Conseils de Microsoft

Les administrateurs doivent se référer aux articles suivants, en fonction de leur infrastructure d’identité, pour savoir comment configurer la jonction Azure AD Hybride.

- [Configurer la jonction Azure Active Directory Hybride pour un environnement fédéré](hybrid-azuread-join-federated-domains.md)
- [Configurer la jonction Azure Active Directory Hybride pour un environnement managé](hybrid-azuread-join-managed-domains.md)

Lors du déploiement d’une VDI non persistante, Microsoft recommande aux administrateurs informatiques d’implémenter les instructions ci-dessous. Dans le cas contraire, votre annuaire contiendra de nombreux appareils Azure AD Hybride obsolètes qui ont été inscrits à partir de votre plateforme VDI non persistante, ce qui entraînera une pression accrue sur votre quota de locataires et un risque d’interruption de service en raison de l’épuisement de ce quota.

- Si vous comptez sur l’outil de préparation système (sysprep.exe) et utilisez une image pré-Windows 10 1809 pour l’installation, vérifiez que cette image ne provient pas d’un appareil déjà inscrit auprès d’Azure AD en tant qu’appareil Azure AD Hybride joint.
- Si vous utilisez une capture instantanée de machine virtuelle pour créer des machines virtuelles supplémentaires, vérifiez qu’elle ne provient pas d'une machine virtuelle déjà inscrite auprès d'Azure AD en tant que jonction Azure AD Hybride.
- Créez et utilisez un préfixe pour le nom d’affichage (par exemple, NPVDI-) de l’ordinateur qui indique le bureau comme étant basé sur une VDI non persistante.
- Pour les appareils Windows de bas niveau :
   - Implémentez la commande **autoworkplacejoin /leave** dans le cadre du script de fermeture de session. Cette commande doit être déclenchée dans le contexte de l’utilisateur et doit être exécutée avant que l’utilisateur ne soit complètement déconnecté et tant qu’il y a encore une connectivité réseau.
- Pour Windows actuel dans un environnement fédéré (par exemple, AD FS) :
   - Implémentez **dsregcmd /join** dans le cadre de la séquence de démarrage de la machine virtuelle.
   - **N’EXÉCUTEZ PAS** dsregcmd /leave dans le cadre du processus d’arrêt/de redémarrage de la machine virtuelle.
- Définissez et implémentez le processus de [gestion des appareils obsolètes](manage-stale-devices.md).
   - Une fois que vous disposez d’une stratégie d’identification de vos appareils Azure AD Hybride joints non persistants (par exemple, en utilisant le préfixe du nom d’affichage de l’ordinateur), vous devez être plus agressif lors du nettoyage de ces appareils pour être certain que votre répertoire ne sera pas consommé par un grand nombre d’appareils obsolètes.
   - Pour les déploiements VDI non persistants sur Windows actuel et de bas niveau, vous devez supprimer les appareils qui ont une propriété **ApproximateLastLogonTimestamp** antérieure à 15 jours.
 
## <a name="next-steps"></a>Étapes suivantes

[Configuration de la jonction Azure AD Hybride pour un environnement fédéré](hybrid-azuread-join-federated-domains.md)
