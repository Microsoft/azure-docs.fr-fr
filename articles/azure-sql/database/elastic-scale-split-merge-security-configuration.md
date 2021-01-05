---
title: Configuration de la sécurité du fractionnement et de la fusion
description: Configurez des certificats x409 pour le chiffrement à l’aide du service de fractionnement/fusion pour la mise à l’échelle élastique.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: VanMSFT
ms.author: vanto
ms.reviewer: sstein
ms.date: 12/18/2018
ms.openlocfilehash: 235efc550fd47d4244a5bf081c75d5e824a8e4b4
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793430"
---
# <a name="split-merge-security-configuration"></a>Configuration de la sécurité du fractionnement et de la fusion
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Pour utiliser le service de fusion et de fractionnement, vous devez configurer correctement la sécurité. Ce service fait partie de la fonctionnalité Mise à l'échelle élastique de Azure SQL Database. Pour plus d’informations, consultez le [didacticiel sur le service de fusion et de fractionnement de l’infrastructure élastique](elastic-scale-configure-deploy-split-and-merge.md)

## <a name="configuring-certificates"></a>Configuration des certificats

Les certificats sont configurés de deux manières. 

1. [Pour configurer le certificat TLS/SSL](#to-configure-the-tlsssl-certificate)
2. [Pour configurer des certificats clients](#to-configure-client-certificates) 

## <a name="to-obtain-certificates"></a>Pour obtenir des certificats

Les certificats peuvent être obtenus à partir d’autorités de certification publiques ou du [Service de certificats Windows](/windows/win32/seccrypto/certificate-services). Il s’agit des méthodes préférées pour obtenir des certificats.

Si ces options ne sont pas disponibles, vous pouvez générer des **certificats auto-signés** .

## <a name="tools-to-generate-certificates"></a>Outils de génération de certificats

* [makecert.exe](/previous-versions/dotnet/netframework-4.0/bfsktky3(v=vs.100))
* [pvk2pfx.exe](/windows-hardware/drivers/devtest/pvk2pfx)

### <a name="to-run-the-tools"></a>Pour exécuter les outils

* Depuis une invite de commandes développeur pour Visual Studio, consultez la rubrique [Invite de commandes Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs) 
  
    Si installée, accédez à :
  
    ```console
    %ProgramFiles(x86)%\Windows Kits\x.y\bin\x86 
    ```

* Procurez-vous le kit WDK à partir de [Windows 8.1 : Télécharger les kits et les outils](https://msdn.microsoft.com/windows/hardware/gg454513#drivers)

## <a name="to-configure-the-tlsssl-certificate"></a>Pour configurer le certificat TLS/SSL

Un certificat TLS/SSL est nécessaire pour chiffrer les communications et authentifier le serveur. Choisissez le plus approprié des trois scénarios ci-dessous et exécutez toutes les étapes associées :

### <a name="create-a-new-self-signed-certificate"></a>Création d’un certificat auto-signé

1. [Création d’un certificat auto-signé](#create-a-self-signed-certificate)
2. [Création d’un fichier PFX pour un certificat TLS/SSL auto-signé](#create-pfx-file-for-self-signed-tlsssl-certificate)
3. [Téléchargement du certificat TLS/SSL vers le service cloud](#upload-tlsssl-certificate-to-cloud-service)
4. [Mise à jour du certificat TLS/SSL dans le fichier de configuration de service](#update-tlsssl-certificate-in-service-configuration-file)
5. [Importation de l’autorité de certification TLS/SSL](#import-tlsssl-certification-authority)

### <a name="to-use-an-existing-certificate-from-the-certificate-store"></a>Pour utiliser un certificat existant du magasin de certificats
1. [Exportation d’un certificat TLS/SSL à partir du magasin de certificats](#export-tlsssl-certificate-from-certificate-store)
2. [Téléchargement du certificat TLS/SSL vers le service cloud](#upload-tlsssl-certificate-to-cloud-service)
3. [Mise à jour du certificat TLS/SSL dans le fichier de configuration de service](#update-tlsssl-certificate-in-service-configuration-file)

### <a name="to-use-an-existing-certificate-in-a-pfx-file"></a>Pour utiliser un certificat existant dans un fichier PFX
1. [Téléchargement du certificat TLS/SSL vers le service cloud](#upload-tlsssl-certificate-to-cloud-service)
2. [Mise à jour du certificat TLS/SSL dans le fichier de configuration de service](#update-tlsssl-certificate-in-service-configuration-file)

## <a name="to-configure-client-certificates"></a>Pour configurer des certificats clients
Les certificats clients sont requis pour authentifier les demandes au service. Choisissez le plus approprié des trois scénarios ci-dessous et exécutez toutes les étapes associées :

### <a name="turn-off-client-certificates"></a>Désactivation des certificats clients
1. [Désactivation de l’authentification par certificat client](#turn-off-client-certificate-based-authentication)

### <a name="issue-new-self-signed-client-certificates"></a>Émission de nouveaux certificats clients auto-signés
1. [Création d’une autorité de certification auto-signée](#create-a-self-signed-certification-authority)
2. [Téléchargement du certificat CA vers le service cloud](#upload-ca-certificate-to-cloud-service)
3. [Mise à jour du certificat CA dans le fichier de configuration de service](#update-ca-certificate-in-service-configuration-file)
4. [Émission de certificats clients](#issue-client-certificates)
5. [Création de fichiers PFX pour les certificats clients](#create-pfx-files-for-client-certificates)
6. [Importation d’un certificat client](#import-client-certificate)
7. [Copie des empreintes numériques des certificats clients](#copy-client-certificate-thumbprints)
8. [Configuration des clients autorisés dans le fichier de configuration de service](#configure-allowed-clients-in-the-service-configuration-file)

### <a name="use-existing-client-certificates"></a>Utilisation de certificats clients existants
1. [Find CA Public Key](#find-ca-public-key)
2. [Téléchargement du certificat CA vers le service cloud](#upload-ca-certificate-to-cloud-service)
3. [Mise à jour du certificat CA dans le fichier de configuration de service](#update-ca-certificate-in-service-configuration-file)
4. [Copie des empreintes numériques des certificats clients](#copy-client-certificate-thumbprints)
5. [Configuration des clients autorisés dans le fichier de configuration de service](#configure-allowed-clients-in-the-service-configuration-file)
6. [Configuration de la vérification de révocation des certificats clients](#configure-client-certificate-revocation-check)

## <a name="allowed-ip-addresses"></a>Adresses IP autorisées
L’accès aux points de terminaison de service peut être limité à des plages d’adresses IP spécifiques.

## <a name="to-configure-encryption-for-the-store"></a>Pour configurer le cryptage pour le magasin
Un certificat est nécessaire pour chiffrer les informations d’identification stockées dans le magasin de métadonnées. Choisissez le plus approprié des trois scénarios ci-dessous et exécutez toutes les étapes associées :

### <a name="use-a-new-self-signed-certificate"></a>Utilisation d’un nouveau certificat auto-signé
1. [Création d’un certificat auto-signé](#create-a-self-signed-certificate)
2. [Création d’un fichier PFX pour un certificat de chiffrement auto-signé](#create-pfx-file-for-self-signed-tlsssl-certificate)
3. [Téléchargement du certificat de chiffrement vers le service cloud](#upload-encryption-certificate-to-cloud-service)
4. [Mise à jour du certificat de chiffrement dans le fichier de configuration de service](#update-encryption-certificate-in-service-configuration-file)

### <a name="use-an-existing-certificate-from-the-certificate-store"></a>Utilisation d’un certificat existant du magasin de certificats
1. [Exportation d’un certificat de chiffrement à partir du magasin de certificats](#export-encryption-certificate-from-certificate-store)
2. [Téléchargement du certificat de chiffrement vers le service cloud](#upload-encryption-certificate-to-cloud-service)
3. [Mise à jour du certificat de chiffrement dans le fichier de configuration de service](#update-encryption-certificate-in-service-configuration-file)

### <a name="use-an-existing-certificate-in-a-pfx-file"></a>Utilisation d’un certificat existant dans un fichier PFX
1. [Téléchargement du certificat de chiffrement vers le service cloud](#upload-encryption-certificate-to-cloud-service)
2. [Mise à jour du certificat de chiffrement dans le fichier de configuration de service](#update-encryption-certificate-in-service-configuration-file)

## <a name="the-default-configuration"></a>Configuration par défaut
La configuration par défaut refuse tout accès au point de terminaison HTTP. Il s’agit du paramètre recommandé, puisque les demandes pour ces points de terminaison peuvent comporter des informations sensibles comme les informations d’identification de la base de données.
La configuration par défaut accepte tout accès au point de terminaison HTTPS. Ce paramètre peut être restreint davantage.

### <a name="changing-the-configuration"></a>Modification de la configuration
Le groupe de règles de contrôle d’accès qui s’appliquent à un point de terminaison est configuré dans la section **\<EndpointAcls>** du **fichier de configuration de service** .

```xml
<EndpointAcls>
    <EndpointAcl role="SplitMergeWeb" endPoint="HttpIn" accessControl="DenyAll" />
    <EndpointAcl role="SplitMergeWeb" endPoint="HttpsIn" accessControl="AllowAll" />
</EndpointAcls>
```

Les règles dans un groupe de contrôle d’accès sont configurées dans une section \<AccessControl name=""> du fichier de configuration du service. 

Le format est expliqué dans la documentation de listes de contrôle d’accès réseau.
Par exemple, pour autoriser uniquement les adresses IP de la plage 100.100.0.0 à 100.100.255.255 à accéder au point de terminaison HTTPS, les règles ressembleraient à ceci :

```xml
<AccessControl name="Retricted">
    <Rule action="permit" description="Some" order="1" remoteSubnet="100.100.0.0/16"/>
    <Rule action="deny" description="None" order="2" remoteSubnet="0.0.0.0/0" />
</AccessControl>
<EndpointAcls>
    <EndpointAcl role="SplitMergeWeb" endPoint="HttpsIn" accessControl="Restricted" />
</EndpointAcls>
```

## <a name="denial-of-service-prevention"></a>Prévention du déni de service
Il existe deux mécanismes différents pris en charge pour détecter et prévenir les attaques par déni de service :

* Limiter le nombre de demandes simultanées par hôte distant (désactivé par défaut)
* Limiter le taux d’accès par hôte distant (activé par défaut)

Ces mécanismes sont basés sur les fonctionnalités plus longuement documentées dans la rubrique Sécurité IP dynamique dans IIS. Lors de la modification de cette configuration, soyez attentif aux facteurs suivants :

* Le comportement des proxys et des appareils de traduction d’adresses réseau sur les informations de l’hôte distant
* Chaque demande à destination d’une ressource du rôle web est prise en compte (par exemple, le chargement de scripts, des images, etc.)

## <a name="restricting-number-of-concurrent-accesses"></a>Limitation du nombre d'accès simultanés
Les paramètres qui configurent ce comportement sont les suivants :

```xml
<Setting name="DynamicIpRestrictionDenyByConcurrentRequests" value="false" />
<Setting name="DynamicIpRestrictionMaxConcurrentRequests" value="20" />
```

Modifiez DynamicIpRestrictionDenyByConcurrentRequests sur true pour activer cette protection.

## <a name="restricting-rate-of-access"></a>Restriction du taux d’accès
Les paramètres qui configurent ce comportement sont les suivants :

```xml
<Setting name="DynamicIpRestrictionDenyByRequestRate" value="true" />
<Setting name="DynamicIpRestrictionMaxRequests" value="100" />
<Setting name="DynamicIpRestrictionRequestIntervalInMilliseconds" value="2000" />
```

## <a name="configuring-the-response-to-a-denied-request"></a>Configuration de la réponse à une demande refusée
Le paramètre suivant configure la réponse à une demande refusée :

```xml
<Setting name="DynamicIpRestrictionDenyAction" value="AbortRequest" />
```

Reportez-vous à la documentation relative à la sécurité IP dynamique dans IIS pour les autres valeurs prises en charge.

## <a name="operations-for-configuring-service-certificates"></a>Opérations de configuration des certificats de service
Cette rubrique sert de référence uniquement. Suivez les étapes de configuration décrites dans :

* Configurer le certificat TLS/SSL
* Configuration des certificats clients

## <a name="create-a-self-signed-certificate"></a>Créer un certificat auto-signé
Exécutez :

```console
makecert ^
  -n "CN=myservice.cloudapp.net" ^
  -e MM/DD/YYYY ^
  -r -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.1" ^
  -a sha256 -len 2048 ^
  -sv MySSL.pvk MySSL.cer
```

Pour personnaliser :

* -n avec l’URL du service. Les caractères génériques (CN=*.cloudapp.net) et d’autres noms (CN=myservice1.cloudapp.net, CN=myservice2.cloudapp.net) sont pris en charge.
* -e avec la date d'expiration du certificat Créez un mot de passe fort et spécifiez-le lorsque vous y êtes invité.

## <a name="create-pfx-file-for-self-signed-tlsssl-certificate"></a>Créer fichier PFX pour un certificat TLS/SSL auto-signé
Exécutez :

```console
pvk2pfx -pvk MySSL.pvk -spc MySSL.cer
```

Entrez le mot de passe et exportez le certificat avec les options suivantes :

* Oui, exporter la clé privée
* Exporter toutes les propriétés étendues

## <a name="export-tlsssl-certificate-from-certificate-store"></a>Exporter un certificat TLS/SSL à partir du magasin de certificats
* Recherchez le certificat
* Cliquez sur Actions -> Toutes les tâches -> Exporter...
* Exportez le certificat dans un fichier .PFX avec les options suivantes :
  * Oui, exporter la clé privée
  * Inclure tous les certificats dans le chemin d’accès de certification si possible *Exporter toutes les propriétés étendues

## <a name="upload-tlsssl-certificate-to-cloud-service"></a>Télécharger le certificat TLS/SSL vers le service cloud
Téléchargez le certificat avec le fichier .PFX existant ou généré avec la paire de clés TLS :

* Entrez le mot de passe protégeant les informations de clés privées

## <a name="update-tlsssl-certificate-in-service-configuration-file"></a>Mettre à jour certificat TLS/SSL dans le fichier de configuration de service
Mettez à jour la valeur de l’empreinte numérique du paramètre suivant du fichier de configuration de service avec l’empreinte numérique du certificat téléchargé vers le service cloud :

```console
<Certificate name="SSL" thumbprint="" thumbprintAlgorithm="sha1" />
```

## <a name="import-tlsssl-certification-authority"></a>Importer l’autorité de certification TLS/SSL
Suivez les étapes suivantes pour tous les comptes/ordinateurs qui communiquent avec le service :

* Double-cliquez sur le fichier .CER dans l'Explorateur Windows
* Dans la boîte de dialogue Certificat, cliquez sur Installer le certificat...
* Importez le certificat dans le magasin racine des autorités de certification approuvées

## <a name="turn-off-client-certificate-based-authentication"></a>Désactivation de l’authentification par certificat client
Seule l’authentification par certificat client est prise en charge et sa désactivation autorise l’accès public aux points de terminaison de service, à moins que d’autres mécanismes soient en place (par exemple, Réseau virtuel Microsoft Azure).

Définissez ces paramètres sur false dans le fichier de configuration de service pour désactiver la fonctionnalité :

```xml
<Setting name="SetupWebAppForClientCertificates" value="false" />
<Setting name="SetupWebserverForClientCertificates" value="false" />
```

Ensuite, copiez la même empreinte numérique que celle du certificat TLS/SSL dans le paramètre du certificat de l’autorité de certification :

```xml
<Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />
```

## <a name="create-a-self-signed-certification-authority"></a>Création d’une autorité de certification auto-signée
Exécutez les étapes suivantes pour créer un certificat auto-signé qui agit comme une autorité de certification :

```console
makecert ^
-n "CN=MyCA" ^
-e MM/DD/YYYY ^
 -r -cy authority -h 1 ^
 -a sha256 -len 2048 ^
  -sr localmachine -ss my ^
  MyCA.cer
```

Pour le personnaliser

* -e avec la date d’expiration du certificat

## <a name="find-ca-public-key"></a>Recherche de la clé publique de l’autorité de certification
Tous les certificats clients doivent avoir été émis par une autorité de certification approuvée par le service. Recherchez la clé publique d’accès à l’autorité de certification qui a émis les certificats clients qui seront utilisés pour l’authentification afin de la télécharger vers le service cloud.

Si le fichier comportant la clé publique n’est pas disponible, exportez-le à partir du magasin de certificats :

* Recherchez le certificat
  * Recherchez un certificat client émis par la même autorité de certification
* Double-cliquez sur le certificat.
* Sélectionnez l’onglet Chemin d’accès de certification dans la boîte de dialogue Certificat.
* Double-cliquez sur l’entrée de l’autorité de certification dans le chemin d’accès.
* Prenez note des propriétés du certificat.
* Fermez la boîte de dialogue **Certificat** .
* Recherchez le certificat
  * Recherchez l’autorité de certification indiquée ci-dessus.
* Cliquez sur Actions -> Toutes les tâches -> Exporter...
* Exportez le certificat dans un fichier .CER avec les options suivantes :
  * **Non, ne pas exporter la clé privée**
  * Inclure tous les certificats dans le chemin d’accès de certification si possible.
  * Exporter toutes les propriétés étendues.

## <a name="upload-ca-certificate-to-cloud-service"></a>Téléchargement du certificat CA vers le service cloud
Téléchargez le certificat avec le fichier .CER existant ou généré avec la clé publique de l’autorité de certification.

## <a name="update-ca-certificate-in-service-configuration-file"></a>Mise à jour du certificat CA dans le fichier de configuration de service
Mettez à jour la valeur de l’empreinte numérique du paramètre suivant du fichier de configuration de service avec l’empreinte numérique du certificat téléchargé vers le service cloud :

```xml
<Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />
```

Mettez à jour la valeur du paramètre suivant avec la même empreinte numérique :

```xml
<Setting name="AdditionalTrustedRootCertificationAuthorities" value="" />
```

## <a name="issue-client-certificates"></a>Émission de certificats clients
Chaque personne autorisée à accéder au service doit disposer d’un certificat client émis pour son utilisation exclusive et doit choisir un mot de passe fort pour protéger sa clé privée. 

Les étapes suivantes doivent être exécutées sur l’ordinateur sur lequel le certificat auto-signé de l’autorité de certification a été généré et stocké :

```console
makecert ^
  -n "CN=My ID" ^
  -e MM/DD/YYYY ^
  -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.2" ^
  -a sha256 -len 2048 ^
  -in "MyCA" -ir localmachine -is my ^
  -sv MyID.pvk MyID.cer
```

Personnalisation :

* -n avec un identifiant client qui sera authentifié avec ce certificat
* -e avec la date d'expiration du certificat
* MyID.pvk et MyID.cer avec des noms de fichier uniques pour ce certificat client

Cette commande vous demande de créer un mot de passe et de l’utiliser une seule fois. Utilisez un mot de passe fort.

## <a name="create-pfx-files-for-client-certificates"></a>Création de fichiers PFX pour les certificats clients
Pour chaque certificat client généré, exécutez :

```console
pvk2pfx -pvk MyID.pvk -spc MyID.cer
```

Personnalisation :

```console
MyID.pvk and MyID.cer with the filename for the client certificate
```

Entrez le mot de passe et exportez le certificat avec les options suivantes :

* Oui, exporter la clé privée
* Exporter toutes les propriétés étendues
* La personne pour laquelle ce certificat a été émis doit choisir le mot de passe d’exportation

## <a name="import-client-certificate"></a>Importation d’un certificat client
Chaque personne pour laquelle un certificat client a été émis doit importer la paire de clés dans les machines qu’elle utilise pour communiquer avec le service :

* Double-cliquez sur le fichier .PFX dans l'Explorateur Windows
* Importez un certificat dans le magasin Personnel avec au moins l'option suivante :
  * Inclure toutes les propriétés étendues activées

## <a name="copy-client-certificate-thumbprints"></a>Copie des empreintes numériques des certificats clients
Chaque personne pour laquelle un certificat client a été émis doit suivre ces étapes pour obtenir l’empreinte numérique de son certificat, qui sera ajoutée au fichier de configuration de service :

* Exécutez certmgr.exe
* Sélectionnez l'onglet Personnel
* Double-cliquez sur le certificat client à utiliser pour l'authentification
* Dans la boîte de dialogue Certificat qui s'ouvre, sélectionnez l'onglet Détails
* Veillez à ce que l'option Afficher indique Tous
* Sélectionnez le champ nommé Empreinte numérique dans la liste
* Copiez la valeur de l'empreinte
  * Supprimez les caractères Unicode non visibles devant le premier chiffre
  * Supprimez tous les espaces

## <a name="configure-allowed-clients-in-the-service-configuration-file"></a>Configuration des clients autorisés dans le fichier de configuration de service
Mettez à jour la valeur du paramètre suivant dans le fichier de configuration de service avec une liste séparée par des virgules des empreintes numériques des certificats clients autorisés à accéder au service :

```xml
<Setting name="AllowedClientCertificateThumbprints" value="" />
```

## <a name="configure-client-certificate-revocation-check"></a>Configuration de la vérification de révocation des certificats clients
Le paramètre par défaut ne vérifie pas l’état de révocation du certificat client auprès de l’autorité de certification. Pour activer les vérifications, si l’autorité de certification qui a émis les certificats clients prend en charge ces vérifications, définissez le paramètre suivant avec l’une des valeurs définies dans l’énumération X509RevocationMode :

```xml
<Setting name="ClientCertificateRevocationCheck" value="NoCheck" />
```

## <a name="create-pfx-file-for-self-signed-encryption-certificates"></a>Création d’un fichier PFX pour un certificat de chiffrement auto-signé
Pour un certificat de chiffrement, exécutez :

```console
pvk2pfx -pvk MyID.pvk -spc MyID.cer
```

Personnalisation :

```console
MyID.pvk and MyID.cer with the filename for the encryption certificate
```

Entrez le mot de passe et exportez le certificat avec les options suivantes :

* Oui, exporter la clé privée
* Exporter toutes les propriétés étendues
* Vous aurez besoin du mot de passe lors du téléchargement du certificat vers le service cloud.

## <a name="export-encryption-certificate-from-certificate-store"></a>Exportation d’un certificat de chiffrement à partir du magasin de certificats
* Recherchez le certificat
* Cliquez sur Actions -> Toutes les tâches -> Exporter...
* Exportez le certificat dans un fichier .PFX avec les options suivantes : 
  * Oui, exporter la clé privée
  * Inclure tous les certificats dans le chemin d’accès de certification si possible 
* Exporter toutes les propriétés étendues

## <a name="upload-encryption-certificate-to-cloud-service"></a>Téléchargement du certificat de chiffrement vers le service cloud
Téléchargez le certificat avec le fichier .PFX existant ou généré avec la paire de clés de chiffrement :

* Entrez le mot de passe protégeant les informations de clés privées

## <a name="update-encryption-certificate-in-service-configuration-file"></a>Mise à jour du certificat de chiffrement dans le fichier de configuration de service
Mettez à jour la valeur de l’empreinte numérique des paramètres suivants du fichier de configuration de service avec l’empreinte numérique du certificat téléchargé vers le service cloud :

```xml
<Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />
```

## <a name="common-certificate-operations"></a>Opérations courantes de certificat
* Configurer le certificat TLS/SSL
* Configuration des certificats clients

## <a name="find-certificate"></a>Recherchez le certificat
Procédez comme suit :

1. Exécutez mmc.exe.
2. Fichier -> Ajouter/supprimer un composant logiciel enfichable...
3. Sélectionnez **Certificats** .
4. Cliquez sur **Add** .
5. Choisissez l’emplacement du magasin de certificats.
6. Cliquez sur **Terminer** .
7. Cliquez sur **OK** .
8. Développez les **certificats** .
9. Développez le nœud du magasin du certificat.
10. Développez le nœud enfant du certificat.
11. Sélectionnez un certificat dans la liste.

## <a name="export-certificate"></a>Exportation du certificat
Dans l’ **Assistant Exportation de certificat** :

1. Cliquez sur **Suivant** .
2. Sélectionnez l’option **Oui** , puis **Exporter la clé privée** .
3. Cliquez sur **Suivant** .
4. Sélectionnez le format de fichier de sortie souhaité.
5. Vérifiez les options de votre choix.
6. Vérifiez le **mot de passe** .
7. Entrez un mot de passe fort et confirmez-le.
8. Cliquez sur **Suivant** .
9. Tapez ou sélectionnez un nom de fichier dans lequel stocker le certificat (utilisez une extension .PFX).
10. Cliquez sur **Suivant** .
11. Cliquez sur **Terminer** .
12. Cliquez sur **OK** .

## <a name="import-certificate"></a>Importation d’un certificat
Dans l'Assistant Importation de certificat :

1. Sélectionnez l’emplacement du magasin.
   
   * Sélectionnez **Utilisateur actuel** si seuls les processus s’exécutant sous l’utilisateur actuel accèdent au service.
   * Sélectionnez **Ordinateur local** si d’autres processus de cet ordinateur accèdent au service
2. Cliquez sur **Suivant** .
3. Si vous importez depuis un fichier, vérifiez le chemin d’accès.
4. Si vous importez depuis un fichier .PFX :
   1. Entrez le mot de passe protégeant la clé privée
   2. Sélectionnez les options d’importation
5. Sélectionnez « Placer » les certificats dans le magasin suivant
6. Cliquez sur **Parcourir** .
7. Sélectionnez le magasin de votre choix.
8. Cliquez sur **Terminer** .
   
   * Si le magasin racine des autorités de certification approuvées a été choisi, cliquez sur **Oui** .
9. Cliquez sur **OK** dans toutes les fenêtres des boîtes de dialogue.

## <a name="upload-certificate"></a>Téléchargement d’un certificat
Dans le [portail Azure](https://portal.azure.com/) :

1. Sélectionnez **Services Cloud** .
2. Sélectionnez le service cloud.
3. Dans le menu supérieur, cliquez sur **Certificats** .
4. Dans la barre inférieure, cliquez sur **Télécharger** .
5. Sélectionnez le fichier de certificat.
6. S’il s’agit d’un fichier .PFX, entrez le mot de passe de la clé privée.
7. Lorsque vous avez terminé, copiez l’empreinte de certificat à partir de la nouvelle entrée dans la liste.

## <a name="other-security-considerations"></a>Autres considérations liées à la sécurité
Les paramètres TLS décrits dans ce document chiffrent les communications entre le service et ses clients lorsque le point de terminaison HTTPS est utilisé. Ceci est important car les informations d’identification pour l’accès à la base de données, et éventuellement à d’autres informations sensibles, sont contenues dans les communications. Notez, néanmoins, que le service conserve l'état interne, y compris les informations d'identification, dans ses tables internes de la base de données dans Azure SQL Database que vous avez fournies pour le stockage des métadonnées dans votre abonnement Microsoft Azure. Cette base de données a été définie dans le paramètre suivant dans votre fichier de configuration de service (fichier .CSCFG) : 

```xml
<Setting name="ElasticScaleMetadata" value="Server=…" />
```

Les informations d’identification stockées dans cette base de données sont chiffrées. Toutefois, il est recommandé de s’assurer que les rôles Web et de travail de vos déploiements de service sont mis à jour et sécurisés, car les deux types de rôle ont accès à la base de données de métadonnées et au certificat utilisé pour le chiffrement et le déchiffrement des informations d’identification stockées. 

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]