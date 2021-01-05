---
title: Utiliser OpenFaaS avec Azure Kubernetes Service (AKS)
description: Apprenez à déployer et utiliser OpenFaaS sur un cluster Azure Kubernetes Service (AKS) pour créer des fonctions serverless avec des conteneurs.
author: justindavies
ms.topic: conceptual
ms.date: 03/05/2018
ms.author: juda
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 319107127b79383fc3b49f0eeb856a0e6c5b09f8
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92747773"
---
# <a name="using-openfaas-on-aks"></a>Utilisation d’OpenFaaS avec AKS

[OpenFaaS][open-faas] est une infrastructure qui permet de créer des fonctions serverless grâce à l’utilisation de conteneurs. Ce projet open source a été très largement adopté par la communauté. Ce document décrit en détail l’installation et l’utilisation d’OpenFaas sur un cluster Azure Kubernetes Service (AKS).

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, vous avez besoin des éléments suivants :

* Des connaissances de base sur Kubernetes
* Un cluster Azure Kubernetes Service (AKS) et des informations d’identification AKS configurées sur votre système de développement.
* Azure CLI installé sur votre système de développement.
* L’installation des outils en ligne de commande Git sur votre système

## <a name="add-the-openfaas-helm-chart-repo"></a>Ajouter le dépôt de chart Helm OpenFaaS

