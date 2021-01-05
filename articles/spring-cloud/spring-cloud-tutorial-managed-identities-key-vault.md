---
title: Tutoriel - Identité managée pour connecter Key Vault à une application Azure Spring Cloud
description: Configurer une identité managée pour connecter Key Vault à une application Azure Spring Cloud
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 07/08/2020
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: fc44dd6cf91d687f47afadf1c3378956d838bc9d
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94579502"
---
# <a name="tutorial-use-a-managed-identity-to-connect-key-vault-to-an-azure-spring-cloud-app"></a>Tutoriel : Utiliser une identité managée pour connecter Key Vault à une application Azure Spring Cloud

**Cet article s’applique à :** ✔️ Java

Cet article montre comment créer une identité managée pour une application Azure Spring Cloud et comment l’utiliser pour accéder à Azure Key Vault.

Azure Key Vault peut être utilisé pour stocker de manière sécurisée et contrôler étroitement l’accès aux jetons, mots de passe, certificats, clés API et autres secrets pour votre application. Vous pouvez utiliser une identité managée dans Azure Active Directory (AAD) et vous authentifier auprès de n’importe quel service prenant en charge l’authentification AAD, notamment Key Vault, sans avoir à afficher les informations d’identification dans votre code.

## <a name="prerequisites"></a>Prérequis

