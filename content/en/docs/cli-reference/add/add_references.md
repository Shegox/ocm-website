---
title: references
name: add references
url: /docs/cli/add/references/
date: 2022-10-19T11:39:28+01:00
draft: false
images: []
menu:
  docs:
    parent: add
toc: true
isCommand: true
---
### Usage

```
ocm add references [<options>] <target> {<resourcefile> | <var>=<value>}
```

### Options

```
      --addenv                 access environment for templating
  -h, --help                   help for references
  -s, --settings stringArray   settings file with variable settings (yaml)
      --templater string       templater to use (subst, spiff, go) (default "subst")
```

### Description


Add aggregation information specified in a resource file to a component version.
So far only component archives are supported as target.

Templating:
All yaml/json defined resources can be templated.
Variables are specified as regular arguments following the syntax <code>&lt;name>=&lt;value></code>.
Additionally settings can be specified by a yaml file using the <code>--settings <file></code>
option. With the option <code>--addenv</code> environment variables are added to the binding.
Values are overwritten in the order environment, settings file, command line settings.

Note: Variable names are case-sensitive.

Example:
<pre>
<command> <options> -- MY_VAL=test <args>
</pre>

There are several templaters that can be selected by the <code>--templater</code> option:
- <code>go</code> go templating supports complex values.

  <pre>
    key:
      subkey: "abc {{.MY_VAL}}"
  </pre>

- <code>spiff</code> [spiff templating](https://github.com/mandelsoft/spiff).

  It supports complex values. the settings are accessible using the binding <code>values</code>.
  <pre>
    key:
      subkey: "abc (( values.MY_VAL ))"
  </pre>

- <code>subst</code> simple value substitution with the <code>drone/envsubst</code> templater.

  It supports string values, only. Complex settings will be json encoded.
  <pre>
    key:
      subkey: "abc ${MY_VAL}"
  </pre>



This command accepts reference specification files describing the references
to add to a component version.

The resource specification supports the following blob input types, specified
with the field <code>type</code> in the <code>input</code> field:

- Input type <code>dir</code>

  The path must denote a directory relative to the resources file, which is packed
  with tar and optionally compressed
  if the <code>compress</code> field is set to <code>true</code>. If the field
  <code>preserveDir</code> is set to true the directory itself is added to the tar.
  If the field <code>followSymLinks</code> is set to <code>true</code>, symbolic
  links are not packed but their targets files or folders.
  With the list fields <code>includeFiles</code> and <code>excludeFiles</code> it is
  possible to specify which files should be included or excluded. The values are
  regular expression used to match relative file paths. If no includes are specified
  all file not explicitly excluded are used.

  This blob type specification supports the following fields:
  - **<code>path</code>** *string*

    This REQUIRED property describes the file path to directory relative to the
    resource file location.

  - **<code>mediaType</code>** *string*

    This OPTIONAL property describes the media type to store with the local blob.
    The default media type is application/x-tar and
    application/gzip if compression is enabled.

  - **<code>compress</code>** *bool*

    This OPTIONAL property describes whether the file content should be stored
    compressed or not.

  - **<code>preserveDir</code>** *bool*

    This OPTIONAL property describes whether the specified directory with its
    basename should be included as top level folder.

  - **<code>followSymlinks</code>** *bool*

    This OPTIONAL property describes whether symbolic links should be followed or
    included as links.

  - **<code>excludeFiles</code>** *list of regex*

    This OPTIONAL property describes regular expressions used to match files
    that should NOT be included in the tar file. It takes precedence over
    the include match.

  - **<code>includeFiles</code>** *list of regex*

    This OPTIONAL property describes regular expressions used to match files
    that should be included in the tar file. If this option is not given
    all files not explicitly excluded are used.


- Input type <code>docker</code>

  The path must denote an image tag that can be found in the local
  docker daemon. The denoted image is packed as OCI artefact set.

  This blob type specification supports the following fields:
  - **<code>path</code>** *string*

    This REQUIRED property describes the image name to import from the
    local docker daemon.

  - **<code>repository</code>** *string*

    This OPTIONAL property can be used to specify the repository hint for the
    generated local artefact access. It is prefixed by the component name if
    it does not start with slash "/".

- Input type <code>dockermulti</code>

  This input type describes the composition of a multi-platform OCI image.
  The various variants are taken from the local docker daemon. They should be
  built with the buildx command for cross platform docker builds.
  The denoted images, as well as the wrapping image index is packed as OCI artefact set.

  This blob type specification supports the following fields:
  - **<code>variants</code>** *[]string*

    This REQUIRED property describes a set of  image names to import from the
    local docker daemon used to compose a resulting image index.

  - **<code>repository</code>** *string*

    This OPTIONAL property can be used to specify the repository hint for the
    generated local artefact access. It is prefixed by the component name if
    it does not start with slash "/".

- Input type <code>file</code>

  The path must denote a file relative the resources file.
  The content is compressed if the <code>compress</code> field
  is set to <code>true</code>.

  This blob type specification supports the following fields:
  - **<code>path</code>** *string*

    This REQUIRED property describes the file path to the helm chart relative to the
    resource file location.

  - **<code>mediaType</code>** *string*

    This OPTIONAL property describes the media type to store with the local blob.
    The default media type is application/octet-stream and
    application/gzip if compression is enabled.

  - **<code>compress</code>** *bool*

    This OPTIONAL property describes whether the file content should be stored
    compressed or not.


- Input type <code>helm</code>

  The path must denote an helm chart archive or directory
  relative to the resources file.
  The denoted chart is packed as an OCI artefact set.
  Additional provider info is taken from a file with the same name
  and the suffix <code>.prov</code>.

  If the chart should just be stored as archive, please use the
  type <code>file</code> or <code>dir</code>.

  This blob type specification supports the following fields:
  - **<code>path</code>** *string*

    This REQUIRED property describes the file path to the helm chart relative to the
    resource file location.

  - **<code>version</code>** *string*

    This OPTIONAL property can be set to configure an explicit version hint.
    If not specified the versio from the chart will be used.
    Basically, it is a good practice to use the component version for local resources
    This can be achieved by using templating for this attribute in the resource file.

- Input type <code>ociImage</code>

  The path must denote an OCI image reference.

  This blob type specification supports the following fields:
  - **<code>path</code>** *string*

    This REQUIRED property describes the OVI image reference of the image to
    import.

  - **<code>repository</code>** *string*

    This OPTIONAL property can be used to specify the repository hint for the
    generated local artefact access. It is prefixed by the component name if
    it does not start with slash "/".

- Input type <code>spiff</code>

  The path must denote a [spiff](https://github.com/mandelsoft/spiff) template relative the the resources file.
  The content is compressed if the <code>compress</code> field
  is set to <code>true</code>.

  This blob type specification supports the following fields:
  - **<code>path</code>** *string*

    This REQUIRED property describes the file path to the helm chart relative to the
    resource file location.

  - **<code>mediaType</code>** *string*

    This OPTIONAL property describes the media type to store with the local blob.
    The default media type is application/octet-stream and
    application/gzip if compression is enabled.

  - **<code>compress</code>** *bool*

    This OPTIONAL property describes whether the file content should be stored
    compressed or not.

  - **<code>values</code>** *map[string]any*

    This OPTIONAL property describes an additioanl value binding for the template processing. It will be available
    under the node <code>values</code>.

  - **<code>libraries</code>** *[]string*

    This OPTIONAL property describes a list of spiff libraries to include in template
    processing.




### See Also

* [ocm add](/docs/cli/add)	 &mdash; Add resources or sources to a component archive