Accédez à [https://shell.azure.com](https://shell.azure.com) pour ouvrir Azure Cloud Shell dans votre navigateur.

OpenFaaS gère ses propres charts Helm pour rester informé de toutes les dernières modifications.

```console
helm repo add openfaas https://openfaas.github.io/faas-netes/
helm repo update
```

## <a name="deploy-openfaas"></a>Déployer OpenFaaS

Il est recommandé de stocker OpenFaaS et ses fonctions dans leur propre espace de noms Kubernetes.

Créez un espace de noms pour les fonctions et le système OpenFaaS :

```console
kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml
```

Générez un mot de passe pour le portail de l’interface utilisateur OpenFaaS et l’API REST :

```console
# generate a random password
PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)

kubectl -n openfaas create secret generic basic-auth \
--from-literal=basic-auth-user=admin \
--from-literal=basic-auth-password="$PASSWORD"
```

Vous pouvez obtenir la valeur du secret avec `echo $PASSWORD`.

Le mot de passe que nous créons ici sera utilisé par le chart Helm pour activer l’authentification de base sur la passerelle OpenFaaS, qui est exposée à Internet via un équilibreur de charge cloud.

Un graphique Helm pour OpenFaaS est inclus dans le dépôt cloné. Utilisez ce graphique pour déployer OpenFaaS dans votre cluster AKS.

```console
helm upgrade openfaas --install openfaas/openfaas \
    --namespace openfaas  \
    --set basic_auth=true \
    --set functionNamespace=openfaas-fn \
    --set serviceType=LoadBalancer
```

Sortie :

```output
NAME:   openfaas
LAST DEPLOYED: Wed Feb 28 08:26:11 2018
NAMESPACE: openfaas
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                 DATA  AGE
prometheus-config    2     20s
alertmanager-config  1     20s

{snip}

NOTES:
To verify that openfaas has started, run:

```console
kubectl --namespace=openfaas get deployments -l "release=openfaas, app=openfaas"
```

Une adresse IP publique est créée pour accéder à la passerelle OpenFaaS. Pour récupérer cette adresse IP, utilisez la commande [kubectl get service][kubectl-get]. L’affectation de l’adresse IP au service peut prendre un certain temps.

```console
kubectl get service -l component=gateway --namespace openfaas
```

Sortie :

```output
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
gateway            ClusterIP      10.0.156.194   <none>         8080/TCP         7m
gateway-external   LoadBalancer   10.0.28.18     52.186.64.52   8080:30800/TCP   7m
```

Pour tester le système OpenFaaS, accédez à l’adresse IP externe sur le port 8080 (`http://52.186.64.52:8080` dans cet exemple). Vous serez invité à vous connecter. Pour extraire votre mot de passe, entrez `echo $PASSWORD`.

![Interface utilisateur d’OpenFaaS](media/container-service-serverless/openfaas.png)

Enfin, installez l’interface CLI d’OpenFaaS. Cet exemple utilisait brew, consultez la[ documentation OpenFaaS CLI][open-faas-cli] pour plus d’options.

```console
brew install faas-cli
```

Affectez à `$OPENFAAS_URL` l’adresse IP publique identifiée précédemment.

Connectez-vous avec l’interface de ligne de commande Azure :

```console
export OPENFAAS_URL=http://52.186.64.52:8080
echo -n $PASSWORD | ./faas-cli login -g $OPENFAAS_URL -u admin --password-stdin
```

## <a name="create-first-function"></a>Créer la première fonction

Maintenant qu’OpenFaaS est opérationnel, créez une fonction à l’aide du portail OpenFaas.

Cliquez sur **Deploy New Function** (Déployer une nouvelle fonction), puis recherchez **Figlet** . Sélectionnez la fonction Figlet, puis cliquez sur **Deploy** (Déployer).

![Capture d'écran représentant la boîte de dialogue Déployer une nouvelle fonction, avec le texte Figlet sur la ligne de recherche.](media/container-service-serverless/figlet.png)

Utilisez curl pour appeler la fonction. Dans l’exemple suivant, remplacez l’adresse IP par celle de votre passerelle OpenFaas.

```console
curl -X POST http://52.186.64.52:8080/function/figlet -d "Hello Azure"
```

Sortie :

```output
 _   _      _ _            _
| | | | ___| | | ___      / \    _____   _ _ __ ___
| |_| |/ _ \ | |/ _ \    / _ \  |_  / | | | '__/ _ \
|  _  |  __/ | | (_) |  / ___ \  / /| |_| | | |  __/
|_| |_|\___|_|_|\___/  /_/   \_\/___|\__,_|_|  \___|

```

## <a name="create-second-function"></a>Créer la deuxième fonction

À présent, créez la deuxième fonction. Cet exemple est déployé à l’aide de l’interface CLI d’OpenFaaS. Il inclut une image de conteneur personnalisée et récupère des données à partir de Cosmos DB. Plusieurs éléments doivent être configurés avant la création de la fonction.

D’abord, créez un groupe de ressources pour Cosmos DB.

```azurecli-interactive
az group create --name serverless-backing --location eastus
```

Déployez une instance CosmosDB de type `MongoDB`. L’instance doit avoir un nom unique. Vous devez donc remplacer `openfaas-cosmos` par un nom unique dans votre environnement.

```azurecli-interactive
az cosmosdb create --resource-group serverless-backing --name openfaas-cosmos --kind MongoDB
```

Récupérez la chaîne de connexion de base de données Cosmos et stockez-la dans une variable.

Remplacez la valeur de l’argument `--resource-group` par le nom de votre groupe de ressources, et la valeur de l’argument `--name` par le nom de votre instance Cosmos DB.

```azurecli-interactive
COSMOS=$(az cosmosdb list-connection-strings \
  --resource-group serverless-backing \
  --name openfaas-cosmos \
  --query connectionStrings[0].connectionString \
  --output tsv)
```

Maintenant, remplissez l’instance Cosmos DB avec des données de test. Créez un fichier nommé `plans.json` et copiez-y le code json suivant.

```json
{
    "name" : "two_person",
    "friendlyName" : "Two Person Plan",
    "portionSize" : "1-2 Person",
    "mealsPerWeek" : "3 Unique meals per week",
    "price" : 72,
    "description" : "Our basic plan, delivering 3 meals per week, which will feed 1-2 people.",
    "__v" : 0
}
```

Utilisez l’outil *mongoimport* pour charger des données dans l’instance CosmosDB.

Si nécessaire, installez les outils MongoDB. L’exemple suivant installe ces outils à l’aide de brew. Pour d’autres options, consultez la [documentation MongoDB][install-mongo].

```console
brew install mongodb
```

Chargez les données dans la base de données.

```console
mongoimport --uri=$COSMOS -c plans < plans.json
```

Sortie :

```output
2018-02-19T14:42:14.313+0000    connected to: localhost
2018-02-19T14:42:14.918+0000    imported 1 document
```

Exécutez la commande suivante pour créer la fonction : Remplacez la valeur de l’argument `-g` par l’adresse de votre passerelle OpenFaaS.

```console
faas-cli deploy -g http://52.186.64.52:8080 --image=shanepeckham/openfaascosmos --name=cosmos-query --env=NODE_ENV=$COSMOS
```

Une fois déployée, vous devez voir le point de terminaison OpenFaaS de la fonction que vous venez de créer.

```output
Deployed. 202 Accepted.
URL: http://52.186.64.52:8080/function/cosmos-query
```

Testez la fonction à l’aide de curl. Remplacez l’adresse IP par l’adresse de la passerelle OpenFaaS.

```console
curl -s http://52.186.64.52:8080/function/cosmos-query
```

Sortie :

```json
[{"ID":"","Name":"two_person","FriendlyName":"","PortionSize":"","MealsPerWeek":"","Price":72,"Description":"Our basic plan, delivering 3 meals per week, which will feed 1-2 people."}]
```

Vous pouvez également tester la fonction dans l’interface utilisateur d’OpenFaaS.

![texte de remplacement](media/container-service-serverless/OpenFaaSUI.png)

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez continuer à apprendre avec l’atelier OpenFaaS via un ensemble de travaux pratiques couvrant des sujets tels que la création de votre propre bot GitHub, l’utilisation de secrets, l’affichage de métriques et la mise à l’échelle automatique.

<!-- LINKS - external -->
[install-mongo]: https://docs.mongodb.com/manual/installation/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[open-faas]: https://www.openfaas.com/
[open-faas-cli]: https://github.com/openfaas/faas-cli
[openfaas-workshop]: https://github.com/openfaas/workshop
