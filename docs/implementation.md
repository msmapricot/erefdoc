# Implementation
Application Eref is implemented as an ASP.NET MVC 4.5 project using Visual Studio 2015 (Community Edition). It uses ASP.NET Identity 2.0 to define a set
of user roles. Each user role is associated with a separate MVC controller. Controller inheritance is used to share the functionality of a client
editor across user roles. 

The graphical user interface of Eref is built using Bootstrap 3.0.0. Each user role is associated with its own layout file which defines a Bootstrap
navbar containing links to the Eref features available to users in the role. The ASP.NET Identity system ensures that a user in a specified role cannot
visit any pages outside of those allowed to users in that role.

The source code is the ultimate reference for the implementation.

## The Superadmin
Eref defines a pre-registered superadmin user who has privileges to

* add new agencies
* create new roles
* invite new users to register in a pre-determined role

The credentials for the superadmin are stored in the `<appSettings>` section of file Web.config and are retrieved on file Startup.cs.

A new user of Eref has to be invited to register by the superadmin. This is done to prevent new users from selecting their role. The superadmin will
contact a new user to find out the user name and email address he would like to use to register. The user name and email address will be added to the
**Invitations** table. More than one user can share the same email address. This is not the default beahavior; it is enabled by the setting

    RequireUniqueEmail = false
   
in method ApplicationUserManager.Create on file App_Start/IdentityConfig.cs

When the user registers for Eref, he must use the same user name and email that he provided to the superadmin together with
a password of his choosing.

## Role Controllers
There is an MVC controller defined for each role defined by the superadmin. Each controller defined for a role inherits from ClientManagerController,
which in turn inherits from UsersController. The role controllers generate the views of application Eref. The implementation of each role controller
defines methods that are accessible through the menubar defined on the layout file for the role. For example, the AgentController - which implements
the Client Advocate role - contains methods Home and OurClients invoked from the menubar defined on file

   ~/Shared/_Agent.cshtml.

This is the layout file for the Client Advocate role. Each view returned by the AgentController includes this layout file, thereby ensuring that a
user in the role of Client Advocate will only have access to methods defined by the AgentController.

Each other role controllers is implemented the same way: each has a defined layout file that is included in each view returned by the controller.
The layout file defines a menubar that specifies the methods that users in the role can execute.

## The ClientManagerController
The ClientManagerController implements the editor functionality available to the different roles. Each role controller derives from
ClientManagerController.

## The Data Manager
All access to database tables managed by the the ReferrralsDB data context is handled by the DataManager class. The CRUD operations available in each 
role are defined by the ClientManagerController and implemented by the DataManager.

## The UsersController
The UsersController controls access to the ASP.NET Identity tables used to store registered users and the roles they are in. The method
UsersController.Index is the entry point for an authenticated user. The role an authenticated user is in determines the method the user will be
redirected to from this entry point.

## jqGrid
The entire implementation of application Eref is structured around instances of jqGrid appearing in MVC Views. Each jqGrid is initially populated
by a call to an MVC action made through the url property of the grid. For example, the clientsGrid on file AgencyClients.cshtml is initially populated
by the call

    "@Url.Action("GetAgencyClients", "Agent")"
        
which is the value of the url argument to grid clientsGrid. A grid may have an associated subgrid. The clientsGrid has a subgrid called the
supportingDocumentsGrid. A subgrid is populated when a row of its supergrid is selected. This happens through the function which is the value of the
onSelectRow argument of the supergrid. For example, when a row of grid clientsGrid is selected, the supportingDocumentsGrid is populated by the value of
the MVC action call 

    "@Url.Action("GetSupportingDocuments", "Agent")"
        
which is the value of the url argument to subgrid supportingDocumentsGrid.

Each instance of a jqGride defines a *pager*. The pager of a jqGrid defines the CRUD operations supported by the grid. Each CRUD operation is
implemented by an MVC action of the role controller associated with the grid. 

Initial population of a grid, grid pagination, grid searching and grid CRUD operations are all supported by server side code.

