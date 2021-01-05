---
title: 'Tutoriel : Personnaliser une machine virtuelle Linux avec cloud-init dans Azure'
description: Avec ce tutoriel, vous allez apprendre à utiliser cloud-init et Key Vault pour personnaliser les machines virtuelles Linux lors de leur premier démarrage dans Azure.
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
ms.date: 09/12/2019
ms.author: cynthn
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 456c42dc0b25e168744ce283cddbd63b877813ab
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92747154"
---
# <a name="tutorial---how-to-use-cloud-init-to-customize-a-linux-virtual-machine-in-azure-on-first-boot"></a>Didacticiel : comment utiliser cloud-init pour personnaliser une machine virtuelle Linux dans Azure au premier démarrage

Dans un didacticiel précédent, vous avez appris comment établir une connexion SSH à une machine virtuelle et installer NGINX manuellement. Pour créer des machines virtuelles de façon rapide et cohérente, une certaine forme d’automatisation est généralement souhaitable. Pour personnaliser une machine virtuelle au premier démarrage, l’approche la plus courante consiste à utiliser [cloud-init](https://cloudinit.readthedocs.io). Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un fichier de configuration cloud-init
> * Créer une machine virtuelle utilisant un fichier cloud-init
> * Afficher une application Node.js en cours d’exécution une fois la machine virtuelle est créée
> * Utiliser un Key Vault pour stocker des certificats en toute sécurité
> * Automatiser des déploiements sécurisés de NGINX avec cloud-init

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, ce didacticiel nécessite que vous exécutiez Azure CLI version 2.0.30 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="cloud-init-overview"></a>Présentation de cloud-init
[Cloud-init](https://cloudinit.readthedocs.io) est une méthode largement utilisée pour personnaliser une machine virtuelle Linux lors de son premier démarrage. Vous pouvez utiliser cloud-init pour installer des packages et écrire des fichiers, ou encore pour configurer des utilisateurs ou des paramètres de sécurité. Comme cloud-init s’exécute pendant le processus de démarrage initial, aucune autre étape ni aucun agent ne sont nécessaires pour appliquer votre configuration.

Cloud-init fonctionne aussi sur les différentes distributions. Par exemple, vous n’utilisez pas **apt-get install** ou **yum install** pour installer un package. Au lieu de cela, vous pouvez définir une liste des packages à installer, après quoi cloud-init se charge d’utiliser automatiquement l’outil de gestion de package natif correspondant à la distribution que vous sélectionnez.

Nous collaborons avec nos partenaires pour que cloud-init soit inclus et fonctionne dans les images qu’ils fournissent à Azure. Le tableau suivant présente la disponibilité actuelle de cloud-init sur les images de plateforme Azure :

| Serveur de publication | Offre | SKU | Version | Compatible avec cloud-init |
|:--- |:--- |:--- |:--- |:--- |
|Canonical |UbuntuServer |18.04-LTS |latest |Oui | 
|Canonical |UbuntuServer |16.04-LTS |latest |Oui | 
|Canonical |UbuntuServer |14.04.5-LTS |latest |Oui |
|CoreOS |CoreOS |Stable |latest |Oui |
|OpenLogic 7.6 |CentOS |7-CI |latest |preview |
|RedHat 7.6 |RHEL |7-RAW-CI |7.6.2019072418 |Oui |
|RedHat 7.7 |RHEL |7-RAW-CI |7.7.2019081601 |preview |


## <a name="create-cloud-init-config-file"></a>Créer un fichier de configuration cloud-init
Pour voir le cloud-init en action, créez une machine virtuelle qui installe NGINX et exécute une simple application « Hello World » Node.js. La configuration cloud-init suivante installe les packages, crée une application Node.js, puis initialise et démarre l’application.

À l’invite bash ou dans Cloud Shell, créez un fichier nommé *cloud-init.txt* , puis collez la configuration suivante. Par exemple, tapez `sensible-editor cloud-init.txt` pour créer le fichier et voir la liste des éditeurs disponibles. Vérifiez que l’intégralité du fichier cloud-init est copiée, en particulier la première ligne :

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
    path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
    path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

Pour plus d’informations sur les options de configuration de cloud-init, consultez les [exemples de configuration cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/examples.html).

## <a name="create-virtual-machine"></a>Créer une machine virtuelle
Pour pouvoir créer une machine virtuelle, vous devez créer un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). L’exemple suivant crée un groupe de ressources nommé *myResourceGroupAutomate* dans l’emplacement *westus* :

```azurecli-interactive
az group create --name myResourceGroupAutomate --location eastus
```

Créez maintenant une machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create). Utilisez le paramètre `--custom-data` à transmettre dans votre fichier de configuration cloud-init. Indiquez le chemin complet vers la configuration *cloud-init.txt* si vous avez enregistré le fichier en dehors de votre répertoire de travail actuel. L’exemple suivant crée une machine virtuelle nommée *myVM*  :

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupAutomate \
    --name myAutomatedVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
