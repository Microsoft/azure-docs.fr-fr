---
title: Déf. Azure Cloud Services Schéma WebRole | Microsoft Docs
description: Le rôle web Azure est personnalisé pour la programmation d’applications web prenant en charge ASP.NET, PHP, WCF et FastCGI. En savoir plus sur les éléments de définition de service d’un rôle web.
ms.custom: ''
ms.date: 04/14/2015
services: cloud-services
ms.reviewer: ''
ms.service: cloud-services
ms.suite: ''
ms.tgt_pltfrm: ''
ms.topic: reference
ms.assetid: 85368e4e-a0db-4c02-8dbc-8e2928fa6091
caps.latest.revision: 60
author: tgore03
ms.author: tagore
ms.openlocfilehash: 4368bb38a280461fdd77348de60a0e5793ee9582
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011317"
---
# <a name="azure-cloud-services-definition-webrole-schema"></a>Schéma WebRole de définition d’Azure Cloud Services
Le rôle web Azure est un rôle personnalisé pour la programmation d’applications web, tel que pris en charge par IIS 7, ASP.NET, PHP, Windows Communication Foundation et FastCGI.

L’extension par défaut du fichier de définition de service est .csdef.

## <a name="basic-service-definition-schema-for-a-web-role"></a>Schéma de définition de service de base pour un rôle web  
Le format de base d’un fichier de définition de service contenant un rôle web se présente de la manière suivante.

```xml
<ServiceDefinition …>  
  <WebRole name="<web-role-name>" vmsize="<web-role-size>" enableNativeCodeExecution="[true|false]">  
    <Certificates>  
      <Certificate name="<certificate-name>" storeLocation="<certificate-store>" storeName="<store-name>" />  
    </Certificates>      
    <ConfigurationSettings>  
      <Setting name="<setting-name>" />  
    </ConfigurationSettings>  
    <Imports>  
      <Import moduleName="<import-module>"/>  
    </Imports>  
    <Endpoints>  
      <InputEndpoint certificate="<certificate-name>" ignoreRoleInstanceStatus="[true|false]" name="<input-endpoint-name>" protocol="[http|https|tcp|udp]" localPort="<port-number>" port="<port-number>" loadBalancerProbe="<load-balancer-probe-name>" />  
      <InternalEndpoint name="<internal-endpoint-name>" protocol="[http|tcp|udp|any]" port="<port-number>">  
         <FixedPort port="<port-number>"/>  
         <FixedPortRange min="<minimum-port-number>" max="<maximum-port-number>"/>  
      </InternalEndpoint>  
     <InstanceInputEndpoint name="<instance-input-endpoint-name>" localPort="<port-number>" protocol="[udp|tcp]">  
         <AllocatePublicPortFrom>  
            <FixedPortRange min="<minimum-port-number>" max="<maximum-port-number>"/>  
         </AllocatePublicPortFrom>  
      </InstanceInputEndpoint>  
    </Endpoints>  
    <LocalResources>  
      <LocalStorage name="<local-store-name>" cleanOnRoleRecycle="[true|false]" sizeInMB="<size-in-megabytes>" />  
    </LocalResources>  
    <LocalStorage name="<local-store-name>" cleanOnRoleRecycle="[true|false]" sizeInMB="<size-in-megabytes>" />  
    <Runtime executionContext="[limited|elevated]">  
      <Environment>  
         <Variable name="<variable-name>" value="<variable-value>">  
            <RoleInstanceValue xpath="<xpath-to-role-environment-settings>"/>  
          </Variable>            
      </Environment>  
      <EntryPoint>  
         <NetFxEntryPoint assemblyName="<name-of-assembly-containing-entrypoint>" targetFrameworkVersion="<.net-framework-version>"/>  
      </EntryPoint>  
    </Runtime>  
    <Sites>  
      <Site name="<web-site-name>">  
        <VirtualApplication name="<application-name>" physicalDirectory="<directory-path>"/>  
        <VirtualDirectory name="<directory-path>" physicalDirectory="<directory-path>"/>  
        <Bindings>  
          <Binding name="<binding-name>" endpointName="<endpoint-name-bound-to>" hostHeader="<url-of-the-site>"/>  
        </Bindings>  
      </Site>  
    </Sites>  
    <Startup priority="<for-internal-use-only>">  
      <Task commandLine="<command-to=execute>" executionContext="[limited|elevated]" taskType="[simple|foreground|background]">  
        <Environment>  
         <Variable name="<variable-name>" value="<variable-value>">  
            <RoleInstanceValue xpath="<xpath-to-role-environment-settings>"/>  
          </Variable>            
        </Environment>  
      </Task>  
    </Startup>  
    <Contents>  
      <Content destination="<destination-folder-name>" >  
        <SourceDirectory path="<local-source-directory>" />  
      </Content>  
    </Contents>  
  </WebRole>  
</ServiceDefinition>  
```  

