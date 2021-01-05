---
title: Ajouter ou mettre à jour votre secret d’extraction Red Hat sur un cluster Azure Red Hat OpenShift 4
description: Ajouter ou mettre à jour votre secret d’extraction Red Hat sur des clusters ARO 4.x existants
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: container-service
ms.topic: conceptual
ms.date: 05/21/2020
keywords: secret d’extraction, ARO, openshift, Red Hat
ms.openlocfilehash: 769b7589fb6496fc2f4123665ad1f6fe61d0cce2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89294745"
---
# <a name="add-or-update-your-red-hat-pull-secret-on-an-azure-red-hat-openshift-4-cluster"></a>Ajouter ou mettre à jour votre secret d’extraction Red Hat sur un cluster Azure Red Hat OpenShift 4

Ce guide décrit l’ajout ou la mise à jour de votre secret d’extraction Red Hat pour un cluster Azure Red Hat OpenShift (ARO) 4.x existant.

Si vous créez un cluster pour la première fois, vous pouvez ajouter votre secret d’extraction lors de la création de votre cluster. Pour plus d’informations sur la création d’un cluster ARO avec un secret d’extraction Red Hat, consultez [Créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md#get-a-red-hat-pull-secret-optional).

## <a name="before-you-begin"></a>Avant de commencer

Ce guide part du principe que vous disposez d’un cluster Azure Red Hat OpenShift 4 existant. Vérifiez que vous disposez d’un accès administrateur à votre cluster.

## <a name="prepare-your-pull-secret"></a>Préparer votre secret d’extraction
Lorsque vous créez un cluster ARO sans ajouter de secret d’extraction Red Hat, un secret d’extraction est toujours créé automatiquement sur votre cluster. Toutefois, ce secret d’extraction n’est pas entièrement rempli.

Cette section décrit la mise à jour de cette clé d’extraction avec des valeurs supplémentaires à partir de votre clé secrète d’extraction Red Hat.

1. Récupérez le secret nommé `pull-secret` dans l’espace de noms `openshift-config` et enregistrez-le dans un fichier distinct en exécutant la commande suivante : 

    ```console
    oc get secrets pull-secret -n openshift-config -o template='{{index .data ".dockerconfigjson"}}' | base64 -d > pull-secret.json
    ```

    Le résultat doit ressembler à ce qui suit. (Notez que la valeur réelle du secret a été supprimée.)

    ```json
    {
        "auths": {
            "arosvc.azurecr.io": {
                "auth": "<my-aroscv.azurecr.io-secret>"
            }
        }
    }
    ```

2. Accédez au [portail du gestionnaire de cluster Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) et sélectionnez **Télécharger le secret d’extraction**. Votre secret d’extraction Red Hat ressemble à ce qui suit. (Notez que les valeurs de secret réelles ont été supprimées.)

    ```json
    {
        "auths": {
            "cloud.openshift.com": {
                "auth": "<my-crc-secret>",
                "email": "klamenzo@redhat.com"
            },
            "quay.io": {
                "auth": "<my-quayio-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.connect.redhat.com": {
                "auth": "<my-registry.connect.redhat.com-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.redhat.io": {
                "auth": "<my-registry.redhat.io-secret>",
                "email": "klamenzo@redhat.com"
            }
        }
    }
    ```

3. Modifiez le fichier de secret d’extraction que vous avez obtenu à partir de votre cluster en ajoutant les entrées trouvées dans votre clé secrète d’extraction Red Hat. 

    > [!IMPORTANT]
    > L’inclusion de l’entrée `cloud.openshift.com` de votre secret d’extraction Red Hat entraînera le démarrage de l’envoi de données de télémétrie vers Red Hat par votre cluster. Incluez cette section uniquement si vous souhaitez envoyer des données de télémétrie. Dans le cas contraire, ignorez la section suivante.    
    > ```json
    > {
    >         "cloud.openshift.com": {
    >             "auth": "<my-crc-secret>",
    >             "email": "klamenzo@redhat.com"
    >         }
    > ```

    > [!CAUTION]
    > Ne supprimez pas et ne modifiez pas l’entrée `arosvc.azurecr.io` de votre secret d’extraction. Cette section est nécessaire pour que votre cluster fonctionne correctement.

    ```json
    "arosvc.azurecr.io": {
                "auth": "<my-aroscv.azurecr.io-secret>"
            }
    ```

    Votre fichier final doit ressembler à ce qui suit. (Notez que les valeurs de secret réelles ont été supprimées.)

    ```json
    {
        "auths": {
            "cloud.openshift.com": {
                "auth": "<my-crc-secret>",
                "email": "klamenzo@redhat.com"
            },
            "quay.io": {
                "auth": "<my-quayio-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.connect.redhat.com": {
                "auth": "<my-registry.connect.redhat.com-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.redhat.io": {
                "auth": "<my-registry.redhat.io-secret>",
                "email": "klamenzo@redhat.com"
            },
            "arosvc.azurecr.io": {
                "auth": "<my-aroscv.azurecr.io-secret>"
            }
        }
    }
    ```

4. Assurez-vous que le fichier est dans un format JSON valide. Il existe de nombreuses façons de valider votre fichier JSON. L’exemple suivant utilise jq:

    ```json
    cat pull-secret.json | jq
    ```

    > [!NOTE]
    > Si une erreur se produit dans le fichier, elle s’affiche ainsi : `parse error`.

## <a name="add-your-pull-secret-to-your-cluster"></a>Ajouter votre secret d’extraction à votre cluster

Exécutez la commande suivante pour mettre à jour votre secret d’extraction.

> [!NOTE]
> L’exécution de cette commande entraîne le redémarrage de vos nœuds de cluster un par un à mesure qu’ils sont mis à jour. 

```console
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=./pull-secret.json
```

Après avoir défini le secret, vous êtes prêt à activer les opérateurs certifiés Red Hat.

### <a name="modify-the-configuration-files"></a>Modifier le fichier de configuration

Modifiez les objets suivants pour activer les opérateurs Red Hat.

Tout d’abord, modifiez le fichier de configuration d’opérateur d’exemples. Vous pouvez ensuite modifier le fichier de configuration en exécutant la commande suivante :

```
oc edit configs.samples.operator.openshift.io/cluster -o yaml
```

Changez les valeurs `spec.architectures.managementState` et `status.architecture.managementState` de `Removed` à `Managed`. 

L’extrait de code YAML suivant montre uniquement les sections pertinentes du fichier YAML modifié :

```yaml
apiVersion: samples.operator.openshift.io/v1
kind: Config
metadata:
  
  ...
  
spec:
  architectures:
  - x86_64
  managementState: Managed
status:
  architectures:

  ...

  managementState: Managed
  version: 4.3.27
```

Ensuite, exécutez la commande suivante pour modifier le fichier de configuration de l’opérateur du hub :  

```console
oc edit operatorhub cluster -o yaml
```

Changez les valeurs `Spec.Sources.Disabled` et `Status.Sources.Disabled` de `true` à `false` pour toutes les sources à activer.

L’extrait de code YAML suivant montre uniquement les sections pertinentes du fichier YAML modifié :

```yaml
Name:         cluster

...
                 dd3310b9-e520-4a85-98e5-8b4779ee0f61
Spec:
  Sources:
    Disabled:  false
    Name:      certified-operators
    Disabled:  false
    Name:      redhat-operators
Status:
  Sources:
    Disabled:  false
    Name:      certified-operators
    Status:    Success
    Disabled:  false
    Name:      community-operators
    Status:    Success
    Disabled:  false
    Name:      redhat-operators
    Status:    Success
Events:        <none>
```

Enregistrez le fichier pour appliquer vos modifications.

## <a name="validate-that-your-secret-is-working"></a>Vérifier que votre clé secrète fonctionne

Après avoir ajouté votre secret d’extraction et modifié les fichiers de configuration appropriés, la mise à jour de votre cluster peut prendre plusieurs minutes. Pour vérifier que votre cluster a été mis à jour, exécutez la commande suivante pour afficher les opérateurs certifiés et les sources des opérateurs Red Hat disponibles :

```console
$ oc get catalogsource -A
NAMESPACE               NAME                  DISPLAY               TYPE   PUBLISHER   AGE
openshift-marketplace   certified-operators   Certified Operators   grpc   Red Hat     10s
openshift-marketplace   community-operators   Community Operators   grpc   Red Hat     18h
openshift-marketplace   redhat-operators      Red Hat Operators     grpc   Red Hat     11s
```

Si vous ne voyez pas les opérateurs certifiés et les opérateurs Red Hat, patientez quelques minutes et réessayez.

Pour vous assurer que votre secret d’extraction a été mis à jour et qu’il fonctionne correctement, ouvrez OperatorHub et recherchez un opérateur Red Hat vérifié. Par exemple, assurez-vous que l’opérateur de stockage de conteneurs OpenShift est disponible et vérifiez que vous disposez des autorisations nécessaires pour l’installer.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les secrets d’extraction Red Hat, consultez [Utiliser les secrets d’extraction d’image](https://docs.openshift.com/container-platform/4.5/openshift_images/managing_images/using-image-pull-secrets.html).

Pour en savoir plus sur Red Hat OpenShift 4, consultez [Azure Red Hat OpenShift 4](https://docs.openshift.com/aro/4/welcome/index.html).
