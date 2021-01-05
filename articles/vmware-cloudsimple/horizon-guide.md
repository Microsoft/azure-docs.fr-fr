---
title: Azure VMware Solution by CloudSimple - Utiliser le site du cloud privé pour héberger une infrastructure de bureau virtuel à l'aide de VMware Horizon
description: Vous explique comment utiliser le site de votre cloud privé CloudSimple pour héberger une infrastructure de bureau virtuel à l'aide de VMware Horizon.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 723821a78ecae308443c93567402e3b232c036f0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77025246"
---
# <a name="use-cloudsimple-private-cloud-site-to-host-a-virtual-desktop-infrastructure-using-vmware-horizon"></a>Utiliser le site du cloud privé CloudSimple pour héberger une infrastructure de bureau virtuel à l'aide de VMware Horizon

Vous pouvez utiliser le site de votre cloud privé CloudSimple pour héberger une infrastructure de bureau virtuel (VDI) à l'aide de VMware Horizon 7.x. La figure suivante illustre l'architecture de la solution logique de l'infrastructure VDI.

![Déploiement Horizon](media/horizon-deployment.png)

Cette solution vous confère un contrôle total sur Horizon View Manager et App Volume. Les interfaces UI, API et CLI vous permettent d'utiliser vos scripts et outils existants.

Pour tirer parti de la solution CloudSimple, vous devez :

* installer, configurer et gérer VMware Horizon 7.x dans votre cloud privé ;
* fournir vos propres licences Horizon.

## <a name="deploy-the-solution"></a>Déployer la solution

Les sections suivantes expliquent comment déployer une solution VDI à l'aide d'Horizon dans votre cloud privé.

