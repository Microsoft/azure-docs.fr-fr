---
title: Configurer un service QnA Maker - QnA Maker
description: Avant de pouvoir créer des bases de connaissances QnA Maker, vous devez tout d’abord configurer un service QnA Maker dans Azure. Toute personne disposant d’autorisations pour créer des ressources dans un abonnement peut configurer un service QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: 83917214705546b21553e997ccab11a7511f77fd
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353304"
---
# <a name="manage-qna-maker-resources"></a>Gérer les ressources QnA Maker

Avant de pouvoir créer des bases de connaissances QnA Maker, vous devez tout d’abord configurer un service QnA Maker dans Azure. Toute personne disposant d’autorisations pour créer des ressources dans un abonnement peut configurer un service QnA Maker.

Veillez à bien assimiler les concepts suivants avant de créer votre ressource :

* [Ressources QnA Maker](../Concepts/azure-resources.md)
* [Création et publication de clés](../Concepts/azure-resources.md#keys-in-qna-maker)

## <a name="create-a-new-qna-maker-service"></a>Créer un nouveau service QnA Maker

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Cette procédure permet de créer les ressources Azure nécessaires pour gérer le contenu de la base de connaissances. Une fois ces étapes terminées, vous trouverez les clés _d’abonnement_ dans la page **Clés** de la ressource dans la portail Azure.

1. Connectez-vous au portail Azure et [créez une ressource QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker).

1. Sélectionnez **Créer** après avoir lu les conditions générales :

    ![Créer un nouveau service QnA Maker](../media/qnamaker-how-to-setup-service/create-new-resource-button.png)

1. Dans **QnA Maker**, sélectionnez les niveaux et les régions appropriés :

    ![Créer un service QnA Maker - Niveau tarifaire et régions](../media/qnamaker-how-to-setup-service/enter-qnamaker-info.png)

    * Dans le champ **Nom**, renseignez un nom unique pour identifier ce service QnA Maker. Ce nom identifie également le point de terminaison QnA Maker auquel vos bases de connaissances seront associées.
    * Choisissez **l’abonnement** dans lequel la ressource QnA Maker sera déployée.
    * Sélectionnez le **Niveau tarifaire** pour les services d’administration de QnA Maker (portail et API de gestion). Voir [plus d’informations sur la tarification des références SKU](https://aka.ms/qnamaker-pricing).
    * Créez un nouveau **Groupe de ressources** (recommandé) ou utilisez un groupe de ressources existant dans lequel déployer cette ressource QnA Maker. QnA Maker crée plusieurs ressources Azure. Lorsque vous créez un groupe de ressources pour conserver ces ressources, vous pouvez facilement les rechercher, les gérer et les supprimer par le biais du nom du groupe de ressources.
    * Sélectionnez un **Emplacement du groupe de ressources**.
    * Choisissez le **Niveau tarifaire de recherche** du service Recherche cognitive Azure. Si l’option de niveau Gratuit est indisponible (semble grisée), cela signifie que vous disposez déjà d’un service gratuit déployé dans votre abonnement. Dans ce cas, vous devrez commencer par le niveau De base. Consultez les [Détails de la tarification du service Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/).
    * Choisissez **l’emplacement de recherche** où vous souhaitez déployer les index du service Recherche cognitive Azure. Les restrictions relatives à l’emplacement de stockage des données client vous aideront à déterminer l’emplacement que vous choisissez pour la Recherche cognitive Azure.
    * Dans le champ **Nom de l’application**, entrez un nom pour votre instance Azure App Service.
    * Par défaut, App Service utilise le niveau Standard (S1). Vous pouvez modifier le plan après sa création. Consultez des informations supplémentaires sur [la tarification d’App Service](https://azure.microsoft.com/pricing/details/app-service/).
    * Choisissez **l’emplacement du site web** où App Service sera déployé.

        > [!NOTE]
        > **Emplacement de recherche** peut être différent de **Emplacement du site web**.

    * Choisissez si vous souhaitez activer **Application Insights** ou non. Si **Application Insights** est activé, QnA Maker collecte les données de télémétrie sur le trafic, les journaux d’activité de conversation et les erreurs.
    * Choisissez **l’emplacement d’Application Insights** où la ressource Application Insights sera déployée.
    * Pour réduire vos coûts, vous pouvez [partager](#configure-qna-maker-to-use-different-cognitive-search-resource) certaines des ressources Azure créées pour QnA Maker, mais pas toutes.

1. Une fois tous les champs validés, sélectionnez **Créer**. L’exécution de ce processus peut prendre plusieurs minutes.

1. Une fois le déploiement terminé, vous verrez les ressources suivantes créées dans votre abonnement :

   ![Service QnA Maker créé par une ressource](../media/qnamaker-how-to-setup-service/resources-created.png)

    La ressource avec le type _Cognitive Services_ a vos clés _d’abonnement_.

### <a name="upgrade-qna-maker-sku"></a>Mettre à niveau la référence SKU de QnA Maker

Si vous souhaitez avoir plus de questions et de réponses dans votre base de connaissances, au-delà de votre niveau actuel, mettez à niveau votre niveau tarifaire du service QnA Maker.

Pour mettre à niveau la référence SKU de gestion de QnA Maker :

1. Accédez à votre ressource QnA Maker dans le portail Azure et sélectionnez **Niveau tarifaire**.

    ![Ressource QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-resource.png)

1. Choisissez la référence SKU appropriée et appuyez sur **Sélectionner**.

    ![Tarification de QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-pricing-page.png)

### <a name="upgrade-app-service"></a>Mise à niveau du service d’application

 Lorsque votre base de connaissances doit répondre à un plus grand nombre de demandes de votre application client, mettez à niveau le niveau tarifaire d’App Service.

Vous pouvez effectuer un [scale-up](../../../app-service/manage-scale-up.md) ou un scale-out du service d’application.

Accédez à la ressource App Service dans le portail Azure et sélectionnez l’option **Scale-up** ou **Scale-out** en fonction des besoins.

![Mise à l'échelle du service d’application QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-appservice-scale.png)

### <a name="get-the-latest-runtime-updates"></a>Téléchargement des dernières mises à jour du Runtime

Le runtime QnAMaker fait partie de l’instance Azure App Service, qui est déployé quand vous [créez un service QnAMaker](./set-up-qnamaker-service-azure.md) dans le portail Azure. Des mises à jour du runtime sont effectuées régulièrement. L’instance QnA Maker App Service est en mode de mise à jour automatique à partir de la publication de l’extension de site d’avril 2019 (version 5+). Cette mise à jour a déjà été conçue pour éviter le moindre temps d’arrêt pendant les mises à niveau.

Vous pouvez vérifier votre version actuelle à l’adresse https://www.qnamaker.ai/UserSettings. Si votre version est antérieure à la version 5.x, vous devez redémarrer App Service pour appliquer les dernières mises à jour :

1. Accédez à votre service QnAMaker (groupe de ressources) dans le [portail Azure](https://portal.azure.com).

    > [!div class="mx-imgBorder"]
    > ![Groupe de ressources Azure QnAMaker](../media/qnamaker-how-to-troubleshoot/qnamaker-azure-resourcegroup.png)

1. Sélectionnez l’instance d’App Service et ouvrez la section **Vue d’ensemble**.

    > [!div class="mx-imgBorder"]
    > ![Instance de l’App Service QnAMaker](../media/qnamaker-how-to-troubleshoot/qnamaker-azure-appservice.png)


1. Redémarrez App Service. Le processus de mise à jour doit se terminer en quelques secondes. Les applications ou bots dépendants qui utilisent ce service QnAMaker ne sont pas disponibles pour les utilisateurs finaux durant le redémarrage.

    ![Redémarrage de l’instance d’App Service de QnAMaker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-appservice-restart.png)

### <a name="configure-app-service-idle-setting-to-avoid-timeout"></a>Configurer le paramètre d’inactivité d’App Service pour éviter le délai d’expiration

Le service d’application, qui sert le runtime de prédiction QnA Maker d’une base de connaissances publiée, a une configuration de délai d’inactivité, laquelle utilise par défaut le délai d’expiration automatique si le service est inactif. Pour QnA Maker, cela signifie que votre API generateAnswer de runtime des prédictions expire parfois après des périodes sans trafic.

Pour que l’application de point de terminaison de prédiction soit chargée même en l’absence de trafic, définissez la valeur d’inactivité sur Toujours activé.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez et sélectionnez le service d’application de votre ressource QnA Maker. Il aura le même nom que la ressource QnA Maker, mais il aura un **type** différent d’App Service.
1. Recherchez **Paramètres** puis sélectionnez **Configuration**.
1. Dans le volet de configuration, sélectionnez **Paramètres généraux**, recherchez **Toujours activé**, puis sélectionnez **Activé** comme valeur.

    > [!div class="mx-imgBorder"]
    > ![Dans le volet de configuration, sélectionnez **Paramètres généraux**, puis recherchez **Toujours activé**, et sélectionnez la valeur **Activé**.](../media/qnamaker-how-to-upgrade-qnamaker/configure-app-service-idle-timeout.png)

1. Sélectionnez **Enregistrer** pour enregistrer la configuration.
1. Vous êtes invité à redémarrer l’application pour qu’elle utilise le nouveau paramètre. Sélectionnez **Continuer**.

En savoir plus sur la configuration des [Paramètres généraux](../../../app-service/configure-common.md#configure-general-settings) de l’App Service .

### <a name="configure-app-service-environment-to-host-qna-maker-app-service"></a>Configurer App Service Environment pour héberger le service d’application QnA Maker
App Service Environment peut être utilisé pour héberger le service d’application QnA Maker. Si App Service Environment est interne, vous devez suivre les étapes suivantes :
1. Créez un service d’application et un service de recherche Azure.
2. Exposez le service d’application et autorisez la disponibilité de QnA Maker comme suit :
    * Disponible publiquement : par défaut
    * Étiquette du service DNS : `CognitiveServicesManagement`
3. Créez une instance de service cognitif QnA Maker (Microsoft.CognitiveServices/accounts) à l’aide d’Azure Resource Manager, où le point de terminaison QnA Maker doit être défini sur App Service Environment.

### <a name="network-isolation-for-app-service"></a>Isolement réseau pour App Service

Le service cognitif QnA Maker utilise l’étiquette de service : `CognitiveServicesManagement`. Pour ajouter les plages d’adresses IP à une liste verte, effectuez les étapes suivantes :

* Téléchargez les [plages d’adresses IP de toutes les étiquettes de service](https://www.microsoft.com/download/details.aspx?id=56519).
* Sélectionnez les adresses IP de « CognitiveServicesManagement ».
* Accédez à la section réseau de votre ressource App Service, puis cliquez sur l’option « Configurer une restriction d’accès » pour ajouter les adresses IP à une liste verte.

Nous avons aussi un script automatisé pour faire la même chose pour votre App Service. Vous trouverez le [script PowerShell pour configurer une liste verte](https://github.com/pchoudhari/QnAMakerBackupRestore/blob/master/AddRestrictedIPAzureAppService.ps1) sur GitHub. Vous devez entrer l’ID d’abonnement, le groupe de ressources et le nom App Service réel en tant que paramètres du script. L’exécution du script va automatiquement ajouter les adresses IP à la liste verte App Service.

### <a name="business-continuity-with-traffic-manager"></a>Continuité d'activité avec Traffic Manager

L’objectif principal du plan de continuité d’activité est de créer un point de terminaison de base de connaissances résilient qui garantit qu’il n’y a aucun temps d’arrêt du bot ou de l’application qui la consomme.

> [!div class="mx-imgBorder"]
> ![Plan de continuité d’activité QnA Maker](../media/qnamaker-how-to-bcp-plan/qnamaker-bcp-plan.png)

L’idée générale représentée plus haut est la suivante :

1. Configurez deux [services QnA Maker](set-up-qnamaker-service-azure.md) parallèles dans des [régions Azure associées](../../../best-practices-availability-paired-regions.md).

1. [Sauvegardez](../../../app-service/manage-backup.md) votre service d'application QnA Maker principal et [restaurez-le](../../../app-service/web-sites-restore.md) dans la configuration secondaire. Ainsi, les deux configurations fonctionneront avec les mêmes nom d'hôte et clés.

1. Synchronisez les index de recherche Azure principal et secondaire. Utilisez l’exemple GitHub [ici](https://github.com/pchoudhari/QnAMakerBackupRestore) pour savoir comment sauvegarder/restaurer les index Azure.

1. Sauvegardez Application Insights avec [exportation continue](../../../azure-monitor/app/export-telemetry.md).

1. Une fois que les piles principale et secondaire ont été configurées, utilisez [Gestionnaire de trafic](../../../traffic-manager/traffic-manager-overview.md) pour configurer les deux points de terminaison et définir une méthode de routage.

1. Vous devez créer un certificat Transport Layer Security (TLS), anciennement Secure Sockets Layer (SSL), pour le point de terminaison de votre gestionnaire de trafic. [Liez le certificat TLS/SSL](../../../app-service/configure-ssl-bindings.md) dans vos services d’application.

1. Enfin, utilisez le point de terminaison du gestionnaire de trafic dans votre bot ou dans votre application.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Cette procédure permet de créer les ressources Azure nécessaires pour gérer le contenu de la base de connaissances. Une fois ces étapes terminées, vous trouverez les clés *d’abonnement* dans la page **Clés** de la ressource dans la portail Azure.

1. Connectez-vous au portail Azure et [créez une ressource QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker).

1. Sélectionnez **Créer** après avoir lu les conditions générales :

    ![Créer un nouveau service QnA Maker](../media/qnamaker-how-to-setup-service/create-new-resource-button.png)

1. Dans **QnA Maker**, cochez la case Managé (préversion) et sélectionnez les niveaux et régions appropriés :

    ![Créer un service QnA Maker managé - Niveau tarifaire et régions](../media/qnamaker-how-to-setup-service/enter-qnamaker-v2-info.png)

    * Choisissez **l’abonnement** dans lequel la ressource QnA Maker sera déployée.
    * Créez un **groupe de ressources** (recommandé) ou utilisez-en un existant dans lequel déployer cette ressource QnA Maker managé (préversion). QnA Maker managé (préversion) crée quelques ressources Azure. Lorsque vous créez un groupe de ressources pour conserver ces ressources, vous pouvez facilement les rechercher, les gérer et les supprimer par le biais du nom du groupe de ressources.
    * Dans le champ **Nom**, renseignez un nom unique pour identifier ce service QnA Maker managé (préversion). 
    * Choisissez l’**emplacement** où vous voulez déployer le service QnA Maker managé (préversion). Les API de gestion et le point de terminaison de service seront hébergés à cet emplacement. 
    * Sélectionnez le **niveau tarifaire** du service QnA Maker managé (préversion) (gratuit en préversion). Voir [plus d’informations sur la tarification des références SKU](https://aka.ms/qnamaker-pricing).
    * Choisissez **l’emplacement de recherche** où vous souhaitez déployer les index du service Recherche cognitive Azure. Les restrictions relatives à l’emplacement de stockage des données client vous aideront à déterminer l’emplacement que vous choisissez pour la Recherche cognitive Azure.
    * Choisissez le **Niveau tarifaire de recherche** du service Recherche cognitive Azure. Si l’option de niveau Gratuit est indisponible (semble grisée), cela signifie que vous disposez déjà d’un service gratuit déployé dans votre abonnement. Dans ce cas, vous devrez commencer par le niveau De base. Consultez les [Détails de la tarification du service Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/).

1. Une fois tous les champs validés, sélectionnez **Vérifier + créer**. L’exécution de ce processus peut prendre plusieurs minutes.

1. Une fois le déploiement terminé, vous verrez les ressources suivantes créées dans votre abonnement :

    ![La ressource a créé un service QnA Maker managé (préversion)](../media/qnamaker-how-to-setup-service/resources-created-v2.png)

    La ressource avec le type _Cognitive Services_ a vos clés _d’abonnement_.

---

## <a name="find-authoring-keys-in-the-azure-portal"></a>Rechercher des clés de création dans le portail Azure

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Vous pouvez afficher et réinitialiser vos clés de création à partir du portail Azure où la ressource QnA Maker a été créée. Ces clés sont parfois appelées clés d’abonnement.

1. Accédez à la ressource QnA Maker dans le portail Azure et sélectionnez la ressource qui a le type _Cognitive Services_ :

    ![Liste des ressources QnA Maker](../media/qnamaker-how-to-key-management/qnamaker-resource-list.png)

2. Accédez à **Clés** :

    ![Clé d’abonnement](../media/qnamaker-how-to-key-management/subscription-key.PNG)

### <a name="find-query-endpoint-keys-in-the-qna-maker-portal"></a>Rechercher des clés de point de terminaison de requête dans le portail QnA Maker

Le point de terminaison se trouve dans la même région que la ressource, car les clés de point de terminaison sont utilisées pour appeler la base de connaissances.

Les clés de point de terminaison peuvent être gérées à partir du [portail QnA Maker](https://qnamaker.ai).

1. Connectez-vous au [portail QnA Maker](https://qnamaker.ai), accédez à votre profil, puis sélectionnez **Paramètres de service** :

    ![Clé de point de terminaison](../media/qnamaker-how-to-key-management/Endpoint-keys.png)

2. Affichez ou réinitialisez vos clés :

    > [!div class="mx-imgBorder"]
    > ![Gestionnaire de la clé de point de terminaison](../media/qnamaker-how-to-key-management/Endpoint-keys1.png)

    >[!NOTE]
    >Actualisez vos clés si vous pensez qu’elles ont été compromises. Cette opération peut nécessiter des modifications correspondantes de votre code de bot ou d’application cliente.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Vous pouvez afficher et réinitialiser vos clés de création à partir du portail Azure où la ressource QnA Maker managé (préversion) a été créée. Ces clés sont parfois appelées clés d’abonnement.

1. Accédez à la ressource QnA Maker managé (préversion) dans le portail Azure et sélectionnez la ressource qui a le type *Cognitive Services* :

    ![Liste des ressources QnA Maker managé (préversion)](../media/qnamaker-how-to-key-management/qnamaker-v2-resource-list.png)

2. Accédez aux **clés et point de terminaison** :

    ![Clé d’abonnement QnA Maker managé (préversion)](../media/qnamaker-how-to-key-management/subscription-key-v2.png)

### <a name="update-the-resources"></a>Mettre à jour les ressources

Découvrez comment mettre à niveau les ressources utilisées par votre base de connaissances. QnA Maker managé (préversion) est **gratuit** en préversion. 

---

## <a name="upgrade-the-azure-cognitive-search-service"></a>Mettre à niveau le service Recherche cognitive Azure

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Si vous prévoyez de disposer de plusieurs bases de connaissances, mettez à niveau votre niveau tarifaire du service Recherche cognitive Azure.

Il n’est actuellement pas possible d’effectuer une mise à niveau sur place de la référence SKU de la recherche Azure. Toutefois, vous pouvez créer une ressource de recherche Azure avec la référence SKU souhaitée, restaurer les données vers la nouvelle ressource, puis la lier à la pile QnA Maker. Pour ce faire, procédez comme suit :

1. Créez une ressource de recherche Azure dans le portail Azure et sélectionnez la référence SKU souhaitée.

    ![Ressources de recherche QnA Maker Azure](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-azuresearch-new.png)

1. Restaurez les index de votre ressource de recherche Azure d’origine vers la nouvelle. Consultez [l’exemple de code de restauration de sauvegarde](https://github.com/pchoudhari/QnAMakerBackupRestore).

1. Une fois que les données sont restaurées, accédez à votre nouvelle ressource de recherche Azure, sélectionnez **Clés**, puis notez le **Nom** et la **Clé d’administration** :

    ![Clés de recherche QnA Maker Azure](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-azuresearch-keys.png)

1. Pour lier la nouvelle ressource de recherche Azure à la pile QnA Maker, accédez à l’instance App Service de QnA Maker.

    ![Instance du service d’application QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-resource-list-appservice.png)

1. Sélectionnez **Paramètres d'application** et modifiez les paramètres dans les champs **AzureSearchName** et **AzureSearchAdminKey** à partir de l’étape 3.

    ![Paramètre de service d’application QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-appservice-settings.png)

1. Redémarrez l’instance App Service.

    ![Redémarrage de l’instance d’App Service de QnA Maker](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-appservice-restart.png)

### <a name="cognitive-search-consideration"></a>Considération relative à Recherche cognitive

Recherche cognitive, en tant que ressource distincte, a des configurations différentes que vous devez connaître.

### <a name="configure-qna-maker-to-use-different-cognitive-search-resource"></a>Configurer QnA Maker pour utiliser une autre ressource de Recherche cognitive

Si vous créez un service QnA et ses dépendances (par exemple, la recherche) via le portail, un service de recherche est créé pour vous et lié au service QnA Maker. Une fois ces ressources créées, vous pouvez mettre à jour le paramètre App Service pour utiliser un service de recherche existant au préalable et supprimer celui que vous venez de créer.

La ressource **App Service** de QnA Maker utilise la ressource Recherche cognitive. Pour modifier la ressource Recherche cognitive qu’utilise QnA Maker, vous devez modifier le paramètre sur le Portail Azure.

1. Récupérez la **Clé d’administration** et le **Nom** de la ressource Recherche cognitive que QnA Maker doit utiliser.

1. Connectez-vous au [Portail Azure](https://portal.azure.com) et recherchez l’**App Service** associé à votre ressource QnA Maker. Les deux portent le même nom.

1. Sélectionnez **Paramètres**, puis **Configuration**. Cela permet d’afficher tous les paramètres existants de l’App Service de QnA Maker.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran du portail Azure montrant les paramètres de configuration d’App Service](../media/qnamaker-how-to-upgrade-qnamaker/change-search-service-app-service-configuration.png)

1. Modifiez les valeurs des clés suivantes :

    * **AzureSearchAdminKey**
    * **AzureSearchName**

1. Pour utiliser les nouveaux paramètres, vous devez redémarrer l’App Service. Sélectionnez **Vue d’ensemble**, puis **Redémarrer**.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de Portail Azure redémarrant App Service après modification des paramètres de configuration](../media/qnamaker-how-to-upgrade-qnamaker/screenshot-azure-portal-restart-app-service.png)

Si vous créez un service QnA par le biais de modèles Azure Resource Manager, vous pouvez créer toutes les ressources et contrôler la création de l’App Service pour utiliser un service de recherche existant.

En savoir plus sur la configuration des [paramètres de l’application](../../../app-service/configure-common.md#configure-app-settings) App Service .

### <a name="configuring-cognitive-search-as-a-private-endpoint-inside-a-vnet"></a>Configuration de Recherche cognitive en tant que point de terminaison privé au sein d’un réseau virtuel

Lorsqu’une instance de recherche est créée lors de la création d’une ressource QnA Maker, vous pouvez forcer Recherche cognitive à prendre en charge une configuration de point de terminaison privée entièrement créée au sein du réseau virtuel d’un client.

Afin d’utiliser un point de terminaison privé, toutes les ressources doivent être créées dans la même région.

* Ressource QnA Maker
* nouvelle ressource Recherche cognitive
* nouvelle ressource Réseau virtuel

Procédez comme suit dans le [portail Azure](https://portal.azure.com) :

1. Créez une [ressource QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker).
1. Créez une ressource Recherche cognitive avec une connectivité Point de terminaison (données) définie sur _Privée_. Créez la ressource dans la même région que la ressource QnA Maker créée à l’étape 1. Apprenez-en davantage sur [la création d’une ressource Recherche cognitive](../../../search/search-create-service-portal.md), puis utilisez ce lien pour accéder directement à la [page de création de la ressource](https://ms.portal.azure.com/#create/Microsoft.Search).
1. Créez une [ressource Réseau virtuel](https://ms.portal.azure.com/#create/Microsoft.VirtualNetwork-ARM).
1. Configurez le réseau virtuel sur la ressource App Service créée à l’étape 1 de cette procédure.
    1. Créez une entrée DNS dans le réseau virtuel pour la nouvelle ressource Recherche cognitive créée à l’étape 2 à l’adresse IP Recherche cognitive.
1. [Associez App Service à la nouvelle ressource Recherche cognitive](#configure-qna-maker-to-use-different-cognitive-search-resource) créée à l’étape 2. Vous pouvez ensuite supprimer la ressource Recherche cognitive d’origine créée à l’étape 1.

Sur le portail [QnA Maker](https://www.qnamaker.ai/), créez votre première base de connaissances.


### <a name="inactivity-policy-for-free-search-resources"></a>Stratégie d’inactivité pour les ressources de recherche gratuites

Si vous n’utilisez pas de ressource QnA Maker, vous devez supprimer toutes les ressources. Si vous ne supprimez pas les ressources inutilisées, votre base de connaissances cessera de fonctionner si vous avez créé une ressource de recherche gratuite.

Les ressources de recherche gratuites sont supprimées après 90 jours sans recevoir d’appel d’API.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Si vous prévoyez de disposer de plusieurs bases de connaissances, mettez à niveau votre niveau tarifaire du service Recherche cognitive Azure.

Il n’est actuellement pas possible d’effectuer une mise à niveau sur place de la référence SKU de la recherche Azure. Toutefois, vous pouvez créer une ressource de recherche Azure avec la référence SKU souhaitée, restaurer les données vers la nouvelle ressource, puis la lier à la pile QnA Maker. Pour ce faire, procédez comme suit :

1. Créez une ressource de recherche Azure dans le portail Azure et sélectionnez la référence SKU souhaitée.

    ![Ressources de recherche QnA Maker Azure](../media/qnamaker-how-to-upgrade-qnamaker/qnamaker-azuresearch-new.png)

1. Restaurez les index de votre ressource de recherche Azure d’origine vers la nouvelle. Consultez [l’exemple de code de restauration de sauvegarde](https://github.com/pchoudhari/QnAMakerBackupRestore).

1. Pour lier la nouvelle ressource de recherche Azure au service QnA Maker managé (préversion), consultez la rubrique ci-dessous.

### <a name="configure-qna-maker-managed-preview-service-to-use-different-cognitive-search-resource"></a>Configurer le service QnA Maker managé (préversion) pour utiliser une autre ressource Recherche cognitive

Si vous créez un service QnA managé (préversion) et ses dépendances (par exemple, la recherche) par le biais du portail, un service de recherche est créé pour vous et lié au service QnA Maker managé (préversion). Une fois ces ressources créées, vous pouvez mettre à jour le service de recherche sous l’onglet **Configuration**.

1. Accédez à votre service QnA Maker managé (préversion) dans le portail Azure.

1. Sélectionnez **Configuration**, puis le service Recherche cognitive Azure à lier à votre service QnA Maker managé (préversion).

    ![Capture d’écran de la page de configuration de QnA Maker managé (préversion)](../media/qnamaker-how-to-upgrade-qnamaker/change-search-service-configuration.png)

1. Cliquez sur **Enregistrer**.

> [!NOTE]
> Si vous modifiez le service de recherche Azure associé à QnA Maker, vous allez perdre l’accès à toutes les bases de connaissances déjà présentes dans celui-ci. Veillez à exporter les bases de connaissances existantes avant de modifier le service de recherche Azure.
### <a name="inactivity-policy-for-free-search-resources"></a>Stratégie d’inactivité pour les ressources de recherche gratuites

Si vous n’utilisez pas de ressource QnA Maker, vous devez supprimer toutes les ressources. Si vous ne supprimez pas les ressources inutilisées, votre base de connaissances cessera de fonctionner si vous avez créé une ressource de recherche gratuite.

Les ressources de recherche gratuites sont supprimées après 90 jours sans recevoir d’appel d’API.

---

## <a name="delete-azure-resources"></a>Supprimer les ressources Azure

Si vous supprimez l’une des ressources Azure utilisées pour vos bases de connaissances QnA Maker, celles-ci ne fonctionneront plus. Avant de supprimer des ressources, veillez à exporter vos bases de connaissances à partir de la page **Paramètres**.

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur [App Service](../../../app-service/index.yml) et le [service de recherche](../../../search/index.yml).

> [!div class="nextstepaction"]
> [Découvrir comment créer en équipe](../index.yml)