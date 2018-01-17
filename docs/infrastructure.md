# Infrastructure
The infrastructure of project Eref refers to the tools and technologies used to develop Eref, exclusive of the implementation itself. This
section will be useful to a developer wanting to maintain and further develop Eref.  It describes both the desktop development environment and the
AppHarbor deployment environment for application Eref.

## Hosting Environments
There are 3 hosting environments for Eref: desktop, staging and production. They differ in the database connection string used by each.
The connection string is configured as the value of variable SQLSERVER_CONNECTION_STRING in the `<appSettings>` section of
Web.config. The static value configured there is used by the desktop environment. The static value is overwritten by injection (at AppHarbor)
when Eref is deployed to create either a staging or production release. The transformation files Web.Staging.config and Web.Release.config
play a role in these deployments. The staging deployment at AppHarbor has its Environment variable set to Staging to force Web.Staging.config
to be used upon deployment. This is done in the Settings section of the deployed application. The production deployment at AppHarbor has its
Environment variable set to Release by default. This causes Web.Release.config to be used upon deployment.
 
## Visual Studio Project
The Visual Studio 2015 (Community Edition) project representing application Eref was developed using an ASP.NET Identity 2.0 sample project developed by
Syed Shanu as a starting point. The project is described in the [excellent CodeProject article ASP.NET MVC Security and Creating User Role](https://www.codeproject.com/Articles/1075134/ASP-NET-MVC-Security-And-Creating-User-Role)

The sample project uses the Visual Studio MVC5 project template and makes use of Katana OWIN middleware for user authentication. The use of Katana is built into the ASP.NET Identity 2.0 provider used by the project template, as is explained in the CodeProject article.

The Eref application used the sample project as a starting point. The sample project was downloaded to the folder `C:/Projects/Eref` on the desktop
host’s hard drive and the solution file was renamed to Eref.sln.

The Properties page of the Visual Studio project sets the Local IIS as the server and sets

     http://localhost/Eref
     
as the Project Url. This create an application called Eref under the Default Web Site in IIS and enables project Eref to be run in a desktop version
of IIS under this Url. See the section on configuring IIS below.

The Eref project should be ported to newer versions of Visual Studio as they become available. Visual Studio 2017 is already avaialable and a port
should be performed soon.

## Entity Framework Code First
Since project Eref was just a renaming of the sample project, it was ready to run after the connection string had been established. When it ran for the
first time the SuperAdmin user, sa, was automatically created by method Startup.Configuration on file Eref/Startup.cs. (The class Startup on file
Startup.cs is required by the OWIN specification.) The automatic creation of user sa caused the ASP.NET Identity tables to be created in the ErefDB
database. After these tables were created, file Eref/DataContexts/IdentityDb.cs was created. This file contains a pair of classes which serve to connect 
the data context to the database via the connection string ErefConnection. (See the Database tab for details about the connection string, which must
be obtained and configured in Web.config first!)

Next, the first of the two data contexts used by project Eref, the IdentityDb data context, was initialized in the Visual Studio project by executing
the command 

    PM> Enable-Migrations -ContextTypeName Eref.DataContexts.IdentityDb -MigrationsDirectory DataContexts\IdentityMigrations

in the Package Manager Console, followed by executing the command

    PM> add-migration -ConfigurationTypeName Eref.DataContexts.IdentityMigrations.Configuration "InitialCreate"

Defining class IdentityDb and executing the above two commands is the standard way to begin Entity Framework Code First development with respect to a
data context. Executing the second command automatically created the file DataContexts/IdentityMigrations/*timestamp*_IntialCreate.cs. It is worth
studying this file to see how the various tables of AspNet Identity are defined.

After the IdentityDb data context was created, it was time to create the ReferrralsDB data context. The file Eref/DataContexts/ReferralsDB.cs was
created for this purpose. Like file IdentityDb.cs, this file connects the ReferralsDB data context to the database via the ErefConnection string.

The file defines class ReferralsDB whose initial definition contained the single DbSet called Agencies, corresponding to the desired Agencies table 
in the ErefDB database. The declaration of the DbSet in class ReferralsDB was

    public DbSet<Agency> Agencies { get; set}
 
indicating that the columns of the database table Agencies would be taken from a class called Agency. 

Class Agency is an example of an **entity class**, a class corresponding to a database table. The definition of entity classes is the reason
for the technology name Entity Framework Code First. There are several entity classes in project Eref and
each entity class is defined in a separate subproject called ErefEntities of the Visual Studio Eref solution.

To cause the Agencies table to be built, the following three commands were executed in the Package Manager Console.

    PM> Enable-Migrations -ContextTypeName Eref.DataContexts.ReferralsDB -MigrationsDirectory DataContexts\AgencyMigrations
 
    PM> add-migration -ConfigurationTypeName Eref.DataContexts.ReferralMigrations.Configuration "InitialCreate"
 
 and
 
     PM> update-database -ConfigurationTypeName Eref.DataContexts.ReferralMigrations.Configuration
  
The last of these commands is the one which creates table Agencies in the ErefDB. It does so by executing the Up method in file
DataContexts/ReferralMigrations/*timestamp*_InitialCreate.cs, which was automatically created by the second command. (It was not necessary to execute an 
update-database command for the IdentityMigrations data context because the tables described in IdentityMigrations/*timestamp*_InitialCreate.cs were
created automatically by ASP.NET Identity when the SuperAdmin user, sa, was registered.)

Each additional database change requires a pair of commands: an add-migration command followed by an update-database command. 
Executing an add-migration command creates a .cs file in the folder associated with the ConfigurationTypeName. Study this .cs file before executing the
update-database command. If the database changes indicated in the .cs file are not correct, simply delete the .cs file before running the 
update-database command and then try again.

## Configuring IIS
Development of the Eref application was performed under IIS on the localhost machine. This was done so that the development environment would match the deployment environment at AppHarbor as closely as possible.

The localhost application server, Internet Information Services (IIS), was not pre-installed on the localhost; however, it is part of the operating system that can easily be activated. To activate IIS, go to the Programs section of the Control Panel and turn on the IIS feature:

    Programs > Programs and Features > Turn Windows features on or off > Internet Information Services
  
After checking this box, expand it by clicking the plus sign (+) next to it and go to the section

    World Wide Web Services > Application Development Features
      
In this section, check the checkboxes for

        ASP
        ASP.NET 3.5
        ASP.NET 4.6
        
if they are not already checked. This will cause additional Application Pools to be made available to IIS. 

The Eref application is installed as an application under the Default Web Site in IIS as described in the section describing the Visual Studio Project.
The Basic Settings dialog box for application Eref (accessible from the Actions pane of IIS), will give the physical path to the folder containing the 
source code as

    C:\Projects\Eref\Eref
  
The folder above this (`C:\Projects\Eref`) is the folder containing the project solution file, Eref.sln. Do not change it!
Application Eref must be configured to use the application pool .NET v4.5. in the Basic Settings dialog box. (This application pool became 
available by enabling the features described above.)
 
Finally, change the application identity of the selected application pool (.NET v4.5) to NetworkService. To do this, highlight Application Pools on the
IIS Connections panel. This will cause the available application pools to appear in the IIS body panel. Highlight the .NET v4.5 application pool and
then select Set Application Pool Defaults… to display a dialog box that will enable you to change the identity of the application pool. The dialog box
is reachable either from the context menu of the highlighted application pool (under right mouse click) or from the Actions panel on the right of the
IIS display.

The dialog box contains a section labeled Process Model which contains an entry labeled Identity. Selecting the Identity entry adds an ellipsis next to
the bold ApplicationPoolIdentity. Selecting the ellipsis brings up a dialog box with the pre-selected radio button Built-in account. Select
NetworkService from the dropdown menu associated with this radio button. After approving this selection, the Identity column of the application pool
.NET v4.5 will show NetworkService. This is important! The next section shows how to create user NetworkService in the database.

## SQL Server Express
The desktop version of Eref makes use of a SQL Server Express to store information about referrals. 
Installing SQL Server 2016 Management Studio (SSMS) from Microsoft causes SQL Server Express to be installed as well. The download is large and the
installation from the download takes some time. Just accept all the defaults.

The SQL Server Express database for Eref was created by executing the SQL query

    create database ErefDB  
  
executed inside of SQL Server Management Studio. With this database selected in SSMS, there are two SQL queries that need to be executed to enable IIS
to talk to SQL Server. The first query is

       CREATE USER [NT AUTHORITY\NETWORK SERVICE]
       FOR LOGIN [NT AUTHORITY\NETWORK SERVICE]
       WITH DEFAULT_SCHEMA = dbo;

This query creates the database user NT AUTHORITY\NETWORK SERVICE. The second query is

      EXEC sp_addrolemember 'db_owner', 'NT AUTHORITY\NETWORK SERVICE'
      
This query grants user NT AUTHORITY\NETWORK SERVICE the necessary permissions to communicate with IIS. These same two queries also need to be executed
in the AppHarbor database to prepare it to communicate with IIS. See below for information about the AppHarbor deployment of Eref.

## Git for Windows
Visual Studio 2015 (Community Edition) comes with built-in support for GitHub. A new project can be added to Git source control on the desktop by simply
selecting `Add to Source Control` from the context menu of the Solution file in the Solution Explorer. Once a project is under Git source control it can 
be added to a remote GitHub repository by using tools available through Visual Studio. However, a technique preferred by many developers is to use [Git
for Windows](https://git-for-windows.github.io/). Git for Windows provides a BASH shell interface to GitHub which uses the same set of commands
available at GitHub itself. Git for Windows integrates with Windows Explorer to allow a BASH shell to be opened on a project that has been added to a
desktop Git repository. Simply point Windows Explorer at the folder containing the project solution file and select `Git BASH Here` from the context
menu of the folder to open a Git for Windows BASH shell. Then execute Git commands from this shell window. Git for Windows also offers Git GUI, a
graphical version of most Git command line functions. To open Git GUI simply select Git GUI Here from Windows Explorer.

## GitHub
A Main Street Ministries (MSM) account has been established at github.com with credentials:

    User name: msmapricot
    Email: apricot@msmhouston.org
    Password: <secret>

Git for Windows is used to create a remote to save to the MSM account. The remote is created in the Git BASH shell by opening the shell on the folder which contains the Eref.sln file (folder `C:/Projects/Eref`) and issuing the command

    git remote add origin https://github.com/msmapricot/eref.git
    
Creating this remote only needs to be done once, because Git for Windows stores the remote in Visual Studio. To see this, go to the Team Explorer tab of
the Visual Studio Solution Explorer and select Settings from the Welcome to GitHub for Visual Studio menu. Then select Repository Settings and find the
remote called origin under the Remotes section.

## AppHarbor
AppHarbor (appharbor.com) is a Platform as a Service Provider which uses Amazon Wrb Services infrastructure for hosting applications and Git as a 
versioning tool. When an application is defined at AppHarbor, a Git repository is created to manage versions of the application's deployment.
The Eref application is defined as an application at AppHarbor to create the production repository of the desktop application. The staging version
of the desktop application is defined by a repository called SEref. The Eref repository at AppHarbor is maintained at the Catmaran piad subscription
level and the Seref repository is maintained at the free Canoe subscription level.

A Main Street Ministries (MSM) account has been created at AppHarbor for deployment of the application. The credentials for this account are:

    User name: msmapricot
    Email: apricot@msmhouston.org
    Password: <secret>

Only user msmapricot can deploy directly to an application in the MSM account. Any other user needing to deploy to an application in the MSM account must
be declared a collaborator on this application. A collaborator is a user associated with a different account established at AppHarbor. For example, my
personal account at AppHarbor uses the user name tmhsplb. The user name tmhsplb has been added as a collaborator on the Eref application, thereby
allowing me to deploy to this application.

The remote configured for Eref in Visual Studio is:

    https://msmapricot@appharbor.com/eref.git
    
This remote is configured from a Windows Git BASH shell by the command

    git remote add eref https://msmapricot@appharbor.com/eref.git
    
After the remote is configured in the Git BASH shell, issuing the command

    git push eref master
    
will deploy the master branch of Eref to AppHarbor as application Eref.

If you reset your password at AppHarbor, the 'git push' command will no longer work from the Git BASH shell. You need to have git prompt you for your
new password. To do this on a Windows 10 machine, go to

   Control Panel > User Accounts > Credential Manager > Windows Credentials
   
and remove the AppHarbor entry under Generic Credentials. The next time you push, you will be prompted for your repository password.

An application such as Seref deployed using the free Canoe subscription level at AppHarbor has limitations that make it unsuitable for production
use. Under the Canoe subscription, the IIS application pool of application Seref has a 20 minute timeout, which forces Seref to spin up its resources
again after each 20 minutes of idle time.

The URL of the Canoe version is

    https://eref.apphb.com
    
The free Yocto version of SQL Server is used as an add-on to the Eref deployment. The Yocto version is free but has a limit of 20MB of storage space,
which is adequate for development purposes.  

The remote configured for SEref in Visual Studio is:

    https://msmapricot@appharbor.com/seref.git
    
This remote is configured from a Windows Git BASH shell by the command

    git remote add seref https://msmapricot@appharbor.com/seref.git
    
After the remote is configured in the Git BASH shell, issuing the command

    git push seref master
    
will deploy the master branch of Eref to AppHarbor as application SEref.

The production version of application Eref was created by changing the subsription of application Eref from Canoe to Catamaran. The URL of the production
version is still https://eref.apphb.com.
    
It is possible to use AppHarbor to generate a custom domain name for an application, but this has not been done for the Eref application. The SQL 
Server of the production version of Eref was upgraded from the free Yocto version to the paid uses the Nano version which has a 10GB storage limit. 
The staging application SEref uses the free Canoe service and the free Canoe version of SQL Server.

## jqGrid
Almost every page of the Eref application features a grid produced by the jQuery jqGrid component. It was installed into the Eref project by using
the Package Manager command:

    PM> Install-Package Triand.jqGrid -Version 4.6.0
    
There is a collection of [jqGrid Demos](http://trirand.com/blog/jqgrid/jqgrid.html) that was very helpful during the development of Eref.

## log4net
Application logging is handled by Version 2.0.8 of log4net by the Apache Software Foundation. This package was installed using the Visual Studio NuGet
package manager. The application log for project Eref is maintained as a database table as described in 
[this article](https://logging.apache.org/log4net/release/config-examples.html) describing the AdoNetAppender for log4net. The article includes a script
for creating table Log (renamed AppLog in application Eref). The script must be executed as a query in SSMS to create table AppLog in the database.

The application log is configured by the connection string named ErefConnectionString on Web.config. The value of this connection string is overwritten
when the application is deployed to AppHarbor. See the Connection String section of the Database tab.

## ELMAH
Unhandled application errors are caught by ELMAH. Version 2.1.2 of Elamh.Mvc was installed in project Eref by using the Visual Studio NuGet package
manager. To see ELMAH in action, modify the URL in the browser address bar to, for example,

    eref.apphb.com/Admin/Foo
    
This will generate an unhandled error because the MVC routing system will not be able to resolve the URL. Then go to

    eref.apphb.com/elmah.axd
    
to see that this error has been caught by ELMAH.

Installation of the Elmah.Mvc package adds the necessary DLL's and makes the necessary changes to Web.config to configure ELMAH for use. By default
ELMAH will write to a database table called ELMAH_Error. The DDL Script definition of this table is found in a 
[separate download](https://elmah.github.io/downloads/). Download the DDL Script for MS SQL Server from the referenced web page. The script is a .SQL
file which may be executed as a query inside SSMS to create table ELMAH_Error.

The ELMAH log is configured by the connection string named ErefConnectionString on Web.config. The value of this connection string is overwritten
when the application is deployed to AppHarbor. See the Connection String section of the Database tab.

## Google Maps
The referral letter generated by Eref includes a static Google Map showing the location of Main Street Ministries. A new copy of this map is generated
each time a referral letter is requested. The map is generated using the Google Maps static map API. The key for this API is stored in the
`<appSettings>` section of Web.config under the name GmapKey.

## MkDocs
This document was created using MkDocs as was the [MkDocs website](http://www.mkdocs.org/) itself. MkDocs was installed following the guide
on [this page](https://www.sitepoint.com/building-product-documentation-mkdocs/). An MkDocs document is a static website and can hosted by
any service that supports static sites. This MkDocs document is hosted by [GitHub Pages](https://pages.github.com/). The [Brackets](http://brackets.io/)
open sourece text editor was used to develop the document on the desktop. An MkDocs document uses HTML Markdown for a desktop development version of a
document. GitHub provides a [cheatsheet for Markdown syntax](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

MkDocs provides a built-in preview server which allows the development version to be viewed in a desktop browser at 

    http://127.0.0.1:8000

When it is time to publish a version of a document, use the command

    mkdocs build

to expand the Markdown version of the document into an HTML version into the /site folder. After this define a remote called origin for the document:

    git remote add origin https://github.com/msmapricot/erefdoc

This command references the pre-created GitHub repository erefdoc. Now do

    git add -A

    git commit -a -m 'Comment on new version of eref document'
   
    git push --set-upstream origin master
   
This will push the master branch of the document to the repository identified by the remote called origin. Then go to

    https://msmapricot.github.io/erefdoc/site
   
to view the published document.
   
   