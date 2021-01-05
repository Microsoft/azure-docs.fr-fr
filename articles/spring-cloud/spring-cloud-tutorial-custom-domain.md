---
title: Tutoriel – Mapper un domaine personnalisé existant à Azure Spring Cloud
description: Comment mapper un nom DNS (Distributed Name Service) personnalisé existant à Azure Spring Cloud
author: MikeDodaro
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 03/19/2020
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: d06a6eb8b504f2c5dd09de70d79f50a3ed5d89a3
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844725"
---
# <a name="map-an-existing-custom-domain-to-azure-spring-cloud"></a>Mapper un domaine personnalisé existant à Azure Spring Cloud

**Cet article s’applique à :** ✔️ Java ✔️ C#

DNS (Domain Name Service) est une technique de stockage des noms de nœuds réseau à l’échelle d’un réseau. Ce tutoriel mappe un domaine, tel que www.contoso.com, à l’aide d’un enregistrement CNAME. Il sécurise le domaine personnalisé avec un certificat et montre comment mettre en œuvre le protocole TLS (Transport Layer Security), également connu sous le nom de protocole SSL (Secure Sockets Layer). 

Les certificats chiffrent le trafic web. Ces certificats TLS/SSL peuvent être stockés dans Azure Key Vault. 

## <a name="prerequisites"></a>Prérequis
* Une application déployée dans Azure Spring Cloud (consultez [Démarrage rapide : Lancer une application Azure Spring Cloud existante à partir du portail Azure](spring-cloud-quickstart.md) ou utilisez une application existante).
* Un nom de domaine avec un accès au registre DNS pour le fournisseur de domaine, par exemple GoDaddy.
* Un certificat privé (autrement dit, votre certificat auto-signé) d’un fournisseur tiers. Le certificat doit correspondre au domaine.
* Une instance déployée d’[Azure Key Vault](../key-vault/general/overview.md).

