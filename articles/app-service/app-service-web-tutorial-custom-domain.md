---
title: 'Tutoriel : Mapper un nom DNS personnalisé existant'
description: Découvrez comment ajouter un nom de domaine DNS (domaine personnel) à une application web, au back-end d’une application mobile ou à une application API dans Azure App Service.
keywords: app service, azure app service, mappage de domaine, nom de domaine, domaine existant, nom d'hôte
ms.assetid: dc446e0e-0958-48ea-8d99-441d2b947a7c
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 08/25/2020
ms.custom: mvc, seodec18
ms.openlocfilehash: b45e1fbaf912cc045ba51a79db434baecbabdf43
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608263"
---
# <a name="tutorial-map-an-existing-custom-dns-name-to-azure-app-service"></a>Tutoriel : Mapper un nom DNS personnalisé existant à Azure App Service

[Azure App Service](overview.md) offre un service d’hébergement web hautement évolutif appliquant des mises à jour correctives automatiques. Ce tutoriel vous montre comment mapper un nom DNS personnalisé existant à App Service.

![Capture d’écran montrant la navigation du portail Azure vers une application Azure.](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Mapper un sous-domaine (par exemple, `www.contoso.com`) à l’aide d’un enregistrement CNAME
> * Mapper un domaine racine (par exemple, `contoso.com`) à l’aide d’un enregistrement A
> * Mapper un domaine générique (par exemple, `*.contoso.com`) à l’aide d’un enregistrement CNAME
> * Rediriger l’URL par défaut vers un répertoire personnalisé
> * Automatiser le mappage de domaine à l’aide de scripts

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

* [Créez une application App Service](./index.yml), ou utilisez une application créée pour un autre didacticiel.
* Achetez un nom de domaine et vérifiez que vous avez accès au registre DNS pour votre fournisseur de domaines (par exemple, GoDaddy).

  Par exemple, pour ajouter des entrées DNS pour `contoso.com` et `www.contoso.com`, vous devez pouvoir configurer les paramètres DNS du domaine racine `contoso.com`.

  > [!NOTE]
  > Si vous ne disposez pas encore d’un nom de domaine, vous pouvez [acheter un domaine à l’aide du portail Azure](manage-custom-dns-buy-domain.md).

## <a name="prepare-the-app"></a>Préparer l’application

Pour mapper un nom DNS personnalisé à une application web, le [plan App Service](https://azure.microsoft.com/pricing/details/app-service/) de l’application web doit correspondre à un niveau payant (Partagé, De base, Standard, Premium ou Consommation pour Azure Functions). Au cours de cette étape, vous allez vous assurer que l’application App Service se trouve dans le niveau de tarification pris en charge.

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

### <a name="sign-in-to-azure"></a>Connexion à Azure

Ouvrez le [portail Azure](https://portal.azure.com) et connectez-vous avec votre compte Azure.

### <a name="select-the-app-in-the-azure-portal"></a>Sélectionner l’application dans le portail Azure

1. Recherchez et sélectionnez **App Services**.

   ![Capture d’écran montrant la sélection d’App Services.](./media/app-service-web-tutorial-custom-domain/app-services.png)

1. Dans la page **App Services**, sélectionnez le nom de votre application Azure.

   ![Capture d’écran montrant la navigation dans le portail vers une application Azure.](./media/app-service-web-tutorial-custom-domain/select-app.png)

La page de gestion de l’application App Service s’affiche.

<a name="checkpricing" aria-hidden="true"></a>

### <a name="check-the-pricing-tier"></a>Vérification du niveau tarifaire

1. Dans le volet gauche de la page de l’application, faites défiler jusqu’à la section **Paramètres**, puis sélectionnez **Effectuer un scale-up (plan App Service)** .

   ![Capture d’écran montrant le menu Effectuer un scale-up (plan App Service).](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

1. Le niveau actuel de l’application est encadré d’un rectangle bleu. Vérifiez que l’application ne se trouve pas dans le niveau **F1**. Les DNS personnalisés ne sont pas disponibles dans le niveau **F1**.

   ![Capture d’écran montrant les niveaux tarifaires recommandés.](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

1. Si le plan App Service n’est pas **F1**, fermez la page **Effectuer un scale-up**, puis passez à [Mapper un enregistrement CNAME](#map-a-cname-record).

<a name="scaleup" aria-hidden="true"></a>

### <a name="scale-up-the-app-service-plan"></a>Monter en puissance le plan App Service

1. Sélectionnez un niveau payant (**D1**, **B1**, **B2**, **B3** ou n’importe quel niveau dans la catégorie **Production**). Pour obtenir des options supplémentaires, sélectionnez **Afficher d’autres options**.

1. Sélectionnez **Appliquer**.

   ![Capture d’écran montrant la vérification du niveau tarifaire.](./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png)

   Lorsque la notification suivante s’affiche, cela signifie que l’opération est terminée.

   ![Capture d’écran montrant la confirmation de l’opération de mise à l’échelle](./media/app-service-web-tutorial-custom-domain/scale-notification.png)

<a name="cname" aria-hidden="true"></a>

## <a name="get-a-domain-verification-id"></a>Obtenir l’ID de vérification d’un domaine

Pour ajouter un domaine personnalisé à votre application, vous devez vérifier votre propriété du domaine, en ajoutant un ID de vérification sous forme d’enregistrement TXT avec votre fournisseur de domaine. Dans le volet gauche de la page de votre application, sélectionnez **Domaines personnalisés**. Pour l’étape suivante, copiez l’ID du champ **ID de vérification des domaines personnalisés** dans la page **Domaines personnalisés**.

![Capture d’écran montrant l’ID dans le champ ID de vérification des domaines personnalisés.](./media/app-service-web-tutorial-custom-domain/get-custom-domain-verification-id.png)

> [!WARNING]
> L’ajout d’ID de vérification de domaines à votre domaine personnalisé peut empêcher les entrées DNS non résolues et éviter les acquisitions de sous-domaine. Pour les domaines personnalisés que vous avez précédemment configurés sans cet ID de vérification, vous devez les protéger du même risque en ajoutant l’ID de vérification à votre enregistrement DNS. Pour plus d’informations sur cette menace critique courante, consultez [Acquisition de sous-domaine](../security/fundamentals/subdomain-takeover.md).

## <a name="map-your-domain"></a>Mapper votre domaine

Vous pouvez utiliser un enregistrement CNAME ou un enregistrement A pour mapper un nom DNS personnalisé à App Service. Suivez les étapes respectives :

- [Mapper un enregistrement CNAME](#map-a-cname-record)
- [Mapper un enregistrement A](#map-an-a-record)
- [Mapper un domaine générique (avec un enregistrement CNAME)](#map-a-wildcard-domain)

> [!NOTE]
> Vous devez utiliser des enregistrements CNAME pour tous les noms DNS personnalisés à l’exception des domaines racine (par exemple, `contoso.com`). Pour les domaines racine, utilisez des enregistrements A.

### <a name="map-a-cname-record"></a>Mapper un enregistrement CNAME

Dans l’exemple de ce didacticiel, vous ajoutez un enregistrement CNAME pour le sous-domaine `www` (`www.contoso.com`, par exemple).

Si vous avez un sous-domaine autre que `www`, remplacez `www` par votre sous-domaine (par exemple, par `sub` si votre domaine personnalisé est `sub.constoso.com`).

#### <a name="access-dns-records-with-a-domain-provider"></a>Accès aux enregistrements DNS avec un fournisseur de domaine

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-cname-record"></a>Créer un enregistrement CNAME

Mappez un sous-domaine au nom de domaine par défaut de l’application (`<app-name>.azurewebsites.net`, sachant que `<app-name>` est le nom de votre application). Pour créer un mappage CNAME pour le sous-domaine `www`, créez deux enregistrements :

| Type d’enregistrement | Host | Valeur | Commentaires |
| - | - | - |
| CNAME | `www` | `<app-name>.azurewebsites.net` | Le mappage de domaine lui-même. |
| TXT | `asuid.www` | [L’ID de vérification que vous avez obtenu précédemment](#get-a-domain-verification-id). | App Service accède à l’enregistrement TXT `asuid.<subdomain>` pour vérifier votre propriété du domaine personnalisé. |

Après avoir ajouté les enregistrements CNAME et TXT, la page d’enregistrements DNS ressemble à l’exemple suivant :

![Capture d’écran montrant la navigation dans le portail vers une application Azure.](./media/app-service-web-tutorial-custom-domain/cname-record.png)

#### <a name="enable-the-cname-record-mapping-in-azure"></a>Activer le mappage d’enregistrement CNAME dans Azure

1. Dans le volet gauche de la page d’application du portail Azure, sélectionnez **Domaines personnalisés**.

    ![Capture d’écran montrant le menu Domaines personnalisés.](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. Dans la page **Domaines personnalisés** de l’application, ajoutez le nom DNS personnalisé complet (`www.contoso.com`) à la liste.

1. Sélectionnez **Ajouter un domaine personnalisé**.

    ![Capture d’écran montrant l’élément Ajouter un nom d’hôte.](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Tapez le nom de domaine complet que vous avez ajouté pour un enregistrement CNAME, tel que `www.contoso.com`.

1. Sélectionnez **Valider**. La page **Ajouter un domaine personnalisé** s’affiche.

1. Assurez-vous que **Type d’enregistrement du nom d’hôte** est défini sur **CNAME (www\.exemple.com ou tout sous-domaine)** . Sélectionnez **Ajouter un domaine personnalisé**.

    ![Capture d’écran montrant le bouton Ajouter un domaine personnalisé.](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

    Un certain temps peut être nécessaire pour que le nouveau domaine personnalisé s’affiche dans la page **Domaines personnalisés** de votre application. Actualisez le navigateur pour mettre à jour les données.

    ![Capture d’écran montrant l’ajout de l’enregistrement CNAME.](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

    > [!NOTE]
    > Si un avertissement s’affiche concernant votre domaine personnalisé, cela signifie qu’il n’est pas encore lié à un certificat TLS/SSL. Toute requête HTTPS envoyée par un navigateur à votre domaine personnalisé générera une erreur ou un avertissement, selon le navigateur utilisé. Pour ajouter une liaison TLS/SSL, consultez [Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service](configure-ssl-bindings.md).

    Si vous avez raté une étape ou si vous avez fait une faute de frappe, une erreur de vérification s’affichera au bas de la page.

    ![Capture d’écran montrant une erreur de vérification.](./media/app-service-web-tutorial-custom-domain/verification-error-cname.png)

<a name="a" aria-hidden="true"></a>

### <a name="map-an-a-record"></a>Mapper un enregistrement A

Dans l’exemple de ce didacticiel, vous ajoutez un enregistrement A pour le domaine racine (par exemple, `contoso.com`).

<a name="info"></a>

#### <a name="copy-the-apps-ip-address"></a>Copier l’adresse IP de l’application

Pour mapper un enregistrement A, vous avez besoin de l’adresse IP externe de l’application. Vous trouverez cette adresse IP dans la page **Domaines personnalisés** de l’application dans le portail Azure.

1. Dans le volet gauche de la page d’application du portail Azure, sélectionnez **Domaines personnalisés**.

   ![Capture d’écran montrant le menu Domaines personnalisés.](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. Dans la page **Domaines personnalisés**, copiez l’adresse IP de l’application.

   ![Capture d’écran montrant la navigation dans le portail vers une application Azure.](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

#### <a name="access-dns-records-with-a-domain-provider"></a>Accès aux enregistrements DNS avec un fournisseur de domaine

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-a-record"></a>Créer l’enregistrement A

Pour mapper un enregistrement A à une application, généralement vers le domaine racine, créez deux enregistrements :

| Type d’enregistrement | Host | Valeur | Commentaires |
| - | - | - |
| Un | `@` | Adresse IP de [Copier l’adresse IP de l’application](#info) | Le mappage de domaine lui-même (`@` représente généralement le domaine racine). |
| TXT | `asuid` | [L’ID de vérification que vous avez obtenu précédemment](#get-a-domain-verification-id). | App Service accède à l’enregistrement TXT `asuid.<subdomain>` pour vérifier votre propriété du domaine personnalisé. Pour le domaine racine, utilisez `asuid`. |

> [!NOTE]
> Pour ajouter un sous-domaine (comme `www.contoso.com`) à l’aide d’un enregistrement A au lieu d’un [enregistrement CNAME](#map-a-cname-record) recommandé, votre enregistrement A et votre enregistrement TXT doivent se présenter comme dans le tableau suivant :
>
> | Type d’enregistrement | Host | Valeur |
> | - | - | - |
> | Un | `www` | Adresse IP de [Copier l’adresse IP de l’application](#info) |
> | TXT | `asuid.www` | [L’ID de vérification que vous avez obtenu précédemment](#get-a-domain-verification-id). |
>

Une fois les enregistrements ajoutés, la page d’enregistrements DNS ressemble à l’exemple suivant :

![Capture d’écran montrant une page d’enregistrements DNS.](./media/app-service-web-tutorial-custom-domain/a-record.png)

<a name="enable-a" aria-hidden="true"></a>

#### <a name="enable-the-a-record-mapping-in-the-app"></a>Activer le mappage d’enregistrement A dans l’application

De retour dans la page **Domaines personnalisés** de l’application sur le portail Azure, ajoutez à la liste le nom DNS personnalisé complet (par exemple, `contoso.com`).

1. Sélectionnez **Ajouter un domaine personnalisé**.

    ![Capture d’écran montrant l’ajout d’un nom d’hôte.](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Tapez le nom de domaine complet pour lequel vous avez configuré l’enregistrement A, tel que `contoso.com`. 

1. Sélectionnez **Valider**. La page **Ajouter un domaine personnalisé** s’affiche.

1. Assurez-vous que **Type d’enregistrement du nom d’hôte** est défini sur **Enregistrement A (exemple.com)** . Sélectionnez **Ajouter un domaine personnalisé**.

    ![Capture d’écran montrant l’ajout d’un nom DNS à l’application.](./media/app-service-web-tutorial-custom-domain/validate-domain-name.png)

    Un certain temps peut être nécessaire pour que le nouveau domaine personnalisé s’affiche dans la page **Domaines personnalisés** de votre application. Actualisez le navigateur pour mettre à jour les données.

    ![Capture d’écran montrant l’ajout d’un enregistrement A.](./media/app-service-web-tutorial-custom-domain/a-record-added.png)

    > [!NOTE]
    > Si un avertissement s’affiche concernant votre domaine personnalisé, cela signifie qu’il n’est pas encore lié à un certificat TLS/SSL. Toute requête HTTPS envoyée par un navigateur à votre domaine personnalisé générera une erreur ou un avertissement, selon le navigateur utilisé. Pour ajouter une liaison TLS/SSL, consultez [Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service](configure-ssl-bindings.md).
    
    Si vous avez raté une étape ou si vous avez fait une faute de frappe, une erreur de vérification s’affichera au bas de la page.
    
    ![Capture d’écran montrant une erreur de vérification.](./media/app-service-web-tutorial-custom-domain/verification-error.png)
    
<a name="wildcard" aria-hidden="true"></a>

### <a name="map-a-wildcard-domain"></a>Mapper un domaine générique

Dans l’exemple du didacticiel, vous mappez un [nom DNS générique](https://en.wikipedia.org/wiki/Wildcard_DNS_record) (par exemple, `*.contoso.com`) vers l’application App Service en ajoutant un enregistrement CNAME.

#### <a name="access-dns-records-with-a-domain-provider"></a>Accès aux enregistrements DNS avec un fournisseur de domaine

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-cname-record"></a>Créer un enregistrement CNAME

Mappez un nom générique `*` au nom de domaine par défaut de l’application (`<app-name>.azurewebsites.net`, sachant que `<app-name>` est le nom de votre application). Pour mapper le nom générique, créez deux enregistrements :

| Type d’enregistrement | Host | Valeur | Commentaires |
| - | - | - |
| CNAME | `*` | `<app-name>.azurewebsites.net` | Le mappage de domaine lui-même. |
| TXT | `asuid` | [L’ID de vérification que vous avez obtenu précédemment](#get-a-domain-verification-id). | App Service accède à l’enregistrement TXT `asuid` pour vérifier votre propriété du domaine personnalisé. |

Pour l’exemple de domaine `*.contoso.com`, l’enregistrement CNAME mappera le nom `*` à `<app-name>.azurewebsites.net`.

Après avoir ajouté l’enregistrement CNAME, la page d’enregistrements DNS ressemble à l’exemple suivant :

![Capture d’écran montrant la navigation vers une application Azure.](./media/app-service-web-tutorial-custom-domain/cname-record-wildcard.png)

#### <a name="enable-the-cname-record-mapping-in-the-app"></a>Activer le mappage d’enregistrement CNAME dans l’application

Vous pouvez maintenant ajouter à l’application un sous-domaine qui correspond au nom générique (par exemple, `sub1.contoso.com`, `sub2.contoso.com` et `*.contoso.com` correspondent tous à `*.contoso.com`).

1. Dans le volet gauche de la page d’application du portail Azure, sélectionnez **Domaines personnalisés**.

    ![Capture d’écran montrant le menu Domaines personnalisés.](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. Sélectionnez **Ajouter un domaine personnalisé**.

    ![Capture d’écran montrant l’ajout d’un nom d’hôte.](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Tapez un nom de domaine complet qui correspond au domaine générique (par exemple, `sub1.contoso.com`), puis sélectionnez **Valider**.

    Le bouton **Ajouter un domaine personnalisé** est activé.

1. Assurez-vous que **Type d’enregistrement du nom d’hôte** est défini sur **Enregistrement CNAME (www\.exemple.com ou tout sous-domaine)** . Sélectionnez **Ajouter un domaine personnalisé**.

    ![Capture d’écran qui montre l’ajout d’un nom DNS à l’application.](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname-wildcard.png)

    Un certain temps peut être nécessaire pour que le nouveau domaine personnalisé s’affiche dans la page **Domaines personnalisés** de votre application. Actualisez le navigateur pour mettre à jour les données.

1. Resélectionnez l’icône **+** pour ajouter un autre domaine personnalisé qui correspond au domaine générique. Par exemple, ajoutez `sub2.contoso.com`.

    ![Capture d’écran montrant l’ajout d’un enregistrement CNAME.](./media/app-service-web-tutorial-custom-domain/cname-record-added-wildcard-2.png)

    > [!NOTE]
    > Si un avertissement s’affiche concernant votre domaine personnalisé, cela signifie qu’il n’est pas encore lié à un certificat TLS/SSL. Toute requête HTTPS envoyée par un navigateur à votre domaine personnalisé générera une erreur ou un avertissement, selon le navigateur utilisé. Pour ajouter une liaison TLS/SSL, consultez [Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service](configure-ssl-bindings.md).
    
## <a name="test-in-a-browser"></a>Test dans un navigateur

Dans votre navigateur, accédez aux noms DNS que vous avez configurés précédemment (par exemple, `contoso.com`, `www.contoso.com`,`sub1.contoso.com` et `sub2.contoso.com`).

![Capture d’écran montrant la navigation vers une application Azure.](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

## <a name="resolve-404-not-found"></a>Résoudre une erreur 404 « Introuvable »

Si vous recevez une erreur HTTP 404 (Introuvable) lors de la navigation vers l’URL de votre domaine personnalisé, vérifiez que votre domaine résout l’adresse IP de votre application en utilisant <a href="https://www.whatsmydns.net/" target="_blank">WhatsmyDNS.net</a>. Si ce n’est pas le cas, l’une des raisons suivantes peut en être la cause :

- Un enregistrement A et/ou un enregistrement CNAME est manquant dans le domaine personnalisé configuré.
- Le client du navigateur a mis en cache l'ancienne adresse IP de votre domaine. Effacez le cache et testez à nouveau la résolution DNS. Sur une machine Windows, effacez le cache avec `ipconfig /flushdns`.

<a name="virtualdir" aria-hidden="true"></a>

## <a name="migrate-an-active-domain"></a>Migrer un domaine actif

Pour migrer un site actif et son nom de domaine DNS vers App Service sans interruption de service, voir [Migrer un nom DNS actif vers Azure App Service](manage-custom-dns-migrate-domain.md).

## <a name="redirect-to-a-custom-directory"></a>Rediriger vers un répertoire personnalisé

Par défaut, App Service dirige les demandes web au répertoire racine du code de votre application. Cependant, certains frameworks web ne démarrent pas dans le répertoire racine. Par exemple, [Laravel](https://laravel.com/) démarre dans le sous-répertoire `public`. Pour continuer avec l’exemple DNS `contoso.com`, une telle application est accessible à partir de `http://contoso.com/public`, mais il est préférable de diriger `http://contoso.com` vers le répertoire `public`. Cette étape n’implique pas la résolution DNS, mais la personnalisation du répertoire virtuel.

Pour personnaliser un répertoire virtuel, sélectionnez **Paramètres de l’application** dans le volet gauche de la page de votre application web.

En bas de la page, le répertoire virtuel racine `/` pointe par défaut vers `site\wwwroot`, qui correspond au répertoire racine de votre code d'application. Modifiez-le afin qu'il pointe vers `site\wwwroot\public`, par exemple, puis enregistrez vos modifications.

![Capture d’écran montrant la personnalisation d’un répertoire virtuel.](./media/app-service-web-tutorial-custom-domain/customize-virtual-directory.png)

Une fois l’opération terminée, votre application doit retourner la bonne page au chemin racine (par exemple, `http://contoso.com`).

## <a name="automate-with-scripts"></a>Automatiser des tâches à l’aide de scripts

Vous pouvez automatiser la gestion des domaines personnalisés à l’aide de scripts, en utilisant [Azure CLI](/cli/azure/install-azure-cli) ou [Azure PowerShell](/powershell/azure/).

### <a name="azure-cli"></a>Azure CLI

La commande suivante ajoute un nom DNS personnalisé configuré à une application App Service.

```bash 
az webapp config hostname add \
    --webapp-name <app-name> \
    --resource-group <resource_group_name> \
    --hostname <fully_qualified_domain_name>
``` 

Pour plus d’informations, consultez [Mapper un nom de domaine personnalisé à une application web](scripts/cli-configure-custom-domain.md).

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

La commande suivante ajoute un nom DNS personnalisé configuré à une application App Service.

```powershell  
Set-AzWebApp `
    -Name <app-name> `
    -ResourceGroupName <resource_group_name> ` 
    -HostNames @("<fully_qualified_domain_name>","<app-name>.azurewebsites.net")
```

Pour plus d’informations, consultez [Attribuer un domaine personnalisé à une application web](scripts/powershell-configure-custom-domain.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Mapper un sous-domaine à l’aide d’un enregistrement CNAME
> * Mapper un domaine racine à l’aide d’un enregistrement A
> * Mapper un domaine générique à l’aide d’un enregistrement CNAME
> * Rediriger l’URL par défaut vers un répertoire personnalisé
> * Automatiser le mappage de domaine à l’aide de scripts

Passez au tutoriel suivant pour découvrir comment lier un certificat TLS/SSL personnalisé à une application web.

> [!div class="nextstepaction"]
> [Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service](configure-ssl-bindings.md)
