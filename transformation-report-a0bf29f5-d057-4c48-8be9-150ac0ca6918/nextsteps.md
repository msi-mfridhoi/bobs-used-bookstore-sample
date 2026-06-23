# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Summary

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly in the new target framework:

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or deprecated packages that may need to be updated.

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Address any warnings that surface during the build, particularly those related to nullable reference types or obsolete APIs, as these can indicate areas that may cause runtime issues.

---

## 3. Run Unit Tests

Execute the test project to verify that existing functionality behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test results carefully. Any failing tests should be investigated and resolved before proceeding. If test coverage is low, consider adding tests for critical paths in `Bookstore.Domain` and `Bookstore.Data`.

---

## 4. Verify Data Layer Behavior

Since `Bookstore.Data` handles data access, confirm the following:

- Database connection strings are correctly configured for the new environment (e.g., `appsettings.json` or environment variables).
- Any Entity Framework Core migrations are up to date. Run the following to check the migration status:

```bash
dotnet ef migrations list --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

- If migrations need to be applied to the target database:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

---

## 5. Run the Web Application Locally

Start the web application locally to perform manual validation:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Verify that:

- The application starts without runtime exceptions.
- Core user-facing functionality (browsing, searching, purchasing books, etc.) works as expected.
- Any authentication or authorization flows function correctly.
- Static assets and routing behave as expected.

---

## 6. Review the CDK Project

The `Bookstore.Cdk` project defines infrastructure. Review its configuration to ensure it reflects the correct target environment settings for the migrated application, such as:

- Runtime targets (confirm it references the correct .NET version).
- Any environment-specific parameters (region, resource sizing, etc.) that may need to be updated to align with the new platform.

---

## 7. Check for Deprecated or Replaced APIs

After migrating to cross-platform .NET, some APIs that were available in .NET Framework may have been replaced or removed. Use the .NET Upgrade Assistant compatibility analyzer or review the code manually for:

- Use of `System.Web` namespaces (not available in .NET Core/5+).
- Windows-specific APIs that may not behave correctly on Linux or macOS.
- Any `app.config` or `web.config` settings that should be migrated to `appsettings.json`.

---

## 8. Deploy to Target Environment

Once local validation is complete:

1. Publish the web application:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

2. Verify the contents of the `./publish` directory are complete.
3. Deploy the published output to your target hosting environment according to your infrastructure setup defined in `Bookstore.Cdk`.