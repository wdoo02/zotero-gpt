# Zotero Addon Template

This is an addon/plugin template for [Zotero](https://www.zotero.org/).

[Documentation](https://zotero.yuque.com/books/share/8d230829-6004-4934-b4c6-685a7001bfa0/vec88d)(Chinese, provides English translation)

> 👍You are currently in `bootstrap` extension mode. To use `overlay` mode, plsase switch to `overlay` branch in git.

> ⚠️`overlay` mode will no longer be supported in the coming Zotero 7. Please use the `bootstrap` extension mode instead. See discussion here: https://groups.google.com/g/zotero-dev/c/TT_rcLVpQwg

## Features

- TypeScript support;
- Build addon settings and versions automatically;
- Build and reload code in Zotero automatically;
- Release to GitHub automatically(using [release-it](https://github.com/release-it/release-it));
- Extensive skeleton;
- Some sample code of UI and lifecycle.
- ⭐Compatibilities for Zotero 6 & Zotero 7.

## Quick Start Guide

- Fork this repo;
- Git clone the forked repo;
- Enter the repo folder;
- Modify the settings in `./package.json`, including:

```
  author,
  description,
  homepage,
  releasepage,
  updaterdf,
  addonName,
  addonID,
  addonRef
```

> Be careful to set the addonID and addonRef to avoid confliction.

- Run `npm install` to setup the plugin and install dependencies. If you don't have NodeJS installed, please download it [here](https://nodejs.org/en/);
- Run `npm run build` to build the plugin. The xpi for installation and the built code is under builds folder.

### Examples

#### Menu (file, edit, view, ...) & Right-click Menu (item, collection/library)

**File Menu**

![image](https://user-images.githubusercontent.com/33902321/208077117-e9ae3ca8-f9c7-4549-8835-1de5ea8c665f.png)

https://github.com/windingwind/zotero-addon-template/blob/574ce88b9fd3535a9d062db51cf16e99dda35288/src/views.ts#L52-L60

**Item Menu**

![image](https://user-images.githubusercontent.com/33902321/208078502-75d547b7-1cff-4538-802a-3d5127a8b617.png)

https://github.com/windingwind/zotero-addon-template/blob/574ce88b9fd3535a9d062db51cf16e99dda35288/src/views.ts#L23-L51

`Utils.UI.insertMenuItem` resolved the input object and inject the menu items.

Available types `menuFile`, `menuEdit`, ...: 
```ts
defaultMenuPopupSelectors: {
  menuFile: "#menu_FilePopup",
  menuEdit: "#menu_EditPopup",
  menuView: "#menu_viewPopup",
  menuGo: "#menu_goPopup",
  menuTools: "#menu_ToolsPopup",
  menuHelp: "#menu_HelpPopup",
  collection: "#zotero-collectionmenu",
  item: "#zotero-itemmenu",
},
```

You can choose an anchor element and insert before/after it using `insertPosition` and `anchorElement`. Default the insert position is the end of menu. 

```ts
insertMenuItem: (
  menuPopup: XUL.Menupopup | string,
  options: MenuitemOptions,
  insertPosition?: "before" | "after",
  anchorElement?: XUL.Element
) => boolean;
```

Full options you can use:

```ts
declare interface MenuitemOptions {
  tag: "menuitem" | "menu" | "menuseparator";
  id?: string;
  label?: string;
  // data url (chrome://xxx.png) or base64 url (data:image/png;base64,xxx)
  icon?: string;
  class?: string;
  styles?: { [key: string]: string };
  hidden?: boolean;
  disabled?: boolean;
  oncommand?: string;
  commandListener?: EventListenerOrEventListenerObject;
  // Attributes below are used when type === "menu"
  popupId?: string;
  onpopupshowing?: string;
  subElementOptions?: Array<MenuitemOptions>;
}
```

#### Preference, for both Zotero 6 and Zotero 7 (all in bootstrap)

Zotero 6 doesn't support preference pane injection in bootstrap mode, thus I write a register for Zotero 6 or lower.

You only need to maintain one `preferences.xhtml` which runs natively on Zotero 7 and let the plugin template handle when it is running on Zotero 6.

<table style="margin-left: auto; margin-right: auto;">
    <tr>
        <td>
          <img width="350px" src="https://user-images.githubusercontent.com/33902321/208080125-2a776a98-f427-4c81-8924-7877bf803e3d.png"/>
          <div>Zotero 7</div>
        </td>
        <td>
          <img width="300px" src="https://user-images.githubusercontent.com/33902321/208080491-b7006c08-2679-4f85-9a28-dba8e622d745.png"/>
          <div>Zotero 6</div>
        </td>
    </tr>
</table>

https://github.com/windingwind/zotero-addon-template/blob/574ce88b9fd3535a9d062db51cf16e99dda35288/src/views.ts#L63-L82

Call `Utils.Compat.registerPrefPane` when it's on Zotero 6. Please make sure `defaultXUL` is set `true` as Zotero 6 requires that.

Remember to call `Utils.Compat.unregisterPrefPane()` on plugin onload.

https://github.com/windingwind/zotero-addon-template/blob/574ce88b9fd3535a9d062db51cf16e99dda35288/src/views.ts#L88-L90

#### Create Elements API

The plugin template provides new APIs for bootstrap plugins. We have two reasons to use these APIs, instead of the `createElement/createElementNS`:

- In bootstrap mode, plugins have to clean up all UI elements on exit (disable or uninstall), which is very annoying. Using the `Utils.UI.createElement`, the plugin template will maintain these elements. Just `Utils.UI.removeAddonElements` on exit.
- Zotero 7 requires createElement()/createElementNS() → createXULElement() for remaining XUL elements, while on Zotero 6 doesn't support `createXULElement`. Using `Utils.UI.createElement`, it switches API depending on the current platform automatically.

Definition:

```ts
function createElement (
  doc: Document,
  tagName: string,
  namespace: "html" | "svg" | "xul"
) => XUL.Element | DocumentFragment | HTMLElement | SVGAElement;
```

There are more advanced APIs for creating elements in batch: `Uitls.UI.creatElementsFromJSON`. Input a element tree in JSON and return a fragment/element. These elements are also maintained by plugin template.

Definition:

```ts
function creatElementsFromJSON (
  doc: Document,
  options: ElementOptions
) => XUL.Element | DocumentFragment | HTMLElement | SVGAElement;
```

Available options:

```ts
declare interface ElementOptions {
  tag: string;
  id?: string;
  namespace?: "html" | "svg" | "xul";
  styles?: { [key: string]: string };
  directAttributes?: { [key: string]: string | boolean | number };
  attributes?: { [key: string]: string | boolean | number };
  listeners?: Array<
    | [
        string,
        EventListenerOrEventListenerObject,
        boolean | AddEventListenerOptions
      ]
    | [string, EventListenerOrEventListenerObject]
  >;
  checkExistanceParent?: HTMLElement;
  ignoreIfExists?: boolean;
  removeIfExists?: boolean;
  customCheck?: () => boolean;
  subElementOptions?: Array<ElementOptions>;
}
```

### Directory Structure

This section shows the directory structure of a template.

- All `.js/.ts` code files are in `./src`;
- Addon config files: `./addon/chrome.manifest`, `./addon/install.rdf`;
- UI files: `./addon/chrome/content/*.xul`. The `overlay.xul` also defines the main entrance;
- Locale files: `./addon/chrome/locale/*.dtd`;
- Resource files: `./addon/chrome/skin/default/__addonRef__/*.dtd`;
- Preferences file: `./addon/chrome/defaults/preferences/defaults.js`;
  > Don't break the lines in the `defaults.js`

```shell
│  .gitignore
│  .release-it.json # release-it conf
|  tsconfig.json    # https://code.visualstudio.com/docs/languages/jsconfig#
│  build.js         # esbuild
│  LICENSE
│  package.json     # npm conf
│  README.md        # readme
│  update.rdf       # addon update
│
├─.github           # github conf
│
├─addon             # addon dir
│  │  chrome.manifest  #addon conf
│  │  install.rdf   # addon install conf
│  │  bootstrap.js  # addon load/unload script, like a main.c
│  │
│  └─chrome
│      ├─content    # UI
│      │  │  preferences.xhtml
│      │  │
│      │  ├─icons
│      │  │      favicon.png
│      │  │      favicon@0.5x.png
│      │  │
│      │  └─scripts
│      └─locale     # locale
│         ├─en-US
│         │      overlay.dtd
│         │
│         └─zh-CN
│                overlay.dtd
│
├─builds            # build dir
│  └─.xpi
│
└─src               # source code
    │  index.ts     # main entry
    │  module.ts    # module class
    │  addon.ts     # base class
    │  events.ts    # events class
    │  utils.ts     # Utils class
    │  views.ts     # UI class
    └─ prefs.ts     # preferences class

```

### Build

```shell
# A release-it command: version increase, npm run build, git push, and GitHub release
# You need to set the environment variable GITHUB_TOKEN https://github.com/settings/tokens
# release-it: https://github.com/release-it/release-it
npm run release
```

Alternatively, build it directly using build.js: `npm run build`

### Build Steps

1. Clean `./builds`
2. Copy `./addon` to `./builds`
3. Esbuild to `./builds/addon/chrome/content/scripts`
4. Replace `__buildVersion__` and `__buildTime__` in `./builds/addon`
5. Zip the `./builds/addon` to `./builds/*.xpi`

### Debug

1. Copy zotero command line config file. Modify the commands.

```sh
cp zotero-cmd-default.json zotero-cmd.json
```

2. Setup addon development environment following this [link](https://www.zotero.org/support/dev/client_coding/plugin_development#setting_up_a_plugin_development_environment).

3. Build addon and restart Zotero with this npm command.

4. Launch Firefox 60
5. In Firefox, go to devtools, go to settings, click "enable remote debugging" and the one next to it that's also about debugging(or press `shift+F8`).
6. In Zotero, go to setting, advanced, config editor, look up "debugging" and click on "allow remote debugging"
7. In Firefox, click the hamburger menu in the top right -> web developer -> Connect...
8. Enter localhost:6100
9. Connect
10. Click "Inspect Main Process"

```sh
npm run restart
```

You can also debug code in these ways:

- Test code segments in Tools->Developer->Run Javascript;
- Debug output with `Zotero.debug()`. Find the outputs in Help->Debug Output Logging->View Output;
- UI debug. Zotero is built on the Firefox XUL framework. Debug XUL UI with software like [XUL Explorer](https://udn.realityripple.com/docs/Archive/Mozilla/XUL_Explorer).
  > XUL Documents:  
  > https://www.xul.fr/tutorial/  
  > http://www.xulplanet.com/

### Development

**Search for a Zotero API**  
Zotero docs are outdated or incomplete. Searching the source code of Zotero is unavoidable.  
Clone https://github.com/zotero/zotero and search the keyword globally. You can search the UI text in `.xul`/`.dtd` files, and then search the keys of the text value in `.js`/`.xul` files.

> ⭐The [zotero-types](https://github.com/windingwind/zotero-types) provides most frequently used Zotero APIs. It's included in this template by default.

## Disclaimer

Use this code under AGPL. No warranties are provided. Keep the laws of your locality in mind!

If you want to change the license, please contact me at wyzlshx@foxmail.com

Part of the code of this repo refers to other open-source projects within the allowed scope.

- zotero-better-bibtex(`d.ts`)

## Zotero Addons Build with the Template

- [zotero-better-notes](https://github.com/windingwind/zotero-better-notes): Everything about note management. All in Zotero.
- [zotero-pdf-preview](https://github.com/windingwind/zotero-pdf-preview): PDF Preview for Zotero.
- [zotero-pdf-translate](https://github.com/windingwind/zotero-pdf-translate): PDF Translation for Zotero 6.
- [zotero-tag](https://github.com/windingwind/zotero-tag): Automatically tag items/Batch tagging

- [zotero-theme](https://github.com/iShareStuff/ZoteroTheme): Customize Zotero theme