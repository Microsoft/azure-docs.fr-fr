---
title: Connecter Azure ExpressRoute à Oracle Cloud Infrastructure | Microsoft Docs
description: Connecter Azure ExpressRoute à Oracle Cloud Infrastructure (OCI) FastConnect pour activer des solutions d’applications Oracle dans le cloud
author: dbakevlar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 03/16/2020
ms.author: rogardle
ms.reviewer: cynthn
ms.openlocfilehash: 2717ba307cac82465e0c5df996ef3b668af5e7d4
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94963706"
---
# <a name="set-up-a-direct-interconnection-between-azure-and-oracle-cloud-infrastructure"></a>Configurer une interconnexion directe entre Azure et Oracle Cloud Infrastructure  

Pour créer une [expérience multicloud intégrée](oracle-oci-overview.md), Microsoft et Oracle offrent une interconnexion directe entre Azure et Oracle Cloud Infrastructure (OCI) via [ExpressRoute](../../../expressroute/expressroute-introduction.md) et [FastConnect](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/fastconnectoverview.htm). Via l’interconnexion ExpressRoute et FastConnect, les clients peuvent profiter d’une latence faible, d’un débit élevé et d’une connectivité directe entre deux clouds privés.

> [!IMPORTANT]
> Oracle certifie que ces applications s’exécutent dans Azure lors de l’utilisation de la solution d’interconnexion Azure / Oracle Cloud en mai 2020.
> * E-Business Suite
> * JD Edwards EnterpriseOne
> * PeopleSoft
> * Applications Oracle Retail
> * Oracle Hyperion Financial Management

L’image suivante illustre une vue d’ensemble de l’interconnexion :

![Connexion réseau dans le cloud](media/configure-azure-oci-networking/azure-oci-connect.png)

## <a name="prerequisites"></a>Prérequis

* Pour établir la connectivité entre Azure et OCI, vous devez disposer d’un abonnement Azure actif et une location OCI active.

