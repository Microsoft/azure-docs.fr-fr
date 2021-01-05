---
title: Comprendre et résoudre les problèmes CORS de proxy d’application Azure AD
description: Fournit des informations sur CORS dans le proxy d’application Azure AD et la manière d’identifier et de résoudre les problèmes CORS.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 05/23/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 8836295e9f54260c4e9ff6c1da333ef2a86d58fb
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94651853"
---
# <a name="understand-and-solve-azure-active-directory-application-proxy-cors-issues"></a>Comprendre et résoudre les problèmes CORS dans le proxy d’application Azure Active Directory

Le [partage de ressources cross-origin (CORS)](https://www.w3.org/TR/cors/) peut présenter parfois des difficultés pour les applications et les API que vous publiez via le proxy d’application Azure Active Directory. Cet article traite des problèmes CORS du proxy d’application Azure AD et des solutions à ces problèmes.

La sécurité du navigateur empêche généralement une page web d’adresser des demandes AJAX à un autre domaine. Cette restriction est appelée *stratégie de même origine* et empêche un site malveillant de lire des données sensibles à partir d’un autre site. Toutefois, vous souhaitez parfois laisser d’autres sites appeler votre API web. CORS est une norme W3C qui permet à un serveur d’assouplir la stratégie de même origine et d’autoriser certaines demandes cross-origin tout en en rejetant d’autres.

## <a name="understand-and-identify-cors-issues"></a>Comprendre et identifier les problèmes CORS

Deux URL ont la même origine si elles ont des schémas, des hôtes et des ports ([RFC 6454](https://tools.ietf.org/html/rfc6454)) identiques, telles que :

-   http:\//contoso.com/foo.html
-   http:\//contoso.com/bar.html

Les URL suivantes ont des origines différentes des deux précédentes :

-   http:\//contoso.net - Domaine différent
-   http:\//contoso.com:9000/foo.html - Port différent
-   https:\//contoso.com/foo.html - Schéma différent
-   http:\//www.contoso.com/foo.html - Sous-domaine différent

La stratégie de même origine empêche des applications d’accéder aux ressources à partir d’autres origines, sauf si elles utilisent les bons en-têtes de contrôle d’accès. Si les en-têtes CORS sont absents ou incorrects, les demandes cross-origin échouent. 

Vous pouvez identifier les problèmes CORS à l’aide des outils de débogage de navigateur :

1. Lancez le navigateur et accédez à l’application web.
1. Appuyez sur **F12** pour afficher la console de débogage.
1. Essayez de reproduire la transaction et passez en revue le message de la console. Une violation de CORS génère une erreur de la console relative à l’origine.

Dans la capture d’écran suivante, la sélection du bouton **Try It** a entraîné l’affichage d’un message d’erreur CORS indiquant que https:\//corswebclient-contoso.msappproxy.net est introuvable dans l’en-tête Access-Control-Allow-Origin.

![Problème CORS](./media/application-proxy-understand-cors-issues/image3.png)

## <a name="cors-challenges-with-application-proxy"></a>Défis CORS avec le proxy d’application

L’exemple suivant montre un scénario CORS de proxy d’application Azure AD standard. Le serveur interne héberge un contrôleur d’API web **CORSWebService** et un **CORSWebClient** qui appelle **CORSWebService**. Il existe une demande AJAX de **CORSWebClient** à **CORSWebService**.

![Demande de même origine en local](./media/application-proxy-understand-cors-issues/image1.png)

L’application CORSWebClient fonctionne lorsque vous l’hébergez localement, mais échoue lors de son chargement ou génère une erreur lors de sa publication via le proxy d’application Azure AD. Si vous avez publié les applications CORSWebClient et CORSWebService séparément en tant qu’applications différentes via le proxy d’application, ces deux applications sont hébergées dans des domaines différents. Une demande AJAX de CORSWebClient à CORSWebService est une demande cross-origin et elle échoue.

![Demande CORS de proxy d’application](./media/application-proxy-understand-cors-issues/image2.png)

## <a name="solutions-for-application-proxy-cors-issues"></a>Solutions aux problèmes CORS de proxy d’application

Vous pouvez résoudre le problème CORS précédent de plusieurs façons différentes.

### <a name="option-1-set-up-a-custom-domain"></a>Option 1 : Configurer un domaine personnalisé

Utilisez un [domaine personnalisé](./application-proxy-configure-custom-domain.md) de proxy d’application Azure AD pour publier à partir de la même origine sans avoir à apporter de modifications aux origines, au code ni aux en-têtes de l’application. 

### <a name="option-2-publish-the-parent-directory"></a>Option n°2 : Publier le répertoire parent

Publiez le répertoire parent des deux applications. Cette solution fonctionne particulièrement bien si vous avez seulement deux applications sur le serveur web. Au lieu de publier chaque application séparément, vous pouvez publier le répertoire parent commun, ce qui se traduit par la même origine.

Les exemples suivants montrent la page de proxy d’application Azure AD du portail pour l’application CORSWebClient.  Lorsque l’**URL interne** a la valeur *contoso.com/CORSWebClient*, l’application ne peut pas effectuer de demandes réussies au répertoire *contoso.com/CORSWebService*, car ils sont cross-origin. 

![Publier l’application individuellement](./media/application-proxy-understand-cors-issues/image4.png)

Au lieu de cela, définissez l’**URL interne** pour publier le répertoire parent, qui inclut les deux répertoires *CORSWebClient* et *CORSWebService* :

![Publier le répertoire parent](./media/application-proxy-understand-cors-issues/image5.png)

Les URL d’application obtenues résolvent efficacement le problème CORS :

- https:\//corswebclient-contoso.msappproxy.net/CORSWebService
- https:\//corswebclient-contoso.msappproxy.net/CORSWebClient

### <a name="option-3-update-http-headers"></a>Option 3 : Mettre à jour les en-têtes HTTP

Ajoutez un en-tête de réponse HTTP personnalisé sur le service web pour établir une correspondance avec la demande d’origine. Pour les sites web en cours d’exécution dans Internet Information Services (IIS), utilisez le Gestionnaire IIS pour modifier l’en-tête :

![Ajouter un en-tête de réponse personnalisée dans le Gestionnaire IIS](./media/application-proxy-understand-cors-issues/image6.png)

Cette modification ne nécessite aucune modification de code. Vous pouvez la vérifier dans les traces Fiddler :

**Publier l’ajout de l’en-tête**\
HTTP/1.1 200 OK\
Cache-Control : no-cache\
Pragma: no-cache\
Content-Type: text/plain; charset=utf-8\
Expires: -1\
Vary: Accept-Encoding\
Serveur :  Microsoft-IIS/8.5 Microsoft-HTTPAPI/2.0\
**Access-Control-Allow-Origin: https\://corswebclient-contoso.msappproxy.net**\
X-AspNet-Version: 4.0.30319\
X-Powered-By: ASP.NET\
Content-Length: 17

### <a name="option-4-modify-the-app"></a>Option 4 : Modifier l’application

Vous pouvez modifier votre application pour prendre en charge CORS en ajoutant l’en-tête Access-Control-Allow-Origin avec les valeurs appropriées. La façon d’ajouter l’en-tête dépend du langage de code de l’application. La modification du code est l’option la moins recommandée, car elle nécessite le plus d’efforts.

### <a name="option-5-extend-the-lifetime-of-the-access-token"></a>Option 5 : Étendre la durée de vie du jeton d’accès

Certains problèmes CORS ne peuvent pas être résolus ; par exemple, lorsque votre application redirige vers *login.microsoftonline.com* pour l’authentification et que le jeton d’accès expire. L’appel CORS échoue alors. Une solution de contournement pour ce scénario consiste à étendre la durée de vie du jeton d’accès, pour éviter qu’il arrive à expiration lors d’une session utilisateur. Pour plus d’informations sur la procédure à suivre, consultez [Durées de vie de jeton configurables dans Azure AD](../develop/active-directory-configurable-token-lifetimes.md).

## <a name="see-also"></a>Voir aussi
- [Tutoriel : Ajouter une application locale pour un accès à distance via le proxy d’application d’Azure Active Directory](application-proxy-add-on-premises-application.md) 
- [Planifier un déploiement du proxy d’application Azure AD](application-proxy-deployment-plan.md) 
- [Accès à distance aux applications locales via le proxy d’application Azure Active Directory](application-proxy.md)