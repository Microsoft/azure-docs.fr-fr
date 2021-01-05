---
title: Forum aux questions
description: Apporte des réponses à des questions récurrentes à propos d’Azure VMware Solution.
ms.topic: conceptual
ms.date: 11/19/2020
ms.author: dikamath
ms.openlocfilehash: 3033df2021a8d1780caf9b0b7cd4dbe8de2a6050
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861406"
---
# <a name="frequently-asked-questions-about-azure-vmware-solution"></a>Questions fréquentes sur Azure VMware Solution

Cette page répond aux questions fréquentes à propos d’Azure VMware Solution.

## <a name="general"></a>Général

#### <a name="what-is-azure-vmware-solution"></a>Présentation d’Azure VMware Solution

Tandis que les entreprises mettent en œuvre des stratégies de modernisation informatique pour améliorer la réactivité, réduire les coûts et accélérer l’innovation, les plateformes cloud hybrides apparaissent comme des activateurs clés de la transformation numérique des clients. Azure VMware Solution combine le logiciel SDDC (Software Defined Data Center) de VMware avec l’écosystème du service cloud global de Microsoft Azure. Azure VMware Solution est managée pour satisfaire à une série d’exigences en matière de performances, de disponibilité, de sécurité et de conformité.

## <a name="azure-vmware-solution-service"></a>Service Azure VMware Solution

#### <a name="where-is-azure-vmware-solution-available-today"></a>Où Azure VMware Solution est-elle disponible aujourd’hui ?

