---
sticker: emoji//1f9e1
---
# Home

Github: [Webpage HTML Export](https://github.com/KosmosisDire/obsidian-webpage-export)

> [!warning]
> This documentation is VERY much a work in progress. As such it is incomplete.

## 주요 페이지들

### Dataview
#Feature

See some examples of dataview in action.

### Tasks 2

This file is simply being used to demo [Dataview](/features/dataview.html).

This is a task from a second file
This one is from a different file too, but it is complete

### Tasks

This file is simply being used to demo [Dataview](/features/dataview.html).

This is an incomplete task
This is another incomplete task
Oh this one is complete

### 6. File Picker
#Tutorial

The file picker is part of the [export modal](/getting-started/4.-export-modal.html) and allows you to select which files to include in the export. You can select a whole folder (in which case that folder's children will always be exported), or individual files.

Folders with all files selected are shown in blue, folders with only some files selected are shown in red.

> [!warning]
> Once you have selected the files you want, make sure to press the Save button!

### 7. Settings
#Tutorial

Learn all about the various configurations of the plugin

### 4. Export Modal
#Tutorial

Export to HTML [Export Mode](/getting-started/5.-export-modes.html)

This will export a file structure suitable for uploading to your own web server. Some options are only available in certain modes.

The export modal is the main place where you can select which files to include in your export, and select the location to export to. It can be opened using the [ribbon icon](/getting-started/1.-ribbon-icon.html), the [export settings command](/getting-started/3.-commands.html#Command_%60Set_html_export_settings%60), or the [context menu](/getting-started/2.-context-menus.html) of a file or folder.

On the left is the [file picker](/getting-started/6.-file-picker.html), on the right you can select an [export mode](/getting-started/5.-export-modes.html) and an export location. At the bottom right you can open the plugin's main [settings](/getting-started/7.-settings.html).

### 1. Ribbon Icon
#Tutorial

Learn how to use the ribbon icon to initiate an export.

### 3. Commands
#Tutorial

Learn about the 3 commands included with the plugin.

### 5. Export Modes
#Tutorial

Export modes (which can be set using the [export modal](/getting-started/4.-export-modal.html)) are sort of like presets, in that each one has certain default settings. However, some options are only available in a certain export mode. In the end you should read through the purpose of each mode and select the one that works for your use case.

This mode is meant for people creating a digital garden who plan to upload their HTML files onto a web server. This mode enables a bunch of optimizations that make [Incremental Export](/getting-started/7.-settings.html#Only_export_modified_files) possible, and enable your page to load fast over the internet. This is the preferred mode of export if at all possible.

The downside is that your files get broken up into lots of different files. All the scripts, styles, fonts, etc., are stored separately. This means that these files cannot be opened locally, they must be hosted on a web server.

This mode is meant for people wanting to open or share their documents locally. They don't intend to host them on a web server or create a website. Rather they want a way to share their documents in a way that maintains their interactivity and rich content.

The downside is that the produced HTML files are much larger, and some aspects of the navigation between files is compromised. Additionally, the [Graph View](/features/graph-view.html) and [Search Bar](/features/search-bar.html) are not supported in this mode.

### 0. Initiate an Export
#Tutorial

Get started with some actionable first steps to exporting your HTML files.

### 8. Export Process
#Tutorial

During an export you will see a progress bar like this. It will show you what is currently being worked on during the export.

> [!warning]
> This window MUST stay in the foreground. If you minimize it or leave it for another application it may pause the export.

Sometimes something will need your attention and a log will pop up with warnings or errors. Read the warning or error and determine if you need to do something to solve it. If the warning or error seems to be causing something to break, please file a report on GitHub here: [GitHub Issues](https://github.com/KosmosisDire/obsidian-webpage-export/issues)

### 2. Context Menus
#Tutorial

Learn how to use file and folder context menus to initiate an export.

### Snippets
#Feature

### Document Outline
#Component

The Table of Contents (shown in the right sidebar) allows you to view an outline of all the headers in the currently opened document. This is not the table of contents of the whole vault (that would be the [File Navigation](/features/file-navigation.html)). As a demonstration check out the outline to the right on this page.

You can click on the outline to skip to specific headers.

## This is an H2 header. 
You should always start your document with an H2 if you are using the inline title. And you should only ever have a single H1 in your document.

### This header is nested inside!
And that's it for the outline!

### File Navigation
#Component

The file navigation allows you to browse all the files in your exported vault. You can also filter the file view using the [Search Bar](/features/search-bar.html).

### Graph View
#Component

The graph view is a representation of the links in your vault, same as the graph view in obsidian. It is written from scratch so there are a few differences from obsidian's version, however the behavior is almost identical.

Caveats:
- Color groupings are not supported
- Only the global graph view is supported, not local graphs (yet)

### Search Bar
#Component

The search bar allows you to do a full text search of the whole vault. This search respect aliases, tags, headers, and links.

You can also filter your searches more specifically using certain key words:
- tag:tagname will search for all documents with a specific tag
- header:headername will search for all documents containing a specific header

### Sidebars
#Component

### Home

The home page :)

### Using the API

> [!warning]
> THE API IS STILL UNDER DEVELOPMENT!!!!
> DO NOT USE IN ANY PLUGIN OR PUBLIC PROJECT!!!!!

Render a file to an html string:
```js
const currentFile = app.workspace.getActiveFile();
let html = await WebpageHTMLExport.api.renderFileToString(currentFile, {});
// options are left empty see below for options definitions
```