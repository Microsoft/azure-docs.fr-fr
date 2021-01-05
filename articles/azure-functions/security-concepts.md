---
title: Sécurisation d’Azure Functions
description: Découvrez comment exécuter votre code de fonction dans Azure à partir d’attaques courantes.
ms.date: 4/13/2020
ms.topic: conceptual
ms.openlocfilehash: ee54ff8c1efaee00999888891e6de255060aa416
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491322"
---
# <a name="securing-azure-functions"></a>Sécurisation d’Azure Functions

À de nombreux égards, la planification du développement, du déploiement et de l’exploitation sécurisés des fonctions serverless est très similaire à celle d’une application Web ou hébergée sur le Cloud. [Azure App Service](../app-service/index.yml) fournit l’infrastructure d’hébergement pour vos applications de fonction. Cet article décrit des stratégies de sécurité pour exécuter votre code de fonction, ainsi que la façon dont App Service peut vous aider à sécuriser vos fonctions. 

[!INCLUDE [app-service-security-intro](../../includes/app-service-security-intro.md)]

Pour obtenir un ensemble de suggestions de sécurité qui suivent le [benchmark de sécurité Azure](../security/benchmarks/overview.md), consultez la [base de référence de sécurité pour Azure Functions](security-baseline.md).

## <a name="secure-operation"></a>Opération sécurisée 

Cette section vous guide dans la configuration et l’exécution de votre application de fonction de manière aussi sécurisée que possible. 

### <a name="security-center"></a>Security Center

Security Center s’intègre à votre application de fonction dans le portail. Il évalue rapidement et gratuitement les vulnérabilités potentielles de sécurité liées à la configuration. Les applications de fonction qui s’exécutent dans un plan dédié peuvent également utiliser les fonctionnalités de sécurité en temps réel de Security Center, pour un coût supplémentaire. Pour en savoir plus, consultez [Protéger vos applications Web et API Azure App Service](../security-center/defender-for-app-service-introduction.md). 

### <a name="log-and-monitor"></a>Journal et surveillance

Pour détecter les attaques, il existe une solution qui consiste à utiliser l’activité de surveillance des activités et l’analytique des journaux d'activité. Functions s’intègre à Application Insights pour collecter des données de journal, de performances et d’erreur pour votre application de fonction. Application Insights détecte automatiquement les anomalies de performance et intègre de puissants outils d’analyse conçus pour aider à diagnostiquer les problèmes et à comprendre la manière dont vos fonctions sont utilisées. Pour en savoir plus, consultez [Surveiller l’exécution des fonctions Azure](functions-monitoring.md).

Functions s’intègre également aux journaux d’activité d’Azure Monitor pour vous permettre de consolider les journaux des applications de fonction avec des événements système pour faciliter l’analyse. Vous pouvez utiliser les paramètres de diagnostic pour configurer l’exportation en continu des journaux d’activité et des métriques de la plateforme pour vos fonctions vers la destination de votre choix, par exemple un espace de travail Log Analytics. Pour en savoir plus, consultez [Monitorage d’Azure Functions avec Azure Monitor Logs](functions-monitor-log-analytics.md). 

Pour automatiser la détection et la réponse aux menaces à l’échelle de l’entreprise, transmettez en continu vos journaux et événements dans un espace de travail Log Analytics. Vous pouvez ensuite connecter Azure Sentinel à cet espace de travail. Pour en savoir plus, consultez [Présentation d’Azure Sentinel](../sentinel/overview.md).  

