---
title: Gérer des images signées
description: Découvrez comment activer l’approbation de contenu pour votre registre de conteneurs Azure et comment envoyer (push) ou extraire (pull) des images signées. L’approbation de contenu implémente l’approbation de contenu de Docker et est une fonctionnalité du niveau de service Premium.
ms.topic: article
ms.date: 09/18/2020
ms.openlocfilehash: f44cea09521dc235ad0d555264b165c9a3842a14
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148577"
---
# <a name="content-trust-in-azure-container-registry"></a>Approbation de contenu dans Azure Container Registry

Azure Container Registry implémente le modèle d’[approbation de contenu][docker-content-trust] de Docker, qui permet l’envoi (push) et le tirage (pull) d’images signées. Cet article vous permet de bien démarrer avec l’activation de l’approbation de contenu dans vos registres de conteneur.

> [!NOTE]
> L’Approbation de contenu est une fonctionnalité du [Niveau de service Premium](container-registry-skus.md) d’Azure Container Registry.

## <a name="how-content-trust-works"></a>Fonctionnement de l’approbation de contenu

Pour tout système distribué dont la conception est axée sur la sécurité, il est important de vérifier à la fois la *source* et *l’intégrité* des données qui pénètrent dans le système. Les consommateurs des données doivent être en mesure de vérifier l’éditeur (source) des données, et de s’assurer que ces données n’ont pas été modifiées après leur publication (intégrité). 

En votre qualité d’éditeur d’images, l’approbation de contenu vous permet de **signer** les images que vous envoyez à votre registre par une transmission de type push. Les consommateurs de vos images (personnes ou systèmes qui extraient des images de votre registre) peuvent configurer leurs clients pour qu’ils extraient (pull) *uniquement* des images signées. Lorsqu’un consommateur d’images extrait une image signée, son client Docker vérifie l’intégrité de cette image. Ce modèle offre aux consommateurs l’assurance que les images signées dans votre registre ont été effectivement publiées par vos soins, et qu’elles n’ont pas été modifiées depuis leur publication.

### <a name="trusted-images"></a>Images approuvées

L’approbation de contenu fonctionne avec les **balises** d’un référentiel. Les référentiels d’images peuvent contenir des images présentant des balises aussi bien signées que non signées. Par exemple, vous pouvez choisir de signer uniquement les images `myimage:stable` et `myimage:latest`, et non les images `myimage:dev`.

### <a name="signing-keys"></a>Clés de signature