1. [Vérifier que les versions des produits VMware sont compatibles](#verify-that-vmware-product-versions-are-compatible)
2. [Estimer la taille de votre environnement de bureau](#estimate-the-size-of-your-desktop-environment)
3. [Créer un cloud privé pour votre environnement](#create-a-private-cloud-for-your-environment)
4. [Installer VMware Horizon dans votre cloud privé](#install-vmware-horizon-in-your-private-cloud)

### <a name="verify-that-vmware-product-versions-are-compatible"></a>Vérifier que les versions des produits VMware sont compatibles

* Vérifiez que vos versions actuelles et prévues d'Horizon, App Volumes, Unified Access Gateway et User Environment Manager sont compatibles entre elles ainsi qu'avec vCenter et PSC dans le cloud privé. Pour plus d'informations sur la compatibilité, consultez la [Matrice de compatibilité VMware relative à Horizon 7.5](https://www.vmware.com/resources/compatibility/sim/interop_matrix.php#interop&260=2877&0=).
* Pour connaître les versions de vCenter et de PSC utilisées dans votre cloud privé, accédez à **Ressources** sur le [portail CloudSimple](access-cloudsimple-portal.md), puis sélectionnez votre cloud privé et cliquez sur l'onglet **vSphere Management Network**.

![Versions de vCenter et de PSC](media/private-cloud-vsphere-versions.png)

### <a name="estimate-the-size-of-your-desktop-environment"></a>Estimer la taille de votre environnement de bureau

* Vérifiez que votre configuration se situe dans les limites opérationnelles de VMware.
* Estimez les ressources nécessaires pour tous vos postes de travail et composants de gestion Horizon.

### <a name="create-a-private-cloud-for-your-environment"></a>Créer un cloud privé pour votre environnement

1. Créez un cloud privé à partir du portail CloudSimple en suivant les instructions présentées dans [Configurer un environnement de cloud privé](quickstart-create-private-cloud.md).  CloudSimple crée un utilisateur vCenter par défaut nommé « cloudowner » dans chaque cloud privé nouvellement créé. Pour plus d'informations sur l'utilisateur et le modèle d'autorisation par défaut du cloud privé, consultez [Modèle d'autorisation du cloud privé](learn-private-cloud-permissions.md).
2. Créez un réseau local virtuel (VLAN) dans votre cloud privé pour le plan de gestion Horizon, et attribuez-lui un CIDR de sous-réseau. Pour plus d'informations, consultez [Créer et gérer des réseaux VLAN/sous-réseaux](create-vlan-subnet.md). Il s'agit du réseau sur lequel tous les composants de solution (serveurs Unified Access Gateway, Connection Server, App Volume Server et User Environment Manager) seront installés.
3. Vous pouvez choisir d'utiliser un fournisseur d'identité externe avec l'instance vCenter de votre cloud privé. Dans ce cas, choisissez l'une des options suivantes :
    * Utilisez votre Active Directory local comme fournisseur d'identité externe. Pour plus d'informations, consultez [Sources d'identités vCenter](set-vcenter-identity.md).
    * Configurez un serveur Active Directory du cloud privé du réseau VLAN du plan de gestion Horizon pour l'utiliser comme fournisseur d'identité externe. Pour plus d'informations, consultez [Sources d'identités vCenter](set-vcenter-identity.md).
    * Configurez un serveur DNS et DHCP du réseau VLAN du plan de gestion Horizon dans le cloud privé. Pour plus d'informations, consultez [Configurer des applications et des charges de travail DNS et DHCP dans votre cloud privé CloudSimple](dns-dhcp-setup.md).
4. Configurez le transfert DNS sur le serveur DNS installé dans le cloud privé. Pour plus d'informations, consultez [Créer un redirecteur conditionnel](on-premises-dns-setup.md#create-a-conditional-forwarder).

### <a name="install-vmware-horizon-in-your-private-cloud"></a>Installer VMware Horizon dans votre cloud privé

Le diagramme de déploiement suivant illustre une solution Horizon déployée dans un cloud privé. Unified Access Gateway, AD/DC, View et App Volume Server sont installés dans le réseau VLAN 234 créé par l'utilisateur. Unified Access Gateway dispose d'une adresse IP publique accessible à partir d'Internet. Les machines virtuelles du pool de bureaux Horizon sont déployées dans le réseau VLAN 235 pour fournir un niveau d'isolation et de sécurité supplémentaire.

![Déploiement Horizon dans le cloud privé](media/horizon-private-cloud.png)

Les sections suivantes contiennent des instructions pour configurer un déploiement semblable à celui de la figure. Avant de commencer, vérifiez que les conditions suivantes sont remplies :

* Cloud privé créé à l'aide du portail CloudSimple et doté d'une capacité suffisante pour exécuter vos pools de bureaux
* Bande passante suffisante entre votre environnement local et l'environnement de cloud privé afin de prendre en charge le trafic réseau de vos bureaux
* Tunnel VPN de site à site configuré entre votre centre de données local et le cloud privé
* Accessibilité IP entre les sous-réseaux des utilisateurs finaux de votre environnement local et les sous-réseaux du cloud privé CloudSimple
* AD/DHCP/DNS installés pour votre cloud privé

#### <a name="cloudsimple-portal-create-a-dedicated-vlansubnet-for-desktop-pools"></a>Portail CloudSimple : Créer un réseau local virtuel/sous-réseau dédié pour les pools de bureaux

Créez un réseau local virtuel (VLAN) pour les pools de bureaux Horizon et attribuez-lui un CIDR de sous-réseau. Pour plus d'informations, consultez [Créer et gérer des réseaux VLAN/sous-réseaux](create-vlan-subnet.md). Il s'agit du réseau sur lequel toutes les machines virtuelles de bureau seront exécutées.

Suivez les meilleures pratiques de sécurité standard pour sécuriser votre déploiement Horizon :

* Autorisez uniquement le trafic RDP/SSH de bureau sur vos machines virtuelles de bureau.
* Autorisez uniquement le trafic de gestion entre le réseau VLAN du plan de gestion Horizon et le réseau VLAN du pool de bureaux.
* Autorisez uniquement le trafic de gestion à partir du réseau local.

Vous pouvez appliquer ces meilleures pratiques en configurant des [règles de pare-feu](firewall.md) à partir du portail CloudSimple.

#### <a name="cloudsimple-portal-configure-firewall-rules-to-secure-horizon-management-plane"></a>Portail CloudSimple : Configurer des règles de pare-feu pour sécuriser le plan de gestion Horizon

Configurez les règles suivantes sur le portail CloudSimple. Pour plus d'informations, consultez [Configurer des règles et tables de pare-feu](firewall.md).

1. Configurez des règles de pare-feu sur le pare-feu N-S de CloudSimple pour permettre la communication entre les sous-réseaux locaux et le réseau VLAN de gestion Horizon, de manière à ce que seuls les ports réseau répertoriés dans le document VMware [Liste des ports Horizon](https://docs.vmware.com/en/VMware-Horizon-7/7.1/com.vmware.horizon-client-agent.security.doc/GUID-52807839-6BB0-4727-A9C7-EA73DE61ADAB.html) soient autorisés.

2. Créez des règles de pare-feu E-O entre le réseau VLAN de gestion Horizon et le réseau VLAN du pool de bureaux dans le cloud privé.

#### <a name="cloudsimple-portal-create-a-public-ip-address-for-unified-access-gateway"></a>Portail CloudSimple : Créer une adresse IP publique pour Unified Access Gateway

Créez une adresse IP publique pour l'appliance Unified Access Gateway afin d'activer les connexions clientes de bureau à partir d'Internet. Pour plus d'informations, consultez [Allouer des adresses IP publiques](public-ips.md).

Au terme de l'installation, l'adresse IP publique est attribuée et répertoriée sur la page Adresses IP publiques.

#### <a name="cloudsimple-portal-escalate-privileges"></a>Portail CloudSimple : Élever les privilèges

L'utilisateur « cloudowner » par défaut ne dispose pas des privilèges suffisants pour installer Horizon dans l'instance de vCenter du cloud privé ; les privilèges vCenter de l'utilisateur doivent donc être élevés. Pour plus d'informations, consultez [Élever les privilèges](escalate-private-cloud-privileges.md).

#### <a name="vcenter-ui-create-a-user-in-private-cloud-for-horizon-installation"></a>Interface utilisateur de vCenter : Créer un utilisateur dans le cloud privé pour l'installation d'Horizon

1. Connectez-vous à vCenter à l'aide des informations d'identification de l'utilisateur « cloudowner ».
2. Créez un nouvel utilisateur, « horizon-soln-admin », dans vCenter et ajoutez-le au groupe Administrateurs de vCenter.
3. Déconnectez-vous de vCenter en tant qu'utilisateur « cloudowner » et reconnectez-vous en tant qu'utilisateur « horizon-soln-admin ».

#### <a name="vcenter-ui-install-vmware-horizon"></a>Interface utilisateur de vCenter : Installer VMware Horizon

Comme mentionné précédemment dans la section consacrée à l'architecture logique, la solution Horizon contient les composants suivants :

* VMware Horizon View
* VMware Unified Access Gateway
* VMware App Volume Manager
* VMware User Environment Manager

Installez les composants comme suit :

1. Installez et configurez Unified Access Gateway en suivant les instructions fournies dans le document VMware [Déploiement et configuration de VMware Unified Access Gateway](https://docs.vmware.com/en/Unified-Access-Gateway/3.3.1/com.vmware.uag-331-deploy-config.doc/GUID-F5CE0D5E-BE85-4FA5-BBCF-0F86C9AB8A70.html).

2. Installez Horizon View dans le cloud privé en suivant les instructions du [Guide d'installation de View](https://docs.vmware.com/en/VMware-Horizon-7/7.4/horizon-installation/GUID-37D39B4F-5870-4188-8B11-B6C41AE9133C.html).

3. Installez App Volume Manager en suivant les instructions de la section [Installer et configurer VMware App Volumes](https://docs.vmware.com/en/VMware-App-Volumes/2.10/com.vmware.appvolumes.user.doc/GUID-5E8BAF8C-F5A6-412C-9424-266BA7109BA4.html).

4. Installez et configurez User Environment Manager en suivant les instructions de la section [À propos de l'installation et de la configuration de VMware User Environment Manager](https://docs.vmware.com/en/VMware-User-Environment-Manager/9.4/com.vmware.user.environment.manager-install-config/GUID-DBBC82E4-483F-4B28-9D49-4D28E08715BC.html).

#### <a name="file-a-support-request-to-upload-vmware-horizon-pre-packaged-app-volumes"></a>Ouvrir une demande de support pour le chargement des volumes d'applications prépackagés VMware Horizon

Dans le cadre du processus d'installation, App Volume Manager utilise des volumes prépackagés pour approvisionner des piles d'applications et des volumes accessibles en écriture. Ces volumes servent de modèles pour les piles d'applications et les volumes accessibles en écriture.

Le chargement des volumes dans le magasin de données du cloud privé requiert le mot de passe racine ESXi. Pour obtenir de l'aide, envoyez une [demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). Joignez le pack d'installation d'AppVolumes pour permettre au personnel de support de CloudSimple de charger les modèles dans votre environnement de cloud privé.

#### <a name="cloudsimple-portal-de-escalate-privileges"></a>Portail CloudSimple : Annuler l’élévation des privilèges

Vous pouvez maintenant [annuler l'élévation des privilèges](escalate-private-cloud-privileges.md#de-escalate-privileges) de l'utilisateur « cloudowner ».

## <a name="ongoing-management-of-your-horizon-solution"></a>Gestion continue de votre solution Horizon

Vous disposez d'un contrôle total sur les logiciels Horizon et App Volume Manager de votre environnement de cloud privé, et vous êtes tenu de procéder à la gestion nécessaire du cycle de vie de ceux-ci. Assurez-vous que toute nouvelle version des logiciels est compatible avec les instances de vCenter et PSC du cloud privé avant de mettre à jour ou de mettre à niveau Horizon ou App Volume.
