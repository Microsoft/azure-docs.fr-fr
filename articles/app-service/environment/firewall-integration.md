---
title: Verrouiller le trafic sortant
description: Découvrez comment s’intégrer au pare-feu Azure pour sécuriser le trafic sortant au sein d’un environnement App Service.
author: ccompy
ms.assetid: 955a4d84-94ca-418d-aa79-b57a5eb8cb85
ms.topic: article
ms.date: 09/24/2020
ms.author: ccompy
ms.custom: seodec18, references_regions
ms.openlocfilehash: e5f9cd361d4f130d725f608614159d67fb7b56d1
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93026358"
---
# <a name="locking-down-an-app-service-environment"></a>Verrouiller un environnement App Service

L’environnement ASE (App Service Environment) présente des dépendances externes auxquelles il doit accéder pour fonctionner correctement. L’environnement ASE se trouve dans le réseau virtuel Azure (VNet) des clients. Les clients doivent autoriser le trafic des dépendances de l’environnement ASE, ce qui représente un problème pour ceux qui souhaitent verrouiller tous les sorties provenant de leur réseau virtuel.

Un certain nombre de points de terminaison entrants sont utilisés pour gérer un environnement ASE. Le trafic de gestion entrant ne peut pas être envoyé via un dispositif de pare-feu. Les adresses sources de ce trafic sont connues et publiées dans le document [Adresses de gestion App Service Environment](./management-addresses.md). Une balise de service nommée AppServiceManagement peut également être utilisée avec les groupes de sécurité réseau afin de sécuriser le trafic entrant.

Les dépendances sortantes de l’environnement ASE sont presque entièrement définies avec des noms FQDN, qui n’ont pas d’adresses statiques derrière eux. L’absence d’adresses statiques signifie que les groupes de sécurité réseau ne peuvent pas être utilisés pour verrouiller le trafic sortant d’un environnement ASE. Les adresses changent assez souvent et il n’est donc pas possible de définir des règles basées sur la résolution actuelle et de les utiliser pour créer des groupes NSG. 

La solution pour sécuriser les adresses sortantes réside dans l’utilisation d’un dispositif de pare-feu pouvant contrôler le trafic sortant en fonction des noms de domaine. Le Pare-feu Azure peut restreindre le trafic HTTP et HTTPS sortant en fonction du nom de domaine complet de la destination.  

## <a name="system-architecture"></a>Architecture du système

Déployer un environnement ASE avec un trafic sortant qui transite par un pare-feu matériel nécessite un changement de routes sur le sous-réseau ASE. Les routes opèrent à un niveau IP. Si vous ne faites pas attention au moment de définir vos routes, vous risquez de forcer le trafic de réponse TCP vers la source d’une autre adresse. Lorsque votre adresse de réponse est différente de l’adresse à laquelle le trafic a été envoyé, le problème est appelé routage asymétrique et interrompt TCP.

Il est indispensable de définir des routes afin que le trafic entrant à destination de l’environnement ASE puisse être retourné de la même façon qu’il est arrivé. Des routes doivent être définies pour les demandes de gestion entrantes et pour les demandes d’application entrantes.

Le trafic à destination et en provenance de l’environnement ASE doit être conforme aux conventions suivantes :

* Le trafic à destination d’Azure SQL, du stockage et d’Event Hub n’est pas pris en charge quand un pare-feu est utilisé. Ce trafic doit être envoyé directement à ces services. Pour cela, il convient de configurer des points de terminaison de service pour ces trois services. 
* Des règles de table de routage doivent être définies de telle sorte que le trafic de gestion entrant soit retourné d’où il est venu.
* Des règles de table de routage doivent être définies de telle sorte que le trafic d’application entrant soit retourné d’où il est venu. 
* Tout le reste du trafic quittant l’environnement ASE peut être envoyé à votre pare-feu matériel sans règle de table de routage.

![Environnement ASE avec le flux de connexion du pare-feu Azure][5]

## <a name="locking-down-inbound-management-traffic"></a>Verrouillage du trafic de gestion entrant

