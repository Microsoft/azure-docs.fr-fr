---
title: Didacticiel - Déployer et configurer VMware HCX
description: Découvrez comment déployer et configurer une solution VMware HCX (Hybrid Cloud Extension) pour votre cloud privé Azure VMware Solution.
ms.topic: tutorial
ms.date: 11/25/2020
ms.openlocfilehash: a792f8dbc646f61377cf0a88e1a6e386340f23e8
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97357889"
---
# <a name="deploy-and-configure-vmware-hcx"></a>Déployer et configurer VMware HCX

Cet article vous montre comment déployer et configurer le connecteur VMware HCX en local pour votre cloud privé Azure VMware Solution. Avec VMware HCX, vous pouvez migrer vos charges de travail VMware vers Azure VMware Solution et d’autres sites connectés, par le biais de différents types de migration. Étant donné que la solution VMware Azure déploie et configure le Gestionnaire de clouds HCX, vous devez télécharger, activer et configurer le connecteur HCX dans votre centre de données VMware local.

Le connecteur VMware HCX Advanced est prédéployé dans Azure VMware Solution. Il prend en charge jusqu’à trois connexions de site (localement vers cloud, ou cloud vers cloud). Si vous avez besoin de plus de trois connexions de site, soumettez une [demande de support](https://portal.azure.com/#create/Microsoft.Support) pour activer le module complémentaire [VMware HCX Enterprise](https://cloud.vmware.com/community/2019/08/08/introducing-hcx-enterprise/) . Le module complémentaire est actuellement en préversion. 

>[!TIP]
>Bien que l’outil VMware Configuration Maximum décrive un nombre maximal de 25 paires de sites entre le connecteur local et le gestionnaire de cloud, la gestion des licences limite ce nombre à trois pour Advanced Edition et à 10 pour Enterprise Edition.

>[!NOTE]
>VMware HCX Enterprise est disponible avec Azure VMware Solution sous forme de service en préversion. Il est gratuit et est soumis aux conditions générales d’un service en préversion. Faisant suite à la mise en disponibilité générale du service VMware HCX Enterprise, vous recevrez un préavis de 30 jours vous informant que la facturation sera basculée. Vous aurez également la possibilité de désactiver le service ou de le quitter. Il n’existe pas de chemin de mise à niveau simple, de VMware HCX Enterprise vers VMware HCX Advanced. Si vous décidez de passer à une version antérieure, vous devrez redéployer, ce qui entraînera un temps d’arrêt.

Commencez par passer en revue les sections [Avant de commencer](#before-you-begin), [Exigences de la version logicielle](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-54E5293B-8707-4D29-BFE8-EE63539CC49B.html) et [Prérequis](#prerequisites). 

Ensuite, nous parcourrons pas à pas toutes les procédures nécessaires pour :

> [!div class="checklist"]
> * Télécharger le fichier OVA du connecteur VMware HCX.
> * Déployer le fichier OVA VMware HCX localement (connecteur VMware HCX).
> * Activer le connecteur VMware HCX.
> * Associer votre connecteur VMware HCX local à votre Gestionnaire de clouds HCX d’Azure VMware Solution.
> * Configurer l’interconnexion (profil réseau, profil de calcul et maille de services).
> * Terminer l’installation en contrôlant l’état de l’appliance et en vérifiant que la migration est possible.

L’installation terminée, suivez les prochaines étapes recommandées à la fin de cet article.  

## <a name="before-you-begin"></a>Avant de commencer

Lorsque vous préparez votre déploiement, nous vous recommandons de consulter la documentation VMware suivante :

* [Guide de l’utilisateur VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-E456F078-22BE-494B-8E4B-076EF33A9CF4.html)
* [Migration des machines virtuelles avec VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-D0CD0CC6-3802-42C9-9718-6DA5FEC246C6.html?hWord=N4IghgNiBcIBIGEAaACAtgSwOYCcwBcMB7AOxAF8g)
* [Points à prendre en considération pour le déploiement VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/install-checklist/GUID-C0A0E820-D5D0-4A3D-AD8E-EEAA3229F325.html)
* [Série de blogs VMware – Migration vers le cloud](https://blogs.vmware.com/vsphere/2019/10/cloud-migration-series-part-2.html) 
* [Ports réseau nécessaires à VMware HCX](https://ports.vmware.com/home/VMware-HCX)


## <a name="prerequisites"></a>Prérequis

Si vous envisagez d’utiliser VMware HCX Enterprise, vérifiez que vous avez demandé l’activation par le biais des canaux du support Azure VMware Solution.


### <a name="on-premises-vsphere-environment"></a>Environnement vSphere local

Assurez-vous que votre environnement vSphere local (environnement source) répond à la [configuration minimale demandée](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-54E5293B-8707-4D29-BFE8-EE63539CC49B.html). 

### <a name="network-and-ports"></a>Réseau et ports

* [Azure ExpressRoute Global Reach](tutorial-expressroute-global-reach-private-cloud.md) est configuré entre les circuits locaux et les circuits ExpressRoute du SDDC d’Azure VMware Solution.

* [Tous les ports obligatoires](https://ports.vmware.com/home/VMware-HCX) sont ouverts pour la communication entre les composants locaux et le SDDC Azure VMware Solution.

### <a name="ip-addresses"></a>Adresses IP

[!INCLUDE [hcx-network-segments](includes/hcx-network-segments.md)]
   
## <a name="download-the-vmware-hcx-connector-ova"></a>Télécharger le fichier OVA du connecteur VMware HCX

Avant de déployer l’appliance virtuelle sur votre vCenter local, vous devez télécharger le fichier OVA du connecteur VMware HCX.  

1. Dans le portail Azure, sélectionnez le cloud privé Azure VMware Solution. 

1. Sélectionnez **Gérer** > **Connectivité**, puis sélectionnez l’onglet **HCX** pour identifier l’adresse IP du Gestionnaire HCX d’Azure VMware Solution. 

   :::image type="content" source="media/tutorial-vmware-hcx/find-hcx-ip-address.png" alt-text="Capture d’écran de l’adresse IP VMware HCX." lightbox="media/tutorial-vmware-hcx/find-hcx-ip-address.png":::

1. Sélectionnez **Gérer** > **Identité** et sélectionnez **Mot de passe de l’administrateur du vCenter** pour identifier le mot de passe.

   > [!TIP]
   > Le mot de passe vCenter a été défini lorsque vous avez configuré le cloud privé. Il s’agit du même mot de passe que celui que vous utiliserez pour vous connecter au Gestionnaire HCX d’Azure VMware Solution.

   :::image type="content" source="media/tutorial-vmware-hcx/hcx-admin-password.png" alt-text="Rechercher le mot de passe HCX." lightbox="media/tutorial-vmware-hcx/hcx-admin-password.png":::

1. Ouvrez une fenêtre de navigateur, connectez-vous au Gestionnaire HCX d’Azure VMware Solution sur le port `https://x.x.x.9` 443, avec les informations d’identification de l’utilisateur **cloudadmin\@vsphere.local**.

1. Sélectionnez **Administration** > **System Updates** puis sélectionnez **Request Download Link** (Demander le lien de téléchargement).

1. Sélectionnez l’option de votre choix pour télécharger le fichier OVA du connecteur VMware HCX.

## <a name="deploy-the-vmware-hcx-connector-ova-on-premises"></a>Déployer le fichier OVA de VMware HCX Connector en local

1. Dans votre vCenter local, sélectionnez un [modèle OVF](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vm_admin.doc/GUID-17BEDA21-43F6-41F4-8FB2-E01D275FE9B4.html) pour déployer le connecteur VMware HCX sur votre vCenter local. 

   > [!TIP]
   > Choisissez le fichier OVA que vous avez téléchargé à la section précédente.  

   :::image type="content" source="media/tutorial-vmware-hcx/select-ovf-template.png" alt-text="Capture d’écran de navigation vers un modèle OVF." lightbox="media/tutorial-vmware-hcx/select-ovf-template.png":::


1. Sélectionnez un nom et un emplacement, puis sélectionnez une ressource ou un cluster où déployer le connecteur VMware HCX. Ensuite, consultez les détails et les ressources nécessaires, puis sélectionnez **Next** (Suivant).  

1. Passez en revue les termes du contrat de licence. Si vous les acceptez, sélectionnez le stockage et le réseau nécessaires, puis sélectionnez **Next** (Suivant).

1. Sélectionnez le stockage, puis sélectionnez **Next**.

1. Sélectionnez le segment réseau de la gestion VMware HCX, que vous avez défini précédemment à la section [Prérequis des adresses IP](#ip-addresses).  Ensuite, sélectionnez **Suivant**.

1. Dans **Customize template** (Personnaliser le modèle), entrez toutes les informations demandées, puis sélectionnez **Next**. 

   :::image type="content" source="media/tutorial-vmware-hcx/customize-template.png" alt-text="Capture d’écran des zones de personnalisation d’un modèle." lightbox="media/tutorial-vmware-hcx/customize-template.png":::

1. Vérifiez la configuration, puis sélectionnez **Finish** (Terminer) pour déployer le fichier OVA du connecteur VMware HCX.
   
   > [!IMPORTANT]
   > Vous devrez allumer manuellement l’appliance virtuelle.  Après la mise sous tension, patientez 10 à 15 minutes avant de passer à l’étape suivante.

Pour une vue d’ensemble de bout en bout de cette procédure, consultez la vidéo [Azure VMware Solution : Déploiement de l’appliance HCX](https://www.youtube.com/embed/UKmSTYrL6AY). 


## <a name="activate-vmware-hcx"></a>Activer VMware HCX

Une fois que vous avez déployé le fichier OVA du connecteur VMware HCX en local, et démarré l’appliance, vous êtes prêt à procéder à l’activation. Vous devez tout d’abord vous procurer une clé de licence sur le portail Azure VMware Solution.

1. Dans le portail Azure VMware Solution, accédez à **Manage** (Gérer) > **Connectivity**, sélectionnez l’onglet **HCX**, puis sélectionnez **Add** (Ajouter).

1. Utilisez les informations d’identification de l’**administrateur** pour vous connecter au Gestionnaire de clouds VMware HCX en local sur `https://HCXManagerIP:9443`. 

   > [!TIP]
   > Vous avez défini le mot de passe de l’utilisateur **administrateur** lors du déploiement du fichier OVA du Gestionnaire VMware HCX.

   > [!IMPORTANT]
   > Veillez à inclure le numéro de port `9443` avec l’adresse IP de VMware HCX Manager.

1. Dans **Licensing** (Gestion des licences), entrez votre clé pour **HCX Advanced Key** (Clé HCX Advanced) et sélectionnez **Activate**.  
   
    > [!NOTE]
    > VMware HCX Manager doit avoir un accès Internet ouvert ou un proxy configuré.

1. Dans **Datacenter Location** (Emplacement du centre de données), indiquez l’emplacement le plus proche pour l’installation du Gestionnaire de clouds VMware HCX en local. Sélectionnez **Continuer**.

1. Dans **System Name** (Nom du système), modifiez ou acceptez le nom par défaut et sélectionnez **Continue**.
   
1. Sélectionnez **Yes, Continue** (Oui, continuer).

1. Dans **Connecter votre vCenter**, fournissez le nom de domaine complet ou l’adresse IP de votre serveur vCenter, ainsi que les informations d’identification appropriées, puis sélectionnez **Continuer**.
   
   > [!TIP]
   > Le serveur vCenter est l’emplacement où vous avez déployé le connecteur VMware HCX dans votre centre de données.

1. Dans **Configure SSO/PSC**, indiquez le nom de domaine complet ou l’adresse IP de votre instance Platform Services Controller (PSC), puis sélectionnez **Continue**.
   
   > [!NOTE]
   > En règle générale, il s’agit du nom de domaine complet ou de l’adresse IP de votre vCenter.

1. Vérifiez que les informations indiquées sont correctes, puis sélectionnez **Restart** (Redémarrer).
    
   > [!NOTE]
   > Un certain délai s’écoulera après le redémarrage avant que vous soyez invité à passer à l’étape suivante.

Après le redémarrage des services, vous devez voir l’état de vCenter apparaître en vert dans l’écran qui s’affiche. VCenter et SSO doivent être dotés des paramètres de configuration appropriés : ceux-ci doivent être identiques à ceux de l’écran précédent.

:::image type="content" source="media/tutorial-vmware-hcx/activation-done.png" alt-text="Capture d’écran du tableau de bord avec l’état vCenter vert." lightbox="media/tutorial-vmware-hcx/activation-done.png":::  

Pour une vue d’ensemble de bout en bout de cette procédure, consultez la vidéo [Azure VMware Solution : Activer HCX](https://www.youtube.com/embed/PnVg6SZkQsY?rel=0&amp;vq=hd720).

   > [!IMPORTANT]
   > Que vous utilisiez VMware HCX Advanced ou VMware HCX Enterprise, vous devrez peut-être installer le correctif à partir de l’[article 81558 de la Base de connaissances](https://kb.vmware.com/s/article/81558) de VMware. 

## <a name="configure-the-vmware-hcx-connector"></a>Configurer le connecteur VMware HCX

Vous êtes maintenant prêt à ajouter un couplage de sites, à créer un réseau et un profil de calcul, et à activer des services tels que la migration, l’extension réseau ou la reprise d’activité. 

### <a name="add-a-site-pairing"></a>Ajouter un appairage de site

Vous pouvez connecter ou appairer le gestionnaire de cloud VMware HCX d’Azure VMware Solution au connecteur VMware HCX de votre centre de données. 

1. Connectez-vous au vCenter local puis, sous **Accueil**, sélectionnez **HCX**.

1. Sous **Infrastructure**, sélectionnez **Site Pairing** (Couplage de sites), puis sélectionnez l’option **Connect To Remote Site** (Connexion au site distant) (au milieu de l’écran). 

1. Entrez l’adresse IP ou l’URL du Gestionnaire de clouds HCX d’Azure VMware Solution que vous avez notée précédemment `https://x.x.x.9`, le nom d’utilisateur Azure VMware Solution cloudadmin@vsphere.local et le mot de passe associé. Sélectionnez **Connecter**.

   > [!NOTE]
   > Pour établir une paire de sites :
   > * Votre connecteur VMware HCX doit pouvoir router vers l’IP de votre gestionnaire de cloud HCX via le port 443.
   >
   > * Utilisez le mot de passe qui vous a servi à vous connecter à vCenter. Vous avez défini ce mot de passe sur l’écran de déploiement initial.

   Un écran s’affiche montrant que votre gestionnaire de cloud VMware HCX d’Azure VMware Solution et votre connecteur HCX local sont connectés (appairés).

   :::image type="content" source="media/tutorial-vmware-hcx/site-pairing-complete.png" alt-text="Capture d’écran présentant l’appairage du gestionnaire HCX d’Azure VMware Solution et du connecteur HCX.":::

Pour une vue d’ensemble de bout en bout de cette procédure, consultez la vidéo [Azure VMware Solution : Couplage de sites HCX](https://www.youtube.com/embed/jXOmYUnbWZY?rel=0&amp;vq=hd720).

### <a name="create-network-profiles"></a>Créer des profils réseau

Le connecteur VMware HCX déploie un sous-ensemble d’appliances virtuelles (automatisées) qui nécessitent plusieurs segments IP. Lorsque vous créez vos profils réseau, vous utilisez les segments IP que vous avez identifiés lors de la [phase de préparation et de planification du prédéploiement des segments de réseau VMware HCX](production-ready-deployment-steps.md#vmware-hcx-network-segments).

Vous allez créer quatre profils réseau :

   - Gestion
   - vMotion
   - Réplication
   - Liaison montante

1. Sous **Infrastructure**, sélectionnez **Interconnexion** > **Maillage de service multisite** > **Profils réseau** > **Créer un profil réseau**.

   :::image type="content" source="media/tutorial-vmware-hcx/network-profile-start.png" alt-text="Capture d’écran de sélections permettant de commencer à créer un profil réseau." lightbox="media/tutorial-vmware-hcx/network-profile-start.png":::

1. Pour chaque profil réseau, sélectionnez le réseau ainsi que le groupe de ports, puis fournissez un nom et créez le pool d’adresses IP du segment. Sélectionnez ensuite **Créer**. 

   :::image type="content" source="media/tutorial-vmware-hcx/example-configurations-network-profile.png" alt-text="Capture d’écran des détails d’un nouveau profil réseau.":::

Pour une vue d’ensemble de bout en bout de cette procédure, consultez la vidéo [Azure VMware Solution : Profil réseau HCX](https://www.youtube.com/embed/O0rU4jtXUxc).


### <a name="create-a-compute-profile"></a>Créer un profil de calcul

1. Sous **Infrastructure**, sélectionnez **Interconnect** (Interconnexion) > **Compute Profiles**(Profils de calcul) > **Create Compute Profile (Créer un profil de calcul)** .

   :::image type="content" source="media/tutorial-vmware-hcx/compute-profile-create.png" alt-text="Capture d’écran affichant les sélections qui permettent de commencer à créer un profil de calcul." lightbox="media/tutorial-vmware-hcx/compute-profile-create.png":::

1. Entrez un nom de profil de calcul et sélectionnez **Continuer**.  

   :::image type="content" source="media/tutorial-vmware-hcx/name-compute-profile.png" alt-text="Capture d’écran affichant l’entrée d’un nom de profil de calcul, et le bouton Continue." lightbox="media/tutorial-vmware-hcx/name-compute-profile.png":::

1. Sélectionnez les services à activer, comme la migration, l’extension réseau ou la reprise d’activité, puis sélectionnez **Continue**.
  
   > [!NOTE]
   > En général, rien ne change ici.

1. Dans **Select Service Resources** (Sélectionner des ressources de service), sélectionnez une ou plusieurs ressources de service (clusters) pour lesquelles les services HCX sélectionnés doivent être activés.  

1. Lorsque vous voyez les clusters dans votre centre de données local, sélectionnez **Continuer**.

   :::image type="content" source="media/tutorial-vmware-hcx/select-service-resource.png" alt-text="Capture d’écran montrant les ressources de service sélectionnées, et le bouton Continue." lightbox="media/tutorial-vmware-hcx/select-service-resource.png":::

1. Dans **Select Datastore** (Sélectionner un magasin de données), sélectionnez la ressource de stockage du magasin de données pour déployer les appliances d’interconnexion VMware HCX. Sélectionnez **Continuer**.

   Lorsque plusieurs ressources sont sélectionnées, VMware HCX utilise la première ressource sélectionnée jusqu’à épuisement de sa capacité.   

   :::image type="content" source="media/tutorial-vmware-hcx/deployment-resources-and-reservations.png" alt-text="Capture d’écran montrant la sélection d’une ressource de stockage de données, et celle du bouton Continue." lightbox="media/tutorial-vmware-hcx/deployment-resources-and-reservations.png":::  

1. Dans **Select Management Network Profile**, sélectionnez le profil réseau de gestion que vous avez créé au cours des étapes précédentes. Sélectionnez **Continuer**.  

   :::image type="content" source="media/tutorial-vmware-hcx/select-management-network-profile.png" alt-text="Capture d’écran montrant la sélection d’un profil réseau de gestion, et celle du bouton Continue." lightbox="media/tutorial-vmware-hcx/select-management-network-profile.png":::

1. Dans **Select Uplink Network Profile**, sélectionnez le profil réseau de liaison montante que vous avez créé au cours de la procédure précédente. Sélectionnez **Continuer**.

   :::image type="content" source="media/tutorial-vmware-hcx/select-uplink-network-profile.png" alt-text="Capture d’écran montrant la sélection d’un profil réseau de liaison montante, et celle du bouton Continue." lightbox="media/tutorial-vmware-hcx/select-uplink-network-profile.png":::

1. Dans **Select Motion Network Profile**, sélectionnez le profil réseau vMotion que vous avez créé au cours des étapes précédentes. Sélectionnez **Continuer**.

   :::image type="content" source="media/tutorial-vmware-hcx/select-vmotion-network-profile.png" alt-text="Capture d’écran montrant la sélection d’un profil réseau vMotion, et celle du bouton Continue." lightbox="media/tutorial-vmware-hcx/select-vmotion-network-profile.png":::

1. Dans **Select vSphere Replication Network Profile**, sélectionnez le profil réseau de réplication que vous avez créé au cours des étapes précédentes. Sélectionnez **Continuer**.

   :::image type="content" source="media/tutorial-vmware-hcx/select-replication-network-profile.png" alt-text="Capture d’écran montrant la sélection d’un profil réseau de réplication, et celle du bouton Continue." lightbox="media/tutorial-vmware-hcx/select-replication-network-profile.png":::

1. Dans **Select Distributed Switches for Network Extensions**, sélectionnez les commutateurs qui contiennent les machines virtuelles à migrer vers Azure VMware Solution sur un réseau étendu de couche 2. Sélectionnez **Continuer**.

   > [!NOTE]
   > Si vous ne migrez pas de machines virtuelles sur des réseaux étendus de couche 2, vous pouvez ignorer cette étape.
   
   :::image type=" content" source="media/tutorial-vmware-hcx/select-layer-2-distributed-virtual-switch.png" alt-text="Capture d’écran montrant la sélection des commutateurs virtuels distribués, et celle du bouton Continue." lightbox="media/tutorial-vmware-hcx/select-layer-2-distributed-virtual-switch.png":::

1. Passez en revue les règles de connexion et sélectionnez **Continuer**.  

   :::image type="content" source="media/tutorial-vmware-hcx/review-connection-rules.png" alt-text="Capture d’écran montrant les règles de connexion et le bouton Continue." lightbox="media/tutorial-vmware-hcx/review-connection-rules.png":::

1. Sélectionnez **Terminer** pour créer le profil de calcul.


   :::image type="content" source="media/tutorial-vmware-hcx/compute-profile-done.png" alt-text="Capture d’écran affichant des informations de profil de calcul." lightbox="media/tutorial-vmware-hcx/compute-profile-done.png":::

Pour une vue d’ensemble de bout en bout de cette procédure, consultez la vidéo [Azure VMware Solution : Profil de calcul](https://www.youtube.com/embed/e02hsChI3b8).

### <a name="create-a-service-mesh"></a>Créer une maille de services.

À présent, configurez une maille de services entre le SDDC local et le SDDC Azure VMware Solution.



> [!NOTE]
> Pour établir correctement une maille de services avec VMware Azure Solution :
>
> * Les ports UDP 500/4500 sont ouverts entre les adresses du profil réseau « liaison montante » de votre connecteur VMware HCX local et les adresses du profil réseau « liaison montante » du cloud HCX d’Azure VMware Solution.
>
> * Veillez à examiner les [ports obligatoires VMware HCX](https://ports.vmware.com/home/VMware-HCX).

1. Sous **Infrastructure**, sélectionnez **Interconnexion** > **Maillage du service** > **Créer le maillage du service**.    

   :::image type="content" source="media/tutorial-vmware-hcx/create-service-mesh.png" alt-text="Capture d’écran des sélections permettant de commencer à créer une maille de services." lightbox="media/tutorial-vmware-hcx/create-service-mesh.png":::

1. Passez en revue les sites préremplis, puis sélectionnez **Continuer**. 

   > [!NOTE]
   > S’il s’agit de votre première configuration de maillage de service, vous ne devez pas modifier cet écran.  

1. Sélectionnez les profils de calcul source et distant dans les listes déroulantes, puis sélectionnez **Continue**.  

   Les sélections définissent les ressources où les machines virtuelles peuvent consommer des services VMware HCX.  

   :::image type="content" source="media/tutorial-vmware-hcx/select-compute-profile-source.png" alt-text="Capture d’écran montrant la sélection du profil de calcul source." lightbox="media/tutorial-vmware-hcx/select-compute-profile-source.png":::

   :::image type="content" source="media/tutorial-vmware-hcx/select-compute-profile-remote.png" alt-text="Capture d’écran montrant la sélection du profil de calcul distant." lightbox="media/tutorial-vmware-hcx/select-compute-profile-remote.png":::

1. Passez en revue les services qui seront activés, puis sélectionnez **Continue**.  

1. Dans **Advanced Configuration - Override Uplink Network profiles** (Configuration avancée - Remplacer les profils réseau de liaison montante), sélectionnez **Continue**.  

   Les profils réseau de liaison montante se connectent au réseau par le biais duquel les appliances d’interconnexion du site distant peuvent être jointes.  
  
1. Dans **Configuration avancée - Scale Out des appliances d’extension réseau**, passez les informations en revue et sélectionnez **Continuer**. 

   Vous pouvez disposer de huit VLAN par appliance au maximum, mais vous pouvez déployer une autre appliance pour en ajouter huit de plus. Vous devez aussi disposer d’un espace IP pour prendre en compte les autres appliances, et il faut compter une IP par appliance.  Pour plus d’informations, consultez [VMware HCX Configuration Limits](https://configmax.vmware.com/guest?vmwareproduct=VMware%20HCX&release=VMware%20HCX&categories=41-0,42-0,43-0,44-0,45-0).
   
   :::image type="content" source="media/tutorial-vmware-hcx/extend-networks-increase-vlan.png" alt-text="Capture d’écran montrant où augmenter le nombre VLAN." lightbox="media/tutorial-vmware-hcx/extend-networks-increase-vlan.png":::

1. Dans **Advanced Configuration - Traffic Engineering** (Configuration avancée - Ingénierie du trafic), examinez et apportez les modifications que vous estimez nécessaires, puis sélectionnez **Continue**.

1. Passez en revue l’aperçu de la topologie, puis sélectionnez **Continuer**.

1. Entrez un nom convivial pour ce maillage de service et sélectionnez **Terminer**.  

1. Sélectionnez **Afficher les tâches** pour surveiller le déploiement. 

   :::image type="content" source="media/tutorial-vmware-hcx/monitor-service-mesh.png" alt-text="Capture d’écran montrant le bouton d’affichage des tâches.":::

   Lorsque le déploiement de la maille de services se termine correctement, les services s’affichent en vert.

   :::image type="content" source="media/tutorial-vmware-hcx/service-mesh-green.png" alt-text="Capture d’écran montrant des indicateurs verts sur les services." lightbox="media/tutorial-vmware-hcx/service-mesh-green.png":::

1. Vérifiez l’intégrité de la maille de services en contrôlant l’état de l’appliance. 

1. Sélectionnez **Interconnect** > **Appliances**.

   :::image type="content" source="media/tutorial-vmware-hcx/interconnect-appliance-state.png" alt-text="Capture d’écran affichant les sélections pour la vérification de l’état de l’appliance." lightbox="media/tutorial-vmware-hcx/interconnect-appliance-state.png":::

Pour une vue d’ensemble de bout en bout de cette procédure, consultez la vidéo [Azure VMware Solution : La maille de services](https://www.youtube.com/embed/COY3oIws108).

### <a name="optional-create-a-network-extension"></a>(Facultatif) Créer une extension réseau

Si vous souhaitez étendre un réseau de votre environnement local vers Azure VMware Solution, suivez ces étapes :

1. Sous **Services**, sélectionnez **Network Extension**(Extension réseau) > **Create a Network Extension** (Créer une extension réseau).

   :::image type="content" source="media/tutorial-vmware-hcx/create-network-extension.png" alt-text="Capture d’écran affichant les sélections qui permettent de commencer à créer une extension réseau." lightbox="media/tutorial-vmware-hcx/create-network-extension.png":::

1. Sélectionnez chaque réseau que vous souhaitez étendre vers Azure VMware Solution, puis sélectionnez **Next** (Suivant).

   :::image type="content" source="media/tutorial-vmware-hcx/select-extend-networks.png" alt-text="Capture d’écran présentant la sélection d’un réseau.":::

1. Entrez l’adresse IP de la passerelle locale pour chacun des réseaux que vous étendez, puis sélectionnez **Soumettre**. 

   :::image type="content" source="media/tutorial-vmware-hcx/extend-networks-gateway.png" alt-text="Capture d’écran montrant l’entrée d’une adresse IP de passerelle.":::

   L’extension d’un réseau peut prendre quelques minutes. Quand l’opération est terminée, l’état change et passe à **Extension complete** (Extension terminée).

   :::image type="content" source="media/tutorial-vmware-hcx/extension-complete.png" alt-text="Capture d’écran montrant l’état d’extension terminée." lightbox="media/tutorial-vmware-hcx/extension-complete.png":::

Pour une vue d’ensemble de bout en bout de cette procédure, consultez la vidéo [Azure VMware Solution : Extension de réseau](https://www.youtube.com/embed/gYR0nftKui0).


## <a name="next-steps"></a>Étapes suivantes

Si l’état du tunnel d’interconnexion de l’appliance est **Activé** et vert, vous pouvez migrer et protéger des machines virtuelles Azure VMware Solution en utilisant VMware HCX. Azure VMware Solution prend en charge les migrations de charge de travail (avec ou sans extension réseau). Vous pouvez toujours migrer des charges de travail dans votre environnement vSphere, en même temps que la création locale de réseaux et le déploiement de machines virtuelles sur ces réseaux.  

Pour plus d’informations sur l’utilisation de HCX, accédez à la documentation technique de VMware :

* [Documentation VMware HCX](https://docs.vmware.com/en/VMware-HCX/index.html)
* [Migration des machines virtuelles avec VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-D0CD0CC6-3802-42C9-9718-6DA5FEC246C6.html?hWord=N4IghgNiBcIBIGEAaACAtgSwOYCcwBcMB7AOxAF8g)
* [Ports HCX obligatoires](https://ports.vmware.com/home/VMware-HCX)
