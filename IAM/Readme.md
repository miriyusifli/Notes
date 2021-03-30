# Identity and Access Management

**Authentication** is a mechanism by which computer checks that the user is really the one he prentends to be. 

**Authorization** is a related mechanism by which the computer determines whether to allow of deny specific action to a user.


## Directory Servers
Directory Servers are just databases that store information and built with the primary purpose to provide shared data storage to applications.

<ins>**Directory Servers vs Databases**<ins>

- While databases are built for application-specific data model, 
directory servers usually extend standardized data model which improves interoperability. While

- While databases may be heavyweight and expensive to scale, directory servers are designed to be lightweight and provide massive scalability. That makes directory
servers almost ideal candidates for shared account database. 

Directory servers are good for *storing*, *searching* and *retrieving* data. While the user account 
data often contain entitlement information (permissions, groups, roles, etc.), identity stores are not well suited to evaluated them. A directory server can provide
information what permissions an account has, but it is not designed to make a decision whether to allow or deny a specific operation. Directory servers do not contain
data about user sessions. It means that directory servers do not know whether user is currently logged in or not. Directory servers provide only the very basic capabilities.
There are plug-ins and extensions that provide partial capabilities to support authentication and authorization. But that does not change the funcdamental design 
principles. Directory servers are databases, not authentication or authorization servers. They should be used as such. The right way to do it is to use authentication
server instead of directory server. Access Management (AM) technologies can provide that.

## LDAP
Lightweight Directory Access Protocol (LDAP) is a standart protocol for accessing directory services. It is a very efficient binary protocol that was designed to
support massively distributed shared databases. It has small set of well-defined simple operations. The operations and the data meta-model implied by the protocol
allow very efficient data replication and horizontal scalability of directory servers. These benefits often come at the expense of slow write operations. As
identity data are often read but seldom modified, slower writes are usually a perfectly acceptable trade-off. Therefore, LDAP-based directory servers were and in
many places still remain, the most popular databases for identity data. 


## Single Directory Server Myth
Shared directory server makes user management easier. However, this is not a complete solution and there are serious limitations to this approach. The heterogeneity
of information systems makes it nearly impossible to put all required data into a single directory system. There may be additional sources of information. For example
Management information system may be responsible for determination of user’s roles (e.g. in project-oriented organizational structure). Inventory management 
system may be responsible for assigning telephone number to the user. The groupware system may be an authoritative source of the user’s e-mail address and 
other electronic contact data. There are usually 2 to 20 systems that provide authoritative information for a single user. Therefore, there is no simple way 
how to feed and maintain the data in the directory system. The single directory approach is feasible only in very simple environments or in almost entirely
homogeneous environments. In all other cases there is a need to supplement the solution by other
identity management technologies.

<ins>**Possible drawbacks**</ins>
- Many complex applications need local user
database. They must store the copies of user records in their own databases to operate efficiently.  Keeping the copy synchronized with the directory data may 
seem like a simple task. But it is not. Additionaly, there are legacy systems which usually cannot access the external data at all

- Some services need to keep even more state than just a simple database record. For example file
servers usually create home directories for users. While the account creation can usually be done
in on-demand fashion (e.g. create user directory at first user log-on), the modification and deletion
of the account is much more difficult. Directory server will not do that.

- Perhaps the most painful problem is the complexity of access control policies. Role names and
access control attributes may not have the same meaning in all systems. Different systems usually
have different authorization algorithms that are not mutually compatible. While this issue can be
solved with per-application access control attributes, the maintenance of these attributes is seldom
trivial. If every application has its own set of attributes to control access control policies then the
centralized directory provides only a negligible advantage.

This does not mean that the directory servers or other shared databases are useless. Quite the
contrary. They are very useful if they are used correctly. They just cannot be used alone. More
components are needed to build a complete solution

## Access Management
While directory systems are not designed to handle complex authentication, access management
(AM) systems are built to handle just that. Access management systems handle all the flavors of
authentication, and even some authorization aspects. The principle of all access management
systems is basically the same:

1. Access management system gets between the user and the target application. This can be done
by a variety of mechanisms, the most common method is that the applications themselves
redirect the user to the AM system if they do not have existing session.
2. Access management system prompts user for the username and password, interacts with
authentication token, creates a challenge and prompts for the response or in any other way
initiates the authentication procedure.
3. User enters the credentials
4. Access management system checks the validity of credentials and evaluates access policies.
5. If access is allowed then the AM system redirects user back to the application. The redirection
usually contains an access token: a small piece of information that tells the application that the
user is authenticated.
6. Application validates the token, creates a local session and allows the access.


![How access management works](img/access_management_procedure.png)


After that procedure, the user works with the application normally. Only the first access goes
through the AM server. This is important for AM system performance and sizing, and it impacts
session management functionality


In the LDAP case, it is the application
that prompts for the password. In the AM case, the Access Management server does everything.

## Web Single Sign-On
Single Sign-On (SSO) systems allow user to authenticate once, and access number of different
system after that. There are many SSO systems for web applications, however it looks like these
systems are all using the same basic principle of operation. This general access management flow is
described below:
1. Application A redirects the user to the access management server (SSO server).
2. The access management server authenticates the user.
3. The access management server establishes session (SSO session) with the user browser. This is
the crucial part of the SSO mechanism.
4. User is redirected back to the application A. Application A usually establishes a local session with
the user.
5. User interacts with application A.
6. When user tries to access application B, the application B redirects user to the access
management server.
7. The access management server checks for existence of SSO session. As the user authenticated
with the access management server before, there is a valid SSO session.
8. Access management server does not need to authenticate the user again and immediately
redirects user back to application B.
9. Application B establishes a local session with the user and proceeds normally.
The user usually does not even realize that he was redirected when accessing application B. There is
no interaction between the redirects and the redirects and the processing on the access
management server is usually very fast. It looks like the user was logged into the application B all
the time

![How single sign-on works](img/single_sign_on.png)
