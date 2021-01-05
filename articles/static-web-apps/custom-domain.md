---
title: Configurer un domaine personnalisé dans Azure static Web Apps
description: Découvrez comment mapper un domaine personnalisé avec Azure Static Web Apps
services: static-web-apps
author: burkeholland
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: buhollan
ms.openlocfilehash: 578860883a108bba4b4bcd8cd04e8c08f484d474
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92173680"
---
# <a name="setup-a-custom-domain-in-azure-static-web-apps-preview"></a>Configurer un domaine personnalisé dans Azure Static Web Apps en préversion

Par défaut, Azure Static Web Apps fournit un nom de domaine généré automatiquement. Cet article explique comment mapper un nom de domaine personnalisé avec une application Azure Static Web Apps.

## <a name="prerequisites"></a>Prérequis

- Un nom de domaine acheté
- Accéder aux propriétés de configuration DNS de votre domaine

Lors de la configuration des noms de domaine, les enregistrements « A » sont utilisés pour mapper les domaines racine (par exemple, `example.com`) à une adresse IP. Les domaines racines doivent être mappés directement à une adresse IP, car la spécification DNS n’autorise pas le mappage d’un domaine à un autre.

## <a name="dns-configuration-options"></a>Options de configuration DNS

Il existe plusieurs types de configurations DNS disponibles pour une application.