Le service est régulièrement ajouté à de nouvelles régions. Pour en savoir plus, consultez les [dernières informations de disponibilité du service](https://azure.microsoft.com/global-infrastructure/services/?products=azure-vmware). 

#### <a name="can-workloads-running-in-an-azure-vmware-solution-instance-consume-or-integrate-with-azure-services"></a>Des charges de travail exécutées dans une instance Azure VMware Solution peuvent-elles utiliser des services Azure ou s’intégrer avec ceux-ci ?

Tous les services Azure sont à la disposition des clients d’Azure VMware Solution. Les limitations de performances et de disponibilité pour des services spécifiques doivent être traitées au cas par cas.

#### <a name="do-i-use-the-same-tools-that-i-use-now-to-manage-private-cloud-resources"></a>Dois-je utiliser les mêmes outils que ceux que j’utilise actuellement pour gérer des ressources de cloud privé ?

Oui. Le portail Azure est utilisé pour le déploiement et plusieurs opérations de gestion. vCenter et NSX Manager sont utilisés pour gérer les ressources vSphere et NSX-T.

#### <a name="can-i-manage-a-private-cloud-with-my-on-premises-vcenter"></a>Puis-je gérer un cloud privé avec mon serveur VMware vCenter local ?

Au lancement, Azure VMware Solution ne prend pas en charge une expérience de gestion unique dans des environnements de cloud local et privé. Les clusters de clouds privés sont gérés avec les logiciels vCenter et NSX Manager locaux d’un cloud privé.

#### <a name="can-i-use-vrealize-suite-running-on-premises"></a>Puis-je exécuter la solution vRealize Suite en local ? 

Les intégrations et cas d’usage spécifiques peuvent être évalués individuellement.

#### <a name="can-i-migrate-vsphere-vms-from-on-premises-environments-to-azure-vmware-solution-private-clouds"></a>Puis-je migrer des machines virtuelles vSphere à partir d’environnements locaux vers des clouds privés Azure VMware Solution ?

Oui. Vous pouvez utiliser une migration de machine virtuelle et vMotion pour déplacer des machines virtuelles vers un cloud privé si les configurations requises standard croisées de vCenter et de [vMotion](https://kb.vmware.com/s/article/2106952?lang=en_US&queryTerm=2106952) sont réunies.

#### <a name="is-a-specific-version-of-vsphere-required-in-on-premises-environments"></a>Une version spécifique de vSphere est-elle requise dans des environnements locaux ?

Tous les environnements cloud sont fournis avec VMware HCX et vSphere 5.5 ou versions ultérieures dans des environnements locaux pour vMotion.

#### <a name="what-does-the-change-control-process-look-like"></a>À quoi ressemble le processus de contrôle des modifications ?

Les mises à jour apportées au service proprement dit suivent le processus de gestion des changements standard de Microsoft Azure. Les clients sont responsables des tâches d’administration de la charge de travail, ainsi que des processus de gestion des modifications associés.

#### <a name="how-is-this-different-from-azure-vmware-solution-by-cloudsimple"></a>Quelle est la différence avec Azure VMware Solution by CloudSimple ?

Avec la nouvelle solution Azure VMware, Microsoft et VMware sont partenaires directs pour la prestation de services cloud. La nouvelle solution est entièrement conçue et prise en charge par Microsoft, et approuvée par VMware. Du point de l’architecture, les solutions sont cohérentes, la pile de technologies VMware s’exécutant sur une infrastructure Azure dédiée.



#### <a name="can-azure-vmware-solution-vms-be-managed-by-vmrc"></a>Les machines virtuelles Azure VMware Solution peuvent-elles être gérées par VMRC ?
Oui, à condition que le système sur lequel elles sont installées puisse accéder au cloud privé vCenter et qu’il utilise le service DNS public pour résoudre les noms d’hôtes ESXi.

#### <a name="are-there-special-instructions-for-installing-and-using-vmrc-with-azure-vmware-solution-vms"></a>Existe-t-il des instructions spéciales pour l’installation et l’utilisation de VMRC avec les machines virtuelles Azure VMware Solution ?
Non. Utilisez les [instructions fournies par VMware](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.vm_admin.doc/GUID-89E7E8F0-DB2B-437F-8F70-BA34C505053F.html) pour remplir les conditions préalables des machines virtuelles spécifiés dans ces instructions. 

#### <a name="is-vmware-hcx-supported-on-vpns"></a>VMware HCX est-il pris en charge sur les réseaux VPN ?
Non, en raison de la bande passante et de la latence requises.

#### <a name="can-azure-bastion-be-used-for-connecting-to-azure-vmware-solution-vms"></a>Le service Azure Bastion peut-il être utilisé pour la connexion à des machines virtuelles Azure VMware Solution ?
Azure Bastion est le service recommandé pour se connecter au serveur de rebond afin d’empêcher l’exposition à Internet d’Azure VMware Solution. Vous ne pouvez pas utiliser Azure Bastion pour vous connecter à des machines virtuelles Azure VMware Solution, car il ne s’agit pas d’objets IaaS Azure.

#### <a name="can-azure-load-balancer-internal-be-used-for-azure-vmware-solution-vms"></a>Azure Load Balancer interne peut-il être utilisé pour des machines virtuelles Azure VMware Solution ?
Non. Azure Load Balancer interne ne prend en charge que des machines virtuelles Azure IaaS. Azure Load Balancer ne prend pas en charge les pools principaux basés sur IP, mais uniquement des machines virtuelles Azure ou des objets de groupes de machines virtuelles identiques dans lesquels les machines virtuelles Azure VMware Solution ne sont pas des objets Azure.

#### <a name="can-an-existing-expressroute-gateway-be-used-to-connect-to-azure-vmware-solution"></a>Une passerelle ExpressRoute existante peut-elle être utilisée pour la connexion à Azure VMware Solution ?
Oui, vous pouvez utiliser une passerelle ExpressRoute existante pour vous connecter à Azure VMware Solution, à condition qu’elle ne dépasse pas la limite de quatre circuits ExpressRoute par réseau virtuel.  Toutefois, pour accéder à Azure VMware Solution à partir d’un site local via ExpressRoute, vous devez disposer d’ExpressRoute Global Reach, car la passerelle ExpressRoute ne fournit pas de routage transitif entre ses circuits connectés.

## <a name="compute-network-storage-and-backup"></a>Calcul, réseau, stockage et sauvegarde

#### <a name="is-there-more-than-one-type-of-host-available"></a>Plusieurs types d’hôtes sont-ils disponibles ?

Il n’existe qu’un seul type d’hôte disponible.

#### <a name="what-are-the-cpu-specifications-in-each-type-of-host"></a>Quelles sont les spécifications d’UC dans chaque type d’hôte ?

Les serveurs sont équipés de deux processeurs Intel 18 cœurs cadencés à 2,3 GHz.

#### <a name="how-much-memory-is-in-each-host"></a>Quelle est la quantité de mémoire de chaque hôte ?

Les serveurs disposent de 576 Go de RAM.

#### <a name="what-is-the-storage-capacity-of-each-host"></a>Quelle est la capacité de stockage de chaque hôte ?

Chaque hôte ESXi présente deux groupes de disques vSAN d’une capacité de 15,2 To, et un cache NVMe de 3,2 To (1,6 To dans chaque groupe de disques).

#### <a name="how-much-network-bandwidth-is-available-in-each-esxi-host"></a>Quelle est la quantité de bande passante réseau disponible dans chaque hôte ESXi ?

Chaque hôte ESXi dans Azure VMware Solution est configuré avec quatre cartes réseau de 25 Gbits/s, deux cartes réseau étant configurées pour le trafic système ESXi et deux cartes réseau pour le trafic de charge de travail. 

#### <a name="is-data-stored-on-the-vsan-datastores-encrypted-at-rest"></a>Les données stockées dans les magasins de données vSAN sont-elles chiffrées au repos ?

Oui, toutes les données vSAN sont chiffrées par défaut à l’aide de clés stockées dans Azure Key Vault.

#### <a name="you-document-that-commvault-veritas-and-veeam-have-extended-their-backup-solutions-to-work-with-azure-vmware-solution-what-about-other-independent-software-vendors-isvs-backup-solutions"></a>Vous indiquez que Commvault, Veritas et Veeam ont étendu leurs solutions de sauvegarde pour fonctionner avec Azure VMware Solution. Qu’en est-il des solutions de sauvegarde des autres éditeurs de logiciels indépendants (ISV) ?

À notre connaissance, toute solution de sauvegarde qui utilise VMware VADP avec le mode de transport HotAdd doit fonctionner d’emblée sur Azure VMware Solution.

#### <a name="what-about-support-for-isv-backup-solutions"></a>Qu’en est-il de la prise en charge des solutions de sauvegarde des ISV ?

Quand un client installe et gère une telle solution de sauvegarde, il peut contacter l’éditeur de logiciels indépendant concerné pour obtenir de l’aide. 

#### <a name="what-is-the-correct-storage-policy-for-the-dedupe-setup"></a>Quelle est la bonne stratégie de stockage pour la configuration de la déduplication ?

Utilisez la stratégie de stockage *thin_provision* pour votre modèle de machine virtuelle.  La valeur par défaut est *thick_provision*.

#### <a name="are-the-snmp-infrastructure-logs-shared"></a>Les journaux de l’infrastructure SNMP sont-ils partagés ?

Non.

## <a name="hosts-clusters-and-private-clouds"></a>Hôtes, clusters et clouds privés

#### <a name="is-the-underlying-infrastructure-shared"></a>L’infrastructure sous-jacente est-elle partagée ?

Non, les clusters et hôtes de cloud privé sont dédiés, et effacés en toute sécurité avant et après usage.

#### <a name="what-are-the-minimum-and-the-maximum-number-of-hosts-per-cluster"></a>Quels est le nombre minimal et maximal d’hôtes par cluster ?

L’échelle possible des clusters est comprise entre 3 et 16 hôtes ESXi. Les clusters d’évaluation sont limités à trois hôtes.

#### <a name="can-i-scale-my-private-cloud-clusters"></a>Puis-je mettre à l’échelle mes clusters de clouds privés ?

Oui, les clusters se mettent à l’échelle entre le nombre minimal et le nombre maximal d’hôtes ESXi. Les clusters d’évaluation sont limités à trois hôtes.

#### <a name="what-are-trial-clusters"></a>Que sont les clusters d’évaluation ?

Les clusters d’évaluation sont trois clusters hôtes utilisés pour effectuer des évaluations d’un mois de clouds privés Azure VMware Solution.

#### <a name="can-i-use-high-end-hosts-for-trial-clusters"></a>Puis-je utiliser des hôtes haut de gamme pour des clusters d’évaluation ?

Non. Les hôtes ESXi haut de gamme sont réservés à une utilisation dans des clusters de production.

## <a name="azure-vmware-solution-and-vmware-software"></a>Azure VMware Solution et logiciel VMware

#### <a name="what-versions-of-vmware-software-is-used-in-private-clouds"></a>Quelles sont les versions des logiciels VMware utilisées dans les clouds privés ?

Les clouds privés utilisent vSphere 6.7 U3, vSAN 6.7 U3, VMware HCX et NSX-T 2.5.  Pour plus d’informations, consultez la [configuration requise pour la version du logiciel VMware](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-54E5293B-8707-4D29-BFE8-EE63539CC49B.html).

#### <a name="do-private-clouds-use-vmware-nsx"></a>Les clouds privés utilisent-ils VMware NSX ?

Oui, NSX-T 2.5 est utilisé pour la mise en réseau définie par logiciel dans les clouds privés Azure VMware Solution.

#### <a name="can-i-use-vmware-nsx-v-in-a-private-cloud"></a>Puis-je utiliser VMware NSX-V dans un cloud privé ?

Non. NSX-T est la seule version prise en charge de NSX.

#### <a name="is-nsx-required-in-on-premises-environments-or-networks-that-connect-to-a-private-cloud"></a>NSX est-il requis dans des environnements locaux ou des réseaux qui se connectent à un cloud privé ?

Non, vous n’êtes pas obligé d’utiliser NSX localement.

#### <a name="what-is-the-upgrade-and-update-schedule-for-vmware-software-in-a-private-cloud"></a>Quelle est la planification de mise à niveau et de mise à jour des logiciels VMware dans un cloud privé ?

Les mises à niveau de l’offre logicielle groupée de cloud privé conservent dans la version de l’offre logicielle groupée la plus récente de VMware. Les versions de logiciels de cloud privé peuvent différer des versions les plus récentes des composants logiciels individuels (ESXi, NSX-T, vCenter, vSAN).

#### <a name="how-often-will-the-private-cloud-software-stack-be-updated"></a>À quelle fréquence la pile logicielle du cloud privé sera-t-elle mise à jour ?

Les logiciels du cloud privé sont mis à niveau selon une planification qui suit la version de l’offre logicielle groupée de VMware. Votre cloud privé ne nécessite pas de temps d’arrêt pour les mises à niveau.

## <a name="connectivity"></a>Connectivité

#### <a name="what-network-ip-address-planning-is-required-to-incorporate-private-clouds-with-on-premises-environments"></a>Quelle est la planification d’adresse IP réseau requise pour incorporer des clouds privés avec des environnements locaux ?

Un espace d’adressage /22 de réseau privé est requis pour déployer un cloud privé Azure VMware Solution. Cet espace d’adressage privé ne doit pas chevaucher d’autres réseaux virtuels dans un abonnement ou des réseaux locaux.
 
#### <a name="how-do-i-connect-from-on-premises-environments-to-an-azure-vmware-solution-private-cloud"></a>Comment me connecter à partir d’environnements locaux à un cloud privé Azure VMware Solution ?

Vous pouvez vous connecter au service de deux façons : 

- Avec une machine virtuelle ou une passerelle d’application déployée sur un réseau virtuel Azure appairé au cloud privé via ExpressRoute.
- Via ExpressRoute Global Reach, à partir de votre centre de données local vers un circuit Azure ExpressRoute.

#### <a name="how-do-i-connect-a-workload-vm-to-the-internet-or-an-azure-service-endpoint"></a>Comment connecter une machine virtuelle de charge de travail à Internet ou à un point de terminaison de service Azure ?

Dans le portail Azure, activez la connectivité Internet pour un cloud privé. Avec le gestionnaire NSX-T, créez un routeur NSX-T T1 et un commutateur logique. Utilisez ensuite vCenter pour déployer une machine virtuelle sur le segment réseau défini par le commutateur logique. Cette machine virtuelle aura ainsi un accès réseau à Internet et aux services Azure.

#### <a name="do-i-need-to-restrict-access-from-the-internet-to-vms-on-logical-networks-in-a-private-cloud"></a>Dois-je restreindre l’accès à partir d’Internet à des machines virtuelles sur des réseaux logiques dans un cloud privé ?

Non. Le trafic entrant directement à partir d’Internet dans des clouds privés n’est pas autorisé par défaut.  Toutefois, vous pouvez exposer les machines virtuelles Azure VMware Solution à Internet par le biais de l’option [IP publique](public-ip-usage.md) de votre portail Azure pour votre cloud privé Azure VMware Solution.

#### <a name="do-i-need-to-restrict-internet-access-from-vms-on-logical-networks-to-the-internet"></a>Dois-je restreindre l’accès à Internet de machines virtuelles sur des réseaux logiques ?

Oui. Vous devez utiliser le gestionnaire NSX-T pour créer un pare-feu restreignant l’accès des machines virtuelles à Internet.


#### <a name="can-azure-vmware-solution-use-azure-virtual-wan-hosted-expressroute-gateways"></a>La solution Azure VMware peut-elle utiliser des passerelles ExpressRoute hébergées sur des réseaux étendus virtuels Azure ?
Oui.

#### <a name="can-transit-connectivity-be-established-between-on-premises-and-azure-vmware-solution-through-azure-virtual-wan-over-expressroute-global-reach"></a>La connectivité de transit peut-elle être établie entre la solution locale et VMware Azure via le réseau WAN virtuel Azure avec ExpressRoute Global Reach ?
Le réseau WAN virtuel Azure ne fournit pas de routage transitif entre deux circuits ExpressRoute connectés et une passerelle ExpressRoute WAN non virtuelle. L’utilisation d’ExpressRoute Global Reach permet la connectivité entre la solution locale et Azure VMware, mais transite par le réseau global Microsoft au lieu du hub WAN virtuel.


## <a name="accounts-and-privileges"></a>Comptes et privilèges

#### <a name="what-accounts-and-privileges-will-i-get-with-my-new-azure-vmware-solution-private-cloud"></a>Quels comptes et privilèges vais-je obtenir avec mon nouveau cloud privé Azure VMware Solution ?

Vous recevez des informations d’identification d’utilisateur de CloudAdmin dans vCenter et un accès administrateur sur le gestionnaire NSX-T. Il existe également un groupe CloudAdmin qui peut être utilisé pour incorporer Azure Active Directory. Pour plus d’informations, consultez [Concepts d’accès et d’identité](concepts-identity.md).

#### <a name="can-have-administrator-access-to-esxi-hosts"></a>Un administrateur peut-il accéder à des hôtes ESXi ?

Non, l’accès administrateur à ESXi est restreint conformément aux exigences de sécurité de la solution.

#### <a name="what-privileges-and-permissions-will-i-have-in-vcenter"></a>Quels sont les privilèges et les autorisations dont j’ai besoin dans vCenter ?

Vous disposez de privilèges de groupe CloudAdmin. Pour plus d’informations, consultez [Concepts d’accès et d’identité](concepts-identity.md).

#### <a name="what-privileges-and-permissions-will-i-have-on-the-nsx-t-manager"></a>Quels sont les privilèges et autorisations dont j’ai besoin sur le gestionnaire NSX-T ?

Vous disposerez de privilèges d’administrateur complets sur NSX-T et pourrez gérer le contrôle d’accès en fonction du rôle vSphere comme vous le feriez avec un centre de données NSX-T local. Pour plus d’informations, consultez [Concepts d’accès et d’identité](concepts-identity.md).

> [!NOTE]
> Un routeur T0 est créé et configuré dans le cadre d’un déploiement de cloud privé. Toute modification apportée à ce routeur logique ou aux machines virtuelles du nœud de périmètre NSX-T peut affecter la connectivité à votre cloud privé.

## <a name="billing-and-support"></a>Facturation et support

#### <a name="how-will-pricing-be-structured-for-azure-vmware-solution"></a>Comment les tarifs sont-ils structurés pour Azure VMware Solution ?

Pour toute question générale sur les tarifs, consultez la page [Tarification](https://azure.microsoft.com/pricing/details/azure-vmware) relative à Azure VMware Solution. 

#### <a name="can-azure-vmware-solution-be-purchased-through-a-microsoft-csp"></a>Azure VMware Solution peut-il être acheté via un fournisseur de services de chiffrement (CSP) Microsoft ?

Oui, les clients peuvent déployer Azure VMware Solution au sein d’un abonnement Azure géré par un CSP.

#### <a name="who-supports-azure-vmware-solution"></a>Qui prend en charge Azure VMware Solution ?

Microsoft assure la prise en charge d’Azure VMware Solution. Vous pouvez envoyer une [demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). 

Pour les abonnements gérés par le CSP, le premier niveau de support fournit par le fournisseur de solutions de la même manière que le CSP le fait pour les autres services Azure.

#### <a name="what-accounts-do-i-need-to-create-an-azure-vmware-solution-private-cloud"></a>De quels comptes ai-je besoin pour créer un cloud privé Azure VMware Solution ?

Vous devez disposer d’un compte Azure dans un abonnement Azure.

#### <a name="are-red-hat-solutions-supported-on-azure-vmware-solution"></a>Les solutions Red Hat sont-elles prises en charge sur Azure VMware Solution ?

Microsoft et Red Hat partagent une équipe de support intégrée colocalisée qui fournit un point de contact unifié pour les écosystèmes Red Hat s’exécutant sur la plateforme Azure.  Comme les autres services de la plateforme Azure qui fonctionnent avec Red Hat Enterprise Linux, Azure VMware Solution s’inscrit sous la protection de Cloud Access et du support intégré. Red Hat Enterprise Linux est pris en charge pour s’exécuter par-dessus Azure VMware Solution dans Azure.

#### <a name="is-vmware-hcx-enterprise-available-and-if-so-how-much-does-it-cost"></a>Est-ce que VMware HCX Enterprise est disponible ? Si oui, quel est son prix ?

VMware HCX Enterprise est disponible avec Azure VMware Solution en tant que fonction/service *en préversion*. Bien que la solution VMware HCX Enterprise pour Azure VMware Solution soit en préversion, il s’agit d’une fonction ou d’un service gratuit qui est soumis aux conditions générales du service en préversion. Une fois le service VMware HCX Enterprise mis à la disposition générale, vous recevrez un préavis de 30 jours indiquant le basculement de la facturation. Vous pouvez désactiver ou refuser le service.

#### <a name="how-do-i-request-a-host-quota-increase-for-azure-vmware-solution"></a>Comment faire une requête d’augmentation du quota d’hôtes pour une solution Azure VMware existante ?

Pour les abonnements gérés par le CSP, le client doit envoyer la demande au partenaire. L’équipe partenaire s’engage alors auprès de Microsoft pour obtenir une augmentation du quota de l’abonnement. Pour plus d’informations, consultez l’article [Comment activer la ressource Azure VMware Solution](enable-azure-vmware-solution.md). 

Pour les abonnements EA, utilisez la procédure suivante. Tout d’abord, vous aurez besoin des éléments suivants :

* Un [Contrat Entreprise (EA) Azure](../cost-management-billing/manage/ea-portal-agreements.md) avec Microsoft.
* Un compte Azure avec un abonnement Azure.

Avant de pouvoir créer votre ressource Azure VMware Solution, vous devez soumettre un ticket de support pour que des hôtes vous soient alloués. Cela prend jusqu’à cinq jours ouvrés pour confirmer et traiter votre demande. Si vous disposez d’un cloud privé Azure VMware Solution et que vous souhaitez que davantage d’hôtes vous soient alloués, vous passerez par le même processus.

1. Sur votre portail Azure, sous **Aide + Support**, créez une **[nouvelle demande de support](https://rc.portal.azure.com/#create/Microsoft.Support)** et indiquez les informations suivantes pour le ticket :
   - **Type de problème :** Techniques
   - **Abonnement :** Sélectionnez votre abonnement
   - **Service :** Tous les services > Azure VMware Solution
   - **Ressource :** Question générale 
   - **Résumé :** Capacité requise
   - **Type de problème :** Problèmes de gestion de la capacité
   - **Sous-type de problème :** Demande client de capacité/quota d’hôtes supplémentaire

1. Dans la **description** du ticket de support, indiquez les informations suivantes sous l’onglet **Détails** :

   - POC ou Production 
   - Nom de la région
   - Nombre d’hôtes
   - Autres détails

   >[!NOTE]
   >Azure VMware Solution recommande un minimum de trois hôtes pour mettre en place votre cloud privé et pour la redondance des hôtes N+1. 

1. Sélectionnez **Vérifier + Créer** pour envoyer la demande.

   Il faut compter jusqu’à cinq jours ouvrés pour qu’un représentant du support technique confirme votre demande.

   >[!IMPORTANT] 
   >Si vous disposez déjà d’une instance Azure VMware Solution et que vous demandez des hôtes supplémentaires, veuillez noter que cinq jours ouvrés sont nécessaires pour allouer les hôtes. 

1. Avant d’approvisionner vos hôtes, veillez à inscrire le fournisseur de ressources **Microsoft.AVS** sur le portail Azure.  

   ```azurecli-interactive
   az provider register -n Microsoft.AVS --subscription <your subscription ID>
   ```

   Pour d’autres façons d’inscrire un fournisseur de ressources, consultez [Fournisseurs et types de ressources Azure](../azure-resource-manager/management/resource-providers-and-types.md). 

#### <a name="are-reserved-instances-available-for-purchasing-through-the-cloud-solution-provider-csp-program"></a>Les instances réservées sont-elles disponibles pour l’achat via le programme des fournisseurs de solutions Cloud (CSP) ?

Oui. Le CSP peut acheter des instances réservées pour ses clients. Pour plus d’informations, consultez l’article [Faire des économies avec une instance réservée](reserved-instance.md). 

#### <a name="does-azure-vmware-solution-offer-multi-tenancy-for-hosting-csp-partners"></a>Azure VMware Solution offre-t-elle une architecture mutualisée pour héberger des partenaires CSP ?

Non. Actuellement, Azure VMware Solution ne propose pas d’architecture mutualisée.

#### <a name="will-traffic-between-on-premises-and-azure-vmware-solution-over-expressroute-incur-any-outbound-data-transfer-charge-in-the-metered-data-plan"></a>Le trafic entre la solution locale et Azure VMware Solution sur ExpressRoute entraînera-t-il des frais de transfert de données sortantes dans le plan de facturation à l’usage ?

Le trafic dans le circuit ExpressRoute d’Azure VMware Solution n’est pas mesuré de quelque manière que ce soit. Le trafic entre votre circuit ExpressRoute qui se connecte à votre site local et Azure est facturé conformément aux plans de tarification d’ExpressRoute.


## <a name="customer-communication"></a>Communication du client

#### <a name="how-can-i-receive-an-alert-when-azure-sends-service-health-notifications-to-my-azure-subscription"></a>Comment puis-je recevoir une alerte lorsqu’Azure envoie des notifications d’intégrité de service à mon abonnement Azure ?

Les notifications relatives aux problèmes de service, à la maintenance planifiée, aux avis d’intégrité et aux avis de sécurité sont publiées par l’intermédiaire de **Service Health** dans le portail Azure.  Vous pouvez prendre des mesures opportunes lorsque vous configurez des alertes de journal d’activité pour ces notifications. Pour plus d’informations, consultez [Créer des alertes d’intégrité de service à l’aide du portail Azure](../service-health/alerts-activity-log-service-notifications-portal.md#create-service-health-alert-using-azure-portal).

:::image type="content" source="media/service-health.png" alt-text="Capture d’écran des notifications de Service Health":::



<!-- LINKS - external -->
[kb2106952]: https://kb.vmware.com/s/article/2106952?lang=en_US&queryTerm=21069522

<!-- LINKS - internal -->
[Access and Identity Concepts]: concepts-identity.md
