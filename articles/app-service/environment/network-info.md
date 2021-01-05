---
title: Mise en réseau - Éléments à prendre en compte
description: Découvrez le trafic réseau ASE et comment définir des groupes de sécurité réseau et des routes définies par l’utilisateur avec votre environnement ASE.
author: ccompy
ms.assetid: 955a4d84-94ca-418d-aa79-b57a5eb8cb85
ms.topic: article
ms.date: 07/27/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 91b6134e7c809a8af75aa1cf23523e352e0a1a0e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997339"
---
# <a name="networking-considerations-for-an-app-service-environment"></a>Considérations relatives à la mise en réseau pour un environnement App Service Environment #

## <a name="overview"></a>Vue d’ensemble ##

 [App Service Environment Azure][Intro] est un déploiement d’Azure App Service dans un sous-réseau d’un réseau virtuel Azure (VNet). Il existe deux types de déploiement pour un environnement App Service (ASE) :

- **ASE externe** : expose les applications hébergées dans l’ASE sur une adresse IP accessible via Internet. Pour plus d’informations, consultez [Créer un App Service Environment externe][MakeExternalASE].
- **ASE ILB** : expose les applications hébergées dans l’ASE sur une adresse IP à l’intérieur de votre réseau virtuel. Le point de terminaison interne est un équilibreur de charge interne (ILB), d’où cette appellation d’ASE ILB. Pour plus d’informations, consultez [Créer et utiliser un ASE ILB][MakeILBASE].

Tous les environnements ASE, externes et ILB ont une adresse IP virtuelle publique, qui est utilisée pour le trafic de gestion entrant et en tant qu’adresse de départ lors l’appels de l’ASE vers Internet. Tous les appels d’un ASE à destination d’Internet quittent le réseau virtuel via une adresse IP virtuelle assignée à l’ASE. L’adresse IP publique de cette adresse IP virtuelle constitue l’adresse IP source de tous les appels de l’ASE à destination d’Internet. Si les applications hébergées dans votre environnement ASE appellent des ressources de votre réseau virtuel ou hébergées dans un VPN, l’adresse IP source sera l’une des adresses IP du sous-réseau utilisé par votre environnement ASE. Comme l’ASE se trouve à l’intérieur du réseau virtuel, il peut également accéder aux ressources de celui-ci sans configuration supplémentaire. Si le réseau virtuel est connecté à votre réseau local, les applications dans votre environnement ASE ont également accès aux ressources sans configuration supplémentaire.

![ASE externe][1] 

Si vous avez un ASE externe, l’adresse IP virtuelle publique est également le point de terminaison que vos applications ASE peuvent résoudre :

* HTTP/S 
* FTP/S
* Déploiement web
* Débogage à distance

![ASE ILB][2]

Si vous possédez un ASE ILB, l’adresse de l’équilibreur de charge interne est le point de terminaison pour les protocoles HTTP/S et FTP/S, le déploiement web et le débogage à distance.

## <a name="ase-subnet-size"></a>Taille du sous-réseau de l’ASE ##

La taille du sous-réseau utilisé pour héberger un ASE ne peut pas être modifiée une fois l’ASE déployé.  L’ASE utilise une adresse pour chaque rôle d’infrastructure, ainsi que pour chaque instance de plan App Service Isolé.  De plus, cinq adresses sont utilisées par Azure Networking pour chaque sous-réseau créé.  Un ASE sans aucun plan App Service utilise 12 adresses avant la création d’une application.  S’il s’agit d’un ASE ILB, il utilise 13 adresses avant la création d’une application dans cet environnement. Lorsque vous effectuez un scale-out de votre ASE, les rôles d’infrastructure sont ajoutés à chaque fois que les instances de votre plan App Service sont multipliées par 15 ou 20.

   > [!NOTE]
   > Le sous-réseau doit contenir uniquement l’ASE. Veillez à choisir un espace d’adressage qui permet une croissance future. Vous ne pouvez pas modifier ce paramètre par la suite. Nous vous recommandons une taille de `/24` avec 256 adresses.

