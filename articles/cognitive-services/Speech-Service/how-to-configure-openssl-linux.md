---
title: Guide pratique pour configurer OpenSSL pour Linux
titleSuffix: Azure Cognitive Services
description: Découvrez comment configurer OpenSSL pour Linux.
services: cognitive-services
author: jhakulin
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/16/2020
ms.author: jhakulin
ms.openlocfilehash: 42960c25c4124203b64646fdc5cbca833b246e21
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81683166"
---
# <a name="configure-openssl-for-linux"></a>Configurer OpenSSL pour Linux

Quand vous utilisez le kit SDK Speech dans une version antérieure à 1.9.0, [OpenSSL](https://www.openssl.org) est configuré de façon dynamique en fonction de la version du système hôte. Dans les versions ultérieures du kit SDK Speech, OpenSSL (version [1.1.1b](https://mta.openssl.org/pipermail/openssl-announce/2019-February/000147.html)) est lié de manière statique à la bibliothèque principale du kit SDK Speech.

Pour garantir la connectivité, vérifiez que les certificats OpenSSL ont été installés sur votre système. Exécutez une commande :
```bash
openssl version -d
```

Sur les systèmes basés sur Ubuntu/Debian, la sortie doit être la suivante :
```
OPENSSLDIR: "/usr/lib/ssl"
```

Vérifiez s'il existe un sous-répertoire `certs` sous OPENSSLDIR. Dans l'exemple ci-dessus, il s'agirait de `/usr/lib/ssl/certs`.

* S'il existe un sous-répertoire `/usr/lib/ssl/certs` et qu'il contient de nombreux fichiers de certificats individuels (avec l'extension `.crt` ou `.pem`), aucune action supplémentaire n'est nécessaire.

* Si OPENSSLDIR est différent de `/usr/lib/ssl` et/ou en présence d'un seul fichier de groupement de certificats plutôt que de plusieurs fichiers individuels, vous devez définir une variable d'environnement SSL appropriée pour indiquer l'emplacement des certificats.

## <a name="examples"></a>Exemples

- OPENSSLDIR est `/opt/ssl`. Il y a un sous-répertoire `certs` contenant de nombreux fichiers `.crt` ou `.pem`.
Définissez la variable d'environnement `SSL_CERT_DIR` pour qu'elle pointe sur `/opt/ssl/certs` avant d'exécuter un programme qui utilise le kit de développement logiciel (SDK) Speech. Par exemple :
```bash
export SSL_CERT_DIR=/opt/ssl/certs
```

- OPENSSLDIR a la valeur `/etc/pki/tls` (par exemple sur les systèmes RHEL/CentOS). Il y a un sous-répertoire `certs` contenant un fichier de bundle de certificats, par exemple `ca-bundle.crt`.
Définissez la variable d’environnement `SSL_CERT_FILE` pour qu’elle pointe vers ce fichier avant d’exécuter un programme qui utilise le SDK Speech. Par exemple :
```bash
export SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt
```
> [!NOTE]
> Il est également à noter que certaines distributions de Linux n’ont pas de variable d’environnement TMP ou TMPDIR définie. Le Kit de développement logiciel (SDK) Speech télécharge la liste de révocation de certificat (CRL) à chaque fois, au lieu de la mettre en cache sur le disque pour la réutiliser jusqu’à son expiration. Pour améliorer les performances de la connexion initiale, vous pouvez [créer une variable d’environnement nommée TMPDIR et la définir sur le chemin d’accès du répertoire temporaire que vous avez choisi](https://help.ubuntu.com/community/EnvironmentVariables).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [À propos du kit SDK Speech](speech-sdk.md)
