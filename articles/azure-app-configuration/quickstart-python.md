---
title: Démarrage rapide pour utiliser Azure App Configuration avec des applications Python | Microsoft Docs
description: Dans ce guide de démarrage rapide, créez une application Python avec Azure App Configuration pour centraliser le stockage et la gestion des paramètres d’application en dehors de votre code.
services: azure-app-configuration
author: drewbatgit
ms.service: azure-app-configuration
ms.topic: quickstart
ms.custom: devx-track-python
ms.date: 9/17/2020
ms.author: drewbat
ms.openlocfilehash: 954f4edcd10d701d00d9cd23280aaac7c287992d
ms.sourcegitcommit: 83610f637914f09d2a87b98ae7a6ae92122a02f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91997450"
---
# <a name="quickstart-create-a-python-app-with-azure-app-configuration"></a>Démarrage rapide : Créer une application Python avec Azure App Configuration

Dans ce guide de démarrage rapide, vous utilisez Azure App Configuration pour centraliser le stockage et la gestion des paramètres d’une application à l’aide de la [bibliothèque de client Azure App Configuration pour Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/appconfiguration/azure-appconfiguration).

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)
- Python 2.7 ou 3.5 ou version ultérieure : pour plus d’informations sur la configuration de Python sur Windows, consultez la [documentation de Python sur Windows]( https://docs.microsoft.com/windows/python/)

## <a name="create-an-app-configuration-store"></a>Créer un magasin App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Sélectionnez **Explorateur de configurations** > **Créer** > **Clé-valeur** pour ajouter les paires clé-valeur suivantes :

    | Clé | Valeur |
    |---|---|
    | TestApp:Settings:Message | Data from Azure App Configuration |

    Laissez **Étiquette** et **Type de contenu** vides pour l’instant.

8. Sélectionnez **Appliquer**.

## <a name="setting-up-the-python-app"></a>Configuration de l’application Python

1. Dans ce tutoriel, vous allez créer un répertoire pour le projet nommé *app-configuration-quickstart*.

    ```console
    mkdir app-configuration-quickstart
    ```

1. Basculez vers le répertoire *app-configuration-quickstart*.

    ```console
    cd app-configuration-quickstart
    ```

1. Installez la bibliothèque de client Azure App Configuration à l’aide de la commande `pip install`.

    ```console
    pip install azure-appconfiguration
    ```

1. Créez un fichier appelé *app-configuration-quickstart.py* dans le répertoire *app-configuration-quickstart*, puis ajoutez le code suivant :

    ```python
    import os
    from azure.appconfiguration import AzureAppConfigurationClient, ConfigurationSetting
    
    try:
        print("Azure App Configuration - Python Quickstart")
        # Quickstart code goes here
    except Exception as ex:
        print('Exception:')
        print(ex)
    ```

> [!NOTE]
> Les extraits de code de ce guide de démarrage rapide vous aideront à démarrer avec la bibliothèque de client App Configuration pour Python. Pour votre application, vous devez également envisager de gérer les exceptions en fonction de vos besoins. Pour en savoir plus sur la gestion des exceptions, consultez notre [documentation sur le SDK Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/appconfiguration/azure-appconfiguration).

## <a name="configure-your-app-configuration-connection-string"></a>Configurer votre chaîne de connexion App Configuration

1. Définissez une variable d’environnement nommée **AZURE_APP_CONFIG_CONNECTION_STRING** et affectez-lui la valeur de la clé d’accès à votre magasin App Configuration. Sur la ligne de commande, exécutez la commande suivante :

    ```cmd
    setx AZURE_APP_CONFIG_CONNECTION_STRING "connection-string-of-your-app-configuration-store"
    ```

    Si vous utilisez Windows PowerShell, exécutez la commande suivante :

    ```azurepowershell
    $Env:AZURE_APP_CONFIG_CONNECTION_STRING = "connection-string-of-your-app-configuration-store"
    ```

    Si vous utilisez macOS ou Linux, exécutez la commande suivante :

    ```console
    export AZURE_APP_CONFIG_CONNECTION_STRING='connection-string-of-your-app-configuration-store'
    ```

2. Redémarrez l’invite de commandes pour que la modification soit prise en compte. Imprimez la valeur de la variable d’environnement pour confirmer qu’elle est correctement définie.

## <a name="code-samples"></a>Exemples de code

Les exemples d’extraits de code de cette section vous montrent comment effectuer des opérations courantes avec la bibliothèque de client App Configuration pour Python. Ajoutez ces extraits de code au bloc `try` dans le fichier *app-configuration-quickstart.py* que vous avez créé.

> [!NOTE]
> La bibliothèque de client App Configuration fait référence à un objet clé/valeur sous la forme `ConfigurationSetting`. Ainsi, dans cet article, les **paires clé/valeur** dans le magasin App Configuration sont appelées **paramètres de configuration**.

* [Se connecter à un magasin App Configuration](#connect-to-an-app-configuration-store)
* [Obtenir un paramètre de configuration](#get-a-configuration-setting)
* [Ajouter un paramètre de configuration](#add-a-configuration-setting)
* [Obtenir la liste des paramètres de configuration](#get-a-list-of-configuration-settings)
* [Verrouiller un paramètre de configuration](#lock-a-configuration-setting)
* [Déverrouiller un paramètre de configuration](#unlock-a-configuration-setting)
* [Mettre à jour un paramètre de configuration](#update-a-configuration-setting)
* [Supprimer un paramètre de configuration](#delete-a-configuration-setting)

### <a name="connect-to-an-app-configuration-store"></a>Se connecter à un magasin App Configuration

L’extrait de code suivant crée une instance d’**AzureAppConfigurationClient** à l’aide de la chaîne de connexion stockée dans vos variables d’environnement.

```python
    connection_string = os.getenv('AZURE_APP_CONFIG_CONNECTION_STRING')
    app_config_client = AzureAppConfigurationClient.from_connection_string(connection_string)
```

### <a name="get-a-configuration-setting"></a>Obtenir un paramètre de configuration

L’extrait de code suivant récupère un paramètre de configuration par nom de clé (`key`).

```python
    retrieved_config_setting = app_config_client.get_configuration_setting(key='TestApp:Settings:Message')
    print("\nRetrieved configuration setting:")
    print("Key: " + retrieved_config_setting.key + ", Value: " + retrieved_config_setting.value)
```

### <a name="add-a-configuration-setting"></a>Ajouter un paramètre de configuration

L’extrait de code suivant crée un objet `ConfigurationSetting` avec les champs `key` et `value` et appelle la méthode `add_configuration_setting`. Cette méthode lève une exception si vous essayez d’ajouter un paramètre de configuration qui existe déjà dans votre magasin. Pour éviter cette exception, vous pouvez utiliser la méthode [set_configuration_setting](#update-a-configuration-setting) à la place.

```python
    config_setting = ConfigurationSetting(
        key='TestApp:Settings:NewSetting',
        value='New setting value'
    )
    added_config_setting = app_config_client.add_configuration_setting(config_setting)
    print("\nAdded configuration setting:")
    print("Key: " + added_config_setting.key + ", Value: " + added_config_setting.value)
```

### <a name="get-a-list-of-configuration-settings"></a>Obtenir la liste des paramètres de configuration

L’extrait de code suivant récupère la liste des paramètres de configuration. Vous pouvez fournir les arguments `key_filter` et `label_filter` pour filtrer les paires clé/valeur en fonction de `key` et `label` respectivement. Pour plus d’informations sur le filtrage, découvrez comment [interroger les paramètres de configuration](./concept-key-value.md#query-key-values).

```python
    filtered_settings_list = app_config_client.list_configuration_settings(key_filter="TestApp*")
    print("\nRetrieved list of configuration settings:")
    for item in filtered_settings_list:
        print("Key: " + item.key + ", Value: " + item.value)
```

### <a name="lock-a-configuration-setting"></a>Verrouiller un paramètre de configuration

L’état de verrouillage d’une paire clé/valeur dans App Configuration est indiqué par l’attribut `read_only` de l’objet `ConfigurationSetting`. Si `read_only` a pour valeur `True`, le paramètre est verrouillé. La méthode `set_read_only` peut être appelée avec l’argument `read_only=True` pour verrouiller le paramètre de configuration.

```python
    locked_config_setting = app_config_client.set_read_only(added_config_setting, read_only=True)
    print("\nRead-only status for " + locked_config_setting.key + ": " + str(locked_config_setting.read_only))
```

### <a name="unlock-a-configuration-setting"></a>Déverrouiller un paramètre de configuration

Si l’attribut `read_only` d’un objet `ConfigurationSetting` a pour valeur `False`, le paramètre est déverrouillé. La méthode `set_read_only` peut être appelée avec l’argument `read_only=False` pour déverrouiller le paramètre de configuration.

```python
    unlocked_config_setting = app_config_client.set_read_only(locked_config_setting, read_only=False)
    print("\nRead-only status for " + unlocked_config_setting.key + ": " + str(unlocked_config_setting.read_only))
```

### <a name="update-a-configuration-setting"></a>Mettre à jour un paramètre de configuration

La méthode `set_configuration_setting` peut être utilisée pour mettre à jour un paramètre existant ou créer un paramètre. L’extrait de code suivant change la valeur d’un paramètre de configuration existant.

```python
    added_config_setting.value = "Value has been updated!"
    updated_config_setting = app_config_client.set_configuration_setting(added_config_setting)
    print("\nUpdated configuration setting:")
    print("Key: " + updated_config_setting.key + ", Value: " + updated_config_setting.value)
```

### <a name="delete-a-configuration-setting"></a>Supprimer un paramètre de configuration

L’extrait de code suivant supprime un paramètre de configuration par nom de clé (`key`).

```python
    deleted_config_setting = app_config_client.delete_configuration_setting(key="TestApp:Settings:NewSetting")
    print("\nDeleted configuration setting:")
    print("Key: " + deleted_config_setting.key + ", Value: " + deleted_config_setting.value)
```

## <a name="run-the-app"></a>Exécuter l’application

Dans ce guide de démarrage rapide, vous avez créé une application Python qui utilise la bibliothèque de client Azure App Configuration pour récupérer un paramètre de configuration créé dans le portail Azure, ajouter un nouveau paramètre, récupérer la liste des paramètres existants, verrouiller et déverrouiller un paramètre, mettre à jour un paramètre et enfin supprimer un paramètre.

À ce stade, votre fichier *app-configuration-quickstart.py* doit contenir le code suivant :

```python
import os
from azure.appconfiguration import AzureAppConfigurationClient, ConfigurationSetting

try:
    print("Azure App Configuration - Python Quickstart")
    # Quickstart code goes here

    connection_string = os.getenv('AZURE_APP_CONFIG_CONNECTION_STRING')
    app_config_client = AzureAppConfigurationClient.from_connection_string(connection_string)

    retrieved_config_setting = app_config_client.get_configuration_setting(key='TestApp:Settings:Message')
    print("\nRetrieved configuration setting:")
    print("Key: " + retrieved_config_setting.key + ", Value: " + retrieved_config_setting.value)

    config_setting = ConfigurationSetting(
        key='TestApp:Settings:NewSetting',
        value='New setting value'
    )
    added_config_setting = app_config_client.add_configuration_setting(config_setting)
    print("\nAdded configuration setting:")
    print("Key: " + added_config_setting.key + ", Value: " + added_config_setting.value)

    filtered_settings_list = app_config_client.list_configuration_settings(key_filter="TestApp*")
    print("\nRetrieved list of configuration settings:")
    for item in filtered_settings_list:
        print("Key: " + item.key + ", Value: " + item.value)

    locked_config_setting = app_config_client.set_read_only(added_config_setting, read_only=True)
    print("\nRead-only status for " + locked_config_setting.key + ": " + str(locked_config_setting.read_only))

    unlocked_config_setting = app_config_client.set_read_only(locked_config_setting, read_only=False)
    print("\nRead-only status for " + unlocked_config_setting.key + ": " + str(unlocked_config_setting.read_only))

    added_config_setting.value = "Value has been updated!"
    updated_config_setting = app_config_client.set_configuration_setting(added_config_setting)
    print("\nUpdated configuration setting:")
    print("Key: " + updated_config_setting.key + ", Value: " + updated_config_setting.value)

    deleted_config_setting = app_config_client.delete_configuration_setting(key="TestApp:Settings:NewSetting")
    print("\nDeleted configuration setting:")
    print("Key: " + deleted_config_setting.key + ", Value: " + deleted_config_setting.value)

except Exception as ex:
    print('Exception:')
    print(ex)
```

Dans la fenêtre de votre console, accédez au répertoire contenant le fichier *app-configuration-quickstart.py* et exécutez la commande python suivante pour exécuter l’application :

```console
python app-configuration-quickstart.py
```

Vous devez normalement voir la sortie suivante.

```output
Azure App Configuration - Python Quickstart

Retrieved configuration setting:
Key: TestApp:Settings:Message, Value: Data from Azure App Configuration

Added configuration setting:
Key: TestApp:Settings:NewSetting, Value: New setting value

Retrieved list of configuration settings:
Key: TestApp:Settings:Message, Value: Data from Azure App Configuration
Key: TestApp:Settings:NewSetting, Value: New setting value

Read-only status for TestApp:Settings:NewSetting: True

Read-only status for TestApp:Settings:NewSetting: False

Updated configuration setting:
Key: TestApp:Settings:NewSetting, Value: Value has been updated!

Deleted configuration setting:
Key: TestApp:Settings:NewSetting, Value: Value has been updated!
```

## <a name="clean-up-resources"></a>Nettoyer les ressources


[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un magasin App Configuration et appris à accéder aux paires clé/valeur à partir d’une application Python.

Pour obtenir des exemples de code supplémentaires, visitez :

> [!div class="nextstepaction"]
> [Exemples pour la bibliothèque de client Azure App Configuration](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/appconfiguration/azure-appconfiguration/samples)