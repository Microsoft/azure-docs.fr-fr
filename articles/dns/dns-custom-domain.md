---
title: Intégrer Azure DNS à vos ressources Azure – Azure DNS
description: Dans cet article, découvrez comment utiliser Azure DNS et fournir un DNS pour vos ressources Azure.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 7/13/2019
ms.author: rohink
ms.openlocfilehash: 4d8af5815e544698ab833001e5ce6d0f4a30a264
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92487396"
---
# <a name="use-azure-dns-to-provide-custom-domain-settings-for-an-azure-service"></a>Utiliser DNS Azure pour fournir des paramètres de domaine personnalisé pour un service Azure

DNS Azure fournit un DNS pour un domaine personnalisé, pour toutes vos ressources Azure prenant en charge les domaines personnalisés ou ayant un nom de domaine complet (FQDN). C'est le cas, par exemple, si vous disposez d'une application web Azure et souhaitez que vos utilisateurs y accèdent en utilisant soit contoso.com ou www\.contoso.com comme nom de domaine complet. Cet article vous guide tout au long de la configuration de votre service Azure avec DNS Azure pour l’utilisation de domaines personnalisés.

## <a name="prerequisites"></a>Prérequis

Pour pouvoir utiliser DNS Azure pour votre domaine personnalisé, vous devez tout d’abord déléguer votre domaine à DNS d’Azure. Pour obtenir des instructions sur la façon de configurer vos serveurs de noms pour la délégation, voir [Délégation de domaine à Azure DNS](./dns-delegate-domain-azure-dns.md). Une fois votre domaine délégué à votre zone DNS Azure, vous êtes en mesure de configurer les enregistrements DNS nécessaires.

