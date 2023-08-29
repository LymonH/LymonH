/**
 * @name MessageLoggerV2
 * @version 1.8.25
 * @invite NYvWdN5
 * @donate https://paypal.me/lighty13
 * @website https://1lighty.github.io/BetterDiscordStuff/?plugin=MessageLoggerV2
 * @source https://github.com/1Lighty/BetterDiscordPlugins/blob/master/Plugins/MessageLoggerV2/MessageLoggerV2.plugin.js
 * @updateUrl https://raw.githubusercontent.com/1Lighty/BetterDiscordPlugins/master/Plugins/MessageLoggerV2/MessageLoggerV2.plugin.js
 */
/*@cc_on
@if (@_jscript)
  // Offer to self-install for clueless users that try to run this directly.
  var shell = WScript.CreateObject('WScript.Shell');
  var fs = new ActiveXObject('Scripting.FileSystemObject');
  var pathPlugins = shell.ExpandEnvironmentStrings('%APPDATA%\\BetterDiscord\\plugins');
  var pathSelf = WScript.ScriptFullName;
  // Put the user at ease by addressing them in the first person
  shell.Popup('It looks like you\'ve mistakenly tried to run me directly. \n(Don\'t do that!)', 0, 'I\'m a plugin for BetterDiscord', 0x30);
  if (fs.GetParentFolderName(pathSelf) === fs.GetAbsolutePathName(pathPlugins)) {
    shell.Popup('I\'m in the correct folder already.\nJust go to settings, plugins and enable me.', 0, 'I\'m already installed', 0x40);
  } else if (!fs.FolderExists(pathPlugins)) {
    shell.Popup('I can\'t find the BetterDiscord plugins folder.\nAre you sure it\'s even installed?', 0, 'Can\'t install myself', 0x10);
  } else if (shell.Popup('Should I copy myself to BetterDiscord\'s plugins folder for you?', 0, 'Do you need some help?', 0x34) === 6) {
    fs.CopyFile(pathSelf, fs.BuildPath(pathPlugins, fs.GetFileName(pathSelf)), true);
    // Show the user where to put plugins in the future
    shell.Exec('explorer ' + pathPlugins);
    shell.Popup('I\'m installed!\nJust go to settings, plugins and enable me!', 0, 'Successfully installed', 0x40);
  }
  WScript.Quit();
@else @*/
/*
 * Copyright Â© 2019-2023, _Lighty_
 * All rights reserved.
 * Code may not be redistributed, modified or otherwise taken without explicit permission.
 */


const MLV2_TYPE_L1 = Symbol('MLV2_TYPE_L1');
const MLV2_TYPE_L2 = Symbol('MLV2_TYPE_L2');
const MLV2_TYPE_L3 = Symbol('MLV2_TYPE_L3');

module.exports = class MessageLoggerV2 {
  getName() {
    return 'MessageLoggerV2';
  }
  getVersion() {
    return '1.8.25';
  }
  getAuthor() {
    return 'Lighty';
  }
  getDescription() {
    return 'Saves all deleted and purged messages, as well as all edit history and ghost pings. With highly configurable ignore options, and even restoring deleted messages after restarting Discord.';
  }
  load() { }
  start() {
    let onLoaded = () => {
      try {
        if (global.ZeresPluginLibrary && !this.UserStore) this.UserStore = ZeresPluginLibrary.WebpackModules.getByProps('getCurrentUser', 'getUser');
        if (!global.ZeresPluginLibrary || !this.UserStore || !(this.localUser = this.UserStore.getCurrentUser())) setTimeout(onLoaded, 1000);
        else this.initialize();
      } catch (err) {
        ZeresPluginLibrary.Logger.stacktrace(this.getName(), 'Failed to start!', err);
        ZeresPluginLibrary.Logger.err(this.getName(), `If you cannot solve this yourself, contact ${this.getAuthor()} and provide the errors shown here.`);
        this.stop();
        XenoLib.Notifications.error(`[**${this.getName()}**] Failed to start! Try to CTRL + R, or update the plugin, like so\n![image](https://i.imgur.com/tsv6aW8.png)`, { timeout: 0 });
      }
    };
    this.pluginDir = (BdApi.Plugins && BdApi.Plugins.folder) || window.ContentManager.pluginsFolder;
    this.__isPowerCord = !!window.powercord && typeof BdApi.__getPluginConfigPath === 'function' || typeof global.isTab !== 'undefined';
    let XenoLibOutdated = false;
    let ZeresPluginLibraryOutdated = false;
    if (global.BdApi && BdApi.Plugins && typeof BdApi.Plugins.get === 'function' /* you never know with those retarded client mods */) {
      const versionChecker = (a, b) => ((a = a.split('.').map(a => parseInt(a))), (b = b.split('.').map(a => parseInt(a))), !!(b[0] > a[0])) || !!(b[0] == a[0] && b[1] > a[1]) || !!(b[0] == a[0] && b[1] == a[1] && b[2] > a[2]);
      const isOutOfDate = (lib, minVersion) => lib && lib._config && lib._config.info && lib._config.info.version && versionChecker(lib._config.info.version, minVersion) || typeof global.isTab !== 'undefined';
      let iXenoLib = BdApi.Plugins.get('XenoLib');
      let iZeresPluginLibrary = BdApi.Plugins.get('ZeresPluginLibrary');
      if (iXenoLib && iXenoLib.instance) iXenoLib = iXenoLib.instance;
      if (iZeresPluginLibrary && iZeresPluginLibrary.instance) iZeresPluginLibrary = iZeresPluginLibrary.instance;
      if (isOutOfDate(iXenoLib, '1.4.11')) XenoLibOutdated = true;
      if (isOutOfDate(iZeresPluginLibrary, '2.0.3')) ZeresPluginLibraryOutdated = true;
    }
    if (!global.XenoLib || !global.ZeresPluginLibrary || XenoLibOutdated || ZeresPluginLibraryOutdated) {
      this._XL_PLUGIN = true;
      const a = !!window.powercord && "function" == typeof BdApi.__getPluginConfigPath,
        b = BdApi.findModuleByProps("openModal", "hasModalOpen");
      if (b && b.hasModalOpen(`${this.getName()}_DEP_MODAL`)) return;
      const c = !global.XenoLib,
        d = !global.ZeresPluginLibrary,
        e = c && d || (c || d) && (XenoLibOutdated || ZeresPluginLibraryOutdated),
        f = (() => {
          let a = "";
          return c || d ? a += `Missing${XenoLibOutdated || ZeresPluginLibraryOutdated ? " and outdated" : ""} ` : (XenoLibOutdated || ZeresPluginLibraryOutdated) && (a += `Outdated `), a += `${e ? "Libraries" : "Library"} `, a
        })(),
        g = (() => {
          let a = `The ${e ? "libraries" : "library"} `;
          return c || XenoLibOutdated ? (a += "XenoLib ", (d || ZeresPluginLibraryOutdated) && (a += "and ZeresPluginLibrary ")) : (d || ZeresPluginLibraryOutdated) && (a += "ZeresPluginLibrary "), a += `required for ${this.getName()} ${e ? "are" : "is"} ${c || d ? "missing" : ""}${XenoLibOutdated || ZeresPluginLibraryOutdated ? c || d ? " and/or outdated" : "outdated" : ""}.`, a
        })(),
        h = BdApi.findModuleByDisplayName("Text") || BdApi.findModule(e => e.Text?.displayName === 'Text')?.Text,
        i = BdApi.findModuleByDisplayName("ConfirmModal"),... (224 KB left)
