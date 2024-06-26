---
parser: v2
auto_validation: true
primary_tag: software-product-function>sap-s-4hana-cloud--abap-environment
tags:  [ tutorial>beginner, software-product-function>sap-s-4hana-cloud--abap-environment, programming-tool>abap-development, programming-tool>abap-extensibility]
time: 25
author_name: Merve Temel 
author_profile: https://github.com/mervey45
---

# Create a SAP Fiori App and Deploy it to SAP S/4HANA Cloud, ABAP Environment

<!-- description --> Create a SAP Fiori app for a RAP business object in SAP Business Application Studio and deploy it to SAP S/4HANA Cloud, ABAP Environment.

## Prerequisites

- You have a license for SAP S/4HANA Cloud and have a developer user in it
- You have installed the latest [Eclipse with ADT](abap-install-adt).
- **Trial:** You need an SAP BTP [trial user](hcp-create-trial-account)
- Business Catalog `SAP_CORE_BC_COM` must be assigned to business user
- The user must have the same email address as the user from trial

## You will learn

- How to assign role collections
- How to create dev spaces
- How to set up organization and space
- How to create list report object pages
- How to run SAP Fiori applications
- How to deploy applications 
- How to check BSP library in Eclipse
- How to create IAM apps and business catalogs

## Intro

