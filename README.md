content-editor
==============
[![DX version][dx-image]][dx-url]

Content Editor React extension for DX modules.

### Requirements
- the `qa-module` from https://github.com/Jahia/QA-Modules
- https://github.com/Jahia/dx-commons-webpack that will provide the shared library https://github.com/Jahia/javascript-components
- https://github.com/Jahia/content-media-manager
### How to install

- download, build and install [qa-module](https://github.com/Jahia/QA-Modules) . Only the `qa-module` subproject is needed.
- download, build and install [dx-commons-webpack](https://github.com/Jahia/dx-commons-webpack)
- download, build and install [content-media-manager](https://github.com/Jahia/content-media-manager/tree)
- build and install this module

[dx-image]: https://img.shields.io/badge/DX-7.3.0.0-blue.svg
[dx-url]: https://www.jahia.com

### Form generation

The content editor module has a GraphQL API to generate forms for content editing.
This API is exposed under the "forms" field in the GraphQL Query object type.

The generation of the form is done using the following algorithm.

For a given nodeType:

- If a CND definition existing in the JCR, it is used to generate a form definition dynamically.
- If DX modules define static forms that either override or define new forms, they will be merged in order of priority
with first the dynamically generated forms from the JCR definition (if it exists) and then with the static JSON form 
definitions that have a higher priority.
- Once this is done, the choicelist initializers will be called to generate the initial values for each field.

#### GraphQL API

Here's an example of a GraphQL query to generate a form for an existing node:

    {
      forms {
        form(nodeType: "qant:allFields", locale: "en", nodeIdOrPath: "e4809aae-6df1-4566-8d76-9fdfb97f258a") {
          nodeType
          fields {
            name
            selectorType
            i18n
            readOnly
            multiple
            mandatory
            valueConstraints {
              displayValue
              value {
                type
                string
              }
              properties {
                name
                value
              }
            }
            defaultValues {
              type
              string
            }
            targets {
              name
              rank
            }
          }
        }
      }
    }

The result will look something like this (truncated for length) : 

    {
      "data": {
        "forms": {
          "form": {
            "nodeType": "qant:allFields",
            "fields": [
              {
                "name": "sharedSmallText",
                "selectorType": "SmallText",
                "i18n": false,
                "readOnly": false,
                "multiple": false,
                "mandatory": false,
                "valueConstraints": [],
                "defaultValues": [],
                "targets": [
                  {
                    "name": "content",
                    "rank": 0
                  }
                ]
              },
              ...
              {
                "name": "choicelist",
                "selectorType": "Choicelist",
                "i18n": true,
                "readOnly": false,
                "multiple": false,
                "mandatory": false,
                "valueConstraints": [
                  {
                    "displayValue": "Choice 1",
                    "value": {
                      "type": "String",
                      "string": "choice1"
                    },
                    "properties": []
                  },
                  {
                    "displayValue": "Choice 2",
                    "value": {
                      "type": "String",
                      "string": "choice2"
                    },
                    "properties": []
                  },
                  {
                    "displayValue": "Choice 3",
                    "value": {
                      "type": "String",
                      "string": "choice3"
                    },
                    "properties": []
                  }
                ],
                "defaultValues": [
                  {
                    "type": "String",
                    "string": "choice1"
                  }
                ],
                "targets": [
                  {
                    "name": "content",
                    "rank": 5
                  }
                ]
              },
              ...
              {
                "name": "customField",
                "selectorType": "Text",
                "i18n": true,
                "readOnly": false,
                "multiple": true,
                "mandatory": true,
                "valueConstraints": [
                  {
                    "displayValue": "Value 1",
                    "value": {
                      "type": "String",
                      "string": "value1"
                    },
                    "properties": null
                  },
                  {
                    "displayValue": "Value 2",
                    "value": {
                      "type": "String",
                      "string": "value2"
                    },
                    "properties": null
                  },
                  {
                    "displayValue": "Value 3",
                    "value": {
                      "type": "String",
                      "string": "value3"
                    },
                    "properties": null
                  }
                ],
                "defaultValues": [
                  {
                    "type": "String",
                    "string": "value1"
                  }
                ],
                "targets": [
                  {
                    "name": "content",
                    "rank": -1
                  }
                ]
              }
            ]
          }
        }
      }
    }

#### Defining static forms in DX modules

A DX Module can define static forms by adding JSON files in the META-INF/dx-content-editor-forms directory. The files 
should have a meaning full name, for example for the qant:allFields node type we recommend replacing the colon (:) by an 
underscore so that the file name become qant_allFields.json.

Here's an example of a JSON static form definition coming from this [example module](https://github.com/Jahia/content-editor-formdef-test-module): 

    {
      "nodeType": "qant:allFields",
      "priority": 0.5,
      "fields": [
        {
          "name": "sharedSmallText",
          "selectorType": "SmallText"
        },
        {
          "name": "smallText",
          "removed": true,
          "targets": [
            {
              "name": "content"
            }
          ]
        },
        {
          "name": "sharedTextArea",
          "targets": [
            {
              "name": "content",
              "rank": 1.0
            }
          ]
        },
        {
          "name": "customField",
          "selectorType": "Text",
          "i18n": true,
          "readOnly": false,
          "multiple": true,
          "mandatory": true,
          "valueConstraints": [
            {
              "displayValue": "Value 1",
              "value": {
                "type": "String",
                "string": "value1"
              }
            },
            {
              "displayValue": "Value 2",
              "value": {
                "type": "String",
                "string": "value2"
              }
            },
            {
              "displayValue": "Value 3",
              "value": {
                "type": "String",
                "string": "value3"
              }
            }
          ],
          "defaultValues": [
            {
              "type": "String",
              "string": "value1"
            }
          ],
          "targets": [
            {
              "name": "content",
              "rank": -1.0
            }
          ]
        },
        {
          "name": "jcr:lastModifiedBy",
          "removed": true,
          "targets": [
            {
              "name": "metadata"
            }
          ]
        }
      ]
    }
    
Basically for a single node type, it can have : 
- a JCR definition, which will be used as the basis to generate a form dynamically
- one or multiple static JSON definition files, that will be merged, in order of priority to produce the final resulting form.        

There are some rules for the merging of the field properties. Basically the following cases may apply to a given property:
- case 1 : the property can always be overriden 
- case 2 : the property can only be overridden if its value is not true
- case 3 : the property can only be overridden if its value is not defined

Here are the association between cases and field properties:

| property         | case 1 | case 2 | case 3 |
| :--------------- | :----: | :----: | :----: |
| selectorType     | x      |        |        |
| i18n             |        |        | x      |
| readOnly         |        | x      |        |
| multiple         |        |        | x      |
| mandatory        |        | x      |        |
| defaultValues    | x      |        |        |
| targets          | x      |        |        |
| removed          | x      |        |        |
| selectorOptions  | x      |        |        |
| valueConstraints | x      |        |        |

As you can see these overrides will be done in order of priority so it is very important to remember that if you have 
multiple modules overriding the same node type (although this is not recommended but can be useful)

#### Priority

The priority field in the JSON static files is used to define in which order the definitions will be used to merge into
the final form. The JCR definition will always be used first, and then the JSON files will be used in order of ascending
priority. This makes it possible for multiple different modules to change a form definition and inject themselves where
they need in the form generation process.

#### Selector types

The selectorType property in a form field definition is used to define the UI component that will be used to edit the
field value. It is therefore very useful to set this value according to the needs of the project to build form UIs that 
are easy to use for end-users. In the (near) future it will also be possible to add new selector types in DX modules,
making the form UI expandable.
        
#### Selector options

Selector options make it possible to override the default options that are specified in the JCR definition. These options
are used for the moment to configure choicelist initializers. Here's an example of what selection options could look like:

For example if we have the following CND definition:

    [jnt:latestBlogContent] > jnt:content, jmix:blogContent, jmix:list, mix:title, jmix:renderableList, jmix:studioOnly, jmix:bindedComponent
     - j:subNodesView (string, choicelist[templates=jnt:blogPost,resourceBundle,image]) nofulltext  itemtype = layout

The equivalent part for the `j:subNodeView` property would look like this: 

    {
        "name" :  "j:subNodeView",
        "selectorType" : "Choicelist",
        "selectorOptions" : [
            { "name" : "templates", "value" : "jnt:blogPost" },
            { "name" : "resourceBundle", "value" : null },
            { "name" : "image", "value" : null }
        ],
        "targets" : [ { "name" : "layout", "rank" : 0 } ]
    }
    
Using static form JSON files you could override the selector options to for example change the templates allowed for this 
choicelist.

Important : selectorOptions can *only* be used with fields that have a CND definition !

If you are using purely JSON field definitions, you will instead simply have to use the valueConstraints array, which is
static and not dynamic.

#### Removing a field 

The `removed` property is a special one, which will actually remove a property from the resulting form definition.
