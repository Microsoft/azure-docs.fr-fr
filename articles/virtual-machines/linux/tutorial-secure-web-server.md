---
title: 'Tutoriel : Sécuriser un serveur web Linux à l’aide de certificats TLS/SSL dans Azure'
description: Dans ce tutoriel, vous allez apprendre à utiliser Azure CLI pour sécuriser une machine virtuelle Linux qui exécute le serveur web NGINX avec des certificats SSL stockés dans Azure Key Vault.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/30/2018
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 4a790c51cd0caa2c81275e7eafdd663f2f2f0116
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92740205"
---
# <a name="tutorial-secure-a-web-server-on-a-linux-virtual-machine-in-azure-with-tlsssl-certificates-stored-in-key-vault"></a>Tutoriel : Sécuriser un serveur web sur une machine virtuelle Linux dans Azure avec des certificats TLS/SSL stockés dans Key Vault
Pour sécuriser les serveurs web, vous pouvez utiliser un certificat TLS (Transport Layer Security), anciennement SSL (Secure Sockets Layer), et chiffrer ainsi le trafic web. Ces certificats TLS/SSL peuvent être stockés dans Azure Key Vault et autoriser des déploiements sécurisés de certificats sur des machines virtuelles Linux dans Azure. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un Azure Key Vault
> * Générer ou télécharger un certificat dans Key Vault
> * Créer une machine virtuelle et installer le serveur web NGINX
> * Injecter le certificat dans la machine virtuelle et configurer NGINX à l’aide d’une liaison TLS

Ce tutoriel utilise l’interface CLI disponible dans [Azure Cloud Shell](../../cloud-shell/overview.md), qui est constamment mise à jour vers la dernière version. Pour ouvrir Cloud Shell, sélectionnez **Essayer** en haut d’un bloc de code.

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, ce didacticiel nécessite que vous exécutiez Azure CLI version 2.0.30 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).


## <a name="overview"></a>Vue d’ensemble
Azure Key Vault protège les clés de chiffrement et les secrets, tels que les certificats ou les mots de passe. Key Vault rationalise le processus de gestion de certificats et vous permet de garder le contrôle sur les clés d’accès à ces certificats. Vous pouvez créer un certificat auto-signé à l’intérieur de Key Vault ou charger un certificat approuvé existant que vous avez déjà.

Au lieu d’utiliser une image de machine virtuelle personnalisée qui inclut des certificats intégrés, vous injectez des certificats dans une machine virtuelle en cours d’exécution. Ce processus garantit que les certificats les plus récents sont installés sur un serveur web pendant le déploiement. Si vous renouvelez ou remplacez un certificat, vous n’êtes pas non plus obligé de créer une image de machine virtuelle personnalisée. Les certificats les plus récents sont automatiquement injectés à la création des machines virtuelles supplémentaires. Pendant tout le processus, les certificats ne quittent jamais la plateforme Azure, ni ne sont exposés dans un script, un historique de ligne de commande ou un modèle.


