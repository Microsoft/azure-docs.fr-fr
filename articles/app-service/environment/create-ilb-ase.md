---
title: Créer un environnement ASE ILB avec ARM
description: Découvrez comment créer un environnement ASE (App Service Environment) avec un équilibreur de charge interne (ILB) à l’aide des modèles Azure Resource Manager. Isolez complètement vos applications d’Internet.
author: ccompy
ms.assetid: 0f4c1fa4-e344-46e7-8d24-a25e247ae138
ms.topic: quickstart
ms.date: 09/16/2020
ms.author: ccompy
ms.custom: mvc, seodec18
ms.openlocfilehash: 27c9198558a730d0af49077d6f5baa6db4789416
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009549"
---
# <a name="create-and-use-an-internal-load-balancer-app-service-environment"></a>Créer et utiliser un environnement App Service Environment avec équilibreur de charge interne 

L’environnement Azure App Service est un déploiement d’Azure App Service dans un sous-réseau de réseau virtuel Azure. Il existe deux façons de déployer un environnement App Service (ASE, App Service Environment) : 

- avec une adresse IP virtuelle sur une adresse IP externe, solution souvent appelée ASE externe ;
- avec une adresse IP virtuelle sur une adresse IP interne, solution souvent appelée ASE ILB, car le point de terminaison interne est un équilibreur de charge interne (ILB, Internal Load Balancer). 

Cet article explique comment créer un ASE ILB. Pour une présentation de l’ASE, consultez [Présentation des environnements App Service][Intro]. Pour savoir comment créer un ASE externe, consultez [Créer un environnement App Service externe][MakeExternalASE].

## <a name="overview"></a>Vue d’ensemble 

Vous pouvez déployer un ASE avec un point de terminaison accessible via Internet ou avec une adresse IP de votre réseau virtuel. Pour définir l’adresse IP sur une adresse de réseau virtuel, l’ASE doit être déployé avec un ILB. Lorsque vous déployez votre ASE avec un ILB, vous devez indiquer le nom de votre ASE. Le nom de votre ASE est utilisé dans le suffixe du domaine pour les applications dans votre ASE.  Le suffixe du domaine pour votre ASE ILB est &lt;nom ASE&gt;.appserviceenvironment.net. Les applications qui sont créées dans un ASE ILB ne sont pas placées dans le DNS public. 

Dans les versions antérieures de l’ASE ILB, vous deviez indiquer un suffixe de domaine et un certificat par défaut pour les connexions HTTPS. Le suffixe de domaine n’est plus collecté lors de la création de l’ASE ILB et aucun certificat par défaut n’est collecté. À présent, lorsque vous créez un ASE ILB, le certificat par défaut est fourni par Microsoft et est approuvé par le navigateur. Vous pouvez toujours définir des noms de domaine personnalisés pour les applications dans votre ASE et définir des certificats sur ces noms de domaine personnalisés. 

Avec un ASE ILB, vous pouvez effectuer des tâches telles que :

-   Héberger des applications intranet en toute sécurité dans le cloud, auquel vous accédez via un réseau de site à site ou ExpressRoute.
-   Protéger les applications à l’aide d’un appareil WAF
-   héberger des applications dans le cloud qui ne figurent pas dans les serveurs DNS publics ;
-   créer des applications principales isolées d’Internet auxquelles vos applications frontales peuvent s’intégrer en toute sécurité.

### <a name="disabled-functionality"></a>Fonctionnalités désactivées ###

Lorsque vous utilisez un ASE ILB, vous ne pouvez pas effectuer certaines opérations :

-   utiliser SSL basé sur IP ;
-   attribuer des adresses IP à des applications spécifiques ;
-   acheter et utiliser un certificat avec une application via le portail Azure. Vous pouvez obtenir des certificats directement auprès d’une autorité de certification et les utiliser avec vos applications. Vous ne pouvez pas les obtenir via le portail Azure.

## <a name="create-an-ilb-ase"></a>Créer un environnement ASE ILB ##

Pour créer un ILB ASE :

1. Dans le portail Azure, sélectionnez **Créer une ressource** > **Web** > **App Service Environment**.

2. Sélectionnez votre abonnement.

3. Sélectionnez ou créez un groupe de ressources.

4. Entrez le nom de votre environnement App Service Environment.

5. Sélectionnez Interne pour le type IP virtuel.

    ![Création d’un environnement App Service](media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase.png)

> [!NOTE]
> Le nom de l’environnement App Service Environment ne doit pas dépasser 37 caractères.

6. Sélectionner Mise en réseau

7. Sélectionnez ou créez un réseau virtuel. Si vous créez un nouveau réseau virtuel ici, il sera défini avec une plage d’adresses de 192.168.250.0/23. Pour créer un réseau virtuel avec une autre plage d’adresses ou dans un autre groupe de ressources que l’ASE, utilisez le portail de création du réseau virtuel Azure. 

