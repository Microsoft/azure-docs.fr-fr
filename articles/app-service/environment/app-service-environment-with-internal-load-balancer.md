---
title: Créer un environnement ASE ILB v1
description: Création et utilisation d’un environnement ASE avec un équilibreur de charge (ILB). Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: ccompy
ms.assetid: ad9a1e00-d5e5-413e-be47-e21e5b285dbf
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: c4e5dedf2075a2e13cc91c5eed2c0f03ba498b97
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021518"
---
# <a name="using-an-internal-load-balancer-with-an-app-service-environment"></a>Utilisation d’un équilibreur de charge interne avec un environnement App Service

> [!NOTE] 
> Cet article traite de l’environnement App Service Environment v1. Il existe une version plus récente de l’environnement App Service Environment, plus facile à utiliser et qui s’exécute sur des infrastructures plus puissantes. Pour en savoir plus sur la nouvelle version, commencez par la [Présentation de l’environnement App Service Environment](intro.md).
>

La fonctionnalité ASE (App Service Environment) est une option de service Premium d’Azure App Service offrant une fonction de configuration améliorée qui n’est pas disponible dans les clusters mutualisés. La fonctionnalité ASE déploie essentiellement Azure App Service sur votre réseau virtuel (VNet) Azure. Pour mieux comprendre les possibilités offertes par les environnements App Service, lisez la documentation [Qu'est-ce qu'un environnement App Service ?][WhatisASE]. Si vous ne connaissez pas les avantages de l’utilisation d’un réseau virtuel, consultez la [FAQ sur le réseau virtuel Azure][virtualnetwork]. 

## <a name="overview"></a>Vue d’ensemble
Un environnement App Service peut être déployé avec un point de terminaison accessible via Internet ou avec une adresse IP sur votre réseau virtuel. Pour définir l’adresse IP sur une adresse de réseau virtuel, vous devez déployer votre ASE avec un équilibreur de charge interne (ILB). Lorsque votre environnement App Service est configuré avec un équilibreur de charge interne, vous fournissez :

* votre propre domaine ou sous-domaine. Pour simplifier, ce document suppose que vous utilisez un sous-domaine, mais vous pouvez choisir le scénario qui vous convient. 
* le certificat utilisé pour le protocole HTTPS
* Gestion du service DNS pour votre sous-domaine. 

En retour, vous pouvez effectuer des tâches telles que :

* Héberger des applications intranet (comme des applications métier) en toute sécurité dans le cloud, auquel vous accédez via un VPN Site à Site ou ExpressRoute
* héberger des applications dans le cloud qui ne figurent pas dans les serveurs DNS publics
* Créer des applications backend isolées d’Internet, auxquelles vos applications frontend peuvent s’intégrer en toute sécurité

#### <a name="disabled-functionality"></a>Fonctionnalités désactivées
Lorsque vous utilisez un ASE ILB, certaines opérations ne sont pas autorisées. Ces opérations sont :

* utilisation du protocole IPSSL
* attribution d’adresses IP à des applications spécifiques
* achat et utilisation d’un certificat avec une application via le portail. Vous pouvez toujours obtenir des certificats directement auprès d’une autorité de certification et les utiliser avec vos applications, mais pas via le portail Azure.

## <a name="creating-an-ilb-ase"></a>Création d’un environnement App Service d’équilibrage de charge interne (ILB ASE)
La création d’un ILB ASE n’est pas très différente de la création d’un ASE normalement. Pour plus d’informations sur la création d’un environnement App Service, consultez [Comment créer un environnement App Service][HowtoCreateASE]. Le processus de création d’un environnement ASE LIB est le même que la création d’un réseau virtuel lors de la création de l’ASE ou de la sélection d’un réseau virtuel existant. Pour créer un ILB ASE : 

