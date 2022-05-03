
# DocumentCloud Add-On — Identify/Redact Bad Redactions

This DocumentCloud Add-On allows you to select multiple documents or projects, creating annotations where bad redactions exist, and returning a csv file contraining the document_id's, page numbers, bounding boxes and the text. There’s an option to redact bad redactions and delete any existing bad redaction annotations.

## Files

### main.py

This is the file that implements the `bad-redactions` Add-On specific functionality.

### testing
Example invocation:
```
python main.py --username "DC.USERNAME" --password "DC.PASSWORD" --documents 12345678
```

### config.yaml

This is a YAML file which defines the data your Add-On expects to receive.
DocumentCloud will use it to show a corresponding form with the proper fields.
It uses the [JSON Schema](https://json-schema.org/) format, but allows you to
use YAML for convenience.  You may read more about JSON Schema, but here are
the basics to get started:

```yaml
# The title is the title of your Add-On
title: Bad Redactions
# The description will be shown above the form when activating the Add-On
description: This will create annotations where bad redactions exist, and export their the page numbers, bounding boxes and the text in a csv file. There’s an option to redact bad redactions and delete any existing bad redaction annotations.
# Type should always be object
type: object
# How does this add-on accept documents
# If more than one type is specified, the user will be prompted to choose one
documents:
  # By the currently selected documents
  - selected
# Properties are the fields for your form
properties:
    # the key is the name of the variable that will be returned to your code
    fix:
      title: Redact bad redactions found?
      type: boolean
```

At the top level you have the following properties:

* `title` - The title for your Add-On
* `description` - a description for your Add-On - will be displayed above the
  form when someone runs the add-on
* `type` - This should always be set to `object`
* `properties` - This is an object describing the data fields your add-on accepts

### requirements.txt

This is a standard `pip` `requirements.txt` file.  It allows you to specify
python packages to be installed before running the Add-On.  You may add any
dependencies your Add-On has here.  By default we install the
`python-documentcloud` API library and the `requests` HTTP request package.
You may upgrade the `python-documentcloud` version when new releases come out
in order to take advantage of new features.

### .github/workflows/run-addon.yml

This is the GitHub Actions configuration file for running the add-on.  It
references a reusable workflow from the
`MuckRock/documentcloud-addon-workflows` repository.  This workflow sets up
python, installs dependencies and runs the `main.py` to start the Add-On. It
accepts two inputs:
* `timeout` - Number of minutes to time out.  The default is `5`.  You may
  increase this if your add-on will run for longer than that.
* `python-version` - The version of python you would like to use.  Defaults to `3.10`.

To set an input:
```yaml
jobs:
  Run-Add-On:
    uses: MuckRock/documentcloud-addon-workflows/.github/workflows/update-config.yml@v1
    with:
      timeout: 30
```

It is recommended you use the reusable workflow in order to receive future
improvements to the workflow.  If needed you may fork the reusable workflow and
edit it as needed. If you do edit it, you should leave the first step in place,
which uses the UUID as its name, to allow DocumentCloud to identify the run.

It would be possible to make a similar workflow for other programming languages
if one wanted to write Add-Ons in a language besides Python.

### .github/workflows/update-config.yml

This is the GitHub Actions configuration file for updating the configuration
file.  It references a reusable workflow from the
`MuckRock/documentcloud-addon-workflows` repository.  This workflow sends a
`POST` request to DocumentCloud whenever a new `config.yaml` file is pushed to
the repository.  It accepts one input:
* `url` - The base URL for the DocumentCloud API.  The default is
  "https://api.www.documentcloud.org/api/".  It should only be changed if you
  are running your own instance of DocumentCloud.

### LICENSE

The license this code is provided under, the 3-Clause BSD License

## Reference

### Full parameter reference

This is a reference of all of the data passed in to the Add-On.  A single JSON
object is passed in to `main.py` as a quoted string.  The `init` function
parses this out and converts it to useful python objects for your `main`
function to use.  The following are the top level keys in the object.

* `token` - An access token which will be valid for 5 minutes, giving you API
  access authorized as the user who activated the add-on.  The `init` function
  uses this value to configure the DocumentCloud client object.

* `refresh_token` - A refresh token which will be valid for 1 day, giving you
  API access to new refresh tokens when they expire.  The `init` function uses
  this value to configure the DocumentCloud client object.

* `base_uri` - This can be used to point the API server to other instances,
  such as our internal staging server.  It should not be used unless you are
  running your own instance of DocumentCloud.  It is also used in the
  initialization of the DocumentCloud client.

* `auth_uri` - The corresponding `auth_uri` if a `base_uri` is specified.

*  `documents` - This is the list of Document IDs which is passed in to `main`

*  `query` - This is the search query which is passed in to `main`

*  `data` - This is the Add-On specific data, as defined when registering the
   Add-On with DocumentCloud.  It is passed in to `main` in the `params`
   dictionary under the key `data`

* `user` and `organization` - The user ID and organiation ID of the user who
  activated the Add-On.  They are also passed in to `main` through the `params`
  dictionary under the keys `user` and `organization` respectively.

* `id` - A UUID to uniquely identify this Add-On run.  It allows DocumentCloud
  to identify the run, as well as allowing the run to send back progress,
  status message and file updates.
