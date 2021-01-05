---
title: Utiliser un environnement App Service
description: Découvrez comment utiliser votre App Service Environment pour héberger des applications isolées.
author: ccompy
ms.assetid: 377fce0b-7dea-474a-b64b-7fbe78380554
ms.topic: article
ms.date: 11/16/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 3679bf9d55ddccefddb4bf3b2a96ec1b427315af
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94663219"
---
# <a name="using-an-app-service-environment"></a>Utilisation d’un environnement App Service

L’App Service Environment (ASE) est un déploiement monolocataire d’Azure App Service, qui injecte directement dans un réseau virtuel Azure (VNet) de votre choix. Il s’agit d’un système qui n’est utilisé que par un seul client. Les applications déployées dans l’ASE sont sujettes aux fonctionnalités de mise en réseau appliquées au sous-réseau ASE. Il n’y a pas d’autres fonctionnalités à activer sur vos applications pour qu’elles soient sujettes à ces fonctionnalités de mise en réseau. 

## <a name="create-an-app-in-an-ase"></a>Créer une application dans un ASE

Pour créer une application dans un environnement ASE, vous utilisez le même processus que lorsque vous créez normalement une application, mais avec quelques petites différences. Lorsque vous créez un plan App Service :

- Au lieu de choisir un emplacement géographique dans lequel déployer votre application, vous choisissez un environnement ASE comme emplacement.
- Tous les plans App Service créés dans ASE doivent être dans un niveau tarifaire Isolé v2.

Si vous n’avez pas d’environnement ASE, vous pouvez en créer un en suivant les instructions de l’article [Créer un environnement Azure App Service Environment][MakeASE].

Pour créer une application dans un ASE :

1. Sélectionnez **Créer une ressource** > **Web + Mobile** > **Application web**.

1. Sélectionnez un abonnement.

1. Entrez un nom pour un nouveau groupe de ressources ou sélectionnez **Use existing** (Utiliser existant) et choisissez-en un dans la liste déroulante.

1. Entrez un nom pour l’application. Si vous avez déjà sélectionné un plan App Service dans un environnement ASE, le nom de domaine pour l’application reflète le nom de domaine de l’environnement ASE :

    ![Créer une application dans un ASE][1]

1. Sélectionnez vos Type de publication, Pile et Système d’exploitation.

1.  Sélectionnez une région. Vous devez sélectionner ici un App Service Environment v3 préexistant.  Vous ne pouvez pas créer d’ASEv3 pendant la création d’une application 

1. Sélectionnez un plan App Service ou créez-en un. Si vous créez une application, sélectionnez la taille souhaitée pour votre plan App Service. La seule référence SKU que vous pouvez sélectionner pour votre application est une référence (SKU) de tarification Isolé v2.

    ![Niveaux tarifaires Isolé v2][2]

    > [!NOTE]
    > Des applications Linux et Windows ne peuvent pas se trouver dans un même plan App Service, mais le peuvent dans un même App Service Environment.
    >

1. Sélectionnez ** Suivant : Supervision** Si vous souhaitez activer Application Insights avec votre application, vous pouvez le faire ici pendant le flux de création. 

1.  Sélectionnez **Suivant : Balises** Ajoutez les balises de votre choix à l’application.  

1. Sélectionnez **Vérifier + créer**, vérifiez si les informations sont correctes, puis sélectionnez **Créer**.

## <a name="how-scale-works"></a>Fonctionnement de la mise à l’échelle

Chaque application App Service s’exécute dans un plan App Service. Les environnements App Service contiennent les plans App Service, et les plans App Service contiennent les applications. Lorsque vous mettez à l’échelle une application, vous mettez à l’échelle v le plan App Service, et toutes les applications contenues dans ce même plan.

Lorsque vous mettez à l’échelle un plan App Service, l’infrastructure nécessaire est automatiquement ajoutée. Il existe un délai pour les opérations de mise à l’échelle lorsque l’infrastructure est ajoutée. Lorsque vous mettez à l’échelle un plan App Service, toute autre opération de mise à l’échelle demandée du même système d’exploitation et de la même taille attend que la première opération soit terminée. Une fois l’opération de mise à l’échelle bloquante terminée, toutes les demandes en file d’attente sont traitées en même temps. Une opération de mise à l’échelle sur une taille et un système d’exploitation ne bloque pas les opérations sur d’autres combinaisons de taille et de système d’exploitation. Par exemple, si vous avez mis à l’échelle un plan App Service Windows I2v2, toutes les autres demandes de mise à l’échelle de Windows I2v2 dans cet ASE sont mises en file d’attente jusqu’à ce que l’opération se termine.   