Vous pouvez configurer un domaine personnalisé pour [Applications Azure Function](#azure-function-app), [Adresses IP publiques](#public-ip-address), [App Service (Web Apps)](#app-service-web-apps), [Stockage Blob](#blob-storage) et [Azure CDN](#azure-cdn).

## <a name="azure-function-app"></a>Application Azure Function

Pour configurer un domaine personnalisé pour des applications de fonction Azure, un enregistrement CNAME est créé, ainsi qu’une configuration sur l’application de fonction proprement dite.
 
Accédez à **Function App** , puis sélectionnez votre application de fonction. Cliquez sur **Fonctionnalités de la plateforme** puis, sous **Mise en réseau** , cliquez sur **Domaines personnalisés** .

![Panneau d’application de fonction](./media/dns-custom-domain/functionapp.png)

Notez l’URL actuelle affichée sur le panneau **Domaines personnalisés** . Cette adresse est utilisée en tant qu’alias pour l’enregistrement DNS créé.

![Panneau de domaine personnalisé](./media/dns-custom-domain/functionshostname.png)

Accédez à votre zone DNS, puis cliquez sur **+ Jeu d’enregistrements** . Entrez les informations suivantes sur le panneau **Ajouter un jeu d’enregistrements** , puis cliquez sur **OK** pour créer le jeu d’enregistrements.

|Propriété  |Valeur  |Description  |
|---------|---------|---------|
|Nom     | myFunctionApp        | Cette valeur et l’étiquette du nom du domaine constituent le FQDN pour le nom de domaine personnalisé.        |
|Type     | CNAME        | Utiliser un enregistrement CNAME équivaut à utiliser un alias.        |
|TTL     | 1        | 1 est utilisé pour 1 heure        |
|Unité de durée de vie     | Heures        | Les heures sont utilisées en tant que mesure du temps         |
|Alias     | adatumfunction.azurewebsites.net        | Nom DNS pour lequel vous créez l’alias. Dans cet exemple, il s’agit du nom DNS adatumfunction.azurewebsites.net fourni par défaut à l’application de fonction.        |

Revenez à votre application de fonction, cliquez sur **Fonctionnalités de la plateforme** puis, sous **Mise en réseau** , cliquez sur **Domaines personnalisés** . Ensuite, sous **Noms d’hôte personnalisés** , cliquez sur **+ Ajouter un nom d’hôte** .

Sur le panneau **Ajouter un nom d’hôte** , dans le champ de texte **nom d’hôte** entrez l’enregistrement CNAME, puis cliquez sur **Valider** . Si l’enregistrement a été trouvé, le bouton **Ajouter un nom d’hôte** s’affiche. Pour ajouter l’alias, cliquez sur **Ajouter un nom d’hôte** .

![Panneau Ajouter un nom d’hôte pour application de fonction](./media/dns-custom-domain/functionaddhostname.png)

## <a name="public-ip-address"></a>Adresse IP publique

Pour configurer un domaine personnalisé pour des services qui utilisent une ressource d'adresse IP publique, comme Application Gateway, Load Balancer, le service cloud, les machines virtuelles Resource Manager et Classic, un enregistrement A est utilisé.

Accédez à **Mise en réseau** > **Adresse IP publique** , sélectionnez la ressource d’adresse IP publique, puis cliquez sur **Configuration** . Notez l’adresse IP affichée.

![Panneau Adresse IP publique](./media/dns-custom-domain/publicip.png)

Accédez à votre zone DNS, puis cliquez sur **+ Jeu d’enregistrements** . Entrez les informations suivantes sur le panneau **Ajouter un jeu d’enregistrements** , puis cliquez sur **OK** pour créer le jeu d’enregistrements.


|Propriété  |Valeur  |Description  |
|---------|---------|---------|
|Nom     | mywebserver        | Cette valeur et l’étiquette du nom du domaine constituent le FQDN pour le nom de domaine personnalisé.        |
|Type     | Un        | Utiliser un enregistrement A si la ressource est une adresse IP.        |
|TTL     | 1        | 1 est utilisé pour 1 heure        |
|Unité de durée de vie     | Heures        | Les heures sont utilisées en tant que mesure du temps         |
|Adresse IP     | `<your ip address>`       | Adresse IP publique.|

![Créer un enregistrement A](./media/dns-custom-domain/arecord.png)

Une fois l’enregistrement A créé, exécutez `nslookup` pour valider les résolutions de l’enregistrement.

![Recherche DNS d’adresse IP publique](./media/dns-custom-domain/publicipnslookup.png)

## <a name="app-service-web-apps"></a>App Service (Web Apps)

Les étapes suivantes vous guident tout au long de la configuration d’un domaine personnalisé pour une application web de service d’applications.

Accédez à **App Service** , sélectionnez la ressource pour laquelle vous configurez un nom de domaine personnalisé, puis cliquez sur **Domaines personnalisés** .

Notez l’URL actuelle affichée sur le panneau **Domaines personnalisés** . Cette adresse est utilisée en tant qu’alias pour l’enregistrement DNS créé.

![Panneau Domaines personnalisés](./media/dns-custom-domain/url.png)

Accédez à votre zone DNS, puis cliquez sur **+ Jeu d’enregistrements** . Entrez les informations suivantes sur le panneau **Ajouter un jeu d’enregistrements** , puis cliquez sur **OK** pour créer le jeu d’enregistrements.


|Propriété  |Valeur  |Description  |
|---------|---------|---------|
|Nom     | mywebserver        | Cette valeur et l’étiquette du nom du domaine constituent le FQDN pour le nom de domaine personnalisé.        |
|Type     | CNAME        | Utiliser un enregistrement CNAME équivaut à utiliser un alias. Si la ressource utilisait une adresse IP, un enregistrement A serait utilisé.        |
|TTL     | 1        | 1 est utilisé pour 1 heure        |
|Unité de durée de vie     | Heures        | Les heures sont utilisées en tant que mesure du temps         |
|Alias     | webserver.azurewebsites.net        | Nom DNS pour lequel vous créez l’alias. Dans cet exemple, il s’agit du nom DNS webserver.azurewebsites.net fourni par défaut à l’application web.        |


![Créer un enregistrement CNAME](./media/dns-custom-domain/createcnamerecord.png)

Revenez au service d’applications configuré pour le nom de domaine personnalisé. Cliquez sur **Domaines personnalisés** , puis sur **Noms d’hôte** . Pour ajouter l’enregistrement CNAME que vous avez créé, cliquez sur **+ Ajouter un nom d’hôte** .

![Capture d’écran mettant en évidence le bouton + Ajouter un nom d’hôte.](./media/dns-custom-domain/figure1.png)

Une fois que le processus est terminé, exécutez **nslookup** pour vérifier que la résolution de nom fonctionne.

![figure 1](./media/dns-custom-domain/finalnslookup.png)

Pour en savoir plus sur le mappage d’un domaine personnalisé à App Service, voir [Mapper un nom DNS personnalisé existant à des applications web Azure](../app-service/app-service-web-tutorial-custom-domain.md?toc=%dns%2ftoc.json).

Pour savoir comment migrer un nom DNS actif, consultez [Migrer un nom DNS actif vers Azure App Service](../app-service/manage-custom-dns-migrate-domain.md).

Si vous avez besoin d’acheter un domaine personnalisé, pour en savoir plus sur les domaines App Service, voir [Acheter et configurer un nom de domaine personnalisé dans Azure App Service](../app-service/manage-custom-dns-buy-domain.md).

## <a name="blob-storage"></a>Stockage d'objets blob

Les étapes suivantes vous guident tout au long de la configuration d’un enregistrement CNAME pour un compte Stockage Blob à l’aide de la méthode asverify. Cette méthode garantit l’absence de temps d’arrêt.

Accédez à **Stockage** > **Comptes de stockage** , sélectionnez votre compte de stockage, puis cliquez sur **Domaine personnalisé** . Notez le FQDN de l’étape 2. Cette valeur est utilisée pour créer le premier enregistrement CNAME.

![Domaine personnalisé de stockage BLOB](./media/dns-custom-domain/blobcustomdomain.png)

Accédez à votre zone DNS, puis cliquez sur **+ Jeu d’enregistrements** . Entrez les informations suivantes sur le panneau **Ajouter un jeu d’enregistrements** , puis cliquez sur **OK** pour créer le jeu d’enregistrements.


|Propriété  |Valeur  |Description  |
|---------|---------|---------|
|Nom     | asverify.mystorageaccount        | Cette valeur et l’étiquette du nom du domaine constituent le FQDN pour le nom de domaine personnalisé.        |
|Type     | CNAME        | Utiliser un enregistrement CNAME équivaut à utiliser un alias.        |
|TTL     | 1        | 1 est utilisé pour 1 heure        |
|Unité de durée de vie     | Heures        | Les heures sont utilisées en tant que mesure du temps         |
|Alias     | asverify.adatumfunctiona9ed.blob.core.windows.net        | Nom DNS pour lequel vous créez l’alias. Dans cet exemple, il s’agit du nom DNS asverify.adatumfunctiona9ed.blob.core.windows.net fourni par défaut au compte de stockage.        |

Revenez à votre compte de stockage. Pour ce faire, cliquez sur **Stockage** > **Comptes de stockage** , sélectionnez votre compte de stockage, puis cliquez sur **Domaine personnalisé** . Dans la zone de texte, entrez l’alias que vous avez créé, sans le préfixe asverify. Cochez la case **Utiliser la validation CNAME indirecte** , puis cliquez sur **Enregistrer** . Une fois cette étape accomplie, revenez à votre zone DNS, puis créez un enregistrement CNAME sans le préfixe asverify.  Ensuite, vous pouvez supprimer en toute sécurité l’enregistrement CNAME avec le préfixe cdnverify.

![Capture d’écran montrant la page Domaine personnalisé.](./media/dns-custom-domain/indirectvalidate.png)

Validez la résolution DNS en exécutant `nslookup`.

Pour en savoir plus sur le mappage d’un domaine personnalisé à un point de terminaison Stockage Blob, voir [Configurer un nom de domaine personnalisé pour un point de terminaison de stockage Blob](../storage/blobs/storage-custom-domain-name.md?toc=%dns%2ftoc.json).

## <a name="azure-cdn"></a>Azure CDN

Les étapes suivantes vous guident tout au long de la configuration d’un enregistrement CNAME pour un point de terminaison CDN à l’aide de la méthode cdnverify. Cette méthode garantit l’absence de temps d’arrêt.

Accédez à **Mise en réseau** > **Profils CDN** , et sélectionnez votre profil CDN.

Sélectionnez le point de terminaison que vous utilisez, puis cliquez sur **+ Domaine personnalisé** . Notez la valeur de **Nom d’hôte du point de terminaison** , car il s’agit de l’enregistrement vers lequel pointe l’enregistrement CNAME.

![Domaine personnalisé du CDN](./media/dns-custom-domain/endpointcustomdomain.png)

Accédez à votre zone DNS, puis cliquez sur **+ Jeu d’enregistrements** . Entrez les informations suivantes sur le panneau **Ajouter un jeu d’enregistrements** , puis cliquez sur **OK** pour créer le jeu d’enregistrements.

|Propriété  |Valeur  |Description  |
|---------|---------|---------|
|Nom     | cdnverify.mycdnendpoint        | Cette valeur et l’étiquette du nom du domaine constituent le FQDN pour le nom de domaine personnalisé.        |
|Type     | CNAME        | Utiliser un enregistrement CNAME équivaut à utiliser un alias.        |
|TTL     | 1        | 1 est utilisé pour 1 heure        |
|Unité de durée de vie     | Heures        | Les heures sont utilisées en tant que mesure du temps         |
|Alias     | cdnverify.adatumcdnendpoint.azureedge.net        | Nom DNS pour lequel vous créez l’alias. Dans cet exemple, il s’agit du nom DNS cdnverify.adatumcdnendpoint.azureedge.net fourni par défaut au compte de stockage.        |

Revenez au point de terminaison CDN en cliquant sur **Mise en réseau** > **Profils CDN** , puis sélectionnez votre profil CDN. Cliquez sur **+ Domaine personnalisé** , entrez votre alias d’enregistrement CNAME sans le préfixe cdnverify, puis cliquez sur **Ajouter** .

Une fois cette étape accomplie, revenez à votre zone DNS, puis créez un enregistrement CNAME sans le préfixe cdnverify.  Ensuite, vous pouvez supprimer en toute sécurité l’enregistrement CNAME avec le préfixe cdnverify. Pour plus d’informations sur le CDN et la façon de configurer un domaine personnalisé sans l’étape d’inscription intermédiaire, voir [Mapper du contenu Azure CDN à un domaine personnalisé](../cdn/cdn-map-content-to-custom-domain.md?toc=%dns%2ftoc.json).

## <a name="next-steps"></a>Étapes suivantes

Découvrir comment [Configurer des DNS inversés dans les services hébergés par Azure](dns-reverse-dns-for-azure-services.md).