Si votre sous-réseau ASE n’est pas déjà attribué à un groupe de sécurité réseau, créez-en un. Au sein du groupe de sécurité réseau, définissez la première règle pour autoriser le trafic à partir de la balise de service nommée AppServiceManagement sur les ports 454, 455. La règle autorisant l’accès à partir de la balise AppServiceManagement est le seul élément requis par les adresses IP publiques pour gérer votre ASE. Les adresses situées derrière cette balise de service sont uniquement utilisées pour gérer Azure App Service. Le trafic de gestion qui transite via ces connexions est chiffré et sécurisé moyennant des certificats d’authentification. Sur ce canal, le trafic type inclut des éléments tels que des commandes et sondes d'intégrité initiées par le client. 

Les environnements ASE créés via le portail avec un nouveau sous-réseau s'accompagnent d'un groupe de sécurité réseau contenant la règle d’autorisation de la balise AppServiceManagement.  

Votre ASE doit également autoriser les requêtes entrantes à partir de la balise Load Balancer sur le port 16001. Les requêtes provenant de Load Balancer sur le port 16001 sont conservées entre les serveurs frontaux Load Balancer et ASE. Si le port 16001 est bloqué, votre ASE deviendra non sain.

## <a name="configuring-azure-firewall-with-your-ase"></a>Configuration du pare-feu Azure avec votre environnement ASE 

Les étapes pour verrouiller les sorties de votre environnement ASE existant avec le pare-feu Azure sont les suivantes :

1. Activez les points de terminaison de service pour Azure SQL, Stockage et Event Hub sur votre sous-réseau ASE. Pour activer les points de terminaison de service, accédez au portail réseau > sous-réseaux, et sélectionnez Microsoft.EventHub, Microsoft.SQL et Microsoft.Storage dans la liste déroulante des points de terminaison de service. Si vous activez des points de terminaison de service pour Azure SQL, toutes les dépendances à Azure SQL existantes dans vos applications doivent également être configurées avec les points de terminaison de service. 

   ![sélectionner les points de terminaison de service][2]
  
1. Créez un sous-réseau nommé AzureFirewallSubnet dans le réseau virtuel où se trouve votre environnement ASE. Créez le pare-feu Azure en vous aidant des instructions fournies dans la [documentation sur le pare-feu Azure](../../firewall/index.yml).

1. Dans Interface utilisateur du pare-feu Azure > Règles > Collection de règles d’application, sélectionnez Ajouter une collection de règles d’application. Choisissez un nom, une priorité et l’action Autoriser. Dans la section Balises FQDN, entrez un nom, définissez les adresses sources sur * et sélectionnez les balises FQDN App Service Environment et Windows Update. 
   
   ![Ajouter une règle d’application][1]
   
1. Dans Interface utilisateur du pare-feu Azure > Règles > Collection de règles de réseau, sélectionnez Ajouter une collection de règles de réseau. Choisissez un nom, une priorité et l’action Autoriser. Dans la section Règles, sous Adresses IP, entrez un nom, sélectionnez **N’importe lequel** comme protocole, définissez les adresses source et de destination sur *, et définissez les ports sur 123. Cette règle permet au système de synchroniser l’horloge à l’aide de NTP. Créez une autre règle de la même manière en définissant cette fois le port 12000 pour faciliter l’identification des éventuels problèmes système. 

   ![Ajouter une règle de réseau NTP][3]
   
1. Dans Interface utilisateur du pare-feu Azure > Règles > Collection de règles de réseau, sélectionnez Ajouter une collection de règles de réseau. Choisissez un nom, une priorité et l’action Autoriser. Dans la section Règles, sous Balises de service, indiquez un nom, sélectionnez un protocole **Quelconque** , définissez * sur Adresses sources, sélectionnez une balise de service AzureMonitor et définissez les ports sur 80, 443. Cette règle permet au système de fournir à Azure Monitor des informations d'intégrité et des métriques.

   ![Ajouter une règle de réseau de balise de service NTP][6]
   
1. Créez une table de routage avec les adresses de gestion provenant des [adresses de gestion App Service Environment]( ./management-addresses.md) avec un tronçon suivant Internet. Les entrées de la table de routage sont nécessaires pour éviter des problèmes de routage asymétrique. Ajoutez des routes pour les dépendances d’adresse IP indiquées ci-dessous dans les dépendances d’adresse IP avec un tronçon suivant Internet. Ajoutez une route d’appliance virtuelle à votre table de routage pour 0.0.0.0/0 avec comme tronçon suivant l’adresse IP privée de votre pare-feu Azure. 

   ![Créer une table de routage][4]
   
