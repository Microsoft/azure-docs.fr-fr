---
title: Configurer TLS pour un service cloud | Microsoft Docs
description: Découvrez comment spécifier un point de terminaison HTTPS pour un rôle Web et télécharger un certificat TLS/SSL pour sécuriser votre application. Ces exemples utilisent le portail Azure.
services: cloud-services
documentationcenter: .net
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 05/26/2017
ms.author: tagore
ms.openlocfilehash: c69b74cf91d8e097f8ad8a9ba2a16f3375f483ae
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82024844"
---
# <a name="configuring-tls-for-an-application-in-azure"></a>Configuration de TLS pour une application dans Azure

Le chiffrement Transport Layer Security (TLS), précédemment connu sous le nom Secure Socket Layer (SSL), est la méthode de sécurisation la plus couramment utilisée pour envoyer des données sécurisées sur Internet. Cette tâche présente la spécification d’un point de terminaison HTTPS pour un rôle Web et le téléchargement d’un certificat TLS/SSL pour sécuriser votre application.

> [!NOTE]
> Les procédures décrites dans cette tâche s’appliquent à Azure Cloud Services ; pour App Services, consultez [cette page](../app-service/configure-ssl-bindings.md).
>

Cette tâche utilise un déploiement de production. Vous trouverez des informations sur l’utilisation d’un déploiement intermédiaire à la fin de cette rubrique.

Lisez tout d’abord [ceci](cloud-services-how-to-create-deploy-portal.md) si vous n’avez pas encore créé de service cloud.

## <a name="step-1-get-a-tlsssl-certificate"></a>Étape 1 : Obtenir un certificat TLS/SSL
Pour configurer le chiffrement TLS pour une application, vous devez d’abord obtenir un certificat TLS/SSL signé par une autorité de certification, un tiers approuvé qui émet des certificats à cet effet. Si vous n’en possédez pas, vous devez en obtenir un auprès de la société qui vend des certificats TLS/SSL.

Le certificat TLS/SSL doit répondre aux prérequis suivants dans Azure :

* Le certificat doit contenir une clé publique.
* Le certificat doit être créé pour l'échange de clés et pouvoir faire l'objet d'un export au format Personal Information Exchange (.pfx).
* Le nom d'objet du certificat doit correspondre au domaine servant à accéder au service cloud. Vous ne pouvez pas obtenir de certificat TLS/SSL d’une autorité de certification pour le domaine cloudapp.net. Vous devez acquérir un nom de domaine personnalisé à utiliser pour accéder à votre service. Lorsque vous demandez un certificat auprès d’une autorité de certification, le nom d’objet du certificat doit correspondre au nom de domaine personnalisé que vous utilisez pour accéder à votre application. Par exemple, si votre nom de domaine personnalisé est **contoso.com**, vous demandez un certificat auprès de votre autorité de certification pour * **.contoso.com** ou **www\.contoso.com**.
* Le certificat doit utiliser au minimum un chiffrement à 2048 bits.

Dans le cadre d’un test, vous pouvez [créer](cloud-services-certs-create.md) et utiliser un certificat auto-signé. Un certificat auto-signé n'est pas authentifié par une autorité de certification et peut utiliser le domaine cloudapp.net comme URL de site Web. Par exemple, la tâche ci-dessous utilise un certificat auto-signé dans lequel le nom commun utilisé dans le certificat est **sslexample.cloudapp.net**.

Ensuite, vous devez ajouter des informations sur le certificat dans votre définition de service et dans les fichiers de configuration de service.

<a name="modify"> </a>

## <a name="step-2-modify-the-service-definition-and-configuration-files"></a>Étape 2 : modification des fichiers de définition de service et de configuration
Votre application doit être configurée pour utiliser le certificat, et un point de terminaison HTTPS doit être ajouté. Suite à cette opération, les fichiers de définition de service et de configuration de service doivent être mis à jour.

1. Dans votre environnement de développement, ouvrez le fichier de définition du service (CSDEF), ajoutez une section **Certificates** dans la section **WebRole**, puis ajoutez les informations qui suivent sur le certificat (et les certificats intermédiaires) :

   ```xml
    <WebRole name="CertificateTesting" vmsize="Small">
    ...
        <Certificates>
            <Certificate name="SampleCertificate"
                        storeLocation="LocalMachine"
                        storeName="My"
                        permissionLevel="limitedOrElevated" />
            <!-- IMPORTANT! Unless your certificate is either
            self-signed or signed directly by the CA root, you
            must include all the intermediate certificates
            here. You must list them here, even if they are
            not bound to any endpoints. Failing to list any of
            the intermediate certificates may cause hard-to-reproduce
            interoperability problems on some clients.-->
            <Certificate name="CAForSampleCertificate"
                        storeLocation="LocalMachine"
                        storeName="CA"
                        permissionLevel="limitedOrElevated" />
        </Certificates>
    ...
    </WebRole>
    ```

   La section **Certificates** définit le nom du certificat, son emplacement et le nom du magasin dans lequel il se trouve.

   Les autorisations (attribut `permissionLevel`) peuvent être définies sur les valeurs suivantes :

   | Valeur de l’autorisation | Description |
   | --- | --- |
   | limitedOrElevated |**(Par défaut)** Tous les processus de rôle peuvent accéder à la clé privée. |
   | elevated |Seuls les processus élevés peuvent accéder à la clé privée. |