Lorsque vous montez ou descendez en puissance, de nouveaux rôles (de taille correspondante) sont ajoutés, puis vos charges de travail sont migrées depuis la taille actuelle vers la taille cible. Les machines virtuelles d’origine sont supprimées uniquement une fois les charges de travail migrées. Si vous disposiez d’un ASE comportant 100 instances ASP, vous devez doubler le nombre de machines virtuelles pendant une certaine période.  C’est pourquoi nous vous recommandons d’utiliser « /24 » pour prendre en compte toutes les potentielles modifications.  

## <a name="ase-dependencies"></a>Dépendances d’un ASE ##

### <a name="ase-inbound-dependencies"></a>Dépendances entrantes d’un ASE ###

Pour que l’ASE puisse fonctionner, les ports suivants doivent être ouverts :

| Utilisation | À partir | À |
|-----|------|----|
| Gestion | Adresses de gestion App Service | Sous-réseau de l’ASE : 454, 455 |
|  Communications internes de l’ASE | Sous-réseau de l’ASE : Tous les ports | Sous-réseau de l’ASE : Tous les ports
|  Autoriser le trafic entrant provenant d’Azure Load Balancer | Équilibrage de charge Azure | Sous-réseau de l’ASE : 16001

Il existe 2 autres ports qui peuvent s’afficher comme ouverts lors d’une analyse des ports : 7654 et 1221. Ils répondent avec une adresse IP, rien de plus. Ils peuvent être bloqués, si vous le souhaitez. 

Le trafic de gestion entrant fournit la commande et le contrôle de l’ASE en plus de la surveillance du système. Les adresses sources pour ce trafic sont répertoriées dans le document [Adresses de gestion App Service Environment][ASEManagement]. La configuration de la sécurité réseau doit autoriser l’accès des adresses de gestion de l’ASE sur les ports 454 et 455. Si vous bloquez l’accès à partir de ces adresses, vous mettez en péril l’intégrité de votre ASE qui sera suspendu. Le trafic TCP qui arrive sur les ports 454 et 455 doit repasser par la même adresse IP virtuelle, sans quoi vous rencontrerez un problème de routage asymétrique. 

Le sous-réseau de l’ASE comprend divers ports utilisés pour la communication des composants internes ; ces ports peuvent changer. Tous les ports du sous-réseau de l’ASE doivent être accessibles à partir du sous-réseau de l’ASE. 

Pour permettre la communication entre l’équilibreur de charge Azure et le sous-réseau de l’ASE, les ports 454, 455 et 16001 (au minimum) doivent être ouverts. Le port 16001 sert à maintenir le trafic entre l’équilibreur de charge et le sous-réseau de l’ASE. Si vous utilisez un ASE ILB, vous pouvez limiter le trafic aux ports 454, 455 et 16001.  Si vous utilisez un ASE externe, vous devez prendre en compte les ports d’accès aux applications normaux.  

Les autres ports dont vous devez vous préoccuper sont les ports d’applications :

| Utilisation | Ports |
|----------|-------------|
|  HTTP/HTTPS  | 80, 443 |
|  FTP/FTPS    | 21, 990, 10001-10020 |
|  Débogage distant de Visual Studio  |  4020, 4022, 4024 |
|  Service Web Deploy | 8172 |

Si vous bloquez les ports d’applications, votre ASE peut continuer de fonctionner, mais votre application risque de ne plus être fonctionnelle.  Si vous utilisez des adresses IP affectées par l’application avec un ASE externe, vous devez autoriser le trafic entre les adresses IP affectées à vos applications au sous-réseau ASE sur les ports répertoriés dans le portail ASE, sur la page relative aux adresses IP.

### <a name="ase-outbound-dependencies"></a>Dépendances sortantes d’un ASE ###

