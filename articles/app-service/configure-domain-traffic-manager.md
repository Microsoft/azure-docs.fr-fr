---
title: Configurer des noms DNS avec Traffic Manager
description: Découvrez comment configurer un domaine personnalisé pour une application Azure App Service qui s’intègre à Traffic Manager pour l’équilibrage de charge.
ms.assetid: 0f96c0e7-0901-489b-a95a-e3b66ca0a1c2
ms.topic: article
ms.date: 03/05/2020
ms.custom: seodec18
ms.openlocfilehash: 0e8d5fa14678a2a26234dfcd73f4a50af62ca7aa
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012939"
---
# <a name="configure-a-custom-domain-name-in-azure-app-service-with-traffic-manager-integration"></a>Configurer un nom de domaine personnalisé dans Azure App Service avec l’intégration de Traffic Manager

[!INCLUDE [web-selector](../../includes/websites-custom-domain-selector.md)]

> [!NOTE]
> Pour Services cloud, consultez [Configuration d’un nom de domaine personnalisé pour un service cloud Azure](../cloud-services/cloud-services-custom-domain-name-portal.md).

Lorsque vous utilisez [Azure Traffic Manager](../traffic-manager/index.yml) pour équilibrer la charge du trafic vers [Azure App service](overview.md), l’application App Service est accessible à l’aide de **\<traffic-manager-endpoint>.trafficmanager.net**. Afin de proposer un nom de domaine encore plus reconnaissable pour vos utilisateurs, vous pouvez attribuer un nom de domaine personnalisé, tel que www\.contoso.com, avec votre application App Service.

Cet article explique comment configurer un nom de domaine personnalisé avec une application App Service qui est intégrée à [Traffic Manager](../traffic-manager/traffic-manager-overview.md).

