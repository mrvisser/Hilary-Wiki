# Config API

[Source Code](https://github.com/sakaiproject/Hilary/blob/master/node_modules/oae-config/lib/api.js)

## Requirements

* Every tenant needs to be able to configure their instance from an administrative UI
* Global administrators need to be able to configure every instance and the default configuration for new instances from an administrative UI
* Configuration changes need to be picked up by tenants without the need to restart the server

## Data Model

The goal of the configuration schema is to allow for quick retrieval of specific values and batch retrieval of values grouped by tenant. The following schema is what we came up with:

### Column family: Config
 
<table>
  <thead>
    <tr>
      <th>Row Key (Tenant)
  <tbody>
    <tr>
      <th>tenantId
      <td>oae-authentication/facebook-authentication/facebook-authentication-enabled
      <td>oae-authentication/local-authentication/local-authentication-enabled
      <td>oae-authentication/twitter-authentication/twitter-authentication-enabled
</table>

The column family Config has one column per configuration option. The value and type of the config option can vary and the options can be retrieved by the alias (tenantId) of the tenant. By default, the Config column family is empty until administrators start changing default values that are specified in the configuration files per module.

## Data Feeds

The configuration has 3 different data feeds available.

* Schema feed
* Suppressed simplified feed

### Schema Feed

The schema feed gives back a detailed feed with information on the different configuration options and values. The schema is primarily used for constructing the Admin UI.

```javascript
{
    'oae-tenants': {
        title: 'OAE Tenant Module',
        actions: {
            name: 'Tenant Admin Action',
            description: 'Actions a tenant admin is allowed to do',
            elements: [
                {
                    allowStop: {
                        description: 'Allow a tenant to stop the tenant server',
                        defaultValue: false,
                        name: 'Stop tenant',
                        tenantOverride: false,
                        suppress: true,
                        type: 'boolean'
                    }
                }
            ]
        }
    }
}
```

### Suppressed simplified feed

The suppressed simplified feed gives back a simplified feed with only the necessary information to make use of the configuration values inside of the application. It will also take into account various permissions.

**Note:** Only tenant and global administrators will get back configuration values for options that are marked as `suppressed`.

```javascript
{
    'oae-tenants': {
        actions: {
            allowStop: false
        }
    },
    'oae-authentication': {
        local: {
            allowAccountCreation: true,
            enabled: true
        },
        google: {
            enabled: true
        },
        twitter: {
            enabled: true
        },
        facebook: {
            enabled: false,
            appid: '',
            secret: ''
        }
    },
    'oae-content': {
        visibility: {
            files: 'public',
            sakaidocs: 'public',
            links: 'public'
        },
        storage: {
            backend: 'local',
            'local-dir': '/Users/Bert/sakai/files',
            'amazons3-access-key': '<access-key>',
            'amazons3-secret-key': '<secret-key>',
            'amazons3-region': 'us-east-1',
            'amazons3-bucket': 'oae-files'
        }
    }
}
```

## Configuration files

The configuration is aggregated from different files and stored configurations. Every configurable module has a 'config' directory with one or more configuration files. The name of these configuration files can be anything but they must be javascript files with valid javascript content. Updates to any configuration will be in effect immediately.

### Configuration elements

There is a set of elements that can be defined in the configuration files to render options in the UI. Currently, the following configuration types are supported:

* Collection of fields (ArrayFieldSet)
* Checkbox (Boolean)
* Input field (Text)
* Input area (Textarea)
* Radio button group (Radio)
* Drop down (List)
* Multiple choice (ArrayMultipleChoice)
* Expandable list (ExpandableObject)
* Tree structure (TreeStructure)

Initialization of configuration types is shown in the examples below.

Each configuration field takes an optional options object. The object contains two parameters `suppress` and `tenantOverride`
* suppress: To show or hide a configuration field for non-managers
* tenantOverride: If tenant administrators can changes the default setting

#### Collection of Fields

A collection of fields can be used to logically group together multiple fields. It's mainly used to create a schema for configuration to follow.

```javascript
new Fields.ArrayFieldSet({
    "title": new Fields.Text('Set link title', 'Set link title', 'Example Title', true),
    "href": new Fields.Text('Configure link', 'Configure link', 'Example Link', true),
    "newWindow": new Fields.Boolean('Open in new window', 'Open in new window', true, true)
})
```

#### Checkbox

A checkbox is usually used where a configuration value can only have a true or false value.

```javascript
new Fields.Boolean('Open in new window', 'Open in new window', true, {
    'tenantOverride': false,
    'suppress': true
})
```

#### Input Field

An input field provides a basic text input element. For larger chunks of text, like descriptions, the input area should be used.

```javascript
new Fields.Text('Configure link', 'Configure link', 'Example Link', {
    'tenantOverride': false,
    'suppress': true
})
```

#### Input area

The input area renders a textarea element for larger chunks of texts like descriptions.

```javascript
new Fields.Textarea('Join request body', 'Body of email message sent out to managers when someone wants to join the group', '${user} has requested to join your group: ${group}.', {
    'tenantOverride': false,
    'suppress': true
})
```

#### Radio button group

A radio button group provides the users with a selection of choices. Only one can be selected.

```javascript
new Fields.Radio('Default content copyright', 'Default copyright for new content', 'nocopyright', true, [
    {
        "name": "Creative Commons",
        "value": "creativecommons"
    },
    {
        "name": "Copyrighted",
        "value": "copyrighted"
    },
    {
        "name": "No copyright",
        "value": "nocopyright"
    },
    {
        "name": "Licensed",
        "value": "licensed"
    },
    {
        "name": "Waive copyright",
        "value": "waivecopyright"
    }
], {
    'tenantOverride': false,
    'suppress': true
})
```

#### Drop down

Provides a drop down list where users can make one selection.

```javascript
new Fields.List('Default Access', 'Default content access', 'public', true, [
    {
        "name": "Public",
        "value": "public"
    },
    {
        "name": "Logged in users",
        "value": "everyone"
    },
    {
        "name": "Private",
        "value": "private"
    },
],{
    'tenantOverride': false,
    'suppress': true
})
```

#### Multiple choice

The multiple choice type returns an object for an Array of options that has an Array of configuration values. Each option in the array will have the possibility to enable/disable the specified configuration options.

```javascript
new Fields.ArrayMultipleChoice("Role can share content", "Default content sharing for roles", {
        "public": ["editor", "viewer", "everyone", "anon"],
        "everyone": ["editor", "viewer", "everyone"],
        "private": ["editor", "viewer"]
    }, [
    {
        "name": "Public content",
        "value": "public"
    },
    {
        "name": "Content for logged in users",
        "value": "everyone"
    },
    {
        "name": "Private content",
        "value": "private"
    }
], [
    {
        "name": "Editors",
        "value": "editor"
    },
    {
        "name": "Viewers",
        "value": "viewer"
    },
    {
        "name": "Logged in users",
        "value": "everyone"
    },
    {
        "name": "Anonymous users",
        "value": "anon"
    }
], {
    'tenantOverride': false,
    'suppress': true
})

#### Expandable list

The expandable list will show a list of predefined configuration values and allow the user to add to it.

```javascript
new Fields.ExpandableObject("Right footer links", "Default footer links shown on the right of the footer", {
    "browse": {
        "title": new Fields.Text('Set link title', 'Set link title', 'Browse', true),
        "href": new Fields.Text('Configure link', 'Configure link', '/categories', true),
        "newWindow": new Fields.Boolean('Open in new window', 'Open in new window', false, true)
    },
    "explore": {
        "title": new Fields.Text('Set link title', 'Set link title', 'Explore', true),
        "href": new Fields.Text('Configure link', 'Configure link', '/', true),
        "newWindow": new Fields.Boolean('Open in new window', 'Open in new window', false, true)
    }
}, {
    'tenantOverride': false,
    'suppress': true
})
```

#### Tree structure

The tree structure type will render a tree in the UI with help from the jstree library. The tree structure gets overridden every time a change in the configuration through the administration UI happens. This is different to how other configuration values

```javascript
new Fields.TreeStructure('Directory structure', 'Define the directory structure', [{
    "data": {
        "title": "__MSG__MEDICINE_AND_DENTISTRY__"
    },
    "attributes": {
        "id": "medicineanddentistry"
    },
    "children": [
        {
            "data": {
                "title": "__MSG__PRECLINICAL_MEDICINE__"
            },
            "attributes": {
                "id": "preclinicalmedicine"
            }
        }
    ]
}], {
    'tenantOverride': false,
    'suppress': true
})
```

### Full Example

As an example you can find the `content.js` configuration that handles default configuration for visibility when content is created:

```javascript
/*!
 * Copyright 2013 Apereo Foundation (AF) Licensed under the
 * Educational Community License, Version 2.0 (the "License"); you may
 * not use this file except in compliance with the License. You may
 * obtain a copy of the License at
 *
 *     http://opensource.org/licenses/ECL-2.0
 *
 * Unless required by applicable law or agreed to in writing,
 * software distributed under the License is distributed on an "AS IS"
 * BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
 * or implied. See the License for the specific language governing
 * permissions and limitations under the License.
 */

var Fields = require('oae-config/lib/fields');

module.exports = {
    'title': 'OAE Content Module',
    'visibility': {
        'name': 'Default Visibility Values',
        'description': 'Default visibility settings for new content',
        'elements': {
            'files': new Fields.List('Files Visibility', 'Default visibility for new files', 'public', [
                {
                    'name': 'Public',
                    'value': 'public'
                },
                {
                    'name': 'Logged in users',
                    'value': 'loggedin'
                },
                {
                    'name': 'Private',
                    'value': 'private'
                }
            ]),
            'collabdocs': new Fields.List('Collaborative Document Visibility', 'Default visibility for new Collaborative Documents', 'private', [
                {
                    'name': 'Public',
                    'value': 'public'
                },
                {
                    'name': 'Logged in users',
                    'value': 'loggedin'
                },
                {
                    'name': 'Private',
                    'value': 'private'
                }
            ]),
            'links': new Fields.List('Links Visibility', 'Default visibility for new links', 'public', [
                {
                    'name': 'Public',
                    'value': 'public'
                },
                {
                    'name': 'Logged in users',
                    'value': 'loggedin'
                },
                {
                    'name': 'Private',
                    'value': 'private'
                }
            ])
        }
    },
    'previews': {
        'name': 'Previews expire settings',
        'description': 'The expire settings for previews',
        'tenantOverride': true,
        'elements': {
            'expiration_minimum': new Fields.Text('Minimum Time', 'The minimum amount of time (in seconds) that a signature should be valid', 15*60),
            'expiration_maximum': new Fields.Text('Maximum Time', 'The maximum amount of time (in seconds) that a signature should be valid', 30*60)
        }
    }
};
```