1. Dans le portail Azure, sélectionnez **Créer une ressource ->Web + Mobile ->App Service Environment**.
2. Sélectionnez votre abonnement.
3. Sélectionnez ou créez un groupe de ressources.
4. Sélectionnez ou créez un réseau virtuel.
5. Créez un sous-réseau si vous sélectionnez un réseau virtuel.
6. Sélectionnez **Réseau virtuel/Emplacement -> Configuration du réseau virtuel**, puis définissez le type d’adresse IP virtuelle sur Interne.
7. Indiquez le nom du sous-domaine (il s’agit du sous-domaine utilisé pour les applications créées dans cet environnement App Service).
8. Sélectionnez **OK**, puis **Créer**.

![Montre les écrans utilisés pour créer un environnement ASE ILB.][1]

Dans le volet Réseau virtuel, l’option Configuration de réseau virtuel vous permet de choisir entre une adresse IP virtuelle externe ou interne. La valeur par défaut est Externe. Si vous sélectionnez Externe, votre environnement App Service utilise une adresse IP virtuelle accessible via Internet. Si vous sélectionnez Interne, votre environnement App Service est configuré avec un équilibreur de charge interne sur une adresse IP appartenant à votre réseau virtuel. 

Après avoir sélectionné la valeur Interne, vous ne pouvez plus ajouter d’adresses IP à votre environnement App Service et devez alors spécifier le sous-domaine de ce dernier. Dans un environnement App Service avec une adresse IP virtuelle externe, le nom de l’environnement est utilisé dans le sous-domaine pour les applications créées dans cet environnement. Si votre environnement App Service s’appelle **_contosotest_* _ et votre application _*_mytest_*_, le sous-domaine est au format _*_contosotest.p.azurewebsites.net_*_ et l’URL de cette application est _*_mytest.contosotest.p.azurewebsites.net_*_ . Si vous définissez le type d’adresse VIP sur Interne, le nom de votre ASE n’est pas utilisé dans le sous-domaine pour cet ASE. Vous spécifiez explicitement le sous-domaine. Si votre sous-domaine est _*_contoso.corp.net_*_ et que vous créez une application dans cet environnement App Service nommé _*_timereporting_*_, l’URL de cette application est _*_timereporting.contoso.corp.net_*_.

## <a name="apps-in-an-ilb-ase"></a>Applications d’un ILB ASE
La création d’une application dans un ILB ASE est identique à la création d’une application dans un ASE standard. 

1. Dans le portail Azure, sélectionnez _ *Créer une ressource->Web + Mobile->Web** ou **Mobile** ou **API App**.
2. Entrez le nom de l’application.
3. Sélectionnez votre abonnement.
4. Sélectionnez ou créez un groupe de ressources.
5. Sélectionnez ou créez un plan App Service. Si vous créez un plan App Service, sélectionnez votre environnement App Service comme emplacement, puis choisissez le pool de workers dans lequel vous souhaitez créer votre plan App Service. Lorsque vous créez le plan App Service, vous sélectionnez votre environnement App Service comme emplacement, ainsi que le pool de workers. Lorsque vous spécifiez le nom de l’application, vous voyez que le sous-domaine sous le nom de votre application est remplacé par le sous-domaine de votre environnement App Service. 
6. Sélectionnez **Create** (Créer). Cochez la case **Épingler au tableau de bord** si vous souhaitez que l’application s’affiche dans votre tableau de bord. 

![Montre comment créer une application dans un environnement ASE ILB sur le Portail Azure.][2]

Sous le nom de l’application, le nom du sous-domaine est mis à jour pour refléter le sous-domaine de votre ASE. 

## <a name="post-ilb-ase-creation-validation"></a>Validation après la création de l’ILB ASE
Un ILB ASE est légèrement différent d’un ASE non-ILB. Comme indiqué précédemment, vous devez gérer votre propre DNS et fournir votre propre certificat pour les connexions HTTPS. 

Une fois votre environnement App Service créé, vous remarquerez que le sous-domaine affiche le sous-domaine que vous avez spécifié, et un nouvel élément apparaît dans le menu **Paramètre**, appelé **Certificat ILB**. L’ASE est créé avec un certificat auto-signé qui facilite le test de HTTPS. Le portail vous indique que vous devez fournir votre propre certificat pour le protocole HTTPS, dans le but de vous encourager à avoir un certificat pour votre sous-domaine. 

