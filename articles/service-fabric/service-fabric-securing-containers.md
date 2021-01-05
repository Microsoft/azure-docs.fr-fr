---
title: Importer des certificats dans un conteneur
description: En savoir plus sur l’importation des fichiers de certificat dans un service de conteneurs Service Fabric.
ms.topic: conceptual
ms.date: 2/23/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 219882a3f7f6db665f1ec311098ef53464773b71
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92313698"
---
# <a name="import-a-certificate-file-into-a-container-running-on-service-fabric"></a>Importer un fichier de certificat dans un conteneur en cours d’exécution sur Service Fabric

> [!NOTE]
> Pour les clusters Service Fabric fonctionnant sur Azure, il est recommandé d’utiliser [Service Fabric Application Managed Identity](./concepts-managed-identity.md) pour approvisionner des certificats d’application à partir d’un conteneur. Managed Identity permet d’isoler les secrets et les certificats au niveau du service, et permet à l’approvisionnement des certificats d’application d’être inclus dans le workflow de l’application, plutôt que dans celui de l’infrastructure. Le mécanisme CertificateRef sera déconseillé dans une version ultérieure.

Vous pouvez sécuriser vos services de conteneur en spécifiant un certificat. Service Fabric fournit un mécanisme pour les services à l’intérieur d’un conteneur pour accéder à un certificat installé sur les nœuds dans un cluster Windows ou Linux (version 5.7 ou version ultérieure). Le certificat doit être installé dans un magasin de certificats sous tous les nœuds du cluster de LocalMachine. La clé privée correspondant au certificat doit être disponible, accessible et - sur Windows - exportable. Les informations de certificat sont fournies dans le manifeste d’application avec la balise `ContainerHostPolicies` comme le montre l’extrait de code suivant :

```xml
  <ContainerHostPolicies CodePackageRef="NodeContainerService.Code">
    <CertificateRef Name="MyCert1" X509StoreName="My" X509FindValue="[Thumbprint1]"/>
    <CertificateRef Name="MyCert2" X509FindValue="[Thumbprint2]"/>
 ```

Pour les clusters Windows, lors du démarrage de l’application, le runtime exporte chaque certificat référencé ainsi que la clé privée correspondante dans un fichier PFX, sécurisé à l'aide d'un mot de passe généré de façon aléatoire. Les fichiers PFX et de mot de passe sont accessibles à l’intérieur du conteneur à l’aide des variables d’environnement suivantes : 

* Certificates_ServicePackageName_CodePackageName_CertName_PFX
* Certificates_ServicePackageName_CodePackageName_CertName_Password

Pour les clusters Linux, les certificats (PEM) sont copiés à partir du magasin spécifié par X509StoreName sur le conteneur. Voici les variables d’environnement correspondantes sous Linux :

* Certificates_ServicePackageName_CodePackageName_CertName_PEM
* Certificates_ServicePackageName_CodePackageName_CertName_PrivateKey

Notez que les fichiers `PEM` et `PrivateKey` contiennent le certificat et la clé privée non chiffrée.

Si vous disposez déjà des certificats au format requis et souhaitez y accéder à l’intérieur du conteneur, vous pouvez également créer un package de données à l’intérieur de votre package de l’application et spécifier les éléments suivants au sein de votre manifeste de l’application :

```xml
<ContainerHostPolicies CodePackageRef="NodeContainerService.Code">
  <CertificateRef Name="MyCert1" DataPackageRef="[DataPackageName]" DataPackageVersion="[Version]" RelativePath="[Relative Path to certificate inside DataPackage]" Password="[password]" IsPasswordEncrypted="[true/false]"/>
 ```

Le processus ou le service de conteneur est responsable de l’importation des fichiers de certificats dans le conteneur. Pour importer le certificat, vous pouvez utiliser les scripts `setupentrypoint.sh` ou exécuter un code personnalisé au sein du processus de conteneur. Exemple de code en C# pour importer le fichier PFX :

```csharp
string certificateFilePath = Environment.GetEnvironmentVariable("Certificates_MyServicePackage_NodeContainerService.Code_MyCert1_PFX");
string passwordFilePath = Environment.GetEnvironmentVariable("Certificates_MyServicePackage_NodeContainerService.Code_MyCert1_Password");
X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
string password = File.ReadAllLines(passwordFilePath, Encoding.Default)[0];
password = password.Replace("\0", string.Empty);
X509Certificate2 cert = new X509Certificate2(certificateFilePath, password, X509KeyStorageFlags.MachineKeySet | X509KeyStorageFlags.PersistKeySet);
store.Open(OpenFlags.ReadWrite);
store.Add(cert);
store.Close();
```
Ce certificat PFX peut être utilisé pour authentifier l’application ou le service afin de sécuriser la communication avec d’autres services. Par défaut, les fichiers ne sont présents que dans la liste ACL SYSTEM. Vous pouvez les ajouter à d’autres comptes selon les exigences du service.

Comme prochaine étape, lisez les articles suivants :

* [Déployer un conteneur Windows sur Service Fabric sous Windows Server 2016](service-fabric-get-started-containers.md)
* [Déployer un conteneur Docker sur Service Fabric sous Linux](service-fabric-get-started-containers-linux.md)