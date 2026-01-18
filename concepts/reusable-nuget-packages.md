Consuming a NuGet package directly from a Git repository or an artifact repository depends on how the package is stored. Here are the different ways to do it:

---

### 1. **Using a NuGet Package from a Git Repository (Without a NuGet Feed)**

If the package is stored as a `.nupkg` file in a Git repository without a proper NuGet feed, you can do the following:

#### **Option 1: Manually Add the Package to Your Project**

1. Clone or download the repository containing the `.nupkg` file.
2. Copy the `.nupkg` file to a local folder, e.g., `C:\local-nuget`.
3. Add the local folder as a NuGet source:
    
    ```sh
    dotnet nuget add source "C:\local-nuget" -n LocalNuGet
    ```
    
4. Restore the package:
    
    ```sh
    dotnet restore
    ```
    
5. Add the package reference in `csproj`:
    
    ```xml
    <PackageReference Include="PackageName" Version="X.Y.Z" />
    ```
    

---

### 2. **Using a NuGet Package from a Git Repository (Using Git as a Package Source)**

NuGet does not natively support pulling packages from a Git repository. However, you can use a Git submodule or download the source and build it yourself.

#### **Option 1: Use Git Submodules**

1. Add the package repository as a submodule:
    
    ```sh
    git submodule add https://github.com/your-org/your-package.git packages/your-package
    ```
    
2. Reference the project in your `.csproj` file:
    
    ```xml
    <ProjectReference Include="../packages/your-package/YourPackage.csproj" />
    ```
    

#### **Option 2: Build and Pack It Locally**

1. Clone the Git repository:
    
    ```sh
    git clone https://github.com/your-org/your-package.git
    ```
    
2. Build and create a `.nupkg` file:
    
    ```sh
    dotnet pack --output ./nupkg
    ```
    
3. Add it as a local NuGet source:
    
    ```sh
    dotnet nuget add source ./nupkg -n LocalNuGet
    ```
    
4. Install the package as usual.

---

### 3. **Using a NuGet Package from a Private Artifact Repository (Azure DevOps, GitHub Packages, etc.)**

If your package is hosted on a private NuGet feed, you need to authenticate and configure NuGet accordingly.

#### **For Azure DevOps Artifacts**

1. Add your feed to your NuGet config:
    
    ```sh
    dotnet nuget add source "https://pkgs.dev.azure.com/your-org/_packaging/your-feed/nuget/v3/index.json" -n AzureDevOps
    ```
    
2. Authenticate:
    
    ```sh
    dotnet nuget update source AzureDevOps --username your-username --password your-PAT --store-password-in-clear-text
    ```
    
3. Install the package:
    
    ```sh
    dotnet add package PackageName
    ```
    

#### **For GitHub Packages**

1. Authenticate using a GitHub token:
    
    ```sh
    dotnet nuget add source "https://nuget.pkg.github.com/your-org/index.json" -n GitHub --username your-username --password your-GITHUB_TOKEN --store-password-in-clear-text
    ```
    
2. Install the package:
    
    ```sh
    dotnet add package PackageName
    ```
    

---

### **Which One Should You Use?**

- If the package is not a `.nupkg`, use **Git submodules** or **Project References**.
- If it's a `.nupkg` in a repo, **download and add it as a local NuGet source**.
- If it’s hosted in a private NuGet feed (Azure, GitHub, etc.), **configure NuGet credentials and install it**.