Pour plus d’informations sur les recommandations de sécurité pour l’observabilité, consultez la [Base de référence de sécurité Azure pour Azure Functions](security-baseline.md#logging-and-monitoring). 

### <a name="require-https"></a>Exiger HTTPS

Par défaut, les clients peuvent se connecter à des applications web en utilisant aussi bien le protocole HTTP que le protocole HTTPS. Nous vous recommandons de rediriger le protocole HTTP vers HTTPS, car HTTPS utilise le protocole SSL/TLS pour garantir une connexion sécurisée, à la fois chiffrée et authentifiée. Pour en savoir plus, consultez [Enforce HTTPS (Renforcer HTTPS)](../app-service/configure-ssl-bindings.md#enforce-https).

Lorsque vous avez besoin d’un protocole HTTPS, vous devez également disposer de la dernière version de TLS. Pour en savoir plus, consultez [Enforce TLS versions (Renforcer les versions TLS)](../app-service/configure-ssl-bindings.md#enforce-tls-versions).

Pour plus d’informations, consultez l’article dédié à la [sécurité de la couche de transport (TLS)](../app-service/overview-security.md#https-and-certificates).

### <a name="function-access-keys"></a>Clé d’accès aux fonctions

[!INCLUDE [functions-authorization-keys](../../includes/functions-authorization-keys.md)]

#### <a name="system-key"></a>Clé système 

Les extensions spécifiques peuvent nécessiter une clé gérée par le système pour accéder aux points de terminaison webhook. Les clés système sont conçues pour les points de terminaison de fonctions spécifiques à l’extension qui sont appelés par les composants internes. Par exemple, le [déclencheur Event Grid](functions-bindings-event-grid-trigger.md) exige que l’abonnement utilise une clé système lors de l’appel du point de terminaison du déclencheur. Durable Functions utilise également des clés système pour appeler [API d’extension de tâche durable](durable/durable-functions-http-api.md). 

L’étendue des clés système est déterminée par l’extension, mais elle s’applique généralement à l’ensemble de l’application de fonction. Les clés système peuvent uniquement être créées par des extensions spécifiques, et vous ne pouvez pas définir explicitement leurs valeurs. À l’instar d’autres clés, vous pouvez générer une nouvelle valeur pour la clé à partir du portail ou à l’aide des API clé.

#### <a name="keys-comparison"></a>Comparaison des clés

Le tableau suivant compare les utilisations de différents types de clés d’accès :

| Action                                        | Étendue                    | Clés valides         |
|-----------------------------------------------|--------------------------|--------------------|
| Exécuter une fonction                            | Fonction spécifique        | Fonction           |
| Exécuter une fonction                            | Toutes les fonctions             | Fonction ou hôte   |
| Appeler un point de terminaison administrateur                        | Conteneur de fonctions             | Hôte (maître uniquement) |
| Appeler des API d’extension de tâche durables              | Application de fonction<sup>1</sup> | Système <sup>2</sup> |
| Appeler un webhook spécifique à une extension (interne) | Application de fonction<sup>1</sup> | système<sup>2</sup> |

<sup>1</sup>Étendue déterminée par l’extension.  
<sup>2</sup>Noms spécifiques définis par l’extension.

Pour en savoir plus sur les clés d’accès, consultez [l’article Déclencheurs et liaisons HTTP](functions-bindings-http-webhook-trigger.md#obtaining-keys).


#### <a name="secret-repositories"></a>Référentiels de secrets

Par défaut, les clés sont stockées dans un conteneur de stockage Blob dans le compte fourni par le paramètre `AzureWebJobsStorage`. Vous pouvez utiliser des paramètres d’application spécifiques pour remplacer ce comportement et stocker des clés dans un emplacement différent.

|Emplacement  |Paramètre | Valeur | Description  |
|---------|---------|---------|---------|
|Autre compte de stockage     |  `AzureWebJobsSecretStorageSas`       | `<BLOB_SAS_URL` | Stocke les clés dans le stockage Blob d’un deuxième compte de stockage, en fonction de l’URL SAS fournie. Les clés sont chiffrées avant d’être stockées à l’aide d’un secret unique pour votre application de fonction. |
|Système de fichiers   | `AzureWebJobsSecretStorageType`   |  `files`       | Les clés sont conservées sur le système de fichiers, chiffrées avant le stockage à l’aide d’un secret unique à votre application de fonction. |
|Azure Key Vault  | `AzureWebJobsSecretStorageType`<br/>`AzureWebJobsSecretStorageKeyVaultName` | `keyvault`<br/>`<VAULT_NAME>` | Le coffre doit avoir une stratégie d’accès correspondant à l’identité managée affectée par le système de la ressource d’hébergement. La stratégie d’accès doit accorder à l’identité les autorisations secrètes suivantes : `Get`, `Set`, `List` et `Delete`. <br/>Lors d’une exécution locale, l’identité du développeur est utilisée et les paramètres doivent se trouver dans le [fichier local.settings.json](functions-run-local.md#local-settings-file). | 
|Secrets Kubernetes  |`AzureWebJobsSecretStorageType`<br/>`AzureWebJobsKubernetesSecretName` (facultatif) | `kubernetes`<br/>`<SECRETS_RESOURCE>` | Pris en charge uniquement lors de l’exécution du runtime Functions dans Kubernetes. Lorsque `AzureWebJobsKubernetesSecretName` n’est pas défini, le référentiel est considéré comme étant en lecture seule. Dans ce cas, les valeurs doivent être générées avant le déploiement. La boîte à outils Azure Functions Core Tools génère automatiquement les valeurs lors du déploiement sur Kubernetes.|

### <a name="authenticationauthorization"></a>Authentification/autorisation

Alors que les clés de fonction peuvent offrir une certaine atténuation des risques d’accès indésirable, la seule façon de sécuriser véritablement vos points de terminaison de fonction consiste à implémenter l’authentification positive des clients accédant à vos fonctions. Vous pouvez ensuite prendre des décisions d’autorisation basées sur l’identité.  

[!INCLUDE [functions-enable-auth](../../includes/functions-enable-auth.md)]

### <a name="permissions"></a>Autorisations

Comme pour n’importe quelle application ou n’importe quel service, l’objectif est d’exécuter votre application de fonction avec les autorisations les plus basses possibles. 

#### <a name="user-management-permissions"></a>Autorisations de gestion des utilisateurs

Functions prend en charge le [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../role-based-access-control/overview.md). Les rôles Azure pris en charge par Functions sont [Contributeur](../role-based-access-control/built-in-roles.md#contributor), [Propriétaire](../role-based-access-control/built-in-roles.md#owner) et [Lecteur](../role-based-access-control/built-in-roles.md#owner). 

Les autorisations sont effectives au niveau de l’application de fonction. Le rôle de Contributeur est requis pour effectuer la plupart des tâches au niveau application de fonction. Seul le rôle de Propriétaire peut supprimer une application de fonction. 

#### <a name="organize-functions-by-privilege"></a>Organiser les fonctions par privilège 

Les chaînes de connexion et d’autres informations d’identification stockées dans les paramètres d’application donnent à toutes les fonctions de l’application de fonction le même ensemble d’autorisations dans la ressource associée. Envisagez de réduire le nombre de fonctions ayant accès à des informations d’identification spécifiques en déplaçant des fonctions qui n’utilisent pas lesdites informations d’identification vers une application de fonction distincte. Vous pouvez toujours utiliser des techniques telles que [le chaînage de fonctions](/learn/modules/chain-azure-functions-data-using-bindings/) pour passer des données entre des fonctions dans différentes applications de fonction.  

#### <a name="managed-identities"></a>Identités managées

[!INCLUDE [app-service-managed-identities](../../includes/app-service-managed-identities.md)]

Pour plus d’informations, consultez [Guide pratique pour utiliser des identités managées avec App Service et Azure Functions](../app-service/overview-managed-identity.md?toc=%2fazure%2fazure-functions%2ftoc.json).

#### <a name="restrict-cors-access"></a>Restreindre l’accès CORS

[Le Cross-origin ressource sharing (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) est un moyen d’autoriser l’exécution des applications web dans un autre domaine pour effectuer des demandes à vos points de terminaison de déclencheur HTTP. App Service fournit une prise en charge intégrée pour la gestion des en-têtes CORS requis dans les requêtes HTTP. Les règles CORS sont définies au niveau d’une application de fonction.  

Il est tentant d’utiliser un caractère générique qui permette à tous les sites d’accéder à votre point de terminaison. Toutefois, cela a pour effet de nuire à l’objectif de CORS, qui permet d’éviter les attaques de script entre sites. Ajoutez plutôt une entrée CORS distincte pour le domaine de chaque application web qui doit accéder à votre point de terminaison. 

### <a name="managing-secrets"></a>Gestion des secrets 

Pour pouvoir se connecter aux différents services et ressources qui doivent exécuter votre code, les applications de fonction doivent être en mesure d’accéder aux secrets, tels que les chaînes de connexion et les clés de service. Cette section décrit comment stocker les secrets requis par vos fonctions.

Ne stockez jamais de secrets dans votre code de fonction. 

#### <a name="application-settings"></a>Paramètres de l’application

Par défaut, vous stockez les chaînes de connexion et les secrets utilisés par votre application de fonction et les liaisons en tant que paramètres d’application. Ces informations d’identification sont alors disponibles pour votre code de fonction et les différentes liaisons utilisées par la fonction. Le nom du paramètre d’application (clé) est utilisé pour récupérer la valeur réelle, qui est le secret. 

Par exemple, chaque application de fonction nécessite un compte de stockage associé, qui est utilisé par le runtime. Par défaut, la connexion à ce compte de stockage est stockée dans un paramètre d’application nommé `AzureWebJobsStorage`.

Les paramètres de l’application et les chaînes de connexion sont stockés dans Azure. Ils sont déchiffrés uniquement avant d’être injectés dans la mémoire de processus de votre application au démarrage de celle-ci. Les clés de chiffrement sont régulièrement permutées. Si vous préférez gérer le stockage sécurisé de vos secrets, le paramètre d’application doit plutôt faire référence à Azure Key Vault. 

Vous pouvez également chiffrer les paramètres par défaut dans le fichier local.settings.json lors du développement de fonctions sur votre ordinateur local. Pour plus d’informations, consultez la propriété `IsEncrypted` dans le [fichier de paramètres locaux](functions-run-local.md#local-settings-file).  

#### <a name="key-vault-references"></a>Références Key Vault

Même si les paramètres d’application sont suffisants pour la plupart des fonctions, vous pouvez souhaiter partager les mêmes secrets entre plusieurs services. Dans ce cas, le stockage redondant des secrets entraîne plus de vulnérabilités potentielles. Une approche plus sécurisée consiste à utiliser un service de stockage secret central et à utiliser des références à ce service au lieu des secrets eux-mêmes.      

[Azure Key Vault](../key-vault/general/overview.md) est un service qui fournit une gestion centralisée des secrets, avec un contrôle total sur les stratégies d’accès et l’historique d’audit. Vous pouvez utiliser une référence Key Vault à place d’une chaîne de connexion ou d’une clé dans vos paramètre d’application. Pour en savoir plus, consultez [Utiliser des références Key Vault pour App Service et Azure Functions](../app-service/app-service-key-vault-references.md?toc=%2fazure%2fazure-functions%2ftoc.json).

### <a name="set-usage-quotas"></a>Configurer un quota d’utilisation

Envisagez de paramétrer un quota d’utilisation sur l’exécution des fonctions dans un plan de consommation. Lorsque vous définissez une limite quotidienne de Go-s sur la somme de l’exécution totale des fonctions dans votre application de fonction, l’exécution est arrêtée lorsque la limite est atteinte. Cela peut permettre de réduire le code malveillant qui exécute vos fonctions. Pour savoir comment estimer la consommation de vos fonctions, consultez [Estimation des coûts d’un plan de consommation](functions-consumption-costs.md). 

### <a name="data-validation"></a>Validation des données

Les déclencheurs et les liaisons utilisés par vos fonctions ne fournissent aucune validation de données supplémentaire. Votre code doit valider toutes les données reçues à partir d’un déclencheur ou d’une liaison d’entrée. Si un service en amont est compromis, vous ne voulez pas que les entrées non validées circulent dans vos fonctions. Par exemple, si votre fonction stocke des données à partir d’une file d’attente de stockage Azure dans une base de données relationnelle, vous devez valider les données et paramétrer vos commandes pour éviter les attaques par injection de code SQL. 

Ne partez pas du principe que les données entrant dans votre fonction ont déjà été validées ou assainies. Il est également judicieux de vérifier que les données écrites dans les liaisons de sortie sont valides. 

### <a name="handle-errors"></a>des erreurs

Bien qu’il semble basique, il est important d’écrire une bonne gestion des erreurs dans vos fonctions. Les erreurs non gérées se propagent à l’hôte et sont gérées par le runtime. Les liaisons différentes gèrent le traitement des erreurs différemment. Pour plus d’informations, consultez [Conseils de gestion des erreurs Azure Functions](functions-bindings-error-pages.md).

### <a name="disable-remote-debugging"></a>Désactiver le débogage à distance

Assurez-vous que le débogage à distance est désactivé, sauf lorsque vous déboguez activement vos fonctions. Vous pouvez désactiver le débogage à distance sous l’onglet **Paramètres généraux** de la **Configuration** de votre application de fonction dans le portail. 

### <a name="restrict-cors-access"></a>Restreindre l’accès CORS

[!INCLUDE [functions-cors](../../includes/functions-cors.md)]

N’utilisez pas de caractères génériques dans votre liste d’origines autorisées. Répertoriez plutôt les domaines spécifiques à partir desquels vous souhaitez obtenir des requêtes.

### <a name="store-data-encrypted"></a>Stocker les données chiffrées

[!INCLUDE [functions-storage-encryption](../../includes/functions-storage-encryption.md)]

## <a name="secure-deployment"></a>Déploiement sécurisé

Azure Functions, outil d’intégration, facilite la publication de code de projet de fonction locale dans Azure. Il est important de comprendre le fonctionnement du déploiement lors de la prise en compte de la sécurité d’une topologie d’Azure Functions.   

### <a name="deployment-credentials"></a>Informations d’identification de déploiement

Les déploiements d’App Service nécessitent un ensemble d’informations d’identification de déploiement. Ces informations d’identification de déploiement sont utilisées pour sécuriser vos déploiements d’application de fonction. Les informations d’identification de déploiement sont gérées par la plateforme App Service et sont chiffrées au repos. 

Il existe deux types d’informations d'identification de déploiement :

[!INCLUDE [app-service-deploy-credentials](../../includes/app-service-deploy-credentials.md)]

À ce stade, Key Vault n’est pas pris en charge pour les informations d’identification de déploiement. Pour plus d’informations sur la façon de configurer les informations d’identification de déploiement, consultez [Configurer les informations d’identification de déploiement pour Azure App Service](../app-service/deploy-configure-credentials.md).

### <a name="disable-ftp"></a>Désactiver le protocole FTP

Par défaut, un point de terminaison FTP est activé pour chaque application de fonction. Le point de terminaison FTP est accessible à l’aide des informations d’identification de déploiement. 

Le protocole FTP n’est pas recommandé pour le déploiement de votre code de fonction. Les déploiements FTP sont manuels et nécessitent la synchronisation des déclencheurs. Pour plus d’informations, consultez [Déploiement FTP](functions-deployment-technologies.md#ftp). 

Si vous n’avez pas l’intention d’utiliser le protocole FTP, vous devez le désactiver dans le portail. Si vous choisissez d’utiliser le protocole FTP, vous devez [renforcer les protocoles FTPS](../app-service/deploy-ftp.md#enforce-ftps).

### <a name="secure-the-scm-endpoint"></a>Sécuriser le point de terminaison SCM

Chaque application de fonction a un point de terminaison de service `scm` correspondant qui est utilisé par le service d’Outils avancés (Kudu) pour les déploiements et d’autres [extensions de site](https://github.com/projectkudu/kudu/wiki/Azure-Site-Extensions) d’App Service. Le point de terminaison SCM pour une application de fonction est toujours sous la forme d’une URL`https://<FUNCTION_APP_NAME.scm.azurewebsites.net>`. Lorsque vous utilisez l’isolement réseau pour sécuriser vos fonctions, vous devez également tenir compte de ce point de terminaison. 

En ayant un point de terminaison SCM distinct, vous pouvez contrôler les déploiements et d’autres fonctionnalités d’outils avancés pour une application de fonction qui sont isolées ou en cours d’exécution dans un réseau virtuel. Le point de terminaison SCM prend en charge l’authentification de base (à l’aide des informations d’identification de déploiement) et l’authentification unique avec vos informations d’identification du portail Azure. Pour plus d’informations, consultez [Accessing the Kudu service (Accès au service Kudu)](https://github.com/projectkudu/kudu/wiki/Accessing-the-kudu-service). 

### <a name="continuous-security-validation"></a>Valide continue de la sécurité

Comme la sécurité doit être considérée comme une étape dans le processus de développement, il est logique d’implémenter également des validations de sécurité dans un environnement de déploiement continu. Cette procédure est parfois appelée DevSecOps. L’utilisation d’Azure DevOps pour votre pipeline de déploiement vous permet d’intégrer la validation dans le processus de déploiement. Pour plus d’informations, consultez [Learn how to add continuous security validation to your CI/CD pipeline (Guide pratique pour ajouter une validation de sécurité continue à votre pipeline CI/CD)](/azure/devops/migrate/security-validation-cicd-pipeline).  

## <a name="network-security"></a>Sécurité du réseau

La restriction de l’accès réseau à votre application de fonction vous permet de contrôler qui peut accéder à vos points de terminaison Functions. Functions tire parti de l’infrastructure App Service pour permettre à vos fonctions d’accéder aux ressources sans utiliser d’adresses de routage par Internet ou de restreindre l’accès à Internet à un point de terminaison de fonction. Pour en savoir plus sur les options de mise en réseau, consultez [Options de mise en réseau d’Azure Functions](functions-networking-options.md).

### <a name="set-access-restrictions"></a>Définir les restrictions d’accès

Les restrictions d’accès vous permettent de définir des listes de règles d’autorisation/de refus pour contrôler le trafic vers votre application. Les règles sont évaluées par ordre de priorité. Si aucune règle n’est définie, votre application acceptera le trafic à partir de n’importe quelle adresse. Pour en savoir plus, consultez [Restrictions d’accès dans Azure App Service #](../app-service/app-service-ip-restrictions.md?toc=%2fazure%2fazure-functions%2ftoc.json).

### <a name="private-site-access"></a>Accès aux sites privés

[!INCLUDE [functions-private-site-access](../../includes/functions-private-site-access.md)]

### <a name="deploy-your-function-app-in-isolation"></a>Déployer votre application de fonction en isolement

[!INCLUDE [functions-deploy-isolation](../../includes/functions-deploy-isolation.md)]

### <a name="use-a-gateway-service"></a>Utiliser un service de passerelle

Les services de passerelle, tels que [Azure Application Gateway](../application-gateway/overview.md) et [Azure Front Door](../frontdoor/front-door-overview.md) vous permettent de configurer un pare-feu d’applications Web (WAF). Les règles WAF sont utilisées pour surveiller ou bloquer les attaques détectées, ce qui permet d’avoir une couche supplémentaire de protection pour vos fonctions. Pour configurer un WAF, votre application de fonction doit être exécutée dans un ASE ou à l’aide de points de terminaison privés (préversion). Pour plus d’informations, consultez [Utiliser des points de terminaison privés](../app-service/networking/private-endpoint.md).    

## <a name="next-steps"></a>Étapes suivantes

+ [Base de référence de sécurité Azure pour Azure Functions](security-baseline.md)
+ [Diagnostics d’Azure Functions](functions-diagnostics.md)
