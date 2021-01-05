---
title: Prévention des acquisitions de sous-domaines avec les enregistrements d’alias Azure DNS et la vérification du domaine personnalisé Azure App Service
description: Découvrez comment éviter la menace courante et de gravité élevée que représente l’acquisition de sous-domaine.
services: security
author: memildin
manager: rkarlin
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/29/2020
ms.author: memildin
ms.openlocfilehash: 7c09a7f6c6a313852fc6212c6190a584ba5f67bd
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94409890"
---
# <a name="prevent-dangling-dns-entries-and-avoid-subdomain-takeover"></a>Prévention des entrées DNS non résolues et de l’acquisition de sous-domaine

Cet article décrit la menace de sécurité courante que représente l’acquisition de sous-domaine et la procédure à suivre pour y remédier.


## <a name="what-is-subdomain-takeover"></a>Qu’est-ce que l’acquisition de sous-domaine ?

Les acquisitions de sous-domaines constituent une menace courante et de gravité élevée pour les organisations qui créent et suppriment régulièrement de nombreuses ressources. Une prise de contrôle de sous-domaine peut se produire quand un [enregistrement DNS](../../dns/dns-zones-records.md#dns-records) pointe vers une ressource Azure déprovisionnée. Ces enregistrements DNS sont également appelés entrées « DNS non résolues ». Les enregistrements CNAME sont particulièrement vulnérables à cette menace. Les prises de contrôle de sous-domaines permettent à des acteurs malveillants de rediriger le trafic destiné au domaine d’une organisation vers un site effectuant une activité malveillante.

Voici un cas de figure courant d’acquisition de sous-domaine :

1. **CRÉATION :**

    1. Vous approvisionnez une ressource Azure avec un nom de domaine complet (FQDN) de `app-contogreat-dev-001.azurewebsites.net`.

    1. Vous attribuez un enregistrement CNAME dans votre zone DNS avec le sous-domaine `greatapp.contoso.com` qui route le trafic vers votre ressource Azure.

1. **DÉPPROVISIONNEMENT :**

    1. La ressource Azure est déprovisionnée ou supprimée une fois qu’elle n’est plus nécessaire. 
    
        À ce stade, l’enregistrement CNAME `greatapp.contoso.com` *devrait* être supprimé de votre zone DNS. Si l’enregistrement CNAME n’est pas supprimé, il est publié en tant que domaine actif, mais ne route pas le trafic vers une ressource Azure active. Ceci est la définition d’un enregistrement DNS « non résolu ».

    1. Le sous-domaine non résolu, `greatapp.contoso.com`, est désormais vulnérable, et il est possible d’en prendre le contrôle en l’attribuant à une autre ressource de l’abonnement Azure.

1. **PRISE DE CONTRÔLE :**

    1. À l’aide de méthodes et d’outils couramment disponibles, un acteur de menace découvre le sous-domaine non résolu.  

    1. L’acteur de menace configure une ressource Azure avec le nom de domaine complet de la ressource que vous avez précédemment contrôlée. Dans cet exemple : `app-contogreat-dev-001.azurewebsites.net`.

    1. Le trafic envoyé au sous-domaine `greatapp.contoso.com` est désormais routé vers la ressource de l’acteur malveillant où celui-ci contrôle le contenu.



![Acquisition de sous-domaine à partir d’un site web déprovisionné](./media/subdomain-takeover/subdomain-takeover.png)



## <a name="the-risks-of-subdomain-takeover"></a>Les risques de l’acquisition de sous-domaine

Un enregistrement DNS qui pointe vers une ressource non disponible aurait dû être supprimé de la zone DNS. Dans le cas contraire, il s’agit d’un enregistrement « DNS non résolu », ce qui laisse la possibilité d’une acquisition de sous-domaine.

Les entrées DNS non résolues permettent aux auteurs de menaces de prendre le contrôle du nom DNS associé pour héberger un service ou un site web malveillant. La présence de pages et de services malveillants sur le sous-domaine d’une organisation pourrait avoir les répercussions suivantes :

- **Perte de contrôle sur le contenu du sous-domaine** : mauvaise presse sur l’incapacité de votre organisation à sécuriser son contenu, ainsi que préjudice à la marque et perte de confiance.

- **Collecte de cookies à l’insu des visiteurs** : il est courant que les applications web exposent les cookies de session aux sous-domaines (*.contoso.com) ; par conséquent, n’importe quel sous-domaine peut y accéder. Les auteurs de menaces peuvent utiliser l’acquisition de sous-domaine pour créer une page d’apparence authentique, inciter les utilisateurs non avertis à la visiter et collecter leurs cookies (même sécurisés). Selon une idée reçue, le recours à des certificats SSL protègerait votre site, ainsi que les cookies de vos utilisateurs, de l’acquisition. Or, un auteur de menace peut utiliser le sous-domaine détourné pour demander et recevoir un certificat SSL valide. Celui-ci lui accorde l’accès aux cookies sécurisés et peut augmenter encore la légitimité perçue du site malveillant.

- **Campagnes d’hameçonnage** : des sous-domaines d’apparence authentique pourraient être utilisés dans des campagnes d’hameçonnage. C’est vrai pour les sites malveillants, ainsi que pour les enregistrements MX qui permettent à l’auteur de menace de recevoir des e-mails adressés à un sous-domaine légitime d’une marque fiable connue.

- **Autres risques** : les sites malveillants peuvent servir de base à d’autres attaques classiques (XSS, CSRF, contournement CORS, etc.).



## <a name="identify-dangling-dns-entries"></a>Identifier les entrées DNS non résolues

Pour identifier les entrées DNS au sein de votre organisation qui pourraient être non résolues, utilisez l’outil PowerShell hébergé sur GitHub de Microsoft [« Get-DanglingDnsRecords »](https://aka.ms/DanglingDNSDomains).

Cet outil aide les clients Azure à répertorier tous les domaines avec un enregistrement CNAME associé à une ressource Azure existante qui a été créée sur leurs abonnements ou locataires.

Si vos CNAME se trouvent dans d’autres services DNS et pointent vers des ressources Azure, fournissez les CNAME à l’outil dans un fichier d’entrée.

L’outil prend en charge les ressources Azure répertoriées dans le tableau suivant. L’outil extrait, ou prend en entrée, tous les CNAME du locataire.


| Service                   | Type                                        | FQDNproperty                               | Exemple                         |
|---------------------------|---------------------------------------------|--------------------------------------------|---------------------------------|
| Azure Front Door          | microsoft.network/frontdoors                | properties.cName                           | `abc.azurefd.net`               |
| Stockage Blob Azure        | microsoft.storage/storageaccounts           | properties.primaryEndpoints.blob           | `abc. blob.core.windows.net`    |
| Azure CDN                 | microsoft.cdn/profiles/endpoints            | properties.hostName                        | `abc.azureedge.net`             |
| Adresses IP publiques       | microsoft.network/publicipaddresses         | properties.dnsSettings.fqdn                | `abc.EastUs.cloudapp.azure.com` |
| Azure Traffic Manager     | microsoft.network/trafficmanagerprofiles    | properties.dnsConfig.fqdn                  | `abc.trafficmanager.net`        |
| Azure Container Instance  | microsoft.containerinstance/containergroups | properties.ipAddress.fqdn                  | `abc.EastUs.azurecontainer.io`  |
| Gestion des API Azure      | microsoft.apimanagement/service             | properties.hostnameConfigurations.hostName | `abc.azure-api.net`             |
| Azure App Service         | microsoft.web/sites                         | properties.defaultHostName                 | `abc.azurewebsites.net`         |
| Azure App Service – Emplacements | microsoft.web/sites/slots                   | properties.defaultHostName                 | `abc-def.azurewebsites.net`     |



### <a name="prerequisites"></a>Prérequis

Exécutez la requête en tant qu’utilisateur disposant des privilèges suivants :

- au moins l’accès de niveau lecteur aux abonnements Azure
- accès en lecture au graphe des ressources Azure

Si vous êtes un administrateur général du locataire de votre organisation, élevez votre compte pour avoir accès à l’ensemble de l’abonnement de votre organisation en suivant les instructions fournies dans [Élever l’accès pour gérer tous les abonnements et groupes d’administration Azure](../../role-based-access-control/elevate-access-global-admin.md).


> [!TIP]
> Azure Resource Graph présente des limitations et des limites de pagination que vous devez prendre en compte si vous disposez d’un environnement Azure volumineux. 
> 
> [En savoir plus sur la gestion de gros jeux de données de ressources Azure](../../governance/resource-graph/concepts/work-with-data.md).
> 
> L’outil utilise un traitement par lot d’abonnements pour éviter ces limitations.

### <a name="run-the-script"></a>Exécuter le script

En savoir plus sur le script PowerShell, **Get-DanglingDnsRecords. ps1** , et le télécharger à partir de GitHub : https://aka.ms/DanglingDNSDomains.

## <a name="remediate-dangling-dns-entries"></a>Corriger les entrées de DNS non-résolu 

Examinez vos zones DNS et identifiez les enregistrements CNAME non résolus ou dont vous n’avez plus le contrôle. Si vous trouvez des sous-domaines non résolus ou dont vous avez perdu le contrôle, supprimez les sous-domaines vulnérables et atténuez les risques en procédant comme suit :

1. À partir de votre zone DNS, supprimez tous les enregistrements CNAME qui pointent vers des FQDN de ressources qui ne sont plus approvisionnées.

1. Pour permettre le routage du trafic vers des ressources dont vous avez le contrôle, approvisionnez des ressources supplémentaires avec les FQDN spécifiés dans les enregistrements CNAME des sous-domaines non résolus.

1. Examinez le code de votre application pour voir s’il contient des références à des sous-domaines spécifiques, puis mettez à jour toute référence à un sous-domaine incorrecte ou obsolète.

1. Examinez si une compromission s’est produite, et prenez des mesures conformément aux procédures de réponse aux incidents de votre organisation. Vous trouverez ci-dessous des conseils et des meilleures pratiques pour examiner ce problème.

    Si la logique de votre application est telle que des secrets comme des informations d’identification OAuth ont été envoyées au sous-domaine non résolu, ou si des informations sensibles en lien avec la confidentialité ont été envoyées aux sous-domaines non résolus, il se peut que ces données aient été exposées à des tiers.

1. Comprenez pourquoi l’enregistrement CNAME n’a pas été supprimé de votre zone DNS lorsque la ressource a été déprovisionnée, et prenez les mesures nécessaires pour garantir que les enregistrements DNS seront mis à jour de manière appropriée lors du déprovisionnement de ressources Azure à l’avenir.


## <a name="prevent-dangling-dns-entries"></a>Empêcher les entrées de DNS non résolu

Il est crucial, dans le cadre du programme de sécurité d’une organisation, d’implémenter des processus visant à empêcher les entrées DNS non résolues et les acquisitions de sous-domaines résultantes.

Certains services Azure offrent des fonctionnalités pour vous aider à mettre en place des mesures préventives. Ils sont détaillés ci-dessous. D’autres méthodes pour éviter ce problème doivent être établies par le biais des meilleures pratiques de votre organisation ou de procédures d’exploitation standard.


### <a name="use-azure-dns-alias-records"></a>Enregistrements d’alias Azure DNS

Les [enregistrements d’alias](../../dns/dns-alias.md#scenarios) Azure DNS empêchent les références non résolues en associant le cycle de vie d’un enregistrement DNS à une ressource Azure. Prenons l'exemple d'un enregistrement DNS qualifié en tant qu'enregistrement d'alias et pointant vers une adresse IP publique ou un profil Traffic Manager. Si vous supprimez ces ressources sous-jacentes, l’enregistrement d’alias DNS devient un jeu d’enregistrements vide. Il ne fait plus référence à la ressource supprimée. Il est important de noter qu’il existe des limites à la protection offerte par les enregistrements d’alias. La liste se limite actuellement aux éléments suivants :

- Azure Front Door
- Profils Traffic Manager
- Points de terminaison Azure Content Delivery Network (CDN)
- Adresses IP publiques

Malgré les offres de service limitées à l’heure actuelle, nous vous recommandons d’utiliser des enregistrements d’alias pour vous défendre dans la mesure du possible contre les acquisitions de sous-domaines.

[En savoir plus sur les fonctionnalités des enregistrements d’alias Azure DNS](../../dns/dns-alias.md#capabilities).



### <a name="use-azure-app-services-custom-domain-verification"></a>Vérification du domaine personnalisé Azure App Service

Lors de la création d’entrées DNS pour Azure App Service, créez un enregistrement TXT asuid.{sous-domaine} avec l’ID de vérification du domaine. Lorsqu’il existe un enregistrement TXT de ce type, aucun autre abonnement Azure ne peut valider le domaine personnalisé, ni donc l’acquérir. 

Ces enregistrements n’empêchent pas de créer le service Azure App Service du même nom que celui qui se trouve dans votre entrée CNAME. Sans moyen de prouver la propriété du nom de domaine, les auteurs de menaces ne peuvent pas recevoir de trafic ni contrôler le contenu.

[En savoir plus sur le mappage d’un nom DNS personnalisé existant à Azure App Service](../../app-service/app-service-web-tutorial-custom-domain.md).



### <a name="build-and-automate-processes-to-mitigate-the-threat"></a>Création et automatisation de processus pour atténuer la menace

Il incombe souvent aux développeurs et aux équipes d’exploitation d’exécuter des processus de nettoyage pour éviter les menaces liées aux noms DNS non résolus. Les pratiques ci-dessous vous aideront à faire en sorte que votre organisation ne subisse pas cette menace. 

- **Créer des procédures de prévention :**

    - Apprenez à vos développeurs d’applications à réacheminer les adresses chaque fois qu’ils suppriment des ressources.

    - Placez « Supprimer l’entrée DNS » dans la liste des vérifications requises lors de la désactivation d’un service.

    - Placez des [verrous de suppression](../../azure-resource-manager/management/lock-resources.md) sur les ressources qui comportent une entrée DNS personnalisée. Un verrou de suppression indique que le mappage doit être supprimé avant le déprovisionnement de la ressource. Les mesures comme celles-ci ne peuvent fonctionner qu’associées à des programmes de formation internes.

- **Créer des procédures de découverte :**

    - Vérifiez régulièrement vos enregistrements DNS pour contrôler que vos sous-domaines sont tous mappés à des ressources Azure qui remplissent les conditions suivantes :

        - Les ressources existent : interrogez vos zones DNS pour connaître les ressources pointant vers des sous-domaines Azure tels que *.azurewebsites.net ou *.cloudapp.azure.com (voir la [Liste de références des domaines Azure](azure-domains.md)).
        - Vous en êtes le propriétaire : vérifiez que vous possédez toutes les ressources que vos sous-domaines DNS ciblent.

    - Gérez un catalogue de services de vos points de terminaison de nom de domaine complet (FQDN) Azure et des propriétaires d’applications. Pour créer votre catalogue de services, exécutez le script de requête Azure Resource Graph suivant. Ce script projette les informations des points de terminaison de FQDN des ressources auxquelles vous avez accès et les génère dans un fichier CSV. Si vous avez accès à tous les abonnements de votre locataire, le script les prend tous en compte, comme indiqué dans l’exemple de script suivant. Pour restreindre les résultats à un ensemble spécifique d’abonnements, modifiez le script comme indiqué.


- **Créer des procédures de correction :**
    - Lorsque des entrées DNS non résolues sont trouvées, votre équipe doit déterminer si une compromission s’est produite.
    - Recherchez la raison pour laquelle l’adresse n’a pas été redirigée lorsque la ressource a été retirée.
    - Supprimez l’enregistrement DNS s’il n’est plus utilisé ou faites-le pointer vers la bonne ressource (FQDN) Azure détenue par votre organisation.
 

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les services connexes et les fonctionnalités Azure que vous pouvez utiliser pour vous défendre contre l’acquisition de sous-domaine, consultez les pages suivantes.

- [Empêcher les enregistrements DNS non résolus avec Azure DNS](../../dns/dns-alias.md#prevent-dangling-dns-records)

- [Utiliser un ID de vérification du domaine lors de l’ajout de domaines personnalisés dans Azure App Service](../../app-service/app-service-web-tutorial-custom-domain.md#get-a-domain-verification-id)

- [Démarrage rapide : Exécution d’une requête Resource Graph à l’aide d’Azure PowerShell](../../governance/resource-graph/first-query-powershell.md)
