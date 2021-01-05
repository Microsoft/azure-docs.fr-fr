---
title: Informations sur les réseaux dans la récupération d'urgence de machines virtuelles Azure avec Azure Site Recovery
description: Fournit une vue d’ensemble de la mise en réseau en vue d’une réplication des machines virtuelles Azure avec Azure Site Recovery.
services: site-recovery
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 3/13/2020
ms.author: harshacs
ms.openlocfilehash: b9fdaf8a0791570ecee402442c5faefe2f70a22b
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370438"
---
# <a name="about-networking-in-azure-vm-disaster-recovery"></a>Informations sur les réseaux dans la récupération d'urgence de machines virtuelles Azure



Cet article fournit des instructions détaillées sur la mise en réseau quand vous répliquez et récupérez des machines virtuelles d’une région vers une autre avec [Azure Site Recovery](site-recovery-overview.md).

## <a name="before-you-start"></a>Avant de commencer

Découvrez comment Site Recovery permet la récupération d’urgence pour [ce scénario](azure-to-azure-architecture.md).

## <a name="typical-network-infrastructure"></a>Infrastructure réseau classique

Le diagramme suivant illustre un environnement Azure classique pour des applications qui s’exécutent sur des machines virtuelles Azure :

![Diagramme montrant un environnement Azure classique pour des applications qui s’exécutent sur des machines virtuelles Azure.](./media/site-recovery-azure-to-azure-architecture/source-environment.png)

Si vous utilisez Azure ExpressRoute ou une connexion VPN de votre réseau local vers Azure, l’environnement ressemble à ceci :

![environnement client](./media/site-recovery-azure-to-azure-architecture/source-environment-expressroute.png)

Les réseaux sont généralement protégés à l’aide de pare-feu et de groupes de sécurité réseau. Les balises de service doivent être utilisées pour contrôler la connectivité réseau. Les groupes de sécurité réseau doivent autoriser plusieurs balises de service à contrôler la connectivité sortante.

>[!IMPORTANT]
> L’utilisation d’un proxy authentifié pour contrôler la connectivité réseau n’est pas pris en charge par Site Recovery, et la réplication ne peut pas être activée.

>[!NOTE]
>- Le filtrage basé sur l’adresse IP ne doit pas être effectué pour contrôler la connectivité sortante.
>- Les adresses IP d’Azure Site Recovery ne doivent pas être ajoutées à la table de routage Azure pour contrôler la connectivité sortante.

## <a name="outbound-connectivity-for-urls"></a>Connectivité sortante pour les URL

Si vous utilisez un proxy de pare-feu basé sur des URL pour contrôler la connectivité sortante, autorisez ces URL Site Recovery :

**URL** | **Détails**
--- | ---
*.blob.core.windows.net | Nécessaire pour que les données puissent être écrites dans le compte de stockage de cache dans la région source à partir de la machine virtuelle. Si vous connaissez tous les comptes de stockage de cache de vos machines virtuelles, vous pouvez autoriser l’accès aux URL de compte de stockage spécifiques (par exemple, cache1.blob.core.windows.net et cache2.blob.core.windows.net) au lieu de *.blob.core.windows.net.
login.microsoftonline.com | Nécessaire pour l’autorisation et l’authentification aux URL du service Site Recovery.
*.hypervrecoverymanager.windowsazure.com | Nécessaire pour que la communication du service Site Recovery puisse avoir lieu à partir de la machine virtuelle.
*.servicebus.windows.net | Nécessaire pour que les données de surveillance et de diagnostic Site Recovery puissent être écrites à partir de la machine virtuelle.
*.vault.azure.net | Autorise l’accès à la réplication pour les machines virtuelles compatibles avec ADE via le portail
*.automation.ext.azure.com | Autorise l’activation de la mise à niveau automatique de l’agent de mobilité pour un élément répliqué via le portail

## <a name="outbound-connectivity-using-service-tags"></a>Connectivité sortante à l’aide d’étiquettes de service

Quand vous utilisez un groupe de sécurité réseau pour contrôler la connectivité sortante, ces étiquettes de services doivent être autorisées.

