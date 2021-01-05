---
title: Sécuriser un DNS personnalisé avec une liaison TLS/SSL
description: Sécurisez l’accès HTTPS à votre domaine personnalisé en créant une liaison TLS/SSL avec un certificat. Améliorez la sécurité de votre site web en appliquant HTTPS ou TLS 1.2.
tags: buy-ssl-certificates
ms.topic: tutorial
ms.date: 04/30/2020
ms.reviewer: yutlin
ms.custom: seodec18
ms.openlocfilehash: f7301809b3befc41110a32062d6e478c412fa56e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90981111"
---
# <a name="secure-a-custom-dns-name-with-a-tlsssl-binding-in-azure-app-service"></a>Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service

Cet article explique comment sécuriser le [domaine personnalisé](app-service-web-tutorial-custom-domain.md) dans votre [application App Service](./index.yml) ou [application de fonction](../azure-functions/index.yml) en créant une liaison de certificat. Lorsque vous avez terminé, vous pouvez accéder à votre application App Service au niveau du point de terminaison `https://` pour votre nom DNS personnalisé (par exemple, `https://www.contoso.com`). 

![Application web avec certificat TLS/SSL personnalisé](./media/configure-ssl-bindings/app-with-custom-ssl.png)

La sécurisation d’un [domaine personnalisé](app-service-web-tutorial-custom-domain.md) avec un certificat implique deux étapes :