1. Affectez la table de routage que vous avez créée à votre sous-réseau ASE.

#### <a name="deploying-your-ase-behind-a-firewall"></a>Déployer votre environnement ASE derrière un pare-feu

Les étapes à suivre pour déployer votre environnement ASE derrière un pare-feu sont les mêmes que celles pour configurer votre environnement ASE existant avec un pare-feu Azure, sauf que vous devez créer votre sous-réseau ASE, puis effectuer les étapes précédentes. Pour créer votre environnement ASE dans un sous-réseau existant, utilisez un modèle Resource Manager, comme décrit dans [Créer un ASE à l’aide d’un modèle Azure Resource Manager](./create-from-template.md).

## <a name="application-traffic"></a>Trafic des applications 

Les étapes ci-dessus permettent à votre environnement ASE de fonctionner sans problème. Vous devrez tout de même configurer des éléments pour répondre aux besoins de vos applications. Dans un environnement ASE configuré avec le pare-feu Azure, les applications font face à deux problèmes.  

- Les dépendances des applications doivent être ajoutées au pare-feu Azure ou à la table de routage. 
- Les routes doivent être créées pour le trafic des applications afin d’éviter des problèmes de routage asymétrique.

Si vos applications ont des dépendances, celles-ci doivent être ajoutées à votre pare-feu Azure. Créez des règles d’application pour autoriser le trafic HTTP/HTTPS et des règles de réseau pour tout le reste. 

Si vous connaissez la plage d’adresses d’où provient le trafic de demande de vos applications, vous pouvez l’ajouter dans la table de routage qui est affectée à votre sous-réseau ASE. Si la plage d’adresses est grande ou non spécifiée, vous pouvez utiliser une appliance réseau comme la passerelle Application Gateway qui vous donnera une adresse à ajouter à votre table de routage. Pour plus d’informations sur la configuration d’une passerelle d’application avec votre environnement ASE ILB, lisez [Intégration de votre environnement App Service ILB à une passerelle d’application](./integrate-with-application-gateway.md)

Cette utilisation de la passerelle Application Gateway est un exemple de configuration de votre système. Si vous aviez suivi ce chemin, vous auriez dû ajouter une route dans la table de routage du sous-réseau ASE pour permettre le retour direct du trafic envoyé à Application Gateway. 

## <a name="logging"></a>Journalisation 

Le Pare-feu Azure peut envoyer des journaux d’activité aux services Stockage Azure, Event Hub ou Azure Monitor. Pour intégrer votre application avec n’importe quelle destination prise en charge, accédez au portail Pare-feu Azure &gt; Journaux de diagnostic, puis activez les journaux d’activité pour la destination choisie. Si vous intégrez des journaux d’activité Azure Monitor, vous pouvez suivre dans les journaux d’activité tout le trafic envoyé au Pare-feu Azure. Pour voir le trafic refusé, ouvrez votre portail d’espace de travail Log Analytics &gt; Journaux d’activité et entrez une requête comme celle-ci 

```kusto
AzureDiagnostics | where msg_s contains "Deny" | where TimeGenerated >= ago(1h)
```

L’intégration de votre pare-feu Azure à des journaux d’activité Azure Monitor est utile quand vous préparez une application sans connaître toutes ses dépendances. Pour en savoir plus sur les journaux d’activité Azure Monitor, consultez [Analyser les données de journal d’activité dans Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).
 
## <a name="dependencies"></a>Les dépendances

Les informations suivantes sont requises uniquement si vous souhaitez configurer une appliance de pare-feu autre que le pare-feu Azure. 

