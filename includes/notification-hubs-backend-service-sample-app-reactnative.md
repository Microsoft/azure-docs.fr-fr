---
author: alexeystrakh
ms.author: alstrakh
ms.date: 06/11/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: c8271cc4de558ec5c97f41d6a070f71a9fc49cd2
ms.sourcegitcommit: bcb962e74ee5302d0b9242b1ee006f769a94cfb8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/07/2020
ms.locfileid: "86060436"
---
### <a name="create-the-react-native-solution"></a>Créer la solution React Native

1. Dans `Terminal`, mettez à jour vos outils d’environnement nécessaires pour utiliser React Native à l’aide des commandes suivantes :

    ```bash
    # install node
    brew install node
    # or update
    brew update node
    # install watchman
    brew install watchman
    # or update
    brew upgrade watchman
    # install cocoapods
    sudo gem install cocoapods
    ```

1. Dans `Terminal`, si vous avez l’interface CLI `React Native` installée, exécutez la commande suivante pour la désinstaller. Utilisez `npx` pour accéder automatiquement à la dernière version de l’interface CLI React Native disponible :

    ```bash
    npm uninstall -g react-native-cli
    ```

    > [!NOTE]
    > React Native a une interface de ligne de commande intégrée. Plutôt que d’installer et de gérer globalement une version spécifique de l’interface CLI, nous vous recommandons d’accéder à la version actuelle au moment de l’exécution à l’aide de `npx`, fourni avec Node.js. Avec `npx react-native <command>`, la version stable actuelle de l’interface CLI sera téléchargée et exécutée au moment de l’exécution de la commande.

1. Accédez à votre dossier de projets dans lequel vous souhaitez créer la nouvelle application. Utilisez le modèle basé sur TypeScript en spécifiant le paramètre `--template` :

    ```bash
    # init new project with npx
    npx react-native init PushDemo --template react-native-template-typescript
    ```

1. Exécutez le serveur Metro, qui génère des bundles JavaScript et surveille toutes les mises à jour du code pour actualiser les bundles en temps réel :

    ```bash
    cd PushDemo
    npx react-native start
    ```

1. Exécutez l’application iOS pour vérifier la configuration. Veillez à démarrer un simulateur iOS ou à connecter un appareil iOS, avant d’exécuter la commande suivante :

    ```bash
    npx react-native run-ios
    ```

1. Exécutez l’application Android pour vérifier la configuration. Quelques étapes supplémentaires sont nécessaires pour configurer un émulateur ou un appareil Android afin de pouvoir accéder au serveur Metro dans React Native. Les commandes suivantes génèrent un bundle JavaScript initial pour Android et le placent dans le dossier de ressources.

    ```bash
    # create assets folder for the bundle
    mkdir android/app/scr/main/assets
    # build the bundle
    npx react-native bundle --platform android --dev true --entry-file index.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res
    # enable ability for sim to access the localhost
    adb reverse tcp:8081 tcp:8081
    ```

    Ce script sera prédéployé avec la version initiale de l’application. Une fois déployé, configurez votre émulateur ou appareil pour accéder au serveur Metro en spécifiant l’adresse IP du serveur. Exécutez la commande suivante pour générer et exécuter l’application Android :

    ```bash
    npx react-native run-android
    ```

    Une fois dans l’application, appuyez sur `CMD+M` (émulateur) ou secouez l’appareil pour renseigner les paramètres du développeur, accédez à `Settings` > `Change Bundle Location` et spécifiez l’adresse IP du serveur Metro avec le port par défaut : `<metro-server-ip-address>:8081`.

