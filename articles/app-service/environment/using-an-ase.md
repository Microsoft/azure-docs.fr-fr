---
title: Utiliser et gérer un App Service Environment
description: Apprenez à créer, publier et mettre à l’échelle des applications dans Azure App Service Environment. Recherchez toutes les tâches courantes dans cet article.
author: ccompy
ms.assetid: a22450c4-9b8b-41d4-9568-c4646f4cf66b
ms.topic: article
ms.date: 5/10/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: e4653419d974db2e597111487c64682c75ffc40b
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97826002"
---
# <a name="use-an-app-service-environment"></a>Utiliser un environnement App Service

Un environnement ASE (App Service Environment) est un déploiement d’Azure App Service dans un sous-réseau dans l’instance de réseau virtuel Microsoft Azure d’un client. Un ASE se compose des éléments suivants :

- **Serveurs frontaux :** endroit où aboutissent les protocoles HTTP ou HTTPS dans App Service Environment
- **Workers** : ressources hébergeant vos applications
- **Base de données** : contient des informations qui définissent l’environnement
- **Stockage** : utilisé pour héberger les applications publiées par le client

Vous pouvez déployer un ASE avec une adresse IP virtuelle externe ou interne pour l’accès des applications. Un déploiement avec une adresse IP virtuelle externe est communément appelé *ASE externe*. Un déploiement avec une adresse IP virtuelle interne est appelé *ASE d’équilibrage de charge interne*, car il utilise un équilibreur de charge interne (ILB). Pour plus d’informations sur l'environnement ASE d'équilibrage de charge interne, consultez [Créer et utiliser un équilibreur de charge interne avec un environnement App Service][MakeILBASE].

## <a name="create-an-app-in-an-ase"></a>Créer une application dans un ASE

Pour créer une application dans un environnement ASE, vous utilisez le même processus que lorsque vous créez normalement une application, mais avec quelques petites différences. Lorsque vous créez un plan App Service :

- Au lieu de choisir un emplacement géographique dans lequel déployer votre application, vous choisissez un environnement ASE comme emplacement.
- Tous les plans App Service créés dans ASE doivent être dans un niveau tarifaire Isolé.

Si vous n’avez pas d’environnement ASE, vous pouvez en créer un en suivant les instructions de l’article [Créer un environnement Azure App Service Environment][MakeExternalASE].

Pour créer une application dans un ASE :

1. Sélectionnez **Créer une ressource** > **Web + Mobile** > **Application web**.

1. Entrez un nom pour l’application. Si vous avez déjà sélectionné un plan App Service dans un environnement ASE, le nom de domaine pour l’application reflète le nom de domaine de l’environnement ASE :

    ![Sélection du nom de l’application][1]

1. Sélectionnez un abonnement.

1. Entrez un nom pour un nouveau groupe de ressources ou sélectionnez **Use existing** (Utiliser existant) et choisissez-en un dans la liste déroulante.

1. Sélectionnez votre OS.

1. Sélectionnez un plan App Service existant dans votre environnement ASE ou créez-en un en procédant comme suit :

    a. Dans le menu de gauche du Portail Azure, sélectionnez **Créer une ressource > Application web**.

    b. Sélectionnez l’abonnement.

    c. Sélectionnez ou créez le groupe de ressources.

    d. Entrez le nom de votre application web.

    e. Sélectionnez **Code** ou **DockerContainer**.

    f. Sélectionnez une pile de runtime.

    g. Sélectionnez **Linux** ou **Windows**. 

    h. Sélectionnez votre ASE dans la liste déroulante **Région**. 

    i. Sélectionnez ou créez un plan App Service. Si vous créez un plan de App Service, sélectionnez la taille de référence SKU **Isolée** appropriée.

    ![Niveaux tarifaires isolés][2]

    > [!NOTE]
    > Les applications Linux et les applications Windows ne peuvent pas se trouver dans le même plan App Service, mais elles peuvent être dans le même environnement ASE.
    >

1. Sélectionnez **Vérifier + créer**, vérifiez si les informations sont correctes, puis sélectionnez **Créer**.

## <a name="how-scale-works"></a>Fonctionnement de la mise à l’échelle

Chaque application App Service s’exécute dans un plan App Service. Les environnements App Service contiennent les plans App Service, et les plans App Service contiennent les applications. Lorsque vous mettez à l’échelle une application, vous mettez à l’échelle v le plan App Service, et toutes les applications contenues dans ce même plan.