```

Vous devez patienter quelques minutes le temps que la machine virtuelle soit créée, que les packages soient installés et que l’application démarre. Certaines tâches en arrière-plan continuent à s’exécuter une fois que l’interface CLI Azure vous renvoie à l’invite de commandes. Un délai de quelques minutes peut être nécessaire avant que vous puissiez accéder à l’application. Une fois la machine virtuelle créée, notez la valeur de `publicIpAddress` qui s’affiche dans l’interface Azure CLI. Cette adresse est utilisée pour accéder à l’application Node.js via un navigateur web.

Pour autoriser le trafic web à accéder à votre machine virtuelle, ouvrez le port 80 à partir d’Internet à l’aide de la commande [az vm open-port](/cli/azure/vm#az-vm-open-port) :

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroupAutomate --name myAutomatedVM
```

## <a name="test-web-app"></a>Tester l’application web
Vous pouvez maintenant ouvrir un navigateur web et entrer *http:\/\/\<publicIpAddress>* dans la barre d’adresse. Indiquez votre propre adresse IP publique à partir du processus de création de la machine virtuelle. Votre application Node.js apparaît telle que dans l’exemple suivant :

![Afficher le site NGINX en cours d’exécution](./media/tutorial-automate-vm-deployment/nginx.png)


## <a name="inject-certificates-from-key-vault"></a>Injecter des certificats à partir de Key Vault
Cette section montre comment stocker des certificats en toute sécurité dans Azure Key Vault et comment les injecter lors du déploiement de la machine virtuelle. Au lieu d’utiliser une image personnalisée qui inclut les certificats intégrés, ce processus permet d’injecter les certificats les plus récents dans une machine virtuelle dès le premier démarrage. Pendant le processus, le certificat ne quitte jamais la plateforme Azure ; il est exposé dans un script, un historique de ligne de commande ou un modèle.

Azure Key Vault protège les clés de chiffrement et les secrets, tels que les certificats ou les mots de passe. Key Vault rationalise le processus de gestion de clés et vous permet de garder le contrôle des clés qui accèdent à vos données et les chiffrent. Ce scénario présente quelques concepts de Key Vault permettant de créer et utiliser un certificat ; il ne prétend pas détailler de manière exhaustive l’utilisation de Key Vault.

Les étapes suivantes vous expliquent comment :

- Créer un Azure Key Vault
- Générer ou télécharger un certificat dans Key Vault
- Créer un secret à partir du certificat pour l’injecter dans une machine virtuelle
- Créer une machine virtuelle et injecter le certificat