Pour l’accès sortant, un ASE dépend de plusieurs systèmes externes. Nombre de ces dépendances système sont définies avec des noms DNS et ne sont pas mappées à un ensemble fixe d’adresses IP. Par conséquent, l’ASE requiert un accès sortant vers toutes les adresses IP sur divers ports à partir de son sous-réseau. 

L’ASE communique avec les adresses Internet accessibles sur les ports suivants :

| Utilisations | Ports |
|-----|------|
| DNS | 53 |
| NTP | 123 |
| CRL, mises à jour Windows, dépendances Linux, services Azure | 80/443 |
| Azure SQL | 1433 | 
| Surveillance | 12 000 |

La liste des dépendances sortantes est disponible dans le document portant sur le [verrouillage du trafic sortant d’App Service Environment](./firewall-integration.md). Si l’ASE perd l’accès à ses dépendances, il cesse de fonctionner. Si cela dure plus d’un certain temps, l’ASE est suspendu. 

### <a name="customer-dns"></a>DNS client ###

Si le réseau virtuel est configuré avec un serveur DNS défini par un client, les charges de travail du client l’utilisent. L’ASE utilise Azure DNS à des fins de gestion. Si le réseau virtuel est configuré avec un serveur DNS sélectionné par le client, le serveur DNS doit être accessible à partir du sous-réseau contenant l’ASE.

Pour tester la résolution DNS de votre application web, vous pouvez utiliser la commande de console *nameresolver*. Accédez à la fenêtre de débogage du site scm de votre application ou accédez à l’application dans le portail, puis sélectionnez la console. À partir de l’invite de l’interpréteur de commandes, vous pouvez émettre la commande *nameresolver*, avec le nom DNS que vous souhaitez rechercher. Le résultat que vous obtenez est identique à celui qu’obtiendrait l’application pour la même recherche. Si vous utilisez nslookup, vous effectuez plutôt une recherche à l’aide d’Azure DNS.

Si vous modifiez le paramètre DNS du réseau virtuel dans lequel figure votre environnement ASE, vous devrez redémarrer l’ASE. Pour éviter de redémarrer l’ASE, il est vivement recommandé de configurer les paramètres DNS de votre réseau virtuel avant de créer l’ASE.  

<a name="portaldep"></a>

## <a name="portal-dependencies"></a>Dépendances du portail ##

Outre les dépendances fonctionnelles que présente un ASE, il existe quelques éléments supplémentaires liés à l’utilisation du portail. Certaines des fonctionnalités du portail Azure dépendent d’un accès direct au _site du Gestionnaire de contrôle des services (SCM)_ . Pour chaque application dans Azure App Service, il existe deux URL. La première URL sert à accéder à votre application. La seconde permet d’accéder au site SCM, également désigné sous le nom de _console Kudu_. Voici quelques-unes des fonctionnalités qui utilisent le site SCM :

-   Tâches web
-   Fonctions
-   Diffusion de journaux
-   Kudu
-   Extensions
-   Process Explorer
-   Console

Lorsque vous utilisez un ASE ILB, le site SCM n’est pas accessible depuis l’extérieur du réseau virtuel. Certaines fonctionnalités ne fonctionneront pas sur le portail de l’application, car elles nécessitent un accès au site SCM d’une application. Vous pouvez vous connecter au site SCM directement, au lieu de passer par le portail. 

Si votre ASE ILB est le nom de domaine *contoso.appserviceenvironnment.net* et que le nom de votre application est *testapp*, le système atteint l’application sur *testapp.contoso.appserviceenvironment.net*. Le système atteint le site SCM qui le suit sur *testapp.scm.contoso.appserviceenvironment.net*.

## <a name="ase-ip-addresses"></a>Adresses IP d’un ASE ##

Un ASE présente quelques adresses IP qu’il est important de connaître. Il s'agit de :

