---
title: Résolution des problèmes liés à l’état détérioré d’Azure Traffic Manager
description: Comment résoudre les profils Traffic Manager lorsque l’état est affiché comme dégradé.
services: traffic-manager
documentationcenter: ''
author: duongau
manager: kumudD
ms.service: traffic-manager
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/03/2017
ms.author: duau
ms.openlocfilehash: 83dc432a1f88b443d500bf9a977abfed69211156
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003852"
---
# <a name="troubleshooting-degraded-state-on-azure-traffic-manager"></a>Résolution des problèmes liés à l’état détérioré d’Azure Traffic Manager

Cet article décrit comment résoudre les problèmes d’un profil Azure Traffic Manager qui présente un état détérioré. La première étape de la résolution d’un statut de dégradation d’Azure Traffic Manager consiste à activer la journalisation.  Pour plus d’informations, consultez [Activer les journaux de ressources](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-diagnostic-logs). Pour ce scénario, considérez que vous avez configuré un profil Traffic Manager pointant vers certains de vos services hébergés cloudapp.net. Si le statut de l’intégrité de votre Traffic Manager est **Dégradé**, le statut d’un ou plusieurs points de terminaison peut être **Dégradé** :

![statut du point de terminaison dégradé](./media/traffic-manager-troubleshooting-degraded/traffic-manager-degradedifonedegraded.png)

Si le statut de l’intégrité de votre Traffic Manager est **Inactif**, les deux points de terminaison peuvent être **Inactifs** :

![Statut Traffic Manager inactif](./media/traffic-manager-troubleshooting-degraded/traffic-manager-inactive.png)

## <a name="understanding-traffic-manager-probes"></a>Présentation des sondes de Traffic Manager

* Traffic Manager considère qu’un point de terminaison est EN LIGNE uniquement si la sonde reçoit une réponse HTTP 200 en retour du chemin d’accès de la sonde. Si votre application retourne un autre code de réponse HTTP, vous devez ajouter ce code de réponse aux [Plages de codes d’état attendues](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-monitoring#configure-endpoint-monitoring) de votre profil Traffic Manager.
* Une réponse de redirection 30x est traitée comme un échec, sauf si vous avez spécifié qu’il s'agit d’un code de réponse valide dans les [Plages de codes d’état attendues](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-monitoring#configure-endpoint-monitoring) de votre profil Traffic Manager. Traffic Manager n’effectue pas de sondage sur la cible de redirection.
* Pour les sondes HTTPs, les erreurs de certificat sont ignorées.
* Le contenu réel du chemin d’accès de la sonde n’importe pas, aussi longtemps que la valeur retournée est 200. Le sondage d’une URL pour détecter du contenu statique, tel que « /favicon.ico », est une technique courante. Un contenu dynamique, par exemple, des pages ASP, ne retourne pas toujours 200, même quand l’application est intègre.
* La meilleure pratique consiste à définir le chemin d’accès de la sonde vers un élément disposant d’une logique suffisante pour déterminer si le site fonctionne ou est à l’arrêt. Dans l’exemple précédent, en définissant le chemin d’accès « /favicon.ico », vous ne faites que tester le fait que w3wp.exe répond. Cette sonde n’indique pas que votre application web est intègre. Une meilleure option consisterait à définir un chemin d’accès vers un élément tel que « /Probe.aspx » disposant de la logique nécessaire pour déterminer l’intégrité du site. Par exemple, vous pourriez utiliser des compteurs de performances pour l’utilisation du processeur, ou mesurer le nombre de demandes ayant échoué. Vous pourriez également tenter d’accéder aux ressources de base de données ou à l’état de la session pour vous assurer que l’application web fonctionne.
* Si tous les points de terminaison d’un profil sont détériorés, Traffic Manager traite tous les points de terminaison comme intègres, et achemine le trafic vers tous les points de terminaison. Ce comportement garantit que les problèmes liés au mécanisme de sondage n’entraînent pas d’interruption complète de votre service.

## <a name="troubleshooting"></a>Dépannage

Pour résoudre un problème d’échec d’analyse, vous avez besoin d’un outil qui affiche le code d’état HTTP retourné à partir de l’URL de la sonde. Il existe de nombreux outils qui affichent la réponse HTTP brute.

* [Fiddler](https://www.telerik.com/fiddler)
* [curl](https://curl.haxx.se/)
* [wget](http://gnuwin32.sourceforge.net/packages/wget.htm)

En outre, vous pouvez utiliser l’onglet réseau des outils de débogage F12 dans Internet Explorer pour afficher les réponses HTTP.

Pour cet exemple, voulons voir la réponse de l’URL de notre sonde : http:\//watestsdp2008r2.cloudapp.net:80/Probe. L’exemple PowerShell suivant illustre le problème.

```powershell
Invoke-WebRequest 'http://watestsdp2008r2.cloudapp.net/Probe' -MaximumRedirection 0 -ErrorAction SilentlyContinue | Select-Object StatusCode,StatusDescription
```

Exemple de sortie :

```output
StatusCode StatusDescription
---------- -----------------
        301 Moved Permanently
```

Notez que nous avons reçu une réponse redirigée. Comme indiqué précédemment, tout StatusCode autre que 200 est considéré comme un échec. Traffic Manager modifie l’état du point de terminaison en Hors connexion. Pour résoudre le problème, vérifiez la configuration de site web pour vous assurer que le StatusCode approprié peut être retourné à partir du chemin d’accès de la sonde. Reconfigurez la sonde Traffic Manager pour qu’elle pointe vers un chemin d’accès qui renvoie la valeur 200.

Si votre sonde utilise le protocole HTTPS, vous devez peut-être désactiver la vérification du certificat pour éviter les erreurs SSL/TLS pendant votre test. Les instructions PowerShell suivantes désactivent la validation de certificat pour la session PowerShell :

```powershell
add-type @"
using System.Net;
using System.Security.Cryptography.X509Certificates;
public class TrustAllCertsPolicy : ICertificatePolicy {
    public bool CheckValidationResult(
    ServicePoint srvPoint, X509Certificate certificate,
    WebRequest request, int certificateProblem) {
    return true;
    }
}
"@
[System.Net.ServicePointManager]::CertificatePolicy = New-Object TrustAllCertsPolicy
```

## <a name="next-steps"></a>Étapes suivantes

[À propos des méthodes de routage du trafic de Traffic Manager](traffic-manager-routing-methods.md)

[Qu’est-ce que Traffic Manager ?](traffic-manager-overview.md)

[Cloud Services](https://go.microsoft.com/fwlink/?LinkId=314074)

[Azure App Service](https://azure.microsoft.com/documentation/services/app-service/web/)

[Opérations sur Traffic Manager (Référence sur l’API REST)](https://go.microsoft.com/fwlink/?LinkId=313584)

[Applets de commande Azure Traffic Manager][1]

[1]: https://docs.microsoft.com/powershell/module/az.trafficmanager
