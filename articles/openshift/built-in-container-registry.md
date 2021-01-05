---
title: Configurer le registre de conteneurs intégré pour Azure Red Hat OpenShift 4
description: Configurer le registre de conteneurs intégré pour Azure Red Hat OpenShift 4
author: jiangma
ms.author: jiangma
ms.service: container-service
ms.topic: conceptual
ms.date: 10/15/2020
ms.openlocfilehash: 9ed53721b66dc03bad24e0510e0c8a970c61aec1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492418"
---
# <a name="configure-built-in-container-registry-for-azure-red-hat-openshift-4"></a>Configurer le registre de conteneurs intégré pour Azure Red Hat OpenShift 4

Azure Red Hat OpenShift fournit un registre d’images conteneur intégré appelé [OpenShift Container Registry (OCR)](https://docs.openshift.com/aro/4/registry/architecture-component-imageregistry.html) qui ajoute la possibilité de provisionner automatiquement de nouveaux référentiels d’images à la demande. Cela fournit aux utilisateurs un emplacement intégré pour que leurs builds d’application envoient (par push) les images obtenues.

Cet article explique comment configurer le registre d’images conteneurs intégré pour un cluster Azure Red Hat OpenShift (ARO) 4. Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Configurer Azure AD
> * Configurer OpenID Connect
> * Accéder au registre d’images conteneurs intégré

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous disposez déjà d’un cluster ARO. Si vous avez besoin d’un cluster ARO, consultez le tutoriel ARO [Créer un cluster Azure Red Hat OpenShift 4](./tutorial-create-cluster.md). Veillez à créer le cluster avec l’argument `--pull-secret` pour la commande `az aro create`.  Cela est nécessaire pour configurer l’authentification Azure Active Directory et le registre de conteneurs intégré.

Une fois que vous avez votre cluster, connectez-vous à celui-ci en effectuant les étapes décrites dans [Se connecter à un cluster Azure Red Hat OpenShift 4](./tutorial-connect-cluster.md).
   * Veillez à suivre les étapes décrites dans la section « Installer l’interface CLI OpenShift », car nous allons utiliser la commande `oc` plus loin dans cet article.
   * Prenez note de l’URL de la console du cluster, qui ressemble à `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`. Les valeurs de `<random>` et `<region>` seront utilisées plus loin dans cet article.
   * Notez les informations d’identification de `kubeadmin`. Vous les utiliserez également ultérieurement dans cet article.

### <a name="configure-azure-active-directory-authentication"></a>Configurer l’authentification Azure Active Directory 

Azure Active Directory (Azure AD) implémente OpenID Connect (OIDC). OIDC vous permet d’utiliser Azure AD pour vous connecter au cluster ARO. Effectuez les étapes décrites dans [Configurer l’authentification Azure Active Directory](configure-azure-ad-cli.md) pour configurer votre cluster.

## <a name="access-the-built-in-container-image-registry"></a>Accéder au registre d’images conteneurs intégré

Maintenant que vous avez configuré les méthodes d’authentification sur le cluster ARO, activons l’accès au registre intégré.

#### <a name="define-the-azure-ad-user-to-be-an-administrator"></a>Définir l’utilisateur Azure AD en tant qu’administrateur

1. Connectez-vous à la console web OpenShift à partir de votre navigateur en utilisant les informations d’identification d’un utilisateur Azure AD. Nous allons tirer parti de l’authentification OpenShift OpenID sur Azure Active Directory pour définir l’administrateur à l’aide d’OpenID.

   1. Utilisez une fonctionnalité de fenêtre de navigateur InPrivate, Incognito ou autre fonctionnalité équivalente pour vous connecter à la console. La fenêtre présentera un aspect différent après l’activation d’OIDC.
   
   :::image type="content" source="media/built-in-container-registry/oidc-enabled-login-window.png" alt-text="Fenêtre de connexion OpenID Connect activée.":::
   1. Sélectionnez **openid**

   > [!NOTE]
   > Notez le nom d’utilisateur et le mot de passe que vous utilisez pour vous connecter ici. Ces nom d’utilisateur et mot de passe fonctionneront pour l’administrateur en lien avec d’autres actions décrites dans cet article et dans d’autres.
2. Connectez-vous avec l’interface CLI OpenShift en procédant comme suit.  Pour la discussion, ce processus est appelé `oc login`.
   1. Dans l’angle supérieur droit de la console web, développez le menu contextuel de l’utilisateur connecté, puis sélectionnez **Copier la commande de connexion**.
   2. Connectez-vous à une nouvelle fenêtre d’onglets avec le même utilisateur si nécessaire.
   3. Sélectionnez **Afficher le jeton**.
   4. Copiez la valeur indiquée sous **Connectez-vous avec ce jeton** dans le presse-papiers, puis exécutez le jeton dans un interpréteur de commandes, comme illustré ici.

       ```bash
       oc login --token=XOdASlzeT7BHT0JZW6Fd4dl5EwHpeBlN27TAdWHseob --server=https://api.aqlm62xm.rnfghf.aroapp.io:6443
       Logged into "https://api.aqlm62xm.rnfghf.aroapp.io:6443" as "kube:admin" using the token provided.

       You have access to 57 projects, the list has been suppressed. You can list all projects with 'oc projects'

       Using project "default".
       ```

3. Exécutez `oc whoami` dans la console et notez la sortie en tant que **\<aad-user>** .  Vous utiliserez cette valeur plus loin dans cet article.
4. Déconnectez-vous de la console web OpenShift. Sélectionnez le bouton en haut à droite de la fenêtre du navigateur, intitulé **\<aad-user>** , puis choisissez **Se déconnecter**.


#### <a name="grant-the-azure-ad-user-the-necessary-roles-for-registry-interaction"></a>Accorder à l’utilisateur Azure AD les rôles nécessaires pour l’interaction avec le registre

1. Connectez-vous à la console web OpenShift à partir de votre navigateur en utilisant les informations d’identification de `kubeadmin`.
1. Connectez-vous à l’interface CLI OpenShift avec le jeton pour `kubeadmin` en suivant les étapes décrites pour `oc login` ci-dessus, mais faites-le après vous être connecté à la console web avec `kubeadmin`.
1. Exécutez les commandes suivantes pour activer l’accès au registre intégré pour l’utilisateur **aad-user**.

   ```bash
   # Switch to project "openshift-image-registry"
   oc project openshift-image-registry
   
   # Output should look similar to the following.
   # Now using project "openshift-image-registry" on server "https://api.x8xl3f4y.eastus.aroapp.io:6443".
   ```

   ```bash
   # Expose the registry using "DefaultRoute"
   oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge

   # Output should look similar to the following.
   # config.imageregistry.operator.openshift.io/cluster patched
   ```

   ```bash
   # Add roles to "aad-user" for pulling and pushing images
   # Note: replace "<aad-user>" with the one you wrote down before
   oc policy add-role-to-user registry-viewer <aad-user>

   # Output should look similar to the following.
   # clusterrole.rbac.authorization.k8s.io/registry-viewer added: "kaaIjx75vFWovvKF7c02M0ya5qzwcSJ074RZBfXUc34"
   ```

   ```bash
   oc policy add-role-to-user registry-editor <aad-user>
   # Output should look similar to the following.
   # clusterrole.rbac.authorization.k8s.io/registry-editor added: "kaaIjx75vFWovvKF7c02M0ya5qzwcSJ074RZBfXUc34"
   ```

#### <a name="obtain-the-container-registry-url"></a>Obtenir l’URL du registre de conteneurs

Utilisez la commande `oc get route` ci-dessous pour récupérer l’URL du registre de conteneurs.

```bash
# Note: the value of "Container Registry URL" in the output is the fully qualified registry name.
HOST=$(oc get route default-route --template='{{ .spec.host }}')
echo "Container Registry URL: $HOST"
```

   > [!NOTE]
   > Notez la sortie de la console pour **URL du registre de conteneurs**. Elle sera utilisée comme nom de registre complet dans le cadre de ce guide et des guides suivants.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez configuré le registre d’images conteneurs intégré, vous pouvez commencer en déployant une application sur OpenShift. Pour les applications Java, consultez [Déployer une application Java avec Open Liberty/WebSphere Liberty sur un cluster Azure Red Hat OpenShift 4](howto-deploy-java-liberty-app.md).