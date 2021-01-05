---
title: Référence YAML pour le groupe de conteneurs
description: Informations de référence pour le fichier YAML pris en charge par Azure Container Instances pour configurer un groupe de conteneurs
ms.topic: article
ms.date: 07/06/2020
ms.openlocfilehash: d0ec8d13eebba1c60f5a52f8c43bdd8b90eeb913
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87084758"
---
# <a name="yaml-reference-azure-container-instances"></a>Informations de référence YAML : Azure Container Instances

Cet article traite de la syntaxe et des propriétés du fichier YAML pris en charge par Azure Container Instances pour configurer un [groupe de conteneurs](container-instances-container-groups.md). Vous pouvez utiliser un fichier YAML pour entrer la configuration du groupe dans la commande [az container create][az-container-create] dans Azure CLI. 

Un fichier YAML est un moyen pratique de configurer un groupe de conteneurs pour des déploiements reproductibles. Plutôt que d’utiliser un [modèle Resource Manager](/azure/templates/Microsoft.ContainerInstance/2019-12-01/containerGroups) ou les kits SDK Azure Container Instances pour créer ou mettre à jour un groupe de conteneurs, il s’agit d’une solution de rechange succincte.

> [!NOTE]
> Ces informations de référence s’appliquent aux fichiers YAML pour l’API REST Azure Container Instances version `2019-12-01`.

## <a name="schema"></a>schéma 