## <a name="create-an-azure-key-vault"></a>Créer un Azure Key Vault
Avant de créer un coffre de clés et des certificats, créez un groupe de ressources avec [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroupSecureWeb* à l’emplacement *eastus*  :

```azurecli-interactive 
az group create --name myResourceGroupSecureWeb --location eastus
```

Ensuite, créez un coffre de clés avec la commande [az keyvault create](/cli/azure/keyvault) et activez son utilisation lors du déploiement d’une machine virtuelle. Chaque Key Vault requiert un nom unique en minuscules. Remplacez *\<mykeyvault>* dans l’exemple suivant par le nom unique de votre propre Key Vault :

```azurecli-interactive 
keyvault_name=<mykeyvault>
az keyvault create \
    --resource-group myResourceGroupSecureWeb \
    --name $keyvault_name \
    --enabled-for-deployment
```

## <a name="generate-a-certificate-and-store-in-key-vault"></a>Générer un certificat et le stocker dans Key Vault
Dans un environnement de production, vous devez importer un certificat valide signé par un fournisseur approuvé à l’aide de la commande [az keyvault certificate import](/cli/azure/keyvault/certificate). Pour ce didacticiel, l’exemple suivant vous montre comment générer un certificat auto-signé avec la commande [az keyvault certificate create](/cli/azure/keyvault/certificate) qui utilise la stratégie de certificat par défaut :

```azurecli-interactive 
az keyvault certificate create \
    --vault-name $keyvault_name \
    --name mycert \
    --policy "$(az keyvault certificate get-default-policy)"
```

### <a name="prepare-a-certificate-for-use-with-a-vm"></a>Préparer un certificat en vue de son utilisation avec une machine virtuelle
Pour utiliser le certificat au cours du processus de création de la machine virtuelle, récupérez l’ID de votre certificat à l’aide de la commande [az keyvault secret list-versions](/cli/azure/keyvault/secret). Utilisez la commande [az vm secret format](/cli/azure/vm/secret#az-vm-secret-format) pour convertir le certificat. L’exemple suivant affecte la sortie de ces commandes à des variables, afin de simplifier la procédure dans les étapes suivantes :

```azurecli-interactive 
secret=$(az keyvault secret list-versions \
          --vault-name $keyvault_name \
          --name mycert \
          --query "[?attributes.enabled].id" --output tsv)
vm_secret=$(az vm secret format --secrets "$secret" -g myResourceGroupSecureWeb --keyvault $keyvault_name)
```

### <a name="create-a-cloud-init-config-to-secure-nginx"></a>Créer une configuration cloud-init pour sécuriser NGINX
[Cloud-init](https://cloudinit.readthedocs.io) est une méthode largement utilisée pour personnaliser une machine virtuelle Linux lors de son premier démarrage. Vous pouvez utiliser cloud-init pour installer des packages et écrire des fichiers, ou encore pour configurer des utilisateurs ou des paramètres de sécurité. Comme cloud-init s’exécute pendant le processus de démarrage initial, aucune autre étape ni aucun agent ne sont nécessaires pour appliquer votre configuration.

Lorsque vous créez une machine virtuelle, les certificats et les clés sont stockés dans le répertoire */var/lib/waagent/* protégé. Pour automatiser l’ajout du certificat à la machine virtuelle et la configuration du serveur web, utilisez cloud-init. Dans cet exemple, vous allez installer et configurer le serveur web NGINX. Vous pouvez utiliser le même processus pour installer et configurer Apache. 

Créez un fichier nommé *cloud-init-web-server.txt* et collez la configuration suivante :

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 443 ssl;
        ssl_certificate /etc/nginx/ssl/mycert.cert;
        ssl_certificate_key /etc/nginx/ssl/mycert.prv;
      }
runcmd:
  - secretsname=$(find /var/lib/waagent/ -name "*.prv" | cut -c -57)
  - mkdir /etc/nginx/ssl
  - cp $secretsname.crt /etc/nginx/ssl/mycert.cert
  - cp $secretsname.prv /etc/nginx/ssl/mycert.prv
  - service nginx restart
```

### <a name="create-a-secure-vm"></a>Créer une machine virtuelle sécurisée
Créez maintenant une machine virtuelle avec la commande [az vm create](/cli/azure/vm). Les données de certificat sont injectées à partir de Key Vault avec le paramètre `--secrets`. Vous transmettez la configuration cloud-init à l’aide du paramètre `--custom-data` :

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroupSecureWeb \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-web-server.txt \
    --secrets "$vm_secret"
```

Vous devez patienter quelques minutes le temps que la machine virtuelle soit créée, que les packages soient installés et que l’application démarre. Une fois la machine virtuelle créée, notez la valeur de `publicIpAddress` qui s’affiche dans l’interface Azure CLI. Cette adresse est utilisée pour accéder à votre site à l’aide d’un navigateur web.

Pour autoriser le trafic web sécurisé à accéder à votre machine virtuelle, ouvrez le port 443 à partir d’Internet à l’aide de la commande [az vm open-port](/cli/azure/vm) :

```azurecli-interactive 
az vm open-port \
    --resource-group myResourceGroupSecureWeb \
    --name myVM \
    --port 443
```


### <a name="test-the-secure-web-app"></a>Tester l’application web sécurisée
Vous pouvez maintenant ouvrir un navigateur web et entrer *https:\/\/\<publicIpAddress>* dans la barre d’adresse. Indiquez votre propre adresse IP publique à partir du processus de création de la machine virtuelle. Acceptez l’avertissement de sécurité si vous avez utilisé un certificat auto-signé :

![Accepter l’avertissement de sécurité du navigateur web](./media/tutorial-secure-web-server/browser-warning.png)

Votre site NGINX sécurisé apparaît maintenant comme dans l’exemple suivant :

![Afficher le site NGINX sécurisé en cours d’exécution](./media/tutorial-secure-web-server/secured-nginx.png)


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez sécurisé un serveur web NGINX à l’aide d’un certificat TLS/SSL stocké dans Azure Key Vault. Vous avez appris à :

> [!div class="checklist"]
> * Créer un Azure Key Vault
> * Générer ou télécharger un certificat dans Key Vault
> * Créer une machine virtuelle et installer le serveur web NGINX
> * Injecter le certificat dans la machine virtuelle et configurer NGINX à l’aide d’une liaison TLS

Suivez ce lien pour consulter des exemples de scripts de machine virtuelle prédéfinis.

> [!div class="nextstepaction"]
> [Exemples de scripts de machine virtuelle Linux](./cli-samples.md)
