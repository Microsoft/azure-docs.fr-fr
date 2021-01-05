---
title: Questions fréquentes (FAQ) sur la configuration
description: Obtenez des réponses aux questions fréquentes sur les problèmes de configuration et de gestion d’Azure App Service.
author: genlin
manager: dcscontentpm
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.topic: article
ms.date: 10/30/2018
ms.author: genli
ms.openlocfilehash: fb193637525722bf227241a614cd977fbf70c9ac
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93074180"
---
# <a name="configuration-and-management-faqs-for-web-apps-in-azure"></a>FAQ sur la configuration et la gestion de Web Apps dans Azure

Cet article contient des réponses aux questions fréquemment posées sur les problèmes de configuration et de gestion de la [fonctionnalité Web Apps d’Azure App Service](https://azure.microsoft.com/services/app-service/web/).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="are-there-limitations-i-should-be-aware-of-if-i-want-to-move-app-service-resources"></a>Existe-t-il des limitations dont je dois être informé si je souhaite déplacer des ressources App Service ?

Si vous prévoyez de déplacer des ressources App Service vers un nouveau groupe de ressources ou abonnement, il existe effectivement des limitations que vous devez connaître. Pour en savoir plus, voir [Limitations d’App Service](../azure-resource-manager/management/move-limitations/app-service-move-limitations.md).

## <a name="how-do-i-use-a-custom-domain-name-for-my-web-app"></a>Comment utiliser un nom de domaine personnalisé pour mon application web ?

Pour les réponses aux questions courantes sur l’utilisation d’un nom de domaine personnalisé avec votre application web Azure, regardez notre vidéo de sept minutes [Ajouter un nom de domaine personnalisé](https://channel9.msdn.com/blogs/Azure-App-Service-Self-Help/Add-a-Custom-Domain-Name). La vidéo présente une procédure pas à pas montrant comment ajouter un nom de domaine personnalisé. Elle décrit comment utiliser votre propre URL à la place de l’URL *.azurewebsites.net avec votre application web App Service. Vous pouvez également voir une description détaillée de la manière de [mapper un nom de domaine personnalisé](app-service-web-tutorial-custom-domain.md).


## <a name="how-do-i-purchase-a-new-custom-domain-for-my-web-app"></a>Comment acheter un domaine personnalisé pour mon application web ?

Pour savoir comment acheter et configurer un domaine personnalisé pour votre application web App Service, voir [Acheter et configurer un nom de domaine personnalisé dans Azure App Service](manage-custom-dns-buy-domain.md).


## <a name="how-do-i-upload-and-configure-an-existing-tlsssl-certificate-for-my-web-app"></a>Comment charger et configurer un certificat TLS/SSL existant pour mon application web ?

Pour découvrir comment charger et configurer un certificat TLS/SSL personnalisé existant, consultez [Ajouter un certificat TLS/SSL à votre application App Service](configure-ssl-certificate.md).


## <a name="how-do-i-purchase-and-configure-a-new-tlsssl-certificate-in-azure-for-my-web-app"></a>Comment acheter et configurer un nouveau certificat TLS/SSL dans Azure pour mon application web ?

Pour découvrir comment acheter et configurer un certificat TLS/SSL pour votre application web App Service, voir [Acheter et configurer un certificat TLS/SSL pour votre service Azure App Service](configure-ssl-certificate.md).


## <a name="how-do-i-move-application-insights-resources"></a>Comment déplacer des ressources Application Insights ?

Actuellement, Azure Application Insights ne prend en charge l’opération de déplacement. Si votre groupe de ressources d’origine inclut une ressource Application Insights, vous ne pouvez pas la déplacer. Si vous incluez la ressource Application Insights lorsque vous tentez de déplacer une application App Service, l’opération de déplacement toute entière échoue. Toutefois, Application Insights et le plan App Service ne doivent pas nécessairement résider dans le même groupe de ressources que l’application pour que celle-ci fonctionne correctement.

Pour en savoir plus, voir [Limitations d’App Service](../azure-resource-manager/management/move-limitations/app-service-move-limitations.md).

## <a name="where-can-i-find-a-guidance-checklist-and-learn-more-about-resource-move-operations"></a>Où trouver une liste de conseils et en savoir plus sur les opérations de déplacement de ressources ?

La rubrique [Limitations d’App Service](../azure-resource-manager/management/move-limitations/app-service-move-limitations.md) explique comment déplacer des ressources vers un nouvel abonnement ou un nouveau groupe de ressources dans le même abonnement. Elle contient des informations sur la liste de contrôle de déplacement de ressources, les services qui prennent en charge l’opération de déplacement, les limitations d’App Service et d’autres rubriques.

## <a name="how-do-i-set-the-server-time-zone-for-my-web-app"></a>Comment définir le fuseau horaire du serveur pour mon application web ?

Pour définir le fuseau horaire du serveur pour votre application web :

1. Dans le portail Azure, dans votre abonnement App Service, accédez au menu **Paramètres de l’application**.
2. Sous **Paramètres de l’application** , ajoutez le paramètre suivant :
    * Key = WEBSITE_TIME_ZONE
    * Value = *Fuseau horaire de votre choix*
3. Sélectionnez **Enregistrer**.

Pour les services d’application qui s’exécutent sur Windows, consultez la sortie de la commande Windows `tzutil /L`. Utilisez la valeur de la deuxième ligne de chaque entrée. Exemple : « Tonga Standard Time ». Certaines de ces valeurs figurent également dans la colonne **TimeZone** de [Fuseaux horaires par défaut](/windows-hardware/manufacture/desktop/default-time-zones).

Pour les services d’application qui s’exécutent sur Linux, définissez une valeur à partir de la [base de données IANA TZ](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). Exemple : « America/Adak ».

## <a name="why-do-my-continuous-webjobs-sometimes-fail"></a>Pourquoi mes tâches web continues échouent-elles parfois ?

Par défaut, les applications web sont déchargées si elles restent inactives pendant un laps de temps défini. Cela permet au système d’économiser des ressources. Dans les offres De base et Standard, vous pouvez activer le paramètre **Toujours actif** afin que l’application web reste chargée en permanence. Si votre application web exécute des tâches web en continu, vous devez activer l’option **Toujours actif** , sans quoi les tâches pourraient ne pas s’exécuter de manière fiable. Pour plus d’informations, voir [Création d’une tâche web exécutée en continu](webjobs-create.md#CreateContinuous).

## <a name="how-do-i-get-the-outbound-ip-address-for-my-web-app"></a>Comment obtenir l’adresse IP sortante pour mon application web ?

Pour obtenir la liste des adresses IP sortantes pour votre application web :

1. Dans le portail Azure, dans le panneau d’application web, accédez au menu **Propriétés**.
2. Recherchez les **adresses IP sortantes**.

La liste des adresses IP sortantes s’affiche.

Pour savoir comment obtenir l’adresse IP sortante si votre site web est hébergé dans un environnement App Service, consultez [Adresses réseau sortantes](environment/app-service-app-service-environment-network-architecture-overview.md#outbound-network-addresses).

## <a name="how-do-i-get-a-reserved-or-dedicated-inbound-ip-address-for-my-web-app"></a>Comment obtenir une adresse IP entrante réservée ou dédiée pour mon application web ?

Pour créer une adresse IP dédiée ou réservée aux appels entrants au site web de votre application Azure, installez et configurez un certificat TLS/SSL basé sur IP.

Notez que, pour utiliser une adresse IP dédiée ou réservée aux appels entrants, votre plan App Service doit être un plan de service De base ou supérieur.

## <a name="can-i-export-my-app-service-certificate-to-use-outside-azure-such-as-for-a-website-hosted-elsewhere"></a>Puis-je exporter mon certificat App Service afin de l’utiliser en dehors d’Azure, par exemple, pour un site web hébergé ailleurs ? 

Oui, vous pouvez l’exporter pour l’utiliser en dehors d’Azure. Pour plus d’informations, voir [FAQ sur les certificats App Service et les domaines personnalisés](https://social.msdn.microsoft.com/Forums/azure/f3e6faeb-5ed4-435a-adaa-987d5db43b80/faq-on-app-service-certificates-and-custom-domains?forum=windowsazurewebsitespreview).

## <a name="can-i-export-my-app-service-certificate-to-use-with-other-azure-cloud-services"></a>Puis-je exporter mon certificat App Service afin de l’utiliser avec d’autres services cloud Azure ?

Le portail fournit une expérience de première classe pour le déploiement d’un certificat App Service vers des applications App Service via Azure Key Vault. Toutefois, nous avons reçu des demandes de clients désireux d’utiliser ces certificats en dehors de la plateforme App Service, par exemple, avec des machines virtuelles Azure. Pour savoir comment créer une copie PFX locale de votre certificat App Service afin de pouvoir utiliser le certificat avec d’autres ressources Azure, consultez [Créer une copie PFX locale d’un certificat App Service](https://blogs.msdn.microsoft.com/appserviceteam/2017/02/24/creating-a-local-pfx-copy-of-app-service-certificate/).

Pour plus d’informations, voir [FAQ sur les certificats App Service et les domaines personnalisés](https://social.msdn.microsoft.com/Forums/azure/f3e6faeb-5ed4-435a-adaa-987d5db43b80/faq-on-app-service-certificates-and-custom-domains?forum=windowsazurewebsitespreview).


## <a name="why-do-i-see-the-message-partially-succeeded-when-i-try-to-back-up-my-web-app"></a>Pourquoi le message « Partiellement réussie » s’affiche-t-il quand j’essaie de sauvegarder mon application web ?

Une cause courante de l’échec de la sauvegarde est que l’application utilise certains fichiers. Les fichiers en cours d’utilisation sont verrouillés durant la sauvegarde. Cela empêche la sauvegarde de ces fichiers et peut aboutir à l’état « Partiellement réussie ». Vous pouvez empêcher cela en excluant des fichiers du processus de sauvegarde. Vous pouvez choisir de sauvegarder uniquement ce qui est nécessaire. Pour plus d’informations, voir [Sauvegarder uniquement les parties importantes de votre site avec Azure Web Apps](https://zainrizvi.io/blog/creating-partial-backups-of-your-site-with-azure-web-apps/).

## <a name="how-do-i-remove-a-header-from-the-http-response"></a>Comment supprimer un en-tête de la réponse HTTP ?

Pour supprimer les en-têtes de la réponse HTTP, mettez à jour le fichier web.config de votre site. Pour plus d’informations, voir [Supprimer les en-têtes de serveur standard sur vos sites web Azure](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/).

## <a name="is-app-service-compliant-with-pci-standard-30-and-31"></a>App Service est-il conforme aux normes PCI 3.0 et 3.1 ?

Actuellement, la fonctionnalité Web Apps d’Azure App Service est conforme à la norme PCI DSS (PCI Data Security Standard) version 3.0 niveau 1. La certification PCI DSS version 3.1 figure sur notre feuille de route. Une planification est en cours concernant la manière dont la toute dernière norme sera adoptée.

La certification PCI DSS version 3.1 nécessite la désactivation du protocole TLS version 1.0. Actuellement, la désactivation de TLS 1.0 n’est pas une option pour la plupart des plans App Service. En revanche, si vous utilisez App Service Environment ou souhaitez migrer votre charge de travail vers cet environnement, vous pouvez mieux contrôler votre environnement. Cela implique de désactiver le protocole TLS 1.0 avec l’aide du support technique Azure. Dans un futur proche, nous prévoyons de rendre ces paramètres accessibles aux utilisateurs.

Pour plus d’informations, voir [Conformité des applications web Microsoft Azure App Service avec les normes PCI 3.0 et 3.1](https://support.microsoft.com/help/3124528).

## <a name="how-do-i-use-the-staging-environment-and-deployment-slots"></a>Comment utiliser l’environnement intermédiaire et les emplacements de déploiement ?

Dans les plans App Service Standard et Premium, lorsque vous déployez votre application web vers App Service, vous pouvez cibler un emplacement de déploiement distinct de l’emplacement de production par défaut. Les emplacements de déploiement sont des applications web dynamiques qui ont leurs propres noms d’hôte. Les éléments de contenu et de configuration des applications web peuvent être échangés entre deux emplacements de déploiement, y compris l’emplacement de production.

Pour plus d’informations sur l’utilisation des emplacements de déploiement, voir [Configurer des environnements intermédiaires dans Azure App Service](deploy-staging-slots.md).

## <a name="how-do-i-access-and-review-webjob-logs"></a>Comment accéder aux journaux d’activité de tâche web (WebJob) et les consulter ?

Pour consulter les journaux d’activité de tâche web (WebJob) :

1. Connectez-vous à votre **site web Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net`).
2. Sélectionnez la tâche web.
3. Sélectionnez le bouton **Activer/désactiver la sortie**.
4. Pour télécharger le fichier de sortie, sélectionnez le lien **Télécharger**.
5. Pour des exécutions individuelles, sélectionnez **Appel individuel**.
6. Sélectionnez le bouton **Activer/désactiver la sortie**.
7. Sélectionnez le lien de téléchargement.

## <a name="im-trying-to-use-hybrid-connections-with-sql-server-why-do-i-see-the-message-systemoverflowexception-arithmetic-operation-resulted-in-an-overflow"></a>J’essaie d’utiliser des connexions hybrides avec SQL Server. Pourquoi le message « System.OverflowException : L’opération arithmétique a provoqué un dépassement de capacité » s’affiche-t-il ?

Si vous utilisez des connexions hybrides pour accéder à SQL Server, une mise à jour de Microsoft .NET du 10 mai 2016 pourrait entraîner un échec de connexion. Le message suivant pourrait s’afficher :

```
Exception: System.Data.Entity.Core.EntityException: The underlying provider failed on Open. —> System.OverflowException: Arithmetic operation resulted in an overflow. or (64 bit Web app) System.OverflowException: Array dimensions exceeded supported range, at System.Data.SqlClient.TdsParser.ConsumePreLoginHandshake
```

### <a name="resolution"></a>Résolution

L’exception a été provoquée par un problème avec Hybrid Connection Manager qui a depuis été résolu. Veillez à [mettre à jour votre Hybrid Connection Manager](https://go.microsoft.com/fwlink/?LinkID=841308) pour résoudre ce problème.

## <a name="how-do-i-add-a-url-rewrite-rule"></a>Comment ajouter une règle de réécriture d’URL ?

Pour ajouter une règle de réécriture d’URL, créez un fichier web.config avec les entrées de configuration pertinentes dans le dossier **wwwroot**. Pour plus d’informations, consultez [Azure App Services : Présentation de la réécriture d’URL](/archive/blogs/madhurabharadwaj/azure-app-services-understanding-url-re-write).

## <a name="how-do-i-control-inbound-traffic-to-app-service"></a>Comment contrôler le trafic entrant dans App Service ?

Au niveau du site, vous disposez de deux options pour contrôler le trafic entrant dans App Service :

* Activer les restrictions dynamiques d’adresse IP. Pour découvrir comment activer les restrictions dynamiques d’adresse IP, voir [Restrictions d’adresse IP et de domaine pour les sites web Azure](https://azure.microsoft.com/blog/ip-and-domain-restrictions-for-windows-azure-web-sites/).
* Activer la sécurité du module. Pour savoir comment activer la sécurité du module, voir [Pare-feu d’applications web ModSecurity sur les sites web Azure](https://azure.microsoft.com/blog/modsecurity-for-azure-websites/).

Si vous utilisez App Service Environment, vous pouvez utiliser le [pare-feu Barracuda](https://azure.microsoft.com/blog/configuring-barracuda-web-application-firewall-for-azure-app-service-environment/).

## <a name="how-do-i-block-ports-in-an-app-service-web-app"></a>Comment bloquer des ports dans une application web App Service ?

Dans l’environnement de client partagé App Service, il n’est pas possible de bloquer des ports spécifiques en raison de la nature de l’infrastructure. Les ports TCP 4020, 4022 et 4024 peuvent aussi être ouverts pour un débogage à distance Visual Studio.

Dans App Service Environment, vous contrôlez totalement les trafics entrant et sortant. Vous pouvez utiliser des Groupes de sécurité réseau pour limiter ou bloquer des ports spécifiques. Pour plus d’informations sur App Service Environment, voir [Présentation d’App Service Environment](https://azure.microsoft.com/blog/introducing-app-service-environment/).

## <a name="how-do-i-capture-an-f12-trace"></a>Comment capturer une trace F12

Pour capturer une trace F12, vous disposez de deux options :

* Sortie console F12
* Sortie de la console F12

### <a name="f12-http-trace"></a>Sortie console F12

1. Dans Internet Explorer, accédez à votre site web. Il est important que vous vous connectiez avant de suivre les étapes suivantes. Dans le cas contraire, la trace F12 capture des données de connexion sensibles.
2. Appuyez sur F12.
3. Vérifiez que l’onglet **Réseau** est activé, puis sélectionnez le bouton vert **Lecture**.
4. Effectuez les étapes reproduisant le problème.
5. Sélectionnez le bouton rouge **Arrêt**.
6. Sélectionnez le bouton **Enregistrer** (icône de disque), puis, dans Internet Explorer ou Microsoft Edge, enregistrez le fichier HAR *ou* , dans Chrome, cliquez avec le bouton droit sur celui-ci, puis sélectionnez **Save as HAR with content** (Sauvegarder le contenu au format HAR).

### <a name="f12-console-output"></a>Sortie de la console F12

1. Sélectionnez l’onglet **Console**.
2. Pour chaque onglet contenant au moins un élément, sélectionnez l’onglet ( **Erreur** , **Avertissement** ou **Informations** ). Si l’onglet n’est pas sélectionné, l’icône d’onglet s’affiche en gris ou noir lorsque vous écartez le curseur de celle-ci.
3. Cliquez avec le bouton droit dans la zone de message du volet, puis sélectionnez **Copier tout**.
4. Collez le texte copié dans un fichier, puis enregistrez celui-ci.

Pour afficher un fichier HAR, vous pouvez utiliser [HAR viewer](http://www.softwareishard.com/har/viewer/) (visionneuse HAR).

## <a name="why-do-i-get-an-error-when-i-try-to-connect-an-app-service-web-app-to-a-virtual-network-that-is-connected-to-expressroute"></a>Pourquoi une erreur se produit-elle quand j’essaie de connecter une application web App Service à un réseau virtuel connecté à ExpressRoute ?

Si vous tentez de connecter une application web Azure à un réseau virtuel connecté à Azure ExpressRoute, l’opération échoue. Le message suivant s’affiche : « La passerelle n’est pas une passerelle VPN. »

Actuellement, vous ne pouvez pas avoir de connexion VPN point à site à un réseau virtuel connecté à ExpressRoute. Un réseau VPN de point à site et ExpressRoute ne peuvent pas coexister pour le même réseau virtuel. Pour plus d’informations, voir [Limites et limitations d’ExpressRoute et des connexions VPN site à site](../expressroute/expressroute-howto-coexist-classic.md#limits-and-limitations).

## <a name="how-do-i-connect-an-app-service-web-app-to-a-virtual-network-that-has-a-static-routing-policy-based-gateway"></a>Comment connecter une application web App Service à un réseau virtuel disposant d’une passerelle de routage statique (basée sur une stratégie) ?

Actuellement, la connexion d’une application web App Service à un réseau virtuel disposant d’une passerelle de routage statique (basée sur une stratégie) n’est pas prise en charge. Si votre réseau virtuel cible existe déjà, pour que vous puissiez le connecter à une application, il doit prendre en charge le réseau VPN de point à site avec une passerelle de routage dynamique. Si votre passerelle est configurée pour un routage statique, vous ne pouvez pas activer un réseau VPN de point à site. 

Pour plus d’informations, voir [Intégrer une application à un réseau Azure Virtual Network](web-sites-integrate-with-vnet.md).

## <a name="in-my-app-service-environment-why-can-i-create-only-one-app-service-plan-even-though-i-have-two-workers-available"></a>Dans mon App Service Environment, pourquoi ne puis-je créer qu’un seul plan App Service, même si j’ai deux workers disponibles ?

Pour offrir une tolérance de panne, App Service Environment requiert que chaque pool de workers dispose d’au moins une ressource de calcul supplémentaire. Aucune charge de travail ne peut être assignée à la ressource de calcul supplémentaire.

Pour plus d’informations, voir [Comment créer un environnement App Service](environment/app-service-web-how-to-create-an-app-service-environment.md).

## <a name="why-do-i-see-timeouts-when-i-try-to-create-an-app-service-environment"></a>Pourquoi y a-t-il des délais d’attente quand j’essaie de créer un App Service Environment ?

Parfois, la création d’un App Service Environment échoue. Dans ce cas, l’erreur suivante apparaît dans les journaux d’activité :
```
ResourceID: /subscriptions/{SubscriptionID}/resourceGroups/Default-Networking/providers/Microsoft.Web/hostingEnvironments/{ASEname}
Error:{"error":{"code":"ResourceDeploymentFailure","message":"The resource provision operation did not complete within the allowed timeout period."}}
```

Pour résoudre ce problème, assurez-vous qu’aucune des conditions n’est vraie :
* Le sous-réseau est trop petit.
* Le sous-réseau n’est pas vide.
* ExpressRoute n’est pas compatible avec les exigences de connectivité de réseau d’un App Service Environment.
* Un groupe de sécurité réseau incorrect n’est pas compatible avec les besoins en connectivité réseau d’un App Service Environment.
* Le tunneling forcé est activé.

Pour plus d’informations, voir [Problèmes les plus fréquents lors du déploiement (création) d’un nouvel App Service Environment Azure](/archive/blogs/waws/most-frequent-issues-when-deploying-creating-a-new-azure-app-service-environment-ase).

## <a name="why-cant-i-delete-my-app-service-plan"></a>Pourquoi ne puis-je pas supprimer mon plan App Service ?

Vous ne pouvez pas supprimer un plan App Service si des applications App Service sont associées au ce plan. Avant de supprimer un plan App Service, supprimez toutes les applications App Service du plan.

## <a name="how-do-i-schedule-a-webjob"></a>Comment planifier une tâche web ?

Vous pouvez créer une tâche web planifiée à l’aide d’expressions CRON :

1. Créez un fichier settings.job.
2. Dans ce fichier JSON, incluez une propriété de planification à l’aide d’une expression CRON : 
    ```json
    { "schedule": "{second}
    {minute} {hour} {day}
    {month} {day of the week}" }
    ```

Pour plus d’informations sur les tâches web planifiées, voir [Créer une tâche web planifiée à l’aide d’une expression CRON](webjobs-create.md#CreateScheduledCRON).

## <a name="how-do-i-perform-penetration-testing-for-my-app-service-app"></a>Comment effectuer un test de pénétration pour mon application App Service ?

Pour effectuer un test de pénétration, [soumettez une demande](https://portal.msrc.microsoft.com/engage/pentest).

## <a name="how-do-i-configure-a-custom-domain-name-for-an-app-service-web-app-that-uses-traffic-manager"></a>Comment configurer un nom de domaine personnalisé pour une application web App Service utilisant Traffic Manager?

Pour savoir comment utiliser un nom de domaine personnalisé avec une application App Service utilisant Azure Traffic Manager pour l’équilibrage de charge, voir [Configuration d’un nom de domaine personnalisé pour une application web dans Azure App Service utilisant Traffic Manager](configure-domain-traffic-manager.md).

## <a name="my-app-service-certificate-is-flagged-for-fraud-how-do-i-resolve-this"></a>Mon certificat App Service est marqué pour une fraude. Comment résoudre ce problème ?

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Lors de la vérification du domaine d’un achat de certificat App Service, vous pouvez voir le message suivant :

« Votre certificat a été signalé comme étant une fraude potentielle. La demande est en cours de révision. Si le certificat devient inutilisable sous 24 heures, contactez le support Azure. »

Comme indiqué dans le message, ce processus de vérification de fraude peut prendre jusqu’à 24 heures. Pendant ce temps, ce message continuera à s’afficher.

Si votre certificat App Service continue d’afficher ce message après 24 heures, exécutez le script PowerShell suivant. Le script contacte directement le [fournisseur de certificat](https://www.godaddy.com/) pour résoudre le problème.

```powershell
Connect-AzAccount
Set-AzContext -SubscriptionId <subId>
$actionProperties = @{
    "Name"= "<Customer Email Address>"
    };
Invoke-AzResourceAction -ResourceGroupName "<App Service Certificate Resource Group Name>" -ResourceType Microsoft.CertificateRegistration/certificateOrders -ResourceName "<App Service Certificate Resource Name>" -Action resendRequestEmails -Parameters $actionProperties -ApiVersion 2015-08-01 -Force   
```

## <a name="how-do-authentication-and-authorization-work-in-app-service"></a>Comment fonctionnent l’authentification et l’autorisation dans Azure App Service

Pour obtenir des informations détaillée concernant l’authentification et l’autorisation dans App Service, consultez la documentation relative aux connexions avec les différents fournisseurs d’identité :
* [Azure Active Directory](configure-authentication-provider-aad.md)
* [Facebook](configure-authentication-provider-facebook.md)
* [Google](configure-authentication-provider-google.md)
* [Compte Microsoft](configure-authentication-provider-microsoft.md)
* [Twitter](configure-authentication-provider-twitter.md)

## <a name="how-do-i-redirect-the-default-azurewebsitesnet-domain-to-my-azure-web-apps-custom-domain"></a>Comment rediriger le domaine *.azurewebsites.net par défaut vers le domaine personnalisé de mon application web Azure ?

Lorsque vous créez un site web à l’aide de Web Apps dans Azure, un domaine *nomdusite*.azurewebsites.net par défaut est attribué à votre site. Si vous ajoutez un nom d’hôte personnalisé à votre site et ne souhaitez pas que les utilisateurs puissent accéder au domaine *.azurewebsites.net par défaut, vous pouvez rediriger l’URL par défaut. Pour savoir comment rediriger tout le trafic du domaine par défaut de votre site web vers votre domaine personnalisé, voir [Rediriger le domaine par défaut vers votre domaine personnalisé dans Azure Web Apps](https://zainrizvi.io/blog/block-default-azure-websites-domain/).

## <a name="how-do-i-determine-which-version-of-net-version-is-installed-in-app-service"></a>Comment déterminer la version de .NET installée dans App Service ?

La méthode la plus rapide pour trouver la version de Microsoft .NET installée dans App Service consiste à utiliser la console Kudu. Vous pouvez accéder à la console Kudu à partir du portail ou à l’aide de l’URL de votre application App Service. Pour des instructions détaillées, voir [Comment déterminer la version de .NET installée dans Azure App Services](/archive/blogs/waws/how-to-determine-the-installed-net-version-in-azure-app-services).

## <a name="why-isnt-autoscale-working-as-expected"></a>Pourquoi la mise à l’échelle automatique ne fonctionne-t-elle pas comme prévu ?

Si la fonction de mise à l’échelle automatique Azure n’a pas modifié l’échelle de l’instance d’application web comme vous le souhaitez, vous pouvez passer à un scénario dans lequel nous choisissons intentionnellement de ne pas mettre à l’échelle afin d’éviter une boucle infinie résultant d’un « bagottement ». Cela se produit généralement lorsqu’il n’existe pas de marge suffisante entre les seuils d’augmentation ou de diminution de l’échelle. Pour découvrir comment éviter ce « bagottement » ainsi que d’autres meilleures pratiques de mise à l’échelle automatique, voir [Meilleures pratiques relatives à la mise à l’échelle automatique](../azure-monitor/platform/autoscale-best-practices.md#autoscale-best-practices).

## <a name="why-does-autoscale-sometimes-scale-only-partially"></a>Pourquoi la mise à l’échelle automatique n’opère-t-elle parfois que partiellement ?

Une mise à l’échelle est déclenchée quand des métriques dépassent les limites préconfigurées. Il se peut que vous observiez parfois que la capacité n’est que partiellement complétée par rapport à ce que vous attendiez. Cela peut se produire lorsque le nombre d’instances que vous souhaitez n’est pas disponible. Dans ce scénario, la mise à l’échelle automatique complète partiellement le nombre disponibles d’instances. La mise à l’échelle automatique exécute ensuite la logique de rééquilibrage pour obtenir davantage de capacité. Elle alloue les instances restantes. Notez que cela peut prendre quelques minutes.

Si vous ne voyez pas le nombre attendu d’instances après quelques minutes, ce peut être parce que le rechargement partiel était suffisant pour ramener les mesures dans les limites. Ou bien, il se peut que la mise à l’échelle ait opéré une descente en puissance après avoir atteint la limite inférieure des métriques.

Si aucune de ces conditions ne s’applique et que le problème persiste, envoyez une demande de support.

## <a name="how-do-i-turn-on-http-compression-for-my-content"></a>Comment activer sur la compression HTTP pour mon contenu ?

Pour activer la compression des contenu statiques et dynamiques, ajoutez le code suivant au fichier web.config de niveau application :

```xml
<system.webServer>
    <urlCompression doStaticCompression="true" doDynamicCompression="true" />
</system.webServer>
```

Vous pouvez également spécifier les types MIME dynamiques et statiques spécifiques que vous souhaitez compresser. Pour plus d’informations, voir notre réponse à une question posée sur le forum [httpCompression settings on a simple Azure Website](https://social.msdn.microsoft.com/Forums/azure/890b6d25-f7dd-4272-8970-da7798bcf25d/httpcompression-settings-on-a-simple-azure-website?forum=windowsazurewebsitespreview).

## <a name="how-do-i-migrate-from-an-on-premises-environment-to-app-service"></a>Comment migrer à partir d’un environnement local vers App Service ?

Pour migrer des sites à partir de serveurs web Windows et Linux vers App Service, vous pouvez utiliser l’Assistant Migration d’Azure App Service. L’outil de migration crée les applications et bases de données web dans Azure en fonction des besoins, puis publie le contenu. Pour plus d’informations, voir [Assistant Migration d’Azure App Service](https://appmigration.microsoft.com/).
