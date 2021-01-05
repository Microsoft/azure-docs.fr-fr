---
title: Sécuriser des services web à l’aide du protocole TLS
titleSuffix: Azure Machine Learning
description: Découvrez comment activer HTTPS avec TLS version 1.2 pour sécuriser un service web déployé par le biais d’Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: jmartens
ms.author: aashishb
author: aashishb
ms.date: 11/18/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-azurecli
ms.openlocfilehash: f7e16400f6460f7479cdffd1928126cdd70a8f0c
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97503996"
---
# <a name="use-tls-to-secure-a-web-service-through-azure-machine-learning"></a>Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning


Cet article vous expliquer comment sécuriser un service web déployé par le biais d’Azure Machine Learning.

Vous utilisez [HTTPS](https://en.wikipedia.org/wiki/HTTPS) pour restreindre l’accès aux services web et sécuriser les données soumises par les clients. HTTPS permet de sécuriser les communications entre un client et un service web en chiffrant les communications entre les deux. Le chiffrement utilise [TLS (Transport Layer Security)](https://en.wikipedia.org/wiki/Transport_Layer_Security). TLS est toujours parfois appelé *SSL (Secure Sockets Layer)* , qui était le prédécesseur de TLS.

> [!TIP]
> Le kit de développement logiciel (SDK) Azure Machine Learning utilise le terme « SSL » pour les propriétés liées aux communications sécurisées. Cela ne signifie pas que votre service web n’utilise pas *TLS*. SSL est tout simplement un terme plus répandu.
>
> Plus précisément, les services web déployés par le biais d’Azure Machine Learning prennent uniquement en charge TLS version 1.2 pour AKS et ACI. Pour les déploiements ACI, si vous utilisez une ancienne version de TLS, nous vous recommandons d’effectuer un nouveau déploiement pour obtenir la dernière version de TLS.

TLS et SSL s'appuient tous deux sur des *certificats numériques* qui facilitent le chiffrement et la vérification d'identité. Pour plus d’informations sur le fonctionnement des certificats numériques, consultez l'article Wikipédia [Infrastructure à clé publique](https://en.wikipedia.org/wiki/Public_key_infrastructure).

> [!WARNING]
> Si vous n’utilisez pas HTTPS pour votre service web, les données envoyées vers et depuis le service peuvent être visibles par d'autres personnes sur Internet.
>
> HTTPS permet également au client de vérifier l’authenticité du serveur auquel il se connecte. Cette fonctionnalité protège les clients des attaques [de l'intercepteur](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

Voici le processus général visant à sécuriser un service web :

1. Obtenir un nom de domaine.

2. Obtenir un certificat numérique.

3. Déployer ou mettre à jour le service web sur lequel TLS est activé.

4. Mettre à jour votre DNS afin qu’il pointe vers le service web.

> [!IMPORTANT]
> Si vous effectuez un déploiement vers Azure Kubernetes Service (AKS), vous pouvez acheter votre propre certificat ou utiliser un certificat fourni par Microsoft. Si vous utilisez un certificat fourni par Microsoft, vous n'êtes pas tenu d'obtenir un nom de domaine ou un certificat TLS/SSL. Pour plus d’informations, consultez la section [Activer le protocole TLS et le déployer](#enable) de cet article.

La sécurisation des services web sur les [cibles de déploiement](how-to-deploy-and-where.md) présente de légères différences.

## <a name="get-a-domain-name"></a>Obtenir un nom de domaine

Si vous ne possédez pas de nom de domaine, achetez-en-un auprès d’un *bureau d’enregistrement de noms de domaine*. Le processus et le prix varient selon les bureaux d’enregistrement. Le bureau d’enregistrement propose des outils pour gérer le nom de domaine. Vous utilisez ces outils pour mapper un nom de domaine complet (FQDN) (par exemple, www\.contoso.com) à l’adresse IP qui héberge votre service web.

## <a name="get-a-tlsssl-certificate"></a>Obtenir un certificat TLS/SSL

Il existe de nombreuses façons d’obtenir un certificat TLS/SSL (certificat numérique). Le moyen le plus courant consiste à en acheter un auprès d’une *autorité de certification* (CA). Quel que soit l’endroit où vous obtenez le certificat, vous avez besoin des fichiers suivants :

* Un **certificat**. Le certificat doit contenir la chaîne d’approbation et doit être « codé en PEM ».
* Une **clé**. La clé doit aussi être codée en PEM.

Lorsque vous demandez un certificat, vous devez fournir le nom de domaine complet (FQDN) de l’adresse que vous envisagez d’utiliser pour le service web (par exemple, www\.contoso.com). L’adresse estampillée sur le certificat et l’adresse utilisée par les clients sont comparées pour vérifier l’identité du service web. Si ces adresses ne correspondent pas, le client reçoit un message d’erreur.

> [!TIP]
> Si l’autorité de certification ne peut pas fournir le certificat et la clé sous forme de fichiers PEM, vous pouvez utiliser un utilitaire tel que [OpenSSL](https://www.openssl.org/) pour modifier le format.

> [!WARNING]
> Utilisez les certificats *auto-signés* uniquement pour le développement. Ne les utilisez pas dans des environnements de production. Les certificats autosignés peuvent entraîner des problèmes dans vos applications clientes. Pour plus d’informations, consultez la documentation relative aux bibliothèques réseau que votre application cliente utilise.

## <a name="enable-tls-and-deploy"></a><a id="enable"></a> Activer le protocole TLS et le déployer

Pour déployer (ou redéployer) le service avec le protocole TLS activé, définissez le paramètre *ssl_enabled* sur « True », si nécessaire. Définissez le paramètre *ssl_certificate* sur la valeur du fichier *certificate*. Définissez *ssl_key* sur la valeur du fichier *key*.

### <a name="deploy-on-aks-and-field-programmable-gate-array-fpga"></a>Déployer AKS et FPGA (Field-Programmable Gate Array)

  > [!NOTE]
  > Les informations contenues dans cette section s’appliquent également lorsque vous déployez un service web sécurisé pour le concepteur. Si vous n’êtes pas habitué à utiliser le kit de développement logiciel (SDK) Python, consultez [Présentation du kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py).

Lorsque vous déployez sur AKS, vous pouvez créer un cluster AKS ou attacher un cluster existant. Pour plus d’informations sur la création ou l’attachement d’un cluster, consultez [Déployer un modèle sur un cluster Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).
  
-  Si vous créez un cluster, vous utilisez **[AksCompute.provisioning_configuration()](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py&preserve-view=true#&preserve-view=trueprovisioning-configuration-agent-count-none--vm-size-none--ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--location-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--service-cidr-none--dns-service-ip-none--docker-bridge-cidr-none--cluster-purpose-none--load-balancer-type-none--load-balancer-subnet-none-)** .
- Si vous attachez un cluster existant, vous utilisez **[AksCompute.attach_configuration()](/python/api/azureml-core/azureml.core.compute.akscompute?view=azure-ml-py&preserve-view=true#&preserve-view=trueattach-configuration-resource-group-none--cluster-name-none--resource-id-none--cluster-purpose-none-)** . Tous deux renvoient un objet de configuration présentant une méthode **enable_ssl**.

La méthode **enable_ssl** peut utiliser un certificat fourni par Microsoft ou un certificat que vous achetez.

> [!WARNING]
> Si votre cluster AKS est configuré avec un équilibreur de charge interne, l'utilisation d'un certificat fourni par Microsoft n'est __pas prise en charge__. L'utilisation d'un certificat fourni par Microsoft nécessite une ressource IP publique dans Azure, qui n'est pas disponible si votre cluster AKS est configuré avec un équilibreur de charge interne.

  * Lorsque vous utilisez un certificat fourni par Microsoft, vous devez utiliser le paramètre *leaf_domain_label*. Ce paramètre génère le nom DNS du service. Par exemple, une valeur « contoso » crée un nom de domaine « contoso\<six-random-characters>.\<azureregion>.cloudapp.azure.com », où \<azureregion> correspond à la région contenant le service. Vous pouvez également utiliser le paramètre *overwrite_existing_domain* pour remplacer le paramètre *leaf_domain_label* existant.

    Pour déployer (ou redéployer) le service avec le protocole TLS activé, définissez le paramètre *ssl_enabled* sur « True », si nécessaire. Définissez le paramètre *ssl_certificate* sur la valeur du fichier *certificate*. Définissez *ssl_key* sur la valeur du fichier *key*.

    > [!IMPORTANT]
    > Lorsque vous utilisez un certificat fourni par Microsoft, vous n'êtes pas tenu d'acheter votre propre certificat ou nom de domaine.

    L’exemple suivant montre comment créer une configuration qui utilise un certificat TLS/SSL fourni par Microsoft :

    ```python
    from azureml.core.compute import AksCompute
    # Config used to create a new AKS cluster and enable TLS
    provisioning_config = AksCompute.provisioning_configuration()
    # Leaf domain label generates a name using the formula
    #  "<leaf-domain-label>######.<azure-region>.cloudapp.azure.net"
    #  where "######" is a random series of characters
    provisioning_config.enable_ssl(leaf_domain_label = "contoso")


    # Config used to attach an existing AKS cluster to your workspace and enable TLS
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                          cluster_name = cluster_name)
    # Leaf domain label generates a name using the formula
    #  "<leaf-domain-label>######.<azure-region>.cloudapp.azure.net"
    #  where "######" is a random series of characters
    attach_config.enable_ssl(leaf_domain_label = "contoso")
    ```

  * Lorsque vous utilisez *un certificat que vous avez acheté*, vous utilisez les paramètres *ssl_cert_pem_file*, *ssl_key_pem_file* et *ssl_cname*. L’exemple suivant montre comment utiliser les fichiers *.pem* pour créer une configuration s'appuyant sur un certificat TLS/SSL que vous avez acheté :

    ```python
    from azureml.core.compute import AksCompute
    # Config used to create a new AKS cluster and enable TLS
    provisioning_config = AksCompute.provisioning_configuration()
    provisioning_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    # Config used to attach an existing AKS cluster to your workspace and enable SSL
    attach_config = AksCompute.attach_configuration(resource_group = resource_group,
                                         cluster_name = cluster_name)
    attach_config.enable_ssl(ssl_cert_pem_file="cert.pem",
                                        ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
    ```

Pour plus d’informations sur *enable_ssl*, consultez [AksProvisioningConfiguration.enable_ssl()](/python/api/azureml-core/azureml.core.compute.aks.aksprovisioningconfiguration?preserve-view=true&view=azure-ml-py#&preserve-view=trueenable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-) et [AksAttachConfiguration.enable_ssl()](/python/api/azureml-core/azureml.core.compute.aks.aksattachconfiguration?preserve-view=true&view=azure-ml-py#&preserve-view=trueenable-ssl-ssl-cname-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--leaf-domain-label-none--overwrite-existing-domain-false-).

### <a name="deploy-on-azure-container-instances"></a>Déployer sur Azure Container Instances

Lorsque vous déployez sur Azure Container Instances, vous fournissez des valeurs pour les paramètres TLS, comme l'illustre l’extrait de code suivant :

```python
from azureml.core.webservice import AciWebservice

aci_config = AciWebservice.deploy_configuration(
    ssl_enabled=True, ssl_cert_pem_file="cert.pem", ssl_key_pem_file="key.pem", ssl_cname="www.contoso.com")
```

Pour plus d’informations, consultez [AciWebservice.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.aciwebservice#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none-).

## <a name="update-your-dns"></a>Mettre à jour votre DNS

Vous devez ensuite mettre à jour votre DNS afin qu’il pointe vers le service web.

+ **Pour Container Instances :**

  Utilisez les outils de votre bureau d’enregistrement de noms de domaine pour mettre à jour l’enregistrement DNS pour votre nom de domaine. L’enregistrement doit pointer vers l’adresse IP du service.

  Selon le bureau d’enregistrement et la durée de vie (TTL) configurée pour le nom de domaine, les clients peuvent avoir à patienter quelques minutes voire plusieurs heures avant de pouvoir résoudre le nom de domaine.

+ **Pour AKS :**

  > [!WARNING]
  > Si vous avez utilisé *leaf_domain_label* pour créer le service à l’aide d’un certificat fourni par Microsoft, ne mettez pas à jour manuellement la valeur DNS du cluster. Cette valeur doit être définie automatiquement.
  >
  > Si votre cluster AKS est configuré avec un équilibreur de charge interne, l'utilisation d'un certificat fourni par Microsoft (en définissant *leaf_domain_label*) n'est __pas prise en charge__. L'utilisation d'un certificat fourni par Microsoft nécessite une ressource IP publique dans Azure, qui n'est pas disponible si votre cluster AKS est configuré avec un équilibreur de charge interne.
  Mettez à jour le DNS de l’adresse IP publique du cluster AKS sur l’onglet **Configuration**, sous **Paramètres** dans le volet de gauche. (Consultez l'image suivante.) L’adresse IP publique est un type de ressource créé sous le groupe de ressources qui contient les nœuds d’agent AKS et d’autres ressources de mise en réseau.

  [![Azure Machine Learning : Sécurisation des services web avec TLS](./media/how-to-secure-web-service/aks-public-ip-address.png)](./media/how-to-secure-web-service/aks-public-ip-address-expanded.png)

## <a name="update-the-tlsssl-certificate"></a>Mettre à jour le certificat TLS/SSL

Les certificats TLS/SSL expirent et doivent être renouvelés. En général, cela se produit chaque année. Utilisez les informations des sections suivantes pour mettre à jour et renouveler votre certificat pour les modèles déployés sur Azure Kubernetes Service :

### <a name="update-a-microsoft-generated-certificate"></a>Mettre à jour un certificat généré par Microsoft

Si le certificat a été initialement généré par Microsoft (avec *leaf_domain_label* pour créer le service), utilisez un des exemples suivants pour mettre à jour le certificat :

> [!IMPORTANT]
> * Si le certificat existant est toujours valide, utilisez `renew=True` (Kit de développement logiciel [SDK]) ou `--ssl-renew` (CLI) pour forcer la configuration à le renouveler. Par exemple, si le certificat existant est encore valide pendant 10 jours et que vous n’utilisez pas `renew=True`, le certificat ne peut pas être renouvelé.
> * Lorsque le service a été déployé à l’origine, le `leaf_domain_label` a été utilisé pour créer un nom DNS selon le modèle `<leaf-domain-label>######.<azure-region>.cloudapp.azure.net`. Pour conserver le nom existant (y compris les six chiffres générés à l’origine), utilisez la valeur `leaf_domain_label` d’origine. N’incluez pas les six chiffres qui ont été générés.

**Utilisez le SDK**

```python
from azureml.core.compute import AksCompute
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute.aks import SslConfiguration

# Get the existing cluster
aks_target = AksCompute(ws, clustername)

# Update the existing certificate by referencing the leaf domain label
ssl_configuration = SslConfiguration(leaf_domain_label="myaks", overwrite_existing_domain=True, renew=True)
update_config = AksUpdateConfiguration(ssl_configuration)
aks_target.update(update_config)
```

**Utiliser l’interface CLI**

```azurecli
az ml computetarget update aks -g "myresourcegroup" -w "myresourceworkspace" -n "myaks" --ssl-leaf-domain-label "myaks" --ssl-overwrite-domain True --ssl-renew
```

Pour en savoir plus, consultez les documents de référence suivants :

* [SslConfiguration](/python/api/azureml-core/azureml.core.compute.aks.sslconfiguration?preserve-view=true&view=azure-ml-py)
* [AksUpdateConfiguration](/python/api/azureml-core/azureml.core.compute.aks.aksupdateconfiguration?preserve-view=true&view=azure-ml-py)

### <a name="update-custom-certificate"></a>Mettre à jour un certificat personnalisé

Si le certificat a été initialement généré par une autorité de certification, procédez comme suit :

1. Utilisez la documentation fournie par l’autorité de certification pour renouveler le certificat. Ce processus crée de nouveaux fichiers de certificat.

1. Utilisez le kit de développement logiciel (SDK) ou l’interface CLI pour mettre à jour le service avec le nouveau certificat :

    **Utilisez le SDK**

    ```python
    from azureml.core.compute import AksCompute
    from azureml.core.compute.aks import AksUpdateConfiguration
    from azureml.core.compute.aks import SslConfiguration
    
    # Read the certificate file
    def get_content(file_name):
        with open(file_name, 'r') as f:
            return f.read()

    # Get the existing cluster
    aks_target = AksCompute(ws, clustername)
    
    # Update cluster with custom certificate
    ssl_configuration = SslConfiguration(cname="myaks", cert=get_content('cert.pem'), key=get_content('key.pem'))
    update_config = AksUpdateConfiguration(ssl_configuration)
    aks_target.update(update_config)
    ```

    **Utiliser l’interface CLI**

    ```azurecli
    az ml computetarget update aks -g "myresourcegroup" -w "myresourceworkspace" -n "myaks" --ssl-cname "myaks"--ssl-cert-file "cert.pem" --ssl-key-file "key.pem"
    ```

Pour en savoir plus, consultez les documents de référence suivants :

* [SslConfiguration](/python/api/azureml-core/azureml.core.compute.aks.sslconfiguration?preserve-view=true&view=azure-ml-py)
* [AksUpdateConfiguration](/python/api/azureml-core/azureml.core.compute.aks.aksupdateconfiguration?preserve-view=true&view=azure-ml-py)

## <a name="disable-tls"></a>Désactiver le protocole TLS

Pour désactiver TLS pour un modèle déployé sur Azure Kubernetes Service, créez un `SslConfiguration` avec `status="Disabled"`, puis effectuez une mise à jour :

```python
from azureml.core.compute import AksCompute
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute.aks import SslConfiguration

# Get the existing cluster
aks_target = AksCompute(ws, clustername)

# Disable TLS
ssl_configuration = SslConfiguration(status="Disabled")
update_config = AksUpdateConfiguration(ssl_configuration)
aks_target.update(update_config)
```

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment :
+ [Utiliser un modèle Machine Learning déployé en tant que service web](how-to-consume-web-service.md)
+ [Vue d’ensemble de l’isolement et de la confidentialité des réseaux virtuels](how-to-network-security-overview.md)