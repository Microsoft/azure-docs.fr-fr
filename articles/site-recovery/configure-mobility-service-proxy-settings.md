---
title: Configurer les paramètres proxy du service Mobilité pour la reprise d’activité d’Azure vers Azure | Microsoft Docs
description: Fournit des détails sur la façon de configurer le service Mobilité lorsque les clients utilisent un proxy dans leur environnement source.
services: site-recovery
author: sideeksh
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 03/18/2020
ms.author: sideeksh
ms.openlocfilehash: 429ffcab147142ae2e96de13b7c9e1e5ee1ac7ba
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86133230"
---
# <a name="configure-mobility-service-proxy-settings-for-azure-to-azure-disaster-recovery"></a>Configurer les paramètres proxy du service Mobilité pour la reprise d’activité d’Azure vers Azure

Cet article fournit des conseils sur la façon de personnaliser les configurations réseau sur la machine virtuelle Azure quand vous répliquez et récupérez des machines virtuelles Azure d’une région vers une autre avec [Azure Site Recovery](site-recovery-overview.md).

L’objectif de ce document est d’indiquer les étapes à suivre pour configurer les paramètres proxy pour le service Mobilité Azure Site Recovery dans le scénario de reprise d’activité d’Azure vers Azure. 

Les proxies sont des passerelles réseau qui autorisent ou interdisent les connexions réseau à des points de terminaison. En général, un proxy est une machine située hors de la machine client qui tente d’accéder aux points de terminaison réseau. Une liste de contournement permet au client d’établir des connexions directement avec les points de terminaison sans passer par le proxy. Un nom d’utilisateur et un mot de passe peuvent éventuellement être définis pour un proxy par les administrateurs réseau afin que seuls les clients authentifiés puissent utiliser le proxy. 

## <a name="before-you-start"></a>Avant de commencer

Découvrez comment Site Recovery permet la reprise d’activité pour [ce scénario](azure-to-azure-architecture.md).
Assimilez les [instructions sur le réseau](azure-to-azure-about-networking.md) quand vous répliquez et récupérez les machines virtuelles Azure d’une région sur une autre en utilisant [Azure Site Recovery](site-recovery-overview.md).
Vérifiez que votre proxy est configuré de façon appropriée en fonction des besoins de votre organisation.

## <a name="configure-the-mobility-service"></a>Configurer le service Mobilité

Le service Mobilité prend en charge uniquement les proxies non authentifiés. Il offre deux moyens d’entrer les détails du proxy pour la communication avec les points de terminaison Site Recovery. 

### <a name="method-1-auto-detection"></a>Méthode 1 : Détection automatique

Le service Mobilité détecte automatiquement les paramètres de proxy à partir des paramètres d’environnement ou des paramètres Internet Explorer (Windows uniquement) pendant l’activation de la réplication. 

- Système d’exploitation Windows : Pendant l’activation de la réplication, le service Mobilité détecte les paramètres proxy configurés dans Internet Explorer pour l’utilisateur Système local. Pour configurer le proxy pour le compte Système local, un administrateur peut utiliser psexec pour lancer une invite de commandes, puis Internet Explorer. 
- Système d’exploitation Windows : Les paramètres proxy sont configurés sous la forme des variables d’environnement http_proxy et no_proxy. 
- Système d’exploitation Linux : Les paramètres proxy sont configurés dans /etc/profile ou /etc/environment sous la forme des variables d’environnement http_proxy et no_proxy. 
- Les paramètres proxy détectés automatiquement sont enregistrés dans le fichier de configuration de proxy du service Mobilité, ProxyInfo.conf. 
- Emplacement par défaut de ProxyInfo.conf 
    - Windows : C:\ProgramData\Microsoft Azure Site Recovery\Config\ProxyInfo.conf 
    - Linux : /usr/local/InMage/config/ProxyInfo.conf


### <a name="method-2-provide-custom-application-proxy-settings"></a>Méthode 2 : Fournir des paramètres proxy d’application personnalisés

Dans ce cas, le client fournit des paramètres proxy d’application personnalisés dans le fichier de configuration du service Mobilité, ProxyInfo.conf. Cette méthode permet aux clients de fournir un proxy uniquement pour le service Mobilité ou un proxy pour le service Mobilité Azure Site Recovery et un autre proxy (ou aucun proxy) pour le reste des applications sur la machine.

## <a name="proxy-template"></a>Modèle de proxy
ProxyInfo.conf contient le modèle suivant [proxy] Address=http://1.2.3.4 Port=5678 BypassList=hypervrecoverymanager.windowsazure.com,login.microsoftonline.com,blob.core.windows.net. BypassList ne prend pas en charge les caractères génériques tels que '*.windows.net', mais fournir windows.net suffit pour effectuer le contournement. 

## <a name="next-steps"></a>Étapes suivantes :
- Lisez les [instructions sur le réseau](./azure-to-azure-about-networking.md) pour la réplication des machines virtuelles Azure.
- Déployez la reprise d’activité en [répliquant des machines virtuelles Azure](./azure-to-azure-quickstart.md).