Lorsque vous mettez à l’échelle un plan App Service, l’infrastructure nécessaire est automatiquement ajoutée. Il existe un délai pour les opérations de mise à l’échelle lorsque l’infrastructure est ajoutée. Si vous effectuez plusieurs opérations de mise à l’échelle de façon séquentielle, la première requête de mise à l’échelle de l’infrastructure est traitée et les autres sont mises en file d’attente. Une fois la première opération de mise à l’échelle terminée, les autres requêtes d’infrastructure opèrent ensemble. Et lorsque l’infrastructure est ajoutée, les plans App Service sont affectés en fonction des besoins. La création d’un plan App Service est elle-même une opération de mise à l’échelle car elle demande du matériel supplémentaire.

Dans l’App Service mutualisé, la mise à l’échelle est immédiate, car il existe un pool de ressources instantanément disponible pour la prendre en charge. Dans un environnement ASE, il n’existe pas de mémoire tampon et les ressources sont allouées en fonction des besoins.

Dans un ASE, vous pouvez mettre à l’échelle un plan App Service jusqu’à 100 instances. Un ASE peut avoir jusqu’à 201 instances dans l’ensemble des plans App Service qu’il comprend.

## <a name="ip-addresses"></a>Adresses IP

App Service peut allouer une adresse IP dédiée à une application. Cette fonctionnalité est disponible après la configuration d'un certificat SSL basé sur IP, comme indiqué ici : [Lier un certificat TLS/SSL personnalisé existant à Azure App Service][ConfigureSSL]. Dans un ASE d’équilibrage de charge interne (ILB), vous ne pouvez pas ajouter plus d’adresses IP à utiliser pour un protocole SSL basé sur IP.

Avec un ASE externe, vous pouvez configurer le protocole SSL basé sur IP pour votre application de la même manière que dans l’App Service mutualisé. Il existe toujours une adresse de secours dans l’ASE, dans la limite de 30 adresses IP. Chaque fois que vous en utilisez une, une autre est ajoutée afin qu’une adresse soit toujours disponible. Un délai est nécessaire pour allouer une autre adresse IP. Ce délai empêche l’ajout rapide d’adresses IP.

## <a name="front-end-scaling"></a>Mise à l’échelle du serveur frontal

Lorsque vous montez en charge vos plans App Service, les workers sont automatiquement ajoutés pour les prendre en charge. Chaque environnement ASE est créé avec deux serveurs frontaux. Les serveurs frontaux montent en charge automatiquement à raison d’un serveur frontal pour chaque ensemble de 15 instances du plan App Service. Par exemple, si vous avez trois plans App Service comprenant cinq instances chacun, vous avez un total de 15 instances et de trois serveurs frontaux. Si vous effectuez une mise à l’échelle pour atteindre un total de 30 instances, vous avez quatre serveurs frontaux. Ce modèle se poursuit au fur et à mesure que vous effectuez un scale-out.

Le nombre de serveurs frontaux alloués par défaut est correct pour une charge modérée. Vous pouvez réduire le ratio à la valeur d’un serveur frontal toutes les cinq instances. Vous pouvez également modifier la taille des serveurs frontaux. Par défaut, ils n’ont qu’un seul cœur. Dans le Portail Azure, vous pouvez changer leur taille à deux ou quatre cœurs à la place.