![Montre le sous-domaine spécifié lors de la création de l’environnement ASE.][3]

Si vous effectuez simplement des tests et ignorez comment créer un certificat, vous pouvez utiliser l’application console MMC IIS pour créer un certificat auto-signé. Une fois le certificat créé, vous pouvez l’exporter sous forme de fichier .pfx, puis le charger dans l’interface utilisateur du certificat ILB. Lorsque vous accédez à un site sécurisé avec un certificat auto-signé, votre navigateur vous avertit que le site auquel vous accédez n’est pas sécurisé en raison de l’impossibilité de valider le certificat. Pour éviter cet avertissement, vous devez utiliser un certificat dûment signé correspondant à votre sous-domaine et contenant une chaîne de confiance reconnue par votre navigateur.

![Montre comment utiliser l’application console MMC IIS pour créer un certificat auto-signé.][6]

Si vous souhaitez essayer le flux avec vos propres certificats et tester l’accès HTTP et HTTPS à votre ASE :

1. Accédez à l’interface utilisateur de l’environnement App Service après sa création : **Environnement App Service -> Paramètres -> Certificats ILB**.
2. Définissez le certificat ILB en sélectionnant le fichier .pfx du certificat et fournissez un mot de passe. Cette étape prend un certain temps et un message indiquant qu’une opération de mise à l’échelle est en cours s’affiche.
3. Obtenez l’adresse ILB pour votre environnement App Service (**Environnement App Service -> Propriétés -> Adresse IP virtuelle**).
4. Créez une application web dans l’environnement App Service après sa création. 
5. Créez une machine virtuelle si vous n’avez pas de réseau virtuel (ne la placez pas dans le même sous-réseau que l’environnement App Service pour éviter tout dysfonctionnement).
6. Configurez le DNS de votre sous-domaine. Vous pouvez utiliser un caractère générique avec votre sous-domaine dans votre DNS ou, si vous voulez faire de simples tests, modifier le fichier hosts sur votre machine virtuelle pour définir le nom de l’application web sur l’adresse IP virtuelle. Si votre environnement App Service comprend le sous-domaine .ilbase.com et si vous avez utilisé mytestapp pour votre application web afin de l’adresser vers mytestapp.ilbase.com, définissez cette valeur dans votre fichier hosts. (Sur Windows, le fichier hosts se situe sous C:\Windows\System32\drivers\etc\)
7. Utilisez un navigateur sur cette machine virtuelle et accédez à `https://mytestapp.ilbase.com` (ou le nom de votre application web avec votre sous-domaine).
8. Utilisez un navigateur sur cette machine virtuelle et accédez à `https://mytestapp.ilbase.com`. Si vous utilisez un certificat auto-signé, vous devez accepter le manque de sécurité. 

L’adresse IP de votre ILB est répertoriée dans vos propriétés en tant qu’adresse IP virtuelle.

![Montre que l’adresse IP de l’équilibreur ILB apparaît dans les propriétés sous la forme de l’adresse IP virtuelle.][4]

## <a name="using-an-ilb-ase"></a>Utilisation d’un ILB ASE
#### <a name="network-security-groups"></a>Network Security Group
Un environnement App Service ILB permet l’isolement réseau de vos applications. Les applications ne sont pas accessibles ou même connues d’Internet. Cette méthode est excellente pour l’hébergement de sites intranet comme les applications métier. Si vous avez besoin de restreindre davantage l’accès, vous pouvez toujours utiliser des groupes de sécurité réseau (NSG) pour contrôler l’accès au niveau du réseau. 

