---
title: "Use Remote Notifications"
#category: "Native Mobile"
#parent: "native-mobile"
#menu_order: 11
#description: "Learn to set up native push notifications with the Native builder."
#tags: ["mobile", "push notification", "remote", "push", "notification"]
---

## 1 Introduction

remote notifications only work with apps created through the Native Builder. You cannot send notifications to an app inside the Make it Native app.

## 2 Prerequisites

If you want to use push notifications with custom apps which created with native builder, make sure you have completed the following prerequisites:

* Setting up native push notifications with native builder [Native Builder](https://docs.mendix.com/howto/mobile/native-builder#1-introduction) 

## 3 Setting Up Your Mendix Project

Create a Mendix project using the Native Mobile Quickstart starter app. [todo add link]

### 3.1 Installing Your Module 

1. Add the [Community Commons](https://appstore.home.mendix.com/link/app/170/) module to your app project.
2. Add the [Encryption] module to your module [todo add link].
3. Set the encryption private key by doing the following:<br />
	a. Double-click **Settings**.<br />
	b. Click **Edit**.<br />
	c. Click **Constants**.<br />
	d. Click **New**.<br />
	e. Type *encryptionkey*, select it, and add a 16 character **Value**:

	![Capabilities](attachments/native-remote-push/modeler/setEncryption.png)

3. Add the [Push Notifications Connector](https://appstore.home.mendix.com/link/app/3003/) module to your app project.

### 3.2 Set Up a Notification Widget

1. Drag and drop an app events widget onto your app's home page, double-click it, and do the following:<br />
    a. Set **Page load** > **On load** to **Call a nanoflow**, then specify **PushNotifications.OnPageLoad_RegisterPushNotifications**.<br />
    b. Set **App resume** > **On resume** to **Call a nanoflow**, then specify **PushNotifications.OnPageLoad_RegisterPushNotifications**:<br />
    
    ![AppEvents](attachments/native-remote-push/modeler/AppEvents.png)

    This configured app events widget will allow for user devices to register with your notification interface, so that you can choose who to send push notifications to. Specifically it will register devices when they open the app or resume the app.

2. Create an entity called *NativeNotification* in your domain model with one `objectGUID` field:

    ![NotificationEntity](attachments/native-remote-push/modeler/NotificationEntity.png)

3. Create a new *DS_Notification* nanoflow which creates a **NativeNotification** entity object and then returns it:

    ![DS_Notification](attachments/native-remote-push/modeler/DS_Notification.png)

4. Drag and drop a data view widget onto your home page, set its **Data source** > **Nanoflow** to **DS_Notification**:

    ![Dataview](attachments/native-remote-push/modeler/Dataview.png)

5. Inside of this data view, drag and drop a notifications widget.
6. Set the notifications widget's GUID to **NativeNotification.objectGUID**:

    ![NotificationsGUID](attachments/native-remote-push/modeler/NotificationsGUID.png)

    This will allow you to pass objects with notifications.

7.  Open **Navigation**, in the **Responsive** pane click **New Item**, then add a new **Show page** item **PushNotifications/_USE ME/Administration**: 

    ![ProfileHomePage](attachments/native-remote-push/modeler/ProfileHomePage.png)

### 3.3 Adding Actions to Your Notifications Widget

To set up actions which will occur after tapping or receiving a notification, do the following:

1. Create two nanoflows (*ACT_OnRecieve* and *ACT_OnOpen*) which will create two different logs (**onRecieve triggered** and **onOpen triggered**):

    ![ACT_OnRecieve](attachments/native-remote-push/modeler/ACT_OnRecieve.png)

2. Double-click your notifications widget and do the following:<br />
    a. Add an action called *logIt*.<br />
    b. For **On recieve** select **ACT_OnRecieve**.<br />
    c. For **On open** select **ACT_OnOpen**:

    ![LogitAction](attachments/native-remote-push/modeler/logitAction.png)

### 3.4 Adding Firebase Configurations

Deploy your project and open your administration page in a web browser. Do the following:

1. Add a new FCM configuration.
2. Select **Enabled**.
3. Name your configuration.
4. Set it as **Development** (this will not affect any functionality, it is only a label).
5. Set the **Project id** to the project id you set in the [Native Apps] section of *How to Set Up the Google Firebase Cloud Messaging Server* [todo: fix link].
6.  Upload your private key (which you got in the [SOMETHING] section of *Setting up Remote Push Notifications* [todo: fix link]:

    ![FCMConfig](attachments/native-remote-push/modeler/FCMConfig.png)

7. For both your Android and iOS **Messaging service settings**, select your FCM configuration.
8.  For the Android and iOS **Messaging service types**, select FCM:

    ![FCMConfig2](attachments/native-remote-push/modeler/FCMConfig2.png)

Next you will test the implementation of your configurations.

### 3.5 Sending a Push Notification

1. Reload the app on your phone.
2. Put the app in the background by returning to your phone's home screen.
3. On your desktop browser, open the **Administration** page and click the **Devices** tab.

Now you should be able to see registered devices (which is probably only one: your testing device):

1. Click your device listed under **Registered Devices** and click **New Message**.
2.  Type some text into **Title** and **Body** fields, and in **Action name** type *logIt*:

    ![SimpleMessage](attachments/native-remote-push/modeler/SimpleMessage.png)

3. Click **Send**.

You will see your notification with the text you configured:

    ![PushRecieved](attachments/native-remote-push/modeler/PushRecieved.png)

1. Tap the notification. You will see a log message in your modeler console: **onOpen triggered**.
2. Now send and tap a notification while keeping the app open. You will see a different log in your modeler console: **onRecieve triggered**. 

## 4 Sending Data Using Push Notifications

Imagine your business has several products, and you want to send one product to a user using an administration module interface. How would you achieve this goal?

In this section you will learn the following:

* How to show a push notification to a user if their app is in the backgroud — when a user taps it, they will be brought to a product page
* How to show a small view to a user if their app is in the foreground — when a user taps the button in the animation, they will be brought to a product page

### 4.1 Push notificaiton when app is in background

#### 4.1.1 Setting Up an Example Entity

1. Add `Product` entity with `ProductName` attribute 
2.  Right-click `ProductName` [todo: redo language to match product]to generate overview pages  => `Product_NewEdit`, `Product_Overview`

    ![GeneratePages](attachments/native-remote-push/modeler/GeneratePages.png)

    ![GeneratePages](attachments/native-remote-push/modeler/GeneratePages2.png)

3. Drag and drop `Product_Overview` onto your app's homepage to make it more accessible. [todo: do you mean easier to nav to?]
4. Create a native page called `NativeProductOverview` that has a dataview which listens contexts with entity: Product. Fill the contents => This page will be opened with proper product when user taps the notification

![NativeProductOverview](attachments/native-remote-push/modeler/NativeProductOverview.png)

#### 4.1.2 Synchronizing Unused Entities [todo clarify title and section]

Studio Pro does smart data syncing, meaning if an entity has not been retrieved in native side, it will not be there. This situation will not occur in most apps since Studio Pro does retrieve entities which you want to show. [todo: add link to relavant doc link. Offline First?]

Your case does not retrieve any products in any of the pages [todo "which means you risk not seeing X"]. Fix this by doing the following:

1. Create a list of `Products` [todo: is this a list of data objects?] in one of your app's native pages. The data source does not matter since the list is bound to retrieve the entities
2. Change Navigation/Native mobile/ Sync config/ Product => Download All Object

    ![SyncConfig](attachments/native-remote-push/modeler/SyncConfig.png)

#### 4.1.3 Getting the GUIDs of the Objects in Edit View

For an example we want to keep the things simple:

1.  Create a nanoflow `ACT_GetGUIDAndLog` following these steps:<br />
    a. Product object as a parameter.<br />
    b. Javascript action Get guid, set the object the Parameter object.<br />
    c. Log the returned value:
    
    ![ACT_GetGUIDAndLog](attachments/native-remote-push/modeler/ACT_GetGUIDAndLog.png)

2.  Drag and drop this nanoflow to the `Product_NewEdit` inside of the Dataview:

    ![getGUIdAndLogButton](attachments/native-remote-push/modeler/getGUIdAndLogButton.png)

#### 4.1.4 Creating a Data Passing Nanoflow

1.  Create a nanoflow `ACT_GetProductAndShowPage` following these steps:<br />
    a. Notification object as a parameter<br />
    
    ![getGUIdAndLogButton](attachments/native-remote-push/modeler/ACT_GetProductAndShowPage.png)
    
    b. JS action `get object from a GUID` where `Entity type` is `Product` and GUID is `parameter/objectGUID` name the return value to `ProductObject`<br />
    
    ![getGUIdAndLogButton](attachments/native-remote-push/modeler/ACT_GetProductAndShowPage2.png)
    
    c. Show `NativeProductOverview` page with passed object: `ProductObject`
    
    ![getGUIdAndLogButton](attachments/native-remote-push/modeler/ACT_GetProductAndShowPage3.png)

2.  Go to your Home_Native/ Notification widget and create new action named `sendProduct`, on open triggers `ACT_GetProductAndShowPage`

    ![pushSendProduct](attachments/native-remote-push/modeler/pushSendProduct.png)

#### 4.1.5 Testing the Implementation

1. Get a Product GUID by clicking the button [todo: rename to "click X"?] you created in [Get the GUIDs of the objects in Edit view](todo: set anchor link)

2. Follow the steps in the [simple push notification](#sending-simple-push-notification) section above again, but with the following changes:<br />
    a. Set the action name to `sendProduct`<br />
    b. Set `Context object guid` to the GUID you got

    ![openProductPage](attachments/native-remote-push/modeler/openProductPage.png)

3. Put the app in the backgorund, send the message, and tap the notification. This will navigate to the `NativeProductOverview` page with the proper object.

## 4.2 Now lets cover when the app is in the foreground [todo: fix title]

1.  Add one more `boolean` field named `showNotification` to the `NativePush`:

    ![showNotification](attachments/native-remote-push/modeler/showNotification.png)

2. In your `Home_Native` page inside of the NativeNotification Dataview:<br />
    a. add a Container.<br />
    b. Sets its visibility to `NativeNotification/showNotification`.<br />
    c. Add a text field saying `You have recieved a product`.<br />
    d. Drag and drop `ACT_GetProductAndShowPage` nanoflow next to it.

    ![ContainerVisibility](attachments/native-remote-push/modeler/ContainerVisibility.png)

3. Create a nanoflow called `ACT_ShowNotificationOnRecieve` which will be responsible for switching `NativeNotification/showNotification` attribute:<br />
![ACT_ShowNotificationOnRecieve](attachments/native-remote-push/modeler/ACT_ShowNotificationOnRecieve.png)
    a. NativeNotification as a parameter.<br />
    b. Change the `NativeNotification/showNotification` to `true`, without committing.<br />
    c. Javascript action `Wait` for `5000` ms.<br />
    d. Change the `NativeNotification/showNotification` to `false`, without committing.

4. Home_Native/ Notification widget => Change action named `sendProduct`, on recieve triggers `ACT_ShowNotificationOnRecieve`

    ![sendProductOnRecieve](attachments/native-remote-push/modeler/sendProductOnRecieve.png)

Follow steps for the previous sections in [here](###testing-the-implementation) but this time put the app in the foreground. You will see the the text with a button for 5 seconds.

![onRecieveShowDV](attachments/native-remote-push/modeler/onRecieveShowDV.png)

## 5 Sending notifications programetcally via Push Notifications API [todo fix title and number, also clarify what the advantage of this approach is]

What if you want to send messages to all your users' devices, but you do nott want to handle the GUID retrieval? The section below will illustrate this example. Specifically, you will send a [todo check rest of this sentence] product from web to all devices with a single button click.

### 5.1 Creating a microflow which will send particular product to all devices [todo fix title and number]

Create a microflow *ACT_SendProductToAllDevices* with the following elements [todo check where pics should go]:

![SendProductToAll](attachments/native-remote-push/modeler/SendProductToAll.png)

1. Product as a parameter
2. Retrieve list of devices from database: `PushNotifications.Device`:

    ![retrieveDevices](attachments/native-remote-push/modeler/retrieveDevices.png)
    
3. PrepareMessageData Microflow from `PushNotifications/_USE ME/API`:<br />
	a. title: myTitle.<br />
	b. body: myBody.<br />
	c. time to live: 0.<br />
	d. badge: 0.<br />
	e. actionName `sendProduct`.<br />
	f. ContextObjectGuid to `empty`:
	
	![prepareMessageData](attachments/native-remote-push/modeler/prepareMessageData.png)

	**ContextObjectGuid** is set to empty since you will pass the object itself to the **SendMessageToDevices** Java action where it will be retrieved automatically. 

4. SendMessageToDevices Java Action in `PushNotifications/_USE ME/API`:<br />
	a. Message data param: $MessageToBeSent.<br />
	b. Device param: $Devices.<br />
	c. Context object: $Product:
	
	![sendMessagesJava](attachments/native-remote-push/modeler/sendMessagesJava.png)
  
5. Go to `Product_NewEdit`, drag and drop the `ACT_SendProductToAllDevices` inside of dataview so that we can trigger this microflow

	![sendProductToAllButton](attachments/native-remote-push/modeler/sendProductToAllButton.png)

### 5.2 Testing the Implementation [todo is this title used elsewhere?]

Now run the app by doing the following:

1, Put the Native app in the background
2. In web go to a particular product and press `ACT_SendProductToAllDevices` microflow button. 

This will send notification to all available devices and when the user taps the notification it will be redirected to the particular product page that we modeled.

### 5.3 More Java Action Explanations

All JAVA actions which is available in Push notifications module with small explanations:

#### 5.3.1 PrepareMessageData Microflow

This allows users to create their own user interface in order to alter and create a push notification message. 

#### 5.3.2 SendMessageToDevice & SendMessageToDevices Java Action

We covered this Java action in this documentation. Params:
- `MessageDataParam` (PushNotifications.MessageData): This param can be generated by the PrepareMessageData microflow.
- `DeviceParam` (List of PushNotifications.Device): Can be used to send same message for a list of devices.
- `ContextObject`: Any mendix object which will be passed to the notification as GUID string.

#### 5.3.3 SendMessageToUsers & SendMessageToUser Java Action

Every user is allowed to have more than one device. In case of sending push notifications to every device of a particular user `SendMessageToUser` can be used.

In case of sending a push notification to all users `SendMessageToUsers` can be used.

## 6 Read More