- Les services compatibles avec les points de terminaison de service doivent être configurés avec des points de terminaison de service.
- Les dépendances d’adresses IP pour le trafic non-HTTP/S (à la fois le trafic TCP et UDP)
- Les points de terminaison HTTP/HTTPS avec des noms FQDN peuvent être placés dans votre dispositif de pare-feu.
- Les points de terminaison HTTP/HTTPS avec des caractères génériques sont des dépendances qui peuvent varier avec votre environnement ASE selon le nombre de qualificateurs. 
- Les dépendances Linux sont uniquement un problème si vous déployez des applications Linux dans votre environnement ASE. Si vous ne déployez pas d’applications Linux dans votre environnement ASE, vous n’avez pas besoin d’ajouter ces adresses à votre pare-feu. 

#### <a name="service-endpoint-capable-dependencies"></a>Dépendances compatibles avec les points de terminaison de service 

| Point de terminaison |
|----------|
| Azure SQL |
| Stockage Azure |
| Azure Event Hub |

#### <a name="ip-address-dependencies"></a>Dépendances des adresses IP

| Point de terminaison | Détails |
|----------| ----- |
| \*:123 | Vérification de l’horloge NTP. Le trafic est vérifié à plusieurs points de terminaison sur le port 123 |
| \*:12000 | Ce port est utilisé pour la supervision système. S’il est bloqué, certains problèmes seront plus difficiles à identifier, mais votre environnement ASE continuera de fonctionner |
| 40.77.24.27:80 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 40.77.24.27:443 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 13.90.249.229:80 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 13.90.249.229:443 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 104.45.230.69:80 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 104.45.230.69:443 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 13.82.184.151:80 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 13.82.184.151:443 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |

Avec un pare-feu Azure, tout ce qui suit est automatiquement configuré avec les balises FQDN. 

#### <a name="fqdn-httphttps-dependencies"></a>Dépendances HTTP/HTTPS FQDN 

| Point de terminaison |
|----------|
|graph.microsoft.com:443 |
|login.live.com:443 |
|login.windows.com:443 |
|login.windows.net:443 |
|login.microsoftonline.com:443 |
|client.wns.windows.com:443 |
|definitionupdates.microsoft.com:443 |
|go.microsoft.com:80 |
|go.microsoft.com:443 |
|www.microsoft.com:80 |
|www.microsoft.com:443 |
|wdcpalt.microsoft.com:443 |
|wdcp.microsoft.com:443 |
|ocsp.msocsp.com:443 |
|ocsp.msocsp.com:80 |
|oneocsp.microsoft.com:80 |
|oneocsp.microsoft.com:443 |
|mscrl.microsoft.com:443 |
|mscrl.microsoft.com:80 |
|crl.microsoft.com:443 |
|crl.microsoft.com:80 |
|www.thawte.com:443 |
|crl3.digicert.com:80 |
|ocsp.digicert.com:80 |
|ocsp.digicert.com:443 |
|csc3-2009-2.crl.verisign.com:80 |
|crl.verisign.com:80 |
|ocsp.verisign.com:80 |
|cacerts.digicert.com:80 |
|azperfcounters1.blob.core.windows.net:443 |
|azurewatsonanalysis-prod.core.windows.net:443 |
|global.metrics.nsatc.net:80 |
|global.metrics.nsatc.net:443 |
|az-prod.metrics.nsatc.net:443 |
|antares.metrics.nsatc.net:443 |
|azglobal-black.azglobal.metrics.nsatc.net:443 |
|azglobal-red.azglobal.metrics.nsatc.net:443 |
|antares-black.antares.metrics.nsatc.net:443 |
|antares-red.antares.metrics.nsatc.net:443 |
|prod.microsoftmetrics.com:443 |
|maupdateaccount.blob.core.windows.net:443 |
|clientconfig.passport.net:443 |
|packages.microsoft.com:443 |
|schemas.microsoft.com:80 |
|schemas.microsoft.com:443 |
|management.core.windows.net:443 |
|management.core.windows.net:80 |
|management.azure.com:443 |
|www.msftconnecttest.com:80 |
|shavamanifestcdnprod1.azureedge.net:443 |
|validation-v2.sls.microsoft.com:443 |
|flighting.cp.wd.microsoft.com:443 |
|dmd.metaservices.microsoft.com:80 |
|admin.core.windows.net:443 |
|prod.warmpath.msftcloudes.com:443 |
|prod.warmpath.msftcloudes.com:80 |
|gcs.prod.monitoring.core.windows.net:80|
|gcs.prod.monitoring.core.windows.net:443|
|azureprofileruploads.blob.core.windows.net:443 |
|azureprofileruploads2.blob.core.windows.net:443 |
|azureprofileruploads3.blob.core.windows.net:443 |
|azureprofileruploads4.blob.core.windows.net:443 |
|azureprofileruploads5.blob.core.windows.net:443 |
|azperfmerges.blob.core.windows.net:443 |
|azprofileruploads1.blob.core.windows.net:443 |
|azprofileruploads10.blob.core.windows.net:443 |
|azprofileruploads2.blob.core.windows.net:443 |
|azprofileruploads3.blob.core.windows.net:443 |
|azprofileruploads4.blob.core.windows.net:443 |
|azprofileruploads6.blob.core.windows.net:443 |
|azprofileruploads7.blob.core.windows.net:443 |
|azprofileruploads8.blob.core.windows.net:443 | 
|azprofileruploads9.blob.core.windows.net:443 |
|azureprofilerfrontdoor.cloudapp.net:443 |
|settings-win.data.microsoft.com:443 |
|maupdateaccount2.blob.core.windows.net:443 |
|maupdateaccount3.blob.core.windows.net:443 |
|dc.services.VisualStudio.com:443 |
|gmstorageprodsn1.blob.core.windows.net:443 |
|gmstorageprodsn1.file.core.windows.net:443 |
|gmstorageprodsn1.queue.core.windows.net:443 |
|gmstorageprodsn1.table.core.windows.net:443 |
|rteventservice.trafficmanager.net:443 |
|ctldl.windowsupdate.com:80 |
|ctldl.windowsupdate.com:443 |
|global-dsms.dsms.core.windows.net:443 |

