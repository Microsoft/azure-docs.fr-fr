---
title: Package et modèle Cloud Service dans Azure | Microsoft Docs
description: Décrit le modèle de service cloud (.csdef, .cscfg) et le package (.cspkg) dans Azure.
services: cloud-services
author: tanmaygore
ms.service: cloud-services
ms.topic: article
ms.date: 07/05/2017
ms.author: tagore
ms.openlocfilehash: 180295599082a762fc525c4740079ceefc0954a1
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077182"
---
# <a name="what-is-the-cloud-service-model-and-how-do-i-package-it"></a>Qu’est-ce que le modèle Cloud Service, et comment en créer un package ?
Un service cloud est créé à partir de trois composants : la définition de service *(.csdef)* , la configuration de service *(.cscfg)* et un package de service *(.cspkg)* . Les deux fichiers XML **ServiceDefinition.csdef** et **ServiceConfig.cscfg** décrivent la structure du service cloud et sa configuration, qui désignent collectivement le modèle. Le fichier ZIP **ServicePackage.cspkg** est généré à partir du fichier **ServiceDefinition.csdef** et il contient, entre autres, toutes les dépendances binaires requises. Azure crée un service cloud à partir des fichiers **ServicePackage.cspkg** et **ServiceConfig.cscfg**.

Une fois que le service cloud s’exécute dans Azure, vous pouvez le reconfigurer via le fichier **ServiceConfig.cscfg** , mais vous ne pouvez pas en modifier la définition.