There is a collection of [jqGrid Demos](http://trirand.com/blog/jqgrid/jqgrid.html) that was very helpful during the development of Eref.

## NowServing
An important concept in the implementation of Eref is the concept of the client currently being served by a registerd Eref user. When a row in the
clientsGrid defined on AgencyClients.cshtml is selected, the function that is the value of the onSelectRow argument of the grid is invoked. When the
function is invoked, the value passed to its nowServing argument is the id associated with the client represented by the selected row. The code
shows that the function is used to populate the subgrid supportingDocumentsGrid through the action call

    "@Url.Action("GetSupportingDocuments", "Agent")"

which is the value of the url argument to the subgrid (see the section on the jqGrid). In the source code of AgencyClients.cshtml, the line above
this action call is

    postData: { nowServing: nowServing }
    
The effect of this line is to have the value of the JavaScript variable nowServing be added to the post to method GetSupportingDocuments. Inspecting
the definition of this method (found on file ClientManagerController due to controller inheritance) it is seen that the method has an optional
argument called nowServing. MVC data binding will cause this variable to be bound to the JavaScript variable in the post.

The subsequnet call to SetNowServing (found on file UsersController due to controller inheritance) will cause the NowServing database field in table
**AspNetUsers** of the usr who clicked on the client row in the clientsGrid to receive the value of JavaScript variable nowServing.

Knowing which client is being served is necessary when either the

      Services for Selected Client
      
or the

      Letter for Selected Client
      
button at the bottom of AgencyClients.cshtml is clicked. Tracing what happens is left as an exercise to the reader.

## Visits
In most cases a client will visit an agency once and only once. But there will be times when a client will pay a return visit to an ageny for help
with additional services. For this reason there is a one-to-many relationship between the **Clients** table and the **Visits** table. The Client_Id
field of the **Visits** table is a foreign key defined by the Id field of table **Clients**.

The first time a client visits an agency, a record corresponding to this visit is created in the **Visits** table. This happens by the call to method 
AddClientVisit in method GetClientVisits of the DataManager. The parameter nowServing of this method will be set to the Id of the client being served.

Method AddClientVisit returns a Visit entity which has its Id field set (as a side effect) to the index of the visit which has been inserted into the 
**Visits** table. A [comment](https://stackoverflow.com/questions/5212751/how-can-i-get-id-of-inserted-entity-in-entity-framework) appears at this point 
in the code to highlight this side effect. When method AddClientVisit returns control to method GetClientVisits, the visit entity it returns has its Id 
property copied to a VisitViewModel object, vvm, being constructed:

    vvm.Id = visit.Id

The same [comment](https://stackoverflow.com/questions/5212751/how-can-i-get-id-of-inserted-entity-in-entity-framework) is repeated at this point in
method GetClientVisits.

Without the setting to the VisitViewModel made above, a visit being selected from the table on Visits.cshtml would passs 0 as the value of nowVisiting
when the visit's row is selected:

    onSelectRow: function (nowVisiting)
    
With the setting to the VisitViewModel made above, the Id of the selected row will be passed as the value of nowVisiting which will ultimately point
back to the identity of the client being served and will allow the Supporting Documents subgrid on Visits.cshtml to be populated.

## Referral Letters
The referral letter for a client is generated from data collected in the **Clients** and **Visits** tables. (See the database tab.) Each client is
represented by a single
record in the **Clients** table. In addition to a client's name, the **Clients** table contains a boolean data field for each supporting document
the client has presented to the Client Advocate that he/she is working with. There is a text data field called Notes in the **Clients** table that 
is used to collect information about a client not captured by any other data field. If a client has dependents, each dependent will be entered
into the **Clients** table with a pointer back to the client. This is done through the Family data field in the **Clients** table. The Family
data field of the head of a household will be set to 0. The Family data field of any dependent of a client will be set to the client Id of the client
who is the head of the household.

The **Visits** table is related to the **Clients** table by the foreign key stored in its Id field. The table is used to collect service requests for 
a client. Each record in the **Visits** table is related to a client in the **Clients** table through the Id foreign key and stores the service requests
made by a client on a particuar date. 

A Client Advocate with an understanding of the Operation ID identification requirements will be able to determine whether the supporting documents
presented by a client are sufficient for the services a client seeks during a given visit. If the supporting documents are sufficient, the Client
Advocate may use Eref to generate a referral letter that will list the supporting documents presented together with the services requested by a client.
If the supporting documents are clearly not sufficient for the services sought, the Client Advocate may work with the client to obtain the documents
required before sending him/her to Operation ID. If there is a question about the sufficiency of the supporting documents, the Client Advocate may
generate a referral letter and send the client to Operation ID where a determination will be made. A future release of Eref will use a built in set
of rules to help determine the sufficiency of documents presented in support of identification services.

When a client is entered into the **Clients** table, a unique 6 letter token (example: DSQMNJ) will be stored with the client's record in the
**Clients** table. This token will be included in the referral letter as means of uniquely identifying a client in case the client's name is not
sufficient. The token is also an anti-forgery safeguard. Any attempt to forge a referral letter will be defeated by the requirement of containing
a 6 letter token that matches the token associated with a client in the **Clients** table.
