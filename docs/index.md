# Background
Per agreement, a client referred to Operation ID - a ministry of Main Street Ministries - by a partner agency must present a referral letter provided by
the agency to be eligible for identification services. The Electronic Referral System for Operation ID (Eref) is designed to support the generation of
referral letters by partner agencies in a way which will both simplify the process of serving clients and enhance record keeping by agencies and
Operation ID alike. Eref will be available as a password protected website to all subscribing partner agencies. This document describes the design and
implementation of Eref. The Infrastructure tab describes how project Eref is maintained on a desktop host and how it is deployed to the .NET hosting
service AppHarbor.

## Users
Eref is a role-based system. Each registered user of Eref will be assigned a user role by the Eref administrator. The role that a user is in will determine the Eref features available to the user.

Most users of Eref will be members of agencies referring clients to Operation ID and will be in the role of Client Advocate. In the role of Client
Advocate, a user will be able to refer new clients by generating referral letters and to review identification services requested for existing clients.
A Client Advocate will only be able to refer/review clients being assisted through his/her agency.

Users of Eref registered as members of Main Street Ministries will be assigned a role that will enable them to review clients referred by all partner
agencies. They will also be able to generate a new referral letter for an agency client in case the services requested by a client change on the day 
the client appears at Operation ID.

The remainder of this document will focus on how Eref will be used by a user in the role of Client Advocate.

## The Client Advocate
A user in the role of Client Advocate has the responsibility of interviewing prospective Operation ID clients to determine the identification services
they need and to determine the documents they currently have in support of the services they seek. The services provided by Operation ID include

*	Birth Certificate
*	Texas ID
*	Texas DL

There are several other services that are occasionally requested and these will be discussed below.

Operation ID will provide vouchers for requested services to enable clients to pay for them. It is a client’s responsibility to obtain the identification
document by presenting a voucher and supporting documents to the appropriate agency. In the case of an out-of-state birth certificate, Operation ID will
complete the necessary paperwork and submit it together with a voucher to the appropriate out-of-state agency.

In addition to registering a client through Eref, a Client Advocate may also register dependents of a client together with the services requested for each
dependent.

With the click of a button, a Client Advocate user will generate a referral letter summarizing the services requested by a client (plus any dependents)
together with the documents available in support of the requested services. 