#### <a name="wildcard-httphttps-dependencies"></a>Dépendances HTTP/HTTPS avec caractères génériques 

| Point de terminaison |
|----------|
|gr-Prod-\*.cloudapp.net:443 |
| \*.management.azure.com:443 |
| \*.update.microsoft.com:443 |
| \*.windowsupdate.microsoft.com:443 |
| \*.identity.azure.net:443 |
| \*.ctldl.windowsupdate.com:80 |
| \*.ctldl.windowsupdate.com:443 |

#### <a name="linux-dependencies"></a>Dépendances Linux 

| Point de terminaison |
|----------|
|wawsinfraprodbay063.blob.core.windows.net:443 |
|registry-1.docker.io:443 |
|auth.docker.io:443 |
|production.cloudflare.docker.com:443 |
|download.docker.com:443 |
|us.archive.ubuntu.com:80 |
|download.mono-project.com:80 |
|packages.treasuredata.com:80|
|security.ubuntu.com:80 |
|oryx-cdn.microsoft.io:443 |
| \*.cdn.mscr.io:443 |
| \*.data.mcr.microsoft.com:443 |
|mcr.microsoft.com:443 |
|\*.data.mcr.microsoft.com:443 |
|packages.fluentbit.io:80 |
|packages.fluentbit.io:443 |
|apt-mo.trafficmanager.net:80 |
|apt-mo.trafficmanager.net:443 |
|azure.archive.ubuntu.com:80 |
|azure.archive.ubuntu.com:443 |
|changelogs.ubuntu.com:80 |
|13.74.252.37:11371 |
|13.75.127.55:11371 |
|13.76.190.189:11371 |
|13.80.10.205:11371 |
|13.91.48.226:11371 |
|40.76.35.62:11371 |
|104.215.95.108:11371 |

## <a name="us-gov-dependencies"></a>Dépendances du gouvernement des États-Unis

