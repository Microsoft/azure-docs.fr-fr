---
title: Bien démarrer avec la mise à l’échelle automatique dans Azure
description: Découvrez comment mettre à l’échelle votre ressource Application web, Service cloud, Machine virtuelle ou Jeu de mise à l’échelle de machines virtuelles dans Azure.
ms.topic: conceptual
ms.date: 07/07/2017
ms.subservice: autoscale
ms.openlocfilehash: 364309301b403234936da1bac6e1b74af24c2fdb
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573304"
---
# <a name="get-started-with-autoscale-in-azure"></a>Bien démarrer avec la mise à l’échelle automatique dans Azure
Cet article décrit comment configurer vos paramètres de mise à l’échelle automatique pour votre ressource dans le portail Microsoft Azure.

La mise à l’échelle automatique Azure Monitor s’applique uniquement aux [groupes de machines virtuelles identiques](https://azure.microsoft.com/services/virtual-machine-scale-sets/), aux [services cloud](https://azure.microsoft.com/services/cloud-services/), à [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) et aux [services de gestion des API](../../api-management/api-management-key-concepts.md).

## <a name="discover-the-autoscale-settings-in-your-subscription"></a>Découvrir les paramètres de mise à l’échelle automatique dans votre abonnement

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4u7ts]

Vous pouvez découvrir toutes les ressources pour lesquelles la mise à l’échelle automatique est applicable dans Azure Monitor. Pour une procédure pas à pas, procédez comme suit :

1. Ouvrez le [portail Azure][1].
1. Cliquez sur l’icône Azure Monitor dans le volet gauche.
  ![Ouvrez Azure Monitor][2]
1. Cliquez sur **Mise à l’échelle automatique** pour afficher toutes les ressources pour lesquelles la mise à l’échelle automatique est applicable, ainsi que leur état actuel de mise à l’échelle automatique.
  ![Découvrir la mise à l’échelle automatique dans Azure Monitor][3]

Vous pouvez utiliser le volet de filtre en haut pour réduire l’étendue de la liste afin de sélectionner des ressources dans un groupe de ressources spécifique, des types de ressources spécifiques ou une ressource spécifique.

Pour chaque ressource, vous trouverez le nombre d’instances en cours ainsi que son état de mise à l’échelle automatique. L’état de mise à l’échelle automatique peut être :

- **Non configuré** : vous n’avez pas encore activé la mise à l’échelle automatique pour cette ressource.
- **Enabled** : vous avez activé la mise à l’échelle automatique pour cette ressource.
- **Disabled** : vous avez désactivé la mise à l’échelle automatique pour cette ressource.

## <a name="create-your-first-autoscale-setting"></a>Créez votre premier paramètre de mise à l’échelle automatique

Suivons maintenant une procédure simple pour créer votre premier paramètre de mise à l’échelle automatique.

1. Ouvrez la panneau **Mise à l’échelle automatique** dans Azure Monitor et sélectionnez une ressource à mettre à l’échelle. (les étapes ci-dessous utilisent un plan App Service associé à une application Web. Vous pouvez [créer votre première application web ASP.NET dans Azure en 5 minutes][4].)
1. Notez que le nombre d’instances actuel est 1. Cliquez sur **Activer la mise à l’échelle automatique**.
  ![Paramètre d’échelle pour la nouvelle application web][5]
1. Fournissez un nom pour le paramètre de mise à l’échelle, puis cliquez sur **Ajouter une règle**. Notez les options de règle de mise à l’échelle qui s’ouvrent dans un volet contextuel dans la partie droite. Par défaut, l’option de mise à l’échelle du nombre d’instances est définie sur 1 si le pourcentage processeur de la ressource dépasse 70 %. Laissez les valeurs par défaut et cliquez sur **Ajouter**.
  ![Créer le paramètre de mise à l’échelle pour une application web][6]
1. Vous avez créé votre première règle de mise à l’échelle. Notez que l’expérience utilisateur recommande les meilleures pratiques et indique « qu’il est recommandé d’avoir au moins une règle de scale-in. » Pour ce faire :

    a. Cliquez sur **Ajouter une règle**.

    b. Définissez **Opérateur** sur **moins de**.

    c. Définissez **Seuil** sur **20**.

    d. Définissez **Opération** sur **Diminuer le nombre par**.

   Vous devez maintenant avoir un paramètre de mise à l’échelle qui fait monter/diminuer en puissance en fonction de l’utilisation du processeur.
   ![Mise à l’échelle en fonction du processeur][8]