## <a name="what-would-you-like-to-know-more-about"></a>Quels sont les sujets sur lesquels vous souhaitez avoir des informations supplémentaires ?
* Je souhaite en savoir plus sur les fichiers [ServiceDefinition.csdef](#csdef) et [ServiceConfig.cscfg](#cscfg).
* Je connais déjà cela, mais donnez-moi [quelques exemples](#next-steps) de configuration.
* Je souhaite créer le fichier [ServicePackage.cspkg](#cspkg).
* J’utilise Visual Studio et souhaite...
  * [Créer un service cloud][vs_create]
  * [Reconfigurer un service cloud existant][vs_reconfigure]
  * [Déployer un projet de service cloud][vs_deploy]
  * [Un Bureau à distance sur une instance de service cloud][remotedesktop]

<a name="csdef"></a>

## <a name="servicedefinitioncsdef"></a>ServiceDefinition.csdef
Le fichier **ServiceDefinition.csdef** spécifie les paramètres qui sont utilisés par Azure pour configurer un service cloud. Le [schéma de définition du service Azure (fichier .csdef)](/previous-versions/azure/reference/ee758711(v=azure.100)) indique le format autorisé pour un fichier de définition de service. L’exemple suivant présente les paramètres qui peuvent être définis pour le rôle web et le rôle de travail :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceDefinition name="MyServiceName" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WebRole name="WebRole1" vmsize="Medium">
    <Sites>
      <Site name="Web">
        <Bindings>
          <Binding name="HttpIn" endpointName="HttpIn" />
        </Bindings>
      </Site>
    </Sites>
    <Endpoints>
      <InputEndpoint name="HttpIn" protocol="http" port="80" />
      <InternalEndpoint name="InternalHttpIn" protocol="http" />
    </Endpoints>
    <Certificates>
      <Certificate name="Certificate1" storeLocation="LocalMachine" storeName="My" />
    </Certificates>
    <Imports>
      <Import moduleName="Connect" />
      <Import moduleName="Diagnostics" />
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
    <LocalResources>
      <LocalStorage name="localStoreOne" sizeInMB="10" />
      <LocalStorage name="localStoreTwo" sizeInMB="10" cleanOnRoleRecycle="false" />
    </LocalResources>
    <Startup>
      <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" />
    </Startup>
  </WebRole>

  <WorkerRole name="WorkerRole1">
    <ConfigurationSettings>
      <Setting name="DiagnosticsConnectionString" />
    </ConfigurationSettings>
    <Imports>
      <Import moduleName="RemoteAccess" />
      <Import moduleName="RemoteForwarder" />
    </Imports>
    <Endpoints>
      <InputEndpoint name="Endpoint1" protocol="tcp" port="10000" />
      <InternalEndpoint name="Endpoint2" protocol="tcp" />
    </Endpoints>
  </WorkerRole>
</ServiceDefinition>
```

Vous pouvez vous reporter au [schéma de définition de service](/previous-versions/azure/reference/ee758711(v=azure.100)) pour mieux comprendre le schéma XML utilisé ici. Toutefois, voici une brève explication de certains éléments :

**Sites**  
Contient les définitions des sites ou applications web hébergés dans IIS 7.0.

**InputEndpoints**  
Contient les définitions des points de terminaison qui permettent de contacter le service cloud.

**InternalEndpoints**  
Contient les définitions des points de terminaison qui sont utilisés par les instances de rôle pour communiquer entre eux.

**ConfigurationSettings**  
Contient les définitions de paramètre des fonctionnalités d’un rôle spécifique.

**Certificates**  
Contient les définitions des certificats nécessaires à un rôle. L’exemple de code précédent illustre un certificat qui est utilisé pour la configuration d’Azure Connect.

**LocalResources**  
Contient les définitions des ressources de stockage local. Une ressource de stockage local est un répertoire réservé dans le système de fichiers de la machine virtuelle dans lequel s’exécute l’instance d’un rôle.

**Imports**  
Contient les définitions des modules importés. L’exemple de code précédent illustre les modules Connexion Bureau à distance et Azure Connect.

**Startup**  
Contient les tâches qui sont exécutées au démarrage du rôle. Les tâches sont définies dans un fichier .cmd ou exécutable.

<a name="cscfg"></a>

## <a name="serviceconfigurationcscfg"></a>ServiceConfiguration.cscfg
La configuration des paramètres du service cloud est déterminée par les valeurs indiquées dans le fichier **ServiceConfiguration.cscfg** . Vous spécifiez le nombre d’instances que vous souhaitez déployer pour chaque rôle dans ce fichier. Les valeurs des paramètres de configuration que vous avez définis dans le fichier de définition de service sont ajoutées au fichier de configuration de service. Les empreintes numériques des certificats de gestion qui sont associés au service cloud sont également ajoutées au fichier. Le [schéma de configuration du service Azure (fichier .cscfg)](/previous-versions/azure/reference/ee758710(v=azure.100)) indique le format autorisé pour un fichier de configuration de service.

Le fichier de configuration de service n’est pas fourni dans le package de l’application, mais est chargé vers Azure en tant que fichier distinct et permet de configurer le service cloud. Vous pouvez charger un nouveau fichier de configuration de service sans redéployer votre service cloud. Les valeurs de configuration du service cloud peuvent être modifiées pendant l’exécution du service cloud. L’exemple suivant présente les paramètres de configuration qui peuvent être définis pour le rôle web et le rôle de travail :

```xml
<?xml version="1.0"?>
<ServiceConfiguration serviceName="MyServiceName" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration">
  <Role name="WebRole1">
    <Instances count="2" />
    <ConfigurationSettings>
      <Setting name="SettingName" value="SettingValue" />
    </ConfigurationSettings>

    <Certificates>
      <Certificate name="CertificateName" thumbprint="CertThumbprint" thumbprintAlgorithm="sha1" />
      <Certificate name="Microsoft.WindowsAzure.Plugins.RemoteAccess.PasswordEncryption"
         thumbprint="CertThumbprint" thumbprintAlgorithm="sha1" />
    </Certificates>
  </Role>
</ServiceConfiguration>
```

Vous pouvez vous reporter au [schéma de configuration de service](/previous-versions/azure/reference/ee758710(v=azure.100)) pour mieux comprendre le schéma XML utilisé ici. Toutefois, voici une brève explication des éléments :

**Instances**  
Configure le nombre d’instances du rôle en cours d’exécution. Pour empêcher le service cloud d’être potentiellement indisponible pendant les mises à niveau, il est conseillé de déployer plusieurs instances de vos rôles web. En déployant plus d’une instance, vous respectez les recommandations du [contrat de niveau de service de Calcul Azure](https://azure.microsoft.com/support/legal/sla/), ce qui garantit une connectivité externe à 99,95 % pour les rôles Internet lorsque deux instances de rôle au moins sont déployées pour un service.

**ConfigurationSettings**  
Configure les paramètres des instances en cours d’exécution d’un rôle. Le nom des éléments `<Setting>` doit correspondre aux définitions de paramètre dans le fichier de définition de service.

**Certificates**  
Configure les certificats utilisés par le service. L’exemple de code précédent montre comment définir le certificat pour le module RemoteAccess. La valeur de l’attribut *thumbprint* doit être définie sur l’empreinte numérique du certificat à utiliser.

<p/>

> [!NOTE]
> L’empreinte du certificat peut être ajoutée au fichier de configuration à l’aide d’un éditeur de texte. La valeur peut également être ajoutée dans l’onglet **Certificats** de la page **Propriétés** du rôle dans Visual Studio.
> 
> 

## <a name="defining-ports-for-role-instances"></a>Définition des ports pour les instances de rôle
Azure n’autorise qu’un point d’entrée à un rôle web. En d’autres termes, tout le trafic s’effectue via une adresse IP. Vous pouvez configurer vos sites web pour partager un port en configurant l’en-tête de l’hôte pour diriger la demande vers l’emplacement approprié. Vous pouvez également configurer les applications pour qu’elles écoutent aux ports connus sur l’adresse IP.

L’exemple suivant illustre la configuration d’un rôle web avec un site web et une application web. Le site web est configuré comme emplacement d’entrée par défaut sur le port 80, et les applications web sont configurées pour recevoir des demandes d’un en-tête d’hôte différent, appelé « mail.mysite.cloudapp.net ».

```xml
<WebRole>
  <ConfigurationSettings>
    <Setting name="DiagnosticsConnectionString" />
  </ConfigurationSettings>
  <Endpoints>
    <InputEndpoint name="HttpIn" protocol="http" port="80" />
    <InputEndpoint name="Https" protocol="https" port="443" certificate="SSL"/>
    <InputEndpoint name="NetTcp" protocol="tcp" port="808" certificate="SSL"/>
  </Endpoints>
  <LocalResources>
    <LocalStorage name="Sites" cleanOnRoleRecycle="true" sizeInMB="100" />
  </LocalResources>
  <Site name="Mysite" packageDir="Sites\Mysite">
    <Bindings>
      <Binding name="http" endpointName="HttpIn" />
      <Binding name="https" endpointName="Https" />
      <Binding name="tcp" endpointName="NetTcp" />
    </Bindings>
  </Site>
  <Site name="MailSite" packageDir="MailSite">
    <Bindings>
      <Binding name="mail" endpointName="HttpIn" hostHeader="mail.mysite.cloudapp.net" />
    </Bindings>
    <VirtualDirectory name="artifacts" />
    <VirtualApplication name="storageproxy">
      <VirtualDirectory name="packages" packageDir="Sites\storageProxy\packages"/>
    </VirtualApplication>
  </Site>
</WebRole>
```


## <a name="changing-the-configuration-of-a-role"></a>Modification de la configuration d’un rôle
Vous pouvez mettre à jour la configuration du service cloud pendant son exécution dans Azure, sans le mettre hors connexion. Pour modifier les informations de configuration, vous pouvez charger un nouveau fichier de configuration ou modifier le fichier de configuration existant et l’appliquer à votre service en cours d’exécution. Les modifications suivantes peuvent être apportées à la configuration d’un service :

* **Modification des valeurs des paramètres de configuration**  
  Lorsqu’un paramètre de configuration est changé, une instance de rôle peut choisir d’appliquer la modification pendant que l’instance est en ligne ou de recycler l’instance normalement et d’appliquer la modification pendant qu’elle est hors connexion.
* **Modification de la topologie de service des instances de rôle**  
  Les modifications de la topologie n’affectent pas les instances en cours d’exécution, sauf lorsqu’une instance est supprimée. Généralement, vous n’avez pas besoin de recycler les instances restantes, mais vous pouvez choisir de recycler des instances de rôle en réponse à une modification de la topologie.
* **Modification de l’empreinte de certificat**  
  Vous ne pouvez mettre à jour un certificat que lorsqu’une instance de rôle est hors connexion. Si un certificat est ajouté, supprimé ou modifié pendant qu’une instance de rôle est en ligne, Azure la met normalement hors connexion pour mettre à jour le certificat avant de la remettre en ligne une fois la modification effectuée.

### <a name="handling-configuration-changes-with-service-runtime-events"></a>Gestion des modifications de configuration à l’aide des événements de service Runtime
La [bibliothèque Runtime Azure](/previous-versions/azure/reference/mt419365(v=azure.100)) inclut l’espace de noms [Microsoft.WindowsAzure.ServiceRuntime](/previous-versions/azure/reference/ee741722(v=azure.100)), qui fournit des classes pour interagir avec l’environnement Azure à partir d’un rôle. La classe [RoleEnvironment](/previous-versions/azure/reference/ee773173(v=azure.100)) définit les événements suivants qui sont déclenchés avant et après une modification de la configuration :

* **[Modification](/previous-versions/azure/reference/ee758134(v=azure.100)) d’un événement**  
  Se produit avant que la modification de la configuration ne soit appliquée à une instance spécifiée d’un rôle, ce qui vous permet de supprimer les instances de rôle si nécessaire.
* **[Événement](/previous-versions/azure/reference/ee758129(v=azure.100)) modifié**  
  Se produit après l’application de la modification de la configuration à l’instance spécifiée d’un rôle.

> [!NOTE]
> Comme les modifications de certificat placent toujours les instances d’un rôle hors connexion, elles ne déclenchent pas les événements RoleEnvironment.Changing ou RoleEnvironment.Changed.
> 
> 

<a name="cspkg"></a>

## <a name="servicepackagecspkg"></a>ServicePackage.cspkg
> [!NOTE]
> La taille maximale de package qui peut être déployée est de 600 Mo

Pour déployer une application en tant que service cloud dans Azure, vous devez d’abord créer un package de l’application dans le format approprié. Vous pouvez utiliser l’outil de ligne de commande **CSPack** (installé avec le [Kit de développement logiciel (SDK) Azure](https://azure.microsoft.com/downloads/)) pour créer le fichier de package en tant qu’alternative à Visual Studio.

**CSPack** utilise le contenu du fichier de définition de service et du fichier de configuration de service pour définir le contenu du package. **CSPack** génère un fichier de package d’application (.cspkg) que vous pouvez charger dans Azure à l’aide du [portail Azure](cloud-services-how-to-create-deploy-portal.md#create-and-deploy). Par défaut, le package est nommé `[ServiceDefinitionFileName].cspkg`, mais vous pouvez indiquer un autre nom à l’aide de l’option `/out` de **CSPack**.

**CSPack** se situe dans  
`C:\Program Files\Microsoft SDKs\Azure\.NET SDK\[sdk-version]\bin\`

> [!NOTE]
> Le fichier CSPack.exe (sur Windows) est disponible en exécutant le raccourci de l’ **invite de commandes Microsoft Azure** , qui est installé avec le Kit de développement logiciel (SDK).  
> 
> Exécutez le programme CSPack.exe pour consulter la documentation relative à l’ensemble des commutateurs et commandes possibles.
> 
> 

<p />

> [!TIP]
> Exécutez votre service cloud localement dans **l’émulateur de calcul Microsoft Azure** et utilisez l’option **/copyonly**. Cette option copie les fichiers binaires de l’application dans une disposition de répertoire d’où ils peuvent être exécutés dans l’émulateur de calcul.
> 
> 

### <a name="example-command-to-package-a-cloud-service"></a>Exemple de commande pour créer un package de service cloud
L’exemple suivant crée un package d’application qui contient les informations relatives à un rôle web. La commande spécifie le fichier de définition de service à utiliser, le répertoire dans lequel les fichiers binaires se trouvent, et le nom du fichier de package.

```cmd
cspack [DirectoryName]\[ServiceDefinition]
       /role:[RoleName];[RoleBinariesDirectory]
       /sites:[RoleName];[VirtualPath];[PhysicalPath]
       /out:[OutputFileName]
```

Si l’application contient à la fois un rôle web et un rôle de travail, la commande suivante est utilisée :

```cmd
cspack [DirectoryName]\[ServiceDefinition]
       /out:[OutputFileName]
       /role:[RoleName];[RoleBinariesDirectory]
       /sites:[RoleName];[VirtualPath];[PhysicalPath]
       /role:[RoleName];[RoleBinariesDirectory];[RoleAssemblyName]
```

Où les variables sont définies comme suit :

| Variable | Valeur |
| --- | --- |
| \[DirectoryName\] |Sous-répertoire du répertoire racine du projet qui contient le fichier .csdef du projet Azure. |
| \[ServiceDefinition\] |Nom du fichier de définition de service. Par défaut, ce fichier se nomme ServiceDefinition.csdef. |
| \[OutputFileName\] |Nom du fichier de package généré. En règle générale, il correspond au nom de l’application. Si aucun nom de fichier n’est spécifié, le package d’application est créé sous le nom \[ApplicationName\].cspkg. |
| \[RoleName\] |Nom du rôle défini dans le fichier de définition de service. |
| \[RoleBinariesDirectory] |Emplacement des fichiers binaires correspondant au rôle. |
| \[VirtualPath\] |Répertoires physiques pour chaque chemin d’accès virtuel défini dans la section Sites de la définition de service. |
| \[PhysicalPath\] |Répertoires physiques du contenu pour chaque chemin d’accès virtuel défini dans le nœud Site de la définition de service. |
| \[RoleAssemblyName\] |Nom du fichier binaire correspondant au rôle. |

## <a name="next-steps"></a>Étapes suivantes
Je crée un package de service cloud et je souhaite...

* [Configurer un Bureau à distance pour une instance de service cloud][remotedesktop]
* [Déployer un projet de service cloud][deploy]

J’utilise Visual Studio et souhaite...

* [Créer un service cloud][vs_create]
* [Reconfigurer un service cloud existant][vs_reconfigure]
* [Déployer un projet de service cloud][vs_deploy]
* [Configurer un Bureau à distance pour une instance de service cloud][vs_remote]

[deploy]: cloud-services-how-to-create-deploy-portal.md
[remotedesktop]: cloud-services-role-enable-remote-desktop-new-portal.md
[vs_remote]: cloud-services-role-enable-remote-desktop-visual-studio.md
[vs_deploy]: /visualstudio/azure/vs-azure-tools-cloud-service-publish-set-up-required-services-in-visual-studio
[vs_reconfigure]: /visualstudio/azure/vs-azure-tools-configure-roles-for-cloud-service
[vs_create]: /visualstudio/azure/vs-azure-tools-azure-project-create