## <a name="schema-elements"></a>Éléments du schéma  
Le fichier de définition de service inclut ces éléments, ils sont détaillés dans les sections suivantes de cette rubrique :  

[WebRole](#WebRole)

[ConfigurationSettings](#ConfigurationSettings)

[Paramètre](#Setting)

[LocalResources](#LocalResources)

[LocalStorage](#LocalStorage)

[Points de terminaison](#Endpoints)

[InternalEndpoint](#InternalEndpoint)

[InstanceInputEndpoint](#InstanceInputEndpoint)

[AllocatePublicPortFrom](#AllocatePublicPortFrom)

[FixedPort](#FixedPort)

[FixedPortRange](#FixedPortRange)

[Certificates](#Certificates)

[Certificate](#Certificate)

[Imports](#Imports)

[Importer](#Import)

[Runtime](#Runtime)

[Environment](#Environment)

[Variable](#Variable)

[RoleInstanceValue](#RoleInstanceValue)

[NetFxEntryPoint](#NetFxEntryPoint)

[Sites](#Sites)

[Site](#Site)

[VirtualApplication](#VirtualApplication)

[VirtualApplication](#VirtualApplication)

[Bindings](#Bindings)

[Binding](#Binding)

[Startup](#Startup)

[Tâche](#Task)

[Contents](#Contents)

[Contenu](#Content)

[SourceDirectory](#SourceDirectory)

##  <a name="webrole"></a><a name="WebRole"></a> WebRole  
L’élément `WebRole` décrit un rôle personnalisé pour la programmation d’applications web, tel que pris en charge par IIS 7 et ASP.NET. Un service peut contenir zéro ou plusieurs rôles web.

Le tableau suivant décrit les attributs d’un de l’élément `WebRole`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Nom du rôle web. Le nom du rôle doit être unique.|  
|enableNativeCodeExecution|boolean|facultatif. La valeur par défaut est `true` ; les fonctions d’exécution du code natif et de confiance totale sont activées par défaut. Définissez cet attribut sur `false` afin de désactiver l’exécution du code natif pour le rôle web, et utilisez à la place la fonction de confiance partielle Azure.|  
|vmSize|string|facultatif. Définissez cette valeur pour modifier la taille de la machine virtuelle allouée au rôle. La valeur par défaut est `Small`. Pour plus d’informations, consultez [Tailles de machine virtuelle pour les services cloud](cloud-services-sizes-specs.md).|  

##  <a name="configurationsettings"></a><a name="ConfigurationSettings"></a> ConfigurationSettings  
L’élément `ConfigurationSettings` décrit la collection de paramètres de configuration pour un rôle web. Cet élément est le parent de l’élément `Setting`.

##  <a name="setting"></a><a name="Setting"></a> Setting  
L’élément `Setting` décrit une paire nom-valeur qui spécifie un paramètre de configuration pour une instance d’un rôle.

Le tableau suivant décrit les attributs d’un de l’élément `Setting`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Nom unique du paramètre de configuration.|  

Les paramètres de configuration d’un rôle sont des paires nom-valeur déclarées dans le fichier de définition de service et définies dans le fichier de configuration de service.

##  <a name="localresources"></a><a name="LocalResources"></a> LocalResources  
L’élément `LocalResources` décrit la collection de ressources de stockage local pour un rôle web. Cet élément est le parent de l’élément `LocalStorage`.

##  <a name="localstorage"></a><a name="LocalStorage"></a> LocalStorage  
L’élément `LocalStorage` identifie une ressource de stockage local qui fournit l’espace de système de fichiers pour le service lors de l’exécution. Un rôle peut définir zéro ou plusieurs ressources de stockage local.

> [!NOTE]
>  L’élément `LocalStorage` peut apparaître en tant qu’enfant de l’élément `WebRole` pour prendre en charge la compatibilité avec les versions antérieures du kit SDK Azure.

Le tableau suivant décrit les attributs d’un de l’élément `LocalStorage`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Nom unique du magasin local.|  
|cleanOnRoleRecycle|boolean|facultatif. Indique si le magasin local doit être nettoyé lorsque le rôle est redémarré. La valeur par défaut est `true`.|  
|sizeInMb|int|facultatif. Quantité souhaitée d’espace de stockage à allouer pour le magasin local, en Mo. S’il n’est pas spécifié, l’espace de stockage alloué par défaut est de 100 Mo. La quantité d’espace de stockage minimale pouvant être allouée est de 1 Mo.<br /><br /> La taille maximale des ressources locales dépend de la taille de la machine virtuelle. Pour plus d’informations, consultez [Tailles de machine virtuelle pour les services cloud](cloud-services-sizes-specs.md).|  
  
Le nom du répertoire alloué à la ressource de stockage local correspond à la valeur fournie pour l’attribut name.

##  <a name="endpoints"></a><a name="Endpoints"></a> Endpoints  
L’élément `Endpoints` décrit la collection de points de terminaison d’entrée (externes), internes et d’entrée d’instance pour un rôle. Cet élément est le parent des éléments `InputEndpoint`, `InternalEndpoint` et `InstanceInputEndpoint`.

Les points de terminaison d’entrée et internes sont alloués séparément. Un service peut cumuler un total de 25 points de terminaison d’entrée, d’entrée d’instance et internes qui peuvent être alloués aux 25 rôles autorisés dans un service. Par exemple, avec 5 rôles vous pouvez au choix allouer 5 points de terminaison d’entrée par rôle, 25 points de terminaison d’entrée à un même rôle ou 1 point de terminaison d’entrée à chacun des 25 rôles.

> [!NOTE]
>  Chaque rôle déployé nécessite une instance par rôle. L’approvisionnement par défaut d’un abonnement est limité à 20 cœurs, et donc à 20 instances d’un rôle. Si votre application nécessite plus d’instances que ce qui est fourni par l’approvisionnement par défaut, consultez [Support pour la facturation, la gestion des abonnements et les quotas](https://azure.microsoft.com/support/options/) pour obtenir plus d’informations sur l’augmentation de votre quota.

##  <a name="inputendpoint"></a><a name="InputEndpoint"></a> InputEndpoint  
L’élément `InputEndpoint` décrit un point de terminaison externe pour un rôle web.

Vous pouvez définir plusieurs points de terminaison combinant à la fois des points de terminaison HTTP, HTTPS, UDP et TCP. Vous pouvez spécifier le numéro de port de votre choix pour un point de terminaison d’entrée, mais les numéros de port précisés pour chaque rôle dans le service doivent être uniques. Par exemple, si vous spécifiez qu’un rôle web utilise le port 80 pour HTTP, et le port 443 pour HTTPS, vous pouvez ensuite spécifier qu’un deuxième rôle web utilise le port 8080 pour HTTP, et le port 8043 pour HTTPS.

Le tableau suivant décrit les attributs d’un de l’élément `InputEndpoint`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Nom unique du point de terminaison externe.|  
|protocol|string|Obligatoire. Protocole de transport du point de terminaison externe. Pour un rôle web, les valeurs possibles sont `HTTP`, `HTTPS`, `UDP` ou `TCP`.|  
|port|int|Obligatoire. Port du point de terminaison externe. Vous pouvez spécifier le numéro de port de votre choix, mais les numéros de port précisés pour chaque rôle dans le service doivent être uniques.<br /><br /> Les valeurs possibles sont comprises entre 1 et 65535 inclus (Kit SDK Azure version 1.7 ou supérieure).|  
|certificat|string|Obligatoire pour un point de terminaison HTTPS. Nom d’un certificat défini par un élément `Certificate`.|  
|localPort|int|facultatif. Spécifie le port utilisé pour les connexions internes sur le point de terminaison. L’attribut `localPort` mappe le port externe sur le point de terminaison à un port interne sur un rôle. Cela est utile dans les scénarios où un rôle doit communiquer avec un composant interne sur un port différent de celui qui est exposé en externe.<br /><br /> Si elle n’est pas spécifiée, la valeur de `localPort` est la même que celle de l’attribut `port`. Définissez la valeur de `localPort` sur « * » pour attribuer automatiquement un port non alloué, détectable à l’aide de l’API d’exécution.<br /><br /> Les valeurs possibles sont comprises entre 1 et 65535 inclus (Kit SDK Azure version 1.7 ou supérieure).<br /><br /> L’attribut `localPort` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.|  
|ignoreRoleInstanceStatus|boolean|facultatif. Lorsque la valeur de cet attribut est définie sur `true`, l’état d’un service est ignoré et le point de terminaison n’est pas supprimé par l’équilibreur de charge. Définir cette valeur sur `true` est utile pour déboguer les instances occupées d’un service. La valeur par défaut est `false`. **Remarque :**  un point de terminaison peut toujours recevoir du trafic, même si le rôle n'est pas à l'état Prêt.|  
|loadBalancerProbe|string|facultatif. Nom de la sonde de l’équilibreur de charge associée au point de terminaison d’entrée. Pour plus d’informations, consultez [Schéma LoadBalancerProbe](schema-csdef-loadbalancerprobe.md).|  

##  <a name="internalendpoint"></a><a name="InternalEndpoint"></a> InternalEndpoint  
L’élément `InternalEndpoint` décrit un point de terminaison interne pour un rôle web. Un point de terminaison interne est uniquement disponible pour les autres instances de rôle en cours d’exécution dans le service ; il n’est pas disponible pour les clients en dehors du service. Les rôles web qui n’incluent pas l’élément `Sites` ne peuvent avoir qu’un seul point de terminaison interne HTTP, UDP ou TCP.

Le tableau suivant décrit les attributs d’un de l’élément `InternalEndpoint`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Nom unique du point de terminaison interne.|  
|protocol|string|Obligatoire. Protocole de transport du point de terminaison interne. Les valeurs possibles sont `HTTP`, `TCP`, `UDP` ou `ANY`.<br /><br /> La valeur `ANY` spécifie que tous les protocoles et tous les ports sont autorisés.|  
|port|int|facultatif. Port utilisé pour les connexions internes à charge équilibrée sur le point de terminaison. Un point de terminaison à charge équilibrée utilise deux ports : le port utilisé pour l’adresse IP publique, et le port utilisé sur l’adresse IP privée. Ces adresses sont généralement définies sur le même port, mais vous pouvez utiliser des ports différents.<br /><br /> Les valeurs possibles sont comprises entre 1 et 65535 inclus (Kit SDK Azure version 1.7 ou supérieure).<br /><br /> L’attribut `Port` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.|  

##  <a name="instanceinputendpoint"></a><a name="InstanceInputEndpoint"></a> InstanceInputEndpoint  
L’élément `InstanceInputEndpoint` décrit un point de terminaison d’entrée d’instance pour un rôle web. Un point de terminaison d’entrée d’instance est associé à une instance de rôle spécifique à l’aide du réacheminement de port dans l’équilibreur de charge. Chaque point de terminaison d’entrée d’instance est mappé à un port spécifique à partir d’une plage de ports possibles. Cet élément est le parent de l’élément `AllocatePublicPortFrom`.

L’élément `InstanceInputEndpoint` n’est disponible que par le biais du kit SDK Azure version 1.7 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `InstanceInputEndpoint`.
  
| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Nom unique du point de terminaison.|  
|localPort|int|Obligatoire. Spécifie le port interne que toutes les instances de rôle doivent écouter pour recevoir le trafic entrant transféré à partir de l’équilibreur de charge. Les valeurs possibles sont comprises entre 1 et 65535 inclus.|  
|protocol|string|Obligatoire. Protocole de transport du point de terminaison interne. Les valeurs possibles sont `udp` ou `tcp`. Utilisez `tcp` pour le trafic http/https.|  
  
##  <a name="allocatepublicportfrom"></a><a name="AllocatePublicPortFrom"></a> AllocatePublicPortFrom  
L’élément `AllocatePublicPortFrom` décrit la plage de ports publics qui peut être utilisée par les clients externes pour accéder à chaque point de terminaison d’entrée d’instance. Le numéro de port public (adresse IP virtuelle) est alloué à partir de cette plage, et affecté à chaque point de terminaison d’instance de rôle individuel pendant le déploiement et la mise à jour du locataire. Cet élément est le parent de l’élément `FixedPortRange`.

L’élément `AllocatePublicPortFrom` n’est disponible que par le biais du kit SDK Azure version 1.7 ou supérieure.

##  <a name="fixedport"></a><a name="FixedPort"></a> FixedPort  
L’élément `FixedPort` spécifie le port du point de terminaison interne, ce qui permet les connexions à charge équilibrée sur le point de terminaison.

L’élément `FixedPort` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `FixedPort`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|port|int|Obligatoire. Port du point de terminaison interne. Cela a le même effet que de définir les attributs min et max de l’élément `FixedPortRange` sur le même port.<br /><br /> Les valeurs possibles sont comprises entre 1 et 65535 inclus (Kit SDK Azure version 1.7 ou supérieure).|  

##  <a name="fixedportrange"></a><a name="FixedPortRange"></a> FixedPortRange  
L’élément `FixedPortRange` spécifie la plage de ports affectés au point de terminaison interne ou au point de terminaison d’entrée d’instance, et définit le port utilisé pour les connexions à charge équilibrée sur le point de terminaison.

> [!NOTE]
>  L’élément `FixedPortRange` fonctionne différemment selon l’élément dans lequel il se trouve. Lorsque l’élément `FixedPortRange` se trouve dans l’élément `InternalEndpoint`, il ouvre tous les ports sur l’équilibreur de charge dans la plage des attributs min et max pour toutes les machines virtuelles sur lesquelles le rôle s’exécute. Lorsque l’élément `FixedPortRange` se trouve dans l’élément `InstanceInputEndpoint`, il ouvre seulement un port dans la plage des attributs min et max sur chaque machine virtuelle exécutant le rôle.

L’élément `FixedPortRange` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `FixedPortRange`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|min|int|Obligatoire. Port minimal dans la plage. Les valeurs possibles sont comprises entre 1 et 65535 inclus (Kit SDK Azure version 1.7 ou supérieure).|  
|max|string|Obligatoire. Port maximal dans la plage. Les valeurs possibles sont comprises entre 1 et 65535 inclus (Kit SDK Azure version 1.7 ou supérieure).|  

##  <a name="certificates"></a><a name="Certificates"></a> Certificates  
L’élément `Certificates` décrit la collection de certificats pour un rôle web. Cet élément est le parent de l’élément `Certificate`. Un rôle peut avoir n’importe quel nombre de certificats associés. Pour plus d’informations sur l’utilisation de l’élément certificates, consultez [Modifier le fichier de définition de service avec un certificat](cloud-services-configure-ssl-certificate-portal.md#step-2-modify-the-service-definition-and-configuration-files).

##  <a name="certificate"></a><a name="Certificate"></a> Certificate  
L’élément `Certificate` décrit un certificat associé à un rôle web.

Le tableau suivant décrit les attributs d’un de l’élément `Certificate`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Nom de ce certificat, utilisé pour y faire référence quand il est associé à un élément `InputEndpoint` HTTPS.|  
|storeLocation|string|Obligatoire. Emplacement du magasin de certificats dans lequel se trouve ce certificat sur la machine locale. Les valeurs possibles sont `CurrentUser` et `LocalMachine`.|  
|storeName|string|Obligatoire. Nom du magasin de certificats dans lequel se trouve ce certificat sur la machine locale. Les valeurs possibles comprennent les noms de magasin intégrés `My`, `Root`, `CA`, `Trust`, `Disallowed`, `TrustedPeople`, `TrustedPublisher`, `AuthRoot`, `AddressBook`, ou un nom de magasin personnalisé. Si un nom de magasin personnalisé est indiqué, le magasin est automatiquement créé.|  
|permissionLevel|string|facultatif. Spécifie les autorisations d’accès données aux processus de rôle. Si vous souhaitez que seuls les processus élevés puissent accéder à la clé privée, spécifiez l’autorisation `elevated`. Une autorisation `limitedOrElevated` permet à tous les processus de rôle d’accéder à la clé privée. Les valeurs possibles sont `limitedOrElevated` ou `elevated`. La valeur par défaut est `limitedOrElevated`.|  

##  <a name="imports"></a><a name="Imports"></a> Imports  
L’élément `Imports` décrit une collection de modules d’importation pour un rôle web qui ajoutent des composants au système d’exploitation invité. Cet élément est le parent de l’élément `Import`. Cet élément est facultatif et un rôle ne peut avoir qu’un bloc d’importations. 

L’élément `Imports` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

##  <a name="import"></a><a name="Import"></a> Import  
L’élément `Import` spécifie un module à ajouter au système d’exploitation invité.

L’élément `Import` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `Import`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|moduleName|string|Obligatoire. Nom du module à importer. Les modules d’importation valides sont les suivants :<br /><br /> -   RemoteAccess<br />-   RemoteForwarder<br />-   Diagnostics<br /><br /> Les modules RemoteAccess et RemoteForwarder vous permettent de configurer votre instance de rôle pour les connexions Bureau à distance. Pour plus d’informations, consultez [Activer la connexion Bureau à distance](cloud-services-role-enable-remote-desktop-new-portal.md).<br /><br /> Le module Diagnostics vous permet de collecter les données de diagnostic d’une instance de rôle.|  

##  <a name="runtime"></a><a name="Runtime"></a> Runtime  
L’élément `Runtime` décrit une collection de paramètres de variables d’environnement pour un rôle web qui contrôlent l’environnement d’exécution du processus hôte d’Azure. Cet élément est le parent de l’élément `Environment`. Cet élément est facultatif, et un rôle ne peut avoir qu’un seul bloc runtime.

L’élément `Runtime` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

Le tableau suivant décrit les attributs de l’élément `Runtime` :  

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|executionContext|string|facultatif. Spécifie le contexte dans lequel le processus de rôle est lancé. Le contexte par défaut est `limited`.<br /><br /> -   `limited` – Le processus est lancé sans privilèges d’administrateur.<br />-   `elevated` – Le processus est lancé avec des privilèges d’administrateur.|  

##  <a name="environment"></a><a name="Environment"></a> Environment  
L’élément `Environment` décrit une collection de paramètres de variables d’environnement pour un rôle web. Cet élément est le parent de l’élément `Variable`. Un rôle peut avoir n’importe quel nombre de variables d’environnement définies.

##  <a name="variable"></a><a name="Variable"></a> Variable  
L’élément `Variable` spécifie une variable d’environnement à définir dans le système d’exploitation invité.

L’élément `Variable` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

Le tableau suivant décrit les attributs de l’élément `Variable` :  

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Nom de la variable d'environnement à définir.|  
|value|string|facultatif. Valeur à définir pour la variable d’environnement. Vous devez inclure un attribut value ou un élément `RoleInstanceValue`.|  

##  <a name="roleinstancevalue"></a><a name="RoleInstanceValue"></a> RoleInstanceValue  
L’élément `RoleInstanceValue` spécifie l’attribut xPath à partir duquel récupérer la valeur de la variable.

Le tableau suivant décrit les attributs d’un de l’élément `RoleInstanceValue`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|xpath|string|facultatif. Chemin de l’emplacement des paramètres de déploiement pour l’instance. Pour plus d’informations, consultez [Variables de configuration avec XPath](cloud-services-role-config-xpath.md).<br /><br /> Vous devez inclure un attribut value ou un élément `RoleInstanceValue`.|  

##  <a name="entrypoint"></a><a name="EntryPoint"></a> EntryPoint  
L’élément `EntryPoint` spécifie le point d’entrée d’un rôle. Cet élément est le parent des éléments `NetFxEntryPoint`. Ces éléments vous permettent de spécifier une autre application que l’application par défaut WaWorkerHost.exe pour agir comme point d’entrée du rôle.

L’élément `EntryPoint` n’est disponible que par le biais du kit SDK Azure version 1.5 ou supérieure.

##  <a name="netfxentrypoint"></a><a name="NetFxEntryPoint"></a> NetFxEntryPoint  
L’élément `NetFxEntryPoint` spécifie le programme à exécuter pour un rôle.

> [!NOTE]
>  L’élément `NetFxEntryPoint` n’est disponible que par le biais du kit SDK Azure version 1.5 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `NetFxEntryPoint`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|assemblyName|string|Obligatoire. Chemin et nom de fichier de l’assembly contenant le point d’entrée. Le chemin est relatif au dossier **\\%ROLEROOT%\Approot** (ne spécifiez pas **\\%ROLEROOT%\Approot** dans `commandLine`, car il est supposé). **%ROLEROOT%** est une variable d’environnement gérée par Azure, qui représente l’emplacement du dossier racine de votre rôle. Le dossier **\\%ROLEROOT%\Approot** désigne le dossier d’application de votre rôle.<br /><br /> Pour les rôles HWC, le chemin est toujours relatif au dossier **\\%ROLEROOT%\Approot\bin**.<br /><br /> S’il est impossible de trouver l’assembly relatif au dossier **\\%ROLEROOT%\Approot** des rôles web IIS et IIS Express complets, le dossier **\\%ROLEROOT%\Approot\bin** est recherché.<br /><br /> Ce comportement de secours pour le rôle web IIS complet n’est pas une bonne pratique recommandée, et il est possible qu’il soit supprimé dans les prochaines versions.|  
|targetFrameworkVersion|string|Obligatoire. La version de .NET framework sur laquelle l’assembly a été créé. Par exemple : `targetFrameworkVersion="v4.0"`.|  

##  <a name="sites"></a><a name="Sites"></a> Sites  
L’élément `Sites` décrit une collection d’applications web et de sites web hébergés dans un rôle web. Cet élément est le parent de l’élément `Site`. Si vous ne spécifiez aucun élément `Sites`, votre rôle web est hébergé en tant que rôle web hérité, et vous ne pouvez disposer que d’un seul site web hébergé dans votre rôle web. Cet élément est facultatif, et un rôle ne peut avoir qu’un seul bloc sites.

L’élément `Sites` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

##  <a name="site"></a><a name="Site"></a> Site  
L’élément `Site` spécifie une application web ou un site web faisant partie du rôle web.

L’élément `Site` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `Site`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Nom de l’application ou du site web.|  
|physicalDirectory|string|Emplacement du répertoire de contenu pour la racine du site. L’emplacement peut être spécifié en tant que chemin absolu ou relatif à l’emplacement .csdef.|  

##  <a name="virtualapplication"></a><a name="VirtualApplication"></a> VirtualApplication  
L’élément `VirtualApplication` définit une application dans Internet Information Services (IIS) 7 en tant que regroupement de fichiers fournissant du contenu ou des services via des protocoles, tels que HTTP. Lorsque vous créez une application dans IIS 7, son chemin est intégré à l’URL du site.

L’élément `VirtualApplication` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `VirtualApplication`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Spécifie un nom pour identifier l’application virtuelle.|  
|physicalDirectory|string|Obligatoire. Spécifie le chemin sur la machine de développement contenant l’application virtuelle. Dans l’émulateur de calcul, IIS est configuré pour récupérer le contenu à partir de cet emplacement. Lors du déploiement sur Azure, le contenu du répertoire physique est empaqueté avec le reste du service. Lorsque le package de service est déployé sur Azure, IIS est configuré avec l’emplacement du contenu non empaqueté.|  

##  <a name="virtualdirectory"></a><a name="VirtualDirectory"></a> VirtualDirectory  
L’élément `VirtualDirectory` spécifie un nom de répertoire (également appelé chemin) que vous indiquez dans IIS et mappez à un répertoire physique sur un serveur local ou distant.

L’élément `VirtualDirectory` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `VirtualDirectory`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Spécifie un nom pour identifier le répertoire virtuel.|  
|value|physicalDirectory|Obligatoire. Spécifie le chemin sur la machine de développement contenant le site web ou le contenu du répertoire virtuel. Dans l’émulateur de calcul, IIS est configuré pour récupérer le contenu à partir de cet emplacement. Lors du déploiement sur Azure, le contenu du répertoire physique est empaqueté avec le reste du service. Lorsque le package de service est déployé sur Azure, IIS est configuré avec l’emplacement du contenu non empaqueté.|  

##  <a name="bindings"></a><a name="Bindings"></a> Bindings  
L’élément `Bindings` décrit une collection de liaisons d’un site web. Il est l’élément parent de l’élément `Binding`. Cet élément est nécessaire pour chaque élément `Site`. Pour plus d’informations sur la configuration des points de terminaison, consultez [Activer la communication pour les instances de rôle](cloud-services-enable-communication-role-instances.md).

L’élément `Bindings` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

##  <a name="binding"></a><a name="Binding"></a> Binding  
L’élément `Binding` spécifie les informations de configuration nécessaires aux demandes pour communiquer avec un site web ou une application web.

L’élément `Binding` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|name|string|Obligatoire. Spécifie un nom pour identifier la liaison.|  
|endpointName|string|Obligatoire. Spécifie le nom de point de terminaison à lier.|  
|hostHeader|string|facultatif. Spécifie un nom d’hôte qui vous permet d’héberger plusieurs sites, avec différents noms d’hôte, sur une combinaison unique Adresse IP-Numéro de port.|  

##  <a name="startup"></a><a name="Startup"></a> Startup  
L’élément `Startup` décrit une collection de tâches qui s’exécutent au démarrage du rôle. Cet élément peut être le parent de l’élément `Variable`. Pour plus d’informations sur l’utilisation des tâches de démarrage de rôle, consultez [Guide pratique pour configurer des tâches de démarrage](cloud-services-startup-tasks.md). Cet élément est facultatif et un rôle peut n’avoir qu’un seul bloc startup.

Le tableau suivant décrit l’attribut de l’élément `Startup`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|priority|int|À usage interne uniquement.|  

##  <a name="task"></a><a name="Task"></a> Task  
L’élément `Task` spécifie la tâche de démarrage qui s’exécute au démarrage du rôle. Les tâches de démarrage peuvent être utilisées pour réaliser des tâches en préparation de l’exécution du rôle, telles que l’installation de composants logiciels ou l’exécution d’autres applications. Les tâches s’exécutent dans l’ordre selon lequel elles apparaissent dans le bloc de l’élément `Startup`.

L’élément `Task` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `Task`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|commandLine|string|Obligatoire. Script, comme un fichier CMD, contenant les commandes à exécuter. La commande de démarrage et les fichiers de commandes doivent être enregistrés au format ANSI. Les formats de fichier qui définissent un indicateur d’ordre des octets au début du fichier ne sont pas traités correctement.|  
|executionContext|string|Spécifie le contexte dans lequel le script est exécuté.<br /><br /> -   `limited` [Par défaut] – Exécution avec les mêmes privilèges que le rôle qui héberge le processus.<br />-   `elevated` – Exécution avec des privilèges d’administrateur.|  
|taskType|string|Spécifie le comportement d’exécution de la commande.<br /><br /> -   `simple` [Par défaut] – Le système attend que la tâche se termine avant de lancer d’autres tâches.<br />-   `background` – Le système n’attend pas que la tâche se termine.<br />-   `foreground` – Similaire à background, à la différence près que le rôle n’est redémarré que quand toutes les tâches foreground sont terminées.|  

##  <a name="contents"></a><a name="Contents"></a> Contents  
L’élément `Contents` décrit la collection de contenus pour un rôle web. Cet élément est le parent de l’élément `Content`.

L’élément `Contents` n’est disponible que par le biais du kit SDK Azure version 1.5 ou supérieure.

##  <a name="content"></a><a name="Content"></a> Content  
L’élément `Content` définit l’emplacement source du contenu à copier sur la machine virtuelle Azure, et le chemin de destination dans lequel il est copié.

L’élément `Content` n’est disponible que par le biais du kit SDK Azure version 1.5 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `Content`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|destination|string|Obligatoire. Emplacement sur la machine virtuelle Azure dans lequel le contenu est placé. Cet emplacement est relatif au dossier **%ROLEROOT%\Approot**.|  

Cet élément est le parent de l’élément `SourceDirectory`.

##  <a name="sourcedirectory"></a><a name="SourceDirectory"></a> SourceDirectory  
L’élément `SourceDirectory` définit le répertoire local à partir duquel le contenu est copié. Utilisez cet élément pour spécifier le contenu local à copier sur la machine virtuelle Azure.

L’élément `SourceDirectory` n’est disponible que par le biais du kit SDK Azure version 1.5 ou supérieure.

Le tableau suivant décrit les attributs d’un de l’élément `SourceDirectory`.

| Attribut | Type | Description |  
| --------- | ---- | ----------- |  
|path|string|Obligatoire. Chemin relatif ou absolu d’un répertoire local dont le contenu doit être copié sur la machine virtuelle Azure. L’extension des variables d’environnement dans le chemin du répertoire est prise en charge.|  
  
## <a name="see-also"></a>Voir aussi
[Schéma de définition de services cloud (classique)](schema-csdef-file.md)