1. Cliquez sur **Enregistrer**.

Félicitations ! Vous avez maintenant correctement créé votre premier paramètre de mise à l’échelle pour mettre à l’échelle automatiquement votre application Web en fonction de l’utilisation du processeur.

> [!NOTE]
> Les mêmes étapes sont applicables pour bien démarrer avec un jeu de mise à l’échelle de machines virtuelles ou un rôle de service cloud.

## <a name="other-considerations"></a>Autres considérations
### <a name="scale-based-on-a-schedule"></a>Mise à l'échelle en fonction d’une planification
En plus de la mise à l’échelle en fonction du processeur, vous pouvez aussi définir votre mise à l’échelle différemment pour certains jours de la semaine.

1. Cliquez sur **Ajouter une condition de mise à l’échelle**.
1. La définition du mode de mise à l’échelle et des règles est identique à la condition par défaut.
1. Sélectionnez **Répéter des jours spécifiques** pour la planification.
1. Sélectionnez les jours et les heures de début/fin auxquels la condition de mise à l’échelle doit être appliquée.

![Condition de mise à l’échelle basée sur une planification][9]
### <a name="scale-differently-on-specific-dates"></a>Mettre à l’échelle différemment à des dates spécifiques
En plus de la mise à l’échelle en fonction du processeur, vous pouvez aussi définir votre mise à l’échelle différemment pour certaines dates spécifiques.

1. Cliquez sur **Ajouter une condition de mise à l’échelle**.
1. La définition du mode de mise à l’échelle et des règles est identique à la condition par défaut.
1. Sélectionnez **Spécifier les dates de début et de fin** pour la planification.
1. Sélectionnez les dates et les heures de début/fin auxquelles la condition de mise à l’échelle doit être appliquée.

![Condition de mise à l’échelle en fonction des dates][10]

### <a name="view-the-scale-history-of-your-resource"></a>Afficher l’historique de mise à l’échelle de votre ressource
Chaque fois que votre ressource monte/diminue en puissance, un événement est enregistré dans le journal d’activité. Vous pouvez afficher l’historique de la mise à l’échelle de votre ressource pour les dernières 24 heures en basculant vers l’onglet **Exécuter l’historique**.

![Historique d’exécution][11]