### <a name="create-an-azure-key-vault"></a>Créer un Azure Key Vault
Commencez par créer un Key Vault avec la commande [az keyvault create](/cli/azure/keyvault#az-keyvault-create) et activez son utilisation lors du déploiement d’une machine virtuelle. Chaque Key Vault requiert un nom unique en minuscules. Remplacez *mykeyvault* dans l’exemple suivant par le nom unique de votre propre Key Vault :

```azurecli-interactive
keyvault_name=mykeyvault
az keyvault create \
    --resource-group myResourceGroupAutomate \
    --name $keyvault_name \
    --enabled-for-deployment
```

### <a name="generate-certificate-and-store-in-key-vault"></a>Générer le certificat et le stocker dans Key Vault
Dans un environnement de production, vous devez importer un certificat valide signé par un fournisseur approuvé à l’aide de la commande [az keyvault certificate import](/cli/azure/keyvault/certificate#az-keyvault-certificate-import). Pour ce didacticiel, l’exemple suivant vous montre comment générer un certificat auto-signé avec la commande [az keyvault certificate create](/cli/azure/keyvault/certificate#az-keyvault-certificate-create) qui utilise la stratégie de certificat par défaut :

```azurecli-interactive
az keyvault certificate create \
    --vault-name $keyvault_name \
    --name mycert \
    --policy "$(az keyvault certificate get-default-policy --output json)"
```


### <a name="prepare-certificate-for-use-with-vm"></a>Préparer le certificat en vue de son utilisation avec la machine virtuelle
Pour utiliser le certificat au cours du processus de création de la machine virtuelle, récupérez l’ID de votre certificat à l’aide de la commande [az keyvault secret list-versions](/cli/azure/keyvault/secret#az-keyvault-secret-list-versions). Le certificat doit respecter un certain format pour être injecté au démarrage du système par la machine virtuelle. Par conséquent, convertissez le certificat au [format secret az vm](/cli/azure/vm). L’exemple suivant affecte la sortie de ces commandes à des variables, afin de simplifier la procédure dans les étapes suivantes :

```azurecli-interactive
secret=$(az keyvault secret list-versions \
          --vault-name $keyvault_name \
          --name mycert \
          --query "[?attributes.enabled].id" --output tsv)
vm_secret=$(az vm secret format --secret "$secret" --output json)
```


### <a name="create-cloud-init-config-to-secure-nginx"></a>Créer la configuration cloud-init pour sécuriser NGINX
Lorsque vous créez une machine virtuelle, les certificats et les clés sont stockés dans le répertoire */var/lib/waagent/* protégé. Pour ajouter le certificat à la machine virtuelle et configurer NGINX de façon automatique, vous pouvez utiliser une configuration cloud-init mise à jour par rapport à l’exemple précédent.

Créez un fichier nommé *cloud-init-secured.txt* et collez la configuration suivante. Si vous utilisez Cloud Shell, créez le fichier config cloud-init ici, et non sur votre machine locale. Par exemple, tapez `sensible-editor cloud-init-secured.txt` pour créer le fichier et voir la liste des éditeurs disponibles. Vérifiez que l’intégralité du fichier cloud-init est copiée, en particulier la première ligne :

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
    path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        listen 443 ssl;
        ssl_certificate /etc/nginx/ssl/mycert.cert;
        ssl_certificate_key /etc/nginx/ssl/mycert.prv;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
    path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - secretsname=$(find /var/lib/waagent/ -name "*.prv" | cut -c -57)
  - mkdir /etc/nginx/ssl
  - cp $secretsname.crt /etc/nginx/ssl/mycert.cert
  - cp $secretsname.prv /etc/nginx/ssl/mycert.prv
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

### <a name="create-secure-vm"></a>Créer une machine virtuelle sécurisée
Créez maintenant une machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create). Les données de certificat sont injectées à partir de Key Vault avec le paramètre `--secrets`. Comme dans l’exemple précédent, vous allez également transmettre la configuration de cloud-init avec le paramètre `--custom-data` :

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupAutomate \
    --name myVMWithCerts \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-secured.txt \
    --secrets "$vm_secret"
```

Vous devez patienter quelques minutes le temps que la machine virtuelle soit créée, que les packages soient installés et que l’application démarre. Certaines tâches en arrière-plan continuent à s’exécuter une fois que l’interface CLI Azure vous renvoie à l’invite de commandes. Un délai de quelques minutes peut être nécessaire avant que vous puissiez accéder à l’application. Une fois la machine virtuelle créée, notez la valeur de `publicIpAddress` qui s’affiche dans l’interface Azure CLI. Cette adresse est utilisée pour accéder à l’application Node.js via un navigateur web.

Pour autoriser le trafic web sécurisé à accéder à votre machine virtuelle, ouvrez le port 443 à partir d’Internet à l’aide de la commande [az vm open-port](/cli/azure/vm#az-vm-open-port) :

```azurecli-interactive
az vm open-port \
    --resource-group myResourceGroupAutomate \
    --name myVMWithCerts \
    --port 443
```

### <a name="test-secure-web-app"></a>Tester l’application web sécurisée
Vous pouvez maintenant ouvrir un navigateur web et entrer *https:\/\/\<publicIpAddress>* dans la barre d’adresse. Fournissez votre propre adresse IP publique comme indiqué dans la sortie du processus précédent de création de machine virtuelle. Acceptez l’avertissement de sécurité si vous avez utilisé un certificat auto-signé :

![Accepter l’avertissement de sécurité du navigateur web](./media/tutorial-automate-vm-deployment/browser-warning.png)

Votre site NGINX sécurisé et votre application Node.js apparaissent maintenant dans l’exemple suivant :

![Afficher le site NGINX sécurisé en cours d’exécution](./media/tutorial-automate-vm-deployment/secured-nginx.png)


## <a name="next-steps"></a>Étapes suivantes
Ce didacticiel vous a montré comment configurer des machines virtuelles au premier démarrage avec cloud-init. Vous avez appris à :

> [!div class="checklist"]
> * Créer un fichier de configuration cloud-init
> * Créer une machine virtuelle utilisant un fichier cloud-init
> * Afficher une application Node.js en cours d’exécution une fois la machine virtuelle est créée
> * Utiliser un Key Vault pour stocker des certificats en toute sécurité
> * Automatiser des déploiements sécurisés de NGINX avec cloud-init

Passez au didacticiel suivant pour découvrir comment créer des images de machine virtuelle personnalisées.

> [!div class="nextstepaction"]
> [Créer des images de machine virtuelle personnalisées](./tutorial-custom-images.md)
