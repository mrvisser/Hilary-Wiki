### OAE Phoenix 1.1 (01 August 2013)



***

### OAE Phoenix 1.0 (27 June 2013)

The Apereo Open Academic Environment (OAE) project team is pleased and excited to announce the first release of the Apereo Open Academic Environment; OAE Phoenix 1.0. This release consists of the first production ready release of Apereo OAE and focusses on providing support for various forms of academic collaboration.

Apereo OAE is designed as a multi-tenant platform that can be run at large scale, allowing for a single installation to support multiple institutions at the same time. OAE Phoenix 1.0 is a first and important step and attempts to provide a basic, but solid foundation that can be used as the basis for many more collaborative scenarios.

##### Try it out

The source code has been tagged with version number 0.2.0 can be downloaded from the following repositories:
 
Back-end: https://github.com/oaeproject/Hilary/tree/0.2.0
Front-end: https://github.com/oaeproject/3akai-ux/tree/0.2.0
 
Documentation on how to install the system can be found at https://github.com/oaeproject/Hilary/blob/0.2.0/README.md.
 
The repository containing all deployment scripts can be found at https://github.com/oaeproject/puppet-hilary.

##### Main features

**Administration UI**
- Global administration panel allowing for tenant creation, management and configuration
- Tenant administation panel allowing for tenant maintenance and configuration
- Tenant skinning panel
- Tenant permeability allowing for collaboration across institutions

**Users and user management**
- Internal user creation
- CAS integration, configurable through Admin panel
- Shibboleth integration, configurable through Admin panel
- Facebook, Twitter and Google authentication integration
- Basic user profiles
- User permissions
- User preferences
- Profile pictures

**Groups and group management**
- Create group
- Flexible and poweful group permissions
- Group summary page
- Manage group members
- Various joinability options

**Content and content management**
- File upload
- Add links
- Create collaborative documents (Etherpad integration)
- Share files, links, collaborative documents with users and groups
- Content and discussion profiles
- Comment on and discuss content items
- Flexible and powerful content permissions
- Revisions and revision history
- User and groups libraries

**Discussions**
- Create discussions
- Share discussions with users and groups
- Discussion profile pages
- Flexible and powerful discussion permissions

**Activities and notifications**
- User and group activity feeds (activitystrea.ms specification)
- Notifications for important activities
- E-mail notifications

**Search**
- Search users
- Search groups
- Search content
- Search discussions