| Pour | Alors |
|--|--|
| Prendre en charge `www.example.com` ou `blog.example.net` | [Mapper un enregistrement CNAME](#map-a-cname-record) |
| Prise en charge d’`example.com` | [Configurer un domaine racine](#configure-a-root-domain) |
| Diriger tous les sous-domaines vers `www.example.com` | [Mapper un domaine générique](#map-a-wildcard-domain) |

## <a name="map-a-cname-record"></a>Mapper un enregistrement CNAME

Un enregistrement CNAME mappe un domaine à un autre. Vous pouvez utiliser un enregistrement CNAME pour mapper `www.example.com`, `blog.example.com` ou un autre sous-domaine généré automatiquement fourni par Azure Static Web Apps.

1. Ouvrez le [portail Azure](https://portal.azure.com) et connectez-vous avec votre compte Azure.

1. Rechercher et sélectionner **Static Web Apps**

1. Dans la page _Static Web Apps_ , sélectionnez le nom de votre application web.

1. Cliquez sur **Domaines personnalisés** dans le menu.

1. Cliquez sur le bouton **Ajouter** .

1. Dans la fenêtre _Domaines personnalisés_ , copiez l’URL dans le champ **Valeur** .

### <a name="configure-dns-provider"></a>Configurer un fournisseur DNS

1. Connectez-vous au site web de votre fournisseur de domaine.

2. Trouvez la page de gestion des enregistrements DNS. Chaque fournisseur de domaine ayant sa propre interface d’enregistrements DNS, consultez la documentation de votre fournisseur. Recherchez les zones du site qui portent les mentions **Nom de domaine** , **DNS** ou **Gestion du nom de serveur** .

3. Vous trouvez généralement la page des enregistrements DNS en affichant vos informations de compte, puis en recherchant un lien comme **Mes domaines** . Accédez à cette page et recherchez un lien nommé **Fichier de zone** , **Enregistrements DNS** ou **Configuration avancée** .

    La capture d’écran suivante est un exemple d’une page d’enregistrements DNS :

    :::image type="content" source="media/custom-domain/example-record-ui.png" alt-text="Exemple de configuration de fournisseur DNS":::

4. Créez un enregistrement **CNAME** avec les valeurs suivantes...

    | Paramètre             | Valeur                     |
    | ------------------- | ------------------------- |
    | Type                | CNAME                     |
    | Host                | www                       |
    | Valeur               | Coller à partir du presse-papiers |
    | TTL (le cas échéant) | Laisser la valeur par défaut    |

5. Enregistrez les modifications avec votre fournisseur DNS.

### <a name="validate-cname"></a>Valider CNAME

1. Revenez à la fenêtre _Domaines personnalisés_ dans le Portail Azure.

1. Entrez votre domaine, y compris la partie `www` dans la section _Valider le domaine personnalisé_ .

1. Cliquez sur le bouton **Valider** .

À présent que le domaine personnalisé est configuré, le fournisseur DNS peut prendre plusieurs heures pour propager les modifications dans le monde entier. Vous pouvez vérifier l’état de la propagation en accédant à [dnspropagation.net](https://dnspropagation.net). Entrez votre domaine personnalisé, y compris la partie `www`, sélectionnez CNAME dans la liste déroulante, puis sélectionnez **Démarrer** .

Si vos modifications DNS sont indiquées, le site web renvoie l’URL générée automatiquement de votre application Static Web App (par exemple, _random-name-123456789c.azurestaticapps.net_ ).

## <a name="configure-a-root-domain"></a>Configurer un domaine racine

Un domaine racine correspond à votre domaine moins n’importe quel sous-domaine, notamment `www`. Par exemple, le domaine racine de `www.example.com` est `example.com`. Un tel domaine est également appelé domaine « APEX ».

Alors que la prise en charge du domaine racine n’est pas disponible pendant la préversion, vous pouvez consulter le billet de blog [Configurer des domaines racines dans Azure Static Web Apps](https://burkeholland.github.io/posts/static-app-root-domain) pour plus d’informations sur la configuration de la prise en charge du domaine racine avec une application Static Web App.

## <a name="map-a-wildcard-domain"></a>Mapper un domaine générique

Parfois, vous souhaitez que tout le trafic envoyé à un sous-domaine soit acheminé vers un autre domaine. Un exemple courant consiste à mapper tout le trafic de sous-domaines vers `www.example.com`. De cette façon, même si quelqu’un tape `w.example.com` au lieu de `www.example.com`, la requête est envoyée à `www.example.com`.

### <a name="configure-dns-provider"></a>Configurer un fournisseur DNS

1. Connectez-vous au site web de votre fournisseur de domaine.

2. Trouvez la page de gestion des enregistrements DNS. Chaque fournisseur de domaine ayant sa propre interface d’enregistrements DNS, consultez la documentation de votre fournisseur. Recherchez les zones du site qui portent les mentions **Nom de domaine** , **DNS** ou **Gestion du nom de serveur** .

3. Vous trouvez généralement la page des enregistrements DNS en affichant vos informations de compte, puis en recherchant un lien comme **Mes domaines** . Accédez à cette page et recherchez un lien nommé **Fichier de zone** , **Enregistrements DNS** ou **Configuration avancée** .

    La capture d’écran suivante est un exemple d’une page d’enregistrements DNS :

    :::image type="content" source="media/custom-domain/example-record-ui.png" alt-text="Exemple de configuration de fournisseur DNS":::

4. Créez un enregistrement **CNAME** avec les valeurs suivantes, en remplaçant `www.example.com` par votre nom de domaine personnalisé.

    | Paramètre | Valeur                  |
    | ------- | ---------------------- |
    | Type    | CNAME                  |
    | Host    | \*                     |
    | Valeur   | www.example.com        |
    | TTL     | Laisser la valeur par défaut |

5. Enregistrez les modifications avec votre fournisseur DNS.

À présent que le domaine générique est configuré, la propagation des changements à travers le monde peut prendre plusieurs heures. Vous pouvez vérifier l’état de la propagation en accédant à [dnspropagation.net](https://dnspropagation.net). Entrez votre domaine personnalisé de domaine avec n’importe quel sous-domaine (sauf `www`), sélectionnez CNAME dans la liste déroulante, puis sélectionnez **Démarrer** .

Si vos modifications DNS sont renseignées, le site web renvoie votre domaine personnalisé configuré pour votre application Static Web App (par exemple, `www.example.com`).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configurer les paramètres d’application](application-settings.md)
