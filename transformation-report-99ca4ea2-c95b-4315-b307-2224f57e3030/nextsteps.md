# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution build produced no errors across all projects after transformation:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

This indicates the migration to cross-platform .NET was completed without introducing any build-breaking changes. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run the following commands from the solution root to confirm a clean restore and build:

```bash
dotnet restore
dotnet build --configuration Release
```

Verify that no warnings are promoted to errors and that all projects build successfully under the `Release` configuration.

---

## 2. Run the Unit Tests

Execute the test project to confirm all existing tests pass on the new runtime:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output for:
- Any tests that were previously passing but are now failing.
- Any tests marked as skipped that may need to be re-enabled.
- Any platform-specific assumptions in tests (e.g., Windows file paths, registry access) that may cause failures on non-Windows environments.

---

## 3. Validate the Data Layer

In `Bookstore.Data`, confirm the following:

- **Database provider**: Ensure the correct NuGet package is referenced for your target database (e.g., `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql.EntityFrameworkCore.PostgreSQL`).
- **Connection strings**: Verify that connection strings in configuration files (`appsettings.json`) are correct for the target environment.
- **Migrations**: If using Entity Framework Core, run the following to verify migrations are up to date:

```bash
dotnet ef migrations list --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

Apply any pending migrations to a test database:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

---

## 4. Validate the Web Project

Run the web application locally to confirm it starts and functions correctly:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- The application starts without runtime exceptions.
- All routes respond as expected.
- Static assets are served correctly.
- Authentication and authorization flows work if applicable.
- Any middleware that was previously Windows-specific (e.g., Windows Authentication, IIS integration) has been replaced or reconfigured appropriately.

---

## 5. Validate the CDK Project

In `Bookstore.Cdk`, review the infrastructure definitions to ensure they reflect the target deployment environment. Confirm:

- All resource configurations (compute, storage, networking) are correct.
- Any environment-specific values are externalized into configuration or environment variables rather than hardcoded.
- The CDK project compiles and synthesizes without errors:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

---

## 6. Check for Runtime Compatibility Issues

Even with a clean build, certain APIs behave differently across platforms. Manually review or use the [.NET Upgrade Assistant](https://learn.microsoft.com/en-us/dotnet/core/porting/upgrade-assistant-overview) compatibility analyzer to check for:

- Use of `System.Windows.Forms` or `System.Web` namespaces.
- Platform-specific P/Invoke calls.
- Registry access (`Microsoft.Win32.Registry`).
- `AppDomain` usage that is not supported in .NET Core and later.
- Any NuGet packages that still target only .NET Framework.

---

## 7. Review Configuration and Secrets

Ensure that configuration has been fully migrated from `Web.config` or `App.config` to the .NET configuration system:

- `appsettings.json` for general settings.
- `appsettings.{Environment}.json` for environment-specific overrides.
- Use `dotnet user-secrets` for local development secrets rather than storing them in configuration files.

---

## 8. Deploy to Target Environment

Once all validation steps pass:

1. Publish the web application:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

2. Verify the contents of the `./publish` directory are complete.
3. Deploy the published output to the target host (e.g., a Linux server, Windows Server, or cloud compute resource).
4. Confirm the application starts correctly in the target environment and that all external dependencies (database, storage, etc.) are reachable.