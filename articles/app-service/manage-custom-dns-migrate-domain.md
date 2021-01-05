---
title: Migrer un nom DNS actif
description: Découvrez comment migrer sans temps d’arrêt vers Azure App Service un nom de domaine personnalisé déjà affecté à un site actif.
tags: top-support-issue
ms.assetid: 10da5b8a-1823-41a3-a2ff-a0717c2b5c2d
ms.topic: article
ms.date: 08/25/2020
ms.custom: seodec18
ms.openlocfilehash: e1b50675bef0f883ff617b3098a742d3491b3c13
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89484294"
---
# <a name="migrate-an-active-dns-name-to-azure-app-service"></a>Migrer un nom DNS actif vers Azure App Service

Cet article montre comment migrer un nom DNS actif vers [Azure App Service](../app-service/overview.md) sans temps d’arrêt.

Lorsque vous migrez un site actif et son nom de domaine DNS vers App Service, ce nom DNS sert déjà un trafic actif. Vous pouvez éviter tout temps d’arrêt dans la résolution DNS lors de la migration en liant à l’avance le nom DNS actif à votre application App Service.

Si vous ne vous inquiétez pas des temps d’arrêt durant la résolution DNS, voir [Mapper un nom DNS personnalisé existant vers Azure App Service](app-service-web-tutorial-custom-domain.md).

## <a name="prerequisites"></a>Prérequis

Pour suivre cette procédure :

- [Vérifiez que votre application App Service n’est pas au niveau Gratuit](app-service-web-tutorial-custom-domain.md#checkpricing).

## <a name="bind-the-domain-name-preemptively"></a>Lier le nom de domaine de manière préemptive

Lorsque vous liez un domaine personnalisé de manière préemptive, vous effectuez les deux opérations suivantes avant d’apporter des modifications à vos enregistrements DNS existants :

- Vérifier la propriété du domaine
- Activer le nom de domaine de votre application

Lorsque vous migrez finalement votre nom DNS personnalisé de l’ancien site à l’application App Service, il n’y a aucun temps d’arrêt durant la résolution DNS.

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records.md)]

### <a name="get-domain-verification-id"></a>Obtenir l’ID de vérification du domaine

Procurez-vous l’ID de vérification du domaine pour votre application en suivant les étapes décrites dans [Obtenir l’ID de vérification du domaine](app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id).

### <a name="create-domain-verification-record"></a>Créer un enregistrement de vérification de domaine

Pour vérifier la propriété du domaine, ajoutez un enregistrement TXT de vérification de domaine. Le nom d’hôte de l’enregistrement TXT dépend du type d’enregistrement DNS que vous souhaitez mapper. Consultez le tableau suivant (`@` représente généralement le domaine racine) :

| Exemple d’enregistrement DNS | Hôte TXT | Valeur TXT |
| - | - | - |
| \@ (racine) | _asuid_ | [ID de vérification du domaine pour votre application](app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id) |
| www (sous-domaine) | _asuid.www_ | [ID de vérification du domaine pour votre application](app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id) |
| \* (caractère générique) | _asuid_ | [ID de vérification du domaine pour votre application](app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id) |

Dans la page des enregistrements DNS, notez le type d’enregistrement du nom DNS que vous souhaitez migrer. App Service prend en charge les mappages d’enregistrements CNAME et A.

> [!NOTE]
> Les enregistrements `*` génériques ne valideront pas les sous-domaines avec un enregistrement CNAME existant. Vous devrez peut-être créer explicitement un enregistrement TXT pour chaque sous-domaine.

### <a name="enable-the-domain-for-your-app"></a>Activer le domaine pour votre application

