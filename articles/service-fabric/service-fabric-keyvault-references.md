---
title: Azure Service Fabric – Utilisation des références KeyVault de l'application Service Fabric
description: Cet article explique comment utiliser la prise en charge de Service Fabric KeyVaultReference pour des secrets d’application.
ms.topic: article
ms.date: 09/20/2019
ms.openlocfilehash: f2221bb3e8e3ee3181b2cff70107dccc203954cf
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92313790"
---
# <a name="keyvaultreference-support-for-service-fabric-applications-preview"></a>Prise en charge de KeyVaultReference pour les applications Service Fabric (préversion)

Un défi courant lors de la création d’applications Cloud est de savoir comment stocker en toute sécurité les secrets requis par votre application. Par exemple, vous souhaiterez peut-être stocker les informations d’identification du référentiel de conteneurs dans le coffre de clés et les référencer dans le manifeste de l’application. Service Fabric KeyVaultReference utilise les identités managées Service Fabric et facilite la référence aux secrets du coffre de clés. Le reste de cet article explique comment utiliser Service Fabric KeyVaultReference et inclut un cas d’utilisation classique.

> [!IMPORTANT]
> L’utilisation de cette fonctionnalité en préversion n’est pas recommandée dans des environnements de production.

> [!NOTE]
> La fonctionnalité en préversion de référence KeyVault ne prend en charge que les secrets dont la [version est gérée](../key-vault/general/about-keys-secrets-certificates.md#objects-identifiers-and-versioning). Les secrets sans version ne sont pas pris en charge.

## <a name="prerequisites"></a>Prérequis

- Identité managée pour application (MIT)
    
    La prise en charge de Service Fabric KeyVaultReference utilise l’identité managée de l’application. Par conséquent, les applications qui envisagent d’utiliser KeyVaultReferences doivent utiliser l’identité managée. Suivez cet [document](concepts-managed-identity.md) pour activer l’identité managée pour votre application.

- Banque de secrets centrale (CSS, Central Secrets Store).

    Central Secrets Store (CSS) est le cache de secrets local chiffré de Service Fabric. CSS est un magasin de secrets local qui garde chiffrées et en mémoire les données sensibles que sont notamment les mots de passe, les jetons et les clés. Les données KeyVaultReference, une fois récupérées, sont mises en cache dans CSS.

    Ajoutez le code ci-dessous à la configuration de votre cluster sous `fabricSettings` pour activer toutes les fonctionnalités requises pour la prise en charge de KeyVaultReference.

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
                    "value":  "3"
                },
                {
                    "name":  "TargetReplicaSetSize",
                    "value":  "3"
                }
                ]
            },
            {
                "name":  "ManagedIdentityTokenService",
                "parameters":  [
                {
                    "name":  "IsEnabled",
                    "value":  "true"
                }
                ]
            }
            ]
    ```

    > [!NOTE] 
    > Il est recommandé d’utiliser un certificat de chiffrement distinct pour CSS. Vous pouvez l’ajouter dans la section « CentralSecretService ».
    

    ```json
        {
            "name": "EncryptionCertificateThumbprint",
            "value": "<EncryptionCertificateThumbprint for CSS>"
        }
    ```
Pour que les modifications prennent effet, vous devez également modifier la stratégie de mise à niveau pour spécifier un redémarrage forcé du runtime Service Fabric sur chaque nœud à mesure que la mise à niveau progresse au sein du cluster. Ce redémarrage garantit que le service système qui vient d’être activé est démarré et en cours d’exécution sur chaque nœud. Dans l’extrait de code ci-dessous, forceRestart est le paramètre essentiel. Utilisez vos valeurs existantes pour les autres paramètres.
```json
"upgradeDescription": {
    "forceRestart": true,
    "healthCheckRetryTimeout": "00:45:00",
    "healthCheckStableDuration": "00:05:00",
    "healthCheckWaitDuration": "00:05:00",
    "upgradeDomainTimeout": "02:00:00",
    "upgradeReplicaSetCheckTimeout": "1.00:00:00",
    "upgradeTimeout": "12:00:00"
}
```
- Accordez l’autorisation d’accès avec identité managée de l’application au coffre de clés

    Pour voir comment accorder un accès avec identité managée au coffre de clés, reportez-vous à ce [document](how-to-grant-access-other-resources.md). Notez également que, si vous utilisez une identité managée affectée par le système, l’identité managée est créée uniquement après le déploiement de l’application.

## <a name="keyvault-secret-as-application-parameter"></a>Secret du coffre de clés en tant que paramètre d’application
Par exemple, si l’application a besoin de lire le mot de passe de la base de données principale stocké dans le coffre de clés, la prise en charge de Service Fabric KeyVaultReference facilite la tâche. L’exemple ci-dessous lit le secret `DBPassword` à partir du coffre de clés en utilisant la prise en charge de Service Fabric KeyVaultReference.

- Ajoutez une section à settings.xml

    Définissez le paramètre `DBPassword` avec le type `KeyVaultReference` et la valeur `<KeyVaultURL>`

    ```xml
    <Section Name="dbsecrets">
        <Parameter Name="DBPassword" Type="KeyVaultReference" Value="https://vault200.vault.azure.net/secrets/dbpassword/8ec042bbe0ea4356b9b171588a8a1f32"/>
    </Section>
    ```
- Référencez la nouvelle section du fichier ApplicationManifest.xml dans `<ConfigPackagePolicies>`

    ```xml
    <ServiceManifestImport>
        <Policies>
        <IdentityBindingPolicy ServiceIdentityRef="WebAdmin" ApplicationIdentityRef="ttkappuser" />
        <ConfigPackagePolicies CodePackageRef="Code">
            <!--Linux container example-->
            <ConfigPackage Name="Config" SectionName="dbsecrets" EnvironmentVariableName="SecretPath" MountPoint="/var/secrets"/>
            <!--Windows container example-->
            <!-- <ConfigPackage Name="Config" SectionName="dbsecrets" EnvironmentVariableName="SecretPath" MountPoint="C:\secrets"/> -->
        </ConfigPackagePolicies>
        </Policies>
    </ServiceManifestImport>
    ```

- Utilisation de KeyVaultReference dans votre application

    À l’instanciation du service, Service Fabric résout le paramètre KeyVaultReference à l’aide de l’identité managée de l’application. Chaque paramètre répertorié sous `<Section  Name=dbsecrets>` sera un fichier dans le dossier sur lequel pointe EnvironmentVariable SecretPath. L’extrait de code C# ci-dessous montre comment lire DBPassword dans votre application.

    ```C#
    string secretPath = Environment.GetEnvironmentVariable("SecretPath");
    using (StreamReader sr = new StreamReader(Path.Combine(secretPath, "DBPassword"))) 
    {
        string dbPassword =  sr.ReadToEnd();
        // dbPassword to connect to DB
    }
    ```
    > [!NOTE] 
    > Pour le scénario de conteneur, vous pouvez utiliser le MountPoint pour contrôler l’emplacement où les `secrets` seront montés.

## <a name="keyvault-secret-as-environment-variable"></a>Secret de coffre de clés en tant que variable d’environnement

Désormais, les variables d’environnement Service Fabric prennent en charge le type KeyVaultReference. L’exemple ci-dessous montre comment lier une variable d’environnement à un secret stocké dans le coffre de clés.

```xml
<EnvironmentVariables>
      <EnvironmentVariable Name="EventStorePassword" Type="KeyVaultReference" Value="https://ttkvault.vault.azure.net/secrets/clustercert/e225bd97e203430d809740b47736b9b8"/>