8. Sélectionnez ou créez un sous-réseau vide. Si vous souhaitez sélectionner un sous-réseau, il doit être vide et non délégué. Une fois l’ASE créé, il n’est plus possible de modifier la taille du sous-réseau. Nous recommandons la taille `/24`, qui comprend 256 adresses et qui peut gérer un ASE de taille maximale ainsi que les besoins de mise à l’échelle. 

    ![Mise en réseau de l’ASE][1]

7. Sélectionnez **Vérifier et créer** puis sélectionnez **Créer**.


## <a name="create-an-app-in-an-ilb-ase"></a>Créer une application dans un ASE ILB ##

Pour créer une application dans un ASE ILB, procédez de la même façon que pour créer une application dans un ASE normalement.

1. Dans le portail Azure, sélectionnez **Créer une ressource** > **Web** > **Application web**.

1. Entrez le nom de l’application.

1. Sélectionnez l’abonnement.

1. Sélectionnez ou créez un groupe de ressources.

1. Sélectionnez le système de publication, la pile d’exécution et le système d’exploitation.

1. Sélectionnez un emplacement où se trouve un ASE ILB existant.  Vous pouvez également créer un ASE lors de la création d’applications en sélectionnant un plan App Service Isolé. Si vous souhaitez créer un ASE, sélectionnez dans quelle région vous souhaitez le créer.

1. Sélectionnez ou créez un plan App Service. 

1. Sélectionnez **Vérifier et créer** puis sélectionnez **Créer** lorsque vous êtes prêt.

### <a name="web-jobs-functions-and-the-ilb-ase"></a>Tâches Web, fonctions et l’environnement App Service ILB 

Les fonctions et les tâches Web sont prises en charge sur un environnement App service ILB, mais pour que le portail interagisse avec eux, vous devez disposer d’un accès réseau vers le site SCM.  Cela signifie que votre navigateur doit être sur un ordinateur hôte qui se trouve dans ou connecté au réseau virtuel. Si votre ASE ILB a un nom de domaine qui ne se termine pas par *appserviceenvironment.net*, vous devrez faire en sorte que votre navigateur approuve le certificat HTTPS utilisé par votre site scm.

## <a name="dns-configuration"></a>Configuration DNS 

Lorsque vous utilisez un environnement ASE externe, les applications effectuées dans votre ASE sont inscrites auprès d’Azure DNS. Il n’est pas nécessaire de passer par d’autres étapes dans un ASE externe pour que vos applications soient disponibles publiquement. Avec un environnement ASE ILB, vous devez gérer votre propre service DNS. Vous pouvez effectuer cette opération sur votre propre serveur DNS ou avec des zones privées Azure DNS.

Pour configurer DNS sur votre propre serveur DNS avec votre ASE ILB :

1. créez une zone pour &lt;nom ASE&gt;.appserviceenvironment.net
2. créez un enregistrement A dans cette zone qui pointe * vers l’adresse IP ILB
3. créez un enregistrement A dans cette zone qui pointe @ vers l’adresse IP ILB
4. créez une zone dans le scm nommé &lt;nom ASE&gt;.appserviceenvironment.net
5. créez un enregistrement A dans la zone scm qui pointe * vers l’adresse IP ILB

Pour configurer DNS dans les zones privées Azure DNS :

1. créez une zone privée Azure DNS nommée &lt;nom ASE&gt;.appserviceenvironment.net
2. créez un enregistrement A dans cette zone qui pointe * vers l’adresse IP ILB
3. créez un enregistrement A dans cette zone qui pointe @ vers l’adresse IP ILB
4. créez un enregistrement A dans cette zone qui pointe *.scm vers l’adresse IP ILB

Les paramètres DNS du suffixe de domaine par défaut de votre ASE ne limitent pas vos applications à être accessibles uniquement par ces noms. Vous pouvez définir un nom de domaine personnalisé sans validation sur vos applications dans un environnement ASE ILB. Si vous souhaitez ensuite créer une zone nommée contoso.net, vous pouvez le faire et la pointer vers l’adresse IP ILB. Le nom de domaine personnalisé fonctionne pour les demandes d’application, mais pas pour le site GCL. Le site GCL est disponible uniquement pour &lt;appname&gt;.scm.&lt;asename&gt;.appserviceenvironment.net.

La zone nommée .&lt;asename&gt;.appserviceenvironment.net est globalement unique. Avant mai 2019, les clients pouvaient spécifier le suffixe de domaine de l’ASE ILB. Si vous souhaitez utiliser .contoso.com comme suffixe de domaine, vous pouvez le faire et inclure le site GCL. Ce modèle présentait quelques contraintes au niveau de la gestion du certificat SSL par défaut, de l’absence d’authentification unique auprès du site GCL, et de la nécessité d’utiliser un certificat générique. Le processus de mise à niveau du certificat par défaut de l’ASE ILB entraînait également une interruption du service et le redémarrage de l’application. Pour résoudre ces problèmes, le comportement de l’ASE ILB a été modifié pour utiliser un suffixe de domaine basé sur le nom de l’ASE, avec un suffixe appartenant à Microsoft. La modification apportée au comportement de l’ASE ILB affecte uniquement les environnements ASE ILB créés après mai 2019. Les environnements ASE ILB préexistants doivent toujours gérer le certificat par défaut de l’ASE et leur configuration DNS.

