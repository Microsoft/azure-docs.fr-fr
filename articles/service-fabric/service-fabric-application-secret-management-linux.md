---
title: Configurer un certificat de chiffrement sur des clusters Linux
description: Découvrez comment configurer un certificat de chiffrement et chiffrer des secrets sur clusters Linux.
author: shsha
ms.topic: conceptual
ms.date: 01/04/2019
ms.author: shsha
ms.openlocfilehash: b8e0a19e3f654fc561e7c7e26c6a2da463e24d5f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78969033"
---
# <a name="set-up-an-encryption-certificate-and-encrypt-secrets-on-linux-clusters"></a>Configurer un certificat de chiffrement et chiffrer des secrets sur clusters Linux
Dans cet article, découvrez comment configurer un certificat de chiffrement et l’utiliser pour chiffrer des secrets sur clusters Linux. Pour les clusters Windows, voir [Configurer un certificat de chiffrement et chiffrer des secrets sur Windows][secret-management-windows-specific-link].

## <a name="obtain-a-data-encipherment-certificate"></a>Obtenir un certificat de chiffrement de données
Un certificat de chiffrement de données est utilisé exclusivement pour le chiffrement et le déchiffrement des [paramètres][parameters-link] du fichier Settings.xml d’un service et des [variables d’environnement][environment-variables-link] du fichier ServiceManifest.xml d’un service. Il n’est pas utilisé pour l’authentification ou la signature du texte chiffré. Le certificat doit répondre aux exigences suivantes :

* Le certificat doit contenir une clé privée.
* L’utilisation d’une clé de certificat doit inclure le chiffrement de données (10) et ne doit pas inclure l’authentification du serveur ou l’authentification du client.

  Par exemple, les commandes suivantes peuvent être utilisées pour générer le certificat nécessaire à l’aide d’OpenSSL :
  
  ```console
  user@linux:~$ openssl req -newkey rsa:2048 -nodes -keyout TestCert.prv -x509 -days 365 -out TestCert.pem
  user@linux:~$ cat TestCert.prv >> TestCert.pem
  ```

## <a name="install-the-certificate-in-your-cluster"></a>Installation du certificat dans votre cluster
Le certificat doit être installé sur chaque nœud du cluster sous `/var/lib/sfcerts`. Le compte d’utilisateur sous lequel le service s’exécute (sfuser par défaut) **doit avoir un accès en lecture** pour le certificat installé (c’est-à-dire, `/var/lib/sfcerts/TestCert.pem` pour l’exemple actuel).

## <a name="encrypt-secrets"></a>Chiffrer des secrets
L’extrait de code suivant permet de chiffrer un secret. Cet extrait de code chiffre uniquement la valeur ; elle ne signe **pas** le texte chiffré. **Vous devez utiliser** le certificat de chiffrement qui est installé dans votre cluster afin de produire le texte chiffré pour les valeurs secrètes.

```console
user@linux:$ echo "Hello World!" > plaintext.txt
user@linux:$ iconv -f ASCII -t UTF-16LE plaintext.txt | tr -d '\n' > plaintext_UTF-16.txt
user@linux:$ openssl smime -encrypt -in plaintext_UTF-16.txt -binary -outform der TestCert.pem | base64 > encrypted.txt
```
La chaîne encodée en base 64 qui en résulte dans le fichier encrypted.txt contient à la fois le texte chiffré secret et plus d’informations sur le certificat qui a été utilisé pour le chiffrement. Vous pouvez vérifier sa validité en la déchiffrant avec OpenSSL.
```console
user@linux:$ cat encrypted.txt | base64 -d | openssl smime -decrypt -inform der -inkey TestCert.prv
```

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [Spécifier des secrets chiffrés dans une application][secret-management-specify-encrypted-secrets-link].

<!-- Links -->
[parameters-link]:service-fabric-how-to-parameterize-configuration-files.md
[environment-variables-link]: service-fabric-how-to-specify-environment-variables.md
[secret-management-windows-specific-link]: service-fabric-application-secret-management-windows.md
[secret-management-specify-encrypted-secrets-link]: service-fabric-application-secret-management.md#specify-encrypted-secrets-in-an-application