Pour les environnements ASE des régions US Gov, suivez les instructions contenues à la section [Configuration du pare-feu Azure avec votre environnement ASE](#configuring-azure-firewall-with-your-ase) de ce document afin de configurer un Pare-feu Azure avec votre environnement ASE.

Si vous souhaitez utiliser un appareil autre que Pare-feu Azure dans US Gov 

* Les services compatibles avec les points de terminaison de service doivent être configurés avec des points de terminaison de service.
* Les points de terminaison HTTP/HTTPS avec des noms FQDN peuvent être placés dans votre dispositif de pare-feu.
* Les points de terminaison HTTP/HTTPS avec des caractères génériques sont des dépendances qui peuvent varier avec votre environnement ASE selon le nombre de qualificateurs.

Linux n’est pas disponible dans les régions US Gov, et n'est dès lors pas répertorié en tant que configuration facultative.

#### <a name="service-endpoint-capable-dependencies"></a>Dépendances compatibles avec les points de terminaison de service ####

| Point de terminaison |
|----------|
| Azure SQL |
| Stockage Azure |
| Azure Event Hub |

#### <a name="ip-address-dependencies"></a>Dépendances des adresses IP

| Point de terminaison | Détails |
|----------| ----- |
| \*:123 | Vérification de l’horloge NTP. Le trafic est vérifié à plusieurs points de terminaison sur le port 123 |
| \*:12000 | Ce port est utilisé pour la supervision système. S’il est bloqué, certains problèmes seront plus difficiles à identifier, mais votre environnement ASE continuera de fonctionner |
| 40.77.24.27:80 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 40.77.24.27:443 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 13.90.249.229:80 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 13.90.249.229:443 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 104.45.230.69:80 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 104.45.230.69:443 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 13.82.184.151:80 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |
| 13.82.184.151:443 | Nécessaire pour superviser et alerter sur les problèmes de l’environnement ASE |

#### <a name="dependencies"></a>Les dépendances ####

| Point de terminaison |
|----------|
| \*.ctldl.windowsupdate.com:80 |
| \*.management.usgovcloudapi.net:80 |
| \*.update.microsoft.com:80 |
|admin.core.usgovcloudapi.net:80 |
|azperfmerges.blob.core.windows.net:80 |
|azperfmerges.blob.core.windows.net:80 |
|azprofileruploads1.blob.core.windows.net:80 |
|azprofileruploads10.blob.core.windows.net:80 |
|azprofileruploads2.blob.core.windows.net:80 |
|azprofileruploads3.blob.core.windows.net:80 |
|azprofileruploads4.blob.core.windows.net:80 |
|azprofileruploads5.blob.core.windows.net:80 |
|azprofileruploads6.blob.core.windows.net:80 |
|azprofileruploads7.blob.core.windows.net:80 |
|azprofileruploads8.blob.core.windows.net:80 |
|azprofileruploads9.blob.core.windows.net:80 |
|azureprofilerfrontdoor.cloudapp.net:80 |
|azurewatsonanalysis.usgovcloudapp.net:80 |
|cacerts.digicert.com:80 |
|client.wns.windows.com:80 |
|crl.microsoft.com:80 |
|crl.verisign.com:80 |
|crl3.digicert.com:80 |
|csc3-2009-2.crl.verisign.com:80 |
|ctldl.windowsupdate.com:80 |
|definitionupdates.microsoft.com:80 |
|download.windowsupdate.com:80 |
|fairfax.warmpath.usgovcloudapi.net:80 |
|flighting.cp.wd.microsoft.com:80 |
|gcwsprodgmdm2billing.queue.core.usgovcloudapi.net:80 |
|gcwsprodgmdm2billing.table.core.usgovcloudapi.net:80 |
|global.metrics.nsatc.net:80 |
|go.microsoft.com:80 |
|gr-gcws-prod-bd3.usgovcloudapp.net:80 |
|gr-gcws-prod-bn1.usgovcloudapp.net:80 |
|gr-gcws-prod-dd3.usgovcloudapp.net:80 |
|gr-gcws-prod-dm2.usgovcloudapp.net:80 |
|gr-gcws-prod-phx20.usgovcloudapp.net:80 |
|gr-gcws-prod-sn5.usgovcloudapp.net:80 |
|login.live.com:80 |
|login.microsoftonline.us:80 |
|management.core.usgovcloudapi.net:80 |
|management.usgovcloudapi.net:80 |
|maupdateaccountff.blob.core.usgovcloudapi.net:80 |
|mscrl.microsoft.com:80
|ocsp.digicert.com:80 |
|ocsp.verisign.com:80 |
|rteventse.trafficmanager.net:80 |
|settings-n.data.microsoft.com:80 |
|shavamafestcdnprod1.azureedge.net:80 |
|shavanifestcdnprod1.azureedge.net:80 |
|v10ortex-win.data.microsoft.com:80 |
|wp.microsoft.com:80 |
|dcpalt.microsoft.com:80 |
|www.microsoft.com:80 |
|www.msftconnecttest.com:80 |
|www.thawte.com:80 |
|\*ctldl.windowsupdate.com:443 |
|\*.management.usgovcloudapi.net:443 |
|\*.update.microsoft.com:443 |
|admin.core.usgovcloudapi.net:443 |
|azperfmerges.blob.core.windows.net:443 |
|azperfmerges.blob.core.windows.net:443 |
|azprofileruploads1.blob.core.windows.net:443 |
|azprofileruploads10.blob.core.windows.net:443 |
|azprofileruploads2.blob.core.windows.net:443 |
|azprofileruploads3.blob.core.windows.net:443 |
|azprofileruploads4.blob.core.windows.net:443 |
|azprofileruploads5.blob.core.windows.net:443 |
|azprofileruploads6.blob.core.windows.net:443 |
|azprofileruploads7.blob.core.windows.net:443 |
|azprofileruploads8.blob.core.windows.net:443 |
|azprofileruploads9.blob.core.windows.net:443 |
|azureprofilerfrontdoor.cloudapp.net:443 |
|azurewatsonanalysis.usgovcloudapp.net:443 |
|cacerts.digicert.com:443 |
|client.wns.windows.com:443 |
|crl.microsoft.com:443 |
|crl.verisign.com:443 |
|crl3.digicert.com:443 |
|csc3-2009-2.crl.verisign.com:443 |
|ctldl.windowsupdate.com:443 |
|definitionupdates.microsoft.com:443 |
|download.windowsupdate.com:443 |
|fairfax.warmpath.usgovcloudapi.net:443 |
|gcs.monitoring.core.usgovcloudapi.net:443 |
|flighting.cp.wd.microsoft.com:443 |
|gcwsprodgmdm2billing.queue.core.usgovcloudapi.net:443 |
|gcwsprodgmdm2billing.table.core.usgovcloudapi.net:443 |
|global.metrics.nsatc.net:443 |
|go.microsoft.com:443 |
|gr-gcws-prod-bd3.usgovcloudapp.net:443 |
|gr-gcws-prod-bn1.usgovcloudapp.net:443 |
|gr-gcws-prod-dd3.usgovcloudapp.net:443 |
|gr-gcws-prod-dm2.usgovcloudapp.net:443 |
|gr-gcws-prod-phx20.usgovcloudapp.net:443 |
|gr-gcws-prod-sn5.usgovcloudapp.net:443 |
|login.live.com:443 |
|login.microsoftonline.us:443 |
|management.core.usgovcloudapi.net:443 |
|management.usgovcloudapi.net:443 |
|maupdateaccountff.blob.core.usgovcloudapi.net:443 |
|mscrl.microsoft.com:443 |
|ocsp.digicert.com:443 |
|ocsp.msocsp.com:443 |
|ocsp.msocsp.com:80 |
|oneocsp.microsoft.com:80 |
|oneocsp.microsoft.com:443 |
|ocsp.verisign.com:443 |
|rteventservice.trafficmanager.net:443 |
|settings-win.data.microsoft.com:443 |
|shavamanifestcdnprod1.azureedge.net:443 |
|shavamanifestcdnprod1.azureedge.net:443 |
|v10.vortex-win.data.microsoft.com:443 |
|wdcp.microsoft.com:443 |
|wdcpalt.microsoft.com:443 |
|www.microsoft.com:443 |
|www.msftconnecttest.com:443 |
|www.thawte.com:443 |
|global-dsms.dsms.core.usgovcloudapi.net:443 |

<!--Image references-->
[1]: ./media/firewall-integration/firewall-apprule.png
[2]: ./media/firewall-integration/firewall-serviceendpoints.png
[3]: ./media/firewall-integration/firewall-ntprule.png
[4]: ./media/firewall-integration/firewall-routetable.png
[5]: ./media/firewall-integration/firewall-topology.png
[6]: ./media/firewall-integration/firewall-ntprule-monitor.png