Dans l’App Service mutualisé, la mise à l’échelle est immédiate, car il existe un pool de ressources instantanément disponible pour la prendre en charge. Dans un environnement ASE, il n’existe pas de mémoire tampon et les ressources sont allouées en fonction des besoins.

## <a name="app-access"></a>Accès de l’application

Dans un ASE, le suffixe de domaine utilisé pour la création d’application est *.&lt;asename&gt;.appserviceenvironment.net*. Si votre ASE s’appelle _mon-ase_ et que vous y hébergez une application appelée _contoso_, vous l’atteignez aux URL suivantes :

- contoso.mon-ase.appserviceenvironment.net
- contoso.scm.mon-ase.appserviceenvironment.net

Pour obtenir des informations sur la création d’un ASE, consultez [Créer un App Service Environment][MakeASE].

L’URL du Gestionnaire de contrôle des services est utilisée pour accéder à la console Kudu ou pour publier votre application à l’aide de Web Deploy. Pour plus d'informations sur la console Kudu, consultez [Console Kudu pour Azure App Service][Kudu]. La console Kudu offre une interface utilisateur web pour le débogage, le chargement et la modification des fichiers, etc.

### <a name="dns-configuration"></a>Configuration DNS 

L’ASE utilise des points de terminaison privés pour le trafic entrant, et est configuré automatiquement avec des zones privées Azure DNS. Si vous souhaitez utiliser votre propre serveur DNS, vous devez ajouter les enregistrements suivants :

1. créez une zone pour &lt;nom ASE&gt;.appserviceenvironment.net
1. Créez un enregistrement A dans cette zone qui pointe * vers l’adresse IP entrante qu’utilise le point de terminaison privé de votre ASE.
1. Créez un enregistrement A dans cette zone qui pointe @ vers l’adresse IP entrante qu’utilise le point de terminaison privé de votre ASE.
1. créez une zone dans le scm nommé &lt;nom ASE&gt;.appserviceenvironment.net
1. Créez un enregistrement A dans la zone scm qui pointe * vers l’adresse IP qu’utilise le point de terminaison privé de votre ASE.

Les paramètres DNS du suffixe de domaine par défaut de votre ASE ne limitent pas vos applications à être accessibles uniquement par ces noms. Vous pouvez définir un nom de domaine personnalisé sans validation sur vos applications dans un ASE. Si vous souhaitez ensuite créer une zone nommée *contoso.net*, vous pouvez le faire et la pointer vers l’adresse IP entrante. Le nom de domaine personnalisé fonctionne pour les demandes d’application, mais pas pour le site GCL. Le site GCL est disponible uniquement pour *&lt;appname&gt;.scm.&lt;asename&gt;.appserviceenvironment.net*. 

## <a name="publishing"></a>Publication

Dans un environnement ASE, comme avec l’App Service mutualisée, vous pouvez publier à l’aide des méthodes suivantes :

- Déploiement web
- Intégration continue (CI)
- Opération de glisser-déplacer dans la console Kudu
- Un IDE tel que Visual Studio, Eclipse ou IntelliJ IDEA

Avec un ASE, les points de terminaison de publication ne sont disponibles que par le biais de l’adresse qu’utilise le point de terminaison privé. Si vous n’avez pas d’accès réseau à l’adresse du point de terminaison privé, vous ne pouvez pas publier d’applications dans cet ASE.  Vos IDE doivent également avoir un accès réseau à l’équilibreur de charge interne pour publier directement dans celui-ci.

Sans modifications supplémentaires, les systèmes d’intégration continue sur Internet, comme GitHub et Azure DevOps, ne fonctionnent pas avec un environnement ASE d’équilibreur de charge interne, car le point de terminaison de publication n’est pas accessible via Internet. Vous pouvez activer la publication sur un ASE ILB à partir d’Azure DevOps en installant un agent de mise en production auto-hébergé dans le réseau virtuel contenant l’ASE ILB. 

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

Si vous effectuez une intégration avec Log Analytics, vous pouvez voir les journaux en sélectionnant **Journaux** dans le portail ASE et en créant une requête sur **AppServiceEnvironmentPlatformLogs**. Les journaux ne sont émis que lorsque votre ASE comporte un événement qui le déclenche. Si votre ASE ne contient aucun événement de ce type, aucun journal n’est généré. Pour voir rapidement des exemples de journaux dans votre espace de travail Log Analytics, effectuez une opération de mise à l’échelle avec l’un des plans d’App Service dans votre ASE. Vous pouvez ensuite exécuter une requête sur **AppServiceEnvironmentPlatformLogs** pour afficher ces journaux. 

**Création d’une alerte**

Pour créer une alerte concernant vos journaux, suivez les instructions de la section [Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../../azure-monitor/platform/alerts-log.md). En bref :