Si vous souhaitez utiliser des NSG pour restreindre davantage l’accès, vous devez veiller à ne pas interrompre la communication nécessaire au fonctionnement de l’environnement App Service. Bien que l’accès HTTP/HTTPS s’effectue uniquement par le biais de l’ILB utilisé par l’environnement App Service, cet environnement dépend toujours de ressources situées en dehors du réseau virtuel. Pour savoir quel accès réseau est encore nécessaire, consultez [Contrôle du trafic entrant vers un environnement App Service][ControlInbound] et [Détails de la configuration réseau pour les environnements App Service avec ExpressRoute][ExpressRoute]. 

Pour configurer vos NSG, vous devez connaître l’adresse IP utilisée par Azure pour gérer votre environnement App Service. Cette adresse IP est également l’adresse IP sortante de votre ASE s’il effectue des demandes Internet. L’adresse IP sortante pour votre environnement App Service reste statique pendant toute la durée de vie de l’environnement. Si vous supprimez et recréez votre ASE, vous obtenez une nouvelle adresse IP. Pour trouver cette adresse IP, sélectionnez **Paramètres -> Propriétés**, puis **Adresse IP sortante**. 

![Montre où trouver l’adresse IP sortante de l’environnement ASE.][5]

#### <a name="general-ilb-ase-management"></a>Gestion générale de l’ILB ASE
La gestion d’un ILB ASE est largement identique à la gestion d’un ASE standard. Vous devez effectuer une montée en puissance de vos pools de workers pour héberger plusieurs instances de plans App Service et faire de même pour vos serveurs frontend dans le but de gérer la hausse du trafic HTTP/HTTPS. Pour obtenir des informations générales sur la gestion de la configuration d’un environnement App Service, consultez [Configuration d’un environnement App Service][ASEConfig]. 

Les éléments d’administration supplémentaires sont la gestion des certificats et la gestion DNS. Vous devez obtenir et charger le certificat utilisé pour le protocole HTTPS après la création de l’environnement App Service ILB, et le remplacer avant son expiration. Étant donné qu’Azure est propriétaire du domaine de base, il peut fournir des certificats pour les environnements App Service ayant une adresse IP externe. Étant donné que le sous-domaine utilisé par un environnement App Service ILB peut être quelconque, vous devez fournir votre propre certificat pour le protocole HTTPS. 

#### <a name="dns-configuration"></a>Configuration DNS
Lorsque vous utilisez une adresse IP virtuelle externe, le service DNS est géré par Azure. Toute application créée dans votre environnement ASE est automatiquement ajoutée au service Azure DNS, qui est un service DNS public. Dans un environnement ASE ILB, vous devez gérer votre propre service DNS. Pour un sous-domaine spécifique comme contoso.corp.net, vous devez créer des enregistrements DNS A qui pointent vers votre adresse ILB :

- \*
- *.scm
- ftp
- Publier

## <a name="getting-started"></a>Prise en main
Pour prendre en main les environnements App Service, consultez [la présentation des environnements App Service][WhatisASE].

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!--Image references-->
[1]: ./media/app-service-environment-with-internal-load-balancer/ilbase-createilbase.png
[2]: ./media/app-service-environment-with-internal-load-balancer/ilbase-createapp.png
[3]: ./media/app-service-environment-with-internal-load-balancer/ilbase-newase.png
[4]: ./media/app-service-environment-with-internal-load-balancer/ilbase-vip.png
[5]: ./media/app-service-environment-with-internal-load-balancer/ilbase-externalvip.png
[6]: ./media/app-service-environment-with-internal-load-balancer/ilbase-ilbcertificate.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[HowtoCreateASE]: app-service-web-how-to-create-an-app-service-environment.md
[ControlInbound]: app-service-app-service-environment-control-inbound-traffic.md
[virtualnetwork]: ../../virtual-network/virtual-networks-faq.md
[AppServicePricing]: https://azure.microsoft.com/pricing/details/app-service/
[ASEAutoscale]: app-service-environment-auto-scale.md
[ExpressRoute]: app-service-app-service-environment-network-configuration-expressroute.md
[vnetnsgs]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[ASEConfig]: app-service-web-configure-an-app-service-environment.md