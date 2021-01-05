---
title: Utiliser le Pare-feu Azure pour protéger Windows Virtual Desktop
description: Découvrez comment utiliser le Pare-feu Azure pour protéger les déploiements de Windows Virtual Desktop
author: vhorne
ms.service: firewall
services: firewall
ms.topic: how-to
ms.date: 05/06/2020
ms.author: victorh
ms.openlocfilehash: ad4f42d0e33f6d70c75abfcd1daab4f5aa9a515f
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94654981"
---
# <a name="use-azure-firewall-to-protect-window-virtual-desktop-deployments"></a>Utiliser le Pare-feu Azure pour protéger les déploiements de Windows Virtual Desktop

Windows Virtual Desktop est un service de virtualisation de bureau et d’application qui s’exécute sur Azure. Quand un utilisateur final se connecte à un environnement Windows Virtual Desktop, sa session est exécutée par un pool d’hôtes. Un pool d’hôtes est une collection de machines virtuelles Azure qui s’inscrivent auprès de Windows Virtual Desktop en tant qu’hôtes de session. Ces machines virtuelles s’exécutent dans votre réseau virtuel et sont soumises aux contrôles de sécurité de ce dernier. Elles ont besoin d’un accès Internet sortant au service Windows Virtual Desktop pour fonctionner correctement et peuvent également nécessiter un accès Internet sortant pour les utilisateurs finaux. Le Pare-feu Azure peut vous aider à verrouiller votre environnement et à filtrer le trafic sortant.

[ ![Architecture de Windows Virtual Desktop](media/protect-windows-virtual-desktop/windows-virtual-desktop-architecture-diagram.png) ](media/protect-windows-virtual-desktop/windows-virtual-desktop-architecture-diagram.png#lightbox)

Suivez les instructions de cet article pour renforcer la protection de votre pool d’hôtes Windows Virtual Desktop à l’aide du Pare-feu Azure.

## <a name="prerequisites"></a>Prérequis


 - Un environnement Windows Virtual Desktop déployé et un pool d’hôtes.

   Pour plus d’informations, consultez [Didacticiel : Créer un pool d’hôtes en utilisant la Place de marché Azure](../virtual-desktop/create-host-pools-azure-marketplace.md) et [Créer un pool d’hôtes avec un modèle Azure Resource Manager](../virtual-desktop/virtual-desktop-fall-2019/create-host-pools-arm-template.md).

Pour en savoir plus sur les environnements Windows Virtual Desktop, consultez [Environnement Windows Virtual Desktop](../virtual-desktop/environment-setup.md).

## <a name="host-pool-outbound-access-to-windows-virtual-desktop"></a>Accès sortant du pool d’hôtes à Windows Virtual Desktop

Les machines virtuelles Azure que vous créez pour Windows Virtual Desktop doivent avoir accès à plusieurs noms de domaine complets (FQDN) pour fonctionner correctement. Le Pare-feu Azure fournit une étiquette FQDN Windows Virtual Desktop pour simplifier cette configuration. Procédez comme suit pour autoriser le trafic sortant de la plateforme Windows Virtual Desktop :

- Déployez le Pare-feu Azure et configurez la route définie par l’utilisateur (UDR) du sous-réseau de pool d’hôtes Windows Virtual Desktop afin que le trafic transite par le Pare-feu Azure. La route par défaut pointe maintenant vers le pare-feu.
- Créez une collection de règles d’application et ajoutez une règle pour activer l’étiquette FQDN *WindowsVirtualDesktop*. La plage d’adresses IP source est le réseau virtuel du pool d’hôtes, le protocole est **https** et la destination est **WindowsVirtualDesktop**.

- L’ensemble des comptes de stockage et Service Bus requis pour votre pool d’hôtes Windows Virtual Desktop étant propre au déploiement, il n’est pas encore capturé dans l’étiquette FQDN WindowsVirtualDesktop. Vous pouvez résoudre cette situation de l’une des manières suivantes :

   - Autorisez l’accès https à partir de votre sous-réseau de pool d’hôtes vers *xt.blob.core.windows.net, *eh.servicebus.windows.net et *xt.table.core.windows.net. Ces noms de domaine complets génériques permettent l’accès requis, mais sont moins restrictifs.
   - Utilisez la requête Log Analytics suivante pour lister les noms de domaine complets requis exacts, puis autorisez-les explicitement dans les règles d’application de votre pare-feu :
   ```
   AzureDiagnostics
   | where Category == "AzureFirewallApplicationRule"
   | search "Deny"
   | search "gsm*eh.servicebus.windows.net" or "gsm*xt.blob.core.windows.net" or "gsm*xt.table.core.windows.net"
   | parse msg_s with Protocol " request from " SourceIP ":" SourcePort:int " to " FQDN ":" *
   | project TimeGenerated,Protocol,FQDN
   ```

- Créez une collection de règles de réseau, puis ajoutez les règles suivantes :

   - Autoriser DNS : autorisez le trafic à partir de votre adresse IP privée ADDS vers * pour les ports TCP et UDP 53.
   - Autoriser KMS : autorisez le trafic à partir de vos machines virtuelles Windows Virtual Desktop vers le port TCP 1688 du service d’activation Windows. Pour plus d’informations sur les adresses IP de destination, consultez [L’activation de Windows échoue dans un scénario de tunneling forcé](../virtual-machines/troubleshooting/custom-routes-enable-kms-activation.md#solution).

> [!NOTE]
> Certains déploiements n’ont peut-être pas besoin de règles DNS ; par exemple, les contrôleurs de domaine Azure Active Directory transfèrent les requêtes DNS à Azure DNS à l’adresse 168.63.129.16.

## <a name="host-pool-outbound-access-to-the-internet"></a>Accès sortant du pool d’hôtes à Internet

Selon les besoins de votre organisation, vous souhaiterez peut-être activer l’accès Internet sortant sécurisé pour vos utilisateurs finaux. Dans les cas où la liste des destinations autorisées est bien définie (par exemple [accès Microsoft 365](/microsoft-365/enterprise/microsoft-365-ip-web-service)), vous pouvez utiliser des règles d’application et de réseau du Pare-feu Azure pour configurer l’accès requis. Le trafic des utilisateurs finaux est ainsi directement routé vers Internet, ce qui optimise les performances.

Si vous souhaitez filtrer le trafic Internet utilisateur sortant à l’aide d’une passerelle web sécurisée locale existante, vous pouvez configurer des navigateurs web ou d’autres applications qui s’exécutent sur le pool d’hôtes Windows Virtual Desktop avec une configuration de proxy explicite. Pour obtenir un exemple, consultez [Procédure d’utilisation des options de ligne de commande MicrosoftEdge pour configurer les paramètres de proxy](/deployedge/edge-learnmore-cmdline-options-proxy-settings). Ces paramètres de proxy n’influencent que l’accès Internet de l’utilisateur final, ce qui permet le trafic sortant de la plateforme Windows Virtual Desktop directement via le Pare-feu Azure.

## <a name="additional-considerations"></a>Considérations supplémentaires

Vous devrez peut-être configurer des règles de pare-feu supplémentaires, en fonction de vos exigences :

- Accès au serveur NTP

   Par défaut, les machines virtuelles exécutant Windows se connectent à time.windows.com via le port UDP 123 pour la synchronisation de l’heure. Créez une règle de réseau pour autoriser cet accès ou pour un serveur de temps que vous utilisez dans votre environnement.


## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur Windows Virtual Desktop : [Qu’est-ce que Windows Virtual Desktop ?](../virtual-desktop/overview.md)