* [Souscrire à un abonnement Azure](https://azure.microsoft.com/free/)
* [Installez Azure CLI 2.0.67 ou version ultérieure](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest)
* [Installez Maven 3.0 ou ultérieur](https://maven.apache.org/download.cgi)

## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Créez un groupe de ressources pour contenir à la fois le coffre de clés et Spring Cloud à l’aide de la commande [az group create](/cli/azure/group?view=azure-cli-latest&preserve-view=true#az-group-create) :

```azurecli-interactive
az group create --name "myResourceGroup" -l "EastUS"
```

## <a name="set-up-your-key-vault"></a>Configurer votre coffre de clés
Pour créer un coffre de clés, utilisez la commande [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest&preserve-view=true#az-keyvault-create):

> [!Important]
> Chaque coffre de clés doit avoir un nom unique. Remplacez <your-keyvault-name> par le nom de votre coffre de clés dans les exemples suivants.

```azurecli-interactive
az keyvault create --name "<your-keyvault-name>" -g "myResourceGroup"
```

Notez la valeur `vaultUri` retournée, qui est au format « https://<your-keyvault-name>.vault.azure.net ». Vous en aurez besoin à la prochaine étape.

Vous pouvez maintenant placer un secret dans votre coffre de clés à l’aide de la commande [az keyvault secret set](/cli/azure/keyvault/secret?view=azure-cli-latest&preserve-view=true#az-keyvault-secret-set) :

```azurecli-interactive
az keyvault secret set --vault-name "<your-keyvault-name>" \
    --name "connectionString" \
    --value "jdbc:sqlserver://SERVER.database.windows.net:1433;database=DATABASE;"
```

## <a name="create-azure-spring-cloud-service-and-app"></a>Créer un service et une application Azure Spring Cloud
Après l’installation de l’extension correspondante, créez une instance Azure Spring Cloud avec la commande Azure CLI `az spring-cloud create`. 

```azurecli-interactive
az extension add --name spring-cloud
az spring-cloud create -n "myspringcloud" -g "myResourceGroup"
```
L’exemple suivant crée une application nommée `springapp` avec une identité managée affectée par le système, comme le demande le paramètre `--assign-identity`.

```azurecli
az spring-cloud app create -n "springapp" -s "myspringcloud" -g "myResourceGroup" --is-public true --assign-identity
export SERVICE_IDENTITY=$(az spring-cloud app show --name "springapp" -s "myspringcloud" -g "myResourceGroup" | jq -r '.identity.principalId')
```

Notez la valeur `url` retournée, qui est au format « https://<your-app-name>.azuremicroservices.io ». Vous en aurez besoin à la prochaine étape.


## <a name="grant-your-app-access-to-key-vault"></a>Autoriser votre application à accéder à Key Vault
Utilisez `az keyvault set-policy` pour accorder l’accès approprié dans Key Vault pour votre application.
```azurecli
az keyvault set-policy --name "<your-keyvault-name>" --object-id ${SERVICE_IDENTITY} --secret-permissions set get list
```
> [!NOTE]
> Utilisez `az keyvault delete-policy --name "<your-keyvault-name>" --object-id ${SERVICE_IDENTITY}` pour supprimer l’accès pour votre application après la désactivation de l’identité managée affectée par le système.

## <a name="build-a-sample-spring-boot-app-with-spring-boot-starter"></a>Créer un exemple d’application Spring Boot avec Spring Boot Starter
Cette application pourra obtenir les secrets d’Azure Key Vault. Utilisez l’application Starter : [Azure Key Vault Secrets Spring Boot Starter](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter-keyvault-secrets).  Azure Key Vault est ajoutée en tant qu’instance de Spring **PropertySource**.  Les secrets stockés dans Azure Key Vault sont facilement accessibles et peuvent être utilisés au même titre que les propriétés de configuration externalisées, comme les propriétés dans les fichiers. 

1. Générez un exemple de projet à partir de start.spring.io avec Azure Key Vault Spring Starter. 
    ```azurecli
    curl https://start.spring.io/starter.tgz -d dependencies=web,azure-keyvault-secrets -d baseDir=springapp -d bootVersion=2.3.1.RELEASE -d javaVersion=1.8 | tar -xzvf -
    ```

2. Spécifiez votre coffre de clés dans votre application. 

    ```azurecli
    cd springapp
    vim src/main/resources/application.properties
    ```

    Pour utiliser une identité managée pour les applications Azure Spring Cloud, ajoutez des propriétés avec le contenu ci-dessous à src/main/resources/application.properties.

    ```
    azure.keyvault.enabled=true
    azure.keyvault.uri=https://<your-keyvault-name>.vault.azure.net
    ```
    > [!Note] 
    > Vous devez ajouter l’URL du coffre de clés dans `application.properties`, comme ci-dessus. Dans le cas contraire, l’URL du coffre de clés peut ne pas être capturée pendant le runtime.

3. Ajoutez l’exemple de code à src/main/java/com/example/demo/DemoApplication.java. Il récupère la chaîne de connexion à partir du coffre de clés. 

    ```Java
    package com.example.demo;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;

    @SpringBootApplication
    @RestController
    public class DemoApplication implements CommandLineRunner {

        @Value("${connectionString}")
        private String connectionString;

        public static void main(String[] args) {
          SpringApplication.run(DemoApplication.class, args);
        }

        @GetMapping("get")
        public String get() {
          return connectionString;
        }

        public void run(String... varl) throws Exception {
          System.out.println(String.format("\nConnection String stored in Azure Key Vault:\n%s\n",connectionString));
        }
      }
    ```

    Si vous ouvrez pom.xml,vous voyez la dépendance de `azure-keyvault-secrets-spring-boot-starter`. Ajoutez cette dépendance à votre projet dans pom.xml. 

    ```xml
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-keyvault-secrets-spring-boot-starter</artifactId>
    </dependency>
    ```

4. Empaquetez votre exemple d’application. 

    ```azurecli
    mvn clean package
    ```

5. Vous pouvez maintenant déployer votre application sur Azure avec la commande Azure CLI `az spring-cloud app deploy`. 

    ```azurecli
    az spring-cloud app deploy -n "springapp" -s "myspringcloud" -g "myResourceGroup" --jar-path target/demo-0.0.1-SNAPSHOT.jar
    ```

6.  Pour tester votre application, accédez au point de terminaison public ou au point de terminaison de test.

    ```azurecli
    curl https://myspringcloud-springapp.azuremicroservices.io/get
    ```

    Vous obtenez le message suivant : « Successfully got the value of secret connectionString from Key Vault https://<your-keyvault-name>.vault.azure.net/: jdbc:sqlserver://SERVER.database.windows.net:1433;database=DATABASE; ».

## <a name="build-sample-spring-boot-app-with-java-sdk"></a>Créer un exemple d’application Spring Boot avec le SDK Java

Cet exemple peut définir et obtenir des secrets à partir d’Azure Key Vault. La [bibliothèque de client Azure Key Vault Secret pour Java](/java/api/overview/azure/security-keyvault-secrets-readme?preserve-view=true&view=azure-java-stablelibrary) prend en charge l’authentification par jeton Azure Active Directory dans le SDK Azure. Elle fournit un ensemble d’implémentations de **TokenCredential** qui peuvent être utilisées pour construire des clients du SDK Azure afin de prendre en charge l’authentification par jeton AAD.

La bibliothèque de client Azure Key Vault Secret vous permet de stocker et de contrôler de manière sécurisée l’accès aux jetons, mots de passe, clés API et autres secrets. La bibliothèque propose des opérations pour créer, récupérer, mettre à jour, supprimer, vider, sauvegarder, restaurer et lister les secrets et leurs versions.

1. Clonez un exemple de projet. 

    ```azurecli
    git clone https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples.git
    ```

2. Spécifiez votre coffre de clés dans votre application. 

    ```azurecli
    cd Azure-Spring-Cloud-Samples/managed-identity-keyvault
    vim src/main/resources/application.properties
    ```

    Pour utiliser une identité managée pour les applications Azure Spring Cloud, ajoutez des propriétés avec le contenu suivant à *src/main/resources/application.properties*.

    ```
    azure.keyvault.enabled=true
    azure.keyvault.uri=https://<your-keyvault-name>.vault.azure.net
    ```

3. Incluez [ManagedIdentityCredentialBuilder](/java/api/com.azure.identity.managedidentitycredentialbuilder?preserve-view=true&view=azure-java-stable) pour récupérer un jeton à partir d’Azure Active Directory et [SecretClientBuilder](/java/api/com.azure.security.keyvault.secrets.secretclientbuilder?preserve-view=true&view=azure-java-stable) pour définir ou obtenir des secrets de Key Vault dans votre code.

    Procurez-vous l’exemple dans [MainController.java](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/blob/master/managed-identity-keyvault/src/main/java/com/microsoft/azure/MainController.java#L28) de l’exemple de projet cloné.

    Incluez également `azure-identity` et `azure-security-keyvault-secrets` comme dépendance dans votre fichier pom.xml. Procurez-vous l’exemple dans [pom.xml](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/blob/master/managed-identity-keyvault/pom.xml#L21) de l’exemple de projet cloné. 

4. Empaquetez votre exemple d’application. 

    ```azurecli
    mvn clean package
    ```

5. Déployez à présent l’application sur Azure avec la commande Azure CLI `az spring-cloud app deploy`. 

    ```azurecli
    az spring-cloud app deploy -n "springapp" -s "myspringcloud" -g "myResourceGroup" --jar-path target/asc-managed-identity-keyvault-sample-0.1.0.jar
    ```

6. Pour tester votre application, accédez au point de terminaison public ou au point de terminaison de test. 

    Premièrement, récupérez la valeur de la clé secrète que vous avez définie dans Azure Key Vault. 
    ```azurecli
    curl https://myspringcloud-springapp.azuremicroservices.io/secrets/connectionString
    ```

    Vous obtenez le message suivant : « Successfully got the value of secret connectionString from Key Vault https://<your-keyvault-name>.vault.azure.net/: jdbc:sqlserver://SERVER.database.windows.net:1433;database=DATABASE; ».

    Créez maintenant une clé secrète, puis récupérez-la à l’aide du SDK Java. 
    ```azurecli
    curl -X PUT https://myspringcloud-springapp.azuremicroservices.io/secrets/test?value=success

    curl https://myspringcloud-springapp.azuremicroservices.io/secrets/test
    ```

    Vous obtenez le message suivant : « Successfully got the value of secret test from Key Vault https://<your-keyvault-name>.vault.azure.net: success ». 

## <a name="next-steps"></a>Étapes suivantes

* [Comment accéder à un objet blob de stockage avec une identité managée dans Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/managed-identity-storage-blob)
* [Comment activer une identité managée affectée par le système pour une application Azure Spring Cloud](./spring-cloud-howto-enable-system-assigned-managed-identity.md)
* [En savoir plus sur les identités managées pour les ressources Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [Authentifier Azure Spring Cloud auprès de Key Vault dans GitHub Actions](./spring-cloud-github-actions-key-vault.md)