* Ouvrez la page Alertes dans votre portail ASE
* Sélectionnez **Nouvelle règle d’alerte**
* Sélectionnez la ressource qui représentera votre espace de travail Log Analytics
* Définissez votre condition avec une recherche de journal personnalisée pour utiliser une requête telle que AppServiceEnvironmentPlatformLogs | où ResultDescription contient « a commencé une mise à l'échelle », ou un modèle de votre choix. Définissez le seuil selon vos besoins. 
* Ajoutez ou créez un groupe d'actions selon vos besoins. Le groupe d’actions est l’emplacement où vous définissez la réponse à l’alerte, par exemple l’envoi d’un e-mail ou d’un message SMS
* Nommez votre alerte et enregistrez-la.

## <a name="internal-encryption"></a>Chiffrement interne

App Service Environment fonctionne comme un système de boîte noire dans laquelle vous ne pouvez pas voir les composants internes ou la communication au sein du système. Pour activer un débit plus élevé, le chiffrement n’est pas activé par défaut entre les composants internes. Le système est sécurisé, car le trafic est totalement inaccessible à des fins de supervision. Si toutefois vous avez une exigence de conformité qui requiert un chiffrement complet du chemin de données de bout en bout, vous pouvez l’activer dans l’interface utilisateur **Configuration** de l’ASE.

![Activer le chiffrement interne][5]

Ceci va chiffrer le trafic réseau interne de votre ASE entre les front-ends et les Workers, chiffrer le fichier d’échange et chiffrer les disques des Workers. Une fois le clusterSetting InternalEncryption activé, il peut y avoir un impact sur les performances de votre système. Quand vous apportez la modification pour activer InternalEncryption, votre ASE sera dans un état instable jusqu’à ce que la modification soit entièrement propagée. La propagation complète de la modification peut prendre quelques heures, en fonction du nombre d’instances présentes dans votre ASE. Nous vous recommandons vivement de ne pas procéder à l’activation sur un ASE pendant qu’il est en cours d’utilisation. Si vous avez besoin d’activer le chiffrement sur un ASE activement utilisé, nous vous recommandons vivement de diriger le trafic vers un environnement de sauvegarde jusqu’à ce que l’opération soit terminée.

## <a name="upgrade-preference"></a>Préférence de mise à niveau

Si vous avez plusieurs ASE, vous souhaiterez peut-être mettre à niveau certains ASE avant d’autres. Dans l’objet du **Gestionnaire des ressources HostingEnvironment** d’ASE, vous pouvez définir une valeur pour **upgradePreference**. Le paramètre **upgradePreference** peut être configuré au moyen d’un modèle, ARMClient, ou https://resources.azure.com. Les trois valeurs possibles sont les suivantes :

- **Aucun** : Azure ne met à niveau votre ASE dans aucun lot particulier. Cette valeur est la valeur par défaut.
- **Tôt** : Votre ASE sera mis à niveau au cours de la première moitié des mises à niveau d’App Service.
- **Tard** : Votre ASE sera mis à niveau au cours de la deuxième moitié des mises à niveau d’App Service.

Pour configurer vos préférences de mise à niveau, accédez à l’interface utilisateur **Configuration** de l’ASE. 

La fonctionnalité **upgradePreferences** est vraiment utile lorsque vous avez plusieurs ASE, car vos ASE paramétrés sur « Tôt » seront mis à niveau avant ceux paramétrés sur « Tard ». Lorsque vous avez plusieurs ASE, vous devez définir vos ASE de développement et de test sur « Tôt » et vos ASE de production sur « Tard ».

## <a name="delete-an-ase"></a>Supprimer un environnement ASE

Pour supprimer un environnement ASE :

1. Sélectionnez **Supprimer** en haut du volet **App Service Environment**.

1. Entrez le nom de votre environnement ASE pour confirmer que vous souhaitez le supprimer. Lorsque vous supprimez un environnement ASE, vous supprimez également l’ensemble de son contenu.

    ![Suppression de ASE][3]

1. Sélectionnez **OK**.

<!--Image references-->
[1]: ./media/using/using-appcreate.png
[2]: ./media/using/using-appcreate-skus.png
[3]: ./media/using/using-delete.png
[4]: ./media/using/using-logsetup.png
[4]: ./media/using/using-logs.png
[5]: ./media/using/using-configuration.png

<!--Links-->
[Intro]: ./overview.md
[MakeASE]: ./creation.md
[ASENetwork]: ./networking.md
[UsingASE]: ./using.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/network-security-groups-overview.md
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[AppDeploy]: ../deploy-local-git.md
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[logalerts]: ../../azure-monitor/platform/alerts-log.md
