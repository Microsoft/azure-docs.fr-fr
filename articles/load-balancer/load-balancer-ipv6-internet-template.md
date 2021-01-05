---
title: Déployer un équilibreur de charge accessible sur Internet avec IPv6 - modèle Azure
titleSuffix: Azure Load Balancer
description: Découvrez comment déployer la prise en charge IPv6 pour un équilibreur de charge Azure et des machines virtuelles à charge équilibrée à l’aide d’un modèle Azure.
services: load-balancer
documentationcenter: na
author: asudbring
keywords: IPv6, équilibreur de charge azure, double pile, adresse ip publique, ipv6 natif, mobile, iot
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: allensu
ms.openlocfilehash: 415c95a441ac0cc6ed2dbf2d6a37f57d7a9e7341
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90562517"
---
# <a name="deploy-an-internet-facing-load-balancer-solution-with-ipv6-using-a-template"></a>Déployer une solution d’équilibrage de charge sur Internet avec IPv6, à l’aide d’un modèle

> [!div class="op_single_selector"]
> * [PowerShell](load-balancer-ipv6-internet-ps.md)
> * [Azure CLI](load-balancer-ipv6-internet-cli.md)
> * [Modèle](load-balancer-ipv6-internet-template.md)


>[!NOTE] 
>Cet article décrit une fonctionnalité IPv6 qui permet aux équilibreurs de charge de base de fournir une connectivité IPv4 et IPv6. Une connectivité IPv6 complète est maintenant disponible avec [IPv6 pour les réseaux virtuels Azure](../virtual-network/ipv6-overview.md), qui intègre la connectivité IPv6 aux réseaux virtuels et inclut des fonctionnalités clés comme les règles de groupe de sécurité réseau IPv6, le routage défini par l’utilisateur IPv6, l’équilibrage de charge IPv6 de base et standard, et plus encore.  Le protocole IPv6 pour les réseaux virtuels Azure est la norme recommandée pour les applications IPv6 dans Azure. Consultez [IPv6 pour le déploiement PowerShell de réseau virtuel Azure](../virtual-network/virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md).  

Un équilibrage de charge Azure est de type Couche 4 (TCP, UDP). L’équilibrage de charge offre une disponibilité élevée en distribuant le trafic entrant parmi les instances de service saines dans les services cloud ou les machines virtuelles dans un jeu d’équilibrage de la charge. Azure Load Balancer peut également présenter ces services sur plusieurs ports, plusieurs adresses IP ou les deux.

## <a name="example-deployment-scenario"></a>Exemple de scénario de déploiement

Le diagramme suivant illustre la solution d’équilibrage de charge déployée à l’aide de l’exemple de modèle décrit dans cet article.

![Le diagramme montre un exemple de scénario utilisé dans cet article, comprenant une station de travail cliente connectée à un équilibreur de charge Azure via Internet, connecté à son tour à deux machines virtuelles.](./media/load-balancer-ipv6-internet-template/lb-ipv6-scenario.png)

Dans ce scénario, vous allez créer les ressources Azure suivantes :

* une interface de réseau virtuel pour chaque machine virtuelle avec des adresses IPv4 et IPv6 affectées ;
* un équilibrage de charge accessible sur Internet avec une adresse IP publique IPv4 et IPv6 ;
* deux règles d’équilibrage de charge pour mapper les adresses IP virtuelles publiques sur les points de terminaison privés.
* un groupe à haute disponibilité contenant les deux VM ;
* deux machines virtuelles.

## <a name="deploying-the-template-using-the-azure-portal"></a>Déploiement du modèle à l’aide du portail Azure

