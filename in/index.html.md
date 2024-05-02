# Aphra

## A simple sitebuilder in Perl

Aphra is a simple yet powerful static site builder, developed in Perl. It
helps you create web pages seamlessly from templates, offering an efficient
way to generate dynamic static websites from structured content. This site
provides all the information you need to get started with Aphra, as well as
additional resources to help you along your journey.

## Why Aphra?

Aphra stands out for its simplicity and flexibility. Here are some key
features:

* **Template Toolkit Integration:** Aphra uses the Template Toolkit to process
templates, allowing for a robust templating system that can incorporate
layouts, fragments, and more.
* **Markdown Support:** Aphra seamlessly integrates with Markdown via the
Pandoc tool, converting content into HTML or any other format Pandoc supports.
* **Site Configuration:** You can store site-wide settings in a convenient
`site.yml` file, making global data available across all templates.
* **Front Matter:** Aphra templates can include a front matter section in YAML
format, providing page-specific data directly accessible within each template.
* **Server:** Aphra includes a simple serve command to launch a local server
for testing and previewing your site.
* **Customizable Extensions:** Easily manage which file extensions Aphra
recognises as templates or content, allowing for a versatile and adaptive
workflow.

## Getting Started

1. **Build Your Site:** Start by organizing your content into a directory
tree, including your templates, fragments, and layouts. Then, run `aphra
build` to process the files and generate a static website.
2. **Preview with `serve`:** To preview your work, use the `serve` command
to launch a local HTTP server and check the output files directly in your
browser.
3. **Configuration Options:** Modify how Aphra works by using various
command-line options, such as source, fragments, layouts, and more. These
help tailor the tool to your specific workflow.
4. **Documentation:** For further information on Aphra's commands, options,
and template processing, explore the detailed documentation available on
this site.