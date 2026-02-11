# Migration Template Engine to JSONForm

We have the goal to migrate the template engine to jsonform.

## Template Engine structure data

```json
{
  "input_schema": {
    "groups": [
      {
        "name": "gh_connection",
        "label": "GitHub Connection",
        "fields": [
          {
            "info": "Github Scope",
            "name": "platform_feature__vcs_integration__uuid",
            "type": "textfield",
            "attrs": {
              "required": true,
              "maxLength": 100
            },
            "label": "Git Scope",
            "value": "",
            "hidden": false,
            "description": "Select the scope for this template.",
            "placeholder": "",
            "instantiation_data_path": ""
          }
        ]
      },
      {
        "name": "eleventy",
        "label": "11ty",
        "fields": [
          {
            "info": "Edge Application Name",
            "name": "az_name",
            "type": "textfield",
            "attrs": {
              "required": true,
              "maxLength": 100
            },
            "label": "Application Name",
            "value": "",
            "hidden": false,
            "validators": [
              {
                "regex": "(^(?![aA][zZ][iI][oO][nN]|[bB]2)[A-Za-z0-9\\-]{6,}$)",
                "errorMessage": "Minimum of 6 characters. Only alphanumeric characters and hyphens are allowed. No spaces. \"Azion\" and other reserved words and cannot be used."
              }
            ],
            "description": "Give a unique and easy-to-remember name to the application.",
            "placeholder": "Enter a name for your new Edge Application.",
            "instantiation_data_path": "envs.[0].value"
          }
        ]
      }
    ]
  }
}
```

## Template Engine with new JSONForm structure data

The new json using JSONForm structure data is a json object that contains the following properties:

- type: The type of the form.
- title: The title of the form.
- properties: The properties of the form.
- required: The required properties of the form.

```json
{
  "input_schema": {
    "type": "object",
    "title": "Edge Function Starter Kit",
    "properties": {
      "platform_feature__vcs_integration__uuid": {
        "type": "string",
        "label": "Git Scope",
        "description": "Select the scope for this template.",
        "placeholder": "",
        "maxLength": 100,
        "instantiation_data_path": ""
      },
      "input_text": {
        "type": "string",
        "label": "Input Text",
        "description": "Name of your Application",
        "placeholder": "Enter a name for your new Application.",
        "maxLength": 100,
        "pattern": "^(?:[A-Za-z]+)(?:[A-Za-z0-9 _]*)$",
        "error": "The use of accents and/or special characters is not allowed.",
        "instantiation_data_path": "envs.[0].value"
      },
      "input_password": {
        "type": "string",
        "format": "password",
        "label": "Input Password",
        "description": "User to enter a password.",
        "placeholder": "Enter a name for your new Application.",
        "instantiation_data_path": "envs.[1].value"
      },
      "input_number": {
        "type": "number",
        "label": "Number",
        "description": "Name of your Application",
        "placeholder": "Enter a name for your new Application.",
        "maxLength": 100,
        "pattern": "^(?:[A-Za-z]+)(?:[A-Za-z0-9 _]*)$",
        "error": "Only numbers are allowed.",
        "instantiation_data_path": "envs.[2].value"
      },
      "input_number_with_no_button": {
        "type": "number",
        "label": "Number no buttons",
        "description": "Name of your Application",
        "placeholder": "Enter a name for your new Application.",
        "showButtons": false,
        "maxLength": 100,
        "error": "Only numbers are allowed.",
        "instantiation_data_path": "envs.[3].value"
      },
      "input_number_integer": {
        "type": "integer",
        "label": "Integer number",
        "description": "ID Application",
        "placeholder": "Enter the ID for your new Application.",
        "minimum": 1000,
        "maximum": 2000,
        "error": "Only numbers are allowed.",
        "instantiation_data_path": "envs.[4].value"
      },
      "select": {
        "type": "string",
        "label": "Select",
        "description": "Select the option.",
        "placeholder": "Select the domain",
        "enum": [
          "www.azion.com",
          "console.azion.com",
          "stage-console.azion.com",
          "dev-console.azion.com"
        ],
        "instantiation_data_path": "envs.[5].value"
      },
      "select": {
        "type": "string",
        "label": "Region",
        "description": "Select the region for deployment.",
        "placeholder": "Select a region",
        "oneOf": [
          {
            "const": "us-eas",
            "title": "East"
          },
          {
            "const": "us-west",
            "title": "US West"
          },
          {
            "const": "eu-west",
            "title": "EU West"
          },
          {
            "const": "ap-south",
            "title": "Asia Pacific South"
          }
        ],
        "error": "Please select a valid region.",
        "instantiation_data_path": "envs.[6].value"
      },
      "textarea": {
        "type": "string",
        "options": {
          "multi": true
        },
        "label": "Textarea",
        "description": "Prompt for the user to enter a text.",
        "placeholder": "Enter your text here",
        "instantiation_data_path": "envs.[7].value"
      }
    },
    "required": ["input_text", "input_password"]
  }
}
```