2. Dans votre fichier de définition du service, ajoutez un élément **InputEndpoint** dans la section **Endpoints** pour activer HTTPS :

   ```xml
    <WebRole name="CertificateTesting" vmsize="Small">
    ...
        <Endpoints>
            <InputEndpoint name="HttpsIn" protocol="https" port="443"
                certificate="SampleCertificate" />
        </Endpoints>
    ...
    </WebRole>
    ```

3. Dans votre fichier de définition du service, ajoutez un élément **Binding** dans la section **Sites**. Cet élément ajoute une liaison HTTPS pour mapper le point de terminaison à votre site :

   ```xml
    <WebRole name="CertificateTesting" vmsize="Small">
    ...
        <Sites>
            <Site name="Web">
                <Bindings>
                    <Binding name="HttpsIn" endpointName="HttpsIn" />
                </Bindings>
            </Site>
        </Sites>
    ...
    </WebRole>
    ```

   Toutes les modifications nécessaires ont été apportées au fichier de définition de service, mais vous devez encore y ajouter les informations de certificat.
4. Dans votre fichier de configuration de service (CSCFG), ServiceConfiguration.Cloud.cscfg, ajoutez la valeur de votre certificat dans **Certificats**. L’exemple de code suivant fournit des détails sur la section **Certificats**, à l’exception de la valeur de l’empreinte numérique.

   ```xml
    <Role name="Deployment">
    ...
        <Certificates>
            <Certificate name="SampleCertificate"
                thumbprint="9427befa18ec6865a9ebdc79d4c38de50e6316ff"
                thumbprintAlgorithm="sha1" />
            <Certificate name="CAForSampleCertificate"
                thumbprint="79d4c38de50e6316ff9427befa18ec6865a9ebdc"
                thumbprintAlgorithm="sha1" />
        </Certificates>
    ...
    </Role>
    ```

(Cet exemple utilise **sha1** comme algorithme d’empreinte numérique. Spécifiez la valeur correspondant à l'algorithme d'empreinte numérique de votre certificat.)

Maintenant que les fichiers de définition du service et de configuration de service ont été mis à jour, créez un package pour votre déploiement afin de le télécharger dans Azure. Si vous utilisez **cspack**, n’utilisez pas l’indicateur **/generateConfigurationFile**, car les informations de certificat que vous venez juste d’entrer seraient écrasées.

## <a name="step-3-upload-a-certificate"></a>Étape 3 : Téléchargement d'un certificat
Connectez-vous au portail Azure et...

1. Dans la section **Toutes les ressources** du portail, sélectionnez votre service cloud.

    ![Publier votre service cloud](media/cloud-services-configure-ssl-certificate-portal/browse.png)

2. Cliquez sur **Certificats**.

    ![Cliquer sur l’icône Certificats](media/cloud-services-configure-ssl-certificate-portal/certificate-item.png)

3. Cliquez sur **Charger** en haut de la zone de certificats.

    ![Cliquez sur l’élément de menu de chargement](media/cloud-services-configure-ssl-certificate-portal/Upload_menu.png)

4. Fournissez le **fichier**, entrez le **mot de passe**, puis cliquez sur **Charger** en bas de la zone de saisie de données.

## <a name="step-4-connect-to-the-role-instance-by-using-https"></a>Étape 4 : connexion à l’instance de rôle à l’aide de HTTPS
Maintenant que votre déploiement est opérationnel dans Azure, vous pouvez vous y connecter via HTTPS.

1. Cliquez sur **l’URL du site** pour ouvrir le navigateur web.

   ![Cliquez sur l'URL du site](media/cloud-services-configure-ssl-certificate-portal/navigate.png)

2. Dans votre navigateur Web, modifiez le lien pour utiliser **HTTPS** au lieu de **HTTP**, puis accédez à la page.

   > [!NOTE]
   > Si vous utilisez un certificat auto-signé, lorsque vous accédez à un point de terminaison HTTPS qui lui est associé, vous pourriez obtenir une erreur de certificat dans le navigateur. Pour remédier à ce problème, utilisez un certificat signé par une autorité de certification approuvée. En attendant, vous pouvez ignorer cette erreur. (Une autre possibilité est d'ajouter le certificat auto-signé au magasin de certificats d'autorité de certification approuvé de l'utilisateur.)
   >
   >

   ![Aperçu du site](media/cloud-services-configure-ssl-certificate-portal/show-site.png)

   > [!TIP]
   > Si vous voulez utiliser TLS pour un déploiement intermédiaire au lieu d’un déploiement de production, vous devez d’abord déterminer l’URL utilisée pour le déploiement intermédiaire. Une fois le service cloud déployé, l’URL de l’environnement intermédiaire est déterminée par le GUID **ID de déploiement** au format suivant : `https://deployment-id.cloudapp.net/`  
   >
   > Créez un certificat avec le nom commun (CN) similaire à l’URL basée sur GUID (par exemple, **328187776e774ceda8fc57609d404462.cloudapp.net**). Utilisez le portail pour ajouter le certificat à votre service cloud intermédiaire. Ensuite, ajoutez les informations du certificat à vos fichiers CSDEF et CSCFG, recréez le package de votre application et mettez à jour votre déploiement intermédiaire pour utiliser le nouveau package.
   >

## <a name="next-steps"></a>Étapes suivantes
* [Configuration générale de votre service cloud](cloud-services-how-to-configure-portal.md).
* Découvrez comment [déployer un service cloud](cloud-services-how-to-create-deploy-portal.md).
* Configurez un [nom de domaine personnalisé](cloud-services-custom-domain-name-portal.md).
* [Gérez votre service cloud](cloud-services-how-to-manage-portal.md).