## <a name="import-certificate"></a>Importation d’un certificat
### <a name="prepare-your-certificate-file-in-pfx-optional"></a>Préparer votre fichier de certificat en PFX (facultatif)
Azure Key Vault prend en charge l’importation d’un certificat privé au format PEM et PFX. Si le fichier PEM que vous avez obtenu de votre fournisseur de certificats ne fonctionne pas dans la section ci-dessous : [Enregistrez le certificat dans Key Vault](#save-certificate-in-key-vault), suivez les étapes indiquées ici pour générer un fichier PFX pour Azure Key Vault.

#### <a name="merge-intermediate-certificates"></a>Fusionner les certificats intermédiaires

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

#### <a name="export-certificate-to-pfx"></a>Exportation du certificat vers PFX

Exportez votre certificat TLS/SSL fusionné avec la clé privée ayant servi à générer votre demande de certificat.

Si vous avez généré votre demande de certificat à l’aide d’OpenSSL, vous avez créé un fichier de clé privée. Pour exporter votre certificat au format PFX, exécutez la commande suivante : Remplacez les espaces réservés _&lt;fichier de clé privée>_ et _&lt;fichier de certificat fusionné>_ par les chemins d’accès menant à votre clé privée et à votre fichier de certificat fusionné.

```bash
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Lorsque vous y êtes invité, définissez un mot de passe d’exportation. Vous utiliserez par la suite ce mot de passe au moment de charger votre certificat TLS/SSL dans Azure Key Vault.

Si vous avez utilisé IIS ou _Certreq.exe_ pour générer votre demande de certificat, installez le certificat sur votre ordinateur local, puis [exportez le certificat au format PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).

### <a name="save-certificate-in-key-vault"></a>Enregistrer le certificat dans Key Vault
La procédure d’importation d’un certificat exige la présence du fichier PEM ou PFX encodé sur le disque, et vous devez disposer de la clé privée. 
#### <a name="portal"></a>[Portail](#tab/Azure-portal)
Pour charger votre certificat dans le coffre de clés :
1. Accédez à votre instance de coffre de clés.
1. Dans le volet de navigation gauche, cliquez sur **Certificats**.
1. Dans le menu du haut, cliquez sur **Générer/importer**.
1. Dans la boîte de dialogue **Créer un certificat**, en dessous de **Méthode de création de certificat**, sélectionnez `Import`.
1. Sous **Charger le fichier de certificat**, accédez à emplacement du certificat et sélectionnez-le.
1. Sous **Mot de passe**, entrez la clé privée de votre certificat.
1. Cliquez sur **Créer**.

    ![Importation du certificat 1](./media/custom-dns-tutorial/import-certificate-a.png)

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/Azure-CLI)

```azurecli
az keyvault certificate import --file <path to .pfx file> --name <certificate name> --vault-name <key vault name> --password <export password>
```
---

### <a name="grant-azure-spring-cloud-access-to-your-key-vault"></a>Accorder à Azure Spring Cloud un accès à votre coffre de clés

Vous devez accorder à Azure Spring Cloud un accès à votre coffre de clés avant d’importer le certificat :
#### <a name="portal"></a>[Portail](#tab/Azure-portal)
1. Accédez à votre instance de coffre de clés.
1. Dans le volet de navigation gauche, cliquez sur **Stratégies d’accès**.
1. Dans le menu supérieur, cliquez sur **Ajouter une stratégie d’accès**.
1. Renseignez les informations, cliquez sur le bouton **Ajouter**, puis **enregistrez** la stratégie d’accès.

| Autorisation de secret | Autorisation de certificat | Sélectionner le principal |
|--|--|--|
| Obtenir, Lister | Obtenir, Lister | Azure Spring Cloud Domain-Management |

![Importation du certificat 2](./media/custom-dns-tutorial/import-certificate-b.png)

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/Azure-CLI)

À l’aide de la commande suivante, accordez à Azure Spring Cloud un accès en lecture au coffre de clés (remplacez `<key vault resource group>` et `<key vault name>`).
```
az keyvault set-policy -g <key vault resource group> -n <key vault name>  --object-id 938df8e2-2b9d-40b1-940c-c75c33494239 --certificate-permissions get list --secret-permissions get list
``` 
---

### <a name="import-certificate-to-azure-spring-cloud"></a>Importer le certificat dans Azure Spring Cloud
#### <a name="portal"></a>[Portail](#tab/Azure-portal)
1. Accédez à votre instance de service. 
1. Dans le volet de navigation gauche de votre application, sélectionnez **Paramètres TLS/SSL**.
1. Cliquez ensuite sur **Importer un certificat Key Vault**.

    ![Importation d’un certificat](./media/custom-dns-tutorial/import-certificate.png)

1. Une fois votre certificat importé, il apparaît dans la liste des **certificats à clé privé**.

    ![Certificat à clé privée](./media/custom-dns-tutorial/key-certificates.png)

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/Azure-CLI)

```
az spring-cloud certificate add --name <cert name> --vault-uri <key vault uri> --vault-certificate-name <key vault cert name>
```

Pour afficher la liste des certificats importés :

```
az spring-cloud certificate list --resource-group <resource group name> --service <service name>
```
---

> [!IMPORTANT] 
> Pour sécuriser un domaine personnalisé avec ce certificat, vous devez quand même lier le certificat à un domaine spécifique. Suivez les étapes décrites dans cette section : [Ajouter une liaison SSL](#add-ssl-binding).

## <a name="add-custom-domain"></a>Ajouter un domaine personnalisé
Vous pouvez utiliser un enregistrement CNAME pour mapper un nom DNS personnalisé à Azure Spring Cloud. 

> [!NOTE] 
> L’enregistrement A n’est pas pris en charge. 

### <a name="create-the-cname-record"></a>Créer un enregistrement CNAME
Accédez à votre fournisseur DNS et ajoutez un enregistrement CNAME pour mapper votre domaine à <nom_service>.azuremicroservices.io. Ici <nom_service> est le nom de votre instance Azure Spring Cloud. Nous prenons en charge les domaines et sous-domaines génériques. Après avoir ajouté l’enregistrement CNAME, la page d’enregistrements DNS ressemble à l’exemple suivant : 

![Page Enregistrements DNS](./media/custom-dns-tutorial/dns-records.png)

## <a name="map-your-custom-domain-to-azure-spring-cloud-app"></a>Mapper votre domaine personnalisé à l’application Azure Spring Cloud
Si vous n’avez pas d’application dans Azure Spring Cloud, suivez les instructions du [Démarrage rapide : Lancer une application Azure Spring Cloud existante à partir du portail Azure](https://review.docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart-launch-app-portal?branch=master).

#### <a name="portal"></a>[Portail](#tab/Azure-portal)
Accédez à la page de l’application.

1. Sélectionner **Domaine personnalisé**.
2. Ensuite, choisissez **Ajouter un domaine personnalisé**. 

    ![Domaine personnalisé](./media/custom-dns-tutorial/custom-domain.png)

3. Tapez le nom de domaine complet pour lequel vous avez ajouté un enregistrement CNAME, tel que www.contoso.com. Vérifiez que le type d’enregistrement de nom d’hôte défini est CNAME (<nom_service>.azuremicroservices.io).
4. Cliquez sur **Valider** pour activer le bouton **Ajouter**.
5. Cliquez sur **Add**.

    ![Ajouter un domaine personnalisé](./media/custom-dns-tutorial/add-custom-domain.png)

Une application peut avoir plusieurs domaines, mais seul un domaine peut être mappé à une même application. Une fois que vous avez mappé votre domaine personnalisé à l’application, il apparaît dans le tableau du domaine personnalisé.

![Tableau du domaine personnalisé](./media/custom-dns-tutorial/custom-domain-table.png)

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/Azure-CLI)
```
az spring-cloud app custom-domain bind --domain-name <domain name> --app <app name> --resource-group <resource group name> --service <service name>
```

Pour afficher la liste des domaines personnalisés :
```
az spring-cloud app custom-domain list --app <app name> --resource-group <resource group name> --service <service name>
```
---

> [!NOTE]
> Si votre domaine personnalisé présente le libellé **Non sécurisé**, cela signifie que n’est pas encore lié à un certificat SSL. Toute requête HTTPS d’un navigateur à votre domaine personnalisé générera une erreur ou un avertissement.

## <a name="add-ssl-binding"></a>Ajouter une liaison SSL

#### <a name="portal"></a>[Portail](#tab/Azure-portal)
Dans le tableau du domaine personnalisé, sélectionnez **Ajouter une liaison SSL** comme illustré dans la figure précédente.  
1. Sélectionnez votre **Certificat** ou importez-le.
1. Cliquez sur **Enregistrer**.

    ![Ajout d’une liaison SSL 1](./media/custom-dns-tutorial/add-ssl-binding.png)

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/Azure-CLI)
```
az spring-cloud app custom-domain update --domain-name <domain name> --certificate <cert name> --app <app name> --resource-group <resource group name> --service <service name>
```
---

Une fois la liaison SSL ajoutée, le domaine passe à un état sécurisé : **Intègre**. 

![Ajout d’une liaison SSL 2](./media/custom-dns-tutorial/secured-domain-state.png)

## <a name="enforce-https"></a>Appliquer le protocole HTTPS
Par défaut, n’importe qui peut accéder à votre application en HTTP, mais vous pouvez rediriger toutes les requêtes HTTP vers le port HTTPS.
#### <a name="portal"></a>[Portail](#tab/Azure-portal)
Dans la page de votre application, dans le volet de navigation gauche, sélectionnez **Domaine personnalisé**. Ensuite, définissez **HTTPS uniquement** sur *Vrai*.

![Ajout d’une liaison SSL 3](./media/custom-dns-tutorial/enforce-http.png)

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/Azure-CLI)
```
az spring-cloud app update -n <app name> --resource-group <resource group name> --service <service name> --https-only
```
---
Une fois l’opération terminée, accédez à une des URL HTTPS qui pointent vers votre application. Notez que les URL HTTP ne fonctionnent pas.

## <a name="see-also"></a>Voir aussi
* [Qu’est-ce qu’Azure Key Vault ?](../key-vault/general/overview.md)
* [Importer un certificat](../key-vault/certificates/certificate-scenarios.md#import-a-certificate)
* [Lancer votre application Spring Cloud en utilisant Azure CLI](./spring-cloud-quickstart.md)