</EnvironmentVariables>
```

```C#
string eventStorePassword =  Environment.GetEnvironmentVariable("EventStorePassword");
```
## <a name="keyvault-secret-as-container-repository-password"></a>Secret du coffre de clés en tant que mot de passe de référentiel du conteneur
KeyVaultReference est un type pris en charge pour le conteneur RepositoryCredentials. L’exemple ci-dessous montre comment utiliser une référence de coffre de clé en tant que mot de passe de référentiel du conteneur.
```xml
 <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="user1" Type="KeyVaultReference" Password="https://ttkvault.vault.azure.net/secrets/containerpwd/e225bd97e203430d809740b47736b9b8"/>
      </ContainerHostPolicies>
```
## <a name="faq"></a>Questions fréquentes (FAQ)
- L’identité managée doit être activée pour la prise en charge de KeyVaultReference. L’activation de votre application échoue si KeyVaultReference est utilisé sans activer l’identité managée.

- Si vous utilisez une identité attribuée par le système, elle n’est créée qu’après le déploiement de l’application, ce qui crée une dépendance circulaire. Une fois votre application déployée, vous pouvez accorder au système une autorisation d’accès avec identité au coffre de clé. Vous pouvez trouver l’identité attribuée par le système par son nom {cluster}/{nom d’application}/{nom de service}

- Le coffre de clé doit se trouver dans le même abonnement que votre cluster Service Fabric. 

## <a name="next-steps"></a>Étapes suivantes

* [Documentation sur le coffre de clés Azure](../key-vault/index.yml)