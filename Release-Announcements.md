### OAE Blue Jay (3.0 - 5 December 2013)

The Apereo Open Academic Environment (OAE) project team is pleased to announce the third major release of the Apereo Open Academic Environment; OAE <Blue Jay> or OAE 3.0.

OAE <Blue Jay> introduces a new and improved document previewer, allowing PDF, Text, Word and Excel files to be displayed and embedded within OAE without having to download these files. This replaces the previous document previewer, and all existing documents can automatically be migrated to this new one.

**High-fidelity previews**

The document previewer available in the OAE Phoenix and Alpine Swift releases was image based. This would generate a set of images for the pages in the document. In the OAE <Blue Jay> release, this is replaced with an approach based on the [pdf2htmlEX](http://pdf2htmlex.blogspot.co.uk) Open Source project, generating high-fidelity HTML for each of the pages.

This brings a wide range of advantages, including significantly improved accessibility around browsing and reading a document, native zooming, searching, etc.

**Easy to use viewer**

These new HTML-based previews are displayed in OAE inside of our new simplified document previewer. This allows you to easily flip through pages, request a specific page or zoom in and out of pages.

On top of that, this new document previewer is also fully internationalizable and skinnable.

**Full screen reading**

In order to allow for distraction-less and focussed reading, documents can now be previewed in full screen mode.

**Solid foundation**

This new document previewer is only a first step. It opens up the door to indexing and searching the full text of these documents, as well as allowing for annotations to be made inside of documents.

#### Try it out

OAE Bearded Dragon can be experienced on the project's QA server at http://oae.oae-qa0.oaeproject.org. It is worth noting that this server is actively used for testing and will be wiped and redeployed every night.

The source code has been tagged with version number 3.0.0 and can be downloaded from the following repositories:

Back-end: https://github.com/oaeproject/Hilary/tree/3.0.0  
Front-end: https://github.com/oaeproject/3akai-ux/tree/3.0.0  

Documentation on how to install the system can be found at https://github.com/oaeproject/Hilary/blob/3.0.0/README.md.

The repository containing all deployment scripts can be found at https://github.com/oaeproject/puppet-hilary.



***



### OAE Alpine Swift (2.0 - 28 October 2013)

The Apereo Open Academic Environment (OAE) project team is excited to announce the second major release of the Apereo Open Academic Environment; OAE Alpine Swift or OAE 2.0.

Apereo OAE Alpine Swift brings a wide range of user-facing improvements such as:

- Following
- Video support
- Terms and Conditions
- A number of usability refinements
- Internationalization and Accessibility improvements

#### Try it out

OAE Alpine Swift can be experienced on the project's QA server at http://oae.oae-qa0.oaeproject.org. It is worth noting that this server is actively used for testing and will be wiped and redeployed every night.

The source code has been tagged with version number 2.0.0 and can be downloaded from the following repositories:

Back-end: https://github.com/oaeproject/Hilary/tree/2.0.0  
Front-end: https://github.com/oaeproject/3akai-ux/tree/2.0.0  

Documentation on how to install the system can be found at https://github.com/oaeproject/Hilary/blob/2.0.0/README.md.

The repository containing all deployment scripts can be found at https://github.com/oaeproject/puppet-hilary.

#### Changelog

**Following**

It is now possible to follow other people, allowing for important and public activities from that person to be shown in your personal activity feed. This provides a way to learn more about the collaboration in which your peers are involved, and offers an opportunity for serendipitous discovery of content, discussions, people and groups at the same time.

**Video Support**

An integration with [MediaCore](http://mediacore.com) is now available, allowing for all videos to be uploaded and played inside of OAE. The integration makes it possible for uploaded videos to be transferred to and processed by MediaCore, generating a thumbnail and video that can be played in all browsers. This video is delivered to the OAE user interface through the MediaCore video player.

The MediaCore video support can be enabled and configured for all tenants through the global administration UI.

**Terms and Conditions**

A Terms and Conditions agreement feature is now available. If configured, this agreement is presented to all users when they sign in for the first time and they will need to accept it before they can use the system. The terms and conditions can be internationalized in multiple languages at the same time.

The Terms and Conditions agreement can be enabled and configured on a tenant-by-tenant basis through the administration UI.

**My Groups**

The My Groups page now follows the same behaviour as the My Library and My Discussions pages when being viewed by other people. Only groups that can already be seen by the person viewing your My Groups page will be shown.

**Monitoring**

Integration with Google Analytics is now available out of the box, allowing for a better understanding around usage patterns and total usage to be gained. This can be enabled and configured on a tenant-by-tenant basis through the administration UI.

**TinCan API**

OAE is now capable of pushing activities to a Learning Record Store using the [TinCan](http://tincanapi.com) or Experience API. This API is a new specification for learning technology that makes it possible to collect data about the wide range of experiences a person has (online and offline). This API captures data in a consistent format about a person or group’s activities from many technologies, of which OAE is now one.

The TinCan API support can be enabled and configured on a tenant-by-tenant basis through the administration UI.

**Internationalization**

The OAE project has decided to adopt [Crowdin](http://crowdin.net/project/apereo-oae) as its official translation platform. This allows for OAE to be translated through the Crowdin web UI without needing to set up a local development environment.

In addition to making it easy to translate all of the OAE keys, it provides translation suggestions based on a Translation Knowledge Base and machine translations. Crowdin provides a very useful overview of the translation completeness of the different supported languages and makes it easy to see which keys still need to be translated. A synchronisation script has also been provided that will automatically make new keys available to Crowdin and will pull the newest translations into the code base.

Through this Crowdin integration, OAE Alpine Swift includes the following complete translations:

- Catalan
- Chinese
- Dutch
- French
- German
- Spanish
- Valencian

A special thanks goes to Frederic Dooremont, Toni Devís López, Samuel Gutiérrez Jiménez-Peña, Harry Wang, Yildiray Ogurol and Mark Breuker for providing these translations.

Finally, OAE will now also automatically detect the preferred language based on the user's browser settings, according to internationalization best practices. We will only default to the language configured in the admin UI if this has not been set.

**CSV User Import**

A tenant can now be pre-populated with a large set of users by uploading a CSV user file. These accounts can be associated with a specific authentication strategy and existing accounts will not be overridden. The CSV upload can be done by a global or a tenant administrator.

**Usability Improvements**

A long list of usability refinements have been added. Some of the more notable ones are:

- Improved previews for activity feed items
- Improved button wording and clarity
- Automatic link recognition in comments and discussions
- Keyboard acccessibility improvements (e.g. refocussing modal triggers after modal is closed)
- etc.

**Automated testing**

A number of improvements have been made to the automated QUnit tests, including making sure that all tests are automatically run after every commit.



***



### OAE Phoenix 1.2 (23 August 2013)

The Apereo Open Academic Environment (OAE) project team is pleased to announce the second minor release of the OAE Phoenix release; OAE Phoenix 1.2. 

OAE Phoenix 1.2 upgrades the Hilary back-end to Node.js 0.10 and brings some user-facing improvements like Administration UI improvements, CAS authentication mappings and a number of bug fixes. In the background, ongoing work on performance testing, automated UI testing and automated cluster deployment has been going on, which is now nearing completion.

#### Try it out

The source code has been tagged with version number 0.2.2 and can be downloaded from the following repositories:

Back-end: https://github.com/oaeproject/Hilary/tree/0.2.2  
Front-end: https://github.com/oaeproject/3akai-ux/tree/0.2.2  

Documentation on how to install the system can be found at https://github.com/oaeproject/Hilary/blob/0.2.2/README.md.

The repository containing all deployment scripts can be found at https://github.com/oaeproject/puppet-hilary.

#### Changelog

**CAS Authentication mappings**

It is now possible to map the OAE display name, e-mail, locale and timezone against CAS attributes released by a CAS authentication server. All of this can be configured on the fly through the Administration UI.

**Administration UI improvements**

- A tenant's host name can now be modified on the fly through the administration UI.
- A number of browser caching and back-end caching consistency bugs in the Tenants API have been fixed.
- Added appropriate validation when creating new tenants
- Refactored the Tenants API and increased test coverage

**Mime type recognition**

Improved mime type recognition has been put in place for uploaded files, ensuring that files uploaded from any browser will be appropriately recognized.

**Accessibility improvements**

All modal dialogs that have a type ahead component as the first focusable element will now receive appropriate focus when opening the modal. Next to that, a number of text alternatives for non-textual content have been added.

**API improvements**

All feeds that support paging now return a `nextToken` parameter that can be used to request the next page of results. This takes away the need to know which field to use for paging and should make it easier to use the REST APIs. A number of improvements have been made to the OAE UI APIs as well.

**UI translations**

The Spanish OAE translation is now back at 100% completeness, thanks to Samuel Gutiérrez Jiménez-Peña.

**Node 0.10 upgrade**

The Hilary back-end, as well as the activity and preview processor nodes, have been upgraded from Node.js 0.8 to 0.10 after extensive performance testing. 

**Production improvements**

A bug that was preventing Etherpad documents from being published when using multiple Etherpad servers has been resolved.



***



### OAE Phoenix 1.1 (01 August 2013)

The Apereo Open Academic Environment (OAE) project team is pleased to announce the first minor release of the OAE Phoenix release; OAE Phoenix 1.1. 

OAE Phoenix 1.1 brings a number of new features, like LDAP authentication and new UI translations, a number of end-user improvements and improves OAE's ease of deployment and development. However, OAE Phoenix 1.1 is mostly a technical release, with lots of work going on in the background around performance testing and automated UI testing.

The first OAE production environment has also been set up and launched, and currently supports 3 institutions on a single instance. All tenants have been skinned, and most of the configuration has taken place, with 2 tenants using Shibboleth authentication and 1 tenant using CAS authentication:

- Georgia Tech: https://oae.gatech.edu
- Marist College: https://marist.oaeproject.org
- University of Cambridge: https://collab.lib.cam.ac.uk

This production environment has been updated to run the OAE Phoenix 1.1 release.

#### Try it out

The source code has been tagged with version number 0.2.1 and can be downloaded from the following repositories:

Back-end: https://github.com/oaeproject/Hilary/tree/0.2.1  
Front-end: https://github.com/oaeproject/3akai-ux/tree/0.2.1  

Documentation on how to install the system can be found at https://github.com/oaeproject/Hilary/blob/0.2.1/README.md.

The repository containing all deployment scripts can be found at https://github.com/oaeproject/puppet-hilary.

#### Changelog

**LDAP authentication**

It is now possible to authenticate against an LDAP server and pull the user's pull basic profile information from LDAP. All of this can be configured and mapped on the fly through the Administration UI.

**UI translations**

- A full Polish translation for OAE is now available (thanks to Katarzyna Napiórkowska)
- A full Italian translation for OAE is now available (thanks to Renato Strazzulla and Toni Devís López)
- A full Valencian translation for OAE is now available (thanks to Toni Devís López)
- The French OAE translation is back at 100% completeness (thanks to Frederic Dooremont)
- The German OAE translation is back at 100% completeness (thanks to Yildiray Ogurol)
- OAE now has a more complete Dutch translation (thanks to Mark Breuker)

**Discussion email notifications**

Email notifications are now being sent for important discussion activities, like someone sharing a discussion with you, someone posting in a discussion you manage or you participated in, etc.

**Activity for restored revisions**

Restoring a content or document revision now generates an activity in the activity feed.

**Automated testing improvements**

The following QUnit tests have been added to the User Interface, although more work will be going into these during the next release:

- Test that checks for code and styling issues in the JavaScript (using JSHint)
- Test that checks for any translation keys in the various bundles that are no longer being used
- Test that checks for any translation keys in the HTML and JavaScript that have not been translated
- Test that checks for translation keys that have been duplicated inside a bundle or across widget bundles
- Test that checks for hard-coded English strings that have not been internationalized
- Test that checks that all CSS is in line with the project's style guide
- Test that checks that all JavaScript is in line with the project's style guide
- Test that checks for WCAG 2.0 Compliance - 1.1.1 Non-text Content / Text Alternatives issues
- Test that checks the completeness of each available translation (limited to core bundles only)

Important work has gone into building out a set of automated functional UI tests. This is currently still in development and expected to be included in the next release.

The project's performance tests have also been significantly updated to reflect the latest state of the UI. This work is available at https://github.com/oaeproject/node-oae-tsung.

**Ease of development**

A number of issues that showed up when deploying and developing OAE on Windows have been addressed, making it more straightforward to get the system up and running on Windows.

Multiple improvements have also been made to the project's installation guide.

**Performance improvements**

The revisions feed for collaborative documents has been made more scalable, with the revision content being available on the individual revisions now instead of the global revision overview feed.

**Production improvements**

Improvements have been made to the re-index all operation, including the ability to re-index all discussions in the system.

Improvements have also been made to the re-generate previews operation, including supporting the re-generation of previews for individual content items.

It is now also possible for global and tenant admins to see the group members of all groups



***



### OAE Phoenix (1.0 - 27 June 2013)

The Apereo Open Academic Environment (OAE) project team is pleased and excited to announce the first release of the Apereo Open Academic Environment; OAE Phoenix 1.0. This release consists of the first production ready release of Apereo OAE and focusses on providing support for various forms of academic collaboration.

Apereo OAE is designed as a multi-tenant platform that can be run at large scale, allowing for a single installation to support multiple institutions at the same time. OAE Phoenix 1.0 is a first and important step and attempts to provide a basic, but solid foundation that can be used as the basis for many more collaborative scenarios.

#### Try it out

The source code has been tagged with version number 0.2.0 and can be downloaded from the following repositories:

Back-end: https://github.com/oaeproject/Hilary/tree/0.2.0  
Front-end: https://github.com/oaeproject/3akai-ux/tree/0.2.0  

Documentation on how to install the system can be found at https://github.com/oaeproject/Hilary/blob/0.2.0/README.md.

The repository containing all deployment scripts can be found at https://github.com/oaeproject/puppet-hilary.

#### Main features

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