- [Ajoutez un certificat privé dans App Service](configure-ssl-certificate.md) qui satisfait à toutes les [exigences du certificat privé](configure-ssl-certificate.md#private-certificate-requirements).
-  Créez une liaison TLS au domaine personnalisé correspondant. Cette deuxième étape est traitée dans cet article.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Mettre à jour le niveau de tarification de votre application
> * Sécuriser un domaine personnalisé à l’aide d’un certificat
> * Appliquer le protocole HTTPS
> * Appliquer le protocole TLS 1.1/1.2
> * Automatiser la gestion TLS avec des scripts

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de ce guide pratique, vous devez au préalable :

- [Création d’une application App Service](./index.yml)
- [Mapper un nom de domaine à votre application](app-service-web-tutorial-custom-domain.md) ou [acheter et configurer un nom de domaine dans Azure](manage-custom-dns-buy-domain.md)
- [Ajouter un certificat privé à votre application](configure-ssl-certificate.md)

> [!NOTE]
> Le moyen le plus simple d’ajouter un certificat privé consiste à [créer un certificat géré App Service gratuit](configure-ssl-certificate.md#create-a-free-certificate-preview) (préversion).

[!INCLUDE [Prepare your web app](../../includes/app-service-ssl-prepare-app.md)]

<a name="upload"></a>

## <a name="secure-a-custom-domain"></a>Sécuriser un domaine personnalisé

Procédez comme suit :

Sur le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, dans le menu de gauche, sélectionnez **App Services** >  **\<app-name>** .

À partir de la barre de navigation gauche de votre application, démarrez la boîte de dialogue **Liaison TLS/SSL** en procédant comme suit :

- Sélectionnez **Domaines personnalisés** > **Ajouter une liaison**
- Sélectionnez **Paramètres TLS/SSL** > **Ajouter une liaison TLS/SSL**

![Ajouter une liaison au domaine](./media/configure-ssl-bindings/secure-domain-launch.png)

Dans **Domaine personnalisé**, sélectionnez le domaine personnalisé pour lequel vous souhaitez ajouter une liaison.

Si votre application a déjà un certificat pour le domaine personnalisé sélectionné, accédez à [Créer une liaison](#create-binding) directement. Dans le cas contraire, continuez.

### <a name="add-a-certificate-for-custom-domain"></a>Ajouter un certificat pour un domaine personnalisé

Si votre application n’a pas de certificat pour le domaine personnalisé sélectionné, vous avez deux options :

- **Charger un certificat PFX** : suivez le flux de travail sur [Téléchargement d’un certificat privé](configure-ssl-certificate.md#upload-a-private-certificate), puis sélectionnez cette option ici.
- **Importer un App Service Certificate** : suivez le flux de travail sur [Importer un App Service Certificate](configure-ssl-certificate.md#import-an-app-service-certificate), puis sélectionnez cette option ici.

> [!NOTE]
> Vous pouvez également [Créer un certificat gratuit](configure-ssl-certificate.md#create-a-free-certificate-preview) (préversion) ou [Importer un certificat Key Vault](configure-ssl-certificate.md#import-a-certificate-from-key-vault), mais vous devez le faire séparément, puis revenir à la boîte de dialogue **Liaison TLS/SSL**.

### <a name="create-binding"></a>Créer une liaison

Dans la boîte de dialogue **Liaisons TLS/SSL**, configurez la liaison TLS en vous aidant du tableau suivant, puis cliquez sur **Ajouter une liaison**.

| Paramètre | Description |
|-|-|
| Domaine personnalisé | Nom du domaine pour lequel ajouter une liaison TLS/SSL. |
| Empreinte numérique du certificat privé | Certificat à lier. |
| Type TLS/SSL | <ul><li>**[SNI SSL](https://en.wikipedia.org/wiki/Server_Name_Indication)** : plusieurs liaisons SNI SSL peuvent être ajoutées. Cette option permet de sécuriser plusieurs domaines sur la même adresse IP avec plusieurs certificats TLS/SSL. La plupart des navigateurs actuels (y compris Internet Explorer, Chrome, Firefox et Opera) prennent en charge SNI (pour plus d’informations, consultez [Indication du nom du serveur](https://wikipedia.org/wiki/Server_Name_Indication)).</li><li>**IP SSL** : une seule liaison IP SSL peut être ajoutée. Cette option permet de sécuriser une adresse IP publique dédiée avec un seul certificat TLS/SSL. Après avoir configuré la liaison, effectuez les étapes décrites dans [Remapper des enregistrements pour IP SSL](#remap-records-for-ip-ssl).<br/>IP SSL est pris en charge seulement dans le niveau **Standard** ou supérieur. </li></ul> |

Une fois l’opération terminée, l’état TLS/SSL du domaine personnalisé est remplacé par **Sécurisé**.

![Liaison TLS/SSL réussie](./media/configure-ssl-bindings/secure-domain-finished.png)

> [!NOTE]
> Quand un domaine présente l’état **Sécurisé** dans **Domaines personnalisés**, cela indique qu’il est sécurisé par un certificat. Toutefois, App Service ne vérifie pas si le certificat est auto-signé ou arrivé à expiration, par exemple, ce qui peut également provoquer l’affichage d’une erreur ou d’un avertissement dans les navigateurs.

## <a name="remap-records-for-ip-ssl"></a>Remapper des enregistrements pour IP SSL

Si vous n’utilisez pas d’IP SSL dans votre application, passez à [Tester HTTPS pour votre domaine personnalisé](#test-https).

Vous devez potentiellement effectuer deux modifications :

- Par défaut, votre application utilise une adresse IP publique partagée. Dès que vous liez un certificat avec IP SSL, App Service crée une adresse IP dédiée pour votre application. Si vous avez mappé un enregistrement A à votre application, mettez à jour votre registre de domaine avec cette nouvelle adresse IP dédiée.

    La page **Domaine personnalisé** de votre application est mise à jour avec la nouvelle adresse IP dédiée. [Copiez cette adresse IP](app-service-web-tutorial-custom-domain.md#info), puis [mappez à nouveau l’enregistrement A](app-service-web-tutorial-custom-domain.md#map-an-a-record) à cette nouvelle adresse IP.

- Si vous avez une liaison SNI SSL à `<app-name>.azurewebsites.net`, [remapper les mappages CNAME](app-service-web-tutorial-custom-domain.md#map-a-cname-record) pour les faire pointer à la place vers `sni.<app-name>.azurewebsites.net` (ajoutez le préfixe `sni`).

## <a name="test-https"></a>Test du protocole HTTPS

Dans différents navigateurs, accédez à `https://<your.custom.domain>` pour vérifier qu’il fournit votre application.

:::image type="content" source="./media/configure-ssl-bindings/app-with-custom-ssl.png" alt-text="Capture d’écran montrant un exemple de navigation vers votre domaine personnalisé avec l’URL contoso.com mise en évidence.":::

Votre code d’application peut inspecter le protocole via l’en-tête « x-appservice-proto ». L’en-tête aura la valeur `http` ou `https`. 

> [!NOTE]
> Si votre application permet de voir les erreurs de validation de certificat, vous utilisez probablement un certificat auto-signé.
>
> Si ce n’est pas le cas, vous pouvez avoir oublié des certificats intermédiaires lorsque vous avez exporté votre certificat vers le fichier PFX.

## <a name="prevent-ip-changes"></a>Empêcher les modifications d’IP

Votre adresse IP entrante peut être modifiée lorsque vous supprimez une liaison, même si cette liaison est une IP SSL. Cela est particulièrement important lorsque vous renouvelez un certificat qui se trouve déjà dans une liaison d’IP SSL. Pour éviter une modification de l’adresse IP de votre application, suivez ces étapes dans l’ordre :

1. Chargez le nouveau certificat.
2. Liez le nouveau certificat au domaine personnalisé souhaité, sans supprimer l’ancien. Cette action remplace la liaison plutôt que de supprimer l’ancienne.
3. Supprimez l’ancien certificat. 

## <a name="enforce-https"></a>Appliquer le protocole HTTPS

Par défaut, n’importe qui peut encore accéder à votre application avec HTTP. Vous pouvez rediriger toutes les demandes HTTP sur le port HTTPS.

Dans le volet de navigation gauche de la page de votre application, sélectionnez **Paramètres SSL**. Ensuite, dans **HTTPS uniquement**, sélectionnez **Activer**.

![Appliquer le protocole HTTPS](./media/configure-ssl-bindings/enforce-https.png)

Lorsque l’opération est terminée, accédez à une des URL HTTP pointant vers votre application. Par exemple :

- `http://<app_name>.azurewebsites.net`
- `http://contoso.com`
- `http://www.contoso.com`

## <a name="enforce-tls-versions"></a>Appliquer des versions TLS

Votre application autorise la version [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.2 par défaut, qui constitue le niveau TLS recommandé par les normes du secteur, telles que [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard). Pour appliquer d’autres versions TLS, procédez comme suit :

Dans le volet de navigation gauche de la page de votre application, sélectionnez **Paramètres SSL**. Ensuite, dans **Version TLS**, sélectionnez la version minimale de TLS souhaitée. Ce paramètre contrôle uniquement les appels entrants. 

![Appliquer le protocole TLS 1.1 ou 1.2](./media/configure-ssl-bindings/enforce-tls1-2.png)

Une fois l’opération terminée, votre application rejette toutes les connexions effectuées avec des versions antérieures de TLS.

## <a name="handle-tls-termination"></a>Gérer l’arrêt TLS

Dans App Service, un [arrêt TLS](https://wikipedia.org/wiki/TLS_termination_proxy) se produit au niveau des équilibreurs de charge réseau. Toutes les requêtes HTTPS accèdent donc à votre application en tant que requêtes HTTP non chiffrées. Si votre logique d’application doit vérifier si les requêtes utilisateur sont chiffrées ou non, inspectez l’en-tête `X-Forwarded-Proto`.

Des guides de configuration spécifiques au langage tels que le [guide de configuration Linux Node.js](configure-language-nodejs.md#detect-https-session) vous montrent comment détecter une session HTTPS dans le code de votre application.

## <a name="automate-with-scripts"></a>Automatiser des tâches à l’aide de scripts

### <a name="azure-cli"></a>Azure CLI

[!code-azurecli[main](../../cli_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.sh?highlight=3-5 "Bind a custom TLS/SSL certificate to a web app")] 

### <a name="powershell"></a>PowerShell

[!code-powershell[main](../../powershell_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.ps1?highlight=1-3 "Bind a custom TLS/SSL certificate to a web app")]

## <a name="more-resources"></a>Plus de ressources

* [Utiliser un certificat TLS/SSL dans votre code dans Azure App Service](configure-ssl-certificate-in-code.md)
* [FORUM AUX QUESTIONS : App Service Certificates](./faq-configuration-and-management.md)