- **Adresse IP entrante publique** : utilisée pour le trafic d’applications dans un ASE externe et pour le trafic de gestion aussi bien dans un ASE externe que dans un ASE ILB.
- **Adresse IP publique sortante**  : utilisée en tant qu’adresse IP source pour les connexions sortantes de l’ASE quittant le réseau virtuel, qui ne sont pas acheminées via un VPN.
- **Adresse IP de l’ILB** : L’adresse IP de l’ILB existe uniquement dans un ASE ILB.
- **Adresse SSL basée sur IP attribuée par l’application** : uniquement possibles avec un ASE externe et lorsque le mode SSL basé sur IP est configuré.

Toutes ces adresses IP sont visibles dans le portail Microsoft Azure, à partir de l’interface utilisateur ASE. Si vous possédez un ASE ILB, l’adresse IP de l’ILB est répertoriée.

   > [!NOTE]
   > Ces adresses IP ne changent pas tant que votre ASE est en cours d’exécution.  Si votre ASE est suspendu et restauré, les adresses utilisées par votre ASE sont modifiées. En général, la suspension d’un ASE est due au blocage de l’accès de gestion entrant ou de l’accès à une dépendance de l’ASE. 

![Adresses IP][3]

### <a name="app-assigned-ip-addresses"></a>Adresses IP attribuées par l’application ###

Avec un ASE externe, vous pouvez assigner des adresses IP à des applications individuelles. Ce n’est pas possible avec un ASE ILB. Pour savoir comment configurer votre application de sorte qu’elle possède sa propre adresse IP, consultez [Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service](../configure-ssl-bindings.md).

Lorsqu’une application possède sa propre adresse SSL basée sur IP, l’ASE réserve deux ports pour le mappage à cette adresse IP. Un port est destiné au trafic HTTP et l’autre au trafic HTTPS. Ces ports sont répertoriés dans l’interface utilisateur de l’ASE, dans la section des adresses IP. Le trafic doit pouvoir atteindre ces ports à partir de l’adresse IP virtuelle. Sinon, les applications ne sont pas accessibles. Il est important de ne pas oublier cela lorsque vous configurez des groupes de sécurité réseau (NSG).

## <a name="network-security-groups"></a>Network Security Group ##

Les [groupes de sécurité réseau][NSGs] permettent de contrôler l’accès réseau au sein d’un réseau virtuel. Lorsque vous utilisez le portail, il existe une règle de refus implicite au niveau de priorité le plus bas qui fait que tout accès est refusé. Ce que vous créez sont vos règles d’autorisation.

Dans un ASE, vous n’avez pas accès aux machines virtuelles utilisées pour héberger l’ASE lui-même. Elles se trouvent dans un abonnement géré par Microsoft. Si vous souhaitez restreindre l’accès aux applications dans l’ASE, définissez les groupes de sécurité réseau sur le sous-réseau de l’ASE. Ce faisant, vous devez prêter une attention particulière aux dépendances de l’ASE. Si vous bloquez certaines dépendances, l’ASE cesse de fonctionner.

Les groupes de sécurité réseau peuvent être configurés à l’aide du portail Azure ou via PowerShell. Seul le portail Azure est illustré ici. Les groupes de sécurité réseau sont créés et gérés en tant que ressources de niveau supérieur dans la section **Mise en réseau** du portail.

Pour permettre à un ASE de fonctionner, les entrées nécessaires dans un groupe de sécurité réseau doivent autoriser le trafic :

**Entrant**
* Protocole TCP de la balise de service IP AppServiceManagement sur les ports 454, 455
* Protocole TCP depuis l’équilibreur de charge sur le port 16001
* depuis le sous-réseau ASE vers le sous-réseau ASE, sur tous les ports.

**Sortante**
* UDP vers toutes les adresses IP sur le port 53
* UDP vers toutes les adresses IP sur le port 123
* Protocole TCP vers toutes les adresses IP sur les ports 80 et 443
* Protocole TCP vers l’étiquette de service IP `Sql` sur le port 1433
* Protocole TCP vers toutes les adresses IP sur le port 12000
* vers le sous-réseau ASE sur tous les ports.

