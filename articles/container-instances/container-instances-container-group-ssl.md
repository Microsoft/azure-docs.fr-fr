---
title: Activer TLS avec un conteneur side-car
description: Créer un point de terminaison SSL ou TLS pour un groupe de conteneurs exécuté sur Azure Container Instances en exécutant Nginx dans un conteneur side-car
ms.topic: article
ms.date: 07/02/2020
ms.openlocfilehash: 6587a84e7cbe655c509f74e9e39e93010e7058be
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558077"
---
# <a name="enable-a-tls-endpoint-in-a-sidecar-container"></a>Activer un point de terminaison TLS dans un conteneur side-car

Cet article montre comment créer un [groupe de conteneurs](container-instances-container-groups.md) avec un conteneur d’application et un conteneur side-car exécutant un fournisseur SSL/TLS. En configurant un groupe de conteneurs avec un point de terminaison TLS distinct, vous activez les connexions TLS pour votre application sans modifier votre code d’application.

Vous configurez un exemple de groupe de conteneurs composé de deux conteneurs :
* Un conteneur d’applications qui exécute une application web simple à l’aide de l’image [aci-helloworld](https://hub.docker.com/_/microsoft-azuredocs-aci-helloworld) Microsoft publique. 
* Un conteneur side-car exécutant l’image [Nginx](https://hub.docker.com/_/nginx) publique, configuré pour utiliser TLS. 

Dans cet exemple, le groupe de conteneurs expose uniquement le port 443 pour Nginx avec son adresse IP publique. Nginx achemine les requêtes HTTPS vers l’application web, qui écoute en interne sur le port 80. Vous pouvez adapter l’exemple pour les applications de conteneur qui écoutent sur les autres ports. 

Consultez la section [Étapes suivantes](#next-steps) pour découvrir d’autres approches d’activation du protocole TLS dans un groupe de conteneurs.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.55 ou une version ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-self-signed-certificate"></a>Créer un certificat auto-signé

Pour configurer Nginx en tant que fournisseur TLS, vous avez besoin d’un certificat TLS/SSL. Cet article explique comment créer et configurer un certificat TLS/SSL auto-signé. Pour les scénarios de production, vous devez obtenir un certificat délivré par une autorité de certification.

Pour créer un certificat TLS/SSL auto-signé, utilisez l’outil [OpenSSL](https://www.openssl.org/) disponible dans Azure Cloud Shell et de nombreuses distributions Linux, ou utilisez un outil client comparables de votre système d’exploitation.

Commencez par créer une demande de certificat (fichier .csr) dans un répertoire de travail local :

```console
openssl req -new -newkey rsa:2048 -nodes -keyout ssl.key -out ssl.csr
```

Suivez les invites pour ajouter les informations d’identification. Pour le nom commun, entrez le nom d’hôte associé au certificat. Lorsque vous êtes invité à saisir un mot de passe, appuyez sur Entrée sans ajouter de texte pour ignorer l’ajout d’un mot de passe.

Exécutez la commande suivante pour créer le certificat auto-signé (fichier .crt) à partir de la demande de certificat. Par exemple :

```console
openssl x509 -req -days 365 -in ssl.csr -signkey ssl.key -out ssl.crt
```

Vous devriez maintenant voir trois fichiers dans le répertoire : la demande de certificat (`ssl.csr`), la clé privée (`ssl.key`) et le certificat auto-signé (`ssl.crt`). Vous utilisez `ssl.key` et `ssl.crt` dans les étapes ultérieures.

## <a name="configure-nginx-to-use-tls"></a>Configurer Nginx pour utiliser TLS

### <a name="create-nginx-configuration-file"></a>Créer un fichier de configuration Nginx

Dans cette section, vous allez créer un fichier de configuration afin que Nginx utilise TLS. Commencez par copier le texte suivant dans un nouveau fichier nommé `nginx.conf`. Dans Azure Cloud Shell, vous pouvez utiliser Visual Studio Code pour créer le fichier dans votre répertoire de travail :

```console
code nginx.conf
```

Dans `location`, veillez à définir `proxy_pass` avec le port approprié pour votre application. Dans cet exemple, nous avons défini le port 80 pour le conteneur `aci-helloworld`.

```console
# nginx Configuration File
# https://wiki.nginx.org/Configuration

# Run as a less privileged user for security reasons.
user nginx;

worker_processes auto;

events {
  worker_connections 1024;
}

pid        /var/run/nginx.pid;

http {

    #Redirect to https, using 307 instead of 301 to preserve post data

    server {
        listen [::]:443 ssl;
        listen 443 ssl;

        server_name localhost;

        # Protect against the BEAST attack by not using SSLv3 at all. If you need to support older browsers (IE6) you may need to add
        # SSLv3 to the list of protocols below.
        ssl_protocols              TLSv1.2;

        # Ciphers set to best allow protection from Beast, while providing forwarding secrecy, as defined by Mozilla - https://wiki.mozilla.org/Security/Server_Side_TLS#Nginx
        ssl_ciphers                ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:ECDHE-RSA-RC4-SHA:ECDHE-ECDSA-RC4-SHA:AES128:AES256:RC4-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!3DES:!MD5:!PSK;
        ssl_prefer_server_ciphers  on;

        # Optimize TLS/SSL by caching session parameters for 10 minutes. This cuts down on the number of expensive TLS/SSL handshakes.
        # The handshake is the most CPU-intensive operation, and by default it is re-negotiated on every new/parallel connection.
        # By enabling a cache (of type "shared between all Nginx workers"), we tell the client to re-use the already negotiated state.
        # Further optimization can be achieved by raising keepalive_timeout, but that shouldn't be done unless you serve primarily HTTPS.
        ssl_session_cache    shared:SSL:10m; # a 1mb cache can hold about 4000 sessions, so we can hold 40000 sessions
        ssl_session_timeout  24h;


        # Use a higher keepalive timeout to reduce the need for repeated handshakes
        keepalive_timeout 300; # up from 75 secs default

        # remember the certificate for a year and automatically connect to HTTPS
        add_header Strict-Transport-Security 'max-age=31536000; includeSubDomains';

        ssl_certificate      /etc/nginx/ssl.crt;
        ssl_certificate_key  /etc/nginx/ssl.key;

        location / {
            proxy_pass http://localhost:80; # TODO: replace port if app listens on port other than 80
            
            proxy_set_header Connection "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
        }
    }
}
```

### <a name="base64-encode-secrets-and-configuration-file"></a>Encoder les secrets et le fichier de configuration au format Base64

Encodez le fichier de configuration Nginx, le certificat TLS/SSL et la clé TLS au format Base64. Dans la section suivante, vous entrez le contenu encodé dans un fichier YAML qui permet de déployer le groupe de conteneurs.

```console
cat nginx.conf | base64 > base64-nginx.conf
cat ssl.crt | base64 > base64-ssl.crt
cat ssl.key | base64 > base64-ssl.key
```

## <a name="deploy-container-group"></a>Déployer le groupe de conteneurs

Déployez maintenant le groupe de conteneurs en spécifiant les configurations de conteneur dans un [fichier YAML](container-instances-multi-container-yaml.md).

### <a name="create-yaml-file"></a>Créer un fichier YAML

Copiez le fichier YAML suivant dans un nouveau fichier nommé `deploy-aci.yaml`. Dans Azure Cloud Shell, vous pouvez utiliser Visual Studio Code pour créer le fichier dans votre répertoire de travail :

```console
code deploy-aci.yaml
```

Entrez le contenu des fichiers encodés au format Base64 à l’endroit indiqué sous `secret`. Par exemple, `cat` chacun des fichiers encodés au format Base64 pour voir son contenu. Au cours du déploiement, ces fichiers sont ajoutés à un [volume secret](container-instances-volume-secret.md) dans le groupe de conteneurs. Dans cet exemple, le volume secret est monté sur le conteneur Nginx.

```YAML
api-version: 2019-12-01
location: westus
name: app-with-ssl
properties:
  containers:
  - name: nginx-with-ssl
    properties:
      image: nginx
      ports:
      - port: 443
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - name: nginx-config
        mountPath: /etc/nginx
  - name: my-app
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld
      ports:
      - port: 80
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  volumes:
  - secret:
      ssl.crt: <Enter contents of base64-ssl.crt here>
      ssl.key: <Enter contents of base64-ssl.key here>
      nginx.conf: <Enter contents of base64-nginx.conf here>
    name: nginx-config
  ipAddress:
    ports:
    - port: 443
      protocol: TCP
    type: Public
  osType: Linux
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

### <a name="deploy-the-container-group"></a>Déployer le groupe de conteneurs

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create) :

```azurecli-interactive
az group create --name myResourceGroup --location westus
```

Déployez le groupe de conteneurs avec la commande [az container create](/cli/azure/container#az-container-create), en transmettant le fichier YAML en tant qu’argument.

```azurecli
az container create --resource-group <myResourceGroup> --file deploy-aci.yaml
```

### <a name="view-deployment-state"></a>Afficher l’état du déploiement

Pour afficher l’état du déploiement, utilisez la commande [az container show](/cli/azure/container#az-container-show) suivante :

```azurecli
az container show --resource-group <myResourceGroup> --name app-with-ssl --output table
```

Si votre déploiement est exécuté avec succès, vous voyez une sortie ressemblant à ce qui suit :

```console
Name          ResourceGroup    Status    Image                                                    IP:ports             Network    CPU/Memory       OsType    Location
------------  ---------------  --------  -------------------------------------------------------  -------------------  ---------  ---------------  --------  ----------
app-with-ssl  myresourcegroup  Running   nginx, mcr.microsoft.com/azuredocs/aci-helloworld        52.157.22.76:443     Public     1.0 core/1.5 gb  Linux     westus
```

## <a name="verify-tls-connection"></a>Vérifier la connexion TLS

Utilisez votre navigateur pour accéder à l’adresse IP publique du groupe de conteneurs. L’adresse IP indiquée dans cet exemple étant `52.157.22.76`, l’URL est **https://52.157.22.76** . Vous devez utiliser HTTPS pour voir l’application en cours d’exécution, en raison de la configuration du serveur Nginx. Les tentatives de connexion via HTTP échouent.

![Capture d’écran du navigateur représentant une application exécutée dans une instance de conteneur Azure](./media/container-instances-container-group-ssl/aci-app-ssl-browser.png)

> [!NOTE]
> Étant donné que cet exemple utilise un certificat auto-signé et non un certificat émanant d’une autorité de certification, le navigateur affiche un avertissement de sécurité lors de la connexion au site via le protocole HTTPS. Vous devrez peut-être accepter l’avertissement ou ajuster les paramètres du navigateur ou du certificat pour passer à la page. Il s’agit du comportement attendu.

>

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a montré comment configurer un conteneur Nginx pour activer les connexions TLS à une application web exécutée dans le groupe de conteneurs. Vous pouvez adapter cet exemple pour les applications qui écoutent sur les ports autres que le port 80. Vous pouvez également mettre à jour le fichier de configuration Nginx pour rediriger automatiquement les connexions au serveur sur le port 80 (HTTP) afin d’utiliser le protocole HTTPS.

Bien que cet article utilise Nginx pour le conteneur side-car, vous pouvez utiliser un autre fournisseur TLS comme [Caddy](https://caddyserver.com/).

Si vous déployez votre groupe de conteneurs sur un [réseau virtuel Azure](container-instances-vnet.md), vous pouvez envisager d’autres options afin d’activer un point de terminaison TLS pour une instance de conteneur back-end, notamment :

* [Azure Functions Proxies](../azure-functions/functions-proxies.md)
* [Gestion des API Azure](../api-management/api-management-key-concepts.md)
* [Azure Application Gateway](../application-gateway/overview.md) : consultez un exemple de [modèle de déploiement](https://github.com/Azure/azure-quickstart-templates/tree/master/201-aci-wordpress-vnet).
