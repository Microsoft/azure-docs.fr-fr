---
title: Configurer l’authentification pour des modèles déployés en tant que services web
titleSuffix: Azure Machine Learning
description: Découvrez comment configurer l’authentification pour les modèles Machine Learning déployés sur des services web dans Azure Machine Learning.
services: machine-learning
author: cjgronlund
ms.author: cgronlun
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: core
ms.date: 11/06/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 5bd938fce347d439c2acb4e3fcace04d5b27d770
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94447431"
---
# <a name="configure-authentication-for-models-deployed-as-web-services"></a>Configurer l’authentification pour des modèles déployés en tant que services web

Azure Machine Learning vous permet de déployer vos modèles Machine Learning formés en tant que services web. Dans cet article, découvrez comment configurer l’authentification pour ces déploiements.

Les déploiements de modèle créés par Azure Machine Learning peuvent être configurés pour utiliser l’une des deux méthodes d’authentification suivantes :

* **Basée sur une clé** : une clé statique est utilisée pour l’authentification auprès du service web.
* **Basée sur un jeton** : un jeton temporaire doit être obtenu auprès de l’espace de travail Azure Machine Learning (à l’aide d’Azure Active Directory) et utilisé pour l’authentification auprès du service web. Ce jeton expire au bout d’un certain temps et doit être actualisé pour continuer à fonctionner avec le service web.

    > [!NOTE]
    > L’authentification basée sur un jeton est disponible uniquement lors du déploiement sur Azure Kubernetes Service.

## <a name="key-based-authentication"></a>Authentification basée sur une clé

L’authentification basée sur une clé est *activée* par défaut pour les services web déployés sur Azure Kubernetes Service (AKS).

Elle est *désactivée* par défaut pour les services déployés sur Azure Container Instances (ACI). Vous pouvez toutefois l’activer en définissant `auth_enabled=True` au moment où vous créez votre service web ACI. Voici un exemple de code montrant comment créer une configuration de déploiement sur ACI lorsque l’authentification basée sur une clé est activée.

```python
from azureml.core.webservice import AciWebservice

aci_config = AciWebservice.deploy_configuration(cpu_cores = 1,
                                                memory_gb = 1,
                                                auth_enabled=True)
```

Vous pouvez ensuite utiliser la configuration ACI personnalisée dans le déploiement avec la classe `Model`.

```python
from azureml.core.model import Model, InferenceConfig


inference_config = InferenceConfig(entry_script="score.py",
                                   environment=myenv)
aci_service = Model.deploy(workspace=ws,
                       name="aci_service_sample",
                       models=[model],
                       inference_config=inference_config,
                       deployment_config=aci_config)
aci_service.wait_for_deployment(True)
```

Pour récupérer les clés d’authentification, utilisez `aci_service.get_keys()`. Pour regénérer une clé, utilisez la fonction `regen_key()` et passez la valeur **Primary** ou **Secondary**.

```python
aci_service.regen_key("Primary")
# or
aci_service.regen_key("Secondary")
```

## <a name="token-based-authentication"></a>Authentification basée sur un jeton

Quand vous activez l’authentification par jeton pour un service web, les utilisateurs doivent présenter un jeton JSON Web Token Azure Machine Learning au service web pour y accéder. Le jeton expire après un laps de temps spécifié, après quoi il doit être actualisé pour continuer à effectuer des appels.

* L’authentification par jeton est **désactivée par défaut** lors d’un déploiement sur Azure Kubernetes Service.
* L’authentification par jeton **n’est pas prise en charge** pour les déploiements sur Azure Container Instances.
* L’authentification par jeton **ne peut pas être utilisée en même temps que l’authentification basée sur une clé**.

Pour contrôler l’authentification par jeton, utilisez le paramètre `token_auth_enabled` quand vous créez ou mettez à jour un déploiement :

```python
from azureml.core.webservice import AksWebservice
from azureml.core.model import Model, InferenceConfig

# Create the config
aks_config = AksWebservice.deploy_configuration()

#  Enable token auth and disable (key) auth on the webservice
aks_config = AksWebservice.deploy_configuration(token_auth_enabled=True, auth_enabled=False)

aks_service_name ='aks-service-1'

# deploy the model
aks_service = Model.deploy(workspace=ws,
                           name=aks_service_name,
                           models=[model],
                           inference_config=inference_config,
                           deployment_config=aks_config,
                           deployment_target=aks_target)

aks_service.wait_for_deployment(show_output = True)
```

Si l’authentification par jeton est activée, vous pouvez utiliser la méthode `get_token` pour récupérer un jeton JWT (JSON Web Token) et son délai d’expiration :

> [!TIP]
> Si vous utilisez un principal de service pour obtenir le jeton et souhaitez qu’il dispose de l’accès minimal requis pour récupérer un jeton, attribuez-le au rôle **lecteur** de l’espace de travail.

```python
token, refresh_by = aks_service.get_token()
print(token)
```

> [!IMPORTANT]
> Vous devrez demander un nouveau jeton après l’heure de `refresh_by` du jeton. Si vous devez actualiser des jetons en dehors du SDK pour Python, l’une des options possibles est d’utiliser l’API REST avec l’authentification du principal de service pour effectuer l’appel `service.get_token()` régulièrement, comme indiqué précédemment.
>
> Nous vous recommandons vivement de créer votre espace de travail Azure Machine Learning dans la même région que celle de votre cluster Azure Kubernetes Service.
>
> Pour s’authentifier avec un jeton, le service web appelle la région dans laquelle votre espace de travail Azure Machine Learning est créé. Si la région de votre espace de travail est indisponible, vous ne pouvez pas extraire de jeton pour votre service web, même si votre cluster se trouve dans une région différente de celle de votre espace de travail. Ainsi, Azure AD Authentication n’est pas disponible tant que la région de votre espace de travail n’est pas à nouveau disponible.
>
> De plus, plus la distance entre la région de votre cluster et celle de votre espace de travail est élevée, plus l’extraction de jeton prend de temps.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’authentification auprès d’un modèle déployé, consultez [Créer un client pour un modèle déployé en tant que service web](how-to-consume-web-service.md).