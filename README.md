/**
 * @name XenoLib
 * @description Simple library to complement plugins with shared code without lowering performance. Also adds needed buttons to some plugins.
 * @author 1Lighty
 * @authorId 239513071272329217
 * @version 1.4.11
 * @invite NYvWdN5
 * @donate https://paypal.me/lighty13
 * @source https://github.com/1Lighty/BetterDiscordPlugins/blob/master/Plugins/1XenoLib.plugin.js
 * @updateUrl https://raw.githubusercontent.com/1Lighty/BetterDiscordPlugins/master/Plugins/1XenoLib.plugin.js
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
      shell.Popup('I\'m in the correct folder already.', 0, 'I\'m already installed', 0x40);
   } else if (!fs.FolderExists(pathPlugins)) {
      shell.Popup('I can\'t find the BetterDiscord plugins folder.\nAre you sure it\'s even installed?', 0, 'Can\'t install myself', 0x10);
   } else if (shell.Popup('Should I copy myself to BetterDiscord\'s plugins folder for you?', 0, 'Do you need some help?', 0x34) === 6) {
      fs.CopyFile(pathSelf, fs.BuildPath(pathPlugins, '1XenoLib.plugin.js'), true);
      // Show the user where to put plugins in the future
      shell.Exec('explorer ' + pathPlugins);
      shell.Popup('I\'m installed!', 0, 'Successfully installed', 0x40);
   }
   WScript.Quit();

@else@*/
/*
 * Copyright Â© 2019-2022, _Lighty_
 * All rights reserved.
 * Code may not be redistributed, modified or otherwise taken without explicit permission.
 */

// eslint-disable-next-line no-undef
if (window.__XL_waitingForWatcherTimeout && !window.__XL_assumingZLibLoaded) clearTimeout(window.__XL_waitingForWatcherTimeout);

function _extractMeta(code/* : string */)/* : BDPluginManifest */ {
  const [firstLine] = code.split('\n');
  if (firstLine.indexOf('//META') !== -1) return _parseOldMeta(code);
  if (firstLine.indexOf('/**') !== -1) return _parseNewMeta(code);
  throw new /* ErrorNoStack */Error('No or invalid plugin META header');
}

function _parseOldMeta(code/* : string */)/* : BDPluginManifest */ {
  const [meta] = code.split('\n');
  const rawMeta = meta.substring(meta.indexOf('//META') + 6, meta.indexOf('*//'));
  try {
    const parsed = JSON.parse(rawMeta);
    if (!parsed.name) throw 'ENONAME';
    parsed.format = 'json';
    return parsed;
  } catch (err) {
    if (err === 'ENONAME') throw new /* ErrorNoStack */Error('Plugin META header missing name property');
    throw new /* ErrorNoStack */Error('Plugin META header could not be parsed');
  }
}

function _parseNewMeta(code/* : string */)/* : BDPluginManifest */ {
  const ret = {};
  let key = '';
  let value = '';
  try {
    const jsdoc = code.substr(code.indexOf('/**') + 3, code.indexOf('*/') - code.indexOf('/**') - 3);
    for (let i = 0, lines = jsdoc.split(/[^\S\r\n]*?(?:\r\n|\n)[^\S\r\n]*?\*[^\S\r\n]?/); i < lines.length; i++) {
      const line = lines[i];
      if (!line.length) continue;
      if (line[0] !== '@' || line[1] === ' ') {
        value += ` ${line.replace('\\n', '\n').replace(/^\\@/, '@')}`;
        continue;
      }
      if (key && value) ret[key] = value.trim();
      const spaceSeperator = line.indexOf(' ');
      key = line.substr(1, spaceSeperator - 1);
      value = line.substr(spaceSeperator + 1);
    }
    ret[key] = value.trim();
    ret.format = 'jsdoc';
  } catch (err) {
    throw new /* ErrorNoStack */Error(`Plugin META header could not be parsed ${err}`);
  }
  if (!ret.name) throw new /* ErrorNoStack */Error('Plugin META header missing name property');
  return ret;
}

module.exports = (() => {
  const canUseAstraNotifAPI = !!(global.Astra && Astra.n11s && Astra.n11s.n11sApi);
  // 1 day interval in milliseconds
  const USER_COUNTER_INTERVAL = 1000 * 60 * 60 * 24 * 1;
  /* Setup */
  const config = {
    main: 'index.js',
    info: {
      name: 'XenoLib',... (78 KB left)