>**Hint:** The administrator receives an welcome e-mail after provisioning. This e-mail includes the system URL. By removing `/ui` you can log into the SAP S/4HANA Cloud ABAP Environment system. Further information can be found [here](https://help.sap.com/docs/SAP_S4HANA_CLOUD/6aa39f1ac05441e5a23f484f31e477e7/4b962c243a3342189f8af460cc444883.html?locale=en-US&state=DRAFT).

---

### Assign role collection to user

  1. Login to [SAP BTP Trial cockpit](https://cockpit.hanatrial.ondemand.com/) and click **Enter Your Trial Account**.

      ![assign role collection](bas1.png)

  2. Select your subaccount **trial**.

      ![assign role collection](bas2.png)

  3. Now you are in the trial overview page. Click **Users** and **>**.

      ![assign role collection](btp.png)

  4. Select the menu and click **Assign Role Collection**.

      ![assign role collection](btp2.png)

      **Hint:** If you are using a licensed system, make sure you have the trust administrator role assigned to your user.

  5. Select **`Business_Application_Studio_Developer`** and click **Assign Role Collection**.

      ![assign role collection](btp3.png)

  6. Check your result. Now your user should have the **`Business_Application_Studio_Developer`** role collection assigned.

      ![assign role collection](btp4.png)

### Configure destination

  1. Select your subaccount **trial**.

     ![assign role collection](bas2.png)

  2. In the navigation pane expand the **Connectivity** section and select **Destinations**. Click **New Destination**.

      ![assign role collection](destinations.png)

      Configure the new destination with the following standard field values.

    |  Field Name     | Value
    |  :------------- | :-------------
    |  Name           | **`System_###_SAML_ASSERTION`**
    |  Type           | **`HTTP`**
    |  Description    | **`SAML Assertion Destination to SAP S/4HANA ABAP Environment system_xxx`**
    |  URL          | In the SAP S/4HANA Cloud tenant, navigate to the **Communication Systems** app and copy the **Host Name** from **Own System** = `Yes`<div><!-- border -->![Own System Host Name in Communication Systems App](s4hc-cs-own-system-host-name.png)</div> and paste it with prefix `https://` for example `https://my12345-api.s4hana.ondemand.com.`
    |  Proxy Type   | **`Internet`**
    |  Authentication | **`SAMLAssertion`**
    |  Audience   | Enter the URL of your system and remove `-api`, for example `https://my12345.s4hana.ondemand.com`.
    |  `AuthnContextClassRef` | **`urn:oasis:names:tc:SAML:2.0:ac:classes:PreviousSession`**

    Select **New Property** and maintain the following **Additional Properties** and values.

    |  Field Name     | Value          | Remark
    |  :------------- | :------------- | :-------------
    |  HTML5.DynamicDestination           | **`true`**   |&nbsp;
    |  HTML5.Timeout           | **`60000`**   | value stated in milliseconds. 60000 equals 1 minute. Required as deployment needs longer than the standard of 30 seconds.
    |  `WebIDEEnabled`    | **`true`**   |&nbsp;
    |  `WebIDEUsage`          | **`odata_abap,dev_abap`**   |&nbsp;
    |  `nameIDFormat`     | **`urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`**  | Required in case your subaccount sends mail address as SAML Name ID for authentication ( **Subject Name Identifier** in Identity Authentication tenant ), although SAP S/4HANA Cloud tenant expects user login by default. **That is the case with a trial Account.** This also requires the mail address to be maintained for SAP S/4HANA Cloud tenant business users.

    Make sure that the **Use default JDK truststore** checkbox is ticked.

      ![Configure Destination](new.png)

    Click **Save**.

  3. Click **Download Trust**.

     ![assign role collection](new2.png)

### Create communication system

  1. Select your system and right-click **Properties**.

      ![dev](new3.png)

  2. Select **ABAP Development** and copy the **System URL** without the `-api`. Paste in a browser and log in to the system.

      ![dev](new4.png)

  3. In your SAP Fiori launchpad select **Communication Systems** under Communication Management.

      ![dev](system.png)

  4. Click **New**.

      ![dev](system2.png)

  5. Create a new communication system:
       - System ID: **``BAS_TRIAL_###``**
       - System Name: **``BAS_TRIAL_###``**

       Click **Create**.

     ![dev](new5.png)

  6. Click the arrow and select **Technical Data**.

      ![dev](new6.png)

  7. Check **Inbound Only** in the general section. Set `SAML Bearer Assertion Provider` **ON** and click **Upload Signing Certificate**.

      ![dev](new7.png)

  8. Click **Browse** and select your trust configuration, then click **Upload**.

      ![dev](new8.png)

  9. Copy everything after **`CN=` of your signing certificate subject** and past it in **`SAML Bearer Issuer`**. Click **Save**.

      ![dev](new9.png)

      Now your communication system is set up.

### Create dev space

  1. Select **trial**.

      ![dev](new10.png)

  2. Select **Service Marketplace** and search for **SAP Business Application Studio**. Select actions and click **Go to Application**.

      ![dev](studio212.png)

  3. Check the privacy statement and click **OK**.

      ![dev](studio2.png)

  4. Now the SAP Business Application Studio has started. Click **Create Dev Space**.

      ![dev](studio3.png)

  5. Create a new dev space:
       - Name: **Fiori**
       - Type: **SAP Fiori**

       Click **Create Dev Space**.

     ![dev](studio4.png)

  6. When your status is **Running**, select your dev space **Fiori**.

      ![dev](studio5.png)

### Set project folder

  1. Now you are in your **Fiori** dev space in SAP Business Application Studio.
     Select the menu on the left side and click **Open Folder** to set your workspace.

      ![organization](basn.png)

  2. Select **`/home/user/projects/`** and click **OK**.

      ![organization](basn2.png)

### Create list report object page

 1. Select the menu on the left side and click **View > Command Palette**.

      ![object](basn7.png)

 2. Search for **Fiori: Open Application Generator** and select it.

      ![object](basn8.png)

 3. Select **List Report Page** and click **Next >**.

      ![object](basn9.png)

 4. Configure data source, system and service:
     - Data source: **Connect to a System**
     - System: **`System_###_SAML_ASSERTION`**
     - Service: **`ZUI_SHOPCART_O4_### (0001) - OData V4`**

     ![object](new11.png)

     Click **Next >**.

 5. Select your main entity **`ShoppingCart`**, check **Yes** for adding automatically table columns and click **Next >**.

     ![object](new12.png)

 6. Configure project attributes:  
     - Name: **`z_purchase_req_###`**
     - Title: **Shopping Cart ###**
     - Description: **A Fiori application.**
     - Add deployment configuration: Yes  
     - Add FLP configuration: Yes 
     - Configure advanced options: No

     Click **Next >**.

     ![object](new13.png)

    **Hint:** Your **application name must** begin with a `z letter` and **must** be in **lowercase letters**.

 7. Configure deployment:

       - Target: ABAP
       - Destination: `System_###_SAML_ASSERTION <your_abap_system_url>`
       - SAP UI5 ABAP Repository: `z_cart_###`
       - Package: `z_purchase_req_###`
       - Transport Request: `<your_transport_request>`

     ![app](new14.png)

       Click **Next >**.

     **Hint:** If you want to copy your transport request, please do following:  Open Eclipse, search your package **`Z_PURCHASE_REQ_###`** and open it. Open your transport organizer to see your transport request. The transport request of the superior folder needs to be chosen. Copy your transport request for later use. You can find your **transport request** underneath the **Modifiable** folder.
      ![app](transportrequest.png)

 8. Configure Fiori Launchpad:

       - Semantic Object: `z_purchase_req_###`
       - Action: display
       - Title: Shopping Cart XXX

      ![app](new15.png)

      Click **Finish**.

 9. Now all files have been generated.

### Run SAP Fiori application for data preview

  1. On the **Application Information** overview page select  **Preview Application**.

      ![run](new16.png)

  2. Select the first entry.

      ![run](new17.png)

  3. You browser opens. Click **Go** and check your result.

     ![run](new18.png)

### Deploy your application

1. On the **Application Information** overview page select  **Deploy Application**.

    ![deploy](new19.png)

2. When prompted, check deployment configuration and press y. 
    ![deploy](new20.png)

3. When the deployment is successful, you will get these two information back as a result: **UIAD details** and **deployment successful**.

    ![deploy](new21.png)

### Check BSP library and SAP Fiori Launchpad app descriptor item in Eclipse

  1. Open Eclipse and check the **BSP library** and **SAP Fiori Launchpad app descriptor item folder** in your package **`Z_PURCHASE_REQ_###`**. If you are not able to see BSP applications and SAP Fiori Launchpad app description items, refresh your package `Z_PURCHASE_REQ_###` by pressing `F5`.

     ![library](new22.png)

### Create IAM App and business catalog

  1. In Eclipse right-click your package **`Z_PURCHASE_REQ_###`** and select **New** > **Other Repository Object**.

      ![iam](new23.png)

  2. Search for **IAM App**, select it and click **Next >**.

      ![iam](new24.png)

  3. Create a new IAM App:
     - Name: **`ZSHOPCART_IAM_###`**
     - Description: IAM App for shopping cart

      ![iam](new25.png)

      Click **Next >**.

  4. Click **Finish**.

      ![iam](new26.png)

  5. Select **Services** and add a new one.

      ![iam](new27.png)

  6. Select following:
      - Service Type: `OData V4`
      - Service Name: `ZUI_SHOPCART_O4_###`    

      ![iam](new28.png)

      **Hint:** With **CTRL + Space** you can search for your Service.

      Click **OK**.

      **Save** and **activate** your IAM app.

      ![iam](new29.png)

  7. Right-click your package **`Z_PURCHASE_REQ_###`** and select  **New** > **Other Repository Object**.

      ![catalog](new30.png)

  8. Search for **Business Catalog**, select it and click **Next >**.

      ![catalog](new31.png)

  9. Create a new business catalog:
     - Name: **`ZSHOPCART_BC_###`**
     - Description: Business catalog for shopping cart

      ![catalog](new32.png)

      Click **Next >**.

  10. Click **Finish**.

      ![catalog](new33.png)

  11. Select **Apps** and add a new one.

      ![catalog](new34.png)

  12. Create a new business catalog:
     - IAM App: `ZSHOPCART_IAM_###_EXT`
     - Name: `ZSHOPCART_BC_###_0001`
     - Description: Business Catalog to IAM App assignment

      ![catalog](new35.png)

      Click **Next >**.

  13. Click **Finish**.

       ![catalog](new36.png)

  14. Click **Publish Locally** to publish your business catalog.

       ![catalog](new37.png)

### Run SAP Fiori application

  1. On the **Application Information** overview page select  **Deploy Application**.

     ![deploy](new38.png)

  2. When prompted, check deployment configuration and press y.

     ![deploy](new39.png)

  3. Press **`CTRL and click the following link`** to open the URL in a browser.

     ![deploy](new40.png)

  4. Log in to your system.

      ![url](login.png)

  5. Click **Go** and check your result.

      ![url](new41.png)

### Test yourself
