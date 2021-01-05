---
title: Résoudre des problèmes de connexion à un registre
description: Symptômes, causes et résolution de problèmes courants de connexion à un registre de conteneurs Azure
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: 5499c64bef8ce36a5f622c4d847b417ef49a5a03
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93379500"
---
# <a name="troubleshoot-registry-login"></a>Résoudre des problèmes de connexion au registre

Cet article vous aide à résoudre des problèmes que vous pourriez rencontrer lors de la connexion à un registre de conteneurs Azure. 

## <a name="symptoms"></a>Symptômes

Peuvent inclure un ou plusieurs des symptômes suivants :

* Impossibilité de se connecter au registre à l’aide de `docker login`, de `az acr login` ou des deux.
* Impossibilité de se connecter au registre et affichage de l’erreur `unauthorized: authentication required` ou `unauthorized: Application not registered with AAD`.
* Impossibilité de se connecter au registre et affichage de l’erreur Azure CLI `Could not connect to the registry login server`.
* Impossibilité d’envoyer ou d’extraire des images et affichage de l’erreur Docker `unauthorized: authentication required`.
* Impossibilité d’accéder au registre à partir d’Azure Kubernetes Service, d’Azure DevOps ou d’un autre service Azure.
* Impossibilité d’accéder au registre et affichage de l’erreur `Error response from daemon: login attempt failed with status: 403 Forbidden`. Consultez [Résoudre des problèmes de réseau avec un registre](container-registry-troubleshoot-access.md).
* Impossibilité de consulter ou d’afficher des paramètres du registre dans le portail Azure ou de gérer le registre à l’aide d’Azure CLI.

## <a name="causes"></a>Causes