La modification du ratio ou des tailles de serveurs frontaux occasionne des frais. Pour en savoir plus, consultez [Tarification d'Azure App Service][Pricing]. Si vous voulez améliorer la capacité de chargement de votre ASE, vous obtiendrez davantage d’amélioration en commençant par opérer une mise à l’échelle vers des serveurs frontaux à deux cœurs avant d’ajuster le ratio de mise à l’échelle. La modification de la taille en cœurs de vos serveurs frontaux entraîne une mise à niveau de votre ASE et doit être effectuée en dehors des heures de bureau normales.

Les ressources du serveur frontal sont le point de terminaison HTTP/HTTPS pour l’environnement ASE. Avec la configuration de serveur frontal par défaut, l’utilisation de mémoire par serveur frontal se situe en permanence autour de 60 %. La raison principale pour mettre à l’échelle vos serveurs frontaux est l’utilisation de l’UC qui est principalement déterminée par le trafic HTTP.

## <a name="app-access"></a>Accès de l’application

Dans un ASE externe, le suffixe de domaine utilisé pour la création d’application est *.&lt;asename&gt;.p.azurewebsites.net*. Si votre ASE s’appelle _external-ase_ et que vous y hébergez une application appelée _contoso_, vous l’atteignez aux URL suivantes :

- contoso.external-ase.p.azurewebsites.net
- contoso.scm.external-ase.p.azurewebsites.net

Pour des informations sur la création d’un ASE externe, voir [Créer un App Service Environment][MakeExternalASE].

Dans un ASE ILB, le suffixe de domaine utilisé pour la création d’application est *.&lt;asename&gt;.appserviceenvironment.net*. Si votre ASE s’appelle _ilb-ase_ et que vous y hébergez une application appelée _contoso_, vous l’atteignez aux URL suivantes :

- contoso.ilb-ase.appserviceenvironment.net
- contoso.scm.ilb-ase.appserviceenvironment.net

Pour plus d’informations sur la façon de créer un environnement App Service ILB, consultez [Créer et utiliser un équilibreur de charge interne avec un environnement Azure App Service Environment][MakeILBASE].

L’URL du Gestionnaire de contrôle des services est utilisée pour accéder à la console Kudu ou pour publier votre application à l’aide de Web Deploy. Pour plus d'informations sur la console Kudu, consultez [Console Kudu pour Azure App Service][Kudu]. La console Kudu offre une interface utilisateur web pour le débogage, le chargement et la modification des fichiers, etc.

### <a name="dns-configuration"></a>Configuration DNS 

Lorsque vous utilisez un environnement ASE externe, les applications effectuées dans votre ASE sont inscrites auprès d’Azure DNS. Il n’est pas nécessaire de passer par d’autres étapes dans un ASE externe pour que vos applications soient disponibles publiquement. Avec un environnement ASE ILB, vous devez gérer votre propre service DNS. Vous pouvez effectuer cette opération sur votre propre serveur DNS ou avec des zones privées Azure DNS.

Pour configurer DNS sur votre propre serveur DNS avec votre ASE ILB :

1. créez une zone pour &lt;nom ASE&gt;.appserviceenvironment.net
1. créez un enregistrement A dans cette zone qui pointe * vers l’adresse IP ILB
1. créez un enregistrement A dans cette zone qui pointe @ vers l’adresse IP ILB
1. créez une zone dans le scm nommé &lt;nom ASE&gt;.appserviceenvironment.net
1. créez un enregistrement A dans la zone scm qui pointe * vers l’adresse IP ILB

Pour configurer DNS dans les zones privées Azure DNS :

1. créez une zone privée Azure DNS nommée &lt;nom ASE&gt;.appserviceenvironment.net
1. créez un enregistrement A dans cette zone qui pointe * vers l’adresse IP ILB
1. créez un enregistrement A dans cette zone qui pointe @ vers l’adresse IP ILB
1. créez un enregistrement A dans cette zone qui pointe *.scm vers l’adresse IP ILB

Les paramètres DNS du suffixe de domaine par défaut de votre ASE ne limitent pas vos applications à être accessibles uniquement par ces noms. Vous pouvez définir un nom de domaine personnalisé sans validation sur vos applications dans un environnement ASE ILB. Si vous souhaitez ensuite créer une zone nommée *contoso.net*, vous pouvez le faire et la pointer vers l’adresse IP ILB. Le nom de domaine personnalisé fonctionne pour les demandes d’application, mais pas pour le site GCL. Le site GCL est disponible uniquement pour *&lt;appname&gt;.scm.&lt;asename&gt;.appserviceenvironment.net*. 

La zone nommée *.&lt;asename&gt;.appserviceenvironment.net* est globalement unique. Avant mai 2019, les clients pouvaient spécifier le suffixe de domaine de l’ASE ILB. Si vous souhaitez utiliser *.contoso.com* comme suffixe de domaine, vous pouvez le faire et inclure le site GCL. Ce modèle présentait quelques contraintes au niveau de la gestion du certificat SSL par défaut, de l’absence d’authentification unique auprès du site GCL, et de la nécessité d’utiliser un certificat générique. Le processus de mise à niveau du certificat par défaut de l’ASE ILB entraînait également une interruption du service et le redémarrage de l’application. Pour résoudre ces problèmes, le comportement de l’ASE ILB a été modifié pour utiliser un suffixe de domaine basé sur le nom de l’ASE, avec un suffixe appartenant à Microsoft. La modification apportée au comportement de l’ASE ILB affecte uniquement les environnements ASE ILB créés après mai 2019. Les environnements ASE ILB préexistants doivent toujours gérer le certificat par défaut de l’ASE et leur configuration DNS.

## <a name="publishing"></a>Publication

Dans un environnement ASE, comme avec l’App Service mutualisée, vous pouvez publier à l’aide des méthodes suivantes :

- Déploiement web
- FTP
- Intégration continue (CI)
- Opération de glisser-déplacer dans la console Kudu
- Un IDE tel que Visual Studio, Eclipse ou IntelliJ IDEA

Avec un environnement ASE externe, toutes ces options de publication fonctionnent toutes de la même façon. Pour plus d'informations, consultez [Déploiement dans Azure App Service][AppDeploy].

Avec un environnement ASE d’équilibreur de charge interne, les points de terminaison de publication sont uniquement disponibles via l’équilibreur de charge interne. L’équilibrage de charge interne se trouve sur une adresse IP privée dans le sous-réseau de l’environnement ASE du réseau virtuel. Si vous n’avez pas d’accès réseau à l’équilibreur de charge interne, vous ne pouvez pas publier d’applications dans cet environnement ASE. Comme indiqué dans [Créer et utiliser un environnement ASE d’équilibrage de charge interne][MakeILBASE], vous devez configurer le DNS pour les applications du système. Cette exigence comprend le point de terminaison SCM. Si les points de terminaison ne sont pas définis correctement, vous ne pouvez pas publier. Vos IDE doivent également avoir un accès réseau à l’équilibreur de charge interne pour publier directement dans celui-ci.

Sans modifications supplémentaires, les systèmes d’intégration continue sur Internet, comme GitHub et Azure DevOps, ne fonctionnent pas avec un environnement ASE d’équilibreur de charge interne, car le point de terminaison de publication n’est pas accessible via Internet. Vous pouvez activer la publication sur un ASE ILB à partir d’Azure DevOps en installant un agent de mise en production auto-hébergé dans le réseau virtuel contenant l’ASE ILB. Vous pouvez aussi utiliser un système d’intégration continue qui s’appuie sur un modèle d’extraction, comme Dropbox.

Les points de terminaison de publication pour les applications d’un environnement ASE d’équilibreur de charge interne utilisent le domaine avec lequel l’environnement ASE d’équilibreur de charge interne a été créé. Vous pouvez le voir dans le profil de publication de l’application et dans le volet du portail de l’application (dans **Vue d’ensemble** > **Bases** et également dans **Propriétés**).

## <a name="storage"></a>Stockage

Un ASE dispose de 1 To de stockage pour toutes les applications qu’il contient. Un plan App Service dans la référence SKU de tarification isolée a une limite de 250 Go. Dans un environnement ASE, 250 Go de stockage sont ajoutés par le plan App Service dans la limite de 1 To. Vous pouvez avoir plus de quatre plans App Service, mais aucun espace de stockage supplémentaire n’est ajouté au-delà de 1 To.

## <a name="logging"></a>Journalisation

Vous pouvez intégrer votre ASE avec Azure Monitor pour envoyer des journaux sur l’ASE à Azure Storage, Azure Event Hubs ou Log Analytics. Ces éléments sont journalisés aujourd’hui :

| Situation | Message |
|---------|----------|
| L’ASE est non sain | L’ASE spécifié n’est pas sain en raison d’une configuration de réseau virtuel non valide. L’ASE est suspendu si l’état non sain perdure. Vérifiez que les instructions définies ici sont respectées : https://docs.microsoft.com/azure/app-service/environment/network-info. |
| Le sous-réseau de l’ASE n’a presque plus d’espace disponible | L’ASE spécifié se trouve dans un sous-réseau qui n’a presque plus d’espace disponible. Il y a {0} adresses restantes. Une fois ces adresses épuisées, l’ASE ne peut pas adapter son échelle.  |
| L’ASE approche la limite d’instance totale | L’ASE spécifié approche la limite d’instance totale de l’ASE. Il contient actuellement {0} instances du plan App Service pour un maximum de 201 instances. |
| L’ASE ne peut pas atteindre une dépendance | L’ASE spécifié ne parvient pas à atteindre {0}.  Vérifiez que les instructions définies ici sont respectées : https://docs.microsoft.com/azure/app-service/environment/network-info. |
| L’ASE est suspendu | L’ASE spécifié est suspendu. La suspension de l’ASE peut être due à un manque de compte ou à une configuration de réseau virtuel non valide. Résolvez la cause racine et reprenez l’ASE pour continuer à traiter le trafic. |
| Une mise à niveau de l’ASE a commencé | Une mise à niveau de plateforme vers l’ASE spécifié a commencé. Attendez-vous à des retards dans les opérations de mise à l’échelle. |
| Une mise à niveau de l’ASE est terminée | Une mise à niveau de plateforme pour l’ASE spécifié s’est terminée. |
| Des opérations de mise à l’échelle ont commencé | Une mise à l’échelle d’un plan App Service ({0}) a commencé. État souhaité : {1} I{2} Workers.
| Les opérations de mise à l’échelle sont terminées | Une mise à l’échelle d’un plan App Service ({0}) est terminée. État actuel : {1} I{2} Workers. |
| Des opérations de mise à l’échelle ont échoué | Une mise à l’échelle d’un plan App Service ({0}) a échoué. État actuel : {1} I{2} Workers. |

Pour activer la journalisation sur votre ASE :

1. Dans le Portail, accédez aux **Paramètres de diagnostic**.
1. Sélectionnez **Ajouter le paramètre de diagnostic**.
1. Fournissez un nom pour l’intégration des journaux.
1. Sélectionnez et configurez les destinations souhaitées pour les journaux.
1. Sélectionnez **AppServiceEnvironmentPlatformLogs**.

![Paramètres du journal de diagnostic d’ASE][4]

Si vous effectuez une intégration avec Log Analytics, vous pouvez voir les journaux en sélectionnant **Journaux** dans le portail ASE et en créant une requête sur **AppServiceEnvironmentPlatformLogs**. Les journaux ne sont émis que lorsque votre ASE comporte un événement qui le déclenche. Si votre ASE ne contient aucun événement de ce type, aucun journal ne sera généré. Pour voir rapidement des exemples de journaux dans votre espace de travail Log Analytics, effectuez une opération de mise à l’échelle avec l’un des plans d’App Service dans votre ASE. Vous pouvez ensuite exécuter une requête sur **AppServiceEnvironmentPlatformLogs** pour afficher ces journaux. 

**Création d’une alerte**

Pour créer une alerte concernant vos journaux, suivez les instructions de la section [Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../../azure-monitor/platform/alerts-log.md). En bref :

* Ouvrez la page Alertes dans votre portail ASE
* Sélectionnez **Nouvelle règle d’alerte**
* Sélectionnez la ressource qui représentera votre espace de travail Log Analytics
* Définissez votre condition avec une recherche de journal personnalisée pour utiliser une requête telle que AppServiceEnvironmentPlatformLogs | où ResultDescription contient « a commencé une mise à l'échelle », ou un modèle de votre choix. Définissez le seuil selon vos besoins. 
* Ajoutez ou créez un groupe d'actions selon vos besoins. Le groupe d’actions est l’emplacement où vous définissez la réponse à l’alerte, par exemple l’envoi d’un e-mail ou d’un message SMS
* Nommez votre alerte et enregistrez-la.

## <a name="upgrade-preference"></a>Préférence de mise à niveau

Si vous avez plusieurs ASE, vous souhaiterez peut-être mettre à niveau certains ASE avant d’autres. Dans l’objet du **Gestionnaire des ressources HostingEnvironment** d’ASE, vous pouvez définir une valeur pour **upgradePreference**. Le paramètre **upgradePreference** peut être configuré au moyen d’un modèle, ARMClient, ou https://resources.azure.com. Les trois valeurs possibles sont les suivantes :

- **Aucun** : Azure ne met à niveau votre ASE dans aucun lot particulier. Cette valeur est la valeur par défaut.
- **Tôt** : Votre ASE sera mis à niveau au cours de la première moitié des mises à niveau d’App Service.
- **Tard** : Votre ASE sera mis à niveau au cours de la deuxième moitié des mises à niveau d’App Service.

Si vous utilisez https://resources.azure.com, procédez comme suit pour définir la valeur d’**upgradePreferences** :

1. accédez à resources.azure.com et connectez-vous avec votre compte Azure.
1. Parcourez les ressources pour accéder aux abonnements\/\[nom d’abonnement\]\/resourceGroups\/\[nom de groupe de ressources\]\/fournisseurs\/Microsoft.Web\/hostingEnvironments\/\[noms de l’ASE\].
1. Sélectionnez **Lecture/écriture** en haut.
1. Sélectionnez **Modifier**.
1. Définissez **upgradePreference** sur l’une des trois valeurs souhaitées.
1. Sélectionnez **Correctif**.

![affichage des ressources azure com][5]

La fonctionnalité **upgradePreferences** est vraiment utile lorsque vous avez plusieurs ASE, car vos ASE paramétrés sur « Tôt » seront mis à niveau avant ceux paramétrés sur « Tard ». Lorsque vous avez plusieurs ASE, vous devez définir vos ASE de développement et de test sur « Tôt » et vos ASE de production sur « Tard ».

## <a name="pricing"></a>Tarifs

La référence SKU de tarification appelée *Isolée* est uniquement destinée à être utilisée avec des ASE. Tous les plans App Service hébergés dans l’ASE font partie de la référence SKU de tarification Isolée. Les tarifs isolés des plans App Service peuvent varier selon la région.

En plus de vos plans App Service, il existe un tarif fixe pour l’environnement ASE en lui-même. Le forfait ne change pas avec la taille de votre ASE. Il couvre l’infrastructure ASE au taux de mise à l’échelle par défaut de 1 serveur frontal supplémentaire pour 15 instances du plan App Service.

Si le taux de mise à l’échelle par défaut de 1 serveur frontal pour 15 instances du plan App Service n’est pas assez rapide, vous pouvez ajuster le ratio auquel des serveurs frontaux sont ajoutés ou la taille des serveurs frontaux. Lorsque vous ajustez le rapport ou la taille, vous payez pour les cœurs frontaux qui ne seraient pas ajoutés par défaut.

Par exemple, si vous ajustez le ratio de mise à l’échelle sur 10, un serveur frontal est ajouté toutes les 10 instances dans vos plans App Service. Le prix fixe couvre un taux de mise à l’échelle d’un serveur frontal toutes les 15 instances. Avec un ratio de mise à l’échelle de 10, vous payez pour le troisième serveur frontal qui est ajouté pour les 10 instances du plan App Service. Vous n’avez pas besoin de payer lorsque vous atteignez 15 instances, car il a été ajouté automatiquement.

Si vous ajustez la taille des frontaux à deux cœurs, mais ne modifiez pas le ratio, vous payez pour les cœurs supplémentaires. Un ASE étant créé avec deux serveurs frontaux, même en dessous du seuil de mise à l’échelle automatique, vous payeriez pour deux cœurs supplémentaires si vous avez porté la taille à des serveurs frontaux de deux cœurs.

Pour en savoir plus, consultez [Tarification d'Azure App Service][Pricing].

## <a name="delete-an-ase"></a>Supprimer un environnement ASE

Pour supprimer un environnement ASE :

1. Sélectionnez **Supprimer** en haut du volet **App Service Environment**.

1. Entrez le nom de votre environnement ASE pour confirmer que vous souhaitez le supprimer. Lorsque vous supprimez un environnement ASE, vous supprimez également l’ensemble de son contenu.

    ![Suppression de ASE][3]

1. Sélectionnez **OK**.

## <a name="ase-cli"></a>Interface CLI avec ASE

Il existe des fonctionnalités de ligne de commande permettant d’administrer un environnement ASE.  Les commandes CLI az sont indiquées ci-dessous.

```azurecli
C:\>az appservice ase --help

Group
    az appservice ase : Manage App Service Environments v2.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    create         : Create app service environment.
    delete         : Delete app service environment.
    list           : List app service environments.
    list-addresses : List VIPs associated with an app service environment.
    list-plans     : List app service plans associated with an app service environment.
    show           : Show details of an app service environment.
    update         : Update app service environment.

For more specific examples, use: az find "az appservice ase"
```



<!--Image references-->
[1]: ./media/using_an_app_service_environment/usingase-appcreate.png
[2]: ./media/using_an_app_service_environment/usingase-pricingtiers.png
[3]: ./media/using_an_app_service_environment/usingase-delete.png
[4]: ./media/using_an_app_service_environment/usingase-logsetup.png
[4]: ./media/using_an_app_service_environment/usingase-logs.png
[5]: ./media/using_an_app_service_environment/usingase-upgradepref.png

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
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[AppDeploy]: ../deploy-local-git.md
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[logalerts]: ../../azure-monitor/platform/alerts-log.md
