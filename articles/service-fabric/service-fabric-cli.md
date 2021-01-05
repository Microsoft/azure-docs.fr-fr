---
title: Prise en main de l’interface de ligne de commande Azure Service Fabric
description: Découvrez comment utiliser l’interface de ligne de commande Azure Service Fabric. Découvrez comment se connecter à un cluster et gérer des applications.
author: jeffj6123
ms.topic: conceptual
ms.date: 5/19/2020
ms.author: jejarry
ms.openlocfilehash: b3714f8401def9bed68e4b0845d025734a480cb3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83681803"
---
# <a name="azure-service-fabric-cli"></a>Interface de ligne de commande Azure Service Fabric

L’interface de ligne de commande Azure Service Fabric est un utilitaire de ligne de commande pour interagir avec des entités Service Fabric, et les gérer. L’interface de ligne de commande Service Fabric peut être utilisée avec des clusters Windows ou Linux. L’interface de ligne de commande Service Fabric s’exécute sur toute plateforme prenant en charge Python.

[!INCLUDE [links to azure cli and service fabric cli](../../includes/service-fabric-sfctl.md)]

## <a name="prerequisites"></a>Prérequis

Avant l’installation, vérifiez que Python et pip sont installés dans votre environnement. Pour plus d’informations, consultez la [documentation de démarrage rapide de pip](https://pip.pypa.io/en/latest/quickstart/) et la [documentation d’installation officielle de Python](https://wiki.python.org/moin/BeginnersGuide/Download).

L’interface CLI prend en charge Python version 2.7, 3.5, 3.6 et 3.7. La version 3.x est recommandée, car le support de Python 2.7 sera bientôt arrêté.

### <a name="service-fabric-target-runtime"></a>Runtime Service Fabric cible

L’interface CLI Service Fabric a pour objectif de prendre en charge la dernière version du runtime du Kit de développement logiciel (SDK) Service Fabric. Utilisez le tableau suivant pour déterminer la version de CLI que vous devez installer :

| Version de la CLI   | Version du runtime prise en charge |
|---------------|---------------------------|
| Plus récente (~=10) | Plus récente (~=7,1)            |
| 9.0.0         | 7.1                       |
| 8.0.0         | 6.5                       |
| 7.1.0         | 6.4                       |
| 6.0.0         | 6.3                       |
| 5.0.0         | 6.2                       |
| 4.0.0         | 6.1                       |
| 3.0.0         | 6.0                       |
| 1.1.0         | 5.6, 5.7                  |

Vous pouvez éventuellement spécifier une version cible de l’interface CLI à installer ajoutant à la commande `pip install` le suffixe `==<version>`. Par exemple, pour la version 1.1.0, la syntaxe serait la suivante :

```shell
pip install -I sfctl==1.1.0
```

Remplacez la commande `pip install` par la commande mentionnée précédemment, si nécessaire.

Pour plus d’informations sur les versions de l’interface de ligne de commande de Service Fabric, consultez la [documentation GitHub](https://github.com/Azure/service-fabric-cli/releases).

## <a name="install-pip-python-and-the-service-fabric-cli"></a>Installer pip, Python et l’interface de ligne de commande Service Fabric

Il existe différentes façons d’installer pip et Python sur votre plateforme. Voici quelques étapes permettant de configurer rapidement les principaux systèmes d’exploitation avec Python 3 et pip.

### <a name="windows"></a>Windows

Pour Windows 10, Windows Server 2016 et Windows Server 2012 R2, utilisez les instructions d’installation officielles standard. Par défaut, le programme d’installation de Python installe aussi pip.

1. Accédez à la [page officielle des téléchargements de Python](https://www.python.org/downloads/) pour télécharger la dernière version de Python 3.7.

2. Démarrez le programme d’installation.

3. Au bas de l’invite, sélectionnez **Ajouter Python 3.7 à PATH**.

4. Sélectionnez **Installer maintenant**et terminez l’installation.

Vous pouvez maintenant ouvrir une nouvelle fenêtre de commande et obtenir la version de Python et celle de pip.

```shell
python --version
pip --version
```

Exécutez ensuite la commande suivante pour installer Azure Service Fabric CLI (sfctl) et afficher la page d’aide CLI :

```shell
pip install sfctl
sfctl -h
```

### <a name="ubuntu-and-windows-subsystem-for-linux"></a>Sous-systèmes Windows et Ubuntu pour Linux

Exécutez les commandes suivantes pour installer l’interface CLI Service Fabric :

```bash
sudo apt-get install python3
sudo apt-get install python3-pip
pip3 install sfctl
```

Vous pouvez ensuite tester l’installation avec la commande suivante :

```bash
sfctl -h
```

Si vous recevez une erreur de type « command not found » comme suit :

`sfctl: command not found`

Assurez-vous que le répertoire `~/.local/bin` est accessible depuis le chemin d’accès `$PATH` :

```bash
export PATH=$PATH:~/.local/bin
echo "export PATH=$PATH:~/.local/bin" >> .shellrc
```

Si l’installation sur le sous-système Windows de Linux échoue en signalant que les autorisations du dossier sont incorrectes, vous devrez peut-être réessayer, en utilisant des autorisations élevées :

```bash
sudo pip3 install sfctl
```

### <a name="red-hat-enterprise-linux-74-service-fabric-preview-support"></a>Red Hat Enterprise Linux 7.4 (prise en charge de la préversion de Service Fabric)

Exécutez les commandes suivantes pour installer l’interface CLI de Service Fabric sur Red Hat :

```bash
sudo yum install -y python34
sudo yum install python34-setuptools
sudo easy_install-3.4 pip
sudo pip3 install sfctl
```

Pour tester l’installation, vous pouvez utiliser les étapes indiquées dans la section **Sous-systèmes Windows et Ubuntu pour Linux**.

<a name = "cli-mac"></a>
### <a name="macos"></a>MacOS

Pour MacOS, nous vous recommandons d’utiliser le [Gestionnaire de package HomeBrew](https://brew.sh). Si HomeBrew n’est pas déjà installé, installez-le en exécutant la commande suivante :

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Ensuite, à partir du terminal, installez Python 3.7, pip et l’interface CLI de Service Fabric en exécutant les commandes suivantes :

```bash
brew install python3
pip3 install sfctl
sfctl -h
```

## <a name="cli-syntax"></a>Syntaxe d’Azure CLI

Les commandes portent toujours le préfixe `sfctl`. Pour obtenir des informations générales sur toutes les commandes dont vous pouvez vous servir, utilisez `sfctl -h`. Pour obtenir de l’aide avec une seule commande, utilisez `sfctl <command> -h`.

Les commandes suivent une structure répétable, où la cible de la commande précède le verbe ou l’action.

```shell
sfctl <object> <action>
```

Dans cet exemple, `<object>` est la cible de `<action>`.

## <a name="select-a-cluster"></a>Sélectionner un cluster

Avant d’effectuer toute opération, vous devez sélectionner un cluster auquel vous connecter. Vous pouvez par exemple exécuter la commande suivante pour sélectionner le cluster nommé `testcluster.com` et vous y connecter :

> [!WARNING]
> N’utilisez pas de clusters Service Fabric non sécurisés dans un environnement de production.

```shell
sfctl cluster select --endpoint http://testcluster.com:19080
```

Le point de terminaison de cluster doit inclure le préfixe `http` ou `https`. Il doit inclure le port pour la passerelle HTTP. Le port et l’adresse sont identiques à l’URL de Service Fabric Explorer.

Pour les clusters sécurisés avec un certificat, vous pouvez spécifier un certificat codé PEM. Le certificat peut être spécifié en tant que fichier unique ou en tant que certificat et paire de clé. S’il s’agit d’un certificat auto-signé qui n’est pas signé par une autorité de certification, vous pouvez passer l’option `--no-verify` pour ignorer la vérification de l’autorité de certification.

```shell
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem --no-verify
```

Pour plus d’informations, consultez [Se connecter à un cluster sécurisé](service-fabric-connect-to-secure-cluster.md).

## <a name="basic-operations"></a>Opérations de base

Les informations de connexion du cluster sont conservées dans plusieurs sessions de l’interface de ligne de commande Service Fabric. Une fois qu’un cluster Service Fabric est sélectionné, vous pouvez exécuter n’importe quelle commande Service Fabric sur le cluster.

Par exemple, pour obtenir l’état d’intégrité d’un cluster Service Fabric, utilisez la commande suivante :

```shell
sfctl cluster health
```

La commande débouche sur le résultat suivant :

```json
{
  "aggregatedHealthState": "Ok",
  "applicationHealthStates": [
    {
      "aggregatedHealthState": "Ok",
      "name": "fabric:/System"
    }
  ],
  "healthEvents": [],
  "nodeHealthStates": [
    {
      "aggregatedHealthState": "Ok",
      "id": {
        "id": "66aa824a642124089ee474b398d06a57"
      },
      "name": "_Test_0"
    }
  ],
  "unhealthyEvaluations": []
}
```

## <a name="tips-and-troubleshooting"></a>Conseils et résolution des problèmes

Voici quelques suggestions et conseils pour résoudre les problèmes courants.

### <a name="convert-a-certificate-from-pfx-to-pem-format"></a>Convertir un certificat au format PFX en PEM

L’interface de ligne de commande Service Fabric prend en charge les certificats côté client en tant que fichiers PEM (extension .pem). Si vous utilisez des fichiers PFX à partir de Windows, vous devez convertir ces certificats au format PEM. Pour convertir un fichier PFX en fichier PEM, utilisez la commande suivante :

```shell
openssl pkcs12 -in certificate.pfx -out mycert.pem -nodes
```

De même, pour convertir un fichier PEM en fichier PFX, vous pouvez utiliser la commande suivante (aucun mot de passe n’est fourni ici) :

```shell
openssl  pkcs12 -export -out Certificates.pfx -inkey Certificates.pem -in Certificates.pem -passout pass:'' 
```

Pour plus d’informations, consultez la [documentation OpenSSL](https://www.openssl.org/docs/).

### <a name="connection-problems"></a>Problèmes de connexion

Certaines opérations peuvent générer le message suivant :

`Failed to establish a new connection`

Vérifiez que le point de terminaison du cluster spécifié est disponible et à l’écoute. Vérifiez également que l’interface utilisateur de Service Fabric Explorer est disponible au niveau de cet hôte et de ce port. Utilisez `sfctl cluster select` pour mettre à jour le point de terminaison.

### <a name="detailed-logs"></a>Journaux d’activité détaillés

Les journaux d’activité détaillés sont souvent utiles lorsque vous déboguez ou signalez un problème. L’indicateur `--debug` augmente les commentaires de la sortie.

### <a name="command-help-and-syntax"></a>Aide et syntaxe de commande

Pour obtenir de l’aide sur une commande spécifique ou un groupe de commandes, utilisez l’indicateur `-h`.

```shell
sfctl application -h
```

Voici un autre exemple :

```shell
sfctl application create -h
```

## <a name="updating-the-service-fabric-cli"></a>Mettre à jour l’interface de ligne de commande Service Fabric 

Pour mettre à jour l’interface de ligne de commande Service Fabric, exécutez les commandes suivantes (remplacez `pip` par `pip3` selon ce que vous avez choisi lors de l’installation initiale) :

```shell
pip uninstall sfctl
pip install sfctl
```

## <a name="next-steps"></a>Étapes suivantes

* [Déployer une application avec l’interface de ligne de commande Azure Service Fabric](service-fabric-application-lifecycle-sfctl.md)
* [Prise en main de Service Fabric sur Linux](service-fabric-get-started-linux.md)
