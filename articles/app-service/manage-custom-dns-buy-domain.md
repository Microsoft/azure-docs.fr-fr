---
title: Acheter un nom de domaine personnalisé
description: Découvrez comment acheter un domaine App Service et comment vous en servir de domaine personnalisé pour votre application Azure App Service.
ms.assetid: 70fb0e6e-8727-4cca-ba82-98a4d21586ff
ms.topic: article
ms.date: 11/30/2020
ms.custom: seodec18
ms.openlocfilehash: 6f0ff7a54c2ad1fa1af649c8082498b442783c7e
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608075"
---
# <a name="buy-a-custom-domain-name-for-azure-app-service"></a>Acheter un nom de domaine personnalisé pour Azure App Service

Les domaines App Service sont des domaines personnalisés gérés directement dans Azure. Ils facilitent la gestion des domaines personnalisés pour [Azure App Service](overview.md). Ce tutoriel explique comment acheter un domaine App Service et attribuer des noms DNS dans Azure App Service.

Pour la machine virtuelle Azure ou le stockage Azure, consultez [Assign App Service domain to Azure VM or Azure Storage](https://azure.github.io/AppService/2017/07/31/Assign-App-Service-domain-to-Azure-VM-or-Azure-Storage) (Attribuer un domaine App Service à une machine virtuelle Azure ou un stockage Azure). Pour Services cloud, consultez [Configuration d’un nom de domaine personnalisé pour un service cloud Azure](../cloud-services/cloud-services-custom-domain-name-portal.md).

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

* [Créez une application App Service](./index.yml), ou utilisez une application créée pour un autre didacticiel. L’application doit se trouver dans une région publique Azure. À ce stade, les clouds nationaux Azure ne sont pas pris en charge.
* [Supprimez la limite de dépense de votre abonnement](../cost-management-billing/manage/spending-limit.md#remove). Vous ne pouvez pas acheter des domaines App Service avec des crédits d'abonnement gratuits.

## <a name="buy-an-app-service-domain"></a>Acheter un domaine App Service

Pour obtenir des informations de tarification sur les domaines App Service, rendez-vous sur la [page Tarification App Service](https://azure.microsoft.com/pricing/details/app-service/windows/) et faites défiler la page jusqu’à la section Domaine App Service.

1. Ouvrez le [portail Azure](https://portal.azure.com) et connectez-vous avec votre compte Azure.

1. Dans la barre de recherche, recherchez et sélectionnez **Domaines App Service**.

    ![Navigation au sein du portail pour accéder aux domaines App Service](./media/app-service-web-tutorial-custom-domain/view-app-service-domains.png)

1. Dans la vue **Domaines App Service**, cliquez sur **Ajouter**.

    ![Cliquer sur Ajouter dans Domaines App Service](./media/app-service-web-tutorial-custom-domain/add-app-service-domain.png)

1. Sélectionnez **Cliquez pour essayer la version plus récente de l’expérience de création de domaines App Service**.

    ![Créer un domaine App Service à l’aide de la nouvelle expérience](./media/app-service-web-tutorial-custom-domain/select-new-create-experience.png)

### <a name="basics-tab"></a>Onglet Informations de base

1. Dans l’onglet **Informations de base**, configurez les paramètres à l’aide du tableau suivant :  

   | Paramètre  | Description |
   | -------- | ----------- |
   | **Abonnement** | Abonnement à utiliser pour acheter le domaine. |
   | **Groupe de ressources** | Groupe de ressources dans lequel placer le domaine. Par exemple, le groupe de ressources dans lequel se trouve votre application. |
   | **Domaine** | Saisissez le domaine de votre choix. Par exemple, **contoso.com**. Si le domaine souhaité n’est pas disponible, vous pouvez effectuer votre choix parmi une liste de suggestions de domaines disponibles ou essayer un autre domaine. |

    > [!NOTE]
    > Les [domaines de niveau supérieur](https://wikipedia.org/wiki/Top-level_domain) suivants sont pris en charge par les domaines App Service : _com_, _net_, _co.uk_, _org_, _nl_, _in_, _biz_, _org.uk_ et _co.in_.
    >
    >
    
2. Une fois que vous avez terminé, cliquez sur **Suivant : Coordonnées**.

### <a name="contact-information-tab"></a>Onglet Coordonnées

1. Fournissez les informations requises par l’[ICANN](https://go.microsoft.com/fwlink/?linkid=2116641) pour inscrire le domaine. 

    Il est très important de remplir tous les champs obligatoires aussi précisément que possible. L’inexactitude des coordonnées fournies peut empêcher l’achat du domaine.

1. Une fois que vous avez terminé, cliquez sur **Suivant : Avancé**.

### <a name="advanced-tab"></a>Onglet Avancé

1. Dans l’onglet **Avancé**, configurez les paramètres facultatifs :  

   | Paramètre  | Description |
   | -------- | ----------- |
   | **Renouvellement automatique** | Option activée par défaut. Votre domaine App Service est inscrit sous votre nom avec des incréments d’une année. Le renouvellement automatique garantit que l’inscription de votre domaine n’expire pas et que vous restez propriétaire du domaine. Les frais annuels d’inscription du domaine sont automatiquement facturés sur votre abonnement Azure au moment du renouvellement. Pour vous désinscrire, sélectionnez **Désactiver**. Si le renouvellement automatique est désactivé, vous pouvez [le renouveler manuellement](#renew-the-domain). |
   | **Protection des données personnelles** | Option activée par défaut. La protection des données personnelles masque les coordonnées d’inscription du domaine dans la base de données WHOIS. La protection des données personnelles est déjà incluse dans les frais annuels d’inscription du domaine. Pour vous désinscrire, sélectionnez **Désactiver**. |

2. Une fois que vous avez terminé, cliquez sur **Suivant : Balises**.

### <a name="finish"></a>Finish

1. Dans l’onglet **Balises**, définissez les balises souhaitées pour votre domaine App Service. Le balisage n’est pas obligatoire pour l’utilisation des domaines App Service, mais il s’agit d’une [fonctionnalité Azure qui vous aide à gérer vos ressources](../azure-resource-manager/management/tag-resources.md).

1. Cliquez sur **Suivant : Vérifier + créer**.

1. Dans l’onglet **Vérifier + créer**, examinez votre commande de domaine. Lorsque vous avez terminé, cliquez sur **Créer**.

    > [!NOTE]
    > Les domaines App Service utilisent GoDaddy pour l’inscription du domaine, et Azure DNS pour l’hébergement. Outre les frais annuels d’inscription de domaine, des frais d’utilisation d’Azure DNS s’appliquent. Pour en savoir plus, consultez la page relative à la [tarification Azure DNS](https://azure.microsoft.com/pricing/details/dns/).
    >
    >

1. Une fois l’inscription du domaine terminée, vous voyez un bouton **Accéder à la ressource**. Sélectionnez-le pour afficher sa page de gestion.

    ![Domaine App Service créé. Accéder à la ressource](./media/app-service-web-tutorial-custom-domain/deployment-complete.png)

Vous êtes maintenant prêt à affecter une application App Service à ce domaine personnalisé.

## <a name="prepare-the-app"></a>Préparer l’application

Pour mapper un nom DNS personnalisé à une application web, le [plan App Service](https://azure.microsoft.com/pricing/details/app-service/) de l’application web doit correspondre à un niveau payant (Partagé, De base, Standard, Premium ou Consommation pour Azure Functions). Au cours de cette étape, vous allez vous assurer que l’application App Service se trouve dans le niveau de tarification pris en charge.

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

### <a name="navigate-to-the-app-in-the-azure-portal"></a>Accéder à l’application dans le portail Azure

1. Dans la barre de recherche du haut, recherchez et sélectionnez **App Services**.

    ![Rechercher App Services](./media/app-service-web-tutorial-custom-domain/app-services.png)

1. Sélectionnez le nom de l’application.

    ![Navigation au sein du portail pour accéder à l’application Azure](./media/app-service-web-tutorial-custom-domain/select-app.png)

    La page de gestion de l’application App Service s’affiche.  

### <a name="check-the-pricing-tier"></a>Vérification du niveau tarifaire

1. Dans la navigation gauche de la page de l’application, faites défiler jusqu’à la section **Paramètres** et sélectionnez **Monter en puissance (plan App Service)** .

    ![Menu Monter en puissance](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

1. Le niveau actuel de l’application est encadré d’un rectangle bleu. Vérifiez que l’application ne se trouve pas dans le niveau **F1**. Les DNS personnalisés ne sont pas disponibles dans le niveau **F1**. 

    :::image type="content" source="./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png" alt-text="Capture d’écran montrant le menu de navigation gauche de la page d’application avec l’option Monter en puissance (plan App Service) sélectionnée.":::

1. Si le plan App Service n’est pas dans le niveau **F1**, fermez la page **Monter en puissance** et passez à [Acheter le domaine](#buy-an-app-service-domain).

### <a name="scale-up-the-app-service-plan"></a>Monter en puissance le plan App Service

1. Sélectionnez un niveau payant (**D1**, **B1**, **B2**, **B3** ou n’importe quel niveau dans la catégorie **Production**). Pour obtenir des options supplémentaires, cliquez sur **Afficher d’autres options**.

1. Cliquez sur **Appliquer**.

    :::image type="content" source="./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png" alt-text="Capture d’écran montrant les niveaux tarifaires de domaine personnalisés dans la catégorie Production avec l’onglet Production, le plan B1 et le bouton Appliquer en surbrillance.":::

    Lorsque la notification suivante s’affiche, cela signifie que l’opération est terminée.

    ![Confirmation d’opération de mise à l’échelle](./media/app-service-web-tutorial-custom-domain/scale-notification.png)

## <a name="map-app-service-domain-to-your-app"></a>Mapper le domaine App Service à votre application

Il est facile de mapper un nom d’hôte dans votre domaine App Service à une application App Service, à condition qu’il se trouve dans le même abonnement. Vous mappez le domaine App Service ou l’un de ses sous-domaines directement dans votre application, et Azure crée pour vous les enregistrements DNS nécessaires.

> [!NOTE]
> Si le domaine et l’application se trouvent dans des abonnements différents, mappez le domaine App Service à l’application comme vous le feriez en [mappant un domaine acheté en externe](app-service-web-tutorial-custom-domain.md). Dans ce cas, Azure DNS est le fournisseur de domaine externe, et vous devez [ajouter manuellement les enregistrements DNS requis](#manage-custom-dns-records).
>

### <a name="map-the-domain"></a>Mappage du domaine

1. Dans la navigation gauche de la page de l’application, faites défiler jusqu’à la section **Paramètres** et sélectionnez **Domaines personnalisés**.

    ![Capture d’écran montrant le menu Domaines personnalisés.](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. Sélectionnez **Ajouter un domaine personnalisé**.

    ![Capture d’écran montrant l’élément Ajouter un nom d’hôte.](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Saisissez le domaine App Service (p. ex., **contoso.com**) ou un sous-domaine (p. ex., **www.contoso.com**), puis cliquez sur **Valider**.

    > [!NOTE]
    > Si vous avez fait une faute de frappe dans le nom du domaine App Service, une erreur de vérification apparaît en bas de page pour vous indiquer qu’il vous manque certains enregistrements DNS. Vous n’avez pas besoin d’ajouter ces enregistrements manuellement pour un domaine App Service. Veillez simplement à saisir correctement le nom de domaine, puis cliquez à nouveau sur **Valider**.
    >
    > ![Capture d’écran montrant une erreur de vérification.](./media/app-service-web-tutorial-custom-domain/verification-error-cname.png)

1. Acceptez le **type d’enregistrement du nom d’hôte** et cliquez sur **Ajouter un domaine personnalisé**.

    ![Capture d’écran montrant le bouton Ajouter un domaine personnalisé.](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

1. Un certain temps peut être nécessaire pour que le nouveau domaine personnalisé s’affiche dans la page **Domaines personnalisés** de votre application. Actualisez le navigateur pour mettre à jour les données.

    ![Capture d’écran montrant l’ajout de l’enregistrement CNAME.](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

    > [!NOTE]
    > Si votre domaine personnalisé présente l’étiquette **Non sécurisé**, cela signifie qu’il n’est pas encore lié à un certificat TLS/SSL. Toute requête HTTPS envoyée par un navigateur à votre domaine personnalisé générera une erreur ou un avertissement, selon le navigateur utilisé. Pour ajouter une liaison TLS/SSL, consultez [Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service](configure-ssl-bindings.md).
    
### <a name="test-the-custom-domain"></a>Test du domaine personnalisé

Pour tester le domaine personnalisé, accédez à celui-ci dans le navigateur.

## <a name="renew-the-domain"></a>Renouveler le domaine

Le domaine App Service que vous avez acheté est valable un an à compter de la date d'achat. Par défaut, le domaine est configuré pour se renouveler automatiquement, avec une facturation selon votre mode de paiement, pour l'année suivante. Vous pouvez renouveler manuellement votre nom de domaine.

Si vous souhaitez désactiver le renouvellement automatique ou renouveler manuellement votre domaine, suivez les étapes décrites ici.

1. Dans la barre de recherche, recherchez et sélectionnez **Domaines App Service**.

    ![Navigation au sein du portail pour accéder aux domaines App Service](./media/app-service-web-tutorial-custom-domain/view-app-service-domains.png)

1. Dans la section **Domaines App Service**, sélectionnez le domaine que vous souhaitez configurer.

1. Dans la barre de navigation à gauche du domaine, sélectionnez **Renouvellement de domaine**. Pour mettre fin au renouvellement automatique de votre domaine, sélectionnez **Désactivé**. Le paramètre prend immédiatement effet.

    ![Capture d’écran montrant l’option de renouvellement automatique de votre domaine.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-autorenew.png)

    > [!NOTE]
    > Quand vous quittez la page, ignorez l’erreur « Vos modifications non enregistrées seront ignorées » en cliquant sur **OK**.
    >

Pour renouveler manuellement votre domaine, sélectionnez **Renouveler le domaine**. Cependant, ce bouton n'est actif que [90 jours avant l'expiration du domaine](#when-domain-expires).

Si le renouvellement de votre domaine est réussi, vous recevrez une notification par e-mail dans les 24 heures.

## <a name="when-domain-expires"></a>Lorsque le domaine expire

Azure traite les domaines App Service expirés ou sur le point d’expirer comme suit :

* Si le renouvellement automatique est désactivé : 90 jours avant l’expiration de domaine, une notification par e-mail est envoyée et le bouton **Renouveler le domaine** est activé dans le portail.
* Si le renouvellement automatique est activé : Le jour après la date d’expiration de votre domaine, Azure tente de vous facturer pour le renouvellement du nom de domaine.
* Si une erreur se produit pendant le renouvellement automatique (par exemple, si votre carte enregistrée est arrivée à expiration), ou si le renouvellement automatique est désactivé et que vous autorisez l’expiration du domaine, Azure vous avertit de l’expiration du domaine et réserve votre nom de domaine. Vous pouvez [renouveler manuellement](#renew-the-domain) votre domaine.
* Les 4 et 12e jours après expiration, Azure vous envoie des e-mails de notification supplémentaires. Vous pouvez [renouveler manuellement](#renew-the-domain) votre domaine.
* Le 19e jour après expiration, votre domaine reste réservé, mais fait alors l’objet de frais d’échange. Vous pouvez appeler le support technique pour renouveler votre nom de domaine, soumis aux frais de renouvellement et d’échange applicables.
* Le 25e jour après expiration, Azure met votre domaine aux enchères avec un service de vente aux enchères de noms de domaine du secteur. Vous pouvez appeler le support technique pour renouveler votre nom de domaine, soumis aux frais de renouvellement et d’échange applicables.
* Le 30e jour après expiration, vous n’êtes plus en mesure de récupérer votre domaine.

<a name="custom"></a>

## <a name="manage-custom-dns-records"></a>Gérer des enregistrements DNS personnalisés

Dans Azure, les enregistrements DNS d’un domaine App Service sont gérés à l’aide d’[Azure DNS](https://azure.microsoft.com/services/dns/). Vous pouvez ajouter, supprimer et mettre à jour les enregistrements DNS, comme pour un domaine acheté en externe. Pour gérer les enregistrements DNS personnalisés :

1. Dans la barre de recherche, recherchez et sélectionnez **Domaines App Service**.

    ![Navigation au sein du portail pour accéder aux domaines App Service](./media/app-service-web-tutorial-custom-domain/view-app-service-domains.png)

1. Dans la section **Domaines App Service**, sélectionnez le domaine que vous souhaitez configurer.

1. À partir de la page **Vue d’ensemble**, sélectionnez **Gérer les enregistrements DNS**.

    ![Capture d’écran montrant où accéder aux enregistrements DNS.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-dns-zone.png)

Pour savoir comment modifier des enregistrements DNS, consultez [Gérer des zones DNS dans le portail Azure](../dns/dns-operations-dnszones-portal.md).

## <a name="cancel-purchase-delete-domain"></a>Annuler l’achat (supprimer le domaine)

Après avoir acheté le domaine App Service, vous disposez de cinq jours pour annuler votre achat et bénéficier d’un remboursement intégral. Au-delà de cinq jours, vous pouvez supprimer le domaine App Service, mais vous ne serez pas remboursé.

1. Dans la barre de recherche, recherchez et sélectionnez **Domaines App Service**.

    ![Navigation au sein du portail pour accéder aux domaines App Service](./media/app-service-web-tutorial-custom-domain/view-app-service-domains.png)

1. Dans la section **Domaines App Service**, sélectionnez le domaine que vous souhaitez configurer.

1. Dans le menu de navigation gauche du domaine, sélectionnez **Liaisons de noms d’hôte**. Les liaisons de nom d’hôte de tous les services Azure sont répertoriées ici.

    ![Capture d’écran montrant la page de liaisons de nom d’hôte.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostname-bindings.png)

1. Supprimez chaque liaison de nom d’hôte en sélectionnant **...**  > **Supprimer**. Après avoir supprimé toutes les liaisons, sélectionnez **Enregistrer**.

    <!-- ![Screenshot that shows where to delete the hostname bindings.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-delete-hostname-bindings.png) -->

1. Dans le menu de navigation gauche du domaine, sélectionnez **Vue d’ensemble**. 

1. Si la période d’annulation pour domaine acheté n’est pas écoulée, sélectionnez **Annuler l’achat**. Dans le cas contraire, vous verrez le bouton **Supprimer**. Pour supprimer le domaine sans remboursement, sélectionnez **Supprimer**.

    ![Capture d’écran montrant où supprimer ou annuler un domaine acheté.](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-cancel.png)

1. Pour confirmer l’opération, sélectionnez **Oui**.

    Une fois l’opération terminée, le domaine n’est plus lié à votre abonnement et est de nouveau disponible à l’achat. 

## <a name="direct-default-url-to-a-custom-directory"></a>Diriger l'URL par défaut vers un répertoire personnalisé

Par défaut, App Service dirige les demandes web au répertoire racine du code de votre application. Pour les diriger vers un sous-répertoire, par exemple `public`, consultez [Rediriger vers un répertoire personnalisé](app-service-web-tutorial-custom-domain.md#redirect-to-a-custom-directory).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment lier un certificat TLS/SSL personnalisé à App Service.

> [!div class="nextstepaction"]
> [Sécuriser un nom DNS personnalisé avec une liaison TLS dans Azure App Service](configure-ssl-bindings.md)