- Pour les comptes de stockage dans la région source :
    - Créez une règle de groupe de sécurité réseau basée sur une [balise de service de stockage](../virtual-network/network-security-groups-overview.md#service-tags) pour la région source.
    - Autorisez ces adresses pour que les données puissent être écrites dans le compte de stockage de cache, à partir de la machine virtuelle.
- Créer une règle de groupe de sécurité réseau basée sur une [balise de service Azure Active Directory (AAD)](../virtual-network/network-security-groups-overview.md#service-tags) pour autoriser l’accès à toutes les adresses IP correspondant à AAD
- Créez une règle de groupe de sécurité réseau basée sur une balise du service EventsHub pour la région cible, en autorisant l’accès à la supervision de Site Recovery.
- Créez une règle de groupe de sécurité réseau basée sur une balise du service AzureSiteRecovery pour autoriser l’accès au service Site Recovery dans n’importe quelle région.
- Créez une règle NSG basée sur une étiquette de service AzureKeyVault. Cela est nécessaire uniquement pour l’activation de la réplication des machines virtuelles compatibles avec ADE via le portail.
- Créez une règle NSG basée sur une balise de service GuestAndHybridManagement. Cela est nécessaire uniquement pour l’activation de la mise à niveau automatique de l’agent de mobilité pour un élément répliqué via le portail.
- Nous vous recommandons de créer les règles de groupe de sécurité réseau requises sur un groupe de sécurité réseau de test, et de vérifier qu’il n’y a aucun problème avant de créer les règles sur un groupe de sécurité réseau de production.

## <a name="example-nsg-configuration"></a>Exemple de configuration de groupe de sécurité réseau

Cet exemple montre comment configurer des règles de groupes de sécurité réseau pour une machine virtuelle à répliquer.

- Si vous utilisez des règles de groupe de sécurité réseau pour contrôler la connectivité sortante, utilisez des règles « Autoriser le trafic HTTPS sortant » sur port:443 pour toutes les plages d’adresses IP requises.
- L’exemple part du principe que « USA Est » est l’emplacement source de la machine virtuelle, et que « USA Centre » en est l’emplacement cible.

### <a name="nsg-rules---east-us"></a>Règles de groupe de sécurité réseau - USA Est

1. Créez une règle de sécurité HTTPS sortante (443) pour « Storage.EastUS » sur le groupe de sécurité réseau comme indiqué dans la capture d’écran ci-dessous.

      ![Capture d’écran montrant Ajouter une règle de sécurité de trafic sortant applicable à un groupe de sécurité réseau pour le point de stockage USA Est.](./media/azure-to-azure-about-networking/storage-tag.png)

2. Créez une règle de sécurité HTTPS sortante (443) pour « AzureActiveDirectory » sur le groupe de sécurité réseau comme indiqué dans la capture d’écran ci-dessous.

      ![Capture d’écran montrant Ajouter une règle de sécurité de trafic sortant applicable à un groupe de sécurité réseau pour Azure AD.](./media/azure-to-azure-about-networking/aad-tag.png)

3. Comme pour les règles de sécurité ci-dessus, créez une règle de sécurité HTTPS sortante (443) pour « EventHub.CentralUS » sur le groupe de sécurité réseau qui correspond à la position cible. Celle-ci permet d’accéder à la supervision de Site Recovery.

4. Créez une règle de sécurité HTTPS sortante (443) pour « AzureSiteRecovery » sur le groupe de sécurité réseau. Celle-ci permet d’accéder au service Site Recovery dans n’importe quelle région.

### <a name="nsg-rules---central-us"></a>Règles de groupe de sécurité réseau - USA Centre

Ces règles sont nécessaires pour que la réplication puisse être activée de la région cible vers la région source après le basculement :

1. Créez une règle de sécurité HTTPS sortante (443) pour « Storage.CentralUS » sur le groupe de sécurité réseau.

2. Créez une règle de sécurité HTTPS sortante (443) pour « AzureActiveDirectory » sur le groupe de sécurité réseau.

3. Comme pour les règles de sécurité ci-dessus, créez une règle de sécurité HTTPS sortante (443) pour « EventHub.Eastus » sur le groupe de sécurité réseau qui correspond à la position source. Celle-ci permet d’accéder à la supervision de Site Recovery.

4. Créez une règle de sécurité HTTPS sortante (443) pour « AzureSiteRecovery » sur le groupe de sécurité réseau. Celle-ci permet d’accéder au service Site Recovery dans n’importe quelle région.

## <a name="network-virtual-appliance-configuration"></a>Configuration des appliances virtuelles réseau

Si vous utilisez des appliances virtuelles réseau pour contrôler le trafic réseau sortant des machines virtuelles, l’appliance risque de présenter des limitations si tout le trafic de réplication passe par elle. Nous vous recommandons de créer un point de terminaison de service réseau dans votre réseau virtuel pour « Stockage » afin que le trafic de réplication n’atteigne pas l’appliance virtuelle réseau.

### <a name="create-network-service-endpoint-for-storage"></a>Créer un point de terminaison de service réseau pour le stockage
Vous pouvez créer un point de terminaison de service réseau dans votre réseau virtuel pour « Stockage » afin que le trafic de réplication ne quitte pas la limite Azure.

- Sélectionnez votre réseau virtuel Azure et cliquez sur Points de terminaison de service

    ![storage-endpoint](./media/azure-to-azure-about-networking/storage-service-endpoint.png)

- Cliquez sur Ajouter et l’onglet Ajouter des points de terminaison de service s’ouvre
- Sélectionnez Microsoft.Storage sous Service et les sous-réseaux requis sous le champ Sous-réseaux, puis cliquez sur Ajouter

>[!NOTE]
>Ne limitez pas l’accès au réseau virtuel aux comptes de stockage utilisés pour ASR. Vous devez autoriser l’accès de Tous les réseaux

### <a name="forced-tunneling"></a>Tunneling forcé

Vous pouvez remplacer l’itinéraire système par défaut d’Azure pour le préfixe d’adresse 0.0.0.0/0 par un [itinéraire personnalisé](../virtual-network/virtual-networks-udr-overview.md#custom-routes) et rediriger le trafic des machines virtuelles vers une appliance virtuelle réseau locale, mais cette configuration n’est pas recommandée pour la réplication Site Recovery. Si vous utilisez des itinéraires personnalisés, vous devez [créer un point de terminaison de service de réseau virtuel](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage) dans votre réseau virtuel pour « Stockage » afin que le trafic de réplication ne quitte pas la limite Azure.

## <a name="next-steps"></a>Étapes suivantes
- Commencer à protéger vos charges de travail en [répliquant des machines virtuelles Azure](./azure-to-azure-quickstart.md).
- En savoir plus sur la [conservation des adresses IP](site-recovery-retain-ip-azure-vm-failover.md) pour le basculement de machines virtuelles Azure.
- En savoir plus sur la récupération d’urgence des [machines virtuelles Azure via ExpressRoute](azure-vm-disaster-recovery-with-expressroute.md).