Ces ports n’incluent pas ceux dont vos applications ont besoin pour fonctionner correctement. Par exemple, votre application peut avoir besoin d’appeler un serveur MySQL sur le port 3306. Le protocole Network Time Protocol (NTP) sur le port 123 est le protocole de synchronisation date/heure utilisé par le système d'exploitation. Les points de terminaison NTP ne sont pas spécifiques à App Services, ils peuvent varier avec le système d’exploitation et ne figurent pas dans une liste d’adresses bien définie. Pour éviter les problèmes de synchronisation date/heure, vous devez autoriser le trafic UDP vers toutes les adresses sur le port 123. Le trafic TCP sortant du port 12000 est destiné à la prise en charge et à l’analyse du système. Les points de terminaison sont dynamiques et ne se trouvent pas dans un ensemble bien défini d’adresses.

Les ports d’accès normaux pour les applications sont les suivants :

| Utilisation | Ports |
|----------|-------------|
|  HTTP/HTTPS  | 80, 443 |
|  FTP/FTPS    | 21, 990, 10001-10020 |
|  Débogage distant de Visual Studio  |  4020, 4022, 4024 |
|  Service Web Deploy | 8172 |

En tenant compte des exigences liées au trafic entrant et sortant, les groupes de sécurité réseau doivent se ressembler aux groupes de sécurité réseau présentés dans cet exemple. 

![Règles de sécurité de trafic entrant][4]

Une règle par défaut permet la communication entre les adresses IP dans le réseau virtuel avec le sous-réseau ASE. Une autre règle par défaut permet la communication entre l’équilibrage de charges, également appelé l’adresse IP virtuelle publique, et l’ASE. Vous pouvez afficher les règles par défaut en sélectionnant **Règles par défaut** en regard de l’icône **Ajouter**. Si vous placez une règle de refus pour toute autre communication avant les règles par défaut, vous bloquez le trafic entre l’adresse IP virtuelle et l’ASE. Pour éviter le trafic provenant de l’intérieur du réseau virtuel, ajoutez votre propre règle pour autoriser le trafic entrant. Utilisez une source égale à AzureLoadBalancer avec une destination **Tout** et une plage de ports *\** _. Étant donné que la règle de groupes de sécurité réseau est appliquée au sous-réseau de l’ASE, vous n’avez pas besoin de définir une destination spécifique.

Si vous avez attribué une adresse IP à votre application, assurez-vous que vous conservez les ports ouverts. Vous pouvez consulter les ports utilisés en sélectionnant *App Service Environment** > **Adresses IP**.  

Tous les éléments affichés dans les règles de trafic sortant suivants sont nécessaires, à l’exception du dernier élément. Ils autorisent l’accès réseau aux dépendances de l’ASE décrites plus haut dans ce document. Si vous bloquez un d'entre eux, votre ASE cesse de fonctionner. Le dernier élément de la liste autorise votre ASE à communiquer avec les autres ressources de votre réseau virtuel.

![Règles de sécurité de trafic entrant][5]

Une fois vos groupes de sécurité réseau définis, vous devez les attribuer au sous-réseau dans lequel se trouve votre ASE. Si vous ne connaissez pas le réseau virtuel ou le sous-réseau de l’ASE, vous pouvez l’afficher dans la page du portail de l’ASE. Pour assigner le groupe de sécurité réseau à votre sous-réseau, accédez à l’interface utilisateur du sous-réseau et sélectionnez le groupe de sécurité réseau.

## <a name="routes"></a>Itinéraires ##

Le tunneling forcé consiste à définir des itinéraires dans un réseau virtuel de façon à ce que le trafic sortant n’accède pas directement à Internet, mais soit dirigé ailleurs, par exemple vers une passerelle ExpressRoute ou une appliance virtuelle.  Si vous souhaitez configurer ainsi votre ASE, lisez le document [Configurer un environnement App Service Environment avec le tunneling forcé][forcedtunnel].  Il vous indiquera les options possibles pour travailler avec ExpressRoute et le tunneling forcé.

