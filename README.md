# Python console application for Microsoft Graph

This sample uses Microsoft Graph to read your user profile, download your profile photo, upload the photo to OneDrive, create a sharing link, and send an email on your behalf (to yourself by default).

Authentication is handled via [device flow authentication](#device-flow-authentication), the recommended approach for console applications. If you're looking for examples of how to work with Microsoft Graph from Python _web applications_, see [Python authentication samples for Microsoft Graph](https://github.com/microsoftgraph/python-sample-auth). For a web app sample that does the same things as this console app sample, see [Sending mail via Microsoft Graph from Python](https://github.com/microsoftgraph/python-sample-send-mail).

* [Installation](#installation)
* [Running the sample](#running-the-sample)
* [Device Flow authentication](#device-flow-authentication)
* [Helper functions](#helper-functions)
* [Contributing](#contributing)
* [Resources](#resources)

## Installation

This section covers how to install and configure the sample application.

## Prerequisites

Before installing the sample:

* Install Python from [https://www.python.org/](https://www.python.org/). We've tested the code with Python 3.6.2, but any Python 3.x version should work. If your code base is running under Python 2.7, you may find it helpful to use the [3to2](https://pypi.python.org/pypi/3to2) tools to port the code to Python 2.7.
* This sample requires an [Office 365 for business account](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account).
* To register your application in the Azure Portal, you will need an Azure account associated with your Office 365 account. No charges will be incurred for registering your application in the Azure Portal, but you must have an account. If you don't have one, you can sign up for an [Azure Free Account](https://azure.microsoft.com/en-us/free/free-account-faq/).

## Installing the sample code

Follow these steps to install the sample code:

1. Clone the repo with this command:
    * ```git clone https://github.com/microsoftgraph/python-sample-console-app.git```

2. Create and activate a virtual environment (optional). If you're new to Python virtual environments, [Miniconda](https://conda.io/miniconda.html) is a great place to start.

3. In the root folder of your cloned repo, install the dependencies for the sample as listed in the ```requirements.txt``` file with this command: ```pip install -r requirements.txt```.

## Application Registration

To run the sample, you will need to register an application and add the registered application's ID to the configuration information in the [config.py](https://github.com/microsoftgraph/python-sample-console-app/blob/master/helpers.py) file. Follow these steps to register and configure your application:

1. Sign in to the [Azure Portal](https://portal.azure.com) and choose **Azure Active Directory** in the sidebar, then choose **App registrations** and **New application registration**.

2. Enter a unique name for your application, choose **Native** for the Application type, and enter any valid URL for **Redirect URI**. (The Redirect URI is not used in this sample, but you must enter a valid URL during app registration.) Then choose **Create**.
![create application](images/registration1.png)

3. After the application is created, you'll be returned to the **App registrations** list. Choose your app from the list to open the settings page. (If the list of apps is long, you can type the beginning of the app name into the search box, and the list will show only apps that start with the specified string.) Once you have the app settings page open, copy the **Application ID** and save it. You'll need this later to complete the setup process.
![copy Application ID](images/registration2.png)

4. The final step in app registration is to add the permissions that the sample app requires. From the app settings page, choose **Required permissions**, then **Add**, then **Select an API**. Then choose **Microsoft Graph** and **Select**. Here you'll see a list of _Delegated Permissions_. Check the box next to these permissions:
**Have full access to user files** (this is the _Files.ReadWrite_ permission needed to upload files to your OneDrive account)
**Send email as a user** (this is the _Mail.Send_ permission needed to send email)

After registering your application, modify the ```config.py``` file in the root folder of your cloned repo, and follow the instructions to enter your Client ID (the Application ID value you copied in Step 3 earlier). Save the change, and you're ready to run the sample.

## Running the sample

Follow these steps to run the sample app:

1. At the command prompt, run the command ```python sample.py```. You'll see a message telling you to open a page in your browser and enter a code.
![launch the sample](images/running1.png)

2. After entering the code at https://aka.ms/devicelogin, you'll be prompted to select an identity or enter an email address to identify yourself. The identity you use must be in the same organization/tenant where the application was registered. Sign in, and then you'll be asked to consent to the application's delegated permissions as shown below. Choose **Accept**.
![consenting to permissions](images/running2.png)

3. After consenting to permissions, you'll see a message saying "You have signed in to the console-app-sample application on your device. You may now close this window." Close the browser and return to the console. You are now authenticated, and the app has a token that can be used for Microsoft Graph requests. The app will request your user profile and display your name and email address, then prompt you for a destination email address. You may enter one or more email recipients (delimited with ;), or press **Enter** to send the email to yourself.
![entering recipients](images/running3.png)

4. After entering email recipients, you'll see console output showing the Graph endpoints and responses for the remaining steps in the sample app: getting your profile photo, uploading it to OneDrive, creating a sharing link, and sending the email.
![sending the mail](images/running4.png)

Check your email, and you'll see the email that has been sent. It includes your profile photo as an attachment, as well as a view-only sharing link to the copy of your profile photo that was uploaded to the root folder of your OneDrive account.

![receiving the mail](images/running5.png)

## Device Flow authentication

Microsoft Graph uses Azure Active Directory (Azure AD) for authentication, and Azure AD supports a variety of [OAuth 2.0](http://www.rfc-editor.org/rfc/rfc6749.txt) authentication flows. The recommended authorization flow for Python console apps is [device flow](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-07), and this sample uses the [Microsoft ADAL for Python](https://github.com/AzureAD/azure-activedirectory-library-for-python) to implement device flow as shown in the diagram below.

![device flow](images/deviceflow.png)

The device flow implementation in the sample app can be found in the ```device_flow_auth()``` function of [helpers.py](https://github.com/microsoftgraph/python-sample-console-app/blob/master/helpers.py). They key concept for device flow is that our app gets a _user code_ and shows it to the user, and that code must be entered on the _authorization URL_ page before it expires. When we call ADAL's ```acquire_token_with_device_code()``` method, it begins polling to check whether the code has been entered, and if the code is entered and the user consents to the requested permissions, an access token is returned.

This sample doesn't use a **refresh token**, but it's easy to obtain a refresh token if you'd like to provide your users with a "remember me" experience that doesn't require logging in every time they run your app. To get a refresh token, register the application with ```offline_access``` permission, and then you'll receive a refresh token which you can use with ADAL's [acquire_token_with_refresh_token](https://github.com/AzureAD/azure-activedirectory-library-for-python/blob/dev/sample/refresh_token_sample.py#L47-L69) method to refresh a Graph access token.

## Helper functions

Several helper functions in [helpers.py](https://github.com/microsoftgraph/python-sample-console-app/blob/master/helpers.py) provide simple wrappers for common Graph operations, and provide examples of how to make authenticated Graph requests. These functions can be used with any auth library &mdash; the only requirement is that there is a valid Graph access token stored in the default HTTP headers of the session. The  ```device_flow_session()``` function handles as explained below.

### api_endpoint(url)

Converts a relative path such as ```/me/photo/$value``` to a full URI based on the current RESOURCE and API_VERSION settings in config.py.

### device_flow_session(client_id, auto=False)

Obtains an access token from Azure AD (via device flow) and create a Requests session instance ready to make authenticated calls to Microsoft Graph. The only required argument is the **client_id** of the [registered application](#application-registration).

The optional **auto** argument can be set to ```True``` to automatically launch the authorization URL in your default browser and copy the user code to the clipboard. This can save time during repetitive dev/test activities.

### profile_photo(session, *, user_id='me', save_as=None)

Gets a profile photo, and optionally saves a local copy. Returns a tuple of the raw photo data, HTTP status code, content type, and saved filename.

### send_mail(session, *, subject, recipients, body='', content_type='HTML', attachments=None)

Sends email from current user. Returns the Requests response object for the POST.

### sharing_link(session, *, item_id, link_type='view')

Creates a sharing link for an item in OneDrive.

### upload_file(session, *, filename, folder=None)

Uploads a file to OneDrive for Business.

## Contributing

These samples are open source, released under the [MIT License](https://github.com/microsoftgraph/python-sample-console-app/blob/master/LICENSE). Issues (including feature requests and/or questions about this sample) and [pull requests](https://github.com/microsoftgraph/python-sample-console-app/pulls) are welcome. If there's another Python sample you'd like to see for Microsoft Graph, we're interested in that feedback as well &mdash; please log an [issue](https://github.com/microsoftgraph/python-sample-console-app/issues) and let us know!

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Resources

* [OAuth 2.0 Device Flow for Browserless and Input Constrained Devices](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-07)
* [Microsoft ADAL for Python](https://github.com/AzureAD/azure-activedirectory-library-for-python)
* [Sending mail via Microsoft Graph from Python](https://github.com/microsoftgraph/python-sample-send-mail)
* [Python authentication samples for Microsoft Graph](https://github.com/microsoftgraph/python-sample-auth)
* Graph API documentation:
    * [Get a user](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_get)
    * [Get photo](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/profilephoto_get)
    * [Send mail](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_sendmail)
    * [Create a sharing link for a DriveItem](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/driveitem_createlink)
    * [Upload or replace the contents of a DriveItem](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/driveitem_put_content)