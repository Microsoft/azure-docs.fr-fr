---
title: Tutoriel d’application web – Écrire une application web
description: Ce tutoriel décrit en détail un exemple de déploiement d’application web simple. Cette section du tutoriel vous guide dans l’écriture de l’application web.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.openlocfilehash: bb9c206a17a11f0cf710ac4ee3ac7ccae0fa9eb9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023293"
---
# <a name="write-azure-web-application-to-read-fhir-data"></a>Écrire une application web Azure pour lire des données FHIR
Maintenant que vous savez comment vous connecter à votre serveur FHIR et transmettre des données (POST), vous êtes prêt à écrire une application web capable de lire des données FHIR. Dans cette dernière étape du tutoriel, nous allons vous montrer dans le détail comment écrire et accéder à l’application web.

## <a name="create-web-application"></a>Créer l’application web
Dans Azure, sélectionnez **Créer une ressource** et **Application web**. Veillez à attribuer à l’application web le nom que vous avez spécifié dans l’URI de redirection pour votre application cliente, ou revenez en arrière et mettez à jour l’URI de redirection avec le nouveau nom. 

![Créer l’application web](media/tutorial-web-app/create-web-app.png)

Une fois l’application web disponible, choisissez **Accéder à la ressource**. Sélectionnez **Éditeur App Service (Préversion)** sous Outils de développement à droite, puis sélectionnez **Accéder**. Cela a pour effet d’ouvrir l’Éditeur App Service. Cliquez avec le bouton droit dans l’espace gris en dessous de *Explorer*, puis créez un fichier sous le nom **index.html**.

Vous pouvez entrer le code ci-après dans **index.html**. Vous aurez besoin des éléments suivantes :
* **clientId** – À mettre à jour avec l’ID de votre application cliente. Cet ID est le même que celui que vous avez extrait au moment de récupérer votre jeton
* **authority** – À mettre à jour avec votre ID de locataire Azure AD
* **FHIRendpoint** – À mettre à jour avec le nom de votre service FHIR
* **scopes** – À mettre à jour pour faire apparaître l’URL complète de votre audience

``` HTML

<!DOCTYPE html>
<html>

<head>
    <title>FHIR Patient browser sample app</title>
    <script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.0/js/msal.js"></script>
</head>

<body>
    <div class="leftContainer">
        <p id="WelcomeMessage">Welcome to the FHIR Patient browsing sample Application</p>
        <button id="SignIn" onclick="signIn()">Sign In</button>
    </div>

    <div id="patientTable">
    </div>

    <script>
        var msalConfig = {
            auth: {
                clientId: '<CLIENT-ID>',
                authority: "https://login.microsoftonline.com/<AZURE-AD-TENANT-ID>"
            },
            cache: {
                cacheLocation: "localStorage",
                storeAuthStateInCookie: true
            }
        }

        var FHIRConfig = {
            FHIRendpoint: "https://<FHIR-SERVER-NAME>.azurehealthcareapis.com"
        }
        var requestObj = {
            scopes: ["https://<FHIR-SERVER-NAME>.azurehealthcareapis.com/user_impersonation"]
        }

        function authRedirectCallBack(error, response) {
            if (error) {
                console.log(error);
            } else {
                if (response.tokenType === "access_token") {
                    callFHIRServer(FHIRConfig.FHIRendpoint + '/Patient', 'GET', null, response.accessToken, FHIRCallback);
                }
            }
        }

        var myMSALObj = new Msal.UserAgentApplication(msalConfig);
        myMSALObj.handleRedirectCallback(authRedirectCallBack);

        function signIn() {
            myMSALObj.loginPopup(requestObj).then(function (loginResponse) {
                showWelcomeMessage();
                acquireTokenPopupAndCallFHIRServer();
            }).catch(function (error) {
                console.log(error);
            })
        }

        function showWelcomeMessage() {
            var divWelcome = document.getElementById('WelcomeMessage');
            divWelcome.innerHTML = "Welcome " + myMSALObj.getAccount().userName + " to FHIR Patient Browsing App";
            var loginbutton = document.getElementById('SignIn');
            loginbutton.innerHTML = 'Sign Out';
            loginbutton.setAttribute('onclick', 'signOut()')
        }

        function signOut() {
            myMSALObj.logout();
        }

        function acquireTokenPopupAndCallFHIRServer() {
            myMSALObj.acquireTokenSilent(requestObj).then(function (tokenResponse) {
                callFHIRServer(FHIRConfig.FHIRendpoint + '/Patient', 'GET', null, tokenResponse.accessToken, FHIRCallback);
            }).catch(function (error) {
                console.log(error);
                if (requiresInteraction(error.errorCode)) {
                    myMSALObj.acquireTokenPopup(requestObj).then(function (tokenResponse) {
                        callFHIRServer(FHIRConfig.FHIRendpoint + '/Patient', 'GET', null, tokenResponse.accessToken, FHIRCallback);
                    }).catch(function (error) {
                        console.log(error);
                    })
                }
            });
        }

        function callFHIRServer(theUrl, method, message, accessToken, callBack) {
            var xmlHttp = new XMLHttpRequest();
            xmlHttp.onreadystatechange = function () {
                if (this.readyState == 4 && this.status == 200)
                    callBack(JSON.parse(this.responseText));
            }
            xmlHttp.open(method, theUrl, true);
            xmlHttp.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
            xmlHttp.setRequestHeader('Authorization', 'Bearer ' + accessToken);
            xmlHttp.send(message);
        }

        function FHIRCallback(data) {
            patientListHtml = '<ol>';
            data.entry.forEach(function(e) {
                patientListHtml += '<li>' + e.resource.name[0].family + ', ' + e.resource.name[0].given + ' (' + e.resource.id + ')';
            });
            patientListHtml += '</ol>';
            document.getElementById("patientTable").innerHTML = patientListHtml;
        }
    </script>
</body>

</html>
```

À partir de là, vous pouvez revenir à votre ressource d’application web et ouvrir l’URL figurant dans la page Vue d’ensemble. Connectez-vous pour constater la présence du patient James Tiberious Kirk que vous avez créé précédemment.

## <a name="next-steps"></a>Étapes suivantes
Vous êtes parvenu à déployer l’API Azure pour FHIR, à inscrire une application cliente publique, à tester l’accès et à créer une petite application web. Découvrez maintenant les fonctionnalités prises en charge par l’API Azure pour FHIR.

>[!div class="nextstepaction"]
>[Fonctionnalités prises en charge](fhir-features-supported.md)