Le schéma du fichier YAML vous est présenté ci-dessous. Il comporte des commentaires pour mettre en avant les principales propriétés. Pour obtenir une description des propriétés de ce schéma, consultez la section [Valeurs de propriétés](#property-values).

```yml
name: string  # Name of the container group
apiVersion: '2019-12-01'
location: string
tags: {}
identity: 
  type: string
  userAssignedIdentities: {}
properties: # Properties of container group
  containers: # Array of container instances in the group
  - name: string # Name of an instance
    properties: # Properties of an instance
      image: string # Container image used to create the instance
      command:
      - string
      ports: # External-facing ports exposed on the instance, must also be set in group ipAddress property 
      - protocol: string
        port: integer
      environmentVariables:
      - name: string
        value: string
        secureValue: string
      resources: # Resource requirements of the instance
        requests:
          memoryInGB: number
          cpu: number
          gpu:
            count: integer
            sku: string
        limits:
          memoryInGB: number
          cpu: number
          gpu:
            count: integer
            sku: string
      volumeMounts: # Array of volume mounts for the instance
      - name: string
        mountPath: string
        readOnly: boolean
      livenessProbe:
        exec:
          command:
          - string
        httpGet:
          path: string
          port: integer
          scheme: string
        initialDelaySeconds: integer
        periodSeconds: integer
        failureThreshold: integer
        successThreshold: integer
        timeoutSeconds: integer
      readinessProbe:
        exec:
          command:
          - string
        httpGet:
          path: string
          port: integer
          scheme: string
        initialDelaySeconds: integer
        periodSeconds: integer
        failureThreshold: integer
        successThreshold: integer
        timeoutSeconds: integer
  imageRegistryCredentials: # Credentials to pull a private image
  - server: string
    username: string
    password: string
  restartPolicy: string
  ipAddress: # IP address configuration of container group
    ports:
    - protocol: string
      port: integer
    type: string
    ip: string
    dnsNameLabel: string
  osType: string
  volumes: # Array of volumes available to the instances
  - name: string
    azureFile:
      shareName: string
      readOnly: boolean
      storageAccountName: string
      storageAccountKey: string
    emptyDir: {}
    secret: {}
    gitRepo:
      directory: string
      repository: string
      revision: string
  diagnostics:
    logAnalytics:
      workspaceId: string
      workspaceKey: string
      logType: string
      metadata: {}
  networkProfile: # Virtual network profile for container group
    id: string
  dnsConfig: # DNS configuration for container group
    nameServers:
    - string
    searchDomains: string
    options: string
  sku: string # SKU for the container group
  encryptionProperties:
    vaultBaseUrl: string
    keyName: string
    keyVersion: string
  initContainers: # Array of init containers in the group
  - name: string
    properties:
      image: string
      command:
      - string
      environmentVariables:
      - name: string
        value: string
        secureValue: string
      volumeMounts:
      - name: string
        mountPath: string
        readOnly: boolean
```

## <a name="property-values"></a>Valeurs de propriétés

Les tableaux suivants décrivent les valeurs que vous devez définir dans le schéma.



### <a name="microsoftcontainerinstancecontainergroups-object"></a>Objet Microsoft.ContainerInstance/containerGroups

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  name | string | Oui | Nom du groupe de conteneurs. |
|  apiVersion | enum | Oui | 2018-10-01 |
|  location | string | Non | Emplacement de la ressource. |
|  tags | object | Non | Étiquettes de la ressource. |
|  identité | object | Non | Identité du groupe de conteneurs, si elle est configurée. - [Objet ContainerGroupIdentity](#containergroupidentity-object) |
|  properties | object | Oui | [Objet ContainerGroupProperties](#containergroupproperties-object) |




### <a name="containergroupidentity-object"></a>Objet ContainerGroupIdentity

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  type | enum | Non | Type d’identité utilisé pour le groupe de conteneurs. Le type « SystemAssigned, UserAssigned » comprend à la fois une identité créée implicitement et un ensemble d’identités attribuées par l’utilisateur. Le type « None » supprime les identités du groupe de conteneurs. - SystemAssigned, UserAssigned, SystemAssigned, UserAssigned, None |
|  userAssignedIdentities | object | Non | Liste d’identités utilisateur associées au groupe de conteneurs. Les références clés du dictionnaire d’identités utilisateur seront les ID de ressource Azure Resource Manager sous la forme : '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identityName}'. |




### <a name="containergroupproperties-object"></a>Objet ContainerGroupProperties

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  containers | tableau | Oui | Conteneurs présents dans le groupe de conteneurs. - [Objet Container](#container-object) |
|  imageRegistryCredentials | tableau | Non | Informations d’identification du registre d’images à partir desquelles le groupe de conteneurs est créé. - [Objet ImageRegistryCredential](#imageregistrycredential-object) |
|  restartPolicy | enum | Non | Stratégie de redémarrage pour tous les conteneurs présents dans le groupe de conteneurs. - `Always` Toujours redémarrer – `OnFailure` Redémarrer en cas d’échec – `Never` Ne jamais redémarrer. - Always, OnFailure, Never |
|  ipAddress | object | Non | Type d’adresse IP du groupe de conteneurs. - [Objet IpAddress](#ipaddress-object) |
|  osType | enum | Oui | Type de système d’exploitation exigé par les conteneurs présents dans le groupe de conteneurs. - Windows ou Linux |
|  volumes | tableau | Non | Liste des volumes qui peuvent être montés par les conteneurs présents dans ce groupe de conteneurs. - [Objet Volume](#volume-object) |
|  diagnostics | object | Non | Informations de diagnostic pour un groupe de conteneurs. - [Objet ContainerGroupDiagnostics](#containergroupdiagnostics-object) |
|  networkProfile | object | Non | Informations de profil réseau pour un groupe de conteneurs. - [Objet ContainerGroupNetworkProfile](#containergroupnetworkprofile-object) |
|  dnsConfig | object | Non | Informations de configuration DNS pour un groupe de conteneurs. - [Objet DnsConfiguration](#dnsconfiguration-object) |
| sku | enum | Non | La référence (SKU) pour un groupe de conteneurs : standard ou dédié |
| encryptionProperties | object | Non | Propriétés de chiffrement pour un groupe de conteneurs. - [Objet EncryptionProperties](#encryptionproperties-object) | 
| initContainers | tableau | Non | Conteneurs d'initialisation d'un groupe de conteneurs. - [Objet InitContainerDefinition](#initcontainerdefinition-object) |




### <a name="container-object"></a>Objet Container

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  name | string | Oui | Nom fourni par l’utilisateur de l’instance de conteneur. |
|  properties | object | Oui | Propriétés de l’instance de conteneur. - [Objet ContainerProperties](#containerproperties-object) |




### <a name="imageregistrycredential-object"></a>Objet ImageRegistryCredential

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  server | string | Oui | Serveur de registre d’images Docker sans protocole comme « http » ou « https ». |
|  username | string | Oui | Nom d’utilisateur du registre privé. |
|  mot de passe | string | Non | Mot de passe du registre privé. |




### <a name="ipaddress-object"></a>Objet IpAddress

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  ports | tableau | Oui | Liste des ports exposés dans le groupe de conteneurs. - [Objet Port](#port-object) |
|  type | enum | Oui | Indique si l’adresse IP est exposée à l’Internet public ou à un réseau virtuel privé. - Public ou Private |
|  ip | string | Non | Adresse IP exposée à l’Internet public. |
|  dnsNameLabel | string | Non | Étiquette du nom DNS de l’adresse IP. |




### <a name="volume-object"></a>Objet Volume

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  name | string | Oui | Nom du volume. |
|  azureFile | object | Non | Volume de fichier Azure. - [Objet AzureFileVolume](#azurefilevolume-object) |
|  emptyDir | object | Non | Volume de répertoire vide. |
|  secret | object | Non | Volume de secret. |
|  gitRepo | object | Non | Volume de dépôt git. - [Objet GitRepoVolume](#gitrepovolume-object) |




### <a name="containergroupdiagnostics-object"></a>Objet ContainerGroupDiagnostics

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  logAnalytics | object | Non | Informations analytiques des journaux du groupe de conteneurs. - [Objet logAnalytics](#loganalytics-object) |




### <a name="containergroupnetworkprofile-object"></a>Objet ContainerGroupNetworkProfile

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  id | string | Oui | Identificateur d’un profil réseau. |




### <a name="dnsconfiguration-object"></a>Objet DnsConfiguration

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  nameServers | tableau | Oui | Serveurs DNS du groupe de conteneurs. - string |
|  searchDomains | string | Non | Domaines de recherche DNS pour la recherche de nom d’hôte dans le groupe de conteneurs. |
|  options | string | Non | Options DNS du groupe de conteneurs. |


### <a name="encryptionproperties-object"></a>Objet EncryptionProperties

| Nom  | Type  | Obligatoire  | Valeur |
|  ---- | ---- | ---- | ---- |
| vaultBaseUrl  | string    | Oui   | URL de base du coffre de clés. |
| keyName   | string    | Oui   | Nom de la clé de chiffrement. |
| keyVersion    | string    | Oui   | Version de la clé de chiffrement. |

### <a name="initcontainerdefinition-object"></a>Objet InitContainerDefinition

| Nom  | Type  | Obligatoire  | Valeur |
|  ---- | ---- | ---- | ---- |
| name  | string |  Oui | Nom du conteneur d'initialisation. |
| properties    | object    | Oui   | Propriétés du conteneur init. - [Objet InitContainerPropertiesDefinition](#initcontainerpropertiesdefinition-object)


### <a name="containerproperties-object"></a>Objet ContainerProperties

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  image | string | Oui | Nom de l’image utilisée pour créer l’instance de conteneur. |
|  command | tableau | Non | Commandes à exécuter dans l’instance de conteneur dans l’exec form. - string |
|  ports | tableau | Non | Port exposé dans l’instance de conteneur. - [Objet ContainerPort](#containerport-object) |
|  environmentVariables | tableau | Non | Variable d’environnement à définir dans l’instance de conteneur. - [Objet EnvironmentVariable](#environmentvariable-object) |
|  les ressources | object | Oui | Besoins en ressources de l’instance de conteneur. - [Objet ResourceRequirements](#resourcerequirements-object) |
|  volumeMounts | tableau | Non | Montages de volume accessibles à l’instance de conteneur. - [Objet VolumeMount](#volumemount-object) |
|  livenessProbe | object | Non | Probe liveness. - [Objet ContainerProbe](#containerprobe-object) |
|  readinessProbe | object | Non | Probe readiness. - [Objet ContainerProbe](#containerprobe-object) |




### <a name="port-object"></a>Objet Port

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  protocol | enum | Non | Protocole associé au port. - TCP ou UDP |
|  port | entier | Oui | Numéro de port. |




### <a name="azurefilevolume-object"></a>Objet AzureFileVolume

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  shareName | string | Oui | Nom du partage de fichiers Azure à monter en tant que volume. |
|  readOnly | boolean | Non | Indicateur précisant si le partage de fichiers Azure monté en tant que volume est en lecture seule. |
|  storageAccountName | string | Oui | Nom du compte de stockage qui contient le partage de fichiers Azure. |
|  storageAccountKey | string | Non | Clé d’accès du compte de stockage utilisée pour accéder au partage de fichiers Azure. |




### <a name="gitrepovolume-object"></a>Objet GitRepoVolume

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  directory | string | Non | Nom du répertoire cible. Ne doit pas contenir ou commencer par « .. ».  Si « . » est spécifié, le répertoire du volume est le dépôt git.  Sinon, s’il est spécifié, le volume contient le dépôt git dans le sous-répertoire avec le nom donné. |
|  repository | string | Oui | URL du dépôt |
|  revision | string | Non | Hachage de validation pour la révision spécifiée. |



### <a name="loganalytics-object"></a>Objet LogAnalytics

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  workspaceId | string | Oui | ID d’espace de travail pour l’analytique des journaux. |
|  workspaceKey | string | Oui | Clé d’espace de travail pour l’analytique des journaux. |
|  logType | enum | Non | Type de journal à utiliser. - ContainerInsights ou ContainerInstanceLogs |
|  metadata | object | Non | Métadonnées pour l’analytique des journaux. |


### <a name="initcontainerpropertiesdefinition-object"></a>Objet InitContainerPropertiesDefinition

| Nom  | Type  | Obligatoire  | Valeur |
|  ---- | ---- | ---- | ---- |
| image | string    | Non    | Image du conteneur d'initialisation. |
| command   | tableau | Non    | Commande à exécuter dans le conteneur d'initialisation dans le formulaire exec. - string |
| environmentVariables | tableau  | Non |Variables d'environnement à définir dans le conteneur d'initialisation. - [Objet EnvironmentVariable](#environmentvariable-object)
| volumeMounts |tableau   | Non    | Montages de volumes disponibles pour le conteneur d'initialisation. - [Objet VolumeMount](#volumemount-object)

### <a name="containerport-object"></a>Objet ContainerPort

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  protocol | enum | Non | Protocole associé au port. - TCP ou UDP |
|  port | entier | Oui | Numéro de port exposé dans le groupe de conteneurs. |




### <a name="environmentvariable-object"></a>Objet EnvironmentVariable

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  name | string | Oui | Nom de la variable d’environnement. |
|  value | string | Non | Valeur de la variable d’environnement. |
|  secureValue | string | Non | Valeur de la variable d’environnement sécurisée. |




### <a name="resourcerequirements-object"></a>Objet ResourceRequirements

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  requêtes | object | Oui | Demandes en ressources de cette instance de conteneur. - [Objet ResourceRequests](#resourcerequests-object) |
|  limites | object | Non | Limites en ressources de cette instance de conteneur. - [Objet ResourceLimits](#resourcelimits-object) |




### <a name="volumemount-object"></a>Objet VolumeMount

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  name | string | Oui | Nom du montage de volume. |
|  mountPath | string | Oui | Chemin de l’emplacement dans le conteneur où le volume doit être monté. Ne doit pas contenir de signe deux-points (:). |
|  readOnly | boolean | Non | Indicateur précisant si le montage du volume est en lecture seule. |




### <a name="containerprobe-object"></a>Objet ContainerProbe

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  exec | object | Non | Commande d’exécution à sonder – [Objet ContainerExec](#containerexec-object) |
|  httpGet | object | Non | Paramètres Http Get à sonder – [Objet ContainerHttpGet](#containerhttpget-object) |
|  initialDelaySeconds | entier | Non | Retard initial en secondes. |
|  periodSeconds | entier | Non | Période en secondes. |
|  failureThreshold | entier | Non | Seuil de défaillance. |
|  successThreshold | entier | Non | Seuil de réussite. |
|  timeoutSeconds | entier | Non | Délai d’attente en secondes. |




### <a name="resourcerequests-object"></a>Objet ResourceRequests

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  memoryInGB | nombre | Oui | Demande de mémoire en Go de cette instance de conteneur. |
|  cpu | nombre | Oui | Demande de processeur de cette instance de conteneur. |
|  gpu | object | Non | Demande GPU de cette instance de conteneur. - [Objet GpuResource](#gpuresource-object) |




### <a name="resourcelimits-object"></a>Objet ResourceLimits

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  memoryInGB | nombre | Non | Limite de mémoire en Go de cette instance de conteneur. |
|  cpu | nombre | Non | Limite processeur de cette instance de conteneur. |
|  gpu | object | Non | Limite GPU de cette instance de conteneur. - [Objet GpuResource](#gpuresource-object) |




### <a name="containerexec-object"></a>Objet ContainerExec

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  command | tableau | Non | Commandes à exécuter dans le conteneur. - string |




### <a name="containerhttpget-object"></a>Objet ContainerHttpGet

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  path | string | Non | Chemin de la sonde. |
|  port | entier | Oui | Numéro de port vers la sonde. |
|  scheme | enum | Non | Schéma. - http ou https |




### <a name="gpuresource-object"></a>Objet GpuResource

|  Nom | Type | Obligatoire | Valeur |
|  ---- | ---- | ---- | ---- |
|  count | entier | Oui | Nombre de la ressource GPU. |
|  sku | enum | Oui | Référence SKU de la ressource GPU. - K80, P100, V100 |


## <a name="next-steps"></a>Étapes suivantes

Consultez le tutoriel [Déployer un groupe multiconteneur à l’aide d’un fichier YAML](container-instances-multi-container-yaml.md).

Consultez des exemples d’utilisation d’un fichier YAML pour déployer des groupes de conteneurs dans un [réseau virtuel](container-instances-vnet.md) ou pour [monter un volume externe](container-instances-volume-azure-files.md).

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create