Si vous souhaitez afficher l’historique de la mise à l’échelle complet (jusqu'à 90 jours), sélectionnez **Cliquez ici pour obtenir plus de détails**. Ce journal d’activité s’ouvre, avec la mise à l’échelle automatique présélectionnée pour la ressource et la catégorie.

### <a name="view-the-scale-definition-of-your-resource"></a>Afficher la définition de mise à l’échelle de votre ressource
La mise à l’échelle est une ressource Azure Resource Manager. Vous pouvez afficher la définition de la mise à l’échelle dans JSON en basculant vers l’onglet **JSON**.

![Définition de mise à l’échelle][12]

Vous pouvez apporter des modifications dans le JSON directement, si nécessaire. Ces modifications apparaîtront après que les avoir enregistrées.

### <a name="disable-autoscale-and-manually-scale-your-instances"></a>Désactiver la mise à l’échelle automatique et mettre à l’échelle manuellement vos instances
Il peut arriver que vous souhaitiez désactiver vos paramètres actuels de mise à l’échelle et mettre à l’échelle manuellement votre ressource.

Cliquez sur le bouton **Désactiver la mise à l’échelle automatique** en haut.
![Désactiver la mise à l’échelle automatique][13]

> [!NOTE]
> Cette option désactive votre configuration. Toutefois, vous pouvez revenir dessus une fois que vous activez à nouveau la mise à l’échelle automatique.

Vous pouvez maintenant définir le nombre d’instances à mettre à l’échelle sur manuellement.

![Définir l’échelle manuelle][14]

Vous pouvez toujours revenir à la mise à l’échelle automatique en cliquant sur **Activer la mise à l’échelle automatique** puis sur **Enregistrer**.

## <a name="route-traffic-to-healthy-instances-app-service"></a>Acheminer le trafic vers des instances saines (App Service)

Lorsque vous effectuez un scale-out vers plusieurs instances, App Service peut effectuer des contrôles d’intégrité sur vos instances pour acheminer le trafic uniquement vers les instances saines. Pour ce faire, ouvrez le portail de votre App Service, puis sélectionnez **Contrôle d’intégrité** sous **Analyse**. Sélectionnez **Activer** et fournissez un chemin d’URL valide pour votre application, par exemple `/health` ou `/api/health`. Cliquez sur **Enregistrer**.

Pour activer la fonctionnalité avec les modèles ARM, définissez la propriété `healthcheckpath` de la ressource `Microsoft.Web/sites` sur le chemin d’accès de contrôle d’intégrité sur votre site, par exemple : `"/api/health/"`. Pour désactiver la fonctionnalité, redéfinissez la propriété sur la chaîne vide, `""`.

### <a name="health-check-path"></a>Chemin de contrôle d'intégrité

Le chemin d’accès doit répondre dans un délai d’une minute avec un code d’état compris entre 200 et 299 (inclus). Si le chemin d’accès ne répond pas dans la minute ou s’il retourne un code d’état en dehors de cette plage, l’instance est considérée comme « non saine ». App Service ne suit pas les redirections 302 sur le chemin de contrôle d’intégrité. Le contrôle d’intégrité s’intègre aux fonctionnalités d’authentification et d’autorisation d’App Service. Le système atteindra le point de terminaison même si ces fonctionnalités de sécurité sont activées. Si vous utilisez votre propre système d’authentification, le chemin du contrôle d’intégrité doit autoriser l’accès anonyme. Si HTTP **S** uniquement est activé sur le site, la requête Healthcheck sera envoyée via HTTP **S**.

Le chemin du contrôle d'intégrité doit vérifier les composants critiques de votre application. Par exemple, si votre application dépend d’une base de données et d’un système de messagerie, le point de terminaison de contrôle d’intégrité doit se connecter à ces composants. Si l’application ne peut pas se connecter à un composant critique, le chemin d’accès doit retourner un code de réponse de niveau 500 pour indiquer que l’application n’est pas saine.

#### <a name="security"></a>Sécurité 

Les équipes de développement des grandes entreprises doivent souvent adhérer à des exigences de sécurité pour leurs API exposées. Pour sécuriser le point de terminaison du contrôle d’intégrité vous devez d’abord utiliser des fonctionnalités telles que des [restrictions d’adresse IP](../../app-service/app-service-ip-restrictions.md#set-an-ip-address-based-rule), des [certificats clients](../../app-service/app-service-ip-restrictions.md#set-an-ip-address-based-rule) ou un réseau virtuel pour restreindre l’accès à l’application. Vous pouvez sécuriser le point de terminaison du contrôle d’intégrité proprement dit en exigeant que la propriété `User-Agent` de la requête entrante corresponde à `ReadyForRequest/1.0`. La propriété User-Agent ne peut pas être falsifiée, car la requête a déjà été sécurisée par les fonctionnalités de sécurité précédentes.

### <a name="behavior"></a>Comportement

Lorsque le chemin du contrôle d’intégrité est fourni, App Service effectue un test ping du chemin toutes les instances. Si un code de réponse correct n’est pas reçu après 5 tests ping, cette instance est considérée comme « non saine ». Les instances non saintes sont exclues de la rotation de l’équilibreur de charge si vous effectuez un scale-out jusqu’à 2 instances ou plus et que vous utilisez le [niveau de base](../../app-service/overview-hosting-plans.md) ou supérieur. Vous pouvez configurer le nombre requis de tests Ping ayant échoué avec le paramètre d’application `WEBSITE_HEALTHCHECK_MAXPINGFAILURES`. Ce paramètre d’application peut être défini sur n’importe quel entier compris entre 2 et 10. Par exemple, si cette valeur est définie sur `2`, vos instances seront supprimées de l’équilibreur de charge après deux échecs de test Ping. En outre, lorsque vous effectuez un scale-up ou un scale-out, App Service effectue un test Ping sur le chemin du contrôle d’intégrité pour s’assurer que les nouvelles instances sont prêtes à recevoir des requêtes avant d’être ajoutées à l’équilibreur de charge.

> [!NOTE]
> N’oubliez pas que votre plan App Service doit faire l’objet d’un scale-out à hauteur de 2 instances et utiliser le **niveau de base ou supérieur** pour que l’exclusion de l’équilibreur de charge se produise. Si vous n’avez qu’une seule instance, elle n’est pas supprimée de l’équilibreur de charge, même si elle n’est pas saine. 

Les instances saines restantes peuvent subir une augmentation de charge. Pour éviter de submerger les instances restantes, jusqu’à la moitié de vos instances sera exclue. Par exemple, si un scale-out du plan d’App Service vers 4 instances dont 3 qui ne sont pas saines est effectué, au moins 2 instances seront exclues de la rotation exclu de la rotation de LoadBalancer. Les 2 autres instances (1 saine et 1 non saine) continueront de recevoir des requêtes. Dans le pire des cas où toutes les instances sont non saines, aucune ne sera exclue. Si vous souhaitez remplacer ce comportement, vous pouvez définir le paramètre d’application `WEBSITE_HEALTHCHECK_MAXUNHEALTHYWORKERPERCENT` sur une valeur comprise entre `0` et `100`. Si vous attribuez une valeur plus élevée, vous supprimez les instances non saines (la valeur par défaut est 50).

Si une instance n’est pas saine pendant une heure, elle sera remplacée par une nouvelle instance. Une instance au plus sera remplacée chaque heure, avec un maximum de trois instances par jour et par plan App Service.

### <a name="monitoring"></a>Surveillance

Après avoir fourni le chemin de contrôle d’intégrité de votre application, vous pouvez surveiller l’intégrité de votre site à l’aide d’ Azure Monitor. Dans le panneau **Contrôle d’intégrité** du portail, cliquez sur **Métriques** dans la barre d’outils supérieure. Un nouveau panneau s’ouvre, dans lequel vous pouvez voir l’état d’intégrité historique du site et créer une nouvelle règle d’alerte. Pour plus d’informations sur la surveillance de vos sites, [consultez le guide sur Azure Monitor](../../app-service/web-sites-monitor.md).

## <a name="moving-autoscale-to-a-different-region"></a>Déplacement de la mise à l’échelle automatique vers une autre région
Cette section explique comment déplacer la mise à l’échelle automatique Azure vers une autre région sous le même abonnement et le même groupe de ressources. Vous pouvez utiliser l’API REST pour déplacer les paramètres de mise à l’échelle automatique.
### <a name="prerequisite"></a>Configuration requise
1. Vérifiez que l’abonnement et le groupe de ressources sont disponibles et que les détails dans les régions source et de destination sont identiques.
1. Vérifiez que la mise à l’échelle automatique Azure est disponible dans la [région Azure vers laquelle vous souhaitez effectuer le déplacement](https://azure.microsoft.com/global-infrastructure/services/?products=monitor&regions=all).

### <a name="move"></a>Déplacer
Utilisez l’[API REST](/rest/api/monitor/autoscalesettings/createorupdate) pour créer un paramètre de mise à l’échelle automatique dans le nouvel environnement. Le paramètre de mise à l’échelle automatique créé dans la région de destination est une copie du paramètre de mise à l’échelle automatique dans la région source.

Les [paramètres de diagnostic](./diagnostic-settings.md) créés en association avec le paramètre de mise à l’échelle automatique dans la région source ne peuvent pas être déplacés. Vous devez recréer les paramètres de diagnostic dans la région de destination une fois créés les paramètres de mise à l’échelle automatique. 

### <a name="learn-more-about-moving-resources-across-azure-regions"></a>En savoir plus sur le déplacement de ressources dans les régions Azure
Pour en savoir plus sur le déplacement de ressources entre régions et sur la reprise d’activité après sinistre dans Azure, consultez [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md).

## <a name="next-steps"></a>Étapes suivantes
- [Créez une alerte de journal d’activité pour surveiller toutes les opérations du moteur de mise à l’échelle automatique dans votre abonnement.](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-alert)
- [Créez une alerte de journal d’activité pour surveiller tous les échecs d’opérations de diminution et d’augmentation de la taille des instances de la mise à l’échelle automatique dans votre abonnement](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-failed-alert)

<!--Reference-->
[1]:https://portal.azure.com
[2]: ./media/autoscale-get-started/azure-monitor-launch.png
[3]: ./media/autoscale-get-started/discover-autoscale-azure-monitor.png
[4]: ../../app-service/quickstart-dotnetcore.md
[5]: ./media/autoscale-get-started/scale-setting-new-web-app.png
[6]: ./media/autoscale-get-started/create-scale-setting-web-app.png
[7]: ./media/autoscale-get-started/scale-in-recommendation.png
[8]: ./media/autoscale-get-started/scale-based-on-cpu.png
[9]: ./media/autoscale-get-started/scale-condition-schedule.png
[10]: ./media/autoscale-get-started/scale-condition-dates.png
[11]: ./media/autoscale-get-started/scale-history.png
[12]: ./media/autoscale-get-started/scale-definition-json.png
[13]: ./media/autoscale-get-started/disable-autoscale.png
[14]: ./media/autoscale-get-started/set-manualscale.png