1. Dans le [portail Azure](https://portal.azure.com), dans le volet de navigation gauche de la page de l’application, sélectionnez **Domaines personnalisés**. 

    ![Menu Domaines personnalisés](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. Dans la page **Domaines personnalisés**, sélectionnez **Ajouter un domaine personnalisé**.

    ![Ajouter un nom d’hôte](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Tapez le nom de domaine complet que vous souhaitez migrer, qui correspond à l’enregistrement TXT que vous créez, par exemple `contoso.com`, `www.contoso.com`ou `*.contoso.com`. Sélectionnez **Valider**.

    Le bouton **Ajouter un domaine personnalisé** est activé. 

1. Assurez-vous que l’option **Type d’enregistrement du nom d’hôte** est bien définie sur le type d’enregistrement DNS que vous voulez migrer. Sélectionnez **Ajouter un nom d’hôte**.

    ![Ajouter un nom DNS à l’application](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

    Un certain temps peut être nécessaire pour que le nouveau nom d’hôte soit reflété sur la page **Domaines personnalisés** de votre application. Essayez d’actualiser le navigateur pour mettre à jour les données.

    ![Enregistrement CNAME ajouté](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

    Votre nom DNS personnalisé est à présent activé dans votre application Azure. 

## <a name="remap-the-active-dns-name"></a>Remapper le nom DNS actif

Il ne vous reste plus qu’à remapper votre enregistrement DNS actif pour qu’il pointe vers App Service. En ce moment précis, il pointe encore sur votre ancien site.

<a name="info"></a>

### <a name="copy-the-apps-ip-address-a-record-only"></a>Copier l’adresse IP de l’application (un enregistrement uniquement)

Si vous remappez un enregistrement CNAME, ignorez cette section. 

Pour remapper un enregistrement A, vous avez besoin de l’adresse IP externe de l’application App Service. Celle-ci est affichée dans la page **Domaines personnalisés**.

Dans la page **Domaines personnalisés**, copiez l’adresse IP de l’application.

![Navigation au sein du portail pour accéder à l’application Azure](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

### <a name="update-the-dns-record"></a>Mettre à jour l’enregistrement DNS

Dans la page des enregistrements DNS de votre fournisseur de domaine, sélectionnez l’enregistrement DNS à remapper.

Pour l’exemple de domaine racine `contoso.com`, remappez l’enregistrement A ou CNAME, comme dans les exemples fournis dans le tableau suivant : 

| Exemple de nom de domaine complet | Type d’enregistrement | Host | Valeur |
| - | - | - | - |
| contoso.com (racine) | Un | `@` | Adresse IP de [Copier l’adresse IP de l’application](#info) |
| www\.contoso.com (sub) | CNAME | `www` | _&lt;nomapplication&gt;.azurewebsites.net_ |
| \*.contoso.com (caractère générique) | CNAME | _\*_ | _&lt;nomapplication&gt;.azurewebsites.net_ |

Enregistrez vos paramètres.

Les requêtes DNS doivent commencer à trouver votre application App Service immédiatement après la propagation DNS.

## <a name="migrate-domain-from-another-app"></a>Migrer un domaine à partir d’une autre application

Vous pouvez migrer un domaine personnalisé actif dans Azure, entre des abonnements ou au sein d’un même abonnement. Toutefois, une telle migration sans temps d’arrêt nécessite que l’application source et l’application cible reçoivent le même domaine personnalisé à un moment donné. Par conséquent, vous devez vous assurer que les deux applications ne sont pas déployées sur la même unité de déploiement (en interne appelée espace web). Un nom de domaine ne peut être attribué qu’à une seule application dans chaque unité de déploiement.

Vous pouvez trouver l’unité de déploiement de votre application en examinant le nom de domaine de l’URL FTP/S`<deployment-unit>.ftp.azurewebsites.windows.net`. Vérifiez et assurez-vous que l’unité de déploiement est différente entre l’application source et l’application cible. L’unité de déploiement d’une application est déterminée par le [plan App Service](overview-hosting-plans.md) à l’intérieur. Elle est sélectionnée de façon aléatoire par Azure lorsque vous créez le plan et ne peut pas être modifiée. Azure s’assure uniquement que deux plans se trouvent dans la même unité de déploiement lorsque vous [les créer dans le même groupe de ressources *et* la même région](app-service-plan-manage.md#create-an-app-service-plan), mais n’a pas de logique pour s’assurer que les plans se trouvent dans des unités de déploiement différentes. La seule façon de créer un plan dans une autre unité de déploiement consiste à continuer à créer un plan dans un nouveau groupe de ressources ou une nouvelle région jusqu’à ce que vous obteniez une unité de déploiement différente.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment lier un certificat TLS/SSL personnalisé à App Service.

> [!div class="nextstepaction"]
> [Sécuriser un nom DNS personnalisé avec une liaison TLS dans Azure App Service](configure-ssl-bindings.md)
