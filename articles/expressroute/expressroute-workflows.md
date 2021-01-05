---
title: 'Azure ExpressRoute : Workflow de configuration de circuit'
description: Cette page vous présente le workflow pour la configuration du circuit ExpressRoute et des peerings
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 08/24/2020
ms.author: duau
ms.custom: contperf-fy21q1
ms.openlocfilehash: 2742b03bcacd73e7e602666b898417f295905f19
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97034069"
---
# <a name="expressroute-workflows-for-circuit-provisioning-and-circuit-states"></a>Workflows ExpressRoute d’approvisionnement du circuit et états du circuit

Cet article vous guide de façon sommaire tout au long des workflows de provisionnement du service et de configuration du routage. Les sections suivantes décrivent les tâches de provisionnement d’un circuit ExpressRoute de bout en bout.

## <a name="workflow-steps"></a>Étapes du workflow

### <a name="1-prerequisites"></a>1. Prérequis

Vérifiez que les prérequis sont respectés. Pour obtenir la liste complète, consultez [Configuration requise et liste de contrôle](expressroute-prerequisites.md).

* Un abonnement Azure a été créé.
* La connectivité physique a été établie avec le partenaire ExpressRoute ou configurée via ExpressRoute Direct. Passez en revue l’emplacement, consultez [Emplacements et partenaires](expressroute-locations-providers.md#partners) pour visualiser le partenaire ExpressRoute et la connectivité ExpressRoute Direct sur les emplacements de peering.

### <a name="2-order-connectivity-or-configure-expressroute-direct"></a>2. Commander la connectivité ou configurer ExpressRoute Direct

Commandez la connectivité à partir du fournisseur de services ou configurez ExpressRoute Direct.

#### <a name="expressroute-partner-model"></a>Modèle de partenaire ExpressRoute

Commandez la connectivité auprès du fournisseur de services. Ce processus varie. Contactez votre fournisseur de connectivité pour plus d’informations sur la commande de connectivité.

* Sélectionnez le partenaire ExpressRoute
* Sélectionnez l’emplacement de peering
* Sélectionnez la bande passante
* Sélectionnez le modèle de facturation
* Sélectionnez le module complémentaire standard ou premium

#### <a name="expressroute-direct-model"></a>Modèle ExpressRoute Direct

* Affichez la capacité ExpressRoute Direct disponible sur l’ensemble des emplacements de peering.
* Réservez des ports en créant la ressource ExpressRoute Direct dans votre abonnement Azure.
* Demandez et recevez la lettre d’autorisation et commandez les connexions croisées physiques à partir du fournisseur d’emplacement de peering.
* Activez l’état administrateur et visualisez les niveaux d’éclairage et le lien physique à l’aide d’[Azure Monitor](expressroute-monitoring-metrics-alerts.md#expressroute-direct-metrics).

### <a name="3-create-an-expressroute-circuit"></a>3. Création d’un circuit ExpressRoute

#### <a name="expressroute-partner-model"></a>Modèle de partenaire ExpressRoute

Vérifiez que le partenaire ExpressRoute est prêt à provisionner la connectivité. Votre circuit ExpressRoute est facturé à partir de l’émission d'une clé de service. Utilisez les instructions fournies dans [Créer un circuit ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) pour créer votre circuit.

#### <a name="expressroute-direct-model"></a>Modèle ExpressRoute Direct

Assurez-vous que le lien physique et l’état administrateur sont activés sur les deux liens. Reportez-vous à [Comment configurer ExpressRoute Direct](how-to-expressroute-direct-portal.md) pour obtenir des instructions. Votre circuit ExpressRoute est facturé à partir de l’émission d'une clé de service. Utilisez les instructions fournies dans [Créer un circuit ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) pour créer votre circuit.

### <a name="4-service-provider-provisions-connectivity"></a>4. Provisionnement de la connectivité par le fournisseur de services

Cette section porte uniquement sur le modèle de connectivité de partenaire ExpressRoute :

* Fournissez la clé de service (s-key) au fournisseur de connectivité.
* Fournissez les informations supplémentaires requises par le fournisseur de connectivité (par exemple, l’ID du VPN).
* Si le fournisseur gère la configuration de routage, fournissez les détails nécessaires.

Vous pouvez vous assurer que le circuit a été correctement provisionné en vérifiant l’état de provisionnement du circuit ExpressRoute via PowerShell, le portail Azure ou l’interface CLI.

### <a name="5-configure-routing-domains"></a>5. Configurer les domaines de routage

Configurez les domaines de routage. Si votre fournisseur de connectivité gère une configuration de couche 3, il configurera le routage pour votre circuit. Si votre fournisseur de connectivité offre uniquement des services de couche 2 ou si vous utilisez ExpressRoute Direct, vous devez configurer le routage conformément aux instructions décrites dans les articles [Configuration requise pour le routage](expressroute-routing.md) et [Configuration du routage](expressroute-howto-routing-classic.md).

#### <a name="for-azure-private-peering"></a>Pour le peering privé Azure

Activez le peering privé pour vous connecter aux machines virtuelles et aux services cloud déployés au sein du réseau virtuel Azure.

* Sous-réseau de peering pour le chemin 1 (/30)
* Sous-réseau de peering pour le chemin 2 (/30)
* ID du réseau local virtuel pour le peering
* ASN pour le peering
* ASN ExpressRoute = 12076
* Hachage MD5 (facultatif)

#### <a name="for-microsoft-peering"></a>Pour le peering Microsoft

Activez cette option pour accéder aux services en ligne de Microsoft, tels que Microsoft 365. De plus, tous les services Azure PaaS sont accessibles via le peering Microsoft. Pour vous connecter à Microsoft, vous devez veiller à utiliser un proxy/périphérique différent de celui que vous utilisez pour Internet. L’utilisation du même appareil edge à la fois pour ExpressRoute et Internet entraîne un routage asymétrique et provoque des pertes de connectivité sur votre réseau.

* Sous-réseau de peering pour le chemin 1 (/30) – doit être une adresse IP publique
* Sous-réseau de peering pour le chemin 2 (/30) – doit être une adresse IP publique
* ID du réseau local virtuel pour le peering
* ASN pour le peering
* Préfixes publiés – doivent être des préfixes d’adresses IP publiques
* ASN du client (facultatif s’il diffère de l’ASN de peering)
* RIR/IRR pour la validation IP et ASN
* ASN ExpressRoute = 12076
* Hachage MD5 (facultatif)

### <a name="6-start-using-the-expressroute-circuit"></a>6. Commencer à utiliser le circuit ExpressRoute

* Vous pouvez lier des réseaux virtuels Azure à votre circuit ExpressRoute pour activer la connectivité à partir d’un emplacement local au réseau virtuel Azure. Reportez-vous à l’article [Connecter un réseau virtuel à un circuit](expressroute-howto-linkvnet-arm.md) pour obtenir des instructions. Ces réseaux virtuels peuvent figurer dans le même abonnement Azure que le circuit ExpressRoute, ou dans un autre abonnement.
* Connectez-vous aux services Azure et aux services cloud Microsoft par le biais du peering Microsoft.

##  <a name="expressroute-partner-circuit-provisioning-states"></a><a name="expressroute-circuit-provisioning-states"></a>États de provisionnement du circuit de partenaire ExpressRoute

La section suivante décrit les différents états de circuit ExpressRoute pour le modèle de connectivité de partenaire ExpressRoute.
Chaque circuit de partenaire ExpressRoute possède deux états :

* **ServiceProviderProvisioningState** représente l’état côté fournisseur de connectivité. Il peut afficher *NotProvisioned*, *Provisioning* ou *Provisioned*. Pour que vous puissiez configurer le peering, le circuit ExpressRoute doit être dans un état Provisionné. **Cet état concerne uniquement les circuits de partenaires ExpressRoute et n’est pas affiché dans les propriétés d’un circuit ExpressRoute Direct**.

* Le **statut** représente l’état de provisionnement de Microsoft. Cette propriété est définie sur Activée lorsque vous créez un circuit ExpressRoute

### <a name="possible-states-of-an-expressroute-circuit"></a>États possibles d'un circuit ExpressRoute

Cette section décrit les états possibles d’un circuit ExpressRoute créé sous le modèle de connectivité de partenaire ExpressRoute.

**Lors de la création**

Le circuit ExpressRoute signale les états suivants lors de la création des ressources.

```output
ServiceProviderProvisioningState : NotProvisioned
Status                           : Enabled
```

**Quand le fournisseur de connectivité est en cours de provisionnement du circuit**

Le circuit ExpressRoute indique les états suivants lorsque le fournisseur de connectivité procède au provisionnement du circuit.

```output
ServiceProviderProvisioningState : Provisioning
Status                           : Enabled
```

**Quand le fournisseur de connectivité a terminé le processus de provisionnement**

Le circuit ExpressRoute indique les états suivants une fois que le fournisseur de connectivité a terminé le provisionnement du circuit.

```output
ServiceProviderProvisioningState : Provisioned
Status                           : Enabled
```

**Quand le fournisseur de connectivité annule le provisionnement du circuit**

Si le circuit ExpressRoute doit être déprovisionné, le circuit signalera les états suivants une fois que le fournisseur de services aura terminé le déprovisionnement.

```output
ServiceProviderProvisioningState : NotProvisioned
Status                           : Enabled
```

Vous pouvez choisir de le réactiver si nécessaire, ou exécuter des applets de commande PowerShell pour supprimer le circuit.  

> [!IMPORTANT]
> Un circuit ne peut pas être supprimé si ServiceProviderProvisioningState a la valeur Provisioning (En cours de provisionnement) ou Provisioned (Provisionné). Le fournisseur de connectivité doit déprovisionner le circuit avant de pouvoir le supprimer. Microsoft continuera de facturer le circuit tant que la ressource de circuit ExpressRoute n’aura pas été supprimée dans Azure.
> 

## <a name="routing-session-configuration-state"></a>État de configuration d’une session de routage

L’état de provisionnement BGP indique si la session BGP a été activée sur le réseau de périmètre Microsoft. L’utilisation du peering privé ou Microsoft doit être activée (Enabled).

Il est important de vérifier l'état de la session BGP, en particulier pour le peering Microsoft. En plus de l'état d’approvisionnement BGP, il existe un autre état appelé *état des préfixes publics publiés*. Les préfixes publics publiés doivent afficher l’état *configured*, pour que la session BGP soit opérationnelle et pour que votre routage fonctionne de bout en bout. 

Si l'état du préfixe public publié indique qu’une *validation est nécessaire* , la session BGP n'est pas activée car les préfixes publiés ne correspondent pas au numéro AS dans un des registres de routage.

> [!IMPORTANT]
> Si l’état des préfixes publics publiés indique *manual validation* (validation manuelle), vous devez ouvrir un ticket de support auprès du [support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) et fournir la preuve que vous êtes le propriétaire des adresses IP publiées, ainsi que le numéro système autonome associé.
> 
> 

## <a name="next-steps"></a>Étapes suivantes

* Configurez votre connexion ExpressRoute.
  
  * [Création d’un circuit ExpressRoute](expressroute-howto-circuit-arm.md)
  * [Configuration du routage](expressroute-howto-routing-arm.md)
  * [Liaison d’un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-arm.md)