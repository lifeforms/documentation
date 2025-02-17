# Core Rule Set Documentation Repository

This repository contains the documentation for the OWASP ModSecurity Core Rule Set.

## For users of the CRS

The generated documentation is automatically updated at https://coreruleset.org/docs/. If you just want to read the documentation, you can find it there. If you intend to _contribute_ to the documentation, the rest of this page will guide you through it.

## Requirements

Just download latest version of [Hugo binary](https://gohugo.io/getting-started/installing/) for your OS (Windows, Linux, Mac) : it’s that simple. 

**Important: You will need the _extended_ version of Hugo, so be sure you download that one.**

## Cloning this repository

After getting hugo, just clone this repository to work locally. This way you can edit and verify quickly that everything is working properly before creating a new pull request.

To clone, use the *recursive* option so you will be getting also the theme to render the pages properly:

```bash
git clone --recursive git@github.com:coreruleset/documentation.git
```

## Editing locally

Now you have all in place to perform your local edits.

Everything is created using markdown, and you will normally use the `content` subdirectory to add your edits.

The theme has many shortcodes and others that you can use to simplify editing. You can get more information about it on [Hugo Relearn theme](https://themes.gohugo.io/themes/hugo-theme-relearn/).

You can run `hugo` to serve the pages, and while you edit and save, your changes will be refreshed!

Use:
```
hugo serve
```

Then check your edits on http://localhost:1313/documentation/.

## Creating a pull request

If you are a CRS developer, you can make a branch on the documentation repository.

If you are an outside contributor, the best way is to fork the [repository](https://github.com/coreruleset/documentation/) to your own GitHub account and work in your fork.

Once you are happy with your changes, [send a PR](https://github.com/coreruleset/documentation/pulls) with your changes.

After review and merging, the documentation is built and published [here](https://coreruleset.org/docs/) after max. 5 minutes.
