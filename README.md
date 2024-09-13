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
6. [Issue Reporting](#issue-reporting)
7. [License](#license)

## Features ✨

- Global auto interactivity in Blazor
- OIDC authentication using Descope as the provider
- Non-interactive token refresh

## Configure the Sample ⚙️

Configure the OIDC provider by updating the `Program.cs` file with the Descope-specific configuration. Follow the instructions below in the [OIDC Configuration in Program.cs](#oidc-configuration-in-programcs) section.

## Run the Sample 🚀

### Visual Studio

1. Open the `BlazorWebAppOidc` solution file in Visual Studio.
2. Select the `BlazorWebAppOidc` project in **Solution Explorer** and start the app with either Visual Studio's Run button or by selecting **Start Debugging** from the **Debug** menu.

### .NET CLI

In a command shell, navigate to the `BlazorWebAppOidc` project folder and use the `dotnet run` command to run the sample.

## Environment Setup 🛠️

1. Set the `DESCOPE_PROJECT_ID` and `DESCOPE_CLIENT_SECRET` environment variables.

- **Windows**:
  ```bash
  setx DESCOPE_PROJECT_ID "YOUR_DESCOPE_PROJECT_ID"
  setx DESCOPE_CLIENT_SECRET "YOUR_CLIENT_SECRET"
  ```

2. Alternatively, use **User Secrets** or **Azure Key Vault** to securely manage the `ClientSecret`.

## OIDC Configuration in Program.cs 🛡️

To integrate OIDC using Descope, modify the `Program.cs` file with the following setup:

```csharp
builder.Services.AddOidcAuthentication(oidcOptions =>
{
    // Set the authority to the Descope OIDC Issuer URL.
    oidcOptions.ProviderOptions.Authority = "{Application Issuer URL}";

    // Set the Client ID to your Descope Project ID.
    oidcOptions.ProviderOptions.ClientId = "{Descope Project ID}";

    // Set the Client Secret securely.
    oidcOptions.ProviderOptions.ClientSecret = "{Descope Client Secret}";
});
```

Ensure the `ClientSecret` is stored securely using environment variables or a secrets manager, as it should not be checked into source control.

## Issue Reporting ⚠️

For any issues or suggestions, please [open an issue](https://github.com/descope-sample-apps/blazor-sample-app/issues) on GitHub.

## License 📜

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
