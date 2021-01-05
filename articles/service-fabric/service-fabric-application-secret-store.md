---
title: Banque de secrets centrale Azure Service Fabric
description: Cet article explique comment utiliser la banque de secrets centrale (CSS, Central Secrets Store) dans Azure Service Fabric.
ms.topic: conceptual
ms.date: 07/25/2019
ms.openlocfilehash: e9fd435803ad5354b0eb2d4f5de50009a8cbbfe2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88869753"
---
# <a name="central-secrets-store-in-azure-service-fabric"></a>Banque de secrets centrale dans Azure Service Fabric 
Cet article explique comment utiliser la banque de secrets centrale (CSS, Central Secrets Store) dans Azure Service Fabric pour créer des secrets dans des applications Service Fabric. CSS est une banque de secrets locale qui garde chiffrées et en mémoire les données sensibles que sont notamment les mots de passe, les jetons et les clés.

  > [!NOTE] 
  > Lors de la première activation de la CSS avant la version 7.1 de SF CU3 (mise à jour cumulative 3), l'activation peut échouer et laisser la CSS dans un état non sain permanent si : la CSS est activée sur un cluster authentifié par Windows ; la CSS est activée sur n'importe quel cluster mais que `EncryptionCertificateThumbprint` est déclaré de manière incorrecte ou que le certificat correspondant n'est pas installé / placé dans la liste de contrôle d'accès sur les nœuds. Pour le cluster d'authentification Windows, passez à la version 7.1 CU3 avant de continuer. Pour les autres clusters, veuillez vérifier ces invariants ou passer à la version 7.1 CU3.
  
## <a name="enable-central-secrets-store"></a>Activer la banque de secrets centrale
Ajoutez le script suivant à la configuration de votre cluster sous `fabricSettings` pour activer CSS. Nous vous recommandons d’utiliser un certificat autre qu’un certificat de cluster pour CSS. Vérifiez que le certificat de chiffrement est installé sur tous les nœuds et que `NetworkService` dispose d’une autorisation de lecture sur la clé privée du certificat.
  ```json
    "fabricSettings": 
    [
        ...
    {
        "name":  "CentralSecretService",
        "parameters":  [
                {
                    "name":  "IsEnabled",
                    "value":  "true"
                },
                {
                    "name":  "MinReplicaSetSize",
                    "value":  "1"
                },
                {
                    "name":  "TargetReplicaSetSize",
                    "value":  "3"
                },
                 {
                    "name" : "EncryptionCertificateThumbprint",
                    "value": "<thumbprint>"
                 }
                ,
                ],
            },
            ]
     }
        ...
     ]
```
## <a name="declare-a-secret-resource"></a>Déclarer une ressource de secret
Vous pouvez créer une ressource de secret à l’aide de l’API REST.
  > [!NOTE] 
  > Si le cluster utilise l’Authentification Windows, la requête REST est envoyée via un canal HTTP non sécurisé. Il est recommandé d’utiliser un cluster basé sur x509 avec des points de terminaison sécurisés.

Pour créer une ressource de secret `supersecret` à l’aide de l’API REST, faites une demande PUT à `https://<clusterfqdn>:19080/Resources/Secrets/supersecret?api-version=6.4-preview`. Vous avez besoin du certificat du cluster ou du certificat client d’administration pour créer une ressource de secret.

```powershell
$json = '{"properties": {"kind": "inlinedValue", "contentType": "text/plain", "description": "supersecret"}}'
Invoke-WebRequest  -Uri https://<clusterfqdn>:19080/Resources/Secrets/supersecret?api-version=6.4-preview -Method PUT -CertificateThumbprint <CertThumbprint> -Body $json
```

## <a name="set-the-secret-value"></a>Définir la valeur du secret

Utilisez le script suivant pour utiliser l’API REST afin de définir la valeur du secret.
```powershell
$Params = '{"properties": {"value": "mysecretpassword"}}'
Invoke-WebRequest -Uri https://<clusterfqdn>:19080/Resources/Secrets/supersecret/values/ver1?api-version=6.4-preview -Method PUT -Body $Params -CertificateThumbprint <ClusterCertThumbprint>
```
### <a name="examine-the-secret-value"></a>Examiner la valeur du secret
```powershell
Invoke-WebRequest -CertificateThumbprint <ClusterCertThumbprint> -Method POST -Uri "https:<clusterfqdn>/Resources/Secrets/supersecret/values/ver1/list_value?api-version=6.4-preview"
```
## <a name="use-the-secret-in-your-application"></a>Utiliser le secret dans votre application

Effectuez les étapes suivantes pour utiliser le secret dans votre application Service Fabric.

1. Ajoutez une section dans le fichier **settings.xml** avec l’extrait de code suivant. Notez ici que la valeur est au format {`secretname:version`}.

   ```xml
     <Section Name="testsecrets">
      <Parameter Name="TopSecret" Type="SecretsStoreRef" Value="supersecret:ver1"/
     </Section>
   ```

1. Importez la section dans **ApplicationManifest.xml**.
   ```xml
     <ServiceManifestImport>
       <ServiceManifestRef ServiceManifestName="testservicePkg" ServiceManifestVersion="1.0.0" />
       <ConfigOverrides />
       <Policies>
         <ConfigPackagePolicies CodePackageRef="Code">
           <ConfigPackage Name="Config" SectionName="testsecrets" EnvironmentVariableName="SecretPath" />
           </ConfigPackagePolicies>
       </Policies>
     </ServiceManifestImport>
   ```

   La variable d’environnement `SecretPath` pointe vers le répertoire où sont stockés tous les secrets. Chaque paramètre listé sous la section `testsecrets` est stocké dans un fichier séparé. L’application peut maintenant utiliser le secret comme ceci :
   ```C#
   secretValue = IO.ReadFile(Path.Join(Environment.GetEnvironmentVariable("SecretPath"),  "TopSecret"))
   ```
1. Montez les secrets dans un conteneur. La seule modification nécessaire pour que les secrets soient disponibles à l’intérieur du conteneur consiste à `specify` un point de montage dans `<ConfigPackage>`.
L’extrait de code suivant correspond au fichier **ApplicationManifest.xml** modifié.  

   ```xml
   <ServiceManifestImport>
       <ServiceManifestRef ServiceManifestName="testservicePkg" ServiceManifestVersion="1.0.0" />
       <ConfigOverrides />
       <Policies>
         <ConfigPackagePolicies CodePackageRef="Code">
           <ConfigPackage Name="Config" SectionName="testsecrets" MountPoint="C:\secrets" EnvironmentVariableName="SecretPath" />
           <!-- Linux Container
            <ConfigPackage Name="Config" SectionName="testsecrets" MountPoint="/mnt/secrets" EnvironmentVariableName="SecretPath" />
           -->
         </ConfigPackagePolicies>
       </Policies>
     </ServiceManifestImport>
   ```
   Les secrets sont disponibles sous le point de montage à l’intérieur de votre conteneur.

1. Vous pouvez lier un secret à une variable d’environnement de processus en spécifiant `Type='SecretsStoreRef`. L’extrait de code suivant est un exemple de liaison de la version de `supersecret` `ver1` à la variable d’environnement `MySuperSecret` dans **ServiceManifest.xml**.

   ```xml
   <EnvironmentVariables>
     <EnvironmentVariable Name="MySuperSecret" Type="SecretsStoreRef" Value="supersecret:ver1"/>
   </EnvironmentVariables>
   ```

## <a name="next-steps"></a>Étapes suivantes
Découvrez-en plus sur la [sécurité des applications et des services](service-fabric-application-and-service-security.md).