Cet article fait référence à un modèle publié dans la galerie [Modèles de démarrage rapide Azure](https://azure.microsoft.com/documentation/templates/201-load-balancer-ipv6-create/). Vous pouvez télécharger le modèle à partir de la galerie ou lancer le déploiement dans Azure directement à partir de la galerie. Cet article suppose que vous avez téléchargé le modèle sur votre ordinateur local.

1. Ouvrez le portail Azure, et connectez-vous avec un compte disposant des autorisations pour créer les machines virtuelles et les ressources de réseau au sein d’un abonnement Azure. En outre, sauf si vous utilisez des ressources existantes, le compte requiert une autorisation pour créer un groupe de ressources et un compte de stockage.
2. Cliquez sur « +Nouveau » dans le menu, puis saisissez « modèle » dans la zone de recherche. Sélectionnez « Déploiement de modèle » dans les résultats de recherche.

    ![La capture d’écran montre le portail Azure avec les éléments Nouveau et Déploiement de modèle sélectionnés.](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step2.png)

3. Dans le panneau Tout, cliquez sur « Déploiement de modèle ».

    ![La capture d’écran montre Déploiement de modèle dans la place de marché.](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step3.png)

4. Cliquez sur « Créer ».

    ![La capture d’écran montre la description de Déploiement de modèle dans la place de marché.](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step4.png)

5. Cliquez sur « Modifier le modèle ». Supprimez le contenu existant et copiez/collez dans le contenu intégral du fichier de modèle (afin d’inclure les symboles { } de début et de fin), puis cliquez sur « Enregistrer ».

    > [!NOTE]
    > Si vous utilisez Microsoft Internet Explorer, lorsque vous procédez au collage, vous recevez une boîte de dialogue vous demandant d’autoriser l’accès au Presse-papiers Windows. Click « Autoriser l’accès ».

    ![La capture d’écran montre l’étape la plus active d’un déploiement personnalisé, qui consiste à modifier le modèle.](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step5.png)

6. Cliquez sur « Modifier les paramètres ». Dans le panneau Paramètres, spécifiez les valeurs suivant les instructions de la section Paramètres de modèle, puis cliquez sur « Enregistrer » pour fermer le volet Paramètres. Dans le volet Déploiement personnalisé, sélectionnez votre abonnement, un groupe de ressource existant, ou créez-en un. Si vous créez un groupe de ressources, sélectionnez un emplacement dédié. Ensuite, cliquez sur **Conditions juridiques**, puis sur **Achat** pour consulter les conditions juridiques. Azure commence le déploiement des ressources. Le déploiement de l’intégralité des ressources prend plusieurs minutes.

    ![La capture d’écran montre les étapes impliquées dans le déploiement personnalisé, en commençant par la saisie des valeurs des paramètres de modèle.](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step6.png)

    Pour plus d’informations sur ces paramètres, consultez la section [Paramètres et variables de modèle](#template-parameters-and-variables) , plus loin dans cet article.

7. Pour consulter les ressources créées par le modèle, cliquez sur Parcourir, faites défiler la liste vers le bas jusqu’à ce que « Groupes de ressources » apparaisse, puis cliquez dessus.

    ![La capture d’écran montre le portail Azure avec les éléments Parcourir et Groupes de ressources sélectionnés.](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step7.png)

8. Dans le volet Groupes de ressources, cliquez sur le nom spécifié à l’étape 6. L’ensemble des ressources déployées s’affichent. Si tout s’est bien passé, la mention « Réussi » apparaît en regard de « Dernier déploiement ». Dans le cas contraire, assurez-vous que le compte utilisé dispose des autorisations pour créer les ressources nécessaires.

    ![La capture d’écran affiche l’état du dernier déploiement d’un groupe de ressources, dans cet exemple, Réussi.](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step8.png)

    > [!NOTE]
    > Si vous accédez directement à vos groupes de ressources après avoir suivi l’étape 6, « Dernier déploiement » comporte la mention « Déploiement en cours » pendant le déploiement des ressources.

9. Cliquez sur « myIPv6PublicIP » dans la liste des ressources. Vous voyez que cet élément comporte une adresse IPv6 sous l’adresse IP, et que son nom DNS est la valeur spécifiée pour le paramètre dnsNameforIPv6LbIP à l’étape 6. Cette ressource est l’adresse IPv6 publique et le nom d’hôte qui sont accessibles aux clients Internet.

    ![La capture d’écran montre l’adresse publique IPv6.](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step9.png)

## <a name="validate-connectivity"></a>Valider la connectivité

Une fois le modèle déployé, vous pouvez valider la connectivité en exécutant les tâches suivantes :

1. Connectez-vous au portail Azure, puis à chacune des machines virtuelles créées par le déploiement de modèle. Si vous avez déployé une machine virtuelle Windows Server, exécutez ipconfig /all à partir d’une invite de commandes. Vous voyez que les machines virtuelles présentent les adresses UPv4 et IPv6. Si vous avez déployé des machines virtuelles Linux, vous devez configurer le système d’exploitation Linux pour recevoir des adresses IPv6 dynamiques en suivant les instructions fournies pour votre distribution Linux.
2. À partir d’un client Internet IPv6, lancez une connexion à l’adresse IPv6 publique de l’équilibrage de charge. Pour confirmer que l’équilibrage de charge est équilibré entre les deux machines virtuelles, vous pouvez installer un serveur web comme Microsoft Internet Information Services (IIS) sur chacune d’entre elles. La page web par défaut de chaque serveur doit comporter le texte « Server0 » ou « Server1» , qui les identifie de façon unique. Ensuite, ouvrez un navigateur Internet sur un client IPv6 connecté, puis accédez au nom d’hôte spécifié pour le paramètre dnsNameforIPv6LbIP de l’équilibrage de charge afin de confirmer la connectivité IPv6 de bout en bout à chaque machine virtuelle. Si vous voyez la page web d’un seul serveur, il vous faudra peut-être effacer le cache de votre navigateur. Ouvrez plusieurs sessions de navigation privées. Une réponse pour chaque serveur doit s’afficher.
3. À partir d’un client Internet IPv4, lancez une connexion à l’adresse IPv4 publique de l’équilibrage de charge. Pour confirmer que la charge est équilibrée entre les deux machines virtuelles, vous pouvez procéder à un test à l’aide d’IIS, tel que détaillé à l’étape 2.
4. À partir de chaque machine virtuelle, lancez une connexion sortante vers un dispositif Internet connecté à IPv4 ou IPv6. Dans les deux cas, l’IP source vue par l’appareil de destination est l’adresse IPv4 ou IPv6 publique de l’équilibrage de charge.

> [!NOTE]
> Le trafic ICMP des adresses IPv4 et IPv6 est bloqué par le réseau Azure. Par conséquent, les outils ICMP comme le test Ping sont toujours mis en échec. Pour tester la connectivité, utilisez une alternative TCP telle que TCPing ou l’applet de commande PowerShell Test-NetConnection. Notez que les adresses IP apparaissant dans le diagramme sont des exemples des valeurs pouvant apparaître. Dans la mesure où les adresses IPv6 sont affectées dynamiquement, les valeurs reçues diffèrent et varient en fonction des régions. En outre, il est courant que l’adresse IPv6 publique de l’équilibrage de charge commence par un préfixe différent des adresses IPv6 privées du pool principal.

## <a name="template-parameters-and-variables"></a>Paramètres et variables de modèle

Un modèle Azure Resource Manager contient plusieurs variables et paramètres que vous pouvez personnaliser selon vos besoins. Les variables sont utilisées pour les valeurs fixes qui ne doivent pas être modifiées par les utilisateurs. Les paramètres sont utilisés pour les valeurs devant être fournies par les utilisateurs lors du déploiement du modèle. L’exemple de modèle est configuré pour le scénario décrit dans cet article. Vous pouvez le personnaliser en fonction des besoins de votre environnement.

L’exemple de modèle utilisé dans cet article comprend les variables et les paramètres suivants :

| Paramètre / Variable | Notes |
| --- | --- |
| adminUsername |Spécifiez le nom du compte administrateur utilisé pour la connexion aux machines virtuelles. |
| adminPassword |Spécifiez le mot de passe du compte administrateur utilisé pour la connexion aux machines virtuelles. |
| dnsNameforIPv4LbIP |Spécifiez le nom d’hôte DNS que vous souhaitez affecter comme nom public de l’équilibrage de charge. Ce nom correspond à l’adresse IPv4 publique de l’équilibrage de charge. Le nom doit être en minuscules et correspondre à l’expression régulière suivante : ^[a-z][a-z0-9-]{1,61}[a-z0-9]$. |
| dnsNameforIPv6LbIP |Spécifiez le nom d’hôte DNS que vous souhaitez affecter comme nom public de l’équilibrage de charge. Ce nom correspond à l’adresse IPv6 publique de l’équilibrage de charge. Le nom doit être en minuscules et correspondre à l’expression régulière suivante : ^[a-z][a-z0-9-]{1,61}[a-z0-9]$. Il peut s’agir du nom de l’adresse IPv4. Lorsqu’un client envoie une requête DNS pour ce nom, Azure renvoie les enregistrements A et AAAA au moment du partage du nom. |
| vmNamePrefix |Spécifiez le préfixe du nom de la machine virtuelle. Le modèle ajoute un chiffre (0,1, etc.) au nom au moment de la création des machines virtuelles. |
| nicNamePrefix |Spécifiez le préfixe de nom d’interface réseau. Le modèle ajoute un chiffre (0,1, etc.) au nom au moment de la création des interfaces réseau. |
| storageAccountName |Entrez le nom d’un compte de stockage existant ou spécifiez le nom d’une nouvelle instance à créer par le modèle. |
| availabilitySetName |Entrez ensuite le nom du groupe à haute disponibilité à utiliser avec les machines virtuelles. |
| addressPrefix |Le préfixe d’adresse utilisé pour définir la plage d’adresses du réseau virtuel |
| subnetName |Le nom du sous-réseau créé pour le réseau virtuel. |
| subnetPrefix |Le préfixe d’adresse utilisé pour définir la plage d’adresses du sous-réseau. |
| vnetName |Spécifiez le nom du réseau virtuel utilisé par les machines virtuelles. |
| ipv4PrivateIPAddressType |La méthode d’allocation utilisé pour l’adresse IP privée (statique ou dynamique) |
| ipv6PrivateIPAddressType |La méthode d’allocation utilisé pour l’adresse IP privée (dynamique). IPv6 prend uniquement en charge l’allocation dynamique. |
| numberOfInstances |Le nombre d’instances à charge équilibrée déployées par le modèle. |
| ipv4PublicIPAddressName |Spécifiez le nom DNS que vous souhaitez utiliser pour communiquer avec l’adresse IPv4 publique de l’équilibrage de charge. |
| ipv4PublicIPAddressType |La méthode d’allocation utilisé pour l’adresse IP publique (statique ou dynamique) |
| Ipv6PublicIPAddressName |Spécifiez le nom DNS que vous souhaitez utiliser pour communiquer avec l’adresse IPv6 publique de l’équilibrage de charge. |
| ipv6PublicIPAddressType |La méthode d’allocation utilisé pour l’adresse IP publique (dynamique). IPv6 prend uniquement en charge l’allocation dynamique. |
| lbName |Spécifiez le nom de l’équilibrage de charge. Le nom est affiché dans le portail ou utilisé lors d’une référence à l’aide d’une commande CLI ou PowerShell. |

Les variables restantes du modèle contiennent des valeurs dérivées qui sont affectées lorsqu’Azure crée les ressources. Ne modifiez pas ces variables.

## <a name="next-steps"></a>Étapes suivantes

Pour connaître la syntaxe JSON et les propriétés d’un équilibreur de charge dans un modèle, consultez [Microsoft.Network/loadBalancers](/azure/templates/microsoft.network/loadbalancers).