> NOTE: When input numbers "showButtons" it is to allow visual arrow to the user to increment or decrement the value. When "useGrouping" it is to allow the user to enter a number without grouping.

Links:

- https://jsonforms.io/docs/uischema/controls
- https://jsonforms.io/docs/renderer-sets

## Sample template migration

### Edge Function Starter Kit

The original template engine data is:

```json
{
  "groups": [
    {
      "name": "edge",
      "label": "Edge Function Starter Kit",
      "fields": [
        {
          "info": "Edge Application Name",
          "name": "az_name",
          "type": "textfield",
          "attrs": {
            "required": true,
            "maxLength": 100
          },
          "label": "Application Name",
          "value": "",
          "hidden": false,
          "validators": [
            {
              "regex": "^(?:[A-Za-z]+)(?:[A-Za-z0-9 _]*)$",
              "errorMessage": "The use of accents and/or special characters is not allowed."
            }
          ],
          "description": "Name of your edge application",
          "placeholder": "Enter a name for your new Edge Application.",
          "instantiation_data_path": "envs.[0].value"
        }
      ]
    }
  ]
}
```

Migrated to JSONForm

```json
{
  "type": "object",
  "title": "Edge Function Starter Kit",
  "properties": {
    "az_name": {
      "type": "string",
      "label": "Application Name",
      "description": "Name of your Application",
      "placeholder": "Enter a name for your new Application.",
      "maxLength": 100,
      "pattern": "^(?:[A-Za-z]+)(?:[A-Za-z0-9 _]*)$",
      "error": "The use of accents and/or special characters is not allowed.",
      "instantiation_data_path": "envs.[0].value"
    }
  },
  "required": ["az_name"]
}
```

### Bot Manager and Tor Block Starter Kit

The original template engine data is:

```json
{
  "groups": [
    {
      "name": "firewall_tor_n_bot",
      "label": "Bot Manager and Tor Block Starter Kit",
      "fields": [
        {
          "info": "Edge Firewall Name",
          "name": "az_edge_firewall_name",
          "type": "textfield",
          "attrs": {
            "required": true,
            "maxLength": 255
          },
          "label": "Edge Firewall Name",
          "value": "",
          "hidden": false,
          "validators": [
            {
              "regex": "(^[A-Za-z0-9\\-]{6,}$)",
              "errorMessage": "Minimum of 6 characters. Only alphanumeric characters and hyphens are allowed. No spaces."
            }
          ],
          "description": "Name of the Edge Firewall to be created by this template",
          "placeholder": "Enter a name for your new Edge Firewall",
          "instantiation_data_path": "envs.[0].value"
        },
        {
          "info": "Domain ID(s)",
          "name": "az_edge_firewall_domains",
          "type": "textfield",
          "attrs": {
            "required": false,
            "maxLength": 255
          },
          "label": "Domain ID(s)",
          "value": "",
          "hidden": false,
          "description": "Enter the IDs of the domains you want to protect with this template. If no value is set, the new edge firewall won’t be linked to any domain. You can add it manually later.",
          "placeholder": "Separate multiple numbers using commas",
          "instantiation_data_path": "envs.[1].value"
        }
      ]
    }
  ]
}
```