## <a name="publish-with-an-ilb-ase"></a>Publier avec un ASE ILB

Pour chaque application créée, il existe deux points de terminaison. Dans un ASE ILB, vous avez *&lt;nom d’application&gt;.&lt;Domaine ASE ILB&gt;* et *&lt;nom d’application&gt;.scm&lt;Domaine ASE ILB&gt;* . 

Le nom du site SCM vous dirige vers la console Kudu nommée **Portail avancé** au sein du portail Azure. La console Kudu vous permet d’afficher des variables d’environnement, d’explorer le disque, d’utiliser une console, et bien plus encore. Pour plus d’informations, consultez [Console Kudu pour Azure App Service][Kudu]. 

Les systèmes d’intégration continue basés sur Internet, comme GitHub et Azure DevOps, continueront de fonctionner avec un environnement ASE d’équilibreur de charge interne si l’agent de build est accessible par Internet et se trouve sur le même réseau que l’environnement ASE d’équilibreur de charge interne. Par conséquent, avec Azure DevOps, si l’agent de build est créé sur le même réseau virtuel que l’environnement ASE d’équilibreur de charge interne (vous pouvez utiliser un autre sous-réseau), il ne pourra pas extraire le code d’Azure DevOps Git et se déployer dans l’environnement ASE d’équilibreur de charge interne. Si vous ne souhaitez pas créer votre propre agent de build, vous devez utiliser un système d’intégration continue qui utilise un modèle d’extraction, par exemple Dropbox.

Les points de terminaison de publication pour les applications d’un environnement ASE d’équilibreur de charge interne utilisent le domaine avec lequel l’environnement ASE d’équilibreur de charge interne a été créé. Ce domaine apparaît dans le profil de publication de l’application et sur le panneau du portail de l’application (**Vue d’ensemble** > **Bases** et également **Propriétés**). Si vous avez un ASE ILB avec le suffixe de domaine *&lt;nom ASE&gt;.appserviceenvironment.net* et une application nommée *mytest*, utilisez *mytest.&lt;nom ASE&gt;.appserviceenvironment.net* pour FTP et *mytest.scm.contoso.net* pour le déploiement web.

## <a name="configure-an-ilb-ase-with-a-waf-device"></a>Configurer un ASE ILB avec un appareil WAF ##

Vous pouvez combiner un appareil de pare-feu d’application web (WAF) avec votre ASE ILB pour exposer uniquement les applications souhaitées sur Internet et conserver le reste accessible uniquement à partir du réseau virtuel. Cela vous permet de créer des applications à plusieurs niveaux sécurisées, entre autres choses.

Pour en savoir plus sur la configuration de votre ASE ILB avec un appareil WAF, consultez [Configurer un pare-feu d’applications web avec votre environnement App Service][ASEWAF]. Cet article explique comment utiliser une appliance virtuelle Barracuda avec votre ASE. Une autre option consiste à utiliser Azure Application Gateway. Azure Application Gateway utilise les règles de base OWASP pour sécuriser les applications placées derrière lui. Pour plus d’informations sur Application Gateway, consultez [Introduction au pare-feu d’applications web Azure][AppGW].

## <a name="ilb-ases-made-before-may-2019"></a>ASE ILB créés avant mai 2019

Pour les ASE ILB créés avant mai 2019, vous deviez définir le suffixe de domaine lors de la création de l’ASE. Il était également nécessaire de charger un certificat par défaut qui était basé sur ce suffixe de domaine. En outre, avec un ASE ILB plus ancien vous ne pouvez pas effectuer l’authentification unique sur la console Kudu avec des applications dans cet ASE ILB. Lors de la configuration de DNS pour un ASE ILB plus ancien, vous devez définir l’enregistrement A de caractère générique dans une zone correspondant à votre suffixe de domaine. 

## <a name="get-started"></a>Bien démarrer ##

* Pour bien démarrer avec les ASE, consultez [Présentation des environnements App Service][Intro]. 

<!--Image references-->
[1]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-network.png
[2]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-webapp.png
[5]: ./media/creating_and_using_an_internal_load_balancer_with_app_service_environment/createilbase-ipaddresses.png

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
[webapps]: ../overview.md
[mobileapps]: /previous-versions/azure/app-service-mobile/app-service-mobile-value-prop
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[customdomain]: ../app-service-web-tutorial-custom-domain.md
[linuxapp]: ../overview.md#app-service-on-linux
