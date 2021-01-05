---
title: Ajouter et gérer des certificats TLS/SSL
description: Créez un certificat gratuit, importez un certificat App Service, importez un certificat Key Vault, ou achetez un certificat App Service dans Azure App Service.
tags: buy-ssl-certificates
ms.topic: tutorial
ms.date: 10/25/2019
ms.reviewer: yutlin
ms.custom: seodec18
ms.openlocfilehash: dff98a5c54d2fee350e2b35dc00148c19ea233b8
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94956498"
---
# <a name="add-a-tlsssl-certificate-in-azure-app-service"></a>Ajouter un certificat TLS/SSL dans Azure App Service

[Azure App Service](overview.md) offre un service d’hébergement web hautement évolutif appliquant des mises à jour correctives automatiques. Cet article explique comment créer, charger ou importer un certificat privé ou public dans App Service. 

Une fois le certificat ajouté à votre application App Service ou à votre [application de fonction](../azure-functions/index.yml), vous pouvez [sécuriser un nom DNS personnalisé avec celui-ci](configure-ssl-bindings.md) ou l’[utiliser dans votre code d’application](configure-ssl-certificate-in-code.md).

Le tableau suivant répertorie les options permettant d’ajouter des certificats dans App Service :

|Option|Description|
|-|-|
| Créer un certificat managé App Service gratuit (préversion) | Certificat privé facile à utiliser si vous devez simplement sécuriser votre `www` [domaine personnalisé](app-service-web-tutorial-custom-domain.md) ou tout autre domaine non nu dans App Service. |
| Acheter un certificat App Service | Certificat privé managé par Azure. Il combine la simplicité d’une gestion automatisée et la flexibilité des options de renouvellement et d’exportation. |
| Importer un certificat à partir de Key Vault | Utile si vous utilisez [Azure Key Vault](../key-vault/index.yml) pour gérer vos [certificats PKCS12](https://wikipedia.org/wiki/PKCS_12). Consultez [Exigences concernant les certificats privés](#private-certificate-requirements). |
| Téléchargement d’un certificat privé | Si vous disposez déjà d’un certificat privé provenant d’un fournisseur tiers, vous pouvez le charger. Consultez [Exigences concernant les certificats privés](#private-certificate-requirements). |
| Téléchargement d’un certificat public | Les certificats publics ne sont pas utilisés pour sécuriser les domaines personnalisés, mais vous pouvez les charger dans votre code si vous en avez besoin pour accéder aux ressources distantes. |

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de ce guide pratique, vous devez au préalable :

- [Créez une application App Service](./index.yml).
- Certificat gratuit uniquement : mappez un sous-domaine (par exemple, `www.contoso.com`) à App Service avec un [enregistrement CNAME](app-service-web-tutorial-custom-domain.md#map-a-cname-record).

## <a name="private-certificate-requirements"></a>Exigences concernant les certificats privés

> [!NOTE]
> Azure Web Apps ne prend **pas** en charge AES256, et tous les fichiers pfx doivent être chiffrés avec TripleDES.

Le [certificat managé App Service gratuit](#create-a-free-certificate-preview) et le [certificat App Service](#import-an-app-service-certificate) répondent déjà aux exigences App Service. Si vous choisissez de charger ou d’importer un certificat privé dans App Service, votre certificat doit :

* Avoir été exporté sous la forme d’un [fichier PFX protégé par mot de passe](https://en.wikipedia.org/w/index.php?title=X.509&section=4#Certificate_filename_extensions)
* Contenir une clé privée d’au moins 2048 bits de long
* Contenir tous les certificats intermédiaires dans la chaîne de certificats

Pour sécuriser un domaine personnalisé dans une liaison TLS/SSL, votre certificat doit répondre à des exigences supplémentaires :

* Contenir une [utilisation améliorée de la clé ](https://en.wikipedia.org/w/index.php?title=X.509&section=4#Extensions_informing_a_specific_usage_of_a_certificate) pour l’authentification du serveur (OID = 1.3.6.1.5.5.7.3.1)
* Être signé par une autorité de certification approuvée

> [!NOTE]
> Les **certificats de chiffrement à courbe elliptique (ECC)** sont compatibles avec App Service, mais ce sujet sort du cadre de cet article. Consultez votre autorité de certification sur les étapes à suivre pour créer des certificats ECC.

[!INCLUDE [Prepare your web app](../../includes/app-service-ssl-prepare-app.md)]

## <a name="create-a-free-certificate-preview"></a>Créer un certificat gratuit (préversion)

Le certificat managé App Service gratuit est une solution clé en main pour la sécurisation de votre nom DNS personnalisé dans App Service. Il s’agit d’un certificat TLS/SSL entièrement fonctionnel géré par App Service et renouvelé automatiquement. Le certificat gratuit comprend les limitations suivantes :

- Ne prend pas en charge les certificats avec caractères génériques.
- Il ne prend pas en charge les domaines nus.
- Il n’est pas exportable.
- N’est pas pris en charge sur App Service Environment (ASE)
- Ne prend pas en charge les enregistrements A. Par exemple, le renouvellement automatique ne fonctionne pas avec les enregistrements A.

> [!NOTE]
> Le certificat gratuit est émis par DigiCert. Pour certains domaines de niveau supérieur, vous devez autoriser explicitement DigiCert comme émetteur de certificat en créant un [enregistrement de domaine CAA](https://wikipedia.org/wiki/DNS_Certification_Authority_Authorization) avec la valeur : `0 issue digicert.com`.
> 

Pour créer un certificat managé App Service gratuit :

Sur le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, dans le menu de gauche, sélectionnez **App Services** >  **\<app-name>** .

Dans le panneau de navigation de gauche de votre application, sélectionnez **Paramètres TLS/SSL** > **Certificats à clé privée (.pfx)**  > **Créer un certificat managé App Service**.

![Créer un certificat gratuit dans App Service](./media/configure-ssl-certificate/create-free-cert.png)

Tous les domaines non nus correctement mappés à votre application avec un enregistrement CNAME sont listés dans la boîte de dialogue. Sélectionnez le domaine personnalisé pour lequel créer un certificat gratuit, puis sélectionnez **Créer**. Vous ne pouvez créer qu’un seul certificat pour chaque domaine personnalisé pris en charge.

Une fois l’opération terminée, le certificat s’affiche dans la liste **Certificats à clé privée**.

![Création du certificat gratuit terminée](./media/configure-ssl-certificate/create-free-cert-finished.png)

> [!IMPORTANT] 
> Pour sécuriser un domaine personnalisé avec ce certificat, vous devez quand même créer une liaison de certificat. Suivez les étapes fournies dans [Créer une liaison](configure-ssl-bindings.md#create-binding).
>

## <a name="import-an-app-service-certificate"></a>Importer un certificat App Service

Si vous achetez un certificat App Service dans Azure, Azure se charge des tâches suivantes :

- Prise en charge du processus d’achat sur GoDaddy
- Vérification du domaine du certificat
- Gestion du certificat dans [Azure Key Vault](../key-vault/general/overview.md)
- Gestion du renouvellement des certificats (voir [Renouveler le certificat](#renew-certificate))
- Synchronisation automatique des certificats avec les copies importées dans les applications App Service

Pour acheter un certificat App Service, consultez [Commander un certificat](#start-certificate-order).

Si vous disposez déjà d’un certificat App Service, vous pouvez :

- [Importer le certificat dans App Service](#import-certificate-into-app-service)
- [Gérer le certificat](#manage-app-service-certificates), par exemple, le renouveler, renouveler sa clé ou l’exporter
> [!NOTE]
> Les certificats App Service ne sont pas pris en charge dans les clouds nationaux Azure à l’heure actuelle.

### <a name="start-certificate-order"></a>Commander un certificat

Commandez un certificat App Service dans la <a href="https://portal.azure.com/#create/Microsoft.SSL" target="_blank">page de création App Service Certificate</a>.

![Démarrer l’achat d’un certificat App Service](./media/configure-ssl-certificate/purchase-app-service-cert.png)

Aidez-vous du tableau suivant pour configurer le certificat. Lorsque vous avez terminé, cliquez sur **Créer**.

| Paramètre | Description |
|-|-|
| Nom | Nom convivial de votre certificat App Service. |
| Nom d’hôte de domaine nu | Spécifiez ici le domaine racine. Le certificat émis sécurise *à la fois* le domaine racine et le sous-domaine `www`. Dans le certificat émis, le champ Nom commun contient le domaine racine et le champ Autre nom de l’objet contient le domaine `www`. Pour sécuriser un sous-domaine uniquement, indiquez ici son nom de domaine complet (par exemple, `mysubdomain.contoso.com`).|
| Abonnement | Abonnement qui contient le certificat. |
| Resource group | Groupe de ressources qui doit contenir le certificat. Vous pouvez utiliser un nouveau groupe de ressources, ou sélectionner le même groupe de ressources que votre application App Service, par exemple. |
| Référence (SKU) de certificat | Détermine le type de certificat à créer (certificat standard ou [certificat générique](https://wikipedia.org/wiki/Wildcard_certificate)). |
| Termes et conditions | Cliquez pour confirmer que vous acceptez les termes et conditions. Les certificats sont obtenus auprès de GoDaddy. |

> [!NOTE]
> Les certificats App Service achetés auprès d’Azure sont émis par GoDaddy. Pour certains domaines de niveau supérieur, vous devez autoriser explicitement GoDaddy comme émetteur de certificat en créant un [enregistrement de domaine CAA](https://wikipedia.org/wiki/DNS_Certification_Authority_Authorization) avec la valeur : `0 issue godaddy.com`
> 

### <a name="store-in-azure-key-vault"></a>Stocker dans Azure Key Vault

Après l’achat du certificat, il vous reste quelques étapes à effectuer avant de commencer à utiliser ce certificat. 

Sélectionnez le certificat dans la page [App Service Certificates](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders), puis cliquez sur **Configuration du certificat** > **Étape 1 : Stocker**.

![Configurer le stockage Key Vault du certificat App Service](./media/configure-ssl-certificate/configure-key-vault.png)

[Key Vault](../key-vault/general/overview.md) est un service Azure qui permet de protéger les clés de chiffrement et les secrets utilisés par les services et les applications cloud. C’est le stockage recommandé pour les certificats App Service.

Dans la page **État de Key Vault**, cliquez sur **Référentiel Key Vault** pour créer un coffre ou choisir un coffre existant. Si vous créez un coffre, configurez le nouveau coffre en vous aidant du tableau ci-dessous, puis cliquez sur Créer. Créez le coffre de clés dans le même abonnement et groupe de ressources que votre application App Service.

| Paramètre | Description |
|-|-|
| Nom | Nom unique composé de caractères alphanumériques et de tirets. |
| Resource group | Nous vous conseillons de choisir le même groupe de ressources que votre certificat App Service. |
| Emplacement | Choisissez le même emplacement que votre application App Service. |
| Niveau tarifaire | Pour obtenir des informations sur les tarifs, consultez [Tarification d’Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/). |
| Stratégies d’accès| Définit les applications et l’accès autorisé aux ressources du coffre. Vous pouvez configurer ce paramètre ultérieurement, en suivant les étapes décrites dans [Attribuer une stratégie d’accès Key Vault](../key-vault/general/assign-access-policy-portal.md). |
| Accès au réseau virtuel | Limitez l’accès au coffre à certains réseaux virtuels Azure. Vous pouvez configurer ce paramètre ultérieurement, en suivant les étapes décrites dans [Configurer les pare-feux et réseaux virtuels d’Azure Key Vault](../key-vault/general/network-security.md) |

Une fois que vous avez sélectionné le coffre, fermez la page **Référentiel Key Vault**. L’option **Étape 1 : Stocker** doit afficher une coche verte de réussite. Gardez cette page ouverte pour l’étape suivante.

### <a name="verify-domain-ownership"></a>Vérifier la propriété du domaine

Dans la même page **Configuration du certificat** utilisée à l’étape précédente, cliquez sur **Étape 2 : Vérifier**.

![Vérifier le domaine du certificat App Service](./media/configure-ssl-certificate/verify-domain.png)

Sélectionnez **Vérification App Service**. Étant donné que vous avez déjà mappé le domaine à votre application web (voir la section [Prérequis](#prerequisites)), le domaine a déjà été vérifié. Cliquez simplement sur **Vérifier** pour terminer cette étape. Cliquez sur le bouton **Actualiser** jusqu’à ce que le message **Le domaine du certificat a été vérifié** s’affiche.

> [!NOTE]
> Quatre types de méthodes de vérification du domaine sont pris en charge : 
> 
> - **App Service** : option la plus pratique quand le domaine a déjà été mappé à une application App Service dans le même abonnement. Elle profite du fait que l’application App Service a déjà vérifié la propriété du domaine.
> - **Domaine** : permet de vérifier un [domaine App Service que vous avez acheté sur Azure](manage-custom-dns-buy-domain.md). Azure ajoute automatiquement l’enregistrement TXT de vérification pour vous et effectue le processus.
> - **E-mail** : permet de vérifier le domaine en envoyant un e-mail à l’administrateur de domaine. Quand vous sélectionnez cette option, des instructions supplémentaires s’affichent.
> - **Manuelle** : permet de vérifier le domaine à l’aide d’une page HTML (certificat **Standard** uniquement) ou d’un enregistrement TXT DNS. Quand vous sélectionnez cette option, des instructions supplémentaires s’affichent.

### <a name="import-certificate-into-app-service"></a>Importer le certificat dans App Service

Sur le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, dans le menu de gauche, sélectionnez **App Services** >  **\<app-name>** .

Dans le panneau de navigation de gauche de votre application, sélectionnez **Paramètres TLS/SSL** > **Certificats à clé privée (.pfx)**  > **Importer un certificat App Service**.

![Importer un certificat App Service dans App Service](./media/configure-ssl-certificate/import-app-service-cert.png)

Sélectionnez le certificat que vous venez d’acheter, puis sélectionnez **OK**.

Une fois l’opération terminée, le certificat s’affiche dans la liste **Certificats à clé privée**.

![Importation du certificat App Service terminée](./media/configure-ssl-certificate/import-app-service-cert-finished.png)

> [!IMPORTANT] 
> Pour sécuriser un domaine personnalisé avec ce certificat, vous devez quand même créer une liaison de certificat. Suivez les étapes fournies dans [Créer une liaison](configure-ssl-bindings.md#create-binding).
>

## <a name="import-a-certificate-from-key-vault"></a>Importer un certificat à partir de Key Vault

Si vous utilisez Azure Key Vault pour gérer vos certificats, vous pouvez importer un certificat PKCS12 à partir de Key Vault dans App Service tant qu’il [répond aux exigences](#private-certificate-requirements).

### <a name="authorize-app-service-to-read-from-the-vault"></a>Autoriser App Service à lire les données du coffre
Par défaut, le fournisseur de ressources App Service n’a pas accès au coffre de clés. Si vous souhaitez utiliser un coffre de clés pour un déploiement de certificats, vous devez [autoriser le fournisseur de ressources à accéder en lecture aux données du coffre de clés](../key-vault/general/assign-access-policy-cli.md). 

`abfa0a7c-a6b6-4736-8310-5855508787cd` est le nom du principal de service du fournisseur de ressources App Service, et il est identique pour tous les abonnements Azure. Pour un environnement cloud Azure Government, utilisez `6a02c803-dafd-4136-b4c3-5a6f318b4714` plutôt que le nom du principal de service du fournisseur de ressources.

### <a name="import-a-certificate-from-your-vault-to-your-app"></a>Importer un certificat dans votre application à partir de votre coffre

Sur le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, dans le menu de gauche, sélectionnez **App Services** >  **\<app-name>** .

Dans le panneau de navigation de gauche de votre application, sélectionnez **Paramètres TLS/SSL** > **Certificats à clé privée (.pfx)**  > **Importer un certificat Key Vault**.

![Importer un certificat Key Vault dans App Service](./media/configure-ssl-certificate/import-key-vault-cert.png)

Aidez-vous du tableau suivant pour sélectionner le certificat.

| Paramètre | Description |
|-|-|
| Abonnement | Abonnement auquel appartient le coffre de clés. |
| Key Vault | Coffre contenant le certificat que vous souhaitez importer. |
| Certificat | Dans le coffre, sélectionnez un certificat PKCS12 dans la liste. Tous les certificats PKCS12 du coffre sont listés avec leurs empreintes numériques, mais tous ne sont pas pris en charge dans App Service. |

Une fois l’opération terminée, le certificat s’affiche dans la liste **Certificats à clé privée**. Si l’importation échoue avec une erreur, c’est que le certificat ne répond pas aux [exigences App Service](#private-certificate-requirements).

![Importation du certificat Key Vault terminée](./media/configure-ssl-certificate/import-app-service-cert-finished.png)

> [!NOTE]
> Si vous remplacez votre certificat par un nouveau certificat dans le coffre de clés, App Service synchronisera automatiquement votre certificat sous 48 heures.

> [!IMPORTANT] 
> Pour sécuriser un domaine personnalisé avec ce certificat, vous devez quand même créer une liaison de certificat. Suivez les étapes fournies dans [Créer une liaison](configure-ssl-bindings.md#create-binding).
>

## <a name="upload-a-private-certificate"></a>Téléchargement d’un certificat privé

Une fois que vous avez obtenu un certificat de votre fournisseur de certificats, effectuez les étapes décrites dans cette section afin de préparer le certificat pour App Service.

### <a name="merge-intermediate-certificates"></a>Fusionner les certificats intermédiaires

Si votre autorité de certification vous donne plusieurs certificats dans la chaîne, vous devez les fusionner dans l’ordre.

Pour ce faire, ouvrez chaque certificat reçu dans un éditeur de texte.

Créez un fichier pour le certificat fusionné, appelé _mergedcertificate.crt_. Dans un éditeur de texte, copiez le contenu de chaque certificat dans ce fichier. L’ordre de vos certificats devrait suivre l’ordre dans la chaîne d’approbation, commençant par votre certificat et finissant par le certificat racine. Cela ressemble à l’exemple suivant :

```
-----BEGIN CERTIFICATE-----
<your entire Base64 encoded SSL certificate>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 1>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 2>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded root certificate>
-----END CERTIFICATE-----
```

### <a name="export-certificate-to-pfx"></a>Exportation du certificat vers PFX

Exportez votre certificat TLS/SSL fusionné avec la clé privée ayant servi à générer votre demande de certificat.

Si vous avez généré votre demande de certificat à l’aide d’OpenSSL, vous avez créé un fichier de clé privée. Pour exporter votre certificat au format PFX, exécutez la commande suivante : Remplacez les espaces réservés _&lt;fichier de clé privée>_ et _&lt;fichier de certificat fusionné>_ par les chemins d’accès menant à votre clé privée et à votre fichier de certificat fusionné.

```bash
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>  
```

Lorsque vous y êtes invité, définissez un mot de passe d’exportation. Vous allez utiliser ce mot de passe lors du chargement de votre certificat TLS/SSL dans App Service.

Si vous avez utilisé IIS ou _Certreq.exe_ pour générer votre demande de certificat, installez le certificat sur votre ordinateur local, puis [exportez le certificat au format PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).

### <a name="upload-certificate-to-app-service"></a>Charger le certificat dans App Service

Vous êtes maintenant prêt à charger le certificat dans App Service.

Sur le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, dans le menu de gauche, sélectionnez **App Services** >  **\<app-name>** .

Dans le panneau de navigation de gauche de votre application, sélectionnez **Paramètres TLS/SSL** > **Certificats à clé privée (.pfx)**  > **Charger le certificat**.

![Charger un certificat privé dans App Service](./media/configure-ssl-certificate/upload-private-cert.png)

Dans **Fichier de certificat PFX**, sélectionnez votre fichier PFX. Dans **Mot de passe du certificat**, tapez le mot de passe que vous avez créé lors de l’exportation du fichier PFX. Lorsque vous avez terminé, cliquez sur **Charger**. 

Une fois l’opération terminée, le certificat s’affiche dans la liste **Certificats à clé privée**.

![Chargement du certificat terminé](./media/configure-ssl-certificate/create-free-cert-finished.png)

> [!IMPORTANT] 
> Pour sécuriser un domaine personnalisé avec ce certificat, vous devez quand même créer une liaison de certificat. Suivez les étapes fournies dans [Créer une liaison](configure-ssl-bindings.md#create-binding).
>

## <a name="upload-a-public-certificate"></a>Téléchargement d’un certificat public

Les certificats publics sont pris en charge au format *.cer*. 

Sur le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, dans le menu de gauche, sélectionnez **App Services** >  **\<app-name>** .

Dans le panneau de navigation de gauche de votre application, cliquez sur **Paramètres TLS/SSL** > **Certificats publics (.cer)**  > **Charger un certificat à clé publique**.

Dans **Nom**, tapez un nom pour le certificat. Dans **Fichier de certificat CER**, sélectionnez votre fichier CER.

Cliquez sur **Télécharger**.

![Charger un certificat public dans App Service](./media/configure-ssl-certificate/upload-public-cert.png)

Une fois le certificat chargé, copiez l’empreinte du certificat, puis reportez-vous à la section [Rendre le certificat accessible](configure-ssl-certificate-in-code.md#make-the-certificate-accessible).

## <a name="manage-app-service-certificates"></a>Gérer les certificats App Service

Cette section vous montre comment gérer le certificat App Service que vous avez acheté dans la section [Importer un certificat App Service](#import-an-app-service-certificate).

- [Recréer la clé du certificat](#rekey-certificate)
- [Renouveler le certificat](#renew-certificate)
- [Exporter le certificat](#export-certificate)
- [Supprimer le certificat](#delete-certificate)

### <a name="rekey-certificate"></a>Recréer la clé du certificat

Si vous pensez que la clé privée de votre certificat est compromise, vous pouvez recréer la clé de votre certificat. Sélectionnez le certificat dans la page [App Service Certificates](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders), puis sélectionnez **Recréer la clé et synchroniser** dans le volet de navigation de gauche.

Cliquez sur le bouton **Recréer la clé** pour lancer le processus. Ce processus peut prendre de 1 à 10 minutes.

![Recréer la clé d’un certificat App Service](./media/configure-ssl-certificate/rekey-app-service-cert.png)

Le renouvellement de la clé de votre certificat remplace le certificat par un nouveau certificat émis par l’autorité de certification.

Une fois l’opération de recréation de clé terminée, cliquez sur **Synchronisation**. L’opération de synchronisation met à jour automatiquement les liaisons de nom d’hôte pour le certificat dans App Service sans perturber le fonctionnement de vos applications.

> [!NOTE]
> Si vous ne cliquez pas sur **Synchronisation**, App Service synchronise automatiquement votre certificat sous 48 heures.

### <a name="renew-certificate"></a>Renouvellement de certificat

Pour activer le renouvellement automatique de votre certificat à tout moment, sélectionnez le certificat dans la page [App Service Certificates](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders), puis cliquez sur **Paramètres de renouvellement automatique** dans le volet de navigation de gauche. Par défaut, les certificats App Service sont valides pendant 1 an.

Sélectionnez **Activé** et cliquez sur **Enregistrer**. Les certificats peuvent être automatiquement renouvelés 60 jours avant leur expiration si vous avez activé le renouvellement automatique.

![Renouveler automatiquement un certificat App Service](./media/configure-ssl-certificate/auto-renew-app-service-cert.png)

Pour renouveler manuellement le certificat, cliquez sur **Renouvellement manuel**. Vous pouvez demander le renouvellement manuellement de votre certificat 60 jours avant expiration.

Une fois l’opération de renouvellement terminée, cliquez sur **Synchronisation**. L’opération de synchronisation met à jour automatiquement les liaisons de nom d’hôte pour le certificat dans App Service sans perturber le fonctionnement de vos applications.

> [!NOTE]
> Si vous ne cliquez pas sur **Synchronisation**, App Service synchronise automatiquement votre certificat sous 48 heures.

### <a name="export-certificate"></a>Exportation du certificat

Étant donné qu’un certificat App Service est un [secret Key Vault](../key-vault/general/about-keys-secrets-certificates.md), vous pouvez exporter une copie de ce fichier PFX et l’utiliser pour d’autres services Azure ou en dehors d’Azure.

Pour exporter le certificat App Service dans un fichier PFX, exécutez les commandes suivantes dans [Cloud Shell](https://shell.azure.com). Vous pouvez également l’exécuter localement [si vous avez installé Azure CLI](/cli/azure/install-azure-cli). Remplacez les espaces réservés par les noms que vous avez utilisés [lorsque vous avez créé le certificat App Service](#start-certificate-order).

```azurecli-interactive
secretname=$(az resource show \
    --resource-group <group-name> \
    --resource-type "Microsoft.CertificateRegistration/certificateOrders" \
    --name <app-service-cert-name> \
    --query "properties.certificates.<app-service-cert-name>.keyVaultSecretName" \
    --output tsv)

az keyvault secret download \
    --file appservicecertificate.pfx \
    --vault-name <key-vault-name> \
    --name $secretname \
    --encoding base64
```

Le fichier *appservicecertificate.pfx* téléchargé est un fichier PKCS12 brut qui contient à la fois des certificats publics et des certificats privés. Dans chaque invite, utilisez une chaîne vide pour le mot de passe d’importation et la phrase secrète PEM.

### <a name="delete-certificate"></a>Suppression d'un certificat 

La suppression d’un certificat App Service est définitive et irréversible. La suppression d’une ressource App Service Certificate entraîne la révocation du certificat. Dans App Service, les liaisons vers ce certificat deviennent alors non valides. Pour éviter toute suppression accidentelle, Azure place un verrou sur le certificat. Pour supprimer un certificat App Service, vous devez d’abord supprimer son verrou de suppression.

Sélectionnez le certificat dans la page [App Service Certificates](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders), puis sélectionnez **Verrous** dans le volet de navigation de gauche.

Dans votre certificat, recherchez le verrou de type **Supprimer**. À droite de celui-ci, sélectionnez **Supprimer**.

![Verrou de suppression du certificat App Service](./media/configure-ssl-certificate/delete-lock-app-service-cert.png)

Vous pouvez maintenant supprimer le certificat App Service. Dans le volet de navigation de gauche, sélectionnez **Vue d’ensemble** > **Supprimer**. Dans la boîte de dialogue de confirmation, tapez le nom du certificat, puis sélectionnez **OK**.

## <a name="automate-with-scripts"></a>Automatiser des tâches à l’aide de scripts

### <a name="azure-cli"></a>Azure CLI

[!code-azurecli[main](../../cli_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.sh?highlight=3-5 "Bind a custom TLS/SSL certificate to a web app")] 

### <a name="powershell"></a>PowerShell

[!code-powershell[main](../../powershell_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.ps1?highlight=1-3 "Bind a custom TLS/SSL certificate to a web app")]

## <a name="more-resources"></a>Plus de ressources

* [Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service](configure-ssl-bindings.md)
* [Appliquer le protocole HTTPS](configure-ssl-bindings.md#enforce-https)
* [Appliquer le protocole TLS 1.1/1.2](configure-ssl-bindings.md#enforce-tls-versions)
* [Utiliser un certificat TLS/SSL dans votre code dans Azure App Service](configure-ssl-certificate-in-code.md)
* [FORUM AUX QUESTIONS : App Service Certificates](./faq-configuration-and-management.md)