* Cette connectivité est uniquement possible là où un emplacement de peering ExpressRoute est à proximité ou dans le même emplacement de peering que le FastConnect OCI. Consultez [Disponibilité dans les régions](oracle-oci-overview.md#region-availability).

## <a name="configure-direct-connectivity-between-expressroute-and-fastconnect"></a>Configurer une connexion directe entre ExpressRoute et FastConnect

1. Créez un circuit ExpressRoute standard sur votre abonnement Azure sous un groupe de ressources. 
    * Lorsque vous créez le circuit ExpressRoute, choisissez **Oracle Cloud FastConnect** comme fournisseur de services. Pour créer un circuit ExpressRoute, consultez notre [guide pas à pas](../../../expressroute/expressroute-howto-circuit-portal-resource-manager.md).
    * Un circuit Azure ExpressRoute fournit des options de bande passante granulaires, tandis que FastConnect prend en charge 1, 2, 5 ou 10 Gbits/s. Par conséquent, nous vous recommandons de choisir une de ces options de bande passante sous ExpressRoute.

    ![Créer un circuit ExpressRoute](media/configure-azure-oci-networking/exr-create-new.png)
1. Notez votre **clé de service** ExpressRoute. Vous devez fournir cette clé lors de la configuration de votre circuit FastConnect.

    ![Clé de service ExpressRoute](media/configure-azure-oci-networking/exr-service-key.png)

    > [!IMPORTANT]
    > Vous serez facturé pour les frais ExpressRoute dès que le circuit ExpressRoute sera approvisionné (même si le paramètre **État du fournisseur** est défini sur **Non approvisionné**).

1. Créez deux espaces d’adresse IP privés de /30 chacun qui ne chevauchent pas votre réseau virtuel Azure ou l’espace d’adresse IP de votre réseau cloud virtuel OCI. Nous appellerons le premier espace d’adresse IP « espace d’adresse primaire » et le second espace d’adresse IP « espace d’adresse secondaire ». Notez les adresses, vous en aurez besoin lors de la configuration de votre circuit FastConnect.
1. Créez une passerelle de routage dynamique. Vous en aurez besoin lors de la création de votre circuit FastConnect. Pour plus d’informations, consultez la documentation sur la [Passerelle de routage dynamique](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingDRGs.htm).
1. Créez un circuit FastConnect sous votre locataire Oracle. Pour plus d’informations, consultez la [documentation Oracle](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/azure.htm).
  
    * Sous Configuration FastConnect, sélectionnez **Microsoft Azure : ExpressRoute** comme fournisseur.
    * Sélectionnez la passerelle de routage dynamique que vous avez configurée à l’étape précédente.
    * Sélectionnez la bande passante à approvisionner. Pour des performances optimales, la bande passante doit correspondre à la bande passante sélectionnée lorsque vous créez le circuit ExpressRoute.
    * Dans **Provider Service Key** (Clé de service du fournisseur), collez la clé de service ExpressRoute.
    * Utilisez le premier espace d’adresses IP privé /30 issu de l’étape précédente en tant qu’**Primary BGP IP Address** (Adresse IP BGP primaire) le deuxième espace d’adresses IP privé /30 en tant qu’adresse **Secondaty BGP IP** (IP BGP secondaire).
        * Affectez la première adresse utilisable des deux plages pour l’adresse IP BGP Oracle (primaire et secondaire) et la seconde adresse à l’adresse IP BGP du client (dans une perspective FastConnect). La première adresse IP utilisable est la deuxième adresse IP dans l’espace adresse /30 (la première adresse IP est réservée par Microsoft).
    * Cliquez sur **Créer**.
1. Terminez la liaison du FastConnect au réseau cloud virtuel sous votre locataire Oracle via Dynamic Routing Gateway, à l’aide de la table de routage.
1. Accédez à Azure et vérifiez que le paramètre **État du fournisseur** pour votre circuit ExpressRoute est défini sur **Approvisionné** et qu’un peering de type **Privé Azure** a été approvisionné. Ceci est nécessaire pour les étapes suivantes.

    ![État du fournisseur ExpressRoute](media/configure-azure-oci-networking/exr-provider-status.png)
1. Cliquez sur le peering **Privé Azure**. Vous verrez que les détails de peering ont été automatiquement configurés en fonction des informations que vous avez saisies lors de la configuration de votre circuit FastConnect.

    ![Paramètres de peering privé](media/configure-azure-oci-networking/exr-private-peering.png)

## <a name="connect-virtual-network-to-expressroute"></a>Connecter un réseau virtuel à ExpressRoute

1. Créez un réseau virtuel et une passerelle de réseau virtuel, si vous ne l’avez pas déjà fait. Pour savoir comment procéder, veuillez consulter notre [guide pas à pas](../../../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md).
1. Configurez la connexion entre la passerelle de réseau virtuel et votre circuit ExpressRoute en exécutant le [script Terraform](https://github.com/microsoft/azure-oracle/tree/master/InterConnect-2) ou en exécutant la commande PowerShell pour [Configurer ExpressRoute FastPath](../../../expressroute/expressroute-howto-linkvnet-arm.md#configure-expressroute-fastpath).

Une fois que vous avez terminé la configuration du réseau, vous pouvez vérifier la validité de votre configuration en cliquant sur **Get ARP Records** (Obtenir les enregistrements ARP) et **Get route table** (Obtenir une table de routage) sous le panneau Peering privé ExpressRoute dans le Portail Microsoft Azure.

## <a name="automation"></a>Automatisation

Microsoft a créé des scripts Terraform pour activer le déploiement automatisé de l’interconnexion de réseau. Les scripts Terraform doivent s’authentifier auprès d’Azure avant leur exécution, car ils nécessitent des autorisations adéquates sur l’abonnement Azure. L’authentification peut être effectuée à l’aide un [Principal de service Azure Active Directory](../../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) ou à l’aide de l’interface de ligne de commande Azure. Pour plus d’informations, voir la documentation de [Terraform](https://www.terraform.io/docs/providers/azurerm/auth/azure_cli.html).

Vous trouverez les scripts de Terraform et la documentation pour déployer l’interconnexion dans ce [référentiel GitHub](https://aka.ms/azureociinterconnecttf).

## <a name="monitoring"></a>Surveillance

En installant des agents sur les deux clouds, vous pouvez tirer parti d’Azure [Network Performance Monitor (NPM)](../../../expressroute/how-to-npm.md) pour surveiller les performances du réseau de bout en bout. NPM vous aide à identifier facilement les problèmes de réseau et à les éliminer.

## <a name="delete-the-interconnect-link"></a>Supprimer le lien d’interconnexion

Pour supprimer l’interconnexion, les étapes suivantes doivent être suivies, dans l’ordre indiqué. Si vous ne respectez pas cet ordre, le circuit ExpressRoute passe en « état d’échec ».

1. Supprimez la connexion ExpressRoute. Supprimez la connexion en sélectionnant l’icône **Supprimer** dans la page de votre connexion. Pour plus d’informations, consultez la [Documentation d’ExpressRoute](../../../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md#clean-up-resources).
1. Supprimez le FastConnect Oracle à partir de la console de cloud Oracle.
1. Une fois le circuit FastConnect Oracle supprimé, vous pouvez supprimer le circuit Azure ExpressRoute.

À ce stade, le processus de suppression et de déprovisionnement est terminé.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la connexion dans le cloud entre OCI et Azure, consultez la [documentation d’Oracle](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/azure.htm).
* Utilisez des [scripts Terraform](https://aka.ms/azureociinterconnecttf) pour déployer l’infrastructure des applications Oracle ciblées sur Azure et configurer l’interconnexion réseau. 