Lorsque vous créez un environnement ASE sur le portail, nous créons également différentes tables d’itinéraires sur le sous-réseau créé avec l’environnement.  Ces itinéraires donnent simplement l’instruction d’envoyer directement le trafic sortant sur Internet.  
Pour créer les mêmes itinéraires manuellement, procédez ainsi :

1. Accédez au portail Azure. Sélectionnez **Mise en réseau** > **Tables d’itinéraires**.

2. Créez une nouvelle table d’itinéraires dans la même région que votre réseau virtuel.

3. À partir de l’interface utilisateur de votre table d’itinéraires, sélectionnez **Itinéraires** > **Ajouter**.

4. Définissez le **Type de tronçon suivant** sur **Internet** et **Préfixe de l’adresse** sur **0.0.0.0/0**. Sélectionnez **Enregistrer**.

    Le résultat suivant s’affiche :

    ![Itinéraires fonctionnels][6]

5. Après avoir créé la nouvelle table d’itinéraires, accédez au sous-réseau contenant votre ASE. Dans la liste du portail, sélectionnez votre table d’itinéraires. Une fois la modification enregistrée, les groupes de sécurité réseau et les itinéraires s’affichent dans les informations de votre sous-réseau.

    ![Itinéraires et groupes de sécurité réseau][7]

## <a name="service-endpoints"></a>Points de terminaison de service ##

Les points de terminaison de service vous permettent de restreindre l’accès aux services multilocataires à un ensemble de sous-réseaux et de réseaux virtuels Azure. Pour en savoir plus sur les points de terminaison de service, consultez la documentation [Points de terminaison de service de réseau virtuel][serviceendpoints]. 

Lorsque vous activez les points de terminaison de service sur une ressource, certains itinéraires sont créés avec une priorité plus élevée que d’autres. Si vous utilisez des points de terminaison de service sur n’importe quel service Azure avec un ASE avec tunneling forcé, le tunneling du trafic vers ces services n’est pas forcé. 

Lorsque les points de terminaison de service sont activés sur un sous-réseau avec une instance Azure SQL, toutes les instances Azure SQL connectées à partir de ce sous-réseau doivent avoir des points de terminaison de service activés. Si vous souhaitez accéder à plusieurs instances Azure SQL à partir du même sous-réseau, vous ne pouvez pas activer les points de terminaison de service sur une instance Azure SQL et pas sur une autre. Aucun autre service Azure ne se comporte comme SQL Azure en ce qui concerne les points de terminaison de service. Lorsque vous activez les points de terminaison de service avec Stockage Azure, vous verrouillez l’accès à cette ressource à partir de votre sous-réseau, mais pourrez toujours accéder aux autres comptes Stockage Azure même si les points de terminaison de service ne sont pas activés.  

![Points de terminaison de service][8]

<!--Image references-->
[1]: ./media/network_considerations_with_an_app_service_environment/networkase-overflow.png
[2]: ./media/network_considerations_with_an_app_service_environment/networkase-overflow2.png
[3]: ./media/network_considerations_with_an_app_service_environment/networkase-ipaddresses.png
[4]: ./media/network_considerations_with_an_app_service_environment/networkase-inboundnsg.png
[5]: ./media/network_considerations_with_an_app_service_environment/networkase-outboundnsg.png
[6]: ./media/network_considerations_with_an_app_service_environment/networkase-udr.png
[7]: ./media/network_considerations_with_an_app_service_environment/networkase-subnet.png
[8]: ./media/network_considerations_with_an_app_service_environment/serviceendpoint.png

<!--Links-->
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/network-security-groups-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[mobileapps]: /previous-versions/azure/app-service-mobile/app-service-mobile-value-prop
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ss-cert.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[ASEManagement]: ./management-addresses.md
[serviceendpoints]: ../../virtual-network/virtual-network-service-endpoints-overview.md
[forcedtunnel]: ./forced-tunnel-support.md
[serviceendpoints]: ../../virtual-network/virtual-network-service-endpoints-overview.md