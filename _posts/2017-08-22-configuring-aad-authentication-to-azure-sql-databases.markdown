---
layout: post
title: Configuring AAD Authentication to Azure SQL Databases
date: '2017-08-22 06:03:40'
tags:
- development
- cloud
---

Azure SQL is a great service - you get your databases into the cloud without having to manage all that nasty server stuff. However, one of the problems with Azure SQL is that you have to authenticate using SQL authentication - a username and password. However, you can also authenticate [via Azure Active Directory (AAD) tokens](https://blogs.msdn.microsoft.com/sqlsecurity/2016/02/09/token-based-authentication-support-for-azure-sql-db-using-azure-ad-auth/). This is analogous to integrated login using Windows Authentication - but instead of Active Directory, you're using AAD.

There are a number of advantages to AAD Authentication:

- You no longer have to share logins since users log in with their AAD credentials, so auditing is better
- You can manage access to databases using AAD groups
- You can enable "app" logins via Service Principals

In order to get this working, you need:

- To enable AAD authentication on the Azure SQL Server
- A Service Principal
- Add logins to the database granting whatever rights required to the service principal
- Add code to get an auth token for accessing the database
- If you're using Entity Framework (EF), create a new constructor for your DbContext

In this post I'll walk through creating a service principal, configuring the database for AAD auth, creating code for retrieving a token and configuring an EF DbContext for AAD auth.

## Create a Service Principal

Azure lets you configure service principals - these are like service accounts on an Active Directory. The advantage to this is that you can configure access to resources for the service and not have to worry about users leaving the org (or domain) and having to change creds and so on. Service principals get keys that can be rotated for better security too. You'll need the service principal when you configure your app to connect to the database.

You can create a service principal using the portal or you can do it easily using:

    # Azure CLI 2.0
    az ad sp create-for-rbac --name CoolAppSP --password SomeStrongPassword
    
    # PowerShell
    # get the application we want a service principal for
    $app = Get-AzureRmADApplication -DisplayNameStartWith MyDemoApp
    New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId -DisplayName CoolAppSP -Password SomeStrongPassword
    

Of course you need to provide a proper strong password! Take a note of the servicePrincipalNames property - the one that looks like a GUID. We'll need this later.

## Configuring AAD on the Database

In order to use AAD against the SQL Server, you'll need to configure an AAD admin (user or group) for the database. You can do this in the portal by browsing to the Azure SQL Server (not the database) and clicking "Active Directory Admin". In the page that appears, click "Set Admin" and assign a user or group as the AAD admin.

Once you've done that, you need to grant Azure AD users (or groups) permissions in the databases (not the server). To do that you have to connect to the database using an Azure AD account. Open Visual Studio or SQL Server Management Studio and connect to the database as the admin (or a member of the admin group) using "Active Directory Password Authentication" or "Azure Directory Integrated Authentication" from the Authentication dropdown:

<!--kg-card-begin: html-->[![image](/assets/images/files/5c7acf37-8eb9-4ff5-b582-05b5016600ee.png "image")](/assets/images/files/48392dc1-e100-4568-8122-413720245376.png)<!--kg-card-end: html-->

If you don't see these options, then you'll need to update your SQL Management Studio or SSDT. If you're domain joined to the Azure Active Directory domain, you can use the integrated method - in my case my laptop isn't domain joined so I used the password method. For username and password, I used my Azure AD (org account) credentials. Once you're logged in and connected to the database, execute the following T-SQL:

    CREATE USER [CoolAppSP] FROM EXTERNAL PROVIDER
    EXEC sp_addrolemember 'db_owner', 'CoolAppSP'

Of course you'll use the name of the Service Principal you created earlier - the name for the Login is the same name as the service principal you created, or can be the email address of a specific user or group display name if you're granting access to specific AAD users or groups so that they can access the db directly. And of course the role doesn't have to be dbowner - it can be whatever role you need it to be.

## Authenticating using the Service Principal

There are a couple of pieces we need in order to authenticate an application to the Azure SQL database using AAD credentials. The first is a token (it's an OAuth token) that identifies the service principal. Secondly, we need to construct a database connection that uses the token to authenticate to the server.

### Retrieve a Token from AAD

To get a token, we'll need to call Azure AD and request one. For this, you'll need the Microsoft.IdentityModel.Clients.ActiveDirectory Nuget package.

Here's the code snippet I used to get a token from AAD:

    public async Task&lt;string&gt; GetAccessTokenAsync(string clientId, string clientSecret, string authority, string resource, string scope)
    {
    	var authContext = new AuthenticationContext(authority, TokenCache.DefaultShared);
    	var clientCred = new ClientCredential(clientId, clientSecret);
    	var result = await authContext.AcquireTokenAsync(resource, clientCred);
    
    	if (result == null)
    	{
    		throw new InvalidOperationException("Could not get token");
    	}
    
    	return result.AccessToken;
    }

Notes:

- Line 1: We need some information in order to get the token: ClientId and ClientSecret are from the service principal. The authority, resource and scope will need to be passed in too (more on this later).
- Line 3: We're getting a token from the "authority" or tenant in Azure
- Line 4: We create a new client credential using the id and secret of the "client" (in this case, the service principal)
- Line 5: We get a token for this client onto the "resource"
- Line 7: We throw if we don't get a token back
- Line 12: If we do get a token, return it to the caller

The client id is the "application ID" of the service principal (the guid in the servicePrincipalNames property of the service principal). To get the secret, log in to the portal and click in the Active Directory blade. Click on "App Registration" and search for your service principal. Click on the service principal to open it. Click on Keys and create a key - make a note of the key so that you can add this to configurations. This key is the clientSecret that the GetAccessToken method needs.

For authority, you'll need to supply the URL to your Azure tenant. You can get this by running "az account show" (Azure CLI 2.0) or "Get-AzureRmSubscription" (PowerShell). Make a note of the tenantId of the subscription (it's a GUID). Once you have that, the authority is simply "https://login.windows.net/{tenantId}". The final piece of info required is the resource - for Azure SQL access, this is simply "https://database.windows.net/". The scope is just empty string - for databases, the security is configured per user (using the role assignments on the DB you configured earlier). The _authentication_ is done using Azure AD via the token - the database is doing _authorization_. In other words, Azure lets an Azure AD user in when they present a valid token - the database defines what the user can do once they're in via roles.

### Creating a SQL Connection

We've now got a way to get a token - so we can create a SQL Connection to the database. Here's a code snippet:

    public async Task&lt;SqlConnection&gt; GetSqlConnectionAsync(string tenantId, string clientId, string clientSecret, string dbServer, string dbName)
    {
    	var authority = string.Format("https://login.windows.net/{0}", tenantId);
    	var resource = "https://database.windows.net/";
    	var scope = "";
    	var token = await GetTokenAsync(clientId, clientSecret, authority, resource, scope);
    
    	var builder = new SqlConnectionStringBuilder();
    	builder["Data Source"] = $"{dbServer}.database.windows.net";
    	builder["Initial Catalog"] = dbName;
    	builder["Connect Timeout"] = 30;
    	builder["Persist Security Info"] = false;
    	builder["TrustServerCertificate"] = false;
    	builder["Encrypt"] = true;
    	builder["MultipleActiveResultSets"] = false;
    
    	var con = new SqlConnection(builder.ToString());
    	con.AccessToken = token;
    	return con;
    }

Notes:

- Line 1: All the info we need for the connection
- Lines 3 - 5: Prepare the info for the call to get the token
- Line 6: Get the access token
- Lines 8-15: Prepare the SQL connection string to the Azure SQL database - tweak the properties (like Connect Timeout) appropriately.
- Line 17: Create the connection
- Line 18: Inject the token into the connection object

You'd now be able to use the connection just like you would any SqlConnection object.

## Entity Framework DataContext Changes

If you're using Entity Framework for data access, you'll notice there's no obvious way to use the SqlConnection object that's now configured to access the Azure SQL database. You'll need to create a constructor on your DbContext:

    public class CoolAppDataContext : DbContext
    {
    	public CoolAppDataContext(SqlConnection con)
    		: base(con, true)
    	{
    		Database.SetInitializer&lt;CoolAppDataContext&gt;(null);
    	}
    
    	public DbSet&lt;Product&gt; Products { get; set; }
    
    	...
    }

Notes:

- Line 3: A constructor that accepts a SqlConnection object
- Line 4: Call the base constructor method
- Line 5: Override the initializer for the context's Database object

Now you can use the above methods to construct a SqlConnection to an Azure SQL database using AAD credentials and pass it in to the DbContext - and you're good to go!

## Conclusion

Configuring an application to use Azure AD credentials to connect to an Azure SQL database is straightforward once you have all the pieces in place. There's some configuration you need to ensure is in place, but once it's configured you can stop using SQL Authentication to access your cloud databases - and that's a win!

Happy connecting!

