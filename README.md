<img width="1400" alt="Screenshot 2024-09-12 at 7 57 46 PM" src="https://github.com/user-attachments/assets/e424bf3f-a540-4610-a156-69d8b552301e">

# Blazor + Descope Sample App (OIDC)

- A Blazor Web App with global Auto interactivity.
  - This adds `PersistingAuthenticationStateProvider` and `PersistentAuthenticationStateProvider` services to the server and client Blazor apps respectively to capture authentication state and flow it between the server and client.
- OIDC authentication using Descope as the provider, without requiring provider-specific packages.
  - This sample serves as a starting point for any OIDC authentication flow.
- Automatic non-interactive token refresh with the help of a custom `CookieOidcRefresher`.

## Table of Contents 📝

1. [Features](#features)
2. [Configure the Sample](#configure-the-sample)
3. [Running the Sample](#run-the-sample)
4. [Environment Setup](#environment-setup)
5. [OIDC Configuration in Program.cs](#oidc-configuration-in-programcs)
6. [Handling ClientSecret, ClientId, and Issuer URL](#handling-clientsecret-clientid-and-issuer-url)
7. [Handling Custom Claims, Roles, and Permissions](#handling-custom-claims-roles-and-permissions)
8. [Issue Reporting](#issue-reporting)
9. [License](#license)

## Features ✨

- Global auto interactivity in Blazor
- OIDC authentication using Descope as the provider
- Non-interactive token refresh
- Custom claims handling from Descope JWTs
- Storing tokens securely in cookies using Blazor

## Configure the Sample ⚙️

Configure the OIDC provider by updating the `Program.cs` file with the Descope-specific configuration. Follow the instructions below in the [OIDC Configuration in Program.cs](#oidc-configuration-in-programcs) section.

## Run the Sample 🚀

This section goes over how you can run this sample application in your local environment.

### Visual Studio

1. Open the `BlazorWebAppOidc` solution file in Visual Studio.
2. Select the `BlazorWebAppOidc` project in **Solution Explorer** and start the app with either Visual Studio's Run button or by selecting **Start Debugging** from the **Debug** menu.

### .NET CLI

In a command shell, navigate to the `BlazorWebAppOidc` project folder and use the `dotnet run` command to run the sample.

## Environment Setup 🛠️

> **Note**: You can find your Descope Project ID under [Project Settings](https://app.descope.com/settings/project). You can create an access key to use as a client secret, under [Access Keys](https://app.descope.com/accesskeys).

1. Set the `DESCOPE_PROJECT_ID`, `DESCOPE_CLIENT_SECRET`, and `DESCOPE_ISSUER_URL` environment variables.

You can find your Descope Project ID under [Project Settings](https://app.descope.com/settings/project).

You can create an access key to use as a client secret, under [Access Keys](https://app.descope.com/accesskeys).

You can find the Issuer URL, under your [Application Configuration](https://app.descope.com/applications) in the Descope Console.

- **Windows**:
  ```bash
  setx DESCOPE_PROJECT_ID "DESCOPE_PROJECT_ID"
  setx DESCOPE_CLIENT_SECRET "DESCOPE_ACCESS_KEY"
  setx DESCOPE_ISSUER_URL "DESCOPE_ISSUER_URL"
  ```

2. Alternatively, securely manage the `ClientSecret` by using **User Secrets** or **Azure Key Vault** to avoid checking sensitive data into source control.

## OIDC Configuration in Program.cs 🛡️

Your Application

To configure OIDC with Descope, modify the `Program.cs` file as follows:

```csharp
builder.Services.AddAuthentication(MS_OIDC_SCHEME)
    .AddOpenIdConnect(MS_OIDC_SCHEME, oidcOptions =>
    {
        // Set the authority to the Descope OIDC Issuer URL. You can get this under your Application Configuration in the Descope Console.
        oidcOptions.ClientId = Environment.GetEnvironmentVariable("DESCOPE_ISSUER_URL");

        // Set the Client ID to your Descope Project ID.
        oidcOptions.ClientId = Environment.GetEnvironmentVariable("DESCOPE_PROJECT_ID");

        // ClientSecret shouldn't be hardcoded; manage it using environment variables or a secure store.
        oidcOptions.ClientSecret = Environment.GetEnvironmentVariable("DESCOPE_CLIENT_SECRET");

        // Additional OIDC configuration settings
        oidcOptions.ResponseType = OpenIdConnectResponseType.Code;
        oidcOptions.SaveTokens = true;
    });
```

### Handling ClientSecret, ClientId, and Issuer URL

- **ClientSecret**: Never hardcode the `ClientSecret` in your source code. Instead, retrieve it using secure methods such as **User Secrets**, **Azure Key Vault**, or environment variables as shown above. This ensures that sensitive data is not stored in the application assembly or checked into version control.
- **ClientId**: The `ClientId` is the Descope Project ID and should be included in the `oidcOptions.ClientId` field.
- **Issuer URL**: The Issuer URL for Descope is set using `oidcOptions.Authority`. This URL is used by the app to retrieve OIDC endpoints necessary for authentication.

You can also set these values using application configuration files, such as `appsettings.json`, if preferred:

```json
{
  "Authentication": {
    "Schemes": {
      "MicrosoftOidc": {
        "ClientId": "DESCOPE_PROJECT_ID",
        "ClientSecret": "DESCOPE_ACCESS_KEY",
        "Authority": "DESCOPE_ISSUER_URL"
      }
    }
  }
}
```

### Name Claim

In the `UserInfo.cs` we map the `Name` and `Sub` claim from the Descope JWT to the `UserInfo` class in the constructor. Make sure that your Descope JWT includes the name claim, with a [Custom Claim action](https://docs.descope.com/actions/custom-claims) in your flow or a [JWT Template](https://docs.descope.com/project-settings/jwt-templates). Otherwise the login will not be successful.

## Handling Custom Claims, Roles, and Permissions 🛠️

### Descope Custom Claims

Descope allows you to include custom claims in the JWT by using the `descope.custom_claims` scope. To handle these custom claims in Blazor, you'll need to:

1. Add the `descope.custom_claims` scope to your authentication request.
2. Modify the `TokenValidationParameters` in your OIDC setup to process the custom claims.

Example:

```csharp
builder.Services.AddAuthentication(MS_OIDC_SCHEME)
    .AddOpenIdConnect(MS_OIDC_SCHEME, oidcOptions =>
    {
        // Set the authority and client information
        oidcOptions.ClientId = Environment.GetEnvironmentVariable("DESCOPE_ISSUER_URL");
        oidcOptions.ClientId = Environment.GetEnvironmentVariable("DESCOPE_PROJECT_ID");

        // Include custom claims scope
        oidcOptions.Scope.Add("descope.custom_claims");

        // Ensure custom claims can be mapped and used in the application
        oidcOptions.TokenValidationParameters.MapInboundClaims = false;
        oidcOptions.TokenValidationParameters.NameClaimType = "custom_name";
        oidcOptions.TokenValidationParameters.RoleClaimType = "custom_role";
    });
```

### Roles and Permissions

If your app manages roles and permissions through Descope, these can also be included in the JWT as claims. You can extract and handle these claims in your Blazor app by using the `AuthenticationStateProvider`:

```csharp
var user = (await AuthenticationStateTask).User;

if (user.Identity.IsAuthenticated)
{
    var roles = user.Claims
                    .Where(c => c.Type == "roles")
                    .Select(c => c.Value);

    var permissions = user.Claims
                          .Where(c => c.Type == "permissions")
                          .Select(c => c.Value);
}
```

### Storing Tokens in Cookies

In Blazor, tokens (such as access and refresh tokens) can be securely stored in cookies using the Blazor cookie libraries:

1. Configure cookie storage in your `Program.cs`:

```csharp
builder.Services.ConfigureCookieOidcRefresh(CookieAuthenticationDefaults.AuthenticationScheme, MS_OIDC_SCHEME);
builder.Services.AddScoped<AuthenticationStateProvider, PersistingAuthenticationStateProvider>();
```

2. Ensure tokens are saved and automatically refreshed:

```csharp
oidcOptions.SaveTokens = true;
oidcOptions.Scope.Add("offline_access");
```

This setup ensures that tokens are securely persisted in cookies and refreshed when necessary without requiring user interaction.

## Issue Reporting ⚠️

For any issues or suggestions, please [open an issue](https://github.com/descope-sample-apps/blazor-sample-app/issues) on GitHub.

## License 📜

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