1. Dans le fichier `App.tsx`, appliquez toute modification à la mise en page, enregistrez-la et la modification est automatiquement reflétée dans les applications iOS et Android.

    > [!NOTE]
    > Un guide de configuration détaillé de l’environnement de développement est disponible dans la [documentation officielle](https://reactnative.dev/docs/environment-setup)

### <a name="install-required-packages"></a>Installer les packages nécessaires

Pour que cet exemple fonctionne, vous avez besoin des trois packages suivants :

1. [Notifications Push iOS dans React Native](https://www.npmjs.com/package/@react-native-community/push-notification-ios) - [Projet GitHub](https://github.com/react-native-community/push-notification-ios)

    Ce package a été créé quand PushNotificationIOS a été fractionné du cœur de React Native. Le package implémente en mode natif des notifications Push pour iOS et fournit une interface React Native pour y accéder. Exécutez la commande suivante pour installer le package :

    ```bash
    yarn add @react-native-community/push-notification-ios
    ```

1. [Multiplateforme de notifications Push React Native](https://www.npmjs.com/package/react-native-push-notification)

    Ce package implémente des notifications locales et distantes sur iOS et Android en mode multiplateforme. Exécutez la commande suivante pour installer le package :

    ```bash
    yarn add react-native-push-notification
    ```

1. [Package d’informations sur l’appareil](https://www.npmjs.com/package/react-native-device-info) Le package fournit des informations sur un appareil en cours d’exécution. Utilisez-le pour définir un identificateur d’appareil, qui est utilisé pour l’inscription à une notification Push. Exécutez la commande suivante pour installer le package :

    ```bash
    yarn add react-native-device-info
    ```

### <a name="implement-the-cross-platform-components"></a>Implémenter les composants multiplateforme

1. Créez et implémentez `DemoNotificationHandler` :

    ```typescript
    import PushNotification from 'react-native-push-notification';

    class DemoNotificationHandler {
      private _onRegister: any;
      private _onNotification: any;

      onNotification(notification: any) {
        console.log('NotificationHandler:', notification);

        if (typeof this._onNotification === 'function') {
          this._onNotification(notification);
        }
      }

      onRegister(token: any) {
        console.log('NotificationHandler:', token);

        if (typeof this._onRegister === 'function') {
          this._onRegister(token);
        }
      }

      attachTokenReceived(handler: any) {
        this._onRegister = handler;
      }

      attachNotificationReceived(handler: any) {
        this._onNotification = handler;
      }
    }

    const handler = new DemoNotificationHandler();

    PushNotification.configure({
      onRegister: handler.onRegister.bind(handler),
      onNotification: handler.onNotification.bind(handler),
      permissions: {
        alert: true,
        badge: true,
        sound: true,
      },
      popInitialNotification: true,
      requestPermissions: true,
    });

    export default handler;
    ```

1. Créez et implémentez `DemoNotificationService` :

    ```typescript
    import PushNotification from 'react-native-push-notification';
    import DemoNotificationHandler from './DemoNotificationHandler';

    export default class DemoNotificationService {
      constructor(onTokenReceived: any, onNotificationReceived: any) {
        DemoNotificationHandler.attachTokenReceived(onTokenReceived);
        DemoNotificationHandler.attachNotificationReceived(onNotificationReceived);
        PushNotification.getApplicationIconBadgeNumber(function(number: number) {
          if(number > 0) {
            PushNotification.setApplicationIconBadgeNumber(0);
          }
        });
      }

      checkPermissions(cbk: any) {
        return PushNotification.checkPermissions(cbk);
      }

      requestPermissions() {
        return PushNotification.requestPermissions();
      }

      cancelNotifications() {
        PushNotification.cancelLocalNotifications();
      }

      cancelAll() {
        PushNotification.cancelAllLocalNotifications();
      }

      abandonPermissions() {
        PushNotification.abandonPermissions();
      }
    }
    ```

1. Créez et implémentez `DemoNotificationRegistrationService` :

    ```typescript
    export default class DemoNotificationService {
        constructor(
            readonly apiUrl: string,
            readonly apiKey: string) {
        }

    async registerAsync(request: any): Promise<Response> {
            const method = 'PUT';
            const registerApiUrl = `${this.apiUrl}/notifications/installations`;
            const result = await fetch(registerApiUrl, {
                method: method,
                headers: {
                    Accept: 'application/json',
                    'Content-Type': 'application/json',
                    'apiKey': this.apiKey
                },
                body: JSON.stringify(request)
            });

            this.validateResponse(registerApiUrl, method, request, result);
            return result;
        }

        async deregisterAsync(deviceId: string): Promise<Response> {
            const method = 'DELETE';
            const deregisterApiUrl = `${this.apiUrl}/notifications/installations/${deviceId}`;
            const result = await fetch(deregisterApiUrl, {
                method: method,
                headers: {
                    Accept: 'application/json',
                    'Content-Type': 'application/json',
                    'apiKey': this.apiKey
                }
            });

            this.validateResponse(deregisterApiUrl, method, null, result);
            return result;
        }

        private validateResponse(requestUrl: string, method: string, requestPayload: any, response: Response) {
            console.log(`Request: ${method} ${requestUrl} => ${JSON.stringify(requestPayload)}\nResponse: ${response.status}`);
            if (!response || response.status != 200) {
                throw `HTTP error ${response.status}: ${response.statusText}`;
            }
        }
    }
    ```

1. Configurez l’application. Ouvrez `package.json` et ajoutez la définition de script suivante :

    ```json
    "configure": "cp .app.config.tsx src/config/AppConfig.tsx"
    ```

    Exécutez ensuite ce script, qui copiera la configuration par défaut dans le dossier `config`.

    ```bash
    yarn configure
    ```

    La dernière étape consiste à mettre à jour le fichier de configuration copié à l’étape précédente avec les informations d’accès à l’API. Spécifiez les paramètres `apiKey` et `apiUrl` :

    ```typescript
    module.exports = {
        appName: "PushDemo",
        env: "production",
        apiUrl: "https://<azure-push-notifications-api-url>/api/",
        apiKey: "<api-auth-key>",
    };
    ```

### <a name="implement-the-cross-platform-ui"></a>Implémenter l’interface utilisateur multiplateforme

1. Définissez la mise en page

    ```typescript
    <View style={styles.container}>
      {this.state.isBusy &&
        <ActivityIndicator></ActivityIndicator>
      }
      <View style={styles.button}>
        <Button title="Register" onPress={this.onRegisterButtonPress.bind(this)} disabled={this.state.isBusy} />
      </View>
      <View style={styles.button}>
        <Button title="Deregister" onPress={this.onDeregisterButtonPress.bind(this)} disabled={this.state.isBusy} />
      </View>
    </View>
    ```

1. Appliquez les styles

    ```css
    const styles = StyleSheet.create({
      container: {
        flex: 1,
        alignItems: "center",
        justifyContent: 'flex-end',
        margin: 50,
      },
      button: {
        margin: 5,
        width: "100%",
      }
    });
    ```

1. Initialisez le composant de page

    ```typescript
      state: IState;
      notificationService: DemoNotificationService;
      notificationRegistrationService: DemoNotificationRegistrationService;
      deviceId: string;

      constructor(props: any) {
        super(props);
        this.deviceId = DeviceInfo.getUniqueId();
        this.state = {
          status: "Push notifications registration status is unknown",
          registeredOS: "",
          registeredToken: "",
          isRegistered: false,
          isBusy: false,
        };

        this.notificationService = new DemoNotificationService(
          this.onTokenReceived.bind(this),
          this.onNotificationReceived.bind(this),
        );

        this.notificationRegistrationService = new DemoNotificationRegistrationService(
          Config.apiUrl,
          Config.apiKey,
        );
      }
    ```

1. Définissez des gestionnaires de clic de bouton

    ```typescript
      async onRegisterButtonPress() {
        if (!this.state.registeredToken || !this.state.registeredOS) {
          Alert.alert("The push notifications token wasn't received.");
          return;
        }

        let status: string = "Registering...";
        let isRegistered = this.state.isRegistered;
        try {
          this.setState({ isBusy: true, status });
          const pnPlatform = this.state.registeredOS == "ios" ? "apns" : "fcm";
          const pnToken = this.state.registeredToken;
          const request = {
            installationId: this.deviceId,
            platform: pnPlatform,
            pushChannel: pnToken,
            tags: []
          };
          const response = await this.notificationRegistrationService.registerAsync(request);
          status = `Registered for ${this.state.registeredOS} push notifications`;
          isRegistered = true;
        } catch (e) {
          status = `Registration failed: ${e}`;
        }
        finally {
          this.setState({ isBusy: false, status, isRegistered });
        }
      }

      async onDeregisterButtonPress() {
        if (!this.notificationService)
          return;

        let status: string = "Deregistering...";
        let isRegistered = this.state.isRegistered;
        try {
          this.setState({ isBusy: true, status });
          await this.notificationRegistrationService.deregisterAsync(this.deviceId);
          status = "Deregistered from push notifications";
          isRegistered = false;
        } catch (e) {
          status = `Deregistration failed: ${e}`;
        }
        finally {
          this.setState({ isBusy: false, status, isRegistered });
        }
      }
    ```

1. Traitez les notifications Push et les inscriptions de jetons reçues

    ```typescript
      onTokenReceived(token: any) {
        console.log(`Received a notification token on ${token.os}`);
        this.setState({ registeredToken: token.token, registeredOS: token.os, status: `The push notifications token has been received.` });

        if (this.state.isRegistered && this.state.registeredToken && this.state.registeredOS) {
          this.onRegisterButtonPress();
        }
      }

      onNotificationReceived(notification: any) {
        console.log(`Received a push notification on ${this.state.registeredOS}`);
        this.setState({ status: `Received a push notification...` });

        if (notification.data.message) {
          Alert.alert(AppConfig.appName, `${notification.data.action} action received`);
        }
      }
    };
    ```