Migrated to JSONForm

```json
{
  "type": "object",
  "title": "Bot Manager and Tor Block Starter Kit",
  "properties": {
    "az_edge_firewall_name": {
      "type": "string",
      "label": "Edge Firewall Name",
      "description": "Name of the Edge Firewall to be created by this template",
      "placeholder": "Enter a name for your new Edge Firewall",
      "maxLength": 255,
      "pattern": "^[A-Za-z0-9\\-]{6,}$",
      "error": "Minimum of 6 characters. Only alphanumeric characters and hyphens are allowed. No spaces.",
      "instantiation_data_path": "envs.[0].value"
    },
    "az_edge_firewall_domains": {
      "type": "number",
      "showButtons": false,
      "useGrouping": false,
      "label": "Domain ID(s)",
      "description": "Enter the IDs of the domains you want to protect with this template. If no value is set, the new edge firewall won't be linked to any domain. You can add it manually later.",
      "placeholder": "Separate multiple numbers using commas",
      "maxLength": 255,
      "instantiation_data_path": "envs.[1].value"
    }
  },
  "required": ["az_edge_firewall_name"]
}
```

### Astro Boilerplate

The original template engine data is:

```json
{
  "groups": [
    {
      "name": "gh_connection",
      "label": "GitHub Connection",
      "fields": [
        {
          "info": "Github Scope",
          "name": "platform_feature__vcs_integration__uuid",
          "type": "textfield",
          "attrs": {
            "required": true,
            "maxLength": 100
          },
          "label": "Git Scope",
          "value": "",
          "hidden": false,
          "description": "Select the scope for this template.",
          "placeholder": "",
          "instantiation_data_path": ""
        }
      ]
    },
    {
      "name": "astro",
      "label": "Astro Boilerplate",
      "fields": [
        {
          "info": "Edge Application Name",
          "name": "az_name",
          "type": "textfield",
          "attrs": {
            "required": true,
            "maxLength": 100
          },
          "label": "Application Name",
          "value": "",
          "hidden": false,
          "validators": [
            {
              "regex": "(^[A-Za-z0-9\\-]{6,}$)",
              "errorMessage": "Minimum of 6 characters. Only alphanumeric characters and hyphens are allowed. No spaces."
            }
          ],
          "description": "Give a unique and easy-to-remember name to the application.",
          "placeholder": "Enter a name for your new Edge Application.",
          "instantiation_data_path": "envs.[0].value"
        }
      ]
    }
  ]
}
```

Migrated to JSONForm

```json
{
  "type": "object",
  "title": "Astro Boilerplate",
  "properties": {
    "platform_feature__vcs_integration__uuid": {
      "type": "string",
      "label": "Git Scope",
      "description": "Select the scope for this template.",
      "placeholder": "",
      "maxLength": 100,
      "instantiation_data_path": ""
    },
    "az_name": {
      "type": "string",
      "label": "Application Name",
      "description": "Give a unique and easy-to-remember name to the application.",
      "placeholder": "Enter a name for your new Edge Application.",
      "maxLength": 100,
      "pattern": "^[A-Za-z0-9\\-]{6,}$",
      "error": "Minimum of 6 characters. Only alphanumeric characters and hyphens are allowed. No spaces.",
      "instantiation_data_path": "envs.[0].value"
    }
  },
  "required": ["az_name"]
}
```

## Custom Renderes not implemented

We have to create custom renderers for the following components:

1. Dynamic Select
2. Checkbox
3. Radio
4. Switch Toggle
