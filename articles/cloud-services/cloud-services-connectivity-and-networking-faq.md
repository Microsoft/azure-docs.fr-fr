---
title: Problèmes de connectivité et de mise en réseau
titleSuffix: Azure Cloud Services
description: Cet article répertorie les questions fréquentes sur la connectivité et la mise en réseau pour Microsoft Azure Cloud Services.
services: cloud-services
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 84985660-2cfd-483a-8378-50eef6a0151d
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 08/23/2018
ms.author: genli
ms.openlocfilehash: d8f57cc16cad4c0b081478932f820c983e4bbdc7
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92070025"
---
# <a name="connectivity-and-networking-issues-for-azure-cloud-services-frequently-asked-questions-faqs"></a>Problèmes de connectivité et de mise en réseau pour Azure Cloud Services : questions fréquentes (FAQ)

Cet article comprend des questions fréquemment posées sur les problèmes de connectivité et de mise en réseau pour [Azure Cloud Services](https://azure.microsoft.com/services/cloud-services). Pour plus d’informations sur la taille, voir la [page sur la taille de machine virtuelle des Services Cloud](cloud-services-sizes-specs.md).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="i-cant-reserve-an-ip-in-a-multi-vip-cloud-service"></a>Je ne parviens pas à réserver une adresse IP dans un service cloud à plusieurs adresses IP virtuelles.
Commencez par vérifier que l’instance de machine virtuelle pour laquelle vous tentez de réserver l’adresse IP est activée. Veillez ensuite à utiliser des adresses IP réservées tant pour les déploiements intermédiaires que pour les déploiements de production. *Ne* modifiez pas les paramètres pendant la mise à niveau du déploiement.

## <a name="how-do-i-use-remote-desktop-when-i-have-an-nsg"></a>Comment utiliser le Bureau à distance quand je possède un groupe de sécurité réseau ?
Ajoutez des règles à un groupe de sécurité réseau qui autorisent le trafic sur les ports **3389** et **20000**. Bureau à distance utilise le port **3389**. Les instances de service cloud étant à charge équilibrée, vous ne pouvez pas contrôler directement l’instance à laquelle se connecter. Les agents *RemoteForwarder* et *RemoteAccess* gèrent le trafic opéré à l’aide du protocole RDP (Remote Desktop Protocol), et permettent au client d’envoyer un cookie RDP ainsi que de spécifier une instance individuelle à laquelle se connecter. Les agents *RemoteForwarder* et *RemoteAccess* exigent que ce port **20000** soit ouvert. Or, celui-ci pourrait être bloqué si vous avez un groupe de sécurité réseau.

## <a name="can-i-ping-a-cloud-service"></a>Est-il possible d’effectuer un test ping sur un service cloud ?

Non, pas en utilisant le test « ping » normal/protocole ICMP. Le protocole ICMP n’est pas autorisé via l’équilibreur de charge Azure.

Pour tester la connectivité, nous vous recommandons d’effectuer un test ping au niveau du port. Si Ping.exe utilise le protocole ICMP (Internet Control Message Protocol), d’autres outils tels que PSPing, Nmap et telnet permettent de tester la connectivité à un port TCP spécifique.

Pour plus d’informations, consultez [Use port pings instead of ICMP to test Azure VM connectivity](/archive/blogs/mast/use-port-pings-instead-of-icmp-to-test-azure-vm-connectivity) (Tester la connectivité des machines virtuelles Azure à l’aide de tests ping au niveau du port à la place d’ICMP).

## <a name="how-do-i-prevent-receiving-thousands-of-hits-from-unknown-ip-addresses-that-might-indicate-a-malicious-attack-to-the-cloud-service"></a>Comment éviter de recevoir des milliers d’accès en provenance d’adresses IP inconnues qui portent à croire que le service cloud est la cible d’une attaque malveillante ?
Azure met en œuvre une sécurité réseau multicouche pour protéger ses services de plateforme contre les attaques par déni de service distribué (DDoS). Le système de défense Azure DDoS fait partie intégrante du processus de surveillance continu d’Azure, qui est constamment amélioré à l’aide de tests de pénétration. Ce système de défense DDoS est conçu non seulement pour résister aux attaques extérieures, mais aussi à celles perpétrées par d’autres locataires Azure. Pour plus d’informations, voir [Sécurité du réseau Azure](https://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf).

Vous pouvez également créer une tâche de démarrage pour bloquer de manière sélective certaines adresses IP spécifiques. Pour plus d’informations, consultez [Bloquer une adresse IP spécifique](cloud-services-startup-tasks-common.md#block-a-specific-ip-address).

## <a name="when-i-try-to-rdp-to-my-cloud-service-instance-i-get-the-message-the-user-account-has-expired"></a>Quand j’essaie de me connecter avec le protocole RDP (Remote Desktop Protocol) à mon instance de service cloud via RDP, j’obtiens le message « Ce compte d’utilisateur est expiré ».
Vous pouvez obtenir le message d’erreur « Ce compte d’utilisateur est expiré » quand vous ignorez la date d’expiration configurée dans vos paramètres du protocole RDP. Vous pouvez modifier la date d’expiration à partir du portail en procédant comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com), accédez à votre service cloud, puis sélectionnez l’onglet **Bureau à distance**.

2. Sélectionnez l’emplacement de déploiement **Production** ou **Intermédiaire**.

3. Modifiez la date dans le champ **Expire le**, puis enregistrez la configuration.

Vous devriez maintenant pouvoir vous connecter à votre machine via RDP.

## <a name="why-is-azure-load-balancer-not-balancing-traffic-equally"></a>Pourquoi Azure Load Balancer n’équilibre-t-il pas le trafic uniformément ?
Pour plus d’informations sur le fonctionnement d’un l’équilibreur de charge interne, voir [Azure Load Balancer new distribution mode](https://azure.microsoft.com/blog/azure-load-balancer-new-distribution-mode/) (Nouveau mode de distribution d’Azure Load Balancer).

L’algorithme de distribution utilisé est un hachage 5-tuple (adresse IP source, port source, IP de destination, port de destination, type de protocole) servant à mapper le trafic aux serveurs disponibles. Il fournit l’adhérence uniquement dans une session de transport. Les paquets de la même session TCP ou UDP sont dirigés vers la même instance IP (DIP) de centre de données derrière le point de terminaison à charge équilibrée. Lorsque le client ferme puis rouvre la connexion, ou démarre une nouvelle session à partir de la même adresse IP source, le port source change et dirige le trafic vers un autre point de terminaison DIP.

## <a name="how-can-i-redirect-incoming-traffic-to-the-default-url-of-my-cloud-service-to-a-custom-url"></a>Comment rediriger le trafic arrivant sur l’URL par défaut de mon service cloud vers une URL personnalisée ?

Le module de réécriture d’URL d’IIS permet de rediriger le trafic arrivant sur l’URL par défaut du service cloud (par exemple, \*. cloudapp.net) vers un nom ou une URL personnalisés. Étant donné que le module de réécriture d’URL est activé par défaut sur les rôles web et que ses règles sont configurées dans le fichier web.config de l’application, il est toujours disponible sur la machine virtuelle, quel que soit le nombre de redémarrages ou de réinitialisations. Pour plus d’informations, voir :

- [Créer des règles de réécriture pour le module de réécriture d’URL](/iis/extensions/url-rewrite-module/creating-rewrite-rules-for-the-url-rewrite-module)
- [Supprimer un lien par défaut](https://stackoverflow.com/questions/32286487/azure-website-how-to-remove-default-link?answertab=votes#tab-top)

## <a name="how-can-i-blockdisable-incoming-traffic-to-the-default-url-of-my-cloud-service"></a>Comment bloquer/désactiver le trafic entrant sur l’URL par défaut de mon service cloud ?

Vous pouvez bloquer le trafic entrant via l’URL/le nom par défaut de votre service cloud (par exemple, \*. cloudapp.net). Définissez l’en-tête de l’hôte pour un nom DNS personnalisé (par exemple, www\.MyCloudService.com) sous la configuration de liaison de site dans le fichier de définition (*.csdef) du service cloud, comme indiqué :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="AzureCloudServicesDemo" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
    <WebRole name="MyWebRole" vmsize="Small">
        <Sites>
            <Site name="Web">
            <Bindings>
                <Binding name="Endpoint1" endpointName="Endpoint1" hostHeader="www.MyCloudService.com" />
            </Bindings>
            </Site>
        </Sites>
        <Endpoints>
            <InputEndpoint name="Endpoint1" protocol="http" port="80" />
        </Endpoints>
        <ConfigurationSettings>
            <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
        </ConfigurationSettings>
    </WebRole>
</ServiceDefinition>
```

Étant donné que cette liaison d’en-tête de l’hôte est appliquée via le fichier csdef, le service est accessible uniquement par le nom personnalisé « www.MyCloudService.com ». Toutes les demandes entrantes parvenant au domaine « *. cloudapp.net » échouent. Si vous utilisez une sonde SLB personnalisée ou un équilibreur de charge interne dans le service, le blocage de l’URL/du nom par défaut du service risque d’interférer avec le comportement de sondage.

## <a name="how-can-i-make-sure-the-public-facing-ip-address-of-a-cloud-service-never-changes"></a>Comment m’assurer que l’adresse IP publique d’un service cloud ne change jamais ?

Pour vous assurer que l’adresse IP publique de votre service cloud (également appelée adresse IP virtuelle) ne change jamais afin qu’elle puisse être approuvée de façon ordinaire par quelques clients spécifiques, nous vous recommandons d’y associer une adresse IP réservée. Autrement, l’adresse IP virtuelle fournie par Azure est désallouée de votre abonnement si vous supprimez le déploiement. Pour que l’opération d’échange d’adresses IP virtuelles réussisse, vous devez disposer d’adresses IP réservées pour les emplacement de production et préproduction. À défaut, l’opération d’échange échoue. Pour réserver une adresse IP et l’associer à votre service cloud, suivez les instructions des articles suivants  :

- [Réserver l’adresse IP d’un service cloud existant](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#reserve-the-ip-address-of-an-existing-cloud-service)
- [Associer une adresse IP réservée à un service cloud à l’aide d’un fichier de configuration de service](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#associate-a-reserved-ip-to-a-cloud-service-by-using-a-service-configuration-file)

Tant que vous avez plus d’une instance pour vos rôles, l’association d’une adresse IP réservée à votre service cloud ne devrait pas occasionner de temps d’arrêt. Vous pouvez aussi ajouter la plage d’adresses IP de votre centre de données Azure à une liste verte. Vous pouvez trouver toutes les plages d’adresses IP Azure dans le [Centre de téléchargement Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=41653).

Ce fichier contient les plages d’adresses IP (dont les plages de calcul, SQL et de stockage) utilisées dans les centres de données Azure. Un fichier mis à jour est publié chaque semaine, qui reflète les plages actuellement déployées et toutes les modifications à venir des plages d’adresses IP. Les nouvelles plages figurant dans le fichier ne sont pas utilisées dans les centres de données avant une semaine minimum. Téléchargez le nouveau fichier xml chaque semaine, et apportez les modifications nécessaires sur votre site pour identifier correctement les services qui s’exécutent dans Azure. Les utilisateurs d’ExpressRoute remarqueront peut-être que ce fichier est utilisé pour mettre à jour la publication de protocole de passerelle frontière de l’espace Azure la première semaine de chaque mois.

## <a name="how-can-i-use-azure-resource-manager-virtual-networks-with-cloud-services"></a>Comment utiliser des réseaux virtuels Azure Resource Manager avec des services cloud ?

Des services cloud ne peuvent pas être placés dans des réseaux virtuels Azure Resource Manager. Des réseaux virtuels Resource Manager et réseaux virtuels de déploiement classiques peuvent être connectés par le biais d’un peering. Pour en savoir plus, consultez [Peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md).


## <a name="how-can-i-get-the-list-of-public-ips-used-by-my-cloud-services"></a>Comment puis-je obtenir la liste des adresses IP publiques utilisées par mes services cloud ?

Vous pouvez utiliser le script PS suivant pour obtenir la liste des adresses IP publiques pour les services cloud dans votre abonnement

```powershell
$services = Get-AzureService  | Group-Object -Property ServiceName

foreach ($service in $services)
{
    "Cloud Service '$($service.Name)'"

    $deployment = Get-AzureDeployment -ServiceName $service.Name
    "VIP - " +  $deployment.VirtualIPs[0].Address
    "================================="
}
```