* Docker n’est pas configuré correctement dans votre environnement : [solution](#check-docker-configuration).
* Le registre n’existe pas ou le nom est incorrect : [solution](#specify-correct-registry-name).
* Les informations d’identification du registre ne sont pas valides : [solution](#confirm-credentials-to-access-registry).
* Les informations d’identification ne sont pas autorisées pour les opérations d’envoi, d’extraction ou d’Azure Resource Manager : [solution](#confirm-credentials-are-authorized-to-access-registry).
* Les informations d’identification ont expiré : [solution](#check-that-credentials-arent-expired)

## <a name="further-diagnosis"></a>Diagnostics plus poussés 

Exécutez la commande [az acr check-health](/cli/azure/acr#az-acr-check-health) pour obtenir des informations supplémentaires sur l’intégrité de l’environnement de registre, et éventuellement l’accès à un registre cible. Par exemple, diagnostiquez des erreurs de configuration de Docker ou des problèmes de connexion à Azure Active Directory. 

Pour des exemples de commandes, consultez [Vérifier l’intégrité d’un registre de conteneurs Azure](container-registry-check-health.md). Si des erreurs sont signalées, examinez les [Informations de référence sur les erreurs](container-registry-health-error-reference.md) et les sections suivantes pour obtenir les solutions recommandées.

> [!NOTE]
> Certaines erreurs d’authentification ou d’autorisation peuvent également se produire s’il existe des configurations de pare-feu ou de réseau qui empêchent l’accès au registre. Consultez [Résoudre des problèmes de réseau avec un registre](container-registry-troubleshoot-access.md).

## <a name="potential-solutions"></a>Solutions possibles

### <a name="check-docker-configuration"></a>Vérifier la configuration de Docker

La plupart des flux d’authentification d’Azure Container Registry requièrent une installation de Docker locale pour vous permettre de vous authentifier auprès de votre registre pour des opérations telles que l’envoi et l’extraction d’images. Vérifiez que le client et le démon de l’interface de ligne de commande de Docker (moteur Docker) s’exécutent dans votre environnement. Vous avez besoin d’un client Docker version 18.03 ou ultérieure.

Liens connexes :

* [Vue d’ensemble de l’authentification](container-registry-authentication.md#authentication-options)
* [FAQ sur le registre de conteneurs](container-registry-faq.md)

### <a name="specify-correct-registry-name"></a>Spécifier un nom de registre correct

Lorsque vous utilisez `docker login`, indiquez le nom complet du serveur de connexion du registre, par exemple *myregistry.azurecr.io*. Veillez à utiliser uniquement des lettres minuscules. Exemple :

```console
docker login myregistry.azurecr.io
```

Lorsque vous utilisez la commande [az acr login](/cli/azure/acr#az-acr-login) avec une identité Azure Active Directory, commencez par vous [connecter à Azure CLI](/cli/azure/authenticate-azure-cli), puis spécifiez le nom de ressource Azure du registre. Le nom de la ressource est le nom fourni lors de la création du registre, par exemple *myregistry* (sans suffixe de domaine). Exemple :

```azurecli
az acr login --name myregistry
```

Liens connexes :

* [la connexion az acr est réussie, mais docker a échoué avec l’erreur : non autorisé : authentification requise](container-registry-faq.md#az-acr-login-succeeds-but-docker-fails-with-error-unauthorized-authentication-required )

### <a name="confirm-credentials-to-access-registry"></a>Confirmer les informations d’identification pour accéder au registre

Vérifiez la validité des informations d’identification que vous utilisez pour votre scénario ou qui vous ont été fournies par un propriétaire du registre. Autres problèmes possibles

* Si vous utilisez un principal de service Active Directory, veillez à utiliser les informations d’identification correctes dans le locataire Active Directory :
  * Nom d'utilisateur - ID d’application du principal de service (également appelé *ID client*)
  * Mot de passe - mot de passe du principal de service (également appelé *clé secrète client*)
* Si vous utilisez un service Azure, tel que Azure Kubernetes Service ou Azure DevOps pour accéder au registre, confirmez la configuration du registre pour votre service.
* Si vous avez exécuté `az acr login` avec l’option `--expose-token` qui active la connexion au registre sans utiliser le démon Docker, veillez à vous authentifier avec le nom d’utilisateur `00000000-0000-0000-0000-000000000000`.
* Si votre registre est configuré pour un [accès par tirage (pull) anonyme](container-registry-faq.md#how-do-i-enable-anonymous-pull-access), les informations d’identification Docker existantes stockées à partir d’une connexion Docker précédente peuvent empêcher l’accès anonyme. Exécutez `docker logout` avant de tenter d’effectuer une opération de tirage (pull) anonyme sur le Registre.

Liens connexes :

* [Vue d’ensemble de l’authentification](container-registry-authentication.md#authentication-options)
* [Connexion individuelle avec Azure AD](container-registry-authentication.md#individual-login-with-azure-ad)
* [Se connecter avec le principal de service](container-registry-auth-service-principal.md)
* [Se connecter avec une identité managée](container-registry-authentication-managed-identity.md)
* [Se connecter avec un jeton délimité par le dépôt](container-registry-repository-scoped-permissions.md)
* [Se connecter avec un compte d’administrateur](container-registry-authentication.md#admin-account)
* [Codes d’erreur d’authentification et d’autorisation Azure AD](../active-directory/develop/reference-aadsts-error-codes.md)
* Référence [az acr login](/cli/azure/acr#az-acr-login)

### <a name="confirm-credentials-are-authorized-to-access-registry"></a>Vérifier que les informations d’identification autorisent l’accès au registre

Vérifiez les autorisations du registre associées aux informations d’identification, telles que le rôle Azure `AcrPull` pour extraire des images du registre, ou le rôle `AcrPush` pour envoyer des images. 

L’accès à un registre dans le portail ou la gestion du registre à l’aide d’Azure CLI nécessitent au moins le rôle `Reader` ou des autorisations équivalentes pour effectuer les opérations Azure Resource Manager.

Si vos autorisations ont été récemment modifiées pour permettre l’accès au registre via le portail, vous devrez peut-être utiliser une session incognito ou privée dans votre navigateur pour éviter tout cache de navigateur ou tous cookies obsolètes.

Vous ou un propriétaire du registre devez disposer de privilèges suffisants dans l’abonnement pour ajouter ou supprimer des attributions de rôle.

Liens connexes :

* [Rôles et autorisations Azure – Azure Container Registry](container-registry-roles.md)
* [Se connecter avec un jeton délimité par le dépôt](container-registry-repository-scoped-permissions.md)
* [Ajouter ou supprimer des attributions de rôles Azure avec le portail Azure](../role-based-access-control/role-assignments-portal.md)
* [Utiliser le portail pour créer une application Azure AD et un principal de service ayant accès aux ressources](../active-directory/develop/howto-create-service-principal-portal.md)
* [Créer un secret d’application](../active-directory/develop/howto-create-service-principal-portal.md#option-2-create-a-new-application-secret)
* [Codes d’authentification et d’autorisation Azure AD](../active-directory/develop/reference-aadsts-error-codes.md)

### <a name="check-that-credentials-arent-expired"></a>Vérifier que les informations d’identification n’ont pas expiré

Les jetons et les informations d’identification d’Active Directory peuvent expirer après des périodes définies, ce qui empêche l’accès au registre. Pour activer l’accès, il se peut que vous deviez réinitialiser ou régénérer les informations d’identification.

* En cas d’utilisation d’une identité AD individuelle, d’une identité managée ou d’un principal de service pour la connexion au registre, le jeton AD expire au bout de 3 heures. Connectez-vous à nouveau au registre.  
* Si vous utilisez un principal de service AD avec une clé secrète client expirée, un administrateur du compte ou un propriétaire de l’abonnement doit réinitialiser les informations d’identification ou générer un nouveau principal de service.
* Si vous utilisez un [jeton d’étendue de référentiel](container-registry-repository-scoped-permissions.md), il se peut que le propriétaire du registre doive réinitialiser un mot de passe ou générer un nouveau jeton.

Liens connexes :

* [Réinitialiser les informations d’identification du principal de service](/cli/azure/ad/sp/credential#az-ad-sp-credential-reset)
* [Régénérer les mots de passe des jetons](container-registry-repository-scoped-permissions.md#regenerate-token-passwords)
* [Connexion individuelle avec Azure AD](container-registry-authentication.md#individual-login-with-azure-ad)

## <a name="advanced-troubleshooting"></a>Dépannage avancé

Si la [collecte des journaux de ressources](container-registry-diagnostics-audit-logs.md) est activée dans le registre, consultez le journal ContainterRegistryLoginEvents. Ce journal stocke les événements d’authentification et l’état, y compris l’identité et l’adresse IP entrantes. Interrogez le journal sur les [échecs d’authentification du registre](container-registry-diagnostics-audit-logs.md#registry-authentication-failures). 

Liens connexes :

* [Journaux pour l’évaluation et l’audit de diagnostics](container-registry-diagnostics-audit-logs.md)
* [FAQ sur le registre de conteneurs](container-registry-faq.md)
* [Meilleures pratiques pour Azure Container Registry](container-registry-best-practices.md)

## <a name="next-steps"></a>Étapes suivantes

Si vous ne résolvez pas votre problème ici, consultez les options suivantes.

* Les autres rubriques de dépannage du registre sont les suivantes :
  * [Résoudre les problèmes de réseau avec le Registre](container-registry-troubleshoot-access.md)
  * [Résoudre les problèmes de performances de registre](container-registry-troubleshoot-performance.md)
* Options de [support de la communauté](https://azure.microsoft.com/support/community/)
* [Microsoft Q&A](/answers/products/)
* [Ouvrir un ticket de support](https://azure.microsoft.com/support/create-ticket/) : en fonction des informations que vous fournissez, vous pouvez effectuer un diagnostic rapide des échecs d’authentification dans votre registre.