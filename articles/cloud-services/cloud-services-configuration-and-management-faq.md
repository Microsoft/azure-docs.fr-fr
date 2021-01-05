---
title: FAQ sur les problèmes de configuration et de gestion
titleSuffix: Azure Cloud Services
description: Cet article répertorie les questions fréquentes sur la configuration et la gestion pour Microsoft Azure Cloud Services.
services: cloud-services
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 84985660-2cfd-483a-8378-50eef6a0151d
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/23/2018
ms.author: genli
ms.openlocfilehash: c4497805e64ef303c9d7340c48a49027b3a26bef
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011017"
---
# <a name="configuration-and-management-issues-for-azure-cloud-services-frequently-asked-questions-faqs"></a>Problèmes de configuration et de gestion pour Azure Cloud Services : Forum Aux Questions (FAQ)

Cet article comprend des questions fréquentes sur les problèmes de configuration et de gestion pour [Microsoft Azure Cloud Services](https://azure.microsoft.com/services/cloud-services). Vous pouvez également consulter la page [Taille de services cloud](cloud-services-sizes-specs.md) pour obtenir des informations sur la taille.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

**Certificates**

- [Pourquoi la chaîne d’approbation du certificat TLS/SSL de mon service cloud est-elle incomplète ?](#why-is-the-certificate-chain-of-my-cloud-service-tlsssl-certificate-incomplete)
- [À quoi sert le « certificat de chiffrement Windows Azure Tools pour les extensions » ?](#what-is-the-purpose-of-the-windows-azure-tools-encryption-certificate-for-extensions)
- [Comment générer une demande de signature de certificat sans se connecter à l’instance via le protocole RDP ?](#how-can-i-generate-a-certificate-signing-request-csr-without-rdp-ing-in-to-the-instance)
- [Le certificat de gestion de mon service cloud arrive à expiration. Comment le renouveler ?](#my-cloud-service-management-certificate-is-expiring-how-to-renew-it)
- [Comment automatiser l’installation d’un certificat TLS/SSL principal et d’un certificat intermédiaire (.p7b) ?](#how-to-automate-the-installation-of-main-tlsssl-certificatepfx-and-intermediate-certificatep7b)
- [Qu’est l’objectif du certificat « Microsoft Azure Service Management pour MachineKey » ?](#what-is-the-purpose-of-the-microsoft-azure-service-management-for-machinekey-certificate)

**Surveillance et journalisation**

- [Quelles prochaines fonctionnalités Service cloud dans le portail Azure peuvent aider à gérer et surveiller des applications ?](#what-are-the-upcoming-cloud-service-capabilities-in-the-azure-portal-which-can-help-manage-and-monitor-applications)
- [Pourquoi IIS cesse-t-il d’écrire dans le répertoire des journaux ?](#why-does-iis-stop-writing-to-the-log-directory)
- [Comment activer la journalisation des diagnostics Microsoft Azure pour les Services cloud ?](#how-do-i-enable-wad-logging-for-cloud-services)

**Configuration réseau**

- [Comment définir le délai d’inactivité d’Azure Load Balancer ?](#how-do-i-set-the-idle-timeout-for-azure-load-balancer)
- [Comment associer une adresse IP statique à mon service cloud ?](#how-do-i-associate-a-static-ip-address-to-my-cloud-service)
- [Quelles sont les fonctionnalités et capacités IPS/ID et DDOS de base sur Azure ?](#what-are-the-features-and-capabilities-that-azure-basic-ipsids-and-ddos-provides)
- [Comment activer HTTP/2 sur la machine virtuelle de services cloud ?](#how-to-enable-http2-on-cloud-services-vm)

**autorisations**

- [Les ingénieurs internes Microsoft peuvent-ils se connecter en utilisant le Bureau à distance à des instances de service cloud sans autorisation ?](#can-microsoft-internal-engineers-remote-desktop-to-cloud-service-instances-without-permission)
- [Je ne peux pas me connecter en utilisant le Bureau à distance à la machine virtuelle de service cloud à l’aide du fichier RDP. J’obtiens l’erreur suivante : Une erreur d’authentification s’est produite (Code : 0x80004005)](#i-cannot-remote-desktop-to-cloud-service-vm--by-using-the-rdp-file-i-get-following-error-an-authentication-error-has-occurred-code-0x80004005)

**Mise à l'échelle**

- [Je ne parviens pas à mettre à l’échelle au-delà de X instances](#i-cannot-scale-beyond-x-instances)
- [Comment puis-je configurer la mise à l’échelle automatique en fonction des métriques de la mémoire ?](#how-can-i-configure-auto-scale-based-on-memory-metrics)

**Générique**

- [Comment ajouter des directives « nosniff » à mon site web ?](#how-do-i-add-nosniff-to-my-website)
- [Comment personnaliser IIS pour un rôle web ?](#how-do-i-customize-iis-for-a-web-role)
- [Quelle est la limite de quota de mon service cloud ?](#what-is-the-quota-limit-for-my-cloud-service)
- [Pourquoi le lecteur de la machine virtuelle de mon service cloud dispose-t-il d’aussi peu d’espace disque ?](#why-does-the-drive-on-my-cloud-service-vm-show-very-little-free-disk-space)
- [Comment puis-je ajouter une extension Antimalware de manière automatisée pour mes services cloud ?](#how-can-i-add-an-antimalware-extension-for-my-cloud-services-in-an-automated-way)
- [Comment activer l’Indication du nom de serveur (SNI) pour les services cloud ?](#how-to-enable-server-name-indication-sni-for-cloud-services)
- [Comment puis-je ajouter des balises à mon service cloud Azure ?](#how-can-i-add-tags-to-my-azure-cloud-service)
- [Le portail Azure n’affiche pas la version du SDK de mon service cloud. Comment puis-je l’obtenir ?](#the-azure-portal-doesnt-display-the-sdk-version-of-my-cloud-service-how-can-i-get-that)
- [Je souhaite arrêter le service cloud pendant plusieurs mois. Comment réduire le coût de facturation du service cloud sans perdre l’adresse IP ?](#i-want-to-shut-down-the-cloud-service-for-several-months-how-to-reduce-the-billing-cost-of-cloud-service-without-losing-the-ip-address)


## <a name="certificates"></a>Certificats

### <a name="why-is-the-certificate-chain-of-my-cloud-service-tlsssl-certificate-incomplete"></a>Pourquoi la chaîne d’approbation du certificat TLS/SSL de mon service cloud est-elle incomplète ?
    
Nous recommandons aux clients d’installer la chaîne de certificats complète (certificat feuille, certificats intermédiaires et certificat racine) au lieu du seul certificat feuille. Quand vous installez uniquement le certificat feuille, vous faites confiance à Windows pour générer la chaîne de certificats en parcourant la liste CTL. Si Azure ou Windows Update rencontre des problèmes réseau ou DNS intermittents pendant que Windows tente de valider le certificat, celui-ci peut être considéré comme non valide. En installant la chaîne de certificats complète, ce problème peut être évité. L’article du blog intitulé [How to install a chained SSL certificate](/archive/blogs/azuredevsupport/how-to-install-a-chained-ssl-certificate) (Comment installer un certificat SSL chaîné) explique comment effectuer cette opération.

### <a name="what-is-the-purpose-of-the-windows-azure-tools-encryption-certificate-for-extensions"></a>À quoi sert le « certificat de chiffrement Windows Azure Tools pour les extensions » ?

Ces certificats sont créés automatiquement chaque fois qu’une extension est ajoutée au service cloud. Il s’agit le plus souvent de l’extension WAD ou RDP, mais il peut s’agit d’autres extensions, notamment Antimalware ou Log Collector. Ces certificats sont utilisés uniquement pour chiffrer et déchiffrer la configuration privée de l’extension. Dans la mesure où la date d’expiration n’est jamais vérifiée, l’expiration du certificat ne pose pas de problème. 

Vous pouvez ignorer ces certificats. Si vous voulez nettoyer les certificats, vous pouvez essayer de tous les supprimer. Azure génère une erreur si vous essayez de supprimer un certificat en cours d’utilisation.

### <a name="how-can-i-generate-a-certificate-signing-request-csr-without-rdp-ing-in-to-the-instance"></a>Comment générer une demande de signature de certificat sans se connecter à l’instance via le protocole RDP ?

Consultez le document d’instructions suivant :

[Obtention d’un certificat pour une utilisation avec Windows Azure Web Sites (WAWS)](https://azure.microsoft.com/blog/obtaining-a-certificate-for-use-with-windows-azure-web-sites-waws/)

La demande de signature de certificat est un simple fichier texte. Il n’est pas nécessaire de la créer à partir de la machine sur laquelle le certificat est finalement utilisé. Même si ce document est écrit pour un service d’application (App Service), la création d’une demande de signature de certificat est générique et s’applique aussi aux services cloud.

### <a name="my-cloud-service-management-certificate-is-expiring-how-to-renew-it"></a>Le certificat de gestion de mon service cloud arrive à expiration. Comment le renouveler ?

Vous pouvez utiliser les commandes PowerShell suivantes pour renouveler vos certificats de gestion :

```powershell
Add-AzureAccount
Select-AzureSubscription -Current -SubscriptionName <your subscription name>
Get-AzurePublishSettingsFile
```

**Get-AzurePublishSettingsFile** crée un certificat de gestion dans **Abonnement** > **Certificats de gestion** dans le portail Azure. Le nom du nouveau certificat ressemble à « [nom_de_votre_abonnement]-[date_actuelle]-credentials ».

### <a name="how-to-automate-the-installation-of-main-tlsssl-certificatepfx-and-intermediate-certificatep7b"></a>Comment automatiser l’installation d’un certificat TLS/SSL principal et d’un certificat intermédiaire (.p7b) ?

Vous pouvez automatiser cette tâche à l’aide d’un script de démarrage (batch/cmd/PowerShell) et enregistrer ce script de démarrage dans le fichier de définition de service. Ajoutez le script de démarrage et le certificat (fichier .p7b) dans le dossier de projet du même répertoire du script de démarrage.

### <a name="what-is-the-purpose-of-the-microsoft-azure-service-management-for-machinekey-certificate"></a>Qu’est l’objectif du certificat « Microsoft Azure Service Management pour MachineKey » ?

Ce certificat est utilisé pour chiffrer les clés de machine sur les rôles web Azure. Pour plus d’informations, consultez [ces conseils](/security-updates/securityadvisories/2018/4092731).

Pour plus d’informations, consultez les articles suivants :
- [Comment configurer et exécuter des tâches de démarrage pour un service cloud](./cloud-services-startup-tasks.md)
- [Tâches courantes de démarrage dans le service cloud](./cloud-services-startup-tasks-common.md)

## <a name="monitoring-and-logging"></a>Surveillance et journalisation

### <a name="what-are-the-upcoming-cloud-service-capabilities-in-the-azure-portal-which-can-help-manage-and-monitor-applications"></a>Quelles prochaines fonctionnalités de service cloud dans le portail Azure peuvent aider à gérer et surveiller des applications ?

La possibilité de générer un nouveau certificat de protocole RDP (Remote Desktop Protocol) est prévue prochainement. Vous pouvez également exécuter ce script :

```powershell
$cert = New-SelfSignedCertificate -DnsName yourdomain.cloudapp.net -CertStoreLocation "cert:\LocalMachine\My" -KeyLength 20 48 -KeySpec "KeyExchange"
$password = ConvertTo-SecureString -String "your-password" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath ".\my-cert-file.pfx" -Password $password
```
La possibilité de choisir l’option « blob » ou « local » pour votre emplacement de chargement csdef et cscfg est prévue prochainement. À l’aide de [New-AzureDeployment](/powershell/module/servicemanagement/azure.service/new-azuredeployment?view=azuresmps-4.0.0), vous pouvez définir chaque valeur d’emplacement.

La possibilité de surveiller les métriques au niveau de l’instance. Des fonctionnalités de surveillance supplémentaires sont disponibles dans [Surveillance de Services cloud](cloud-services-how-to-monitor.md).

### <a name="why-does-iis-stop-writing-to-the-log-directory"></a>Pourquoi IIS cesse-t-il d’écrire dans le répertoire des journaux ?
Vous avez épuisé le quota de stockage local réservé à l’écriture dans le répertoire des journaux. Pour corriger ce problème, vous avez trois solutions :
* Activez les diagnostics pour IIS et déplacez régulièrement les diagnostics sur le Stockage Blob.
* Supprimez manuellement les fichiers journaux du répertoire de journalisation.
* Augmentez la limite de quota pour les ressources locales.

Pour plus d’informations, consultez les documents suivants :
* [Stocker et afficher des données de diagnostic dans le stockage Azure](../storage/common/storage-introduction.md)
* [Le service Journaux d’activité IIS cesse d’écrire dans le service cloud](/archive/blogs/cie/iis-logs-stops-writing-in-cloud-service)

### <a name="how-do-i-enable-wad-logging-for-cloud-services"></a>Comment activer la journalisation des diagnostics Microsoft Azure pour les Services cloud ?
Vous pouvez activer la journalisation Diagnostics Azure pour Windows (WAD) par le biais des options suivantes :
1. [Activer à partir de Visual Studio](/visualstudio/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines#turn-on-diagnostics-in-cloud-service-projects-before-you-deploy-them)
2. [Activer par le biais du code .NET](./cloud-services-dotnet-diagnostics.md)
3. [Activer par le biais de PowerShell](./cloud-services-diagnostics-powershell.md)

Pour obtenir les paramètres actuels des diagnostics Microsoft Azure de votre service cloud, vous pouvez utiliser la cmd ps [Get-AzureServiceDiagnosticsExtensions](./cloud-services-diagnostics-powershell.md#get-current-diagnostics-extension-configuration) ou les voir dans le portail en accédant au panneau « Services cloud --> Extensions ».


## <a name="network-configuration"></a>Configuration réseau

### <a name="how-do-i-set-the-idle-timeout-for-azure-load-balancer"></a>Comment définir le délai d’inactivité d’Azure Load Balancer ?
Vous pouvez spécifier le délai d’expiration dans votre fichier de définition de service (csdef) comme ceci :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="mgVS2015Worker" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRole1" vmsize="Small">
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
    </ConfigurationSettings>
    <Imports>
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
    <Endpoints>
      <InputEndpoint name="Endpoint1" protocol="tcp" port="10100"   idleTimeoutInMinutes="30" />
    </Endpoints>
  </WorkerRole>
```
Pour plus d’informations, consultez [Nouveau : délai d’inactivité configurable pour Azure Load Balancer](https://azure.microsoft.com/blog/new-configurable-idle-timeout-for-azure-load-balancer/).

### <a name="how-do-i-associate-a-static-ip-address-to-my-cloud-service"></a>Comment associer une adresse IP statique à mon service cloud ?
Pour configurer une adresse IP statique, vous devez créer une adresse IP réservée. Cette adresse IP réservée peut être associée à un nouveau service cloud ou à un déploiement existant. Pour plus d’informations, consultez les documents suivants :
* [Comment créer une adresse IP réservée](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#manage-reserved-vips)
* [Réserver l’adresse IP d’un service cloud existant](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#reserve-the-ip-address-of-an-existing-cloud-service)
* [Associer une adresse IP réservée à un nouveau service cloud](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#associate-a-reserved-ip-to-a-new-cloud-service)
* [Associer une adresse IP réservée à un déploiement en cours d’exécution](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#associate-a-reserved-ip-to-a-running-deployment)
* [Associer une adresse IP réservée à un service cloud à l’aide d’un fichier de configuration de service](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip#associate-a-reserved-ip-to-a-cloud-service-by-using-a-service-configuration-file)

### <a name="what-are-the-features-and-capabilities-that-azure-basic-ipsids-and-ddos-provides"></a>Quelles fonctionnalités et capacités IPS/ID et DDOS de base trouve-t-on sur Azure ?
Azure propose des fonctionnalités IPS/IDS sur les serveurs physiques des centres de données pour assurer une protection contre les menaces. Par ailleurs, les clients peuvent déployer des solutions de sécurité tierces, telles que des pare-feu d’applications web, des pare-feu réseau, des logiciels anti-programmes malveillants, des systèmes de détection et de prévention des intrusions (IDS/IPS), etc. Pour plus d’informations, consultez [Protégez vos données et vos biens en respectant les normes internationales de sécurité](https://www.microsoft.com/en-us/trustcenter/Security/AzureSecurity).

Microsoft surveille en continu les serveurs, les réseaux et les applications pour détecter les menaces. L’approche concertée de la gestion des menaces d’Azure utilise également la détection d’intrusion, la prévention des attaques par déni de service distribué (DDoS), le test de pénétration, l’analytique des comportements, la détection d’anomalies et le Machine Learning pour renforcer constamment ses défenses et réduire les risques. Microsoft Antimalware pour Azure protège les services cloud et les machines virtuelles Azure. Vous avez la possibilité de déployer des solutions de sécurité tierces supplémentaires, telles que des pare-feu d’applications web, des pare-feu réseau, des logiciels anti-programmes malveillants, des systèmes de détection et de prévention des intrusions (IDS/IPS), etc.

### <a name="how-to-enable-http2-on-cloud-services-vm"></a>Comment activer HTTP/2 sur la machine virtuelle de services cloud ?

Windows 10 et Windows Server 2016 prennent en charge HTTP/2 à la fois côté client et côté serveur. Si votre client (navigateur) se connecte au serveur IIS sur TLS qui négocie HTTP/2 via les extensions TLS, vous n’avez pas besoin d’effectuer de modifications côté serveur. En effet, sur TLS, l’en-tête h2-14 spécifiant l’utilisation de HTTP/2 est envoyé par défaut. En revanche, si votre client envoie un en-tête de mise à niveau pour mettre à niveau vers HTTP/2, vous devez effectuer la modification ci-dessous côté serveur pour garantir la correcte mise à niveau et l’établissement d’une connexion HTTP/2. 

1. Exécutez regedit.exe.
2. Accédez à la clé de registre : HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HTTP\Parameters.
3. Créez une valeur DWORD nommée **DuoEnabled**.
4. Définissez-la sur 1.
5. Redémarrez votre serveur.
6. Accédez à votre **site web par défaut** et sous **Liaisons**, créez une liaison TLS avec le certificat auto-signé qui vient d’être créé. 

Pour plus d'informations, consultez les pages suivantes :

- [HTTP/2 sur IIS](https://blogs.iis.net/davidso/http2)
- [Vidéo : HTTP/2 dans Windows 10 : Navigateur, applications et serveur Web](https://channel9.msdn.com/Events/Build/2015/3-88)
         

Ces étapes peuvent être automatisées via une tâche de démarrage de manière à pouvoir effectuer les modifications ci-dessus dans le Registre système chaque fois qu’une instance PaaS est créée. Pour plus d’informations, consultez [Comment configurer et exécuter des tâches de démarrage pour un service cloud](cloud-services-startup-tasks.md).

 
Une fois terminé, vous pouvez vérifier si la connexion HTTP/2 a été activée ou non en utilisant l’une des méthodes suivantes :

- Activez la version de protocole dans les journaux d’activité IIS et consultez ces derniers. Vous y trouverez les informations sur HTTP/2. 
- Activez l’outil de développement F12 dans Internet Explorer ou Microsoft Edge et basculez vers l’onglet Réseau pour vérifier le protocole. 

Pour plus d’informations, consultez [HTTP/2 sur IIS](https://blogs.iis.net/davidso/http2).

## <a name="permissions"></a>Autorisations

### <a name="how-can-i-implement-role-based-access-for-cloud-services"></a>Comment implémenter l’accès en fonction du rôle pour les Services cloud ?
Azure Cloud Services ne prend pas en charge le modèle de contrôle d’accès en fonction du rôle Azure (Azure RBAC), car il ne s’agit pas d’un service Azure Resource Manager.

Voir [Comprendre les différents rôles dans Azure](../role-based-access-control/rbac-and-directory-admin-roles.md).

## <a name="remote-desktop"></a>Bureau à distance

### <a name="can-microsoft-internal-engineers-remote-desktop-to-cloud-service-instances-without-permission"></a>Les ingénieurs internes Microsoft peuvent-ils se connecter en utilisant le Bureau à distance à des instances de service cloud sans autorisation ?
Microsoft suit un processus strict qui ne permet pas à ses ingénieurs internes de se connecter en utilisant le Bureau à distance à votre service cloud sans une autorisation écrite (par e-mail ou toute autre communication écrite) de la part du propriétaire ou de son représentant.

### <a name="i-cannot-remote-desktop-to-cloud-service-vm--by-using-the-rdp-file-i-get-following-error-an-authentication-error-has-occurred-code-0x80004005"></a>Je ne peux pas me connecter en utilisant le Bureau à distance à la machine virtuelle de service cloud à l’aide du fichier RDP. J’obtiens l’erreur suivante : Une erreur d’authentification s’est produite (Code : 0x80004005)

Cette erreur peut se produire si vous utilisez le fichier RDP à partir d’une machine jointe à Azure Active Directory. Pour résoudre ce problème, effectuez les étapes suivantes :

1. Cliquez avec le bouton droit sur le fichier RDP que vous avez téléchargé, puis sélectionnez **Modifier**.
2. Ajoutez « &#92; » comme préfixe avant le nom d’utilisateur. Par exemple, utilisez **.\nom_utilisateur** au lieu de **nom_utilisateur**.

## <a name="scaling"></a>Mise à l'échelle

### <a name="i-cannot-scale-beyond-x-instances"></a>Je ne parviens pas à mettre à l’échelle au-delà de X instances
Le nombre de cœurs que vous pouvez utiliser est limité dans votre abonnement Azure. La mise à l’échelle ne fonctionnera pas si vous avez utilisé tous les cœurs disponibles. Par exemple, si vous avez une limite de 100 cœurs, cela signifie que vous pouvez avoir 100 instances de machine virtuelle A1 pour votre service cloud ou 50 instances de machine virtuelle A2.

### <a name="how-can-i-configure-auto-scale-based-on-memory-metrics"></a>Comment puis-je configurer la mise à l’échelle automatique en fonction des métriques de la mémoire ?

La mise à l’échelle automatique en fonction des métriques de la mémoire pour un service cloud n’est pas prise en charge. 

Pour contourner ce problème, vous pouvez utiliser Application Insights. La mise à l’échelle automatique prend en charge Application Insights en tant que source de métriques et peut mettre à l’échelle le nombre d’instances de rôle en fonction d’une métrique invitée telle que « Mémoire ».  Vous devez configurer Application Insights dans votre fichier de package de projet de service cloud (*.cspkg) et activer l’extension Diagnostics Azure sur le service pour implémenter cette fonction.

Pour plus d’informations sur la façon d’utiliser une métrique personnalisée par le biais d’Application Insights afin de configurer la mise à l’échelle automatique sur les services cloud, consultez [Prise en main de la mise à l’échelle automatique par métrique personnalisée dans Azure](../azure-monitor/platform/autoscale-custom-metric.md).

Pour plus d’informations sur la façon d’intégrer Diagnostics Azure à Application Insights pour les services cloud, consultez [Envoyer des données de diagnostic de service cloud, de machine virtuelle ou de Service Fabric à Application Insights](../azure-monitor/platform/diagnostics-extension-to-application-insights.md).

Pour plus d’informations sur la façon d’activer Application Insights pour les services cloud, consultez [Application Insights pour Services cloud Azure](../azure-monitor/app/cloudservices.md).

Pour plus d’informations sur la façon d’activer la journalisation Diagnostics Azure pour les services cloud, consultez [Configurer les diagnostics pour les services cloud et les machines virtuelles Azure](/visualstudio/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines#turn-on-diagnostics-in-cloud-service-projects-before-you-deploy-them).

## <a name="generic"></a>Générique

### <a name="how-do-i-add-nosniff-to-my-website"></a>Comment ajouter des directives « nosniff » à mon site web ?
Pour empêcher les clients de détecter les types MIME, ajoutez un paramètre au fichier *web.config*.

```xml
<configuration>
   <system.webServer>
      <httpProtocol>
         <customHeaders>
            <add name="X-Content-Type-Options" value="nosniff" />
         </customHeaders>
      </httpProtocol>
   </system.webServer>
</configuration>
```

Vous pouvez également ajouter ce paramètre dans IIS. Utilisez la commande suivante avec l’article [tâches courantes de démarrage](cloud-services-startup-tasks-common.md#configure-iis-startup-with-appcmdexe).

```cmd
%windir%\system32\inetsrv\appcmd set config /section:httpProtocol /+customHeaders.[name='X-Content-Type-Options',value='nosniff']
```

### <a name="how-do-i-customize-iis-for-a-web-role"></a>Comment personnaliser IIS pour un rôle web ?
Utilisez le script de démarrage IIS à partir de l’article [tâches courantes de démarrage](cloud-services-startup-tasks-common.md#configure-iis-startup-with-appcmdexe).

### <a name="what-is-the-quota-limit-for-my-cloud-service"></a>Quelle est la limite de quota de mon service cloud ?
Consultez [Limites spécifiques des services](../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits).

### <a name="why-does-the-drive-on-my-cloud-service-vm-show-very-little-free-disk-space"></a>Pourquoi le lecteur de la machine virtuelle de mon service cloud dispose-t-il d’aussi peu d’espace disque ?
Il s’agit d’un comportement normal qui ne devrait pas causer de problèmes à votre application. La journalisation est activée pour le lecteur %approot% des machines virtuelles Azure PaaS, ce qui dans l’absolu a pour effet de consommer normalement le double d’espace que les fichiers. Cependant, cela ne pose pas de problème, et ce pour plusieurs raisons.

La taille du lecteur %AppRoot% est calculée comme suit \<size of .cspkg + max journal size + a margin of free space>, ou est égale à 1,5 Go, la valeur la plus grande étant retenue. La taille de votre machine virtuelle n’a pas d’incidence sur ce calcul. (La taille de machine virtuelle affecte uniquement la taille du lecteur C: temporaire.) 

L’écriture sur le lecteur %approot% n’est pas prise en charge. Si vous écrivez sur la machine virtuelle Azure, vous devez le faire dans une ressource LocalStorage temporaire (ou une autre option, telle que le Stockage Blob, Azure Files, etc..). Autrement dit, la quantité d’espace libre dans le dossier %approot % n’est pas significative. Pour savoir avec certitude si votre application écrit ou non sur le lecteur %approot%, vous pouvez toujours laisser votre service s’exécuter pendant quelques jours et comparer sa taille avant et après. 

Azure n’écrit rien sur le lecteur %approot%. Une fois le disque dur virtuel créé à partir de votre fichier .cspkg et monté dans la machine virtuelle Azure, votre application est la seule à pouvoir écrire sur ce lecteur. 

Les paramètres de journal n’étant pas configurables, vous ne pouvez pas les désactiver.

### <a name="how-can-i-add-an-antimalware-extension-for-my-cloud-services-in-an-automated-way"></a>Comment puis-je ajouter une extension Antimalware de manière automatisée pour mes services cloud ?

Vous pouvez activer une extension Antimalware à l’aide du script PowerShell dans la tâche de démarrage. Suivez les étapes décrites dans les articles suivants pour l’implémenter : 
 
- [Créer une tâche de démarrage PowerShell](cloud-services-startup-tasks-common.md#create-a-powershell-startup-task)
- [Set-AzureServiceAntimalwareExtension](/powershell/module/servicemanagement/azure.service/Set-AzureServiceAntimalwareExtension?view=azuresmps-4.0.0 )

Pour plus d’informations sur les scénarios de déploiement Antimalware et la façon de l’activer à partir du portail, consultez [Scénarios de déploiement Antimalware](../security/fundamentals/antimalware.md#antimalware-deployment-scenarios).

### <a name="how-to-enable-server-name-indication-sni-for-cloud-services"></a>Comment activer l’Indication du nom de serveur (SNI) pour les services cloud ?

Vous pouvez activer SNI dans les services cloud en utilisant l’une des méthodes suivantes :

**Méthode 1 : Utiliser PowerShell**

La liaison SNI peut être configurée à l’aide de l’applet de commande PowerShell **New-WebBinding** dans une tâche de démarrage pour une instance de rôle de service cloud comme indiqué ci-dessous :

```powershell
New-WebBinding -Name $WebsiteName -Protocol "https" -Port 443 -IPAddress $IPAddress -HostHeader $HostHeader -SslFlags $sslFlags
```

Comme décrit [ici](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee790567(v=technet.10)), $sslFlags peut adopter l’une des valeurs suivantes :

|Valeur|Signification|
------|------
|0|Pas de SNI|
|1|SNI activée|
|2|Liaison non SNI qui utilise le magasin de certificats central|
|3|Liaison SNI qui utilise le magasin de certificats central|
 
**Méthode 2 : Utiliser le code**

La liaison SNI peut également être configurée via code au cours du démarrage de rôle comme décrit dans ce [billet de blog](/archive/blogs/jianwu/expose-ssl-service-to-multi-domains-from-the-same-cloud-service) :

```csharp
//<code snip> 
                var serverManager = new ServerManager(); 
                var site = serverManager.Sites[0]; 
                var binding = site.Bindings.Add(":443:www.test1.com", newCert.GetCertHash(), "My"); 
                binding.SetAttributeValue("sslFlags", 1); //enables the SNI 
                serverManager.CommitChanges(); 
    //</code snip>
```
    
Si vous utilisez l’une des approches ci-dessus, les certificats respectifs (*.pfx) pour les noms d’hôte spécifiques doivent être installés au préalable sur les instances de rôle à l’aide d’une tâche de démarrage ou via code pour activer la liaison SNI.

### <a name="how-can-i-add-tags-to-my-azure-cloud-service"></a>Comment puis-je ajouter des balises à mon service cloud Azure ? 

Le service cloud est une ressource classique. Seules les ressources créées via Azure Resource Manager prennent en charge les balises. Vous ne pouvez pas appliquer de balises à des ressources classiques comme un service cloud. 

### <a name="the-azure-portal-doesnt-display-the-sdk-version-of-my-cloud-service-how-can-i-get-that"></a>Le portail Azure n’affiche pas la version du SDK de mon service cloud. Comment puis-je l’obtenir ?

Nous travaillons à l’ajout de cette fonctionnalité au portail Azure. D’ici là, vous pouvez utiliser les commandes PowerShell suivantes pour obtenir la version du SDK :

```powershell
Get-AzureService -ServiceName "<Cloud Service name>" | Get-AzureDeployment | Where-Object -Property SdkVersion -NE -Value "" | select ServiceName,SdkVersion,OSVersion,Slot
```

### <a name="i-want-to-shut-down-the-cloud-service-for-several-months-how-to-reduce-the-billing-cost-of-cloud-service-without-losing-the-ip-address"></a>Je souhaite arrêter le service cloud pendant plusieurs mois. Comment réduire le coût de facturation du service cloud sans perdre l’adresse IP ?

Un service cloud déjà déployé est facturé pour les ressources de calcul et de stockage qu’il utilise. Ainsi, même si vous arrêtez la machine virtuelle Azure, vous êtes facturé pour le stockage. 

Voici ce que vous pouvez faire pour réduire votre facturation sans perdre l’adresse IP de votre service :

1. [Réservez l’adresse IP](/previous-versions/azure/virtual-network/virtual-networks-reserved-public-ip) avant de supprimer les déploiements.  Vous êtes uniquement facturé pour cette adresse IP. Pour plus d’informations sur la facturation d’une adresse IP, consultez [Tarification des adresses IP](https://azure.microsoft.com/pricing/details/ip-addresses/).
2. Supprimez les déploiements. Ne supprimez pas xxx.cloudapp.net, afin que vous puissiez l’utiliser à l’avenir.
3. Si vous souhaitez redéployer le service cloud à l’aide de l’adresse IP de réserve que vous avez réservée dans votre abonnement, consultez [Adresses IP réservées pour les services cloud et les machines virtuelles](https://azure.microsoft.com/blog/reserved-ip-addresses/).