> [!NOTE]
> Seuls les enregistrements [CNAME](https://en.wikipedia.org/wiki/CNAME_record) sont pris en charge lorsque vous configurez un nom de domaine à l’aide du point de terminaison Traffic Manager. Comme les enregistrements A ne sont pas pris en charge, un mappage de domaine racine, tel que contoso.com, n’est pas non plus pris en charge.
> 

## <a name="prepare-the-app"></a>Préparer l’application

Pour mapper un nom DNS personnalisé à une application intégrée à Azure Traffic Manager, le [plan App Service](https://azure.microsoft.com/pricing/details/app-service/) de l’application web doit être au niveau **Standard** ou supérieur. Au cours de cette étape, vous allez vous assurer que l’application App Service se trouve dans le niveau de tarification pris en charge.

### <a name="check-the-pricing-tier"></a>Vérification du niveau tarifaire

Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **App Services**.

Dans la page **App Services**, sélectionnez le nom de votre application Azure.

![Navigation au sein du portail pour accéder à l’application Azure](./media/app-service-web-tutorial-custom-domain/select-app.png)

Dans le volet de navigation de gauche de la page de l’application, sélectionnez **Effectuer un scale-up (plan App Service)** .

![Menu Monter en puissance](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

Le niveau actuel de l’application est encadré d’un rectangle bleu. Assurez-vous que l’application est bien au niveau **Standard** ou supérieur (n’importe quel niveau dans la catégorie **Production** ou **Isolé**). Si c’est le cas, fermez la page **Scale-up** et passez à [Créer le mappage CNAME](#create-the-cname-mapping).

![Vérification du niveau de tarification](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

### <a name="scale-up-the-app-service-plan"></a>Monter en puissance le plan App Service

Si vous avez besoin d’effectuer un scale-up de votre application, sélectionnez un des niveaux tarifaires dans la catégorie **Production**. Pour obtenir des options supplémentaires, cliquez sur **Afficher d’autres options**.

Cliquez sur **Appliquer**.

## <a name="create-traffic-manager-endpoint"></a>Créer le point de terminaison Traffic Manager

En suivant les étapes de la rubrique [Ajout ou suppression de points de terminaison](../traffic-manager/traffic-manager-manage-endpoints.md), ajoutez votre application App Service comme point de terminaison dans votre profil Traffic Manager.

Une fois votre application App Service associé à un niveau tarifaire pris en charge, elle apparaît dans la liste des cibles App Service disponibles lorsque vous ajoutez le point de terminaison. Si votre application n’est pas listée, pensez à [vérifier le niveau tarifaire de votre application](#prepare-the-app).

## <a name="create-the-cname-mapping"></a>Créer le mappage CNAME
> [!NOTE]
> Pour configurer un [domaine App Service que vous avez acheté](manage-custom-dns-buy-domain.md), ignorez cette section et consultez [Activer un domaine personnalisé](#enable-custom-domain).
> 

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

Bien que les spécificités de chaque fournisseur de domaine varient, vous mappez généralement *depuis* un [nom de domaine personnalisé non-racine](#what-about-root-domains) (comme **www.contoso.com**) *vers* le nom de domaine Traffic Manager (**contoso.trafficmanager.net**) qui est intégré à votre application. 

> [!NOTE]
> Si un enregistrement est déjà utilisé et que vous devez y lier préventivement vos applications, vous pouvez créer un enregistrement CNAME supplémentaire. Par exemple, pour lier préalablement **www\.contoso.com** à votre application, créez un enregistrement CNAME entre **awverify.www** et **contoso.trafficmanager.net**. Vous pouvez ensuite ajouter « www\.contoso.com » à votre application sans avoir besoin de modifier l’enregistrement CNAME « www ». Pour plus d’informations, consultez [Migrer un nom DNS actif vers Azure App Service](manage-custom-dns-migrate-domain.md).

Quand vous avez terminé l’ajout ou la modification des enregistrements DNS auprès de votre fournisseur de domaine, enregistrez les modifications.

### <a name="what-about-root-domains"></a>Qu’en est-il des domaines racine ?

Étant donné que Traffic Manager prend seulement en charge le mappage de domaine personnalisé avec des enregistrements CNAME, et que les normes DNS ne prennent pas en charge les enregistrements CNAME pour le mappage des domaines racine (par exemple, **contoso.com**), Traffic Manager ne gère pas le mappage aux domaines racine. Pour contourner ce problème, utilisez une redirection d’URL au niveau de l’application. Dans ASP.NET Core, par exemple, vous pouvez utiliser la [Réécriture d’URL](/aspnet/core/fundamentals/url-rewriting). Ensuite, utilisez Traffic Manager pour équilibrer la charge du sous-domaine (**www.contoso.com**).

Pour les scénarios de haute disponibilité, vous pouvez implémenter une configuration DNS tolérante aux pannes, sans Traffic Manager, en créant plusieurs *enregistrements A* qui pointent du domaine racine vers l’adresse IP de chaque copie de l’application. Ensuite, [mappez le même domaine racine à toutes les copies de l’application](app-service-web-tutorial-custom-domain.md#map-an-a-record). Comme le même nom de domaine ne peut pas être mappé vers deux applications différentes dans la même région, cette configuration fonctionne seulement si les copies de votre application se trouvent dans des régions différentes.

## <a name="enable-custom-domain"></a>Activer un domaine personnalisé
Une fois les enregistrements de votre nom de domaine propagés, utilisez le navigateur pour vérifier que votre nom de domaine personnalisé correspond à votre application App Service.

> [!NOTE]
> La propagation de l’enregistrement CNAME dans le système DNS peut prendre du temps. Vous pouvez utiliser un service tel que <a href="https://www.digwebinterface.com/">https://www.digwebinterface.com/</a> pour vérifier que l'enregistrement CNAME est disponible.
> 
> 

1. Une fois la résolution de domaine réussie, revenez à la page de votre application dans le [portail Azure](https://portal.azure.com).
2. Dans le volet de navigation de gauche, sélectionnez **Domaines personnalisés** > **Ajouter un nom d’hôte**.
4. Tapez le nom de domaine personnalisé que vous avez mappé précédemment et sélectionnez **Valider**.
5. Assurez-vous que **Type d’enregistrement du nom d’hôte** est défini sur **CNAME (www\.exemple.com ou tout sous-domaine)** .

6. Étant donné que l’application App Service est désormais intégrée à un point de terminaison Traffic Manager, vous devez voir le nom de domaine Traffic Manager sous **Configuration CNAME**. Sélectionnez-le et cliquez sur **Ajouter un domaine personnalisé**.

    ![Ajouter un nom DNS à l’application](./media/configure-domain-traffic-manager/enable-traffic-manager-domain.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Sécuriser un nom DNS personnalisé avec une liaison SSL dans Azure App Service](configure-ssl-bindings.md)