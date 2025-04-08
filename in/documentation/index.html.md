[% TAGS star -%]

# Documentation for `aphra`

## NAME

aphra - Simple static sitebuilder in Perl

## SYNOPSIS

```
$ aphra build
```

## DESCRIPTION

`aphra` is a simple static sitebuilder written in Perl. It takes a directory
tree of input template, processes them using the Template Toolkit and outputs
a directory tree of expanded templates.

## COMMANDS

`aphra` has two command modes.

- **build**

    Examines the files in the input directory, processes them in various ways
    and leaves the processed version in the output directory.

    See most of the rest of this documentation for the gory details.

- **serve**

    Starts a local HTTP server which serves the files currently in the
    output directory.

    You need to have [App::HTTPThis](https://metacpan.org/pod/App%3A%3AHTTPThis) installed in order for this to work.

## OPTIONS

`aphra` takes a number of command line options which alter the way that it
works.

- **source**

    The main directory that contains the input files. Any file that is found in
    this directory will be processed and an equivalent output file will be
    created in the **target** directory. The default value for this option is "in".

    ```
    $ aphra --in some/other/directory
    ```

- **fragments**

    A directory that contains other templates which are used in the processing
    of the source templates. For example, you might have a template in the source
    directory called `index.html.tt` which includes other templates called
    `menu.html.tt` and `footer.html.tt`. These templates shouldn't be put in
    the source directory (as they would then be processed and written to the
    target directory, but if they are put in the fragments directory, they will
    be found by the template processor. The default value for this option is
    "fragments".

    ```
    $ aphra --fragments some/directory/of/fragments
    ```

- **layouts**

    A directory that contains templates which are used to control the layout of
    the files which are produced. These are typically used with the `WRAPPER`
    directive in the Template Toolkit. There's really no difference in handling
    the layouts and fragments directories, but I find it useful to keep the two
    types of template separate. The default value for this option is "layouts".

    ```
    $ aphra --layouts some/directory/of/layouts
    ```

- **wrapper**

    The name of a template that will be used as the main `WRAPPER` for the rest
    of the templates. This template should usually contain a `[% content %]` tag
    and will normally be stored in the layouts directory. See the [Template
    Toolkit](https://metacpan.org/pod/Template%0AToolkit) documentation for more details of the `WRAPPER` directive. The
    default value for this option is "page".

    ```perl
    $ aphra --wrapper my_awesome_layout
    ```

- **target**

    The name of the top-level directory where the output files will be written.
    Any directory structure under the source directory will be recreated under
    this directory. The default value for this option is "docs" (as that works
    well with Github pages).

    ```
    $ aphra --target some/output/directory
    ```

- **output**

    The output format that pages will be created in. This can be any output
    format that `pandoc` understands. The default value is "html" and as long
    as you're creating web sites, there's going to be very little reason to change
    that.

    ```
    $ aphra --output docx
    ```

- **extensions**

    Allows you to configure the extensions that are recognised as templates by
    the program. See ["Processing Templates"](#processing-templates) below for more details of this. By
    default, templates with the extension `.tt` are processed by the standard
    Template Toolkit processor and templates with the extension `.md` are
    converted from Markdown to the output format (see above) using `pandoc`
    before they are processed by the Template Toolkit.

    Extensions options are more complex than other options. They consist of an
    extension string and the name of a text format as recognised by `pandoc`.
    These two parts are separated by an equals sign. This option can be
    repeated in order to define multiple extensions.

    ```
    $ aphra --extensions md=markdown --extensions tt=template
    ```

## PROCESSING TEMPLATES

I've tried to make the template processing as simple as possible. Here's how it
works.

The program finds all the files under the source directory. For each file it
finds, it examines the extension of the file. If the extension doesn't match
any of the defined extensions, then the file is copied to a mirror directory
under the output directory.

If the extension does match one of the defined extensions, then one of two
things is true.

The extension matches the special format "template". In this case, the
template is just processed by the Template Toolkit.

The extension matches some other format name. In this case, the template is
processed by `pandoc` to convert the named format to the output format
(html by default) before being processed by the Template Toolkit.

In both cases where the Template Toolkit is involved, the output file is
placed under the output directory in a position that mirrors the position
of the input file under the iput directory. The output file is also renamed
to remove the extension that marked the input file as a template.

An example might make this clearer. Imagine we have all of the default
configuration options and the following directory tree.

```
src/index.html.tt
src/style.css
src/about/index.html.tt
fragments/index_text.md
fragments/about_text.md
```

And assume that `index.html.tt` includes the Template Toolkit directive
`[% INCLUDE index_text.md %]` and `about/index.html.tt` contains a similar
directive refering to `about_text.md`. Here's what will happen.

- `src/index.html.tt` is found. Its extension, `.tt`, matches one of the
defined extensions, so it is processed by the Template Toolkit.
- As part of this processing, the Template Toolkit needs to process
`index_text.md`. This template is found in the fragments directory, but
its extension, `.md`, means that it is pre-processed by `pandoc` (converting
Markdown to HTML) before it is processed by the Template Toolkit.
- The output from processing `src/index.html.tt` is written to
`docs/index.html`. The `.tt` extension is removed.
- `src/style.css` is found. As its extension is not one of the defined ones,
it is simply copied to `docs/style.css`.
- `src/about/index.html.tt` is found. It is processed in a very similar way
to `src/index.html` (including the processing of `fragments/about_text.md`
and the output is written to `docs/about/index.html`.

After the processing is complete, we have the following in the `docs`
directory:

```
docs/index.html
docs/style.css
docs/about/index.html
```

## SITE CONFIGURATION FILE - site.yml

You can store site-wide configuration and data in a file called `site.yml`,
which should be in the same directory as the various directories discussed
above.

The file is in YAML format. Any variables defined inside this file will be
available inside your templates within the \`site\` variable. For example, your
\`site.yml\` could contain the following definition:

```
name: My Cool Site
```

And you could access that text inside your templates with markup like this:

```
<h1><% site.name %></h1>
```

### Special variables in site.yml

There are a few special variables that you can define in your \`site.yml\` file
which will be used by the program.

- uri, protocol, host, port

    These are used to construct the base URL for the site. If you don't define
    these, then the program will try to guess them. It will use the value of the
    \`uri\` variable if it is defined. If not, it will use the values of the
    \`protocol\`, \`host\` and \`port\` variables. If any of these aren't defined, it
    will use the values "https", the result of calling \`hostname\` and no port
    (which is, effectively, port 80) respectively.

- redirects

    This is a list of URLs that should be redirected to other URLs. Each entry in
    the list should be a hash containing the keys \`from\` and \`to\`. For example:

    ```
    redirects:
      - from: /old/page
        to: /new/page
    ```

    The value of "from" should be the path part of the URL that you want to
    redirect and the value of "to" can be either a full URL or a path part of a
    URL. If it is a path part, then the base URL of the site will be prepended to
    it.

    Aphra implements these redirects by creating stub HTML files in the output
    directory which contain a meta refresh tag that redirects the browser to the
    new URL. Redirects are created before other files are processed, so any files
    that have the same URL as a redirect will overwrite the redirect stub HTML
    file.

## FRONT MATTER IN INDIVIDUAL PAGES

At the start of the template for a page, you can add a data section which
contains variable definitions. This section is preceded and followed by lines
consisting of three dashes. The variables are defined using YAML format. The
variables defined in this section are available within templates by using the
\`page\` variable. For example, your template could start with this definition:

```
---
title: Important Page
---
```

And you could access that text inside your templates with markdown like this:

```
## <% page.title %>
```

### Special values in front matter

If your front matter includes a variable called `layout`, then that will
override the default layout for this single page. You should ensure there
is a file of the correct name in the `layouts` directory.

## AUTHOR

Dave Cross <dave@perlhacks.com>

## COPYRIGHT AND LICENCE

Copyright (c) 2017-2024, Magnum Solutions Ltd. All Rights Reserved.

This library is free software; you can redistribute it and/or modify it
under the same terms as Perl itself.
