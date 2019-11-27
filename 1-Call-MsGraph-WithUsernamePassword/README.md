---
topic: sample
languages:
  - python
  - azurepowershell
products:
  - azure-active-directory
  - office-ms-graph
description: "Python app using MSAL Python to get an access token and call Microsoft Graph."
---

# A simple Python application calling Microsoft Graph with a token acquired by a username and password

## About this sample

### Overview

This sample application shows how to use the [Microsoft identity platform endpoint](http://aka.ms/aadv2) to access the data of Microsoft customers.

The app is a Python Desktop Console application. It displays the organizational user's profile information. The user is hardcoding a username and password into the configuration to accommodate legacy processes.  

## Scenario

The console application:

- gets a token from Microsoft identity platform with a username and password
- calls the Microsoft Graph /me endpoint to get a profile

> The username password flow samples are provided to satisfy specific niche, legacy purposes.  This flow is **not recommended** for use. This flow is banned for personal accounts.

## How to run this sample

To run this sample, you'll need:

> - [Python 2.7+](https://www.python.org/downloads/release/python-2713/) or [Python 3+](https://www.python.org/downloads/release/python-364/)
> - An Azure Active Directory (Azure AD) tenant. For more information on how to get an Azure AD tenant, see [how to get an Azure AD tenant.](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant)

### Step 1:  Clone or download this repository

From your shell or command line:

```Shell
git clone https://github.com/Azure-Samples/ms-identity-python-desktop.git
```
Go to the `"1-Call-MsGraph-WithUsernamePassword"` folder

```Shell
cd 1-Call-MsGraph-WithUsernamePassword
```

### Step 2:  Register the sample with your Azure Active Directory tenant

Some registration is required for Microsoft to act as an authority for your application.

#### Choose the Azure AD tenant where you want to create your applications

1. Sign in to the [Azure portal](https://portal.azure.com).
> If your account is present in more than one Azure AD tenant, select `Directory + Subscription`, which is an icon of a notebook with a filter next to the alert icon, and switch your portal session to the desired Azure AD tenant.
2. Select **Azure Active Directory** from the left nav.
3. Select **App registrations** from the new nav blade.

#### Register the client app

1. In **App registrations** page, select **New registration**.
1. When the **Register an application page** appears, enter your application's registration information:
   - In the **Name** section, enter a meaningful application name that will be displayed to users of the app, for example `desktop-sample`.
   - In the **Supported account types** section, select any option.
   > This flow will not work for personal accounts, however you can configure one app and use this flow just for an organizational user. If in doubt just pick the first option.
1. Select **Register** to create the application.
1. On the app **Overview** page, find the **Application (client) ID** value and copy it to your *parameters.json* file's *client_id* entry.
1. In **Authentication* select the recommended Redirect URIs for public clients.
1. Then set the Default Client Type to `Yes` and Save.

1. In the list of pages for the app, select **API permissions**
   - Click the **Add a permission** button and then,
   - Ensure that the **Microsoft APIs** tab is selected
   - In the *Commonly used Microsoft APIs* section, click on **Microsoft Graph**
   - In the **Delegated permissions** section, ensure that the right permissions are checked: **User.Read**, **User.ReadBasic.All**. Use the search box if necessary.
   - Select the **Add permissions** button

1. Permissions are now assigned correctly, but the client app does not allow interaction. Therefore no consent can be presented via a UI and accepted to use the service.

   Click the **Grant/revoke admin consent for {tenant}** button, and then select **Yes** when you are asked if you want to grant consent for the
   requested permissions for all accounts in the tenant.
   You need to be an Azure AD tenant admin to do this.

### Step 3:  Configure the sample to use your Azure AD tenant

In the steps below, "ClientID" is the same as "Application ID" or "AppId".

Open the `parameters.json` file

#### Configure the client project

1. Open the `parameters.json` file
1. Find the string key `organizations` in the `authority` variable and replace the existing value with your Azure AD tenant name.
1. Find the string key `your_client_id` and replace the existing value with the application ID (clientId) of the `desktop-sample` application copied from the Azure portal.
1. Set the username and password as you have configured.

### Step 4: Run the sample

You'll need to install the dependencies using pip as follows:

```Shell
pip install -r requirements.txt
```

Start the application by running below command. It will display some Json string containing the users in the tenant.

```Shell
python username_password_sample.py parameters.json
```

## About the code

The relevant code for this sample is in the `username_password_sample.py` file. The steps are:

1. Create the MSAL public client application.

    ```Python
    app = msal.PublicClientApplication(
        config["client_id"], authority=config["authority"],
        )
    ```

1. The scopes are defined in the parameters.json file.

   In the default parameters.json file you have:

    ```JSon
    "scope": ["User.ReadBasic.All"],
    ```
1. Check the cache.  
    ```Python
    accounts = app.get_accounts(username=config["username"])
    ```

1. Acquire the token

    ```Python
    result = None
    #skipping cache checks
    result = app.acquire_token_by_username_password(
        config["username"], config["password"], scopes=config["scope"])
    ```

1. Call the API

    In that case calling "https://graph.microsoft.com/v1.0/users" with the access token as a bearer token.

    ```Python
    if "access_token" in result:
        # Calling graph using the access token
        graph_data = requests.get(  # Use token to call downstream service
        config["endpoint"],
        headers={'Authorization': 'Bearer ' + result['access_token']}, ).json()
    print("Users from graph: " + str(graph_data))
    else:
        print(result.get("error"))
        print(result.get("error_description"))
        print(result.get("correlation_id"))  # You may need this when reporting a bug
    ```

## Troubleshooting

### Did you forget to provide admin consent?

If you get an error when calling the API `Insufficient privileges to complete the operation.`, this is because the tenant administrator has not granted permissions
to the application. See step 8 of [Register the client app](#register-the-client-app) above.

You will typically see, on the output window, something like the following:

```Json
Failed to call the Web Api: Forbidden
Content: {
  "error": {
    "code": "Authorization_RequestDenied",
    "message": "Insufficient privileges to complete the operation.",
    "innerError": {
      "request-id": "<a guid>",
      "date": "<date>"
    }
  }
}
```

## Community Help and Support

Use [Stack Overflow](http://stackoverflow.com/questions/tagged/msal) to get support from the community.
Ask your questions on Stack Overflow first and browse existing issues to see if someone has asked your question before.
Make sure that your questions or comments are tagged with [`msal` `python`].

If you find a bug in the sample, please raise the issue on [GitHub Issues](../../issues).

If you find a bug in Msal Python, please raise the issue on [MSAL Python GitHub Issues](https://github.com/AzureAD/microsoft-authentication-library-for-python/issues).

To provide a recommendation, visit the following [User Voice page](https://feedback.azure.com/forums/169401-azure-active-directory).

## Contributing

If you'd like to contribute to this sample, see [CONTRIBUTING.MD](/CONTRIBUTING.md).

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## More information

For more information, see conceptual documentation:

- [Quickstart: Register an application with the Microsoft identity platform](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app)
- [Quickstart: Configure a client application to access web APIs](https://docs.microsoft.com/azure/active-directory/develop/quickstart-configure-app-access-web-apis)
