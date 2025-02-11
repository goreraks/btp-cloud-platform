<!-- loio3971151ba22e4faa9b245943feecea54 -->

# Register the Multitenant Application to the SAP SaaS Provisioning Service

To make a multitenant application available for subscription to SaaS consumer tenants, you \(the application provider\) must register the application in the Cloud Foundry environment via the SAP Software-as-a-Service Provisioning service \(technical name: `saas-registry`\).



## Context

The SAP SaaS Provisioning service allows application providers to register multitenant applications and services in the Cloud Foundry environment in SAP BTP. This activity is a one-time procedure per multitenant application.



## Procedure

1.  Create a service instance of the SAP SaaS Provisioning service with a configuration JSON file in the Cloud Foundry space where the multitenant application is deployed.

    > ### Tip:  
    > You can create the SAP SaaS Provisioning service instance directly in the SAP BTP cockpit.

    For general instructions, see [Creating Service Instances](https://help.sap.com/viewer/09cc82baadc542a688176dce601398de/Cloud/en-US/fad874a99a434ae58c59d7340a528bdc.html).

    1.  The configuration JSON file must follow the following format and set these properties \(note that instead of `xsappname` you can use `appId`\):

        ```
        { 
           "xsappname" : "<xsappname>",
           "appUrls": {
              "getDependencies" : "<getDependenciesUrl>", 
              "onSubscription" : "<onSubscriptionUrl>/{tenantId}"
           },
           "displayName" : "<displayName>",
           "description" : "<description>",
           "category" : "<category>",
           "onSubscriptionAsync": true/false,
           "callbackTimeoutMillis": <int>
           "allowContextUpdates": true/false,
        }
        
        ```

        Specify the following parameters:


        <table>
        <tr>
        <th valign="top">

        Parameters


        
        </th>
        <th valign="top">

        Description


        
        </th>
        </tr>
        <tr>
        <td valign="top">

        `xsappname`


        
        </td>
        <td valign="top">

        The `xsappname` configured in the security descriptor file used to create the `xsuaa` instance \(see [Develop the Multitenant Application](Develop_the_Multitenant_Application_ff54047.md)\).


        
        </td>
        </tr>
        <tr>
        <td valign="top">

        `getDependencies`


        
        </td>
        <td valign="top">

        \(Optional\) Any URL that the application exposes for `GET` dependencies. If the application doesn’t have dependencies and the callback isn’t implemented, it shouldn’t be declared.

        > ### Note:  
        > The JSON response of the callback must be encoded as either UTF8, UTF16, or UTF32, otherwise an error is returned.


        
        </td>
        </tr>
        <tr>
        <td valign="top">

        `onSubscription`


        
        </td>
        <td valign="top">

        Any URL that the application exposes via `PUT` and `DELETE` subscription. It must end with `/{tenantId}`. The tenant for the subscription is passed to this callback as a path parameter. You must keep `{tenantId}` as a parameter in the URL so that it’s replaced at real time with the tenant calling the subscription. This callback URL is called when a subscription between a multitenant application and a consumer tenant is created \(`PUT`\) and when the subscription is removed \(`DELETE`\).


        
        </td>
        </tr>
        <tr>
        <td valign="top">

        `displayName`


        
        </td>
        <td valign="top">

        \(Optional\) The display name of the application when viewed in the cockpit. For example, in the application's tile. If left empty, takes the application's technical name.


        
        </td>
        </tr>
        <tr>
        <td valign="top">

        `description`


        
        </td>
        <td valign="top">

        \(Optional\) The description of the application when viewed in the cockpit. For example, in the application's tile. If left empty, takes the application's display name.


        
        </td>
        </tr>
        <tr>
        <td valign="top">

        `category`


        
        </td>
        <td valign="top">

        \(Optional\) The category to which the application is grouped in the *Subscriptions* page in the cockpit. If left empty, gets assigned to the default category.


        
        </td>
        </tr>
        <tr>
        <td valign="top">

        `onSubscriptionAsync`


        
        </td>
        <td valign="top">

        Whether the subscription callback is asynchronous.

        If set to true, `callbackTimeoutMillis` is mandatory.


        
        </td>
        </tr>
        <tr>
        <td valign="top">

        `callbackTimeoutMillis`


        
        </td>
        <td valign="top">

        The number of milliseconds the SAP SaaS Provisioning service waits for the application's subscription asynchronous callback to execute, before it changes the subscription status to `FAILED`.


        
        </td>
        </tr>
        <tr>
        <td valign="top">

        `allowContextUpdates`


        
        </td>
        <td valign="top">

        Whether to send updates about the changes in contextual data for the service instance.

        For example, when a subaccount with which the instance is associated is moved to a different global account.

        Defaut value is false.


        
        </td>
        </tr>
        </table>
        
        > ### Sample Code:  
        > Configuration JSON file \(example: config.json\):
        > 
        > ```
        > {
        >    "xsappname":"saas-app",
        >    "appUrls": {
        >       "getDependencies" : "https://saas-app.cfapps.eu10.hana.ondemand.com/callback/v1.0/dependencies",
        >       "onSubscription" : "https://saas-app.cfapps.eu10.hana.ondemand.com/callback/v1.0/tenants/{tenantId}"
        >    },
        >    "displayName" : "Hello World",
        >    "description" : "My multitenant biz app",
        >    "category" : "Test"
        > }
        > ```

    2.  Execute the following cf CLI command to create the service instance with the JSON configuration file:

        ```
        cf create-service saas-registry application <SAAS_REGISTRY_SERVICE_INSTANCE> -c <JSON_CONFIG_FILE>
        ```

        Specify the following parameters:


        <table>
        <tr>
        <th valign="top">

        Parameters


        
        </th>
        <th valign="top">

        Description


        
        </th>
        </tr>
        <tr>
        <td valign="top">

        `SAAS_REGISTRY_SERVICE_INSTANCE`


        
        </td>
        <td valign="top">

        The new name for your service instance. Use only alphanumeric characters, hyphens, and underscores.


        
        </td>
        </tr>
        <tr>
        <td valign="top">

        `JSON_CONFIG_FILE`


        
        </td>
        <td valign="top">

        The file name of the service-specific configuration parameters, in a valid JSON object \(described above\).


        
        </td>
        </tr>
        </table>
        
        > ### Sample Code:  
        > ```
        > cf create-service saas-registry application saas-registry-application -c config.json
        > ```


2.  To ensure that the application trusts the JWT issued for another identity zone \(`sap-provisioning`\), add the environment variable: `SAP_JWT_TRUST_ACL`

    ```
    SAP_JWT_TRUST_ACL:'[{"clientid":"*","identityzone":"sap-provisioning"}]'
    ```

    > ### Note:  
    > The client libraries \(java-security, spring-xsuaa, container security api for node.js \>= 3.0.6, approuter \>= 8.x, and sap\_xssec for Python \>= 2.1.0\) have been updated. When using these libraries, setting the parameter `SAP_JWT_TRUST_ACL` has become obsolete.
    > 
    > Instead, the `xsuaa` service adds audiences to the issued JSON Web Token audience field \(`aud`\). Client libraries provide an audience validator to check whether the token is issued for your service or application. Technically, this information is implicitly derived from the audience field \(`aud`\) or from the scopes.
    > 
    > Therefore, this update comes with a change regarding scopes. For a business application A that wants to call an application B, it's now mandatory that application B grants at least one scope to the calling business application A. Furthermore, business application A has to accept these granted scopes or authorities as part of the application security descriptor. You can grant scopes with the `xs-security.json` file.
    > 
    > For more information, see [Application Security Descriptor Configuration Syntax](Application_Security_Descriptor_Configuration_Syntax_517895a.md).

3.  Bind the SAP SaaS Provisioning service instance that you created to the multitenant application that you deployed in your Cloud Foundry space using the following cf CLI command:

    ```
    cf bind-service <APP_NAME> <SAAS_REGISTRY_SERVICE_INSTANCE>
    ```

    Specify the following parameters:


    <table>
    <tr>
    <th valign="top">

    Parameters


    
    </th>
    <th valign="top">

    Description


    
    </th>
    </tr>
    <tr>
    <td valign="top">

    `APP_NAME`


    
    </td>
    <td valign="top">

    The ID of your deployed multitenant application.


    
    </td>
    </tr>
    <tr>
    <td valign="top">

    `SAAS_REGISTRY_SERVICE_INSTANCE`


    
    </td>
    <td valign="top">

    The name of the SAP SaaS Provisioning service instance you created in step 1 above.


    
    </td>
    </tr>
    </table>
    
    > ### Sample Code:  
    > ```
    > cf bind-service saas-app saas-registry-application
    > ```

    > ### Tip:  
    > You can bind your application to the SAP SaaS Provisioning service instance directly in the SAP BTP cockpit.

    For general instructions, see [Binding Service Instances to Applications](Binding_Service_Instances_to_Applications_e98280a.md).

4.  To ensure that the environment variables in the application take effect, execute the following cf CLI command:

    ```
    cf restage <APP_NAME>
    ```

    Specify the following parameter:


    <table>
    <tr>
    <th valign="top">

    Parameters


    
    </th>
    <th valign="top">

    Description


    
    </th>
    </tr>
    <tr>
    <td valign="top">

    `APP_NAME`


    
    </td>
    <td valign="top">

    The ID of your deployed multitenant application.


    
    </td>
    </tr>
    </table>
    
    > ### Sample Code:  
    > ```
    > cf restage saas-app
    > ```




<a name="loio3971151ba22e4faa9b245943feecea54__postreq_ul5_h3l_r2b"/>

## Next Steps

1.  In your global account, create a subaccount for each consumer tenant.

2.  Subscribe your consumer tenants to the multitenant applications using the cockpit.

3.  Test your multitenant application from consumer tenants to ensure availability, subscription lifecycle, and optional application configurations.

4.  Supply your customers or users of the multitenant application with their consumer-specific URL to access the application.


For more information, see [Providing Multitenant Applications to Consumers in the Cloud Foundry Environment](Providing_Multitenant_Applications_to_Consumers_in_the_Cloud_Foundry_Environment_7a013f1.md).