L’approbation de contenu est gérée par le biais d’un ensemble de clés de signatures de chiffrement. Ces clés sont associées à un référentiel spécifique dans un registre. Les clients Docker et votre registre utilisent plusieurs types de clés de signature dans le cadre de la gestion de l’approbation relative aux balises d’un référentiel. Lorsque vous activez l’approbation de contenu et que vous l’intégrez à votre pipeline de publication et de consommation de conteneurs, vous devez gérer ces clés avec soin. Pour plus d’informations, consultez la section [Gestion des clés](#key-management) dans la suite de cet article, ainsi que l’article [Manage keys for content trust][docker-manage-keys] (Gérer les clés de l’approbation de contenu) dans la documentation Docker.

> [!TIP]
> Il s’agissait là d’une vue d’ensemble très générale du modèle d’approbation de contenu de Docker. Pour obtenir une présentation détaillée de l’approbation de contenu, consultez l’article [Content trust in Docker][docker-content-trust] (Approbation de contenu dans Docker).

## <a name="enable-registry-content-trust"></a>Activer l’approbation de contenu sur le registre

La première étape consiste à activer l’approbation de contenu au niveau du registre. Une fois que vous activez l’approbation de contenu, les clients (utilisateurs ou services) peuvent envoyer des images signées à votre registre. L’activation de l’approbation de contenu sur votre registre ne limite pas l’utilisation du registre uniquement aux consommateurs ayant activé l’approbation de contenu. Les consommateurs pour lesquels l’approbation de contenu est désactivée peuvent continuer à utiliser votre registre normalement. Toutefois, les consommateurs qui ont activé l’approbation de contenu dans leurs clients seront *uniquement* en mesure de visualiser les images signées dans votre registre.

Pour activer l’approbation de contenu pour votre registre, commencez par accéder au registre dans le Portail Azure. Sous **Stratégies** , sélectionnez **Approbation de contenu** > **Activé** > **Enregistrer** . Vous pouvez également utiliser la commande [az acr config content-trust update][az-acr-config-content-trust-update] dans Azure CLI.

![Capture d’écran montrant l’activation de l’approbation de contenu pour un registre dans le portail Azure.][content-trust-01-portal]

## <a name="enable-client-content-trust"></a>Activer l’approbation de contenu dans les clients

Pour utiliser les images approuvées, les éditeurs et les consommateurs d’images doivent tous activer l’approbation de contenu pour leurs clients Docker. Si vous êtes un éditeur, vous pouvez signer les images que vous envoyez à un registre sur lequel l’activation de contenu est activée. Si vous êtes un consommateur et que vous avez activé l’approbation de contenu, vous ne verrez que les images signées d’un registre. L’approbation de contenu est désactivée par défaut dans les clients Docker, mais vous pouvez l’activer par session d’interpréteur de commandes ou par commande.

Pour activer l’approbation de contenu pour une session d’interpréteur de commandes, définissez la variable d’environnement `DOCKER_CONTENT_TRUST` sur **1** . Par exemple, dans l’interpréteur de commandes Bash :

```bash
# Enable content trust for shell session
export DOCKER_CONTENT_TRUST=1
```

Si vous préférez activer ou désactiver l’approbation de contenu pour une commande spécifique, plusieurs commandes Docker prennent en charge l’argument `--disable-content-trust`. Pour activer l’approbation de contenu pour une seule commande :

```bash
# Enable content trust for single command
docker build --disable-content-trust=false -t myacr.azurecr.io/myimage:v1 .
```

Si vous avez activé l’approbation de contenu pour votre session d’interpréteur de commandes et que vous souhaitez la désactiver pour une commande spécifique :

```bash
# Disable content trust for single command
docker build --disable-content-trust -t myacr.azurecr.io/myimage:v1 .
```

## <a name="grant-image-signing-permissions"></a>Accorder des autorisations de signature d’image

Seuls les utilisateurs ou les systèmes qui disposent des autorisations requises peuvent envoyer des images approuvées à votre registre. Pour accorder l’autorisation d’envoi d’images approuvées à un utilisateur (ou à un système à l’aide d’un principal de service), octroyez à leurs identités Azure Active Directory le rôle `AcrImageSigner`. Ce rôle vient s’ajouter au rôle `AcrPush` (ou équivalent) requis pour l’envoi (push) d’images au registre. Pour plus d’informations, consultez [Autorisations et rôles Azure Container Registry](container-registry-roles.md).

> [!IMPORTANT]
> Vous ne pouvez pas accorder d’autorisation d’envoi (push) d’images approuvées aux comptes d’administration suivants : 
> * le [compte administrateur](container-registry-authentication.md#admin-account) d’un registre de conteneurs Azure ;
> * un compte d’utilisateur dans Azure Active Directory avec le [rôle d’administrateur système classique](../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles).

Les procédures détaillées d’octroi du rôle `AcrImageSigner` dans le Portail Azure et dans l’interface de ligne de commande Azure (Azure CLI) sont décrites ci-après.

### <a name="azure-portal"></a>Portail Azure

Accédez à votre registre dans le portail Azure, puis sélectionnez **Contrôle d’accès (IAM)**  > **Ajouter une attribution de rôle** . Sous **Ajouter une attribution de rôle** , sélectionnez `AcrImageSigner` sous **Rôle** , sélectionnez un ou plusieurs utilisateurs ou principaux de service dans le champ **Sélectionner** , puis cliquez sur **Enregistrer** .

Dans cet exemple, le rôle `AcrImageSigner` a été attribué à deux entités : un principal de service nommé « service-principal » et un utilisateur nommé « Azure User ».

![Accorder des autorisations de signature d’image ACR dans le portail Azure][content-trust-02-portal]

### <a name="azure-cli"></a>Azure CLI

Pour accorder des autorisations de signature à un utilisateur à l’aide d’Azure CLI, assignez à cet utilisateur le rôle `AcrImageSigner`, limité à votre registre. Le format de la commande est le suivant :

```azurecli
az role assignment create --scope <registry ID> --role AcrImageSigner --assignee <user name>
```

Par exemple, pour octroyer le rôle à un utilisateur non administrateur, vous pouvez exécuter les commandes ci-après dans une session Azure CLI authentifiée. Modifiez la valeur `REGISTRY` pour qu’elle reflète le nom de votre registre de conteneurs Azure.

```bash
# Grant signing permissions to authenticated Azure CLI user
REGISTRY=myregistry
REGISTRY_ID=$(az acr show --name $REGISTRY --query id --output tsv)
```

```azurecli
az role assignment create --scope $REGISTRY_ID --role AcrImageSigner --assignee azureuser@contoso.com
```

Vous pouvez également octroyer à un [principal de service](container-registry-auth-service-principal.md) les droits d’envoyer des images approuvées à votre registre. L’utilisation d’un principal de service est utile pour les systèmes de génération et pour d’autres systèmes sans assistance qui doivent envoyer des images approuvées à votre registre. Le format de la commande ci-après est semblable à celui de l’octroi d’une autorisation utilisateur, mais vous devez spécifier un ID de principal de service pour la valeur `--assignee`.

```azurecli
az role assignment create --scope $REGISTRY_ID --role AcrImageSigner --assignee <service principal ID>
```

La variable `<service principal ID>` peut correspondre à l’élément **appId** ou **objectId** du principal de service, ou à l’un de ses éléments **servicePrincipalNames** . Pour plus d’informations sur l’utilisation des principaux de service et d’Azure Container Registry, consultez l’article [Authentification Azure Container Registry avec des principaux de service](container-registry-auth-service-principal.md).

> [!IMPORTANT]
> Après toute modification de rôle, exécutez `az acr login` pour actualiser le jeton d’identité local pour Azure CLI afin que les nouveaux rôles puissent entrer en vigueur. Pour plus d’informations sur la vérification des rôles d’une identité, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](../role-based-access-control/role-assignments-cli.md) et [Résoudre les problèmes liés à Azure RBAC](../role-based-access-control/troubleshooting.md).

## <a name="push-a-trusted-image"></a>Envoyer une image approuvée

Pour envoyer une balise d’image approuvée à votre registre de conteneurs, activez l’approbation de contenu, puis envoyez l’image avec `docker push`. Après la première exécution d’envoi (push) avec une balise signée, vous êtes invité à créer une phrase secrète à la fois pour une clé de signature racine et pour une clé de signature de référentiel. La clé racine et la clé de référentiel sont générées et stockées localement sur votre machine.

```console
$ docker push myregistry.azurecr.io/myimage:v1
[...]
The push refers to repository [myregistry.azurecr.io/myimage]
ee83fc5847cb: Pushed
v1: digest: sha256:aca41a608e5eb015f1ec6755f490f3be26b48010b178e78c00eac21ffbe246f1 size: 524
Signing and pushing trust metadata
You are about to create a new root signing key passphrase. This passphrase
will be used to protect the most sensitive key in your signing system. Please
choose a long, complex passphrase and be careful to keep the password and the
key file itself secure and backed up. It is highly recommended that you use a
password manager to generate the passphrase and keep it safe. There will be no
way to recover this key. You can find the key in your config directory.
Enter passphrase for new root key with ID 4c6c56a:
Repeat passphrase for new root key with ID 4c6c56a:
Enter passphrase for new repository key with ID bcd6d98:
Repeat passphrase for new repository key with ID bcd6d98:
Finished initializing "myregistry.azurecr.io/myimage"
Successfully signed myregistry.azurecr.io/myimage:v1
```

Après l’exécution de votre première commande `docker push` avec l’approbation de contenu activée, le client Docker utilise la même clé racine pour les envois suivants. Pour chaque envoi ultérieur au même référentiel, vous êtes uniquement invité à fournir la clé de référentiel. Chaque fois que vous envoyez une image approuvée à un nouveau référentiel, vous devez fournir une phrase secrète pour une nouvelle clé de référentiel.

## <a name="pull-a-trusted-image"></a>Extraire une image approuvée

Pour extraire une image approuvée, activez l’approbation de contenu et exécutez la commande `docker pull` normalement. Pour tirer (pull) des images approuvées, le rôle `AcrPull` suffit pour les utilisateurs normaux. Aucun rôle supplémentaire comme le rôle `AcrImageSigner` n’est nécessaire. Les consommateurs ayant activé l’approbation de contenu peuvent uniquement extraire les images dotées de balises signées. Voici un exemple d’extraction de balise signée :

```console
$ docker pull myregistry.azurecr.io/myimage:signed
Pull (1 of 1): myregistry.azurecr.io/myimage:signed@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b: Pulling from myimage
8e3ba11ec2a2: Pull complete
Digest: sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
Status: Downloaded newer image for myregistry.azurecr.io/myimage@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
Tagging myregistry.azurecr.io/myimage@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b as myregistry.azurecr.io/myimage:signed
```

Si un client pour lequel l’approbation de contenu est activée tente d’extraire une balise non signée, l’opération échoue avec une erreur semblable à la suivante :

```console
$ docker pull myregistry.azurecr.io/myimage:unsigned
Error: remote trust data does not exist
```

### <a name="behind-the-scenes"></a>Dans les coulisses

Quand vous exécutez la commande `docker pull`, le client Docker utilise la même bibliothèque que dans l’[interface de ligne de commande Notary][docker-notary-cli] pour demander le mappage sur SHA-256 du code de hachage de la balise que vous extrayez. Après la validation des signatures sur les données d’approbation, le client demande au moteur Docker d’effectuer une « extraction par code de hachage ». Pendant l’extraction, le moteur utilise la somme de contrôle SHA-256 comme adresse du contenu pour demander et valider le manifeste d’image à partir du registre de conteneurs Azure.

> [!NOTE]
> Azure Container Registry ne prend pas officiellement en charge l’interface de ligne de commande Notary, mais est compatible avec l’API Notary Server qui est incluse avec Docker Desktop. Actuellement, la version **0.6.0** est recommandée.

## <a name="key-management"></a>Gestion des clés

Comme indiqué pour la sortie `docker push` lorsque vous envoyez votre première image approuvée, la clé racine constitue l’information la plus sensible. Veillez à sauvegarder votre clé racine et à la stocker en lieu sûr. Par défaut, le client Docker stocke les clés de signature dans le répertoire suivant :

```sh
~/.docker/trust/private
```

Sauvegardez vos clés racine et de référentiel en les compressant dans une archive et en les stockant dans un emplacement sécurisé. Par exemple, dans Bash :

```bash
umask 077; tar -zcvf docker_private_keys_backup.tar.gz ~/.docker/trust/private; umask 022
```

Parallèlement aux clés racine et de référentiel générées localement, plusieurs autres clés sont générées et stockées par Azure Container Registry lorsque vous envoyez une image approuvée. Pour obtenir une présentation détaillée des différentes clés dans l’implémentation de l’approbation de contenu de Docker, et notamment des instructions de gestion supplémentaires, consultez l’article [Manage keys for content trust][docker-manage-keys] (Gérer les clés de l’approbation de contenu) dans la documentation Docker.

### <a name="lost-root-key"></a>Clé racine perdue

Si vous perdez l’accès à votre clé racine, vous perdez l’accès aux balises signées dans tous les référentiels dont les balises ont été signées avec cette clé. Azure Container Registry ne peut pas restaurer l’accès aux balises d’image qui ont été signées avec une clé racine perdue. Pour supprimer toutes les données d’approbation (signatures) de votre registre, commencez par désactiver l’approbation de contenu pour le registre, puis réactivez-la.

> [!WARNING]
> La désactivation et la réactivation de l’approbation de contenu dans votre registre **suppriment toutes les données d’approbation pour toutes les balises signées dans chaque référentiel de votre registre** . Cette opération est irréversible : Azure Container Registry ne peut pas récupérer les données d’approbation supprimées. La désactivation de l’approbation de contenu ne supprime pas les images proprement dites.

Pour désactiver l’approbation de contenu pour votre registre, accédez au registre dans le Portail Azure. Sous **Stratégies** , sélectionnez **Approbation de contenu** > **Désactivé** > **Enregistrer** . Un message vous signale la perte de toutes les signatures dans le registre. Sélectionnez **OK** pour supprimer définitivement toutes les signatures dans votre registre.

![Désactivation de l’approbation de contenu pour un registre dans le Portail Azure][content-trust-03-portal]

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’approbation de contenu, notamment les commandes d’[approbation Docker](https://docs.docker.com/engine/reference/commandline/trust/) et les [délégations d’approbation](https://docs.docker.com/engine/security/trust/trust_delegation/), voir [Approbation de contenu dans Docker][docker-content-trust]. Bien que cet article ait abordé plusieurs points clés, l’approbation de contenu est un vaste sujet couvert de manière plus approfondie dans la documentation Docker.

* Consultez la documentation [Azure Pipelines](/azure/devops/pipelines/build/content-trust) pour obtenir un exemple d’utilisation de l’approbation de contenu quand vous générez et effectuez un Push d’une image Docker.

<!-- IMAGES> -->
[content-trust-01-portal]: ./media/container-registry-content-trust/content-trust-01-portal.png
[content-trust-02-portal]: ./media/container-registry-content-trust/content-trust-02-portal.png
[content-trust-03-portal]: ./media/container-registry-content-trust/content-trust-03-portal.png

<!-- LINKS - external -->
[docker-content-trust]: https://docs.docker.com/engine/security/trust/content_trust
[docker-manage-keys]: https://docs.docker.com/engine/security/trust/trust_key_mng/
[docker-notary-cli]: https://docs.docker.com/notary/getting_started/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-config-content-trust-update]: /cli/azure/acr/config/content-trust#az-acr